# INSTALL DNS

## &#x20; <a href="#fdldqdo5uds4" id="fdldqdo5uds4"></a>

<table data-header-hidden><thead><tr><th valign="top"></th><th valign="top"></th></tr></thead><tbody><tr><td valign="top">Datos de la Práctica</td><td valign="top">Análisis y Resolución del Sistema DNS</td></tr><tr><td valign="top">Módulo</td><td valign="top">0375 Servicios de red</td></tr><tr><td valign="top">Ciclo Formativo</td><td valign="top">ASIX</td></tr><tr><td valign="top">Alumno</td><td valign="top"> Daniel Valero</td></tr><tr><td valign="top">Formato de Entrega</td><td valign="top">URL al informe documentado en Gitbook personal</td></tr></tbody></table>

&#x20;

Objetivos

Instalar y configurar servidores de DNS

&#x20;&#x20;

Instala – configura - verifica \[5p]

1. Instalar y configurar un servidor DNS.

&#x20;

En una VM con Debian, instala y configura el servicio de DNS. Para ello, puedes seguir la guía que está en punkymo.

&#x20;

&#x20;

Preguntas

1\.    ¿Qué función cumple el servicio DNS en la red y por qué es esencial para el funcionamiento de Internet\
<br>

* Básicamente el DNS funciona como la lista de contactos del móvil. A las personas nos resulta muy fácil acordarnos de nombres como `google.com` o `debian-dns.daniel.lan`, pero las máquinas y los routers solo trabajan con números (direcciones IP como `192.168.1.10`). El DNS se encarga de traducir esos nombres en IPs de forma automática en cuanto intentamos conectar.\
  \
  Es esencial porque nadie podría memorizar las IPs de cada web o servicio que usa a diario. Además, si un servidor cambia de IP o de máquina física, con cambiar el registro en el DNS los usuarios siguen entrando con el mismo nombre de siempre sin enterarse del cambio.



2\.    ¿Cómo se estructura la jerarquía del DNS y qué rol cumple cada nivel?\
<br>

* **La raíz (`.`):** Es la cima de la jerarquía. No almacena nombres de sitios web individuales; su rol es redirigir la consulta hacia los servidores encargados de cada terminación de dominio.
* **Dominio de nivel superior (TLD - Top-Level Domain):** Son las terminaciones como `.com`, `.org`, `.es`, o de uso privado e infraestructural como `.lan` y `.arpa`. Su rol es indicar qué servidor autoritativo gestiona cada organización registrada bajo esa extensión.
* **Dominio de segundo nivel (SLD):** Es el nombre del proyecto, empresa o dominio particular (en nuestro caso, `daniel` dentro de `daniel.lan`).
* **Subdominios y nombres de host:** Representan equipos, departamentos o servicios individuales dentro de esa red privada (por ejemplo, `debian-dns` o `servidor` en `daniel.lan`).<br>

3\.    ¿Cuál es la diferencia entre un servidor DNS iterativo y un servidor recursivo?\
<br>

* **Servidor Recursivo:** Trabaja de principio a fin por el cliente. Si le preguntas por una dirección que desconoce, él mismo realiza todas las consultas necesarias a través de Internet, saltando de servidor en servidor, hasta obtener la respuesta definitiva y entregársela al usuario.
* **Servidor Iterativo:** Se limita a responder con la mejor información que tiene a mano sin hacer trabajo adicional. Si no conoce la respuesta, no busca en otros sitios; simplemente le dice al cliente: _"Yo no lo sé, ve a preguntarle a este otro servidor"_.<br>

4\.    ¿Qué son los registros de recursos (RR) en DNS y cuáles son los más importantes? Haz una lista.<br>

* **SOA (Start of Authority):** La ficha principal. Indica quién es el administrador de la zona, cuál es el servidor maestro y qué número de serie (versión) tiene la configuración.
* **NS (Name Server):** Indica qué máquinas actúan como servidores de nombres autorizados para ese dominio.
* **A (Address):** Traduce un nombre de equipo a una dirección IPv4 (Nombre → IPv4).
* **AAAA:** Cumple la misma función que el registro A, pero apunta a una dirección moderna IPv6.
* **PTR (Pointer):** Realiza la traducción inversa; toma una dirección IP y devuelve el nombre de equipo asociado (IPv4 → Nombre).
* **CNAME (Canonical Name):** Es un alias o apodo que redirige un nombre hacia otro registro principal ya existente.
* **MX (Mail Exchange):** Especifica a qué servidor deben enviarse los correos electrónicos dirigidos a ese dominio.
* **TXT (Text):** Almacena texto informativo, utilizado principalmente para firmas de seguridad y validación de correo electrónico (SPF, DKIM).



5\.    ¿Qué vulnerabilidades existen en el servicio DNS  y qué mecanismos de seguridad pueden mitigarlas?

&#x20;

* **Vulnerabilidades existentes:**
  * _Envenenamiento de caché (Cache Poisoning / Spoofing):_ Un atacante introduce respuestas falsas en la memoria intermedia de un servidor DNS para desviar a los usuarios hacia servidores fraudulentos.
  * _Ataques de denegación de servicio (DDoS por amplificación):_ Se aprovechan servidores DNS mal configurados para inundar de tráfico a una víctima y dejarla sin conexión.
  * _Ataques de intermediario (Man-in-the-Middle):_ Como las consultas DNS viajan normalmente en texto plano por la red, terceros pueden espiar qué sitios se visitan o alterar las respuestas sobre la marcha.
* **Mecanismos de mitigación:**
  * **DNSSEC:** Sistema de firmas criptográficas que permite al cliente comprobar que la respuesta proviene del servidor auténtico y no fue modificada en el camino.
  * **Cifrado de consultas (DoT / DoH):** Cifra el tráfico DNS mediante TLS o HTTPS para que nadie en la red local o intermedia pueda espiar las solicitudes.
  * **Control de acceso:** Restringir la recursión en el servidor DNS para atender únicamente a los ordenadores de la propia red local, evitando que sea utilizado por atacantes externos.

Reporte técnico \[2p]

Escribe un informe técnico y un manual de usuarios en github.

&#x20;

El informe debe contener:

●        Introducción - Un breve resumen a partir de las preguntas que te dejo más abajo

●        Características de hardware y del sistema operativo utilizados para trabajar el DNS\
<br>

* **Sistema Operativo:** Debian GNU/Linux 13 (Trixie), versión 13.7 de 64 bits.
* **Entorno de virtualización:** VMware Workstation (máquina virtual rotulada como "Ubuntu").
*   **Hardware asignado a la VM:**

    * Procesadores: 4 núcleos virtuales (vCPU).
    * Memoria RAM: 4.9 GB.
    * Almacenamiento: Disco virtual de 60 GB (interfaz SCSI).<br>
      * **Software de resolución:** BIND9 (versión 9.18).
      * **Cliente de pruebas:** PC anfitrión con Windows 11 (PowerShell en la red `192.168.111.x`).


* **Tarjetas de red:**
  * `ens33` (Network Adapter - Modo Bridged): IP `192.168.111.54/24` (para gestión remota y comunicación con Windows).
  * `ens37` (Network Adapter 2 - Modo Host-only): IP estática `192.168.1.10/24` (donde presta servicio el DNS local).

●        Desarrollo\
<br>

Todos los comandos se ejecutaron como superusuario (`root`) usando `su -`:

1.  **Instalación de paquetes:**

    ```
    apt update
    apt install bind9 bind9utils bind9-doc -y
    ```

    Usa el código con precaución.
2.  **Declaración de zonas maestras (`/etc/bind/named.conf.local`):**

    ```
    zone "daniel.lan" {
        type master;
        file "/etc/bind/db.daniel.lan";
    };

    zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.1.168.192";
    };
    ```

    Usa el código con precaución.
3.  **Zona directa (`/etc/bind/db.daniel.lan`):**\
    Se configuró con Serial 3, añadiendo el registro raíz `@` y los nombres de host:

    ```
    $TTL    604800
    @       IN      SOA     debian-dns.daniel.lan. root.daniel.lan. (
                                  3         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      debian-dns.daniel.lan.
    @       IN      A       192.168.1.10

    debian-dns IN   A       192.168.1.10
    servidor   IN   A       192.168.1.10
    ```

    Usa el código con precaución.
4.  **Zona inversa (`/etc/bind/db.1.168.192`):**\
    Se configuró el puntero PTR para resolver la IP al nombre canónico:

    ```
    $TTL    604800
    @       IN      SOA     debian-dns.daniel.lan. root.daniel.lan. (
                                  2         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      debian-dns.daniel.lan.

    10      IN      PTR     debian-dns.daniel.lan.
    ```

    Usa el código con precaución.
5. **Configuración de opciones (`/etc/bind/named.conf.options`):**\
   Se habilitó la recepción de consultas de clientes (`allow-query { any; };`) y se añadieron los servidores DNS de Google (`8.8.8.8`) y Cloudflare (`1.1.1.1`) para consultas hacia Internet.
6.  **Persistencia del resolver y servicio en Debian:**&#x62;ash

    ```
    echo -e "search daniel.lan\nnameserver 192.168.1.10" > /etc/resolv.conf
    chattr +i /etc/resolv.conf
    systemctl enable named
    ```

    Usa el código con precaución.
7.  **Verificación de sintaxis:**

    ```
    named-checkconf
    named-checkzone daniel.lan /etc/bind/db.daniel.lan
    named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.1.168.192
    rndc reload
    ```



●        Análisis de incidencias técnicas\
<br>

* **Falta de permisos al editar archivos:** Al abrir los ficheros con el usuario estándar `daniel`, el sistema impedía guardar los cambios en `/etc/bind/`. Se solucionó accediendo como superusuario mediante `su -`.
* **Error `No answer` en `daniel.lan`:** Al consultar el dominio a secas no devolvía IP porque faltaba el registro raíz (`@ IN A 192.168.1.10`). Se añadió la línea, se incrementó el serial a 3 y se recargó con `rndc reload`.
* **El archivo `/etc/resolv.conf` se borraba tras reiniciar:** El demonio DHCP volvía a colocar los DNS del router al reiniciar la máquina. Se solucionó blindando el fichero con el atributo inmutable `chattr +i /etc/resolv.conf`.
* **Aviso de enlace simbólico al habilitar BIND9:** La orden `systemctl enable bind9` daba una advertencia porque en Debian 12/13 es un alias de `named.service`. Se solucionó ejecutando `systemctl enable named`.
* **Tiempo de espera agotado desde Windows:** Windows estaba en la subred `192.168.111.x` y no alcanzaba directamente la tarjeta `192.168.1.10`. Se solucionó consultando a la IP compartida `192.168.111.54`.

●        Conclusiones



* Montar un servidor DNS local optimiza la red interna al permitir la comunicación por nombres de host sin depender de conexiones externas.
* La zona inversa es fundamental para certificar la autenticidad de los equipos en la red mediante registros PTR.
* El uso de herramientas de verificación (`named-checkzone`) y el blindaje con `chattr +i` garantizan que el servidor arranque y opere de forma estable tras cualquier reinicio.

El manual de usuario debe contener:

●        Los aspectos más importantes de la configuración del servicio de DNS.

&#x20;

&#x20;

Para que cualquier ordenador de la red pueda utilizar este servidor DNS, debe configurarse con los siguientes parámetros:

* **Dominio de búsqueda:** `daniel.lan`
* **Servidor DNS principal (Red Interna):** `192.168.1.10`
* **Servidor DNS alternativo (Red Compartida):** `192.168.111.54`



1. Pulsa **Aceptar** en ambas ventanas.

Configuración en clientes Linux

1.  Abre la terminal y edita el resolver:

    ```
    sudo nano /etc/resolv.conf
    ```


2.  Añade estas líneas:

    ```
    search daniel.lan
    nameserver 192.168.1.10
    ```


3. Guarda con **Ctrl + O** y sal con **Ctrl + X**.



*   **Comprobación directa:**

    ```
    nslookup debian-dns.daniel.lan
    ```


*   **Comprobación inversa:**

    ```
    nslookup 192.168.1.10
    ```


*   **Comprobación por ping:**

    ```
    ping debian-dns.daniel.lan
    ```



Links

&#x20;

1. Instalar Bind9: [https://www.isc.org/bind/](https://www.isc.org/bind/)
2. Configurar las IP con NetPlan: [https://netplan.io](https://netplan.io/)
3. [https://punkymo.gitbook.io](https://punkymo.gitbook.io/miwiki/servicios/dns/dns-ubuntu-server-22.04)

&#x20;
