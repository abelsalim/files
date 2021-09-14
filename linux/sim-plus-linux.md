# Instalação do SIM-PLUS no linux Ubuntu

### Adicionar arquitetura 32 bits
	dpkg --add-architecture i386

### Atualizar repositórios
	sudo apt-get update

### Instalar libs
	sudo apt-get install libgtk2.0-0:i386 libxxf86vm1:i386 libsm6:i386 lib32stdc++

### Instalar winetricks e wine32
	sudo apt-get install winetricks wine32

### Criar e acessar diretório da aplicação
	mkdir -l intelbras/wine
	cd intelbras

### Download do sim-plus
	wget https://www.seseguranca.com.br/imagens/downloads/http___seseguranca.com.br_wp-content_themes_seseguranca_anexo_simplus_setup.zip

### Descompactar
	unzip http___seseguranca.com.br_wp-conte    nt_themes_seseguranca_anexo_simplus_setup.zip

### Remover arquivo compactado
	rm -f http___seseguranca.com.br_wp-conte    nt_themes_seseguranca_anexo_simplus_setup.zip

### Exportar
	export WINEPREFIX=$HOME/intelbras/wine

### Configurações no winecfg
	winecfg {acesse a aba gráficos e altere a resolução para 1024 x 768}

### Instalar bibliotecas linux
	winetricks vcrun2005
	winetricks directx9
	winetricks directplay

### Instale o programa 
	wine sim-plus.exe
