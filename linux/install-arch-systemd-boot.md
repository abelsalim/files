# Archlinux installation with systemd boot

### Definir teclado

	loadkeis br-abnt2

### Atualizar relógio do sistema

	timedatectl set-ntp true

### Particionar disco (sem home separada)

	fdisk /dev/sda
	g
	n
	
	
	+500M
	t
	L
	1
	
	n
	
	
	
	w

### Formatar partições

	mkfs.fat -F32 /dev/sda1
	mkfs.ext4 /dev/sda2


### Montar artição e criar diretório /boot

	mount /dev/sda2 /mnt
	mkdir /mnt/boot
	mount /dev/sda1 /mnt/boot

### Instalar pacotes base

	pacstrap /mnt base base-devel linux-lts linux-firmware vim sudo

### Gerar fstab

	genfstab -U /mnt >> /mnt/etc/fstab

### Chroot

	arch-chroot /mnt

### Criar e ativar SWAP

	fallocate -l 4GB /swapfile
	chmod 600 /swapfile
	mkswap /swapfile
	swapon /swapfile
	sed -i '$s/$/\n\n/' /etc/fstab
	echo '# SWAP' >> /etc/fstab
	echo '\swapfile none swap defaults 0 0' >> /etc/fstab

### Definir fuso horário e aplicar alteração

	ln -sf /usr/share/zoneinfo/America/Fortaleza /etc/localtime
	hwclock --systohc
	
### Localização

	vim /etc/locale.gen # Defina a localidade
	locale-gen

### Variável de linguagem

	echo LANG=pt_BR.UTF-8 >> /etc/locale.conf                                                                                                                                                      
                                                                                                                                                                                               
### Linguagem de teclado                                                                                                                                                                         

	echo KEYMAP=br-abnt2 >> /etc/vconsole.conf

### Hostname

	echo "zenir-moveis" >> /etc/hostname

### Hosts

	vim /etc/hosts

	{                                                                   
	127.0.0.1     localhost                                                                                                                                       
	::1           localhost                                                                                                                                       
	127.0.1.1     zenir-moveis.localdomain	zenir-moveis    meuhostname                                                                                                                                           
	} 

### Definir senha

	passwd

### Instalação de pacotes

	pacman -S efibootmgr networkmanager network-manager-applet dialog mtools dosfstools linux-lts-headers

### Instalar systemd-boot

	bootctl --path=/boot install

### Aplicar configurações

	vim /boot/loader/loader.conf
	{
	timeout 3
	default arch-*
	}
	
	vim /boot/loader/entries/arch.conf
	{
	title zenir-moveis
	linux /vmlinuz-linux
	initrd /initramfs-linux.img
	options root=/dev/sda2 rw
	}

### Habilitar NetworkManager

	systemctl enable NetworkManager

### Criar usuário e definir senha

	useradd -mG wheel user
	passwd user

### Desmontar unidades e sair

	exit
	umount -a
	reboot

