# Servidor DNS esclavo con BIND9

**Transferencia de zonas entre debian-dns y debian-dns2**

| Módulo | 0375 Servicios de red |
|---|---|
| Ciclo formativo | ASIX |
| Alumno | Daniel Valero |
| Dominio | daniel.lan |
| Sistema | Debian GNU/Linux 13 (Trixie) · BIND 9.20 |
| Fecha | 6 de octubre de 2026 |

## 1. Introducción

En la práctica anterior monté un servidor DNS maestro (`debian-dns`) para el dominio `daniel.lan`. En esta práctica añado un segundo servidor, `debian-dns2`, que funciona como **esclavo**: no tiene ficheros de zona propios, sino que copia las zonas del maestro mediante transferencia de zona (AXFR).

Así, si el maestro deja de funcionar, los clientes pueden seguir resolviendo nombres con el esclavo. Además, cada vez que se cambia una zona en el maestro y se sube su número de serie, el maestro avisa al esclavo (NOTIFY) y este se actualiza solo.

He seguido la guía «Configuración de un servidor DNS esclavo con BIND9» de javiercd.es, adaptando las IPs, los nombres y las rutas a mi escenario.

## 2. Escenario

|  | Maestro | Esclavo |
|---|---|---|
| Nombre (FQDN) | `debian-dns.daniel.lan` | `debian-dns2.daniel.lan` |
| IP red interna (Host-only) | `192.168.1.10` (`ens37`) | `192.168.1.11` (`ens37`) |
| IP gestión (Bridged) | `192.168.111.55` (DHCP) | `192.168.111.56` (DHCP) |
| Zonas | `daniel.lan` y `1.168.192.in-addr.arpa` (master) | Las mismas (slave) |
| Ficheros de zona | `/etc/bind/` | `/var/cache/bind/slaves/` |

Las dos máquinas son VMs de VMware Workstation con dos tarjetas: una en modo Bridged para tener Internet y entrar por SSH desde Windows, y otra en modo Host-only, que es la red por la que se comunican los dos servidores DNS.

## 3. Creación de la máquina esclava

Creé una VM nueva con Debian 13 y la misma configuración de red que el maestro: Network Adapter en Bridged y Network Adapter 2 en Host-only.

*Figura 1. Hardware de la VM: adaptador Bridged y adaptador Host-only.*

Durante la instalación puse `debian-dns2` como nombre de máquina y `daniel.lan` como dominio, el mismo que el maestro.

*Figura 2. Nombre de la máquina en el instalador.*

En la selección de programas solo marqué el servidor SSH y las utilidades estándar. Un servidor DNS no necesita entorno gráfico.

*Figura 3. Selección de programas: sin escritorio, con SSH.*

## 4. Red del esclavo

La tarjeta Bridged (`ens33`) coge IP por DHCP. A la tarjeta Host-only (`ens37`) le puse la IP fija `192.168.1.11` en `/etc/network/interfaces`:

```text
auto ens37

iface ens37 inet static
    address 192.168.1.11
    netmask 255.255.255.0
```

*Figura 4. Configuración de `ens37` en `/etc/network/interfaces`.*

La activé con `ifup ens37` y comprobé con `ip a` que estaba UP con su IP.

*Figura 5. `ens37` activa con `192.168.1.11/24`.*

## 5. Paso 1: Preparación inicial del esclavo

Como en la instalación ya puse el nombre y el dominio, `/etc/hostname` contiene `debian-dns2` y `/etc/hosts` ya asocia el FQDN:

```text
127.0.1.1       debian-dns2.daniel.lan  debian-dns2
```

*Figura 6. Fichero `/etc/hosts` del esclavo.*

Lo comprobé con `hostname -f`, que devuelve `debian-dns2.daniel.lan`.

## 6. Paso 2: Instalación de BIND9

```bash
apt update && apt install bind9 bind9utils bind9-doc rsync -y
```

Trabajo como root (`su -`), así que los comandos van sin `sudo`. En Debian 13 se instala BIND 9.20 y el servicio real se llama `named` (`bind9` es un alias).

*Figura 7. Instalación de BIND9 completada y comprobación del hostname.*

## 7. Paso 3: Configuración básica (`named.conf.options`)

En `/etc/bind/named.conf.options` del esclavo permito consultas desde localhost y la red interna, activo la recursión y pongo como reenviadores a Cloudflare y Google:

```text
options {
    directory "/var/cache/bind";

    allow-query { 127.0.0.1; 192.168.1.0/24; };

    recursion yes;

    dnssec-validation no;

    forwarders {
        1.1.1.1;
        8.8.8.8;
    };
};
```

*Figura 8. `named.conf.options` del esclavo.*

Con `named-checkconf` compruebo que la sintaxis es correcta (si no muestra nada, está bien) y con `systemctl status bind9` que el servicio está activo.

*Figura 9. BIND activo y `named-checkconf` sin errores.*

**Nota:** los avisos «network unreachable» con direcciones `2001:…` son intentos de BIND de usar IPv6, que mi red no tiene. No afectan al funcionamiento.

## 8. Paso 4: Configuración de las zonas esclavas

En `/etc/bind/named.conf.local` del esclavo declaro las dos zonas como **slave**, indicando la IP del maestro y dónde guardar la copia:

```text
zone "daniel.lan" {
    type slave;
    masters { 192.168.1.10; };
    file "/var/cache/bind/slaves/db.daniel.lan";
};

zone "1.168.192.in-addr.arpa" {
    type slave;
    masters { 192.168.1.10; };
    file "/var/cache/bind/slaves/db.1.168.192";
};
```

*Figura 10. `named.conf.local` del esclavo con las dos zonas slave.*

Creo la carpeta donde BIND guardará las zonas copiadas y le doy como propietario al usuario `bind`:

```bash
mkdir -p /var/cache/bind/slaves
chown bind:bind /var/cache/bind/slaves
```

## 9. Paso 5: Reiniciar el servicio

```bash
systemctl restart bind9
ls -l /var/cache/bind/slaves
```

*Figura 11. La carpeta de zonas esclavas todavía vacía.*

En este punto la carpeta está vacía (`total 0`). Es lo esperado: el maestro todavía no permite que el esclavo copie sus zonas.

## 10. Paso 6: Actualización del servidor maestro

### 10.1 Autorizar la transferencia

En el maestro (`debian-dns`), en `/etc/bind/named.conf.local`, añado `allow-transfer` con la IP del esclavo en cada zona:

```text
zone "daniel.lan" {
    type master;
    file "/etc/bind/db.daniel.lan";
    allow-transfer { 192.168.1.11; };
};

zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.1.168.192";
    allow-transfer { 192.168.1.11; };
};
```

*Figura 12. `named.conf.local` del maestro con `allow-transfer`.*

### 10.2 Zona directa

En `/etc/bind/db.daniel.lan` añado el esclavo como segundo servidor de nombres (registro **NS**) y su registro **A**. También subo el **serial de 3 a 4**.

*Figura 13. Zona directa con el NS y el A de `debian-dns2` (serial 4).*

### 10.3 Zona inversa

En `/etc/bind/db.1.168.192` añado el NS del esclavo y su **PTR** (`11 → debian-dns2`), y subo el **serial de 2 a 3**.

*Figura 14. Zona inversa con el NS y el PTR del esclavo (serial 3).*

### 10.4 Comprobar y recargar

```bash
named-checkzone daniel.lan /etc/bind/db.daniel.lan
named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.1.168.192
named-checkconf
rndc reload
```

*Figura 15. Las dos zonas cargan correctamente y BIND se recarga.*

### 10.5 Primera transferencia

En el esclavo pedí la copia de las zonas y ahora sí aparecen los dos ficheros, con propietario `bind`:

```bash
rndc retransfer daniel.lan
rndc retransfer 1.168.192.in-addr.arpa
ls -l /var/cache/bind/slaves
```

*Figura 16. Zonas copiadas en el esclavo.*

## 11. Paso 7: Comprobaciones

### 11.1 Conectividad entre servidores

*Figura 17. El maestro llega al esclavo por la red interna.*

### 11.2 El esclavo responde con autoridad

Consultando al esclavo (`dig @192.168.1.11`) la respuesta es la misma que la del maestro, con el flag **aa** (*authoritative answer*). Esto indica que el esclavo es servidor autoritativo de la zona y no está reenviando la consulta.

*Figura 18. A la izquierda responde el esclavo (`.11`) y a la derecha el maestro (`.10`).*

### 11.3 Sincronización automática

Para probar la sincronización, en el esclavo dejé abiertos los logs con `journalctl -u named -f`. En el maestro añadí un registro nuevo y subí el serial de 4 a 5:

```text
sentinel    IN  A       192.168.1.25
```

*Figura 19. Registro `sentinel` añadido y serial 5 en el maestro.*

Al reiniciar el maestro (`systemctl restart bind9`), en los logs del esclavo se ve el proceso completo:

- Recibe el **notify** del maestro para `daniel.lan` con **serial 5**.
- Empieza la transferencia (*Transfer started*).
- Termina con éxito: *Transfer completed: 9 records … (serial 5)*.
- La zona inversa responde *zone is up to date* porque no se modificó.

*Figura 20. Logs del esclavo: NOTIFY y transferencia completada (serial 5).*

### 11.4 Los dos servidores devuelven lo mismo

```bash
rndc sync
dig @192.168.1.10 sentinel.daniel.lan
dig @192.168.1.11 sentinel.daniel.lan
```

*Figura 21. `sentinel.daniel.lan` resuelve a `192.168.1.25` en los dos servidores.*

Los dos servidores devuelven `NOERROR`, el flag `aa` y la misma respuesta: **`192.168.1.25`**.

## 12. Incidencias

- **Error de réplica en el instalador:** la VM no tenía salida a Internet durante la instalación. Lo resolví volviendo a «Configurar la red» desde el menú del instalador.
- **`sudo: orden no encontrada`:** ya estaba como root, así que bastaba con quitar `sudo` de los comandos.
- **«El fichero no es de escritura» en nano:** había abierto el fichero como `daniel`. Lo solucioné entrando con `su -` antes de editar.
- **`journalctl -xeu bind9` sin entradas:** en Debian 13 la unidad real es `named.service`, así que hay que usar `journalctl -u named`.
- **Nombre de interfaz distinto:** en el instalador la segunda tarjeta salía como `ens34`, pero en el sistema instalado es `ens37`. Lo comprobé con `ip a` antes de configurarla.
- **Cambio de IP Bridged del maestro:** por DHCP pasó de `192.168.111.54` a `.55`. No afecta al DNS, que usa la red interna `192.168.1.0/24`.

## 13. Conclusiones

- Un DNS esclavo da redundancia: si el maestro cae, el esclavo sigue respondiendo con autoridad.
- Las zonas solo se editan en el maestro. El esclavo las recibe por transferencia (AXFR) cuando recibe un NOTIFY.
- El número de serie es lo que decide si hay que transferir. Si no se sube, el esclavo considera que ya está al día.
- `allow-transfer` limita quién puede copiar las zonas, lo que evita que cualquiera pueda descargar todos los registros del dominio.

## 14. Referencias

- [Configuración de un servidor DNS esclavo con BIND9 — javiercd.es](https://www.javiercd.es/posts/servicios/dns/bind9/dns_esclavo/dns_esclavo/)
- [ISC BIND 9](https://www.isc.org/bind/)