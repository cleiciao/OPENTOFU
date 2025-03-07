
# Instalação e configuração do OPENTOFU em ambiente Proxmox para automatizas a criação de VMs/CTs (EM DESENVOLVIMENTO)

![alt text](https://www.proxmox.com/images/proxmox/Proxmox_logo_standard_hex_400px.png)</br></br></br>


Cleicião Diego Moro<br>



 🤩🤩
Nesse artigo irei demonstrar como trabalhar com OPENTOFU junto com o  Proxmox para criação automatizada de maquinas virtuais.<br>
⚠️ ATENÇÃO: Nesse artigo não será abordado a instalação do Proxmox, isso será tratado em outro artigo.

Conteudo apresentado nesse artigo

#1 - Instalação do OPENTOFU no Servidor Proxmox<br>
#2 - Criando as configurações no arquivo main.tf<br> 
#3 - Criação da VM template com cloudinit<br>
#4 - Realizando testes para criação de VMs



Site Oficial da ferramenta: https://opentofu.org<br>


================================================================================================================================

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


===============================================================================================================================

#2 - Criando as configurações no arquivo main.tf<br>

Antes de mais nada vamos criar um usuário e um token para API no Proxmox.<br>
⚠️ ATENÇÃO: Evite usar o root para tarefas rotineiras, crie um usuário para manutenção e suporte ao servidor.<br>


Acesso seu Proxmox e crie um usuário que iremos utilizar para nosso artigo.

Datacenter >
           Users >
                 

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
