# 🥊 Knockout Rush

## Descripción general

Knockout Rush es un juego multijugador en tiempo real desarrollado en Unity donde hasta 3 jugadores compiten en una arena para ser el último en pie. Cada jugador puede golpear a los demás con click izquierdo para empujarlos fuera del mapa. Si caes al vacío, quedas eliminado. El último jugador que permanezca en la arena gana la partida.

### Mecánicas principales
- Movimiento en primera persona con físicas de empuje
- Golpe con click izquierdo que aplica fuerza al jugador apuntado
- Eliminación al caer fuera del mapa
- Colores aleatorios por jugador para identificación visual
- Panel de administración para el host con kick y pausa

---

## Comunicación cliente-servidor

Knockout Rush utiliza **UDP (User Datagram Protocol)** para la comunicación en red, implementado con `UdpClient` de .NET. La arquitectura es **cliente-servidor centralizado**: uno de los jugadores actúa como host (servidor + cliente) y los demás se conectan como clientes.

### Flujo de conexión

```
Cliente                        Servidor
  |                                |
  |-------- CONNECT -------------->|
  |<------- CONNECTED|{id} --------|
  |                                |
  |-------- SPAWN|id|x|y|z|r|g|b ->|
  |                                |---- SPAWN broadcast a todos
  |<------- PLAYER_JOINED|id ------|
  |                                |
  |====== Juego en curso ==========|
  |                                |
  |-------- POS|id|x|y|z|rx|ry --->|  (20 veces/segundo)
  |                                |---- POS broadcast a todos
  |-------- PING|id -------------->|
  |<------- PING_RES|id -----------|
```

### Mensajes del protocolo

| Mensaje | Descripción |
|---|---|
| `CONNECT` | Cliente solicita conexión al servidor |
| `CONNECTED\|id` | Servidor confirma conexión y asigna ID |
| `SPAWN\|id\|x\|y\|z\|r\|g\|b` | Jugador aparece en la escena con posición y color |
| `DESPAWN\|id` | Jugador es removido de la escena |
| `POS\|id\|x\|y\|z\|rx\|ry` | Actualización de posición y rotación |
| `PUSH\|id\|dx\|dy\|dz` | Empuje aplicado a un jugador |
| `ELIMINAR\|id` | Jugador eliminado de la partida |
| `KICK\|id` | Host expulsa a un jugador |
| `PAUSE\|0\|1` | Host pausa o reanuda el juego |
| `PING_REQ\|id` | Solicitud de ping |
| `PING_RES\|id` | Respuesta de ping |
| `PING_UPDATE\|id\|ms` | Broadcast del ping de un jugador |
| `PLAYER_JOINED\|id` | Notificación de nuevo jugador |

### Detección de desconexión

El servidor utiliza un sistema de **heartbeat + timeout**: cada cliente envía un `PING` cada 2 segundos. Si el servidor no recibe ningún mensaje de un cliente en 5 segundos, lo considera desconectado y notifica a los demás con un `DESPAWN`.

---

## Instrucciones para ejecutar

### Como Host
1. Abre el juego en tu máquina
2. En el menú principal selecciona **"Host"**
3. El juego cargará la escena principal automáticamente
4. Comparte tu IP local con los demás jugadores

### Como Cliente
1. Abre el juego en tu máquina
2. En el menú principal selecciona **"Join"**
3. Ingresa la IP del host
4. El juego cargará la escena principal automáticamente

### Controles
| Acción | Control |
|---|---|
| Moverse | WASD |
| Saltar | Espacio |
| Mirar | Mouse |
| Golpear | Click izquierdo |
| Menú de pausa | Escape |

---

## Requisitos técnicos

| Requisito | Detalle |
|---|---|
| Motor | Unity 6 o superior |
| Plataforma | Windows |
| Red | LAN o red local compartida |
| Jugadores | 2 a 3 jugadores |
| Puerto | UDP 7777 (configurable) |
| Conexión | El host debe tener el puerto 7777 abierto |

---

## Limitaciones conocidas

- **Máximo 3 jugadores:** el sistema no fue diseñado ni probado para más de 3 jugadores simultáneos.
- **Sin reconexión:** si un jugador pierde la conexión no puede volver a unirse a la misma partida.
- **UDP sin garantía de entrega:** al usar UDP, en redes inestables algunos paquetes pueden perderse, causando leves desincronizaciones de posición.
- **Red local únicamente:** el juego no soporta conexiones a través de internet sin configuración adicional de red (port forwarding o VPN).
- **Sin sistema de lobbies:** los jugadores deben conocer la IP del host de antemano.
