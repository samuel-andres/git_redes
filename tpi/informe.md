
- [Consigna](#consigna)
- [WAN](#wan)
  - [Configuración y topología lógica](#configuración-y-topología-lógica)
  - [Topología física](#topología-física)
- [LANs](#lans)
  - [Topología lógica](#topología-lógica)
  - [VLANs](#vlans)


# Consigna

- Interconectar 3 Redes LAN de una compañía con sede en 3 ciudades diferentes:
  - Villa María
  - Bariloche
  - Mendoza
- La conexión entre las Redes LAN debe ser mediante WAN Frame Relay
- Cada red LAN debe tener:
  - 3 VLANS:
    - VLAN2: 2 PCS
    - VLAN3: 1 PC y una impresora
    - VLAN4: 1 Notebook y 1 Tablet (Conexión Wireless mediante un Router WiFi WRT-300N)
    - Asignación de IPS mediante DHCP

# WAN

## Configuración y topología lógica

Para establecer la conexión entre las 3 ciudades, debíamos utilizar WAN Frame-Relay. Para esto establecimos 3 circuitos virtuales formando una topología full-mesh entre los 3 routers. 

Cada uno de los routers fue conectado mediante un cable serial a la nube frame relay del ISP. En la interfaz serial de cada uno de estos routers se configuraron dos interfaces virtuales con encapsulamiento frame-relay. De este modo la conexión lógica entre los 3 routers es directa (PaP). 

| **Router** | **Interfaz Física** | **Interfaz Virtual** | **Dirección de capa 2 (DLCI)** | **Dirección de capa 3 (IPv4)** | **Dirección de red** |
|------------|---------------------|----------------------|--------------------------------|--------------------------------|----------------------|
| **RVM**    | serial 0/1/0        | s 0/1/0.102          | 102                            | 10.0.1.1/24                    | 10.0.1.0/24          |
| **RVM**    | serial 0/1/0        | s 0/1/0.103          | 103                            | 10.0.3.1/24                    | 10.0.3.0/24          |
| **RBR**    | serial 0/1/0        | s 0/1/0.201          | 201                            | 10.0.1.2/24                    | 10.0.1.0/24          |
| **RBR**    | serial 0/1/0        | s 0/1/0.203          | 203                            | 10.0.2.2/24                    | 10.0.2.0/24          |
| **RMZ**    | serial 0/1/0        | s 0/1/0.301          | 301                            | 10.0.3.2/24                    | 10.0.3.0/24          |
| **RMZ**    | serial 0/1/0        | s 0/1/0.302          | 302                            | 10.0.2.1/24                    | 10.0.2.0/24          |

> ***Topología lógica*** <br/>
> ![fr_logtop](fr_logtop.png)

Para que esto funcione tuvimos que también configurar los circuitos virtuales (PVCs) en la nube Frame Relay:
> ***Circuitos virtuales*** <br/>
> ![fr_pvcs](./fr_pvcs.png)

La configuración de las interfaces seriales involucradas en cada uno de los routers fueron las siguientes:
> RVM:
> ``` code
> interface Serial0/1/0
>  no ip address
>  encapsulation frame-relay
> !
> interface Serial0/1/0.102 point-to-point
>  ip address 10.0.1.1 255.255.255.0
>  frame-relay interface-dlci 102
>  clock rate 2000000
> !
> interface Serial0/1/0.103 point-to-point
>  ip address 10.0.3.1 255.255.255.0
>  frame-relay interface-dlci 103
>  clock rate 2000000
> !
> ```
> RBR:
> ``` code
> interface Serial0/1/0
>  no ip address
> encapsulation frame-relay
> !
> interface Serial0/1/0.201 point-to-point
>  ip address 10.0.1.2 255.255.255.0 
>  frame-relay interface-dlci 201
>  clock rate 2000000
> !
> interface Serial0/1/0.203 point-to-point
>  ip address 10.0.2.2 255.255.255.0
>  frame-relay interface-dlci 203
>  clock rate 2000000
> !
> ```
> RMZ:
> ``` code	
> interface Serial0/1/0
>  no ip address
>  encapsulation frame-relay
> !
> interface Serial0/1/0.301 point-to-point
>  ip address 10.0.3.2 255.255.255.0
>  frame-relay interface-dlci 301
>  clock rate 2000000
> !
> interface Serial0/1/0.302 point-to-point
>  ip address 10.0.2.1 255.255.255.0
>  frame-relay interface-dlci 302
>  clock rate 2000000
> !
> ```

## Topología física

En el mapa de Argentina la topología física se ve de la siguiente manera, considerando que la nube Frame Relay es una representación lógica:

>![WAN_full_phy_top.png](./WAN_full_phy_top.png)

Luego, dentro de cada oficina, en cada edificio de cada ciudad, se encuentra un habitación de cableado con un RACK, donde, entre otras cosas, está el Router, con la conexión serial:

>***Villa María*** <br/>
>![RAC_VM](./WAN_wc_ph_top_VM.png) <br/>
>***Bariloche*** <br/>
>![RAC_BR](./WAN_wc_ph_top_BR.png) <br/>
>***Mendoza*** <br/>
>![RAC_MZ](./WAN_wc_ph_top_MZ.png) <br/>

# LANs

## Topología lógica

La topología lógica de la LAN de Villa María se ve de la siguiente manera: <br/> 
> ![vm_logtop](./vm_logtop.png)

## VLANs

Lo primero fue crear las VLANs en cada switch:
> VM
> ``` code
> SW1-VM(config)#vlan 2
> SW1-VM(config-vlan)#name VLAN2
> SW1-VM(config)#vlan 3
> SW1-VM(config-vlan)#name VLAN3
> SW1-VM(config)#vlan 4
> SW1-VM(config-vlan)#name VLAN4
> ```
> BR
> ``` code
> SW1-BR(config)#vlan 2
> SW1-BR(config-vlan)#name VLAN2
> SW1-BR(config)#vlan 3
> SW1-BR(config-vlan)#name VLAN3
> SW1-BR(config)#vlan 4
> SW1-BR(config-vlan)#name VLAN4
> ```
> MZ
> ``` code
> SW1-MZ(config)#vlan 2
> SW1-MZ(config-vlan)#name VLAN2
> SW1-MZ(config)#vlan 3
> SW1-MZ(config-vlan)#name VLAN3
> SW1-MZ(config)#vlan 4
> SW1-MZ(config-vlan)#name VLAN4
> ```

Una vez creadas las VLANs lo siguiente fue asignar cada puerto (configurándolo en modo acceso) a la VLAN correspondiente:
> ***VM***
> ``` code	
> interface FastEthernet0/1
>  switchport access vlan 2
>  switchport mode access
> !
> interface FastEthernet0/2
>  switchport access vlan 2
>  switchport mode access
> !
> interface FastEthernet0/3
>  switchport access vlan 3
>  switchport mode access
> !
> interface FastEthernet0/4
>  switchport access vlan 3
>  switchport mode access
> !
> interface FastEthernet0/5
>  switchport access vlan 4
>  switchport mode access
> ```
> Además, por seguridad, desactivé todas las interfaces del Switch que no se utilizan. <br/>
> ***Resultando en:*** *ignorar DHCP incorporado más adelante <br/>
> ![sh_int_st_VM](./sh_int_st_VM.png) <br/>
> ***En Bariloche y Mendoza las configuraciones fueron exactamente las mismas, por lo que se muestra la VLAN DB en cada uno de estos:*** <br/>
> ***Bariloche*** <br/>
> ![vl_db_br](./vl_db_br.png) <br/>
> ***Mendoza*** <br/>
> ![vl_db_mz](./vl_db_mz.png) <br/>

Con estas configuraciones, en cada LAN hay conexión en capa 2, entre dispositivos que se encuentran en la misma VLAN, conectados al mismo Switch.

Ahora, lo que se quiere lograr es conectividad total, para esto empezamos por definir una subred para cada VLAN, el esquema de direcciones que establecimos en clase fue:
> ***10.LOC.VLAN.HOST*** <br/>
> Siendo: <br/>
> LOC 1 = Villa María <br/>
> LOC 2 = Bariloche <br/>
> LOC 3 = Mendoza <br/>
> VLAN = 2,3,4 <br/>
> HOST = GW .1 <br/>
> <br/>
> Es decir, direcciones clase A con un largo de prefijo de 24 bits.

Lo primero que se necesita para que esto funcione correctamente es definir un enlace troncal entre el Switch y el Router, para esto utilizamos la técnica denominada "Router-on-a-Stick" que consiste en configurar una interfaz virtual en el Router para cada una de las vlans que se encuentran en la red, y configurar cada una de estas interfaces virtuales para que envíen tramas con el TAG correspondiente por el enlace troncal.
> ***Configuración del enlace troncal en el Switch:***
> ``` code
> interface GigabitEthernet0/1
>  description #AL ROUTER#
>  switchport trunk native vlan 1001
>  switchport trunk allowed vlan 2-4
>  switchport mode trunk
> ```
> Resultando en: <br/>
> ![sh_int_tr_VM](./sh_int_tr_VM.png) <br/>
> Como se puede observar, definí como VLAN nativa una VLAN que no se utiliza, para prevenir el tráfico sin tags por el enlace troncal, además, solo permití el trafico de las VLANs existentes, lo que significa que si se crea una nueva VLAN no se permitirá automáticamente en el enlace troncal, por lo que el administrador de red tiene un control más estricto sobre la misma.
> Esta misma configuración fue replicada tanto en el switch de Bariloche como en el switch Mendoza. <br/>
> ***Configuración de interfaces virtuales en el Router:***
> Para permitir la interconexión entre dispositivos que se encuentran en distintas VLANs, los mismos deben estar separados tanto en capa 2 como en capa 3, y la interconexión se da mediante un Router, con varias interfaces virtuales que funcionan como puerta de enlace predeterminada para cada una de estas redes por las que encamina el tráfico con el TAG de la VLAN correspondente. <br/>
> La configuración necesaria en cada interfaz virtual es: levantar la interfaz física, definir el protocolo de encapsulamiento (802.1q) e indicar el TAG correspondiente, y por último definir la dirección IP de la misma (y máscara), que se acordó sería el primero de cada red. <br/>
> Resultando en la siguiente configuración: <br/>
> ``` code
> interface GigabitEthernet0/0/0
>  no ip address
>  duplex auto
>  speed auto
> !
> interface GigabitEthernet0/0/0.2
>  encapsulation dot1Q 2
>  ip address 10.1.2.1 255.255.255.0
> !
> interface GigabitEthernet0/0/0.3
>  encapsulation dot1Q 3
>  ip address 10.1.3.1 255.255.255.0
> !
> interface GigabitEthernet0/0/0.4
>  encapsulation dot1Q 4
>  ip address 10.1.4.1 255.255.255.0
> ```
> Esta configuración se replicó tanto en Bariloche como en Mendoza.

