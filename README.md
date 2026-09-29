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

#IP fija

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

Ahora pasaremos a ver lo configurado en la GUI de los Fortigates, la mayoría de estas configuraciones serán hechas en ambos, solo que modificadas para su red en especifico y su lado del túnel VPN. 

Como primer paso, para facilitar mis configuraciones hice en cada Fortigate unos objetos representando la red de Usuarios y Servidores:

Forti1

<img width="1426" height="85" alt="image" src="https://github.com/user-attachments/assets/7420bcfa-4fde-4e4b-9a9e-0a46c893a49e" />

Forti2

<img width="1382" height="82" alt="image" src="https://github.com/user-attachments/assets/0b50e435-3afe-40e3-8ee4-8b4c2967fc3c" />

Luego realicé túneles VPN utilizando como red las interfaces públicas de cada Fortigate, configuré la seguridad, y en la fase 2 especifiqué las redes locales y remotas dependiendo del fortigate:

Forti1 (Usuarios)

<img width="827" height="747" alt="image" src="https://github.com/user-attachments/assets/28c47c1c-a7f3-4ffd-beae-2f3ef66d8922" />

Forti2 (Servidor)

<img width="871" height="732" alt="image" src="https://github.com/user-attachments/assets/26d76200-7862-49e9-8f47-0edc1f135b2c" />

Configuré rutas estáticas para que los dispositivos sepan como dirigirse por los túneles VPN creados:

Forti1 (Usuarios)

<img width="1281" height="135" alt="image" src="https://github.com/user-attachments/assets/7423579b-9b7f-4e5a-a5c3-d7b3cc8789dd" />

Forti2 (Servidor)

<img width="1277" height="82" alt="image" src="https://github.com/user-attachments/assets/00f53c64-3cbb-440b-8ace-81e5bc4c4227" />

También he configurado algunas políticas para permitir el acceso de lado a lado a través de los túneles, representado por el puerto que lleva a la red interna y la red remota.

Forti1 (Usuarios), las primeras dos políticas son requisitos de red, para la conexión VPN las otras dos son las que interesan más: 

<img width="1590" height="376" alt="image" src="https://github.com/user-attachments/assets/810c2dbc-3796-4f7e-89e9-006a118a3a9e" />

Forti2 (Servidor)

<img width="1572" height="246" alt="image" src="https://github.com/user-attachments/assets/3fc67f8d-0926-4c68-88fc-19225d83cdce" />

Como se pueden ver, muchas de estas configuraciones se mantienen prácticamente iguales, solos se cambian algunos datos dependiendo del lado de la conexión al que pertenezcan, con esto fui capaz completar los objetivos de la práctica y lo muestro en el vídeo al principio del repositorio.

### Running-Config ###

## ISP ##

ISP#show running-config

Building configuration...

Current configuration : 1852 bytes

! Last configuration change at 04:54:02 UTC Tue Sep 29 2026

version 15.2

service timestamps debug datetime msec

service timestamps log datetime msec

service password-encryption

hostname ISP

boot-start-marker

boot-end-marker

enable secret 5 $1$wCxD$im3GOK8RfYT9LJHWglwKU.

no aaa new-model

no ip icmp rate-limit unreachable

no ip domain lookup

ip domain name laboratorio.local

ip cef

no ipv6 cef

multilink bundle-name authenticated

username admin secret 5 $1$qBjj$/UOWQWOdX9.xc.DRQfwo..

ip tcp synwait-time 5

interface FastEthernet0/0

 no ip address
 
 shutdown
 
 duplex full

interface GigabitEthernet1/0

 description ISP-Forti1
 
 ip address 200.78.7.1 255.255.255.252
 
 negotiation auto

interface GigabitEthernet2/0

 description ISP a Forti2
 
 ip address 200.78.7.5 255.255.255.252
 
 negotiation auto

interface GigabitEthernet3/0

 no ip address
 
 shutdown
 
 negotiation auto

interface Serial4/0

 no ip address
 
 shutdown
 
 serial restart-delay 0

interface Serial4/1

 no ip address
 
 shutdown
 
 serial restart-delay 0

interface Serial4/2

 no ip address
 
 shutdown
 
 serial restart-delay 0

interface Serial4/3

 no ip address
 
 shutdown
 
 serial restart-delay 0

interface FastEthernet5/0

 no ip address
 
 shutdown
 
 duplex full

interface FastEthernet6/0

 no ip address
 shutdown
 
 duplex full

ip forward-protocol nd

no ip http server
no ip http secure-server
ip route 10.78.7.0 255.255.255.128 200.78.7.2
ip route 10.78.7.128 255.255.255.240 200.78.7.6

control-plane

banner motd ^C Acceso restringido: solo personal autorizado por Airam Brazoban ^C

line con 0

 exec-timeout 0 0
 
 privilege level 15
 
 logging synchronous
 
 stopbits 1
 
line aux 0

 exec-timeout 0 0
 
 privilege level 15
 
 logging synchronous
 
 stopbits 1
 
line vty 0 4

 login local
 
 transport input ssh

end


