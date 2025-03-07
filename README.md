#Em desenvolvimento

# Instalação e configuração do OPENTOFU em ambiente Proxmox para automatizas a criação de VMs/CTs


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



#1 - Instalação do OPENTOFU no Servidor Proxmox.<br>

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

