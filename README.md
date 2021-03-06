# kubernetes_necos_ufu

Neste tutorial, considera-se o Ubuntu 16.04 como sistema operacional da máquina hospedeira e Virtualbox 5.2.16 instalado.

I- Criar rede interna no Virtualbox
--------------------------------------------------
**a) Criar a rede interna**

1. File-> Host Network Manager
2. Clicar em Create
3. Marcar DHCP Server
4. Escolher a aba DHCP Server
   - Server Address: 192.168.57.100
   - Server Mask: 255.255.255.0
   - Lower Address Bound: 192.168.57.101
   - Upper Address Bound: 192.168.57.254
 

II - Criar VM do servidor DASH
--------------------------------------------------
**a) Baixar imagem do Ubuntu Server 17.10 64 bits.**

Exemplo:

```
wget http://ubuntu.c3sl.ufpr.br/releases/artful/ubuntu-17.10.1-server-amd64.iso
```

**b) Criar a máquina no Virtualbox**

1. Clicar em New
2. Clicar em Expert Mode
3. Configurar
   - Name: KMaster
   - Type: Linux
   - Version: Ubuntu 64 bits
   - Memory: 4 GB
   - Hard disk: Create a virtual hard disk now
4. Clicar em Create
5. Configurar:
   - File location: KMaster
   - File size: 30 GB
   - File type: VDI
   - Storage on physical HD: Dynamically allocated
6. Clicar em Create 
7. Clicar em Settings (com a VM selecionada)
8. Clicar em Network -> Adapter 2
9. Configurar:
   - Enable Network Adapter (marcar)
   - Attached to: Internal Network
10. Clicar em Network -> Adapter 3
11. Configurar:
    - Enable Network Adapter (marcar)
    - Attached to: Host-only Adapter
    - Name: vboxnet0 (rede criada em I)
12. Clicar em OK


**c) Configurar a ISO de instalação do Ubuntu**

1. Clicar em Settings (com a VM selecionada)
2. Clicar em Storage
3. Selecionar o ícone do CD (empty) em Storage Devices
4. Clicar no ícone do CD em Optical Drive
5. Clicar em "Choose Virtual..."
6. Selecionar a ISO e clicar em Open
7. Marcar Live CD/DVD
8. Clicar em OK

**d) Instalar o Ubuntu**

1.  Selecionar a VM KMaster e clicar em Start
2.  Selecionar English
3.  Selecionar Install Ubuntu Server
4.  Language: English - English
5.  Location: Other -> South America -> Brazil
6.  Locales: United States - en_US.UTF-8
7.  Autodetect Keyboard: No
8.  Keyboard origin: Portuguese (Brazil)
9.  Keyboard layout: Portuguese (Brazil)
10. Primary network: enp0s3 <adaptador1>
11. Hostname: kmaster
12. User: serveruser
13. Username: serveruser
14. Password: 
15. Homedir Encryption: No
16. Clock: America/Sao_Paulo (Yes)
17. Partition Method: Manual
18. Partition Disks: (sda)
    - Create new partition table: Yes
    - Selecionar o espaço livre
    - Create a new partition: "2GB" "primary" "beginning" "use as:swap" Done
    - Selecionar o espaço livre
    - Create a new partition: "30.2GB" "primary" "use as:Ext4..." "mount point:\" Done
    - Finish partitioning(...)
    - Verificar e clicar em Yes
19. Proxy: <nao configurar>
20. Tasksel: No automatic updates
21. Software: OpenSSH server; 
22. GRUB Install: Yes
23. Finalizar a instalação clicando em Continue
		
Remover a ISO após instalação.

III - Configurar o 
-------------------------------------------------
Obs.: Recomendável salvar uma cópia de segurança antes, clonando a máquina (desligada).

**a) Atualizar os pacotes e instalar o unzip**

```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
sudo apt-get install unzip
```

**b) Alterar o /etc/sudoers para não solicitar senha**

```
sudo visudo
```

Incluir a seguinte linha no final do arquivo:

```
serveruser ALL=(ALL:ALL) NOPASSWD: ALL
```

Salvar como /etc/sudoers.

**c) Alterar o /etc/inputrc para pesquisar o histórico com PgUp e PgDn**

```
sudo vim /etc/inputrc
```

Descomentar as linhas (41 e 42):

```
"\e[5~": history-search-backward
"\e[6~": history-search-forward
```

Salvar o arquivo (:x) e logar novamente.

**d) Configurar as interfaces no /etc/network/interfaces.** 

Obs.: Para saber o nome das interfaces use antes o comando ```ip link show```.

```
sudo vim /etc/network/interfaces
```

O arquivo deve incluir a configuração das três interfaces:

```
# NAT
# The primary network interface
auto enp0s3
iface enp0s3 inet dhcp

# Internal Network
auto enp0s8
iface enp0s8 inet static
address 172.16.0.10
netmask 255.255.255.0
network 172.16.0.0
broadcast 172.16.0.255

# Host-only
auto enp0s9
iface enp0s9 inet static
address 192.168.56.10
netmask 255.255.255.0
network 192.168.56.0
broadcast 192.168.56.255
```

Reiniciar o servidor:

```
sudo shutdown -r now
```

O restante do documento pode ser executado via ssh da máquina hospederia (192.168.56.1) para o servidor:

```
ssh serveruser@192.168.56.10
```
