# R1.3 · Ampliación de evidencia mediante lectura offline de tres VMs

**Fecha:** 23-nov-2026 UTC. Complementa el cierre R1.3 publicado en `04c0b89490d345039a3c7360d4bd9948dc850857`.

## Resultado

El usuario apagó Zabbix, Wazuh y Suricata. Se verificó su apagado en ESXi y se leyeron sus discos por rangos, sin montaje, activación de volúmenes, replay del journal ni escritura explícita al VMDK.

**Se pudo superar la falta de acceso SSH a los archivos de estas VMs. No se recuperaron operaciones del piloto del 27–30 sep que resuelvan la atribución de las 158 ventanas.** Las etiquetas, los offsets aplicados (cero), las relaciones temporales nominales y las métricas se conservan. El manuscrito no se modifica.

La actualización cambia el estado de las fuentes: Wazuh ya no es simplemente «archivos inaccesibles»; sus catálogos persistentes se pudieron inspeccionar. Zabbix pasa a «logs inspeccionados offline, tablas históricas no consultadas». Suricata incorpora un contenido EVE verificado de octubre. No se declara la inexistencia global de evidencia histórica.

## 1. Método y límites de integridad

- ESXi hv-siem: VMID 2 Wazuh, 10 Zabbix y 8 Suricata, apagadas al inicio y al final de cada adquisición satisfactoria.
- Los VMX apuntaban directamente a sus descriptores base VMDK. Estos declaran extents VMFS planos, `parentCID=ffffffff`; las consultas de snapshots no mostraron un árbol. Se conservan esos inventarios como precondiciones de la lectura del `-flat.vmdk`.
- Tamaños lógicos: Wazuh 160 GiB; Zabbix y Suricata 40 GiB cada una. No se descargaron imágenes completas.
- Lecturas remotas mediante SSH con clave ya existente en el bastión y comprobación estricta de host. `dd` solo con archivo de entrada, bloque de 64 KiB, sin `of` y sin PTY.
- Parsing de GPT/MBR, LVM y ext4 con `dissect.volume 3.18`, `dissect.extfs 3.15`, `dissect.cstruct 4.7` y `dissect.util 3.24` en su variante Python pura. No se importó código del invitado ni se ejecutaron generadores o modelos.
- Bibliotecas temporales del bastión: se extrajeron en un `TemporaryDirectory` para esta ejecución y se eliminaron al salir. No se instalaron paquetes del sistema, ni se obtuvo sudo, ni se alteraron servicios o claves.
- El primer intento falló antes de leer discos por importación del namespace de las bibliotecas desde wheels. Se conserva el error y el posterior intento correcto.
- Todos los sistemas ext4 reconocidos declaraban `s_state=1` y sin bit `RECOVER` en los flags observados. Es un estado declarado en el superblock, **no una comprobación completa de consistencia ni de transacciones de aplicaciones**.
- Los mapas LVM finales de Wazuh y Suricata tienen un solo PV y segmentos lineales representados como `striped` de una sola stripe. No se aplicó un parser genérico a un RAID/thin/striping complejo.

### Qué comprueban realmente los controles

El bloque inicial se leyó dos veces de forma consecutiva y coincidió. La salida de `ls -ln` del disco fue igual antes/después. La VM figuraba apagada en ambos extremos. Estos son **controles puntuales**, no una imagen atómica, un hash de todo el disco o una exclusión de cualquier escritura concurrente por terceros. La resolución temporal de `ls` es limitada.

Los SHA-256 de archivos extraídos identifican sus bytes. No certifican autenticidad histórica ni exhaustividad. Los directorios ausentes o un error por ruta se revisan individualmente; el campo `complete` de las capturas iniciales significa fin de ejecución y **no debe interpretarse como éxito de todas las rutas solicitadas**. Los sondeos de EFI/swap como no-ext4 son resultados esperados. En las capturas finales se explicitó esa semántica y se reforzaron las extracciones para exigir archivo regular y longitud idéntica al inode.

No se hicieron carving de archivos borrados, recuperación de journal, decodificación completa de Lucene, apertura de PostgreSQL ni arranque de copias de bases de datos. Los errores de rutas de otras particiones no se interpretan como ausencia en la raíz pertinente.

## 2. Wazuh: volumen Docker persistente

La raíz de la VM no contiene un manager tradicional en `/var/ossec`. El manager conserva sus datos en el LV **`wazuh-data`**, cuyo último punto de montaje es `/var/lib/docker`. Se siguieron los volúmenes Docker persistentes, en particular:

- `/volumes/single-node_wazuh_logs/_data/archives`
- `/volumes/single-node_wazuh_logs/_data/alerts`
- `/volumes/single-node_wazuh-indexer-data/_data`

El recorrido acotado de logs y catálogo del indexador enumeró **459 entradas**, sin alcanzar el límite de 4 000. En los catálogos de archives/alerts aparecen `2026/Oct` y `2026/Nov`; las dos rutas `2026/Sep` solicitadas explícitamente no están presentes. La ausencia corresponde a esas rutas y al corte de disco examinado, no a toda posible ubicación o respaldo.

Se leyó y descomprimió completamente la alerta JSON más antigua localizada por su nombre de día, `alerts/2026/Oct/ossec-alerts-18.json.gz`:

| Comprobación | Resultado |
|---|---|
| Tamaño comprimido | 53 744 bytes |
| SHA-256 | `3a98c0be574a3cca6de9a81841889dceb6687051568c3e7cd674dc0964081c13` |
| Eventos JSON | 188 |
| Marca mínima | `2026-10-18T21:16:33.876+0000` |
| Marca máxima | `2026-10-18T23:16:10.114+0000` |
| Eventos rotulados 27–30 sep | 0 |

Se exportaron únicamente el resultado temporal y el hash, no los mensajes de las alertas. Este muestreo completo de un archivo **no sustituye a consultar todos los documentos del indexador**.

Se localizó el catálogo de nodos y directorios de índices Lucene/OpenSearch, pero no se decodificaron sus documentos. Un índice puede contener eventos importados o fechas distintas de las de sus directorios. Por tanto, **los índices continúan sin consulta semántica**, aunque su ubicación ya fue inspeccionada. No se afirma que estén vacíos ni que no contengan septiembre.

## 3. Zabbix: logs del servicio y PostgreSQL

Se identificó la partición ext4 raíz y PostgreSQL 17 en `/var/lib/postgresql/17/main`. Se enumeraron sus directorios y archivos de control por nombre, sin leer credenciales ni volcar la base.

Se extrajeron completamente estas rotaciones localizadas como las más antiguas de sus respectivos listados:

| Archivo | Contenido observado |
|---|---|
| `zabbix_server.log.2.gz` | 2 514 líneas; primera marca `20261112:091435.194`, última `20261114:235121.278` |
| `zabbix_agent2.log.2.gz` | 134 líneas; del 12-nov 09:05:28 al 14-nov 23:50:16, en la representación del propio log |
| `postgresql-17-main.log.2.gz` | 1 467 líneas; 12-nov 09:05:47 a 15-nov 00:08:10, con offset `−05` explícito |

Ninguna de esas lecturas contiene una fecha de evento del 27–30 sep. Las marcas sin zona no se convierten a UTC por suposición. El archivo actual de Zabbix leído termina con la parada del servicio del 22-nov 23:48 en la representación de su log, compatible con el apagado UTC del 23-nov; no se usa para calibrar relojes históricos.

**Las tablas history/trends/events de Zabbix no fueron consultadas.** Los logs de arranque de noviembre y el mtime de PostgreSQL no prueban que la base carezca de historia importada de septiembre. La falta de token autenticado del cierre anterior no queda resuelta por poder leer los archivos offline. Queda separada como limitación concreta: se requiere una consulta de lectura de la base o una copia aislada autorizada y técnicamente consistente para resolver su contenido.

## 4. Suricata: raíz y LV de logs

Se leyeron tanto el catálogo de `/var/log/suricata` de la raíz subyacente como el LV **`suricata-logs`**, cuyo último montaje es `/var/log/suricata`. Así se evita perder archivos ocultos por el punto de montaje.

El respaldo no vacío más antiguo localizado en ambos catálogos, `eve.json.1-2026101800.backup`, tiene un nombre/mtime que **no coincide con la fecha real de sus eventos**:

| Comprobación | Resultado |
|---|---|
| Tamaño | 76 730 bytes |
| SHA-256 | `e053daa596f50be64b8be65fb069d1266c872b1506c0ad82c3fed30df1394479` |
| Eventos JSON | 96 |
| Marca mínima | `2026-10-02T01:46:03.225902+0000` |
| Marca máxima | `2026-10-02T01:46:26.164224+0000` |
| Eventos del 27–30 sep | 0 |

El log de Suricata acompañante tiene 18 líneas y marcas del 2-oct. El EVE se volvió a leer en la captura final de mapeo LVM y coincidió por hash. No se extrapola ese resultado al contenido de todas las rotaciones posteriores, de los bloques no asignados ni de otros respaldos.

## 5. Efecto sobre R1.3 y la matriz

Se entrega **`MATRIZ-158-OFFLINE.csv`**, derivada de la matriz publicada anterior. Cambian solamente:

1. `evidencia_wazuh`: catálogos persistentes inspeccionados, septiembre no localizado en esas rutas; documentos de índices no consultados.
2. `evidencia_zabbix`: logs inspeccionados offline, sin evidencia del piloto en los extraídos; tablas históricas no consultadas.
3. Columna nueva `evidencia_suricata_offline`: respaldo EVE del 2-oct inspeccionado, sin evidencia por ventana del piloto.

Se verifican intactas las 158 claves, las etiquetas históricas, las características, todos los cruces de historias/fronteras, los offsets cero y los estados de atribución incierta. **No se agregan etiquetas ni se modifica ninguna predicción.**

La nueva evidencia reduce la incertidumbre sobre **dónde y qué se pudo inspeccionar**. No resuelve los pares origen/destino ni los tiempos de cada ventana del piloto. Se mantienen las conclusiones sustantivas: 47 cierres nominales dentro, 4 solo por historia y 33 sin intervalo documentado; las 74 referencias continúan sin normalidad verificada. No se recalcula Tabla 8.

### Pendientes definidos

- Wazuh: consulta semántica de los documentos de índices ya localizados, si se quiere agotar esa fuente; catálogo inspeccionado no equivale a consulta de eventos.
- Zabbix: consulta de history/trends/events con acceso de lectura apropiado o copia aislada consistente; no suponer historia vacía por fechas de instalación.
- VM-BACKUP/hv-datos: permanecen los límites del cierre anterior; no estaban entre las tres VMs apagadas y no se intervino en ellos.
- Las fuentes externas de episodios siguen teniendo una incertidumbre de alineación con el sensor que esta lectura no calibra.

No se atribuye un ataque ni normalidad por la ausencia de una alerta, ni se selecciona un offset según predicciones.

## 6. Publicación y reproducibilidad

Las capturas originales locales conservan stdout y su hash, parámetros y resultados, incluidas las primeras importaciones fallidas. Las capturas con archivos de logs extraídos contienen sus bytes codificados y sus SHA-256 y se conservan localmente para trazabilidad.

**El paquete público utiliza copias filtradas**: elimina el contenido base64 de logs de aplicaciones y conserva rutas, tamaños, hashes, inventarios y resultados temporales. Incluye un hash de la captura local original y un hash nuevo del stdout público; no presenta los bytes filtrados como captura literal. Esto evita publicar mensajes operativos ajenos a la pregunta del piloto. No se exportaron documentos de alertas de Wazuh ni credenciales.

Se incluyen scripts propios, planes, inventario consolidado, resumen, matriz actualizada y manifiesto. Las bibliotecas externas se identifican por versión y hash de wheel; no se redistribuyen en el repositorio. Los scripts de adquisición necesitan el acceso original del laboratorio y las bibliotecas; el verificador local no accede a la red.

**Estado operativo al terminar la adquisición:** las tres VMs seguían apagadas. No se encendieron ni se conectaron sus discos en escritura. No se afirma que todos los dispositivos del laboratorio estuvieran apagados.
