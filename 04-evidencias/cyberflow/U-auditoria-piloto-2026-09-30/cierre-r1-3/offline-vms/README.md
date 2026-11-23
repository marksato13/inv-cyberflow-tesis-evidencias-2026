# R1.3 · Inspección offline de Wazuh, Zabbix y Suricata

**Corte:** 23-nov-2026 UTC. El usuario apagó las tres VMs; se verificó ese estado y se accedió por lectura de rangos a sus discos planos desde ESXi. No se montaron sistemas de archivos, reprodujeron journals ni arrancaron bases de datos.

## Qué cambia respecto al cierre anterior

| Fuente | Evidencia nueva | Límite que permanece |
|---|---|---|
| Wazuh | Catálogo de volúmenes Docker persistentes: archives/alerts con Oct/Nov y sin las rutas Sep solicitadas. Archivo de alertas Oct18 leído íntegro: 188 eventos de octubre. | Los documentos de índices Lucene/OpenSearch no se consultaron semánticamente. No se prueba ausencia global de septiembre. |
| Zabbix | Logs y directorios PostgreSQL inspeccionados offline. Las rotaciones extraídas de Zabbix/PostgreSQL comienzan el 12-nov. | Las tablas history/trends/events no se consultaron; podría existir historia importada. |
| Suricata | Raíz subyacente y LV de logs inventariados. Respaldo EVE inspeccionado completo: 96 eventos del 2-oct. | Lectura selectiva, no todas las rotaciones ni recuperación de bloques borrados. |

**No se recuperaron operaciones del piloto que resuelvan las etiquetas de las 158 ventanas.** La matriz actualiza únicamente las columnas de evidencia de estas fuentes. Conserva claves, etiquetas originales, características, cruces de historia/fronteras, offsets cero y atribuciones inciertas. No se cambió el manuscrito ni se recalculó Tabla 8.

## Entrega

- [Informe offline y límites de integridad](INFORME-OFFLINE-R1.3.md).
- [Matriz actualizada de 158 ventanas](MATRIZ-158-OFFLINE.csv).
- [Fuentes y límites actualizados](FUENTES-Y-LIMITES-OFFLINE.csv).
- [Inventario consolidado: 928 entradas](INVENTARIO-OFFLINE.csv).
- [Controles de adquisición y pruebas del lector](VERIFICACION-OFFLINE.json).
- [Manifiesto de entrega](MANIFIESTO-ENTREGA.json).
- [Paquete publicable: scripts, planes y capturas filtradas](evidencia-offline-publicable.zip).

**SHA-256 del ZIP:** `c2f75faa492db49c7fa7d1a4bdd36e3c144a01a217c407d1bc03523ac8578e87` (86 172 bytes; 34 entradas).

El ZIP conserva hashes, rutas, inventarios y resultados temporales, pero excluye el contenido base64 de logs de aplicaciones. Cada captura filtrada identifica su original local por hash y recalcula el del stdout publicado. No se exportaron mensajes completos de alertas de Wazuh, claves privadas o tokens. Las bibliotecas externas solo se identifican por versión y SHA-256, no se redistribuyen.

Se verificaron diez ejecuciones offline satisfactorias, las extracciones solicitadas y los mapas LVM lineales de un solo PV. La VM estaba apagada al inicio/final y la salida de metadatos era igual. **Son controles puntuales, no un hash del disco completo ni una garantía de imagen atómica.** Los errores de sondeo de particiones y rutas se conservan y se interpretan por separado.

Las tres VMs seguían apagadas al terminar la adquisición. VM-BACKUP/hv-datos no estaban entre ellas y mantienen los límites del cierre anterior.

La [entrega anterior](../README.md) permanece como registro histórico; esta ampliación sustituye únicamente sus estados de acceso a estas tres fuentes.
