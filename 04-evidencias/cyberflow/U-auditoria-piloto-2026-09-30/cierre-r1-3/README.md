# R1.3 · Cierre acotado de búsqueda de evidencia y matriz de 158 ventanas

**Corte:** 23-nov-2026 UTC. Complementa la [nota U](../README.md).

## Resultado y alcance

Se examinaron exclusivamente los respaldos pendientes identificados y los registros históricos accesibles de SRV-DMZ, pfSense A/B y monitorización. **No se recuperó evidencia suficiente para certificar por ventana las 84 positivas históricas ni la normalidad de las 74 referencias.** Todas las atribuciones individuales quedan inciertas; no significa que no ocurrieran ataques.

Se cierra esta búsqueda hasta los límites documentados, **no la validación empírica de R1.3**:

- SRV-DMZ: 15 logs pertinentes leídos, sin eventos del periodo; las rotaciones más antiguas inspeccionadas y el journal comienzan el 25-oct.
- pfSense A/B: 75 archivos leídos. Hay NTP/DHCP/resolución de septiembre, pero no tráfico del piloto en las rotaciones conservadas de `filter.log`.
- Wazuh: SSH denegado; archives, alerts e índices **no inspeccionados**. Figura actualmente como receptor syslog de ambos firewalls.
- Zabbix: API 7.0.32 alcanzable, sin token preconfigurado; historia autenticada **no consultada**.
- VM-BACKUP está en hv-backup, no en hv-datos. Se revisó el inventario del host, pero el catálogo y los respaldos dentro del invitado **no se inspeccionaron por falta de acceso**. hv-datos mantiene SSH rechazado.

La matriz conserva etiquetas originales y compara timestamps nominales, sin elegir offsets ni etiquetas a partir de predicciones. **No se cambió el manuscrito ni se recalculó Tabla 8.** Excluir las 74 permitiría una sensibilidad posterior, pero no resolvería por sí solo la atribución de las 84 positivas.

## Archivos publicados

| Archivo | Contenido |
|---|---|
| [Informe de cierre](INFORME-CIERRE-R1.3.md) | Fuentes, cobertura, fallos de acceso, reglas de frontera y evaluación sostenible |
| [Matriz completa CSV](MATRIZ-158.csv) | 158 claves, etiquetas históricas, evidencia y límites por ventana; historias 10/30/60 s |
| [Matriz legible](MATRIZ-158.md) | Vista resumida de las 158 filas |
| [Resumen por episodio](EPISODIOS-RESUMEN.csv) | Coincidencias temporales nominales; no métricas del detector ni etiquetas confirmadas |
| [Fuentes y límites](FUENTES-Y-LIMITES.csv) | Estado de las ocho fuentes/grupos pendientes |
| [Cobertura de logs](COBERTURA-LOGS.csv) | Los 90 archivos de logs leídos, hashes, extremos y conteos |
| [Paquete completo](paquete-completo-r1-3.zip) | 42 archivos originales, incluidos scripts, 27 capturas, fuentes empaquetadas y manifiesto |

El ZIP preserva los bytes de la entrega local y sus 41 hashes. Las seis copias legibles se contrastaron con los archivos del paquete. Después de extraerlo, `validar_entrega.py` comprueba claves, soportes temporales, poblaciones y hashes sin acceder al laboratorio ni cargar modelos.

**SHA-256 del ZIP:** `e8cc0a5ece98da0df6a928e5f4b7b61016460e277bdfa0a47a784978ab8f719e` (194 971 bytes).

**Procedencia:** el informe y el manifiesto originales registran el estado al cerrar la entrega local, anterior a este commit. Sus frases sobre «sin push» describen ese corte, no el estado de esta publicación. Se conservan sin reescribirlos para mantener su integridad. Las rutas absolutas corresponden al portátil de preparación; los enlaces de esta página sirven para consultar o descargar la entrega desde GitHub.
