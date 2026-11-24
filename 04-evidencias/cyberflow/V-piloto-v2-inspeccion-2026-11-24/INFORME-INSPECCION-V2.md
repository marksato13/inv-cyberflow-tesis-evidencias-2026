# CyberFlow V2: inspección previa al piloto de recolección (24-nov-2026)

**Solo lectura.** No se arrancaron generadores, ataques ni capturas nuevas, y no se cambiaron servicios, reglas, configuraciones ni archivos de V1. Equipos revisados: Clientes, Sensor1, srv-dmz, DC1 (solo su DNS) y el bastión como acceso. No se revisaron componentes de otros proyectos.
Horas en UTC salvo que se indique otra cosa. Convención: **[C]** comprobado, **[P]** propuesta, **[PEND]** pendiente.
Versión pública: las capturas, copias de código y nombres del dominio interno se conservan en una copia privada. El código de los generadores y del servidor está en `producto-main/scripts/laboratorio/`.

## 1. Recorrido de la recolección

| Tramo | Estado | Detalle | Respaldo |
|---|---|---|---|
| Generador | [C] | VM Clientes (`clientesadmin`), interfaz `ens34` con 10.10.20.20 y alias 10.10.20.21–26. Puerta de enlace 10.10.20.1 | `captura-01`: `ip -br addr`, `ip route` |
| HTTP y HTTPS | [C] | srv-dmz 10.10.30.10, `lab-web-http` (:80) y `lab-web-https` (:443), activos desde el 22-nov 14:15:25 | `captura-01`: `systemctl cat lab-web-*`, `ss -ltn` |
| DNS | [C] | DC1 10.10.10.20 (Samba AD), reenviador `10.10.10.1` | `captura-03`: `smb.conf` |
| Espejo (SPAN) | [C] parcial | Sensor1 captura en `ens37` (promiscuo, sin IP). Cada conexión se ve en dos VLAN (24-nov: VLAN 10 y 30) y el extractor quita los duplicados (unos 34 700 por pasada) | `captura-04`; `acumular.log`; prueba del 24-nov |
| Configuración del SPAN en el switch | [PEND] | No es accesible desde los equipos revisados; puertos y VLAN de origen sin documentar | — |
| Gestión de Sensor1 | [C] | `ens34` 10.10.60.11, sin promiscuo | `captura-02` |

## 2. Generadores legítimos

| Elemento | Estado | Detalle | Respaldo |
|---|---|---|---|
| Archivo usado | [C] | `/home/adminclientes/laboratorio/clientes_lab.py`, sha `ae5134a7…cdc3`, 15 357 B, del 27-sep 02:41 | `captura-01`; `producto-main/scripts/laboratorio/` |
| Generador de unidades | [C] | `clientes_config.py`, sha `9aa2e2e0…4534` | ídem |
| Otra copia | [C] | srv-dmz tiene un `clientes_lab.py` más antiguo (19-sep, sha `828fadf7…`), que no se usa | `producto-main/laboratorio/configuracion/srvdmz/` |
| Versionado | [C] | La carpeta no está bajo git en Clientes | `git log` sin salida |
| Unidades | [C] | `cyberflow-lab.target` y seis `cyberflow-lab-<perfil>.service`: `--perfil --origen --servidor 10.10.30.10 --dns 10.10.10.20`, `Restart=always`, `StandardOutput=null`. Ahora `disabled` e inactivas; última parada el 15-nov 21:41:26 | `captura-01` |
| Perfiles e IP | [C] | ofimática .21, navegación .22, descargas .23, aplicación .24, ligero .25, errático .26 | unidades |
| Ritmo base (s entre acciones, con factor 1) | [C] | ofimática 3,0 · navegación 1,5 · descargas 8,0 · aplicación 2,0 · ligero 12,0 · errático 5,0 | `clientes_lab.py:46-78` |
| Pesos | [C] | Por perfil, en `PERFILES[...]["acciones"]` (p. ej. ofimática: web_pequena 50, https 20, dns 25…) | `clientes_lab.py:46-78` |
| Esperas | [C] | Exponenciales: `expovariate(1/(ritmo/factor))`, mínimo 0,05 s | `clientes_lab.py:358` |
| Ráfagas | [C] | Solo el perfil errático: probabilidad 0,02 por acción, ráfaga de 20–90 s con el factor ×6 | `clientes_lab.py:336-343` |
| Dependencia de la hora | [C] | `factor_horario(datetime.now())`: 08–13 → 1,0; 13–15 → 0,55; 15–19 → 0,9; 19–23 → 0,35; resto → 0,12; sábado y domingo ×0,3 salvo `--forzar-laborable` | `clientes_lab.py:84-107, 332` |
| **Zona horaria de Clientes** | [C] | **Etc/UTC** desde el 16-feb-2025. La curva sigue la hora UTC: el pico 08–13 UTC son **03:00–08:00 en Perú** | `/etc/localtime`; `timedatectl` |
| Duración finita | [C] **No** | Bucle `while True`; solo existe `--una-vuelta` (una acción de cada tipo) | `clientes_lab.py:320-331` |
| Semilla | [C] **No** | `random` sin semilla | código |
| Programa por tiempo transcurrido | [C] **No** | El factor sale solo del reloj. `--intensidad` es un multiplicador fijo | `clientes_lab.py:285` |
| Recolección de septiembre | [C] | Un solo arranque de cada perfil del 26-sep al 1-oct, sin errores en el journal. Ofimática se reinició el 27-sep 06:59:02 | `captura-01`: `journalctl` |

## 3. Servicios de destino y resultados esperados

| Acción | Destino | Resultado esperado | Tipo |
|---|---|---|---|
| web_pequena | HTTP `/`, `/estatico/4k`, `/estatico/16k` | 200 | normal |
| web_media | `/estatico/64k`, `128k`, `256k` | 200 | normal |
| web_grande | `/estatico/1024k`, `2048k`, `4096k` (máx. 4096 KB) | 200 | normal |
| https | HTTPS `/`, `/api/ok`, `/estatico/32k` (sin verificar certificado) | 200 | normal |
| api_ok / api_post | `/api/ok` / POST `/api/enviar` (200–8 000 B) | 200 / 201, **o 500 en el 10 % de los POST** | normal / error intencional |
| api_auth / api_404 / api_500 | `/api/auth`, `/api/perdido`, `/api/fallo` | 401 / 404 / 500 | **error intencional** |
| api_lento | `/api/lento` | 200 tras 1,5–4 s (cliente con espera de 10 s) | normal |
| abandono | `/estatico/2048k` cortado sin leer | RST o retransmisión | **error intencional** |
| puerto_cerrado | TCP 8081, 9001, 4444, 31337 | RST (no hay servicio; el firewall de srv-dmz tiene `policy accept`) | **error intencional** |
| ping | ICMP a 10.10.30.10 | respuesta | normal |
| dns | 6 nombres del dominio interno | NOERROR | normal |
| dns_inexistente | nombre aleatorio del dominio interno | NXDOMAIN previsto | **error intencional** |

Fuente: `clientes_lab.py:187-272` y `servidor_lab.py:53-91`. Certificado de srv-dmz válido hasta el 19-sep-2028.

**Fallos imprevistos o sin verificar:**

| Hallazgo | Estado | Detalle | Respaldo |
|---|---|---|---|
| NXDOMAIN que no se produce | [C] | El 29-oct, las **1 328** consultas aleatorias recibieron **NOERROR**. `dns_nxdomain_ratio_60s` no recibe la señal prevista | `captura-03`: EVE `eve.json.1-2026103100.backup` |
| Nombres resueltos fuera del DC1 | [C] | Ninguno de los 6 nombres está en las zonas del DC1 (zona del dominio AD, `lab-ppi` y zonas inversas): se resuelven por el reenviador 10.10.10.1 | `captura-03`: `samba-tool dns zonelist` |
| SERVFAIL ocasional | [C] | 10 de unas 27 000 consultas (29-oct) | ídem |
| Éxitos y fallos por transacción | [PEND] | Imposible de verificar: los contadores van a stdout (`StandardOutput=null`) y el servidor no registra peticiones (`log_message` vacío) | unidades; `servidor_lab.py:98` |
| Eventos TLS 1.3 | [PEND] | En la prueba del 24-nov hubo 0 eventos `tls` en 13 conexiones TLS 1.3 (el flujo sí se identifica como TLS). Python 3.12 con OpenSSL 3.0 negocia TLS 1.3 por defecto | prueba del 24-nov (copia privada) |

## 4. Captura

| Elemento | Estado | Detalle | Respaldo |
|---|---|---|---|
| tcpdump | [C] | `ppi-motor-capture`: `tcpdump -i ens37 -n -s 0 -U -B 65536 -Z tcpdump -w /var/lib/ppi-motor-capture/live-%Y%m%d%H%M%S.pcap -G 5`, sin filtro BPF; drop-ins `etapa4.conf` y `etapa4-primer-pcap.conf` | `captura-03` |
| Anillo de PCAP | [C] | Archivos de 5 s; unos 179 en el anillo, cerca de 15 min (`ventana_s` ≈ 898) | `acumular.log`; `ls` |
| Unidad que borra el anillo | [PEND] | No se identificó por nombre; `retener_minutos` no aparece en la sección `[captura]` leída | `captura-04` |
| Suricata | [C] | 7.0.3, `suricata -D --af-packet -c /etc/suricata/suricata.yaml`, YAML `0962ce68…`, sin reglas, EVE `/var/log/suricata/eve.json` con alert, anomaly, http, dns, tls, flow, stats y otros | `captura-02`, `captura-03` |
| Rotación EVE | [C] | logrotate diario, `maxsize 500M`, `rotate 7`, compresión, `copytruncate`. Hay copias rotadas desde el 5-oct | `/etc/logrotate.d/suricata` |
| Espacio | [C] | `/` 76 G, 24 G libres (68 %); `/var/log/suricata` 3,5 G; CSV V1 2,4 GB | `df`, `du` |
| Medida de pérdidas | [C] | tcpdump «0 packets dropped by kernel»; Suricata `kernel_drops 0`, `errors 0`; `softnet_stat` 0 | `captura-04` |
| `rx_dropped` de `ens37` | [PEND] | 296 204 de 3,64 M (unos 8 %), mientras tcpdump y Suricata no pierden nada. Probablemente sean tramas que la pila descarta tras la captura, pero no está demostrado | `ip -s link show ens37` |

## 5. Extracción

| Elemento | Estado | Detalle | Respaldo |
|---|---|---|---|
| Archivos | [C] | `acumular_v3.py` (`9c35b831…`), `extract_multilayer_v3.py` (`5adbfbf4…`), `extract_multilayer_v2.py` (`7130edae…`): **idénticos** a `producto-main` (`d2c9772`) | comparación SHA-256 |
| Esquema | [C] | `configs/features/multilayer-v2.json`, 28 características en el orden listado abajo; igual al repositorio salvo el fin de línea | `producto-main/configs/features/` |
| Ejecución | [C] | `cyberflow-acumular.timer` `OnCalendar=*:0/10` → `acumular_v3.py --salida …/linea-base/multilayer-v3.csv --campaign-id linea-base`, con las mismas `--excluir` que el motor. Cada pasada tarda unos 115 s | `captura-02`; `acumular.log` |
| Paso | [C] | 10 s, anclado a múltiplos de 10 s | `extract_multilayer_v2.py:525,542`; `v3.py:394` |
| Historias | [C] | 10, 30 y 60 s según el sufijo de cada característica | esquema |
| Deduplicación | [C] | `deduplicar_espejo` quita la doble copia del SPAN (desactivable con `--sin-deduplicar`) | `extract_multilayer_v3.py:179` |
| Inactividad | [C] | Sin datos en la ventana se emite 0,0, nunca se interpola. Una entidad sin tráfico IP pero con tramas de capa 2 genera fila igualmente | `v2.py:14`; `v3.py:417` |
| Arranque | [C] | Se descartan las ventanas de los primeros 60 s de cada pasada y de los últimos 30 s; `eligible_training` exige `history_coverage_s ≥ 60` | `acumular_v3.py:112-116,199`; `v2.py:583-596` |
| Clave única | [C] | `(entity_ip, window_end_utc)`. Cada pasada relee el CSV entero: hoy 5,78 M filas | `acumular_v3.py:85-92` |
| Exclusión del bastión | [C] | Solo quita las filas cuya entidad es el bastión; su tráfico sí cuenta en las filas del destino | prueba del 24-nov |
| Motor | [C] | El `ExecStart` efectivo (el último drop-in) usa `manifest-if-limpio-2026-10-alfa001.json` (α 0,01), con `--step-seconds 10 --history-seconds 230 --disparo eventos` | `captura-02` |

Orden de las 28 características: 1 packet_rate_10s · 2 byte_rate_10s · 3 mean_ip_len_10s · 4 large_ip_ratio_10s · 5 unique_dst_ip_ratio_30s · 6 icmp_ratio_10s · 7 flow_attempt_rate_10s · 8 syn_rate_10s · 9 syn_completion_ratio_10s · 10 rst_ratio_10s · 11 unique_dst_port_ratio_30s · 12 http_error_ratio_60s · 13 dns_nxdomain_ratio_60s · 14 tls_session_rate_60s · 15 ttl_mean_10s · 16 fragment_ratio_10s · 17 protocol_diversity_30s · 18 tcp_retransmission_ratio_10s · 19 flow_duration_mean_30s · 20 tx_rx_byte_ratio_30s · 21 http_request_rate_60s · 22 http_method_entropy_60s · 23 http_auth_failure_ratio_60s · 24 dns_query_rate_60s · 25 unique_dns_name_ratio_60s · 26 tls_handshake_failure_ratio_60s · 27 tls_version_ratio_60s · 28 http_status_5xx_ratio_60s.

## 6. Trazabilidad

| Elemento | Estado | Detalle |
|---|---|---|
| Registro por transacción | [C] **No existe** | Ni en el cliente (stdout a null) ni en el servidor (sin log) |
| Asociación perfil ↔ filas | [C] | Solo por la IP de origen (.21–.26) |
| Asociación sesión ↔ filas | [C] **No existe** | `campaign_id` es siempre `linea-base`; no hay identificador de sesión ni de jornada |
| Captura ↔ filas | [C] parcial | `acumular.log` registra por pasada paquetes, PCAP, filas y duplicados, **sin marca de tiempo** |
| Relojes | [C] | Todos sincronizados: Sensor1 por timesyncd con 10.10.60.1 (desfase +0,8 ms), bastión por chrony con un servidor público (−2,6 ms), y Clientes, srv-dmz y DC1 sincronizados. Clientes, Sensor1 y srv-dmz en UTC; bastión y DC1 en America/Lima |
| Corte del 29-sep 18:15–22:14 | [PEND] | Clientes siguió activo (sin paradas en el journal); el corte está del lado de la captura o el acumulador, y el log no tiene horas para demostrarlo |

## 7. Propuesta de adaptación (no aplicada)

Objetivo: que cualquiera de las siete jornadas (6 h; cinco habituales y dos tranquilas) se pueda ejecutar a cualquier hora, sin depender del reloj ambiental.

1. **[P] Generador V2, sin tocar el de V1.** Copia `clientes_lab_v2.py` con:
   - `--duracion` en segundos;
   - `--semilla` por perfil y jornada;
   - `--programa programa.json` con tramos por tiempo transcurrido (inicio, fin y factor), que sustituya a `factor_horario`;
   - `--jornada` como identificador;
   - un registro JSONL por transacción: hora UTC, jornada, perfil, IP, acción, destino, código o resultado, latencia, bytes, y si el error era intencional.

   Hoy el factor 1,0 es «habitual» y 0,12–0,35 es «tranquilo», pero los valores de cada jornada **están por definir**.
2. **[P] Unidades V2 aparte:** `cyberflow-v2@.service` o un target nuevo, con salida a fichero y sin `Restart=always` al terminar la duración. Las unidades de V1 no se modifican.
3. **[P] Acumulador V2 aparte:** instancia o temporizador nuevo con `--salida …/artifacts/v2/<jornada>.csv --campaign-id v2-<jornada>`, fuera de lo que lee `cyberflow-aprendizaje`.
4. **[P] Decidir qué pasa con el acumulador de V1 durante las jornadas.** Si sigue activo, las jornadas entrarán en `linea-base` y en el lote diario. Opciones: pausar `cyberflow-acumular.timer` (requiere tu sudo) o registrar las ventanas en `excluir_ventanas`.
5. **[P] DNS:** usar para NXDOMAIN un nombre dentro de una zona de la que DC1 sea autoridad, y comprobarlo antes del piloto. Decidir si los 6 nombres siguen resolviéndose por el reenviador.
6. **[P] Manifiesto por jornada:** SHA-256 de generador, programa, unidades, YAML de Suricata, extractor, esquema y modelo; inicio y fin en UTC; desfases NTP; tcpdump y Suricata (drops) al inicio y al final; versión de `producto-main`.
7. **[P] Horas en el log del acumulador** para poder reconstruir cortes como el del 29-sep.

## 8. Requisitos para iniciar el piloto

1. Valores de factor por tramo para las jornadas habituales y tranquilas, fijados por escrito antes de arrancar.
2. Generador V2 con duración, semilla, programa y registro por transacción, probado con `--una-vuelta` y una ejecución corta.
3. Destino separado para el CSV de V2 y decisión sobre el acumulador de V1 (punto 7.4).
4. NXDOMAIN verificado.
5. Causa de la ausencia de eventos TLS 1.3 entendida, o la limitación declarada.
6. `rx_dropped` de `ens37` explicado, o el método de medida de pérdidas fijado (tcpdump + Suricata).
7. Configuración del SPAN documentada: puertos y VLAN de origen.
8. Decisión sobre las 38 IP del laboratorio ajenas a los perfiles (el 58 % de la línea base de V1): si entran en V2 o no. Su tráfico sí depende del reloj real.
9. Pendiente heredado: las 11 filas alteradas por la prueba del 24-nov en `multilayer-v3.csv` (24-nov 00:09:10–00:10:00 y 00:11:30–00:12:10, entidad 10.10.30.10) entran en el lote del aprendizaje del **25-nov a las 00:30 UTC** si no se excluyen.
10. Congelar modelo, α, reglas, vía rápida y criterios antes de la primera jornada de evaluación.

## Evidencia

Las capturas (`captura-00` a `captura-04`) y las copias de los archivos inspeccionados se conservan en una copia privada, porque contienen datos de la infraestructura del laboratorio.
