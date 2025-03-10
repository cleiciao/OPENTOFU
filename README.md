
# Instalação e configuração do OPENTOFU em ambiente Proxmox para automatizar a criação de VMs (EM DESENVOLVIMENTO)

<p align="center">
  <img src="https://www.proxmox.com/images/proxmox/Proxmox_logo_standard_hex_400px.png" alt="Proxmox Logo"><br><br>
  <img src="https://www.bigdatawire.com/wp-content/uploads/2024/01/opentofu-logo.png" alt="OpenTofu Logo">
</p>


Cleicião Diego Moro<br>
Linkedin: www.linkedin.com/in/cleicião-diego-moro <br>
Email:  cleiciaodiego@gmail.com
<br><br>


Chave PIX para contribuição no projeto: cleiciaodiego@gmail.com <br>



 🤩🤩
Nesse artigo irei demonstrar como trabalhar com OPENTOFU junto com o  Proxmox para criação automatizada de maquinas virtuais.<br>
⚠️ ATENÇÃO: Nesse artigo não será abordado a instalação do Proxmox, isso será tratado em outro artigo.

Conteudo apresentado nesse artigo

#1 - Instalação do OPENTOFU no Servidor Proxmox<br>
#2 - Criando as configurações no arquivo main.tf<br> 
#3 - Criação da VM template com cloudinit<br>
#4 - Realizando testes para criação de VMs



Site Oficial Opentofu: https://opentofu.org<br>
Site Oficial Proxmox: https://www.proxmox.com/en/

============================================================================================================



#1 - Instalação do OPENTOFU no Servidor Proxmox.<br>
Link para o passo a passo: https://opentofu.org/docs/intro/install/deb/ <br>

```bash

# Download the installer script:
curl --proto '=https' --tlsv1.2 -fsSL https://get.opentofu.org/install-opentofu.sh -o install-opentofu.sh
# Alternatively: wget --secure-protocol=TLSv1_2 --https-only https://get.opentofu.org/install-opentofu.sh -O install-opentofu.sh

# Give it execution permissions:
chmod +x install-opentofu.sh

# Please inspect the downloaded script

# Run the installer:
./install-opentofu.sh --install-method deb

# Remove the installer:
rm -f install-opentofu.sh


```


===========================================================================================================


#2 - Criando as configurações no arquivo main.tf<br>

Antes de mais nada vamos criar um usuário e um token para API no Proxmox.<br>
⚠️ ATENÇÃO: Evite usar o root para tarefas rotineiras, crie um usuário para manutenção e suporte ao servidor.<br>


Acesso seu Proxmox e crie um usuário que iremos utilizar para nosso artigo.

Para criar o usuário você pode realizar via CLI ou via WEBGUI, aqui irei realizar via CLI. Iremos utilizar no arquivo main.tf
mencionado no inicio do artigo.

```bash

#Criando usuário.Dica: Use um gerador de senhas :)
pveum user add opentofu@pam --password "passwd"

#Atribuindo permissão ao usuário
pveum aclmod / -user opentofu@pam -role Administrator

```

                 

Conteúdo do arquivo main.tf

```bash
terraform {
  required_providers {
    proxmox = {
      source  = "Telmate/proxmox"
      version = "~> 2.9.11"
    }
  }
}

provider "proxmox" {
  pm_api_url      = "https://IP-DO-SERVIDOR:8006/api2/json" #URL DE ACESSO AO PVE
  pm_user         = "userapi@pam" #USUARIO
  pm_password     = "pass" #SENHA
  pm_tls_insecure = true 
}

resource "proxmox_vm_qemu" "vm1" {
  name        = "web" #NOME PARA A NOVA VM
  target_node = "pve01" #NOME DO HOST PROXMOX
  vmid        = 1001 #ID DA NOVA VM
  clone       = "vm-base" #NOME DA VM TEMPLATE

  cores       = 2 #CPU PARA A MEMORIA
  memory      = 2048 #MEMORIA
  sockets     = 1 #QTD DE SOCKETS DO SERVIDO

  network {
    model  = "virtio" #TIPO DA REDE
    bridge = "vmbr0" #INTERFACE
  }

  disk {
    size  = "10G" #TAMANHO DO DISK
    type  = "scsi" #TIPO DO DISCO
    storage = "local-lvm" #STORAGE ONDE SERÁ CRIADO
  }

  os_type  = "cloud-init" #TIPO DA VM
}

```

===========================================================================================================


#3 - Criação da VM template com cloudinit

Link para download das imagens debian cloudinit: https://cdimage.debian.org/images/cloud/

⚠️ ATENÇÃO: Antes de criarmos a VM template precisamos instalar o pacote libguestfs-tools no Proxmox para manipulara as imagens de VMs.

```bash
#Instalação do Pacote
apt install libguestfs-tools -y

#Download da Imagem Debian.
wget https://cloud.debian.org/images/cloud/bookworm/latest/debian-12-generic-amd64.qcow2

#Agora vamos instalar o qemu-guest-agent na imagem baixada
virt-customize --add  debian-12-generic-amd64.qcow2  --install qemu-guest-agent

#Criando a VM, as configuração da VM são básicas mas você alterar de acordo com a necessidade.

qm create 9001 \
 --name debian12-cloud-img\
 --numa 0 \
 --ostype l26 \
  --cpu cputype=host \
  --cores 2 \
  --sockets 1 \
  --memory 2018 \
  --net0 virtio,bridge=vmbr0


#Vamos importar a imagem que baixando anteriormente, você precisar estar no mesmo diretorio onde baixou a imagem.

qm importdisk 9001 debian-12-generic-amd64.qcow2 local-lvm


#Definindo configurações da VM
qm set 9001 \
  --scsihw virtio-scsi-pci \
  --scsi0 local-lvm:vm-9001-disk-0 \
  --ide2 local-lvm:cloudinit \
  --boot c \
  --bootdisk scsi0 \
  --vga std \
  --agent enabled=1

#Vocẽ pode aumentar o tamanho do disco (opcional)
qm disk resize 9001 scsi0 +100G

#Transformando VM em Templante
qm template 9001

```
===========================================================================================================

#4 - Realizando testes para criação de VMs

Para criar as VMs com o Opentofu, você pode executar os comando diretamente no Proxmox ou diretamente na sua maquina desde que tenha comunicação com o host.

Irei executar da minha maquina mesmo

```bash

tofu init

tofu apply -auto-approve

```
Se tudo correr bem você irá receber retorno como esse no terminal.

proxmox_vm_qemu.vm1: Creating..

E pode observer na tela do seu Proxmox a magica acontecendo.
