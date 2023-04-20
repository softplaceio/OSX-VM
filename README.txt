------- ------- ------- ------- ------- ------- ------- 
@ GitHub
------- ------- ------- ------- ------- ------- ------- 

Objetivo: Construir um Ambiente do OSX + Developer
Ambiente: Manjaro (Gnome)
Tecnologias: Docker + KVM + QEMU + OpenCore

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> Alternativas
------- ------- ------- ------- ------- ------- ------- 

# Kholia (KVM + QEMU)
https://github.com/kholia/OSX-KVM/

# SickCodes (Docker + KVM + QEMU)
https://github.com/sickcodes/Docker-OSX/

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> ISO
------- ------- ------- ------- ------- ------- ------- 

https://archive.org/details/macos-collection

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> HackingTosh
------- ------- ------- ------- ------- ------- ------- 

https://www.olarila.com/

------- ------- ------- ------- ------- ------- ------- 
@ Documentações
------- ------- ------- ------- ------- ------- ------- 

# Docker
https://docs.docker.com/get-started/overview/

# Docker -> Comandos (CLI)
https://docs.docker.com/engine/reference/run/

# Kubernetes
https://kubernetes.io/pt-br/docs/home/

# KVM
https://www.linux-kvm.org/page/Documents

# QEMU
https://www.qemu.org/documentation/

# OpenCore
https://dortania.github.io/OpenCore-Install-Guide/prerequisites.html#prerequisites

# Clover
https://docs.clover.com/docs

# Portainer
https://docs.portainer.io/

# Podman
https://docs.podman.io/en/latest/

------- ------- ------- ------- ------- ------- ------- 
@ Instalação e Configuração
------- ------- ------- ------- ------- ------- ------- 

# Linux -> Manjaro -> Atualização
sudo pacman -Syyu

# Linux -> Manjaro -> Definir Melhor Mirror
sudo pacman-mirrors -g -c "Brazil"; sudo pacman -Syyu

# Linux -> Instalar Pacotes
sudo pamac install docker
sudo pamac install virt-manager virt-viewer dnsmasq vde2 bridge-utils openbsd-netcat libguestfs libusbmuxd usbmuxd avahi socat

# Linux -> Docker -> Definir Limite de Download Simultâneo
sudo dockerd --max-concurrent-downloads 4

# Linux -> Usuário -> Incluir em Grupos
sudo groupadd docker
sudo usermod -aG docker $USER
sudo gpasswd -a $USER docker
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
sudo usermod -aG libvirtdbus $USER

# Linux -> Serviços -> Habilitar & Iniciar
sudo systemctl enable --now kvm
sudo systemctl enable --now docker
sudo systemctl enable --now usbmuxd
sudo systemctl enable --now virtlogd
sudo systemctl enable --now libvirtd

# Linux -> Carregar Módulos com suas Dependências Manualmente
sudo modprobe kvm

# Linux -> Serviço -> Reiniciar
sudo systemctl restart docker
sudo systemctl restart usbmuxd
sudo systemctl restart virtlogd
sudo systemctl restart libvirtd

# Linux -> Serviços -> Status (Opcional)
sudo systemctl status docker
sudo systemctl status usbmuxd
sudo systemctl status virtlogd
sudo systemctl status libvirtd

# Linux -> Docker -> Gerenciador -> Portainer (Opcional)
docker run -itd --name portainer-sc --mount source=portainer-volume,target=/vol portainer/portainer-ce:2.9.3
Acesso: http://localhost:9000/

------- ------- ------- ------- ------- ------- ------- 
@ Gerar Informações do SMBIOS p/ AppleID
# Modelos de iMac https://everymac.com/systems/apple/imac/index-imac.html
------- ------- ------- ------- ------- ------- ------- 

git clone https://github.com/corpnewt/GenSMBIOS
cd GenSMBIOS
chmod +x GenSMBIOS.py

Opção: 3 (Generate SMBIOS) -> Informe o iMac -> iMac19,3 (Exemplo)
Type:         iMac19,2
Serial:       ...
Board Serial: ...
SmUUID:       ...
Apple ROM:    ...

------- ------- ------- ------- ------- ------- ------- 
@ Compartilhar Pasta p/ OSX
------- ------- ------- ------- ------- ------- ------- 
 
mkdir ~/mnt/osx
sshfs user@localhost:/ -p 50922 ~/mnt/osx
# wait a few seconds, and ~/mnt/osx will have full rootfs mounted over ssh, and in userspace
# automated: sshpass -p <password> sshfs user@localhost:/ -p 50922 ~/mnt/osx

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> iPhone USB (VFIO)
------- ------- ------- ------- ------- ------- ------- 

https://github.com/Silfalion/Iphone_docker_osx_passthrough

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> iPhone USB (USBFLUXD -> Network)
------- ------- ------- ------- ------- ------- ------- 

https://www.youtube.com/watch?v=kTk5fGjK_PM

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> Optimizer
------- ------- ------- ------- ------- ------- ------- 

https://github.com/sickcodes/osx-optimizer

Resultado:
1. Ignore a tela de login da GUI (por sua conta e risco!)
2. Desative a indexação do Spotlight no macOS para acelerar bastante as Instâncias Virtuais.
3. Desative o papel de parede pesado da tela de login
4. Desative as atualizações (por sua conta e risco!)

------- ------- ------- ------- ------- ------- ------- 
@ Criar Container
------- ------- ------- ------- ------- ------- -------

# Catalina -> Pré Instalado
docker run -it \
    --name osx_catalina \
    --device /dev/kvm \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e GENERATE_UNIQUE=true \
    -e "RAM=10" \
    -e "CPU=max" \
    -e "BOOT_ARGS=+vmx" \
    sickcodes/docker-osx:auto

# Ventura -> Instalar
docker run -it \
    --name osx_ventura \
    --device /dev/kvm \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e GENERATE_UNIQUE=true \
    -e "RAM=10" \
    -e "CPU=max" \
    -e "BOOT_ARGS=+vmx" \
    -e MASTER_PLIST_URL='https://raw.githubusercontent.com/sickcodes/osx-serial-generator/master/config-custom.plist' \
    sickcodes/docker-osx:ventura

# Ventura -> Instalar + Informações Dinamica do iMac
docker run -it \
    --name osx_ventura \
    --device /dev/kvm \
    -p 50922:10022 \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -e GENERATE_UNIQUE=true \
    -e "RAM=10" \
    -e "CPU=max" \
    -e "BOOT_ARGS=+vmx" \
    -e MASTER_PLIST_URL='https://raw.githubusercontent.com/sickcodes/osx-serial-generator/master/config-custom.plist' \
    sickcodes/docker-osx:ventura
    
# Ventura -> Instalar + Informações Fixa do iMac
# NOTA 1: Caso já tenha OSX Instalado é só informar o Disco (HDD) do OSX
# NOTA 2: Gere as Informaçõe do iMac com SMBIOS -> GenSMBIOS.py
docker run -it \
    --name osx \
    -v "/home/linux/HDD/Disk_0/mac_hdd_ng.img:/image" \
    --device /dev/kvm \
    -e "DISPLAY=${DISPLAY:-:0.0}" \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -p 50922:10022 \
    -e NOPICKER=true \
    -e GENERATE_SPECIFIC=true \
    -e DEVICE_MODEL="iMac19,3" \
    -e SERIAL="C02YQYYKJWDW" \
    -e BOARD_SERIAL="C02920101GUKGQG1M" \
    -e UUID="0F8630E5-EA25-43E0-85CF-CA0D9F89DD66" \
    -e MAC_ADDRESS="38:0F:4A:A0:BD:62" \
    -e WIDTH=1920 \
    -e HEIGHT=1080 \
    -e "RAM=10" \
    -e "CPU=max" \
    -e "BOOT_ARGS=+vmx" \
    sickcodes/docker-osx:naked

------- ------- ------- ------- ------- ------- ------- 
@ Docker Comandos
------- ------- ------- ------- ------- ------- -------

# Docker -> Inspetor de Objetos
Types: --type container|image|node|network|secret|service|volume|task|plugin

docker inspect --type=volume volume_id
docker inspect --type=container volume_id

------- ------- ------- ------- ------- ------- ------- 

# Image -> Baixar
docker pull alpine (Exemplo)

# Image -> Listar
docker images

# Images -> Inspetor
docker image inspect image_name

------- ------- ------- ------- ------- ------- ------- 

# Container -> Inspetor
docker inspect container_id
docker inspect container_name

# Container -> Listar Ativos
docker container ls

# Container -> Listar Ultimo Container Criado
docker ps -l

# Container -> Exibir Tamanho do Container Ativo
docker ps -s

# Container -> Listar Ativos/Inativos (Todos)
docker ps -a

# Container -> Iniciar
docker start -ai container_id
docker start -ai container_name

# Container -> Iniciar Todos
docker start $(docker ps -a -q)

# Container -> Reiniciar
docker restart container_id
docker restart container_name

# Container -> Reiniciar Todos
docker restart $(docker ps -q)

# Container -> Parar
docker stop container_id
docker stop container_name

# Container -> Parar Todos
docker stop $(docker ps -a -q)

# Container -> Remover
docker rm container_id
docker rm container_name

# Container -> Remover Todos
# Nota: CUIDADO!
docker container prune 

# Container -> Remover Todos (Força Bruta)
# Nota: CUIDADO!
docker container prune

------- ------- ------- ------- ------- ------- ------- 

# Volume -> Listar
docker volume ls

# Volume -> Remover
docker volume rm volume_id

# Volume -> Remover (Força Bruta)
docker volume rm volume_id -f

# Volume -> Remover Todos
# Nota: CUIDADO!
docker volume prune

# Volume -> Remover Todos (Força Bruta)
# Nota: CUIDADO!
docker volume prune -f

------- ------- ------- ------- ------- ------- ------- 
@ OSX -> Instalar e Configurar Flutter/Dart
------- ------- ------- ------- ------- ------- -------

# Instalar Home Brew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Instalar Dart
brew tap dart-lang/dart
brew install dart
brew upgrade dart
brew info dart

# Instalar Flutter
brew install --cask flutter
brew upgrade flutter
brew info flutter

# Instalar VSCode
brew install --cask visual-studio-code
brew upgrade visual-studio-code 

# Instalar VSCodium (Opcional)
brew install --cask vscodium
brew upgrade vscode

# Instalar Android Studio
brew install --cask android-studio
brew uprade android-studio

# Instalar Android SDK (Opcional)
brew install --cask android-sdk
brew uprade android-sdk

# Instalar Android NDK (Opcional)
brew install --cask android-ndk
brew uprade android-ndk

# Instalar XCode
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer/
sudo xcodebuild -license
open -a Simulator

# Definir Path do Flutter
vi ~/.bash_profile
export PATH="´pwd`/flutter/bin:$PATH"

# Flutter Doctor
flutter doctor

------- ------- ------- ------- ------- ------- ------- 
@ KVM (virt-manager)
------- ------- ------- ------- ------- ------- -------

# Exibir Todos os KVM
virsh list --all
