### Configuração da Impressora de Cupons DARUMA no Linux


# Instalando pacotes
    sudo apt-get install apparmor-utils

# Aplicando configurações
    sudo aa-complain cupsd



# Inserir a impressora no printers.conf

    sudo vim /etc/cups/printers.conf	
    
    }
        <Printer printer>
        Location Linux
        DeviceURI serial:/dev/ttyACM0
        State Idle
        Type 4
        Accepting Yes
        Shared Yes
        JobSheets none none
        QuotaPeriod 0
        PageLimit 0
        KLimit 0
        OpPolicy default
        ErrorPolicy retry-job
        </Printer>
    }

# Reinicie o cups
    sudo systemctl restart cups

