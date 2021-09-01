### Desabilitar instalação e publicação automática


# Parando serviço
	/etc/init.d/cups-browsed stop

# Aplicando Configurações
	sudo sed -i '/BrowseRemoteProtocols dnssd cups/d' /etc/cups/cups-browser.conf
	sudo sed -i '41 iBrowseRemoteProtocols none' /etc/cups/cups-browser.conf

# Iniciando serviço
	/etc/init.d/cups-browsed start
