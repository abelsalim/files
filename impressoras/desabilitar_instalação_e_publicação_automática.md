### Desabilitar instalação e publicação automática


# Parar serviço
		/etc/init.d/cups-browsed stop

# Alterar 
		vim /etc/cups/cups-browsed.conf

PROCURAR -> BrowseRemoteProtocols dnssd cups
SUBSTITUIR -> BrowseRemoteProtocols none

# Subir serviço
		/etc/init.d/cups-browsed start
