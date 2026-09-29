# SegRedes_Practica2_Parte1_SitetoSite-Fortigates
Repositorio para la conexión de fortigates por VPN Site-to-Site para la segunda práctica de Seguridad de Redes.

Enlace al vídeo de Youtube en el que explico lo realizado: https://youtu.be/qo_AoZwWLyk

Objetivos de la práctica:

Comunicar el Usuario con el Servidor a través del enlace VPN.

Comprobar que la comunicación solo fluye si el enlace VPN esta activo.

2 Fortigate (Toda configuración y demostración debe ser por GUI)

Configuraciones de Red.

NAT

VPN Site-To-Site entre los Fortigates.

ISP

IP Publicas.

1 Servidor Web (/28)

Web Server (HTTPS).

1 Usuarios (/25)

Vlan 10.

DHCP.

Traceroute hacia el servidor.

Ya con el vídeo y los requisitos de la práctica declarados, podemos empezar a ver los resultados.

Mi diagrama basado en la práctica:

<img width="610" height="545" alt="image" src="https://github.com/user-attachments/assets/17a1d58e-e26d-4f22-9501-2ba80adffe5f" />

Direccionamiento basado en mi matricula:

<img width="707" height="255" alt="image" src="https://github.com/user-attachments/assets/5c02e998-cb74-468d-95fe-6df2bdcf5a40" />

#### Scripts utilizados (principalmente para direccionamiento y conexión básica, ya que la mayoría se configura en la GUI de los FortiGates) ####

#### Forti1 ####

# WAN hacia ISP
config system interface

    edit port1
    
	set mode static
  
        set ip 200.78.7.2 255.255.255.252
        
        set allowaccess ping
        
    next
    
end

# LAN Usuarios (VLAN 10)

config system interface

    edit port2
    
        set ip 10.78.7.1 255.255.255.128
        
        set allowaccess ping https ssh
        
    next
    
end

# DHCP para usuarios

config system dhcp server

    edit 1
    
        set interface "port2"
        
        set lease-time 86400
        
        set default-gateway 10.78.7.1
        
        set netmask 255.255.255.128
        
        config ip-range
        
            edit 1
            
                set start-ip 10.78.7.10
                
                set end-ip 10.78.7.100
                
            next
            
        end
        
    next
    
end



# Ruta por defecto hacia ISP

config router static

    edit 1
    
        set gateway 200.78.7.1
        
        set device port1
        
    next
    
end


#### Forti2 ####

# WAN hacia ISP

config system interface

    edit port1
    
        set ip 200.78.7.6 255.255.255.252
        
        set allowaccess ping
        
    next
    
end

# DMZ Servidor Web

config system interface

    edit port2
    
        set ip 10.78.7.129 255.255.255.240
        
        set allowaccess ping https ssh
        
    next
    
end

# Ruta por defecto hacia ISP

config router static

    edit 1
    
        set gateway 200.78.7.5
        
        set device port1
        
    next
    
end


#### Servidor Web ####

# IP fija

ip addr add 10.78.7.130/28 dev ens3

ip route add default via 10.78.7.129


#### ISP ####
interface GigabitEthernet1/0

 ip address 200.78.7.1 255.255.255.252
 
 no shutdown
 
 description ISP a Forti1
 
exit

interface GigabitEthernet2/0

 ip address 200.78.7.5 255.255.255.252
 
 no shutdown

 description ISP a Forti2
 
exit


! Red de usuarios detrás de Forti1

ip route 10.78.7.0 255.255.255.128 200.78.7.2

! Red de servidores detrás de Forti2

ip route 10.78.7.128 255.255.255.240 200.78.7.6




