# Guia de instalação de criptografia de disco completo do Arch Linux
Este guia fornece instruções para uma instalação do Arch Linux com criptografia de disco completo via LVM no LUKS e uma partição de inicialização criptografada (GRUB) para sistemas UEFI.

Após a instalação principal, há mais instruções para se proteger contra ataques do Evil Maid por meio do registro de chave personalizada do UEFI Secure Boot e do kernel autoassinado e do carregador de inicialização. 

## Prefacio
Você encontrará a maioria dessas informações extraídas do [Arch Wiki](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Encrypted_boot_partition_(GRUB)) e outros recursos vinculados a ele. 

## Pré-instalação
### Conecte-se a internet
Conecte sua Ethernet e vá, ou para wireless consulte o [Arch Wiki](https://wiki.archlinux.org/index.php/installation_guide#Connect_to_the_internet)

### Definir teclado
```
loadkeys br-abnt2
```

### Atualizar relógio do sistema
```
timedatectl set-ntp true
```

### Preparando o disco
#### Criar partições do sistema EFI e Linux LUKS
#### Crie uma partição de inicialização do BIOS de 1 MiB no início, caso seja necessário no futuro 

Number | Start (sector) | End (sector) |    Size    | Code |        Name         |
-------|----------------|--------------|------------|------|---------------------|
   1   |   2048         |   4095       | 1024.0 KiB | EF02 | BIOS boot partition |
   2   |   4096         |   1130495    | 550.0 MiB  | EF00 | EFI System          |
   3   |   1130496      |   976773134  | 465.2 GiB  | 8309 | Linux LUKS          |

```gdisk /dev/sda```
```
o
n
[Enter]
0
+1M
ef02
n
[Enter]
[Enter]
+550M
ef00
n
[Enter]
[Enter]
[Enter]
8309
w
```

#### Crie o contêiner criptografado LUKS1 na partição Linux LUKS
```
cryptsetup luksFormat --type luks1 --use-random -S 1 -s 512 -h sha512 -i 5000 /dev/sda3
```

#### Abra o container (descriptografe-o e disponibilize em /dev/mapper/cryptlvm) 
```
cryptsetup open /dev/sda3 cryptlvm
```

### Preparando Volumes Lógicos
#### Crie um volume físico em cima do contêiner LUKS aberto
```
pvcreate /dev/mapper/cryptlvm
```

#### Crie o grupo de volumes e adicione volume físico a ele
```
vgcreate vg /dev/mapper/cryptlvm
```

#### Crie volumes lógicos no grupo de volumes para swap, root e home
```
lvcreate -L 4G vg -n swap
lvcreate -L 20G vg -n root
lvcreate -l 100%FREE vg -n home
```

O tamanho das partições swap e root são uma questão de preferência pessoal. 

#### Formatar sistemas de arquivos em cada volume lógico 
```
mkfs.ext4 /dev/vg/root
mkfs.ext4 /dev/vg/home
mkswap /dev/vg/swap
```

#### Montar sistemas de arquivos
```
mount /dev/vg/root /mnt
mkdir /mnt/home
mount /dev/vg/home /mnt/home
swapon /dev/vg/swap
```

### Preparando a partição EFI
#### Crie o sistema de arquivos FAT32 na partição do sistema EFI
```
mkfs.fat -F32 /dev/sda2
```

#### Crie o ponto de montagem para a partição do sistema EFI em /efi para compatibilidade com grub-install e monte-o
```
mkdir /mnt/efi
mount /dev/sda2 /mnt/efi
```

## Instalação
### Instalar pacotes base
```
pacstrap /mnt base base-devel linux-lts linux-firmware vim sudo mkinitcpio lvm2 wpa_supplicant
```

## Configurar Sistema
### Gerar fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

#### Alterar `relatime` opção para `noatime`
```/mnt/etc/fstab```
```
# /dev/mapper/vg-root
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx / ext4 rw,noatime 0 1

# /dev/mapper/vg-home
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx / ext4 rw,noatime 0 1
```

Reduz as gravações em disco ao ler um arquivo, mas pode causar problemas com programas que dependem do tempo de acesso ao arquivo 

### Chroot
``````
arch-chroot /mnt
``````
 #### Neste ponto, você deve ter as seguintes partições e volumes lógicos:
``````lsblk``````
 
 NAME           | MAJ:MIN | RM  |  SIZE  | RO  | TYPE  | MOUNTPOINT |
---------------|---------|-----|--------|-----|-------|------------|
nvme0n1        |  259:0  |  0  | 465.8G |  0  | disk  |            |
├─sda1         |  259:4  |  0  |     1M |  0  | part  |            |
├─sda2         |  259:5  |  0  |   550M |  0  | part  | /efi       |
├─sda3         |  259:6  |  0  | 465.2G |  0  | part  |            |
..└─cryptlvm   |  254:0  |  0  | 465.2G |  0  | crypt |            |
....├─vg-swap  |  254:1  |  0  |     8G |  0  | lvm   | [SWAP]     |
....├─vg-root  |  254:2  |  0  |    32G |  0  | lvm   | /          |
....└─vg-home  |  254:3  |  0  | 425.2G |  0  | lvm   | /home      |

### Fuso horário 
### Definir fuso horário e aplicar alteração
Substituir `America/Los_Angeles` com seu respectivo fuso horário encontrado em `/usr/share/zoneinfo
```
ln -sf /usr/share/zoneinfo/America/Fortaleza /etc/localtime
```

#### Execute `hwclock` para gerar ```/etc/adjtime```
```
hwclock --systohc
```

### Localização
#### Remover comentário ```en_US.UTF-8 UTF-8``` no ```/etc/locale.gen``` e gerar localidade 
```
vim /etc/locale.gen # Defina a localidade
locale-gen
```

#### Variável de linguagem
```
echo LANG=pt_BR.UTF-8 >> /etc/locale.conf
```

### Linguagem de teclado
```
echo KEYMAP=br-abnt2 >> /etc/vconsole.conf
```

### Configuração de rede 
#### Crie o arquivo de nome de host
```/etc/hostname```
```
echo "myhostname" >> /etc/hostname
```

Este é um nome exclusivo para identificar sua máquina em uma rede. 

#### Adicionar entradas correspondentes aos hosts 
```/etc/hosts```
```                                                                   
127.0.0.1     localhost
::1           localhost
127.0.1.1     myhostname.localdomain	myhostname                                      
``` 

### Initramfs 
#### Adicione o ```keyboard```, ```encrypt```, e ```lvm2``` ganchos para ```/etc/mkinitcpio.conf```
*Nota*: a ordenação é importante.
```
HOOKS=(base udev autodetect keyboard modconf block encrypt lvm2 filesystems fsck)
```

#### Recria imagen initramfs
```
mkinitcpio -p linux
```

### Senha raiz 
#### Definir senha raiz
```
passwd
```

### Carregador de inicialização 
#### Instalar o GRUB 
```
pacman -S grub
```

#### Configure o GRUB para permitir a inicialização de /boot em uma partição criptografada LUKS1 
```/etc/default/grub```
```
GRUB_ENABLE_CRYPTODISK=y
```

#### Defina o parâmetro do kernel para desbloquear o volume físico do LVM na inicialização usando ```encrypt``` gancho
UUID é a partição que contém o contêiner LUKS
```blkid```
```
/dev/nvme0n1p3: UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" TYPE="crypto_LUKS" PARTLABEL="Linux LUKS" PARTUUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```
```/etc/default/grub```
```
GRUB_CMDLINE_LINUX="... cryptdevice=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx:cryptlvm root=/dev/vg/root ..."
```

#### Instale o GRUB no ESP montado para inicialização UEFI
```
pacman -S efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi
```
#### Ativar atualizações de microcódigo
##### grub-mkconfig detectará automaticamente atualizações de microcódigo e configurará adequadamente
```
pacman -S intel-ucode
```
Use intel-ucode para CPUs Intel e amd-ucode para CPUs AMD.

#### Gerar o arquivo de configuração do GRUB
```
grub-mkconfig -o /boot/grub/grub.cfg
```
### (recomendado) Incorporar um arquivo-chave no initramfs

Isso é feito para evitar ter que digitar a senha de criptografia duas vezes (uma vez para GRUB, uma vez para initramfs.) 

#### Crie um arquivo de chave e adicione-o como chave LUKS
```
mkdir /root/secrets && chmod 700 /root/secrets
head -c 64 /dev/urandom > /root/secrets/crypto_keyfile.bin && chmod 600 /root/secrets/crypto_keyfile.bin
cryptsetup -v luksAddKey -i 1 /dev/nvme0n1p3 /root/secrets/crypto_keyfile.bin
```

#### Adicione o arquivo-chave à imagem initramfs
```/etc/mkinitcpio.conf```
```
FILES=(/root/secrets/crypto_keyfile.bin)
```

#### Recrie a imagem initramfs
```
mkinitcpio -p linux
```

#### Defina os parâmetros do kernel para desbloquear a partição LUKS com o arquivo de chave usando encryptgancho
```/etc/default/grub```
```
GRUB_CMDLINE_LINUX="... cryptkey=rootfs:/root/secrets/crypto_keyfile.bin"
```

#### Regenerar o arquivo de configuração do GRUB
```
grub-mkconfig -o /boot/grub/grub.cfg
```

#### Restringir /bootpermissões
```
chmod 700 /boot
```

#### A instalação está agora concluída. Saia do chroot e reinicie.
```
exit
reboot
```
