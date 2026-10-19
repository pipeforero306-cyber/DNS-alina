# Parte 6: Buenas prácticas del diseño de redes

*Grupo: Pau, Abde y Daniel · Módulo 0375 Servicios de red*

## 11. ¿Qué es lo recomendable a la hora de diseñar una infraestructura DNS? ¿A qué se refiere con el principio de separación estricta de funciones?

Lo recomendable es no montarlo todo en un solo servidor. Lo normal es tener dos servidores autoritativos (un primario y un secundario, mejor si están en máquinas distintas) y aparte los servidores recursivos para los equipos de la red.

La separación estricta de funciones quiere decir que cada servidor hace una sola cosa: o es autoritativo o es recursivo, pero no las dos a la vez. Así, si uno falla o recibe un ataque, el otro servicio sigue funcionando.

## 12. ¿Qué atienden los servidores autoritativos públicos? ¿Qué deberían tener totalmente desactivado?

Atienden las consultas que llegan desde Internet sobre las zonas de nuestro propio dominio, por ejemplo la IP de la web de la empresa.

Deben tener la recursividad totalmente desactivada (`recursion no;`). Si no, cualquiera podría usarlos para resolver dominios ajenos y se podrían aprovechar para ataques de amplificación DDoS. También se limita la transferencia de zona al servidor secundario con `allow-transfer`.

## 13. ¿Qué función cumplen los servidores recursivos internos o resolvers? ¿Con qué opción se restringe únicamente a los rangos IP de la organización?

Resuelven los nombres de cualquier dominio para los equipos de la red interna. Preguntan a los servidores raíz o a un forwarder y guardan la respuesta en caché para ir más rápido la próxima vez.

Para que solo contesten a los rangos de la organización se usa `allow-recursion`, por ejemplo:

```
allow-recursion { 127.0.0.1; 192.168.1.0/24; };
```

## Infografía de la práctica

En la práctica montamos un servidor DNS esclavo (`debian-dns2`) que copia las zonas del maestro (`debian-dns`). Este es el resumen de lo que hicimos. El detalle paso a paso está en el [README principal](../README.md).

![Infografía: DNS maestro y esclavo con BIND9](img/infografia.png)
