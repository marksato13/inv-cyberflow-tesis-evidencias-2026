# U · Auditoría retrospectiva de las 84 ventanas del piloto

**Piloto:** 30-sep-2026. **Corte de esta auditoría:** 23-nov-2026 UTC.

Complementa la [nota M](../M-deteccion-kali-sensor1-2026-09-30.md) y el [linaje de aprendizaje de la nota S](../S-aprendizaje-supervisado-sensor1-2026-10-11.md). Documenta la trazabilidad del conjunto histórico positivo; **no constituye una nueva evaluación independiente ni modifica sus etiquetas o métricas publicadas**.

## Entrega

### Análisis integral de la revisión IJIES

El [análisis del dictamen del Paper 20267855](ANALISIS-REVISORES-20267855.md) desglosa
las siete solicitudes del revisor 1, las cinco del revisor 2 y los requisitos editoriales.
Distingue aclaraciones, reanálisis y evidencia nueva, y precisa por qué el avance R1.3
todavía no equivale a cerrar la observación. No modifica el Word original ni el manuscrito.

### Actualización R1.3: búsqueda acotada

El material ampliado de esta búsqueda se retiró del repositorio público porque incluía inventarios de infraestructura ajena a CyberFlow; se conserva en una copia privada. Aquí queda el [resumen de episodios](cierre-r1-3/EPISODIOS-RESUMEN.csv).

Las 74 referencias abarcan el 27–29 sep: 20 tienen paquetes/10 s y 54 tienen cero; **8 de esas 54 conservan actividad en contadores de historia de 30/60 s**. Hay 7 ventanas positivas cuya historia nominal de 60 s intersecta dos episodios. La atribución individual sigue incierta. Quedan fuentes históricas sin consultar y no se declara ausencia de evidencia en ellas.

Esta actualización no cambia etiquetas, métricas ni manuscrito y no recalcula Tabla 8. Cierra la búsqueda ejecutada hasta sus límites de acceso, no la validación independiente de las etiquetas.

### Entrega inicial: extractor y 84 ventanas

- [Informe crítico completo](REVISION-CRITICA.md): fuentes, extractor candidato, hipótesis temporales y límites de atribución.
- [Paquete de evidencia](paquete-evidencia.zip): 27 archivos, incluidos el CSV original, dos bitácoras completas, inventario recuperado, copias de antecedentes, código histórico, cruces y manifiesto.

El ZIP preserva los bytes originales: evita que la normalización CRLF/LF de Git cambie los hashes del CSV y las bitácoras. La copia legible del informe reproduce el documento del paquete. Las rutas absolutas que aparecen en él documentan su preparación en el portátil; **no son enlaces de descarga ni requisitos para consultar las evidencias del ZIP**.

**SHA-256 del ZIP:** `8800669874b7551ea4699838924d5ade5e746e16ef0aee3cef71ea95572eb005` (441 562 bytes). Se verificaron sus 27 archivos contra los originales y los 26 hashes contenidos en el manifiesto.

## Resultado de la revisión

| Comprobación | Resultado y alcance |
|---|---|
| CSV original | 84 claves únicas; SHA-256 `4affd07fff45f97c91eb531f208d76df0410fbb602ea1b13847336dfb0e2803e` |
| Cruce sin desplazamiento | 47 cierres dentro de intervalos documentados; 4 coincidencias solo por historia retrospectiva hasta 60 s; 33 sin intervalo documentado. No equivalen a 47 ataques confirmados y 37 ventanas falsas. |
| Sensor2, desplazamiento exploratorio +180 s | 80/84 parejas activas; 37 conteos de paquetes idénticos. La existencia de una fila con cero paquetes no confirma actividad. |
| Código histórico candidato | Tres archivos del commit `83599c9e000746af5ba3647ae89ec39e6fb026f5` coinciden con copias conservadas en Sensor2. No acredita la versión ejecutada por Sensor1 durante el piloto. |
| Desfase Sensor1–Kali de −16 s | Hipótesis no medida de manera independiente. Se entrega sensibilidad para 0, −18, −16 y −14 s: 336 registros. |
| E0, NTP y SSH | Explicaciones candidatas, no atribuciones confirmadas por paquetes. |
| Primer Nikto | Persisten 46 min 49 s entre su fin interno y el FIN del script; no se elimina con una corrección entre relojes. |

**Conclusión:** las 84 filas pertenecen al conjunto etiquetado históricamente por una regla de actividad de la estación/intervalo. Su correspondencia exhaustiva con episodios de ataque sigue pendiente. No se ha recuperado el PCAP/EVE original del piloto; «no localizado en las búsquedas realizadas» no significa inexistencia universal.

El código candidato procede del [repositorio del producto](https://github.com/marksato13/inv-cyberflow-sistema/tree/83599c9e000746af5ba3647ae89ec39e6fb026f5/scripts/features). La presente auditoría y sus evidencias corresponden al repositorio documental.

## Verificación local

Después de descargar y extraer el ZIP, `validar_paquete.py` comprueba de forma exclusivamente local los 26 hashes del manifiesto, las 84 claves originales, las 84 filas del cruce, las 336 de sensibilidad y los conteos de comparación. Requiere únicamente Python estándar; no ejecuta extractores, ataques ni consultas remotas.

`verificar_cruce.py` es el script de adquisición/análisis conservado tal como se usó. **No debe confundirse con el verificador local**: contiene rutas del entorno original y consultas SSH de lectura que requieren ese acceso. Los `antecedente-*.py` y `historico-*.py` se conservan como fuentes; no se ejecutaron para producir esta publicación.

El paquete no contiene claves privadas ni el artículo inédito. Los registros incluyen direcciones privadas y rutas del laboratorio, como evidencia técnica del piloto.
