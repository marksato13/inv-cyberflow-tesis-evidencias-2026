# R1.3 · Cierre acotado de búsqueda y matriz de evidencia

**Corte de adquisición:** 23-nov-2026 UTC. **Estado:** entrega local para decisión; no modifica manuscrito, etiquetas ni métricas.

## 1. Conclusión ejecutiva

La búsqueda definida se ha ejecutado hasta los límites de acceso documentados. **No se recuperó evidencia suficiente para certificar por ventana las 84 positivas históricas ni la normalidad de las 74 referencias de la misma estación.** Esto no significa que no ocurrieran ataques ni que todas las referencias sean anómalas: significa que su verdad de referencia individual no quedó demostrada con las fuentes recuperadas.

Se obtuvo evidencia contextual de NTP/DHCP/resolución en pfSense y cobertura explícita de los registros inspeccionados. No aparecieron registros conservados de tráfico del piloto en los `filter.log` accesibles ni registros del periodo en los logs inspeccionados del destino. **Los archivos e índices de Wazuh, la historia autenticada de Zabbix y el contenido de los respaldos dentro de VM-BACKUP siguen sin inspeccionarse por falta de acceso.** No se declara agotada universalmente la evidencia del laboratorio.

**Cierre de búsqueda no equivale a cierre empírico de R1.3.** La evidencia permite sostener trazabilidad de archivos, ejecución documentada de episodios desde el generador y un cruce temporal nominal. No convierte el etiquetado original en una evaluación independiente por ventana.

## 2. Alcance ejecutado y criterio de parada

Se siguió `ALCANCE.md`: respaldos pendientes identificados, SRV-DMZ, Wazuh, Zabbix y pfSense A/B. Se reutilizaron las bitácoras y recortes de sensores ya conservados; no se repitió una búsqueda general de PCAP en los sensores, el portátil o nuevos hipervisores.

La única consulta nueva a Sensor1 fue leer el oro original para identificar exactamente las 74 referencias y sus características. No se consultaron predicciones ni puntuaciones para esta matriz.

No se habilitó SSH, instaló govc, restauró una VM, montó un VMDK, alteró la retención, cambió un reloj o ejecutó un generador. Las consultas autenticadas y sudo producen su auditoría ordinaria. No se intentaron contraseñas interactivas ni se exportaron claves privadas o tokens.

La herramienta govc no estaba instalada/configurada. Se reutilizó el SSH de lectura ya disponible para hv-backup. No se inició una configuración nueva del hipervisor.

## 3. Poblaciones y procedencia

| Población | Identificación | Resultado |
|---|---|---|
| Positivas históricas | `ataques-nota-M.csv`; estación `10.10.20.30`, campaña `piloto-con-dns` | 84 claves únicas; cierres del 30-sep, 00:10:10–01:52:50 UTC |
| Referencias cuestionadas | Selección por `entity_ip=10.10.20.30` dentro del `oro.csv` original | 74 claves únicas; cierres del **27-sep 21:04:30 al 29-sep 22:34:30 UTC** |
| Identidad del oro | SHA-256 completo antes de seleccionar filas | `08604ae3754912a17f1e2895fa199b9758bbc0e91a7e9018070953e2f71b6ac1`; 65 421 filas fuente |
| Identidad de las positivas | SHA-256 del CSV completo | `4affd07fff45f97c91eb531f208d76df0410fbb602ea1b13847336dfb0e2803e` |

Las 74 referencias no son una cola de la sesión del 30-sep. Por ello se cubrieron en la búsqueda sus fechas reales, además del piloto. **20/74 tienen paquetes en los últimos 10 s y 54/74 tienen cero en ese contador. Ocho de esas 54 conservan valores no nulos en alguno de los contadores examinados de historia: intentos de flujo/30 s, HTTP/60 s o DNS/60 s.** No se puede usar `packet_count_10s=0` como verificación de normalidad.

Los IDs de la matriz son `P001`–`P084` y `R001`–`R074`, ordenados por cierre dentro de cada población. Cada fila conserva campaña, IP, cierre y etiqueta histórica. No se ha creado una nueva etiqueta para evaluar detectores.

## 4. Resultado por fuente pendiente

### 4.1 SRV-DMZ: destino de las operaciones

- Se inventariaron `/var/log`, `/var/backups`, `/srv`, `/opt` y el home del administrador con profundidad limitada explícita; no se recorrió universalmente todo el sistema de archivos.
- Se leyeron completos **15 archivos de logs pertinentes**, incluidas rotaciones comprimidas: **2 607 793 líneas** en la lectura final por fecha de evento, sin errores ni truncamiento registrado.
- No se encontraron marcas de evento del 27–30 de septiembre en esos archivos.
- Los extremos de las rotaciones más antiguas de `auth.log` y `syslog` inspeccionadas comienzan el **25-oct-2026**. El listado de boots del journal también comienza el 25-oct.
- La consulta al journal del periodo, tanto general como de sincronización, devolvió `-- No entries --`.
- La configuración **actual** de `lab-web-http.service` y `lab-web-https.service` tiene `StandardOutput=null` y `StandardError=journal`. No demuestra cómo estaban configurados el 30-sep ni prueba que nunca se escribieran otros logs.

**Control contra falsa evidencia:** una búsqueda inicial encontró fechas de septiembre dentro del texto de nuestros comandos sudo, registrados en noviembre. Se preservó esa captura (`17-...json`) y se repitió la clasificación por la fecha al inicio del evento (`27-...json`). Esas menciones se excluyeron: no son operaciones del piloto.

No se recuperaron aquí solicitudes HTTP con hora, cliente y destino del piloto que permitan resolver E0, la actividad de Nikto o la atribución por ventana.

### 4.2 pfSense A y B: tráfico y sincronización

Se leyeron completos los archivos accesibles que coincidían con `filter.log*`, `system.log*`, `ntpd.log*`, `nginx.log*`, `resolver.log*` y `dhcpd.log*`:

| Equipo | Archivos | Líneas leídas | Coincidencias candidatas de fechas 27–30 sep |
|---|---:|---:|---:|
| pfSense-A | 41 | 172 637 | 1 446 |
| pfSense-B | 34 | 156 315 | 382 |

Las coincidencias candidatas no son todas tráfico de la estación. Se examinaron además IPs del piloto y mensajes de reloj/sincronización; las coincidencias relevantes recuperadas fueron **136 mensajes NTP en A y 142 en B**, no pares de flujo por ventana.

- Los `filter.log` y sus rotaciones conservadas corresponden al **22-nov**. No se recuperó ahí el tráfico del 27–30 sep.
- A conserva en `ntpd.log` mensajes rotulados `Sep 30` a partir de **08:49:22**, con errores de resolución de pools y solicitudes posteriores. B conserva algunos del 28/29 sep y otros del 30-sep a partir de **08:47:46**. No se usan como marcas UTC del piloto.
- Estos syslogs no incluyen año ni zona horaria por línea. La configuración actual dice `America/Lima`, pero no se recuperó una copia cuyo nombre codificado fuese anterior al fin del piloto para acreditar retrospectivamente esa configuración.
- Solicitar un servidor NTP o informar un error de resolución no es una medición del offset de Sensor1 ni del reloj de Kali. **No se deduce −16 s ni otro desplazamiento de esos mensajes.**
- Hay registros de DHCP/resolución de septiembre, pero en los filtros aplicados no aparecieron operaciones del par `10.10.20.30`/`10.10.30.10` que resuelvan las ventanas.
- El nginx de pfSense corresponde a administración del firewall, no al servidor HTTP de SRV-DMZ. No se usa para atribuir ataques web al destino.
- La configuración actual de ambos equipos nombra a **Wazuh `10.10.60.10` como receptor syslog**. No prueba que todos los logs fueran enviados en septiembre, pero mantiene a Wazuh como fuente potencial no inspeccionada.

Los resultados negativos se limitan a los archivos, filtros y periodos descritos; no equivalen a negar que hubo tráfico ni a demostrar normalidad.

### 4.3 Wazuh

SSH con las cuentas conocidas probadas (`gadmin`, `root`) y la clave disponible devolvió `Permission denied (publickey)`. **No se leyeron archives, alerts ni índices.** El resultado es falta de acceso, no ausencia de eventos. El receptor syslog configurado en los firewalls hace especialmente relevante esta reserva.

### 4.4 Zabbix

- SSH fue denegado con las cuentas conocidas probadas.
- La API es alcanzable y `apiinfo.version` devolvió **7.0.32**.
- Se examinaron como datos, mediante AST y sin ejecutar los scripts, las asignaciones de tres utilidades existentes del bastión: las tres tienen `TOKEN=None`.
- El primer intento de preparar una consulta autenticada falló por ese `NoneType`; se preservó el error. No llegó a consultar historia. La comprobación posterior confirmó explícitamente la ausencia de token preconfigurado.
- No se inició sesión con una contraseña, generó un token ni ejecutó las utilidades existentes, que podrían modificar configuración.

**No se conoce la retención efectiva ni si la historia de septiembre existe.** Tampoco se presume que métricas agregadas o tendencias horarias, si aparecieran, bastarían para clasificar ventanas de 10 s.

### 4.5 Respaldos: hv-backup, VM-BACKUP y hv-datos

- El inventario conservado del 20-nov y el consultado ahora ubican **VM-BACKUP en hv-backup `172.17.25.1`**, no en hv-datos. Se corrige esa suposición de la adenda anterior.
- `guest info` identifica las IPs de VM-BACKUP: `172.17.25.30` y `10.10.60.42`.
- La consulta de snapshots de esa VM no devolvió un árbol. La búsqueda de archivos candidatos, limitada a profundidad 4 en `datastore1` y a las extensiones/nombres registrados en el comando, no devolvió resultados.
- Eso **no inspecciona el contenido de sus VMDK ni el catálogo dentro del invitado**. No demuestra que no haya respaldos del piloto dentro de VM-BACKUP.
- El acceso por management se detuvo por falta de clave de host conocida y se respetó la comprobación estricta; por VLAN60 fue denegada la clave de usuario. No se agregaron claves ni se relajó la comprobación.
- hv-datos `172.17.25.3` siguió devolviendo `Connection refused` para SSH; no se habilitó el servicio.
- El reloj **actual** de hv-backup informó julio de **2024**. Se registra como advertencia sobre sus fechas de metadatos, no como una estimación del desfase histórico de las VMs en septiembre de 2026.

**Estado:** inventario del host de backup realizado; contenido de los respaldos del invitado aún no inspeccionado por falta de acceso.

## 5. Episodios y conteos nominales reproducibles

Se usan solo los seis intervalos documentados en las dos bitácoras, en su hora Kali convertida explícitamente a UTC. E0 no se convierte en episodio certificado: el historial carece de horas de ejecución. Para E2 se usa el fin interno del escaneo, no la espera interactiva de 46 min 49 s hasta el FIN del script.

**La tabla siguiente es un cruce de relojes nominales, sin offset. No es una tabla de etiquetas confirmadas ni de rendimiento.**

| Episodio | Inicio–fin Kali UTC del 30-sep | Cierres dentro | Intersección historia 10 s | Intersección historia 30 s | Intersección historia 60 s | Historia completa 60 s dentro |
|---|---|---:|---:|---:|---:|---:|
| E1 · nmap | 00:31:05–00:32:46 | 10 | 10 | 10 | 11 | 4 |
| E2 · Nikto 1 | 00:33:46–00:35:48 | 12 | 12 | 12 | 12 | 6 |
| E3 · Nikto 2, timeout | 01:22:46–01:25:16 | 0 | 0 | 1 | 4 | 0 |
| E4 · bucle HTTP | 01:25:46–01:27:46 | 12 | 12 | 13 | 14 | 6 |
| E5 · password-spray | 01:28:16–01:28:18 | 0 | 0 | 1 | 4 | 0 |
| E6 · DNS | 01:28:48–01:31:00 | 13 | 13 | 13 | 13 | 7 |

Los conteos de intersección por episodio **no son mutuamente exclusivos**. Hay **7 ventanas** cuya historia nominal de 60 s intersecta dos episodios. No deben sumarse como si fueran ventanas independientes.

En conjunto: **47 cierres dentro**, **4 ventanas solo con intersección de historia hasta 60 s**, **33 sin intervalo documentado**. Las 74 referencias quedan fuera de la fecha de estas bitácoras, lo que no certifica su normalidad.

La salida cliente de E1 informa host/puertos y la de E2 informa servidor/cabeceras; son evidencias de operaciones desde el emisor, independientes de las puntuaciones. No contienen la traza origen/destino y de tiempo necesaria para certificar cada fila del sensor. Para E3, el `http_200` de la línea 2 de la segunda bitácora pertenece al chequeo previo; no prueba solicitudes Nikto sostenidas durante 150 s.

**No se han establecido atribuciones concluyentes por ventana con la evidencia recuperada.** Registrar cero atribuciones concluyentes en el resumen no equivale a afirmar cero ataques. La incertidumbre afecta a la unión temporal y a la atribución, no borra las bitácoras existentes.

## 6. Tratamiento de historias que cruzan fronteras

Para cada fila de cierre `T`, el soporte de la característica es **`(T−W,T]`**, con `W=10,30,60 s`, conforme al extractor histórico candidato. El código candidato no acredita por sí solo el despliegue exacto del piloto ni resuelve la semántica de timestamp de todos los eventos L7.

La matriz registra por separado:

1. Si el cierre cae en el intervalo documentado.
2. Si la historia de cada duración intersecta un episodio.
3. Si el inicio/fin del episodio cae estrictamente dentro de esa historia.
4. Si toda la historia está dentro de un único episodio nominal.
5. Si la historia de 60 s mezcla episodios.

Las igualdades exactas en un extremo no se llaman «frontera interna». La intersección considera el extremo izquierdo abierto y el derecho cerrado de la ventana; los episodios se tratan como intervalos con ambos extremos incluidos a la resolución registrada de sus bitácoras. Una marca de final en el extremo izquierdo no introduce actividad del episodio en la ventana siguiente.

**No se reasigna una ventana fronteriza a un episodio por la clase que prediga el detector.** Tampoco se convierte una historia posterior al fin de un episodio en normal porque el generador ya terminó. Las características pueden conservar actividad previa. Una futura evaluación por episodio tendrá que fijar de antemano si excluye ventanas mixtas o evalúa por separado interior/frontera; incluso las interiores nominales siguen dependiendo de la incertidumbre entre relojes.

## 7. Qué evaluación se puede sostener y qué decisión queda pendiente

### Sostenible ahora

- Identidad, claves y características originales de las 84 positivas históricas y las 74 referencias cuestionadas.
- Ejecución documentada desde el generador de los seis intervalos indicados, con las salvedades de Nikto.
- Un cruce temporal **exploratorio y nominal**, con ventanas mixtas y límites explícitos.
- Reproducción descriptiva de resultados calculados sobre las **etiquetas históricas**, identificándolas como no verificadas exhaustivamente. Esta entrega no valida las métricas existentes ni las vuelve una prueba independiente.

### No sostenible como conclusión de esta búsqueda

- Que las 84 sean ataques confirmados individualmente o que las 74 sean normales verificadas.
- Que E0 tenga inicio/fin medidos, que las seis ventanas candidatas sean NTP confirmado o que las dos de gestión solo contengan SSH.
- Que −16 s sea una calibración independiente, que Sensor2 sea una evaluación estadísticamente independiente o que la ausencia de un registro pruebe normalidad.
- Que excluir únicamente las 74 resuelva también la incertidumbre de las positivas.

### Decisión necesaria antes de recomputar Tabla 8

1. **Resolver acceso a las fuentes aún inaccesibles** si se desea afirmar que la evidencia histórica está agotada: Wazuh (incluido posible syslog), catálogo/respaldos dentro de VM-BACKUP y consulta histórica autenticada de Zabbix. Las ubicaciones y fallos concretos quedan en el registro; no se deja una búsqueda indefinida.
2. Si se decide cerrar con la evidencia actual, declarar que la verificación independiente por ventana no se consiguió. La normalidad de las 74 queda incierta.
3. En esa situación, una futura sensibilidad debe **excluir exactamente las 74 claves** sin mirar sus predicciones; mantener aparte el resultado histórico y declarar el alcance restante. El conjunto de referencia quedaría con 65 347 filas y, si se conservan las 84 positivas históricas, el total sería 65 431. Son conteos de población, no métricas recalculadas.
4. La comparación con/sin esas 74 no sustituye a la verificación de las 84 positivas. Para una afirmación fuerte de rendimiento independiente se necesitarían etiquetas defendibles o una evaluación independiente con registros adecuados; esta entrega no la ejecuta ni la presupone.

**No se ha recomputado Tabla 8 ni se ha cambiado el manuscrito.** Se entrega la base de evidencia para decidir qué resultado puede mantenerse, cuál debe presentarse como sensibilidad y qué afirmaciones deben limitarse.

## 8. Archivos y verificación

- `MATRIZ-158.csv`: todas las columnas por ventana, procedencia, estado de fuentes, límites, soportes temporales y etiqueta histórica conservada.
- `MATRIZ-158.md`: vista legible de las 158 filas.
- `EPISODIOS-RESUMEN.csv`: conteos nominales por episodio y duración de historia; no son métricas del detector.
- `FUENTES-Y-LIMITES.csv`: cierre de las ocho fuentes/grupos pendientes.
- `COBERTURA-LOGS.csv`: los 90 archivos de logs leídos en la inspección final, con hashes, extremos y conteos. Los archivos vivos se comprobaron por tamaño/mtime durante cada lectura; no se observó cambio en esas capturas.
- `01-...json` a `27-...json`: comandos, cuenta/host, tiempos locales de adquisición, códigos de retorno, stdout/stderr y hash del stdout. Los errores y los primeros intentos se conservan, no se sustituyen por resultados favorables.
- `referencia74-original.csv` y `02-referencia74-original.json`: selección de las 74 y hash del oro completo.
- `EVIDENCIA-FUENTES.zip`: capturas y antecedentes originales pertinentes, incluidas bitácoras, CSV positivo y extractor candidato. No contiene predicciones de detectores.
- `recuperar.py`: código de adquisición de lectura; contiene rutas del entorno original y no se requiere para revisar el paquete sin acceso al laboratorio.
- `construir_matriz.py`: generación local del cruce, sin importar el extractor ni cargar modelos.
- `validar_entrega.py`: verificación local de claves, fórmulas, fuentes y hashes.
- `MANIFIESTO.json` y `ARCHIVOS.md`: integridad y rutas absolutas de toda la entrega.

El manifiesto final y el verificador se generan al cerrar el paquete. El ZIP y sus fuentes se comprueban antes de entregar. La entrega permanece local, sin un nuevo commit ni push.
