# Protocolos dinámicos de enrutamiento
- IGP (Interior Gateway Protocol): protocolos utilizados para compartir rutas en un determinado AS (Sistema Autónomo AKA Organización).
  - Algoritmo Distance Vector:
    - RIP (Routing Information Protocol)
    - IGRP (Interior Gateway Routing Protocol)
    - EIGRP (Enhanced Interior Gateway Routing Protocol)
  - Algoritmo Link State:
    - OSPF (Open Shortest Path First)
    - IS-IS (Intermediate System to Intermediate System)
- EGP (Exterior Gateway Protocol): protocolos utilizados para compartir rutas entre AS’s.
  - Algoritmo Path Vector:
    - BGP (Border Gateway Protocol)
## IGP: Distance Vector
- Distance = metric
  - RIP: hop count
  - IGRP: bandwith & delay (ancho de banda y latencia)
  - EIGRP: bandwith & delay (ancho de banda y latencia)
- Vector = dirección
  - RIP: next hop
  - IGRP & EIGRP: next hop
 
### Funcionamiento del algoritmo:
1. Popular tabla de enrutamiento con las redes asociadas a las interfaces directamente conectadas.
2.	Localizar vecinos 
3.	Informar mediante Route Updates la tabla de enrutamiento.
4.	Recibir Route Updates
5.	Comparar
6.	Actualizar tabla
### Características diferenciales del algoritmo:
- Los routers configurados con protocolos que utilizan algoritmos de vector distancia **SOLO** intercambian información con sus vecinos (conectados directamente).
- Es decir, si la topología es A—B—C, A solo sabrá de C a través de lo que B le informó, no intercambiará información directamente con C, y viceversa. Esto es denominado enrutamiento por rumor por el hecho de que A debe confiar en que B le está compartiendo la información adecuada sobre C.
- Periódicamente se envían advertisements a TODOS los vecinos. 
- Los advertisements (excepto rigurosas excepciones como full horizon, o poisong route) incluyen la TABLA DE ENRUTAMIENTO COMPLETA. Por lo que el router que reciba esta información, debe procesarla para encontrar los nuevos cambios y poder actualizar su tabla.
- Los routers DELEGAN la responsabilidad de esparcir la información de sus tablas de enrutamiento en sus vecinos.
### Timers:
- Invalidation Timer: cuando se cae un enlace, un router espera un tiempo determinado por el protocolo denominado invalidation timer, hasta considerar que el destino que comunicaba ese enlace es unreachable (inalcanzable). Para setear la ruta como unreachable lo que hace es setear una distancia “infinita”, definida en los protocolos según la métrica utilizada. Por ej. En RIP son 16 saltos. Este hecho de setear las rutas “malas” con una métrica “infinita” es denominado Route Poisoning.
- Holdown Timer: explicado en Split Horizon with Poison Reverse
### Problemas comunes
- Problema 1:
  - Counting to Infinity:
    - Cuando se cae un enlace, el Router conectado directamente al mismo va a informar de esto en la próxima ronda de actualización, al mismo tiempo va a recibir una actualización de un vecino al que le dijo que lo podía alcanzar en la ronda anterior y va a actualizar la ruta pensando que ahora su vecino puede alcanzar ese camino siguiendo otro path. La métrica de la ruta va a ser n+1.
    - A su vez, cuando su vecino reciba la información de que la ruta es mala, en la próxima ronda se lo va a volver a informar al router original generando un loop de intercambio de ruta mala ruta buena hasta que el valor de n+1 = un valor infinito (en RIP 16).
- Solución 1:
  - Split Horizon Simple:
    - Define una regla fundamental “no informar rutas a vecinos que nos enseñaron esas rutas”. Esto es, en los advertisements se dejará de enviar la TABLA COMPLETA, ya que se quitarán de esta las rutas que nos ha enseñado ese vecino.
    - Es decir, si D-A-B, A informará a D su tabla completa EXCEPTO las rutas que aprendió de D, e informará a B su tabla completa EXCEPTO las rutas que aprendió de B.
- Solución 2: 
  - Split Horizon con Poison Reverse:
    - Cuando un router detecta la caída de un enlace, esto es, cuando el invalidation timer llega a 0, este no esperará el tiempo de ronda para enviar la actualización de ruta (su tabla), directamente enviará un tipo especial de Route Update/advertisement denominado Triggered Partial Update, que SOLO contendrá la información de la ruta ahora inalcanzable. Sus vecinos que reciban esta información esperarán a la siguiente ronda para informarle a sus vecinos.
    - Cuando su vecino recibe una Triggered Partial Update, SUSPENDE la función de Split Horizon para poder enviarle un ACK al mismo, este ACK no es un ACK en si mismo sino un reenvío del propio TPU.
- Problema 2: 
  - Redundancia:
    - Cuenta al infinito en routers adyacentes tras el envío de un TPU.
- Solución:
  - Implementación de un Holdown timer, que define un tiempo durante el cual los vecinos que recibieron un TPU no escucharán actualizaciones sobre la ruta envenenada, asegurando así una convergencia segura.
## IGP: Link State
Métrica: costo

Costo calculado en base a bando de ancha, inversamente proporcional.
- Ventajas sobre los algoritmos Distance Vector:
  - Mejores tiempos de convergencia.
  - Mejor evasión de loops.
- Desventajas:
  - Complejidad

### Características diferenciales del algoritmo:
- Introducen el concepto de “adyacencias”, que refiere a vecinos que no están DIRECTAMENTE conectados a los mismos. Esto es los algoritmos vector distancia no era posible.
- Los advertisements pasan de llamarse Route Updates a llamarse LSA’s (Link State Advertisements).
- Un LSA está compuesto por:
  - Información del Router
  - Enlaces conectados al mismo
  - Estado de esos enlaces
- Los routers reciben y envían información (en forma de LSA’s) a todos y desde todos los routers de la red (con los que formen adyacencia, es decir que corran el mismo protocolo y que cumplan alguna otra serie de aspectos definidos en el protocolo utilizado). 
- La información de los LSA’s es almacenada en una base de datos que tiene cada router denominada Link State Database (LSDB).
- La información almacenada en la LSDB de cada router es LA MISMA. Lo que les permite tener un mapa completo de la topología de la red LS.
- Cada router aplica un algoritmo a la LSDB para crear las entradas correspondientes en la tabla de enrutamiento que corresponden a la mejor ruta a cada destino. (Camino mas corto primero SPF)
- Para descubrir vecinos se envían mensajes “Hello” en broadcast, y todos los routers de la red que corran el mismo protocolo pueden responder, formando así las adyacencias.
- Existe un intérvalo de 30 minutos donde los routers vuelven a enviar LSA’s para chequear que la información sobre la red está actualizada.
- Sin embargo esto no es más que una medida de protección ya que cuando ocurre un cambio en la red, como la caída de un enlace, esto es informado inmediatamente
- Cada vez que hay un cambio en la red el router que se entera envía un LSA informándolo a todos los routers de la red, estos actualizan su LSDB con esta nueva información, aplican el algoritmo SPF, en base a los resultados deciden si actualizar o no su tabla de enrutamiento.

### Algoritmo aplicado por cada router a su LSDB
- Algoritmo SPF
  - Similar a rapid per vlan spanning tree. Se calculan los costos en base a los datos que se encuentran en la LSDB, y en base a estos resultados se elige el camino más corto, es decir el que tiene menos costo, es decir el que resulta en mayor ancho de banda y se agrega a la tabla de enrutamiento.
## Routers con múltiples protocolos de enrutamiento dinámico
Los routers elijen que con que rutas popular sus tablas basados en:
1.	AD (Distancia Administrativa)
2.	Métrica

La AD es un valor de facto que determina que protocolo utilizar para determinar con que ruta popular una tabla de enrutamiento cuando en un router se están corriendo más de 1 protocolos, esto es, si hay dos rutas al mismo destino, el router tomará como primer criterio la distancia administrativa y priorizará la que tenga un valor de AD más bajo, esto es, utilizará el protocolo con la AD mas baja definida, a partir de esto, para determinar cual de las rutas al mismo destino colocar en la tabla de enrutamiento, se fijara en todas las que ese protocolo identificó y seleccionará la que tiene un valor de métrica más bajo.
 
![AD](./ad.png)


# Routing Information Protocol
Es un protocolo dinámico de enrutamiento que entra en la categoría de Interior Gateway Protocol (IGP; Protocolo de puerta de enlace interna). Utiliza un algoritmo del tipo Distance Vector)
## Características diferenciales del protocolo
- Existen dos versiones:
  - RIPv1: 
    - No soporta CIDR
    - No soporta VLSM
    - Usa la dirección de broadcast 255.255.255.255
  - RIPv2:
    - Soporta CIDR
    - Soporta VLSM
    - Usa la dirección de multicast 224.0.0.9
    - La ventaja de usar multicast en lugar de broadcast es que los dispositivos no tienen que desencapsular el datagrama para descartar el PDU. El descarte se da una capa más abajo.
- Es simple.
- No es útil en redes muy grandes ya que tiene un límite de 15 saltos, 16 o más es considerado unreachable. 
- Administrative Distance: 120
- Métrica – distancia: hops.
- El comando no auto summary desactiva la sumarización automática. Esto es, si no activamos esta opción el router va a advertir a sus vecinos la dirección classfull, no cada una de las subredes que caigan en ese rango.

Los pasos que lleva a cabo el router cuando seteamos una net en el comando rip son los siguientes:
1. router rip
2. ver 2
3. no aut
4. network A.B.C.D.
5. el router asume la dirección classfull
6. chequea en su tabla de enrutamiento todas las direcciones que caigan en ese rango
7. activa rip en esas interfaces y por las mismas envía las direcciones con su prefijo apropiado
8. Si en una interfaz no hay otro router, pero aun así queremos que esa dirección sea informada por las demás interfaces pero no le queremos enviar advertisements al pedo tenemos que usar el comando:
passive-interface *int* dentro de la conf del rip.

EXTRA: para enviar advertisements sobre el gateway de last resort se usa default-information originate.

Por defecto el max path de router rip v2 es 4, esto es, guarda en la tabla hasta 4 rutas a la misma dirección.

Esto se puede cambiar con maximum-paths *num* pero no tiene mucho sentido 

La AD (distancia administrativa) también se puede cambiar, por defecto es 120, pero con:
distance *distancia* se puede cambiar.

Lo más importante de entender en la práctica al utilizar RIP es que la red classfull que indicamos en el comando network en la configuración del router RIP, no es la red que queremos advertir a los vecinos, es una dirección classfull, que indica que todas las redes conectadas al router, que caigan en ese rango, es decir, que aplicando el and lógico con la máscara de subred por defecto de la clase de la red indicada den como resultado la misma dirección, van a ser advertidas a los vecinos.

## Autenticación:
La autenticación en RIP se utiliza para prevenir que se introduzcan routers no permitidos en la red y afecten el funcionamiento de la misma influenciando las rutas advertidas por el protocolo de enrutamiento RIP.
En global-config debemos definir una “key chain” que es una cadena de claves que podemos utilizar:
1.	#key chain *nombre_de_cadena* 
2.	#key *numero_de_clave*
3.	#key-string *clave*
Luego, debemos entrar en cada una de las interfaces que queremos utilizar rip y definir el protocolo de autenticación:
4.	#ip rip authentication mode md5
Y por último, definimos la key-chain que queremos utilizar (en las interfaces):
5.	#ip rip authentication key-chain *nombre_de_cadena*

## RIP Timers
 	Update: 30sec
 	Invalid: 180sec
 	Hold Down: 180sec

# Enhanced Interior Gateway Protocol
Es un protocolo dinámico de enrutamiento que entra en la categoría de Interior Gateway Protocol (IGP; Protocolo de puerta de enlace interna). Utiliza un algoritmo del tipo Distance Vector)
## Características diferenciales del protocolo
- Es un protocolo propietario de cisco, aunque Cisco publicó el código y ahora es open source, de todas formas pocos vendors adoptaron el protocolo por lo que se utiliza mayoritariamente en dispositivos cisco.
- Es considerado un protocolo de enrutamiento dinámico vector distancia avanzado/híbrido.
- Soporta VLSM y CIDR.
- No requiere mucho CPU ni memoria.
- El tiempo de convergencia es menor que en RIP.
- No tiene la limitación de 15 saltos que tiene RIP.
- Utiliza la dirección multicas 224.0.0.10
- Es el único IGP que puede realizar balanceo de carga de costo desigual (unequal-cost load-balancing).
- Por defecto realiza ECMP load-balancig sobre 4 paths como RIP.
- Administrative Distance: 90 (el IGP con la AD más baja)
- Métrica – distancia: ancho de banda y delay. 
- Cuando se configura eigrp en un router hay que especificar el AS (sistema autónomo al que pertenece), esto es, todos los routers que queremos que formen adyacencias EIGRP deben pertenecer al mismo AS.
- El comando no auto summary desactiva la sumarización automática. Esto es, si no activamos esta opción el router va a advertir a sus vecinos la dirección classfull, no cada una de las subredes que caigan en ese rango.
- Cuando se activa EIGRP en un router este envía hello messages a la dirección multicast 224.0.0.10 para encontrar sus vecinos. Para formar una adyacencia con otro router, es decir, para considerarlo vecino se deben cumplir los siguientes requerimientos:
  - Los dos routers tienen que estar en la misma subred.
  - Los dos routers tienen que tener el mismo ASN (authonomous system noumber).
Si se cumplen estos requerimientos entonces el router se convierte en un vecino eigrp, y se pueden comenzar a intercambiar datos sobre la red.
- Cada router que corre eigrp mantiene una tabla denominada neighbours table, que contiene información sobre el estado de los mismos.
- Para asegurarse que los eigrp neighbours están activos existe un hello timer que define cada cuanto se verifica esta conexión mediante el envío de hello messeages.
- Es considerado híbrido por el hecho de que la primera vez que dos routers se convierten en vecinos, intercambian su tabla de enrutamiento completa (full update), pero luego de esto solo intercambian actualizaciones parciales (partial update).
- Similar a la rip database, en eigrp los routers mantienen una Topology Table.

Los pasos que lleva a cabo el router cuando seteamos una net en el comando eigrp son los siguientes:
1. router eigrp *autonomous_system*
2. no aut
3. network A.B.C.D. W.X.Y.Z #siendo W.X.Y.Z la wildcard mask, es opcional
4. el router asume la dirección classfull #si no se especificó wildcard mask
5. chequea en su tabla de enrutamiento todas las direcciones que caigan en ese rango
6. activa eigrp en esas interfaces y por las mismas envía las direcciones con su prefijo apropiado
7. Si en una interfaz no hay otro router, pero aun así queremos que esa dirección sea informada por las demás interfaces pero no le queremos enviar advertisements al pedo tenemos que usar el comando:
passive-interface *int* dentro de la conf del eigrp.

Por defecto el max path de router eigrp es 4, esto es, guarda en la tabla hasta 4 rutas a la misma dirección.

Esto se puede cambiar con maximum-paths *num* pero no tiene mucho sentido 

La AD (distancia administrativa) también se puede cambiar, por defecto es 90 pero con:
distance *distancia* se puede cambiar.

Lo más importante de entender en la práctica al utilizar EIGRP es que la red classfull o con wildcard mask que indicamos en el comando network en la configuración del router EIGRP no es la red que queremos advertir a los vecinos, es una dirección que indica que todas las redes conectadas al router, que caigan en ese rango, es decir, que aplicando el and lógico con la máscara de subred por defecto de la clase de la red indicada (o con la wildcard mask) den como resultado la misma dirección, van a ser advertidas a los vecinos.

## Métrica:	
La métrica de EIGRP es una combinación de 4 factores:
1. Bandwith
   - Determinado por el enlace con el ancho de banda más bajo en el camino al destino.
2. Delay
   - Determinado por la suma de todos los valores de delay en el camino al destino.
3. Interface load (carga sobre la interfaz)
4. Interface reliability (fiabilidad de la interfaz)
   - 3 y 4 no se usan porque si generalmente introducen inestabilidad.


Los valores de red, bandwith y delay que son recibidos por una interfaz en eigrp (el conjunto) es denominado “Reported Distance”. Una vez que el router recibe una reported Distance suma el delay hasta el router del que lo recibió, y compara el ancho de banda para generar una “Feasible distance” al destino.

De las feasible Distance a un mismo destino calculadas a partir de las reported Distances que se reciben por los updates de los eigrp neighbours se elije la más baja para colocar en la tabla de enrutamiento, y esta es denominada “Succesor”. Sin embargo, las demás feasible distances son retenidas por el router en caso de que la succesor se caiga. Estas son denominadas feasible succesors. Es por esto que eigrp tiene una convergencia tan rápida.

Cuando un router en eigrp no tiene ningún feasible successor y se cae un enlace debe solicitar información a sus vecinos para encontrar una nueva ruta, para esto utiliza un algoritmo denominado DUAL (Diffusing Update Algorithm).

Este algoritmo consiste en que cuando se cae un enlace y no se cuenta con feasible successors se envían query messages a los vecinos para encontrar una nueva ruta. Los vecinos si tienen alguna forma de llegar al destino especificado responderán con un reply messeage indicando la nueva ruta.

- K1 y K3 son bandwith y delay. Por defecto vienen activados.
- El ancho de banda del enlace más lento en el camino más la suma de los valores de delay de todos los enlaces en el camino se utilizan para calcular la métrica.
- K2, K4 y K5 vienen apagados por defecto por tanto no se usan para calcular la métrica. Esto puede ser cambiado desde los ajustes.
En EIGRP y OSPF cada Router tiene un identificador único que lo identifica en el AS. Este es determinado de la siguiente forma (de más prioridad a menos):
1. Configuración manual
2. Dirección IP más alta en una interface loopback.
3. Dirección IP más alta en una interface física.

El router ID se configura con:

Router(config-router)#eigrp router-id A.B.C.D

¿Cómo saber si un router es un feasible succesor al successor actual? La reported Distance del mismo debe ser menor a la feasible distance actual.

Para modificar el comportamiento de eigrp lo mejor es modificar el delay de las interfaces.

Esto se hace desde el menú de interfaz con el comando “delay *delay*”.

## EIGRP Timers
- Hello timer: cada cuanto se envían hello’s. El valor por defecto es 5 segundos.
- Hold: cuanto se espera un hello antes de considerar un neigbour como caído. 

## Load Balancing

EIGRP soporta load balancing con rutas de métrica desigual, esto es, por más que la métrica no sea igual se puede hacer balanceo de carga.

Esto se logra modificando un valor denominado “variance” que por defecto viene en “1”.

El algoritmo utilizado para determinar si se puede hacer load balancing consiste en multiplicar el valor de la varianza por la FD, si este valor es mayor que la feasible distance de un feasible successor, entonces se puede hacer load balancing.

Como por defecto la varianza es 1, no se hace load balancing nunca (solo con equal cost paths), pero esto se puede cambiar fácil con el comando variance *multiplicador*


# Open Shortest Path First

Es un protocolo dinámico de enrutamiento que entra en la categoría de Interior Gateway Protocol (IGP; Protocolo de puerta de enlace interna). Utiliza un algoritmo del tipo Link State)
## Características diferenciales del protocolo
- Utiliza el algoritmo del “Camino más corto primero”, del científico de computadoras Edsger Dijkstra. AKA algoritmo de Dijkstra.
- Existen tres versiones de OSPF:
  - OSPFv1 (1989): actualmente no se utiliza.
  - OSPFv2 (1998): utilizado en redes IPv4
  - OSPFv3 (2008): utilizado en redes IPv6 (también soporta IPv4)
- Los routers en OSPF almacenan información acerca de la red en forma de LSA’s (Link State Advertisements), los mismos son organizados en una estructura denominada LSDB (Link State Database)
- Los routers en OSPF difunden LSA’s por inundación hasta que todos los routers en el área OSPF desarrollan el mismo mapa de la red (LSDB)


Funcionamiento básico de un router en una red OSPF que obtiene conocimiento de una nueva red:
1. Se habilita OSPF en la interfaz correspondiente.
2. El router crea un LSA para avisar a sus vecinos sobre la nueva red.
3. El contenido básico del LSA es:
   - Router ID
   - IP de la red
   - Costo
4. El LSA es difundido por inundación en toda la red, hasta que todos los routers del área OSPF lo reciben.
5. Esto resulta en que todos los routers compartan la misma LSDB.
6. Una vez que el LSA fue añadido a la base de datos cada Router aplica el algoritmo de Dijkstra sobre la misma para determinar cual es el camino más conveniente y por tanto que ruta debe agregar a la tabla de enrutamiento.

Los 3 pasos básicos para el compartimiento de LSAs y la determinación de la mejor ruta a una red destino son:
1.	Establecer vecindad: cada router debe establecer una vecindad con los routers conectados en el mismo segmento.
2.	Intercambiar LSAs con los routers vecinos.
3.	Calcular las mejores rutas al destino e insertarla en la tabla de enrutamiento.

## Áreas en OSPF

- OSPF utiliza áreas para dividir la red.
- Las redes pequeñas pueden ser uni-área sin obtener efectos negativos en su performance.
- Sin embargo, en redes más grandes un diseño de red de una sola área puede tener efectos negativos como:
  - El algoritmo del camino más corto implica ocupar demasiadas ranuras de tiempo y poder de procesamiento.
  - La LSDB ocupa demasiada memoria en cada router.
  - Cada cambio en la red provoca demasiado tráfico de broadcast.
- El área “0” es denominada “área backbone”.
- Un área es un conjunto de routers y enlaces que comparten la misma LSDB.
- El área backbone es un área especial a la que todas las demás áreas deben conectarse.

![ospfa](./OSPFAREAS.png)

- Los routers con todas sus interfaces en la misma área se denominan routers internos.
- Los routers con interfaces en múltiples áreas son denominados “área border routers” (ABRs).
- Los ABRs mantienen una LSDB separada por cada área a la que están conectados. 
- Se recomienda conectar un ABR a un máximo de 2 áreas.
- Conectar un ABR a más de 3 áreas puede sobrecargar el router.
- Los routers conectados al área backbone son denominados backbone routers.
- Una ruta cuyo destino se encuentra en la misma área OSPF, es denominada “intra-area route”.
- Una ruta con un destino en un área OSPF distinta es denominada “interarea route”.
- Las áreas en OSPF deben ser contiguas, esto es, una misma área no pude estar dividida en dos segmentos físicamente separados.
- Todas las áreas en OSPF deben tener al menos un ABR conectado al área backbone.
- Las interfaces de OSPF que se encuentran en la misma subred deben estar en la misma área.
- Cuando un router informa a los demás de su área sobre el Gateway de last resort (internet) se convierte en un ASBR (autonomous system Boundary router)
  
## OSPF vs EIGRP:
- El PID no es como el AS number de EIGRP, no es necesario que matchee entre los routers para que se puedan establecer vecindades.
- El comando network funciona igual que en EIGRP
- El comando passive-interface funciona igual que en EIGRP
- El comando default-information originate funciona igual que en EIGRP
- El RID se establece igual que en EIGRP
- Para definir manualmente el RID no es necesario especificar el protocolo como en EIGRP,  simplemente se utiliza (config-router)#router-id A.B.C.D
- Luego de definir manualmente el RID hay que reiniciar el router o los procesos OSPF con #clear ip ospf Process
- OSPF no soporta unequal-cost load balancing, pero si soporta ECMP load balancing (equal cost multiple path). 
