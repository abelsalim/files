### Desabilitar instalação e publicação automática


# Parar serviço
	/etc/init.d/cups-browsed stop

# Alterar 
	vim /etc/cups/cups-browsed.conf

	sudo sed -i '/BrowseRemoteProtocols dnssd cups/d' /etc/cups/cups-browser.conf
	sudo sed -i '41 iBrowseRemoteProtocols none' /etc/cups/cups-browser.conf

# Subir serviço
		/etc/init.d/cups-browsed start
