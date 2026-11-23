# Piloto del 30-sep-2026: revisión del extractor y de la adenda

## Conclusión

Las fuentes nuevas de Sensor2, journal de Kali e historial de comandos amplían la evidencia. **No permiten afirmar todavía que las 84 etiquetas estén verificadas ni que todas las ventanas tengan una explicación confirmada.** El desplazamiento de series y las identificaciones de NTP/E0 deben conservarse como hipótesis, no como verdad de referencia.

Esta revisión no cambia el informe inicial ni la adenda de Claude, ni etiquetas, métricas, CSV fuente o modelos. Se escriben únicamente copias y resultados de auditoría en esta carpeta local. No se cambia el remoto de Git ni se interviene en servicios, relojes, capturas o hipervisores.

## 1. Entrega de fuentes y preservación

Se recuperó **el CSV original completo**, no solo sus claves:

`C:\Users\markp\Downloads\CRUCE-PILOTO-2026-09-30-REVISION-EXTRACTOR\ataques-nota-M.csv`

SHA-256 exacto: `4affd07fff45f97c91eb531f208d76df0410fbb602ea1b13847336dfb0e2803e`. Contiene 84 claves únicas. La ausencia de una columna de etiqueta en ese CSV no se remedia editándolo: la pertenencia al conjunto positivo es parte de la procedencia histórica, y el cruce la registra como original sin cambiarla.

Las dos bitácoras completas se copiaron desde Kali y verificaron contra sus hashes previamente observados:

- `ataques-cyberflow-20260929-193105.txt`: `fe5bdadca0c88e10c50c0f601c12aac43faf3c13c7e18ae177765cf4a2882fd1`.
- `ataques2-20260929-202246.txt`: `7609f3d1c625e5b2fa30dd842025a81b950d5f2fae70911366d5ede5b0f84971`.

Se preservaron las salidas existentes de la búsqueda de Claude desde el bastión: `sensor1.txt` e `inventario-pcap-eve-sensor1.txt`. El inventario tiene alrededor de 38 mil líneas y recoge archivos candidatos; no equivale a haber inspeccionado cada evento ni cada paquete. Las fechas de modificación no certifican la fecha del contenido.

Se conservaron además, con prefijo `antecedente-`, las series, journal, historial y scripts que usó la adenda. Son copias de evidencia ya extraída por Claude, no nuevas mediciones independientes de sus fuentes remotas. El manifiesto diferencia estas procedencias.

## 2. Extractor histórico candidato

Se recuperaron de Git estos archivos del commit **`83599c9e000746af5ba3647ae89ec39e6fb026f5`**, fechado antes del piloto:

| Archivo | SHA-256 |
|---|---|
| extract_multilayer_v2.py | `cbd9251b9bd75899f313332b594878ba939f5bc96c1b6bdbf75824dbcf01af83` |
| extract_multilayer_v3.py | `5adbfbf4845554430a3e8b5a4c00eb1e45c2216cae73532da73d9d3f4898fa7f` |
| acumular_v3.py | `9c35b831453d2107769ca1fe610782b6821b2f164df1f369b3e6b3ef36c41dc8` |

**Corroboración adicional:** los tres hashes coinciden con las copias encontradas en Sensor2 bajo `/home/m4rk/cyberflow.k-20260929/scripts/features/`. Esto fortalece su condición de candidato histórico. La fecha del directorio y sus mtimes no prueban por sí solos qué proceso ejecutó Sensor1 en cada momento del piloto. No se acredita la identidad del ejecutable histórico de Sensor1 sin una referencia o hash contemporáneo.

En Sensor1, la copia actual de v2 tiene otro hash (`7130edae…`); no se usa retrospectivamente como si fuera el código exacto del 30-sep. Los hashes consultados se guardan en `sensor1-codigo-hashes.jsonl` y `sensor2-codigo-hashes.jsonl`.

### Asignación de tiempos

En el candidato histórico v2:

- `parse_eve_timestamp` interpreta la marca del evento; los paquetes conservan la marca leída del PCAP.
- `in_window`, líneas 495–496: `window_end - seconds < timestamp <= window_end`.
- `build_rows`, líneas 542–545: primera ancla = `(floor(primer_timestamp / paso)+1)*paso`; última = `ceil(último_timestamp / paso)*paso`; se avanza en pasos de 10 s.
- La fila almacena `datetime.fromtimestamp(anchor, timezone.utc).isoformat()` como `window_end_utc`, línea 594.
- La capa v3 construye filas base por v2 y añade las variables L2 con la misma regla retrospectiva. Su presencia no demuestra que L2 entre al IF.

En el acumulador candidato:

- El intervalo EVE solicitado es inicio−90 s a fin+5 s; luego descarta observaciones de aplicación previas al inicio del PCAP (líneas 177–195).
- Recorta bordes con `margen_inicio` y `margen_final` (líneas 197–215).
- No vuelve a almacenar una clave `(entity_ip, window_end_utc)` ya escrita (líneas 218–232).

**No aparece una corrección fija de −16 s o −180 s.** La marca es un cierre de ventana calculado sobre los timestamps de entrada, no una lectura simultánea del reloj de Kali. La discretización de 10 s, el tiempo de eventos L7 y el procesamiento/extracción deben separarse de un posible desfase de reloj.

No se reprodujo el extractor porque no se recuperó el PCAP/EVE original. Tampoco se sostiene que el candidato histórico sea causalmente idéntico al motor corregido después: leer el código y ejecutar un replay son comprobaciones diferentes.

## 3. Qué prueba realmente la evidencia de relojes

### Kali

El journal conservado registra contacto con servidor NTP y sincronización inicial a **00:11:37 UTC** después del reinicio. Esto es evidencia favorable sobre Kali a partir de ese evento, **no una medición del desfase de Sensor1 respecto de Kali** ni una cota simultánea ±2 s para toda la sesión. No debe aplicarse automáticamente a las ventanas anteriores a la sincronización/reinicio.

### Sensor2

La copia `sensor2-tiempo.txt` muestra el arranque y mensajes iniciales de conectividad. **La falta de un mensaje de sincronización en ese extracto no prueba por sí sola que nunca sincronizara durante todo el arranque.** Hace falta el historial completo pertinente o una medida independiente.

La serie de Sensor2 es una fuente derivada adicional que puede corroborar patrones. Como usa el mismo SPAN y un extractor emparentado, no sustituye al tráfico crudo ni confirma independientemente la semántica ataque/normal.

### Alineación Sensor1–Sensor2 recalculada

Se exploraron desplazamientos de −600 a +600 s en pasos de 10 s sobre las filas activas conservadas. El mejor de los examinados fue sumar **180 s a Sensor2**:

- **80/84** claves activas de Sensor1 encuentran una clave activa de Sensor2.
- **37/84** tienen idéntico `packet_count_10s` bajo ese desplazamiento.
- Los siguientes desplazamientos también producen coincidencias parciales: +190 s, 74 claves; +170 s, 67 claves.

El CSV `comparacion-sensores.csv` deja vacías las parejas activas ausentes, no las convierte en cero de referencia.

**Error de interpretación en la adenda:** su `cruce.py` carga todas las filas de Sensor2, incluso las que tienen cero paquetes, y llama «pareja» a la mera existencia de una fila en la marca desplazada. Por eso dice 84 parejas aunque algunas son inactivas. Además, `desfase.py` explora solo −60 a +60 s: no demuestra el +180 que luego el cruce fija como constante. La revisión amplía explícitamente la rejilla, pero el mejor ajuste sigue siendo exploratorio, no calibración de relojes.

## 4. Hipótesis Sensor1–Kali de −16 s

La adenda fija `OFF=-16` y tolerancia de 2 s dentro del script. No incorpora una tabla reproducible de mediciones independientes que produzca la estimación y su incertidumbre.

Los bordes de HTTP y DNS son **compatibles con un problema de alineación temporal**. Eso justifica investigarlo, no desplazar las bitácoras y considerar automáticamente confirmadas las ventanas resultantes. Estimar el offset con los mismos eventos que se desean justificar no es verificación independiente de esas etiquetas.

Se entrega una sensibilidad explícita (no elección de offset):

| Segundos añadidos a las bitácoras para compararlas con Sensor1 | Cierre dentro de intervalo | Solo historia hasta 60 s | Sin intervalo documentado |
|---:|---:|---:|---:|
| 0, sin corrección | 47 | 4 | 33 |
| −18, hipótesis | 48 | 7 | 29 |
| −16, hipótesis | 50 | 5 | 29 |
| −14, hipótesis | 48 | 5 | 31 |

Se utilizan únicamente los seis intervalos con bitácora, sin crear E0 ni asignar NTP. El criterio principal de esta tabla es **cierre dentro del intervalo**, mientras la adenda utiliza intersección del intervalo de 10 s con el episodio más una tolerancia. Sus conteos no son directamente intercambiables. La revisión mantiene y describe ambas diferencias en vez de aparentar una contradicción aritmética.

`cruce-84-sin-correccion.csv` conserva las 84 filas y cruces de inicio/fin para 10, 30 y 60 s sin desplazar relojes. `sensibilidad-offsets.csv` contiene las cuatro hipótesis sobre las mismas 84 claves (336 registros). Todas conservan `etiqueta_original_sin_cambiar=1` y `estado_verificacion=incierta_atribucion_no_confirmada_con_trafico_crudo`. No se atribuye esa etiqueta original a una verdad confirmada por esta revisión.

## 5. Actividad 00:12–00:15 (E0)

Sí existe una entrada de historial de Kali para `nmap -sT -sV -T4 -p- --open 10.10.30.10`, y el CSV muestra una rampa de SYN seguida de actividad HTTP. Eso fortalece la hipótesis de un escaneo manual con detección de servicios.

**No están demostrados sus instantes exactos:**

- El historial exportado no conserva un timestamp de ejecución de esa línea.
- El inicio de una sesión SSH a las 00:12:18 no es el lanzamiento de nmap.
- El mismo historial contiene otros comandos de configuración y pruebas; su secuencia no fija cuánto tardó el operador antes de lanzar cada uno.
- El fin asignado en la adenda se reconstruye de la propia serie. No es una marca independiente de fin del generador.

Por ello **E0 puede conservarse como episodio candidato/reconstruido, no como episodio documentado con inicio/fin certificados**. Las ventanas de esa actividad no se convierten en normales ni en ataques confirmados por este informe.

## 6. NTP y SSH de gestión

La sincronización de Kali y la periodicidad de algunas ventanas de dos paquetes hacen plausible la explicación NTP. Sin embargo, los agregados exportados no incluyen par origen/destino, puerto UDP/123 ni identificación de proceso por paquete.

**No es correcto llamar a las seis ventanas tráfico legítimo confirmado solo por el patrón temporal.** El script anterior las asigna por una lista manual de horas, no por inspección de protocolo. Quedan como candidatas a NTP, pendientes de evidencia. Si en el futuro se confirman como normales, su reclasificación requerirá decisión separada y sus predicciones determinarán si son FP/TN: no toda ventana NTP sería automáticamente un falso positivo.

Lo mismo aplica a las dos ventanas candidatas a SSH: una sesión contemporánea es corroboración contextual, no prueba de que todos los paquetes y todas las retrospectivas de esa fila sean gestión.

## 7. Solicitudes anteriores a los inicios documentados

- **00:33:40 Sensor1:** 382 solicitudes HTTP/60 s antes del inicio de Nikto anotado a 00:33:46 en Kali.
- **01:25:40 Sensor1:** 710 solicitudes HTTP/60 s y 15 912 paquetes/10 s antes del inicio de la fuerza bruta anotado a 01:25:46.
- **01:28:40 Sensor1:** tres consultas DNS/60 s antes del inicio de DNS anotado a 01:28:48.

Un offset hipotético puede alinear estos bordes, pero aún falta distinguir desfase de reloj, semántica del timestamp L7, versión de procesamiento y otra actividad concurrente. No se cambia el timestamp original ni se selecciona un corrimiento porque maximice las coincidencias.

## 8. Nikto: dos discrepancias diferentes

La primera bitácora registra:

| Marca del primer Nikto | UTC del 30-sep |
|---|---|
| Lanzamiento por el script | 00:33:46 |
| Inicio interno de Nikto | 00:33:47 |
| Fin interno del escaneo | 00:35:48 |
| FIN del script, después de una pregunta interactiva | 01:22:37 |

**El intervalo de 46 min 49 s entre el fin interno y el FIN del script permanece.** Ocurre dentro de la misma bitácora/reloj; restar 16 s a la comparación con Sensor1 no lo elimina. La adenda confunde este hecho con la discrepancia entre horas de la herramienta y la serie agregada. Deben documentarse por separado.

El segundo Nikto tiene `rc=124` a los 150 s. En los agregados de Kali se observan pocos paquetes al comienzo y cero HTTP en las filas citadas. Eso **no permite demostrar la causa del timeout, que la conexión fue exactamente la de Nikto ni silencio absoluto de toda la red**, porque no hay PCAP/EVE recuperado y los agregados tienen cobertura y atribución limitadas. No se excluye E3 como decisión automática.

## 9. Qué falta realmente y qué no debe hacerse todavía

Continúan sin recuperarse el PCAP/EVE del piloto o logs del destino con operaciones y tiempos que permitan resolver la atribución. Los inventarios conservados documentan búsquedas realizadas; no convierten «no encontrado» en «no existe en ningún lugar».

Otras fuentes siguen siendo rutas potenciales no cerradas. Primero hay que resolver identidad/versión histórica y temporalidad con evidencia directa si está disponible.

Hasta entonces:

1. Mantener el conjunto original intacto e identificarlo como etiquetado histórico por regla de estación/intervalo.
2. Conservar E0, NTP, SSH y offsets como hipótesis con su fuerza de evidencia.
3. No escoger etiquetas o offsets según las predicciones del IF ni según la métrica que mejor quede.
4. No recalcular métricas definitivas ni sustituir la tabla publicada a partir de esta auditoría.
5. Cualquier análisis de sensibilidad de etiquetas debe fijar reglas explícitas y reportarse aparte, sin confundirlo con validación independiente.

## 10. Reproducción y archivos

Código de esta revisión:
`C:\Users\markp\Downloads\CRUCE-PILOTO-2026-09-30-REVISION-EXTRACTOR\verificar_cruce.py`

Requiere Python estándar, Git y acceso SSH existente por bastión. No importa ni ejecuta extractores, no carga modelos y no hace inferencia. Solo copia fuentes por lectura y escribe este paquete local. Antes de repetir, tener en cuenta que fuentes operativas cambiantes pueden variar: el script rechaza sobrescribir salidas distintas, salvo el manifiesto. No deben confundirse nuevas lecturas con el corte original.

Resultados principales:

- `ataques-nota-M.csv`: CSV original completo de las 84 ventanas.
- `cruce-84-sin-correccion.csv`: cruce con bitácoras y retrospectivas, 84 registros.
- `sensibilidad-offsets.csv`: cuatro hipótesis explícitas, 336 registros.
- `comparacion-sensores.csv` y `alineacion-series.csv`: alineación exploratoria y parejas activas.
- `historico-*.py`: copias inmutables de los tres archivos del candidato histórico, sin ejecutar.
- `antecedente-cruce.py` y `antecedente-desfase.py`: scripts de la adenda conservados sin alteración.
- `inventario-pcap-eve-sensor1.txt`, `sensor1.txt`: búsqueda de Claude recuperada del bastión.
- `antecedente-kali-journal-piloto.txt`, `antecedente-kali-zsh-50-140.txt`, `antecedente-sensor2-tiempo.txt`, `antecedente-s1-piloto.csv`, `antecedente-s2-piloto.csv`: fuentes nuevas de la adenda conservadas.
- Las dos bitácoras completas, `VERIFICACION.json` y `MANIFIESTO.json` con hashes.

**Estado final:** evidencia ampliada y adenda contrastada, pero etiquetas aún no confirmadas exhaustivamente. No se cambió el informe inicial, la adenda, los modelos, el remoto Git, los servicios ni las métricas. El paquete queda local, sin commit ni push.
