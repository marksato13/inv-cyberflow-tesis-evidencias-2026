# Análisis de la revisión IJIES · Paper 20267855

**Fuente local:** `C:\Users\markp\Downloads\comment-review-481.docx`.
**SHA-256:** `d11edef6e9e32caa7b66f13b564e55938e33248bfde03c22f3731afca2241bc3`.
**Fecha del análisis:** 23-nov-2026 UTC.

Se leyó el cuerpo completo del Word mediante su XML. El original se conserva intacto y no se publica aquí. Este documento analiza las solicitudes de los revisores; **no es una respuesta enviada, una versión corregida del artículo ni una certificación de cumplimiento**. Salvo el avance específico de R1.3, no se ha vuelto a auditar en esta intervención cada tabla o artefacto de los otros puntos. Las cifras citadas se atribuyen a la revisión, no a nuevos cálculos sobre datos originales.

## 1. Dictamen y lectura principal

Está marcada la opción **“Accept after Major Revision”**. No es rechazo, pero tampoco aceptación definitiva: el editor solicita manuscrito revisado y carta de respuesta para una segunda revisión. El documento no proporciona una fecha límite concreta. La espera de un mes incluida en el formulario se refiere a manuscritos rechazados; no debe aplicarse automáticamente a este dictamen.

Los dos revisores identifican un aporte defendible: **integrar detección/heurísticas con respuesta graduada y distinguir disponibilidad de alerta, instalación, confirmación y efecto observado**. La objeción transversal es que los resultados actuales no establecen todavía el valor operativo adicional de Isolation Forest ni una ventaja general del híbrido.

La revisión no se cierra solo con redacción. Hay cuatro problemas empíricos centrales:

1. **Selección y evaluación sobre datos reutilizados:** si datos y campañas influyen en elegir α, detector o reglas, sus resultados son de desarrollo, no una prueba posterior intacta.
2. **Verdad de referencia insuficiente:** identificar estación y actividad no confirma cada ventana de ataque ni la normalidad de otras ventanas de esa estación.
3. **Aporte del componente aprendido no aislado:** si las heurísticas igualan al híbrido, falta mostrar una ganancia del modelo o limitar la afirmación.
4. **Instalación no equivale a efecto:** comprobar reglas y retirada no demuestra por sí solo cambio de tráfico, y “cero excepciones/5xx inesperados” no demuestra cero fallos de transacción bajo cualquier definición.

## 2. Mapa completo: revisor 1

La numeración R1.1–R1.7 corresponde al orden de los siete párrafos de solicitudes, después del resumen general del revisor.

| ID | Qué pide realmente | Tipo de trabajo y entregable verificable | Riesgo de una respuesta insuficiente |
|---|---|---|---|
| **R1.1** | Reconocer que los datos de referencia/ataque y Campaña A influyeron en elegir detector y α; llamarlos desarrollo. Fijar criterio operativo, congelar detector/umbral/reglas y evaluar después en un bloque intacto. Indicar cuándo y con qué datos se definieron reglas y fast path. | Cronología de decisiones con hashes y fechas; protocolo fijado antes de mirar resultados; manifiesto congelado; evaluación posterior realmente no utilizada. | Cambiar el nombre de “test” no crea independencia. Una campaña posterior en fecha tampoco es intacta si ya se usó para ajustar o seleccionar. |
| **R1.2** | Reconciliar el volumen de datos con seis perfiles y la duración de captura. Explicar las otras entidades/periodos y el orden de todas las exclusiones. | Ledger de filas por fuente temporal, clase de entidad, partición y motivo de exclusión. Cuadrar conjuntos y solapes con las 337 719 filas, las 2 778 de guarda y las 334 941 finales citadas. | Restar exclusiones superpuestas dos veces, mezclar la línea base histórica con otra partición o explicar los conteos únicamente en prosa. La aritmética final no demuestra procedencia. |
| **R1.3** | Verificar las etiquetas mediante registros independientes de episodios; atribuir ventanas por episodio; declarar fronteras de historias retrospectivas; recomputar Tabla 8 excluyendo o resolviendo las 74 referencias no verificadas. | Matriz por clave y fuente; estados de incertidumbre; criterio de frontera predefinido; después, decisión sobre población y recálculo reproducible. | Presentar la auditoría de fuentes como etiquetas confirmadas; escoger offsets por ajuste al patrón; eliminar las 74 y decir que las 84 quedaron validadas. |
| **R1.4** | Comparar con los mismos endpoints y plazos. Separar alertas durante el episodio de las posteriores hasta 60 s. Identificar offline/live y repeticiones HTTP. Separar instalación de regla de confirmación del agente y especificar qué timestamp entra en cada latencia. | Diccionario de eventos/tiempos, una fila por episodio/sistema, conteos durante/después y latencias con origen y destino explícitos. | Ventaja temporal construida con plazos distintos; comparar disponibilidad de alerta de un sistema con confirmación/efecto de otro; mezclar replay y ejecución en vivo. |
| **R1.5** | No sobreinterpretar 567 frente a 570 FP si ambos redondean a 0,87 %. Mostrar cifras sin redondeo o diferencia e incertidumbre. Definir average precision y medir precisión/carga en el punto operativo o región de bajas falsas alarmas fijada de antemano. Aclarar IF-only alerta frente a restricciones. | Estimaciones e incertidumbre que respeten dependencia; definición de AP; métricas operativas en punto elegido; tabla señal→decisión→acción para explicar si IF-only alerta, restringe y bajo qué condiciones. | Llamar superioridad práctica a tres ventanas sin incertidumbre; confundir AP con exactitud/precisión puntual; afirmar que toda alerta IF termina en bloqueo. |
| **R1.6** | Completar reproducibilidad: esquema de características, entornos, parámetros de estimadores, scores/etiquetas/partición por ventana y tiempos emparejados por episodio; scripts de cuantiles/scoring para Tablas 7–13. Explicar LOF fuera de muestra, TLS constante en covarianza y orientación de scores. | Manifiesto y esquema congelados; configuración completa; datos derivados identificados por hashes; ejecución que reconstruya las tablas; documento de decisiones técnicas y datos de acceso controlado si procede. | Considerar suficiente tener un repositorio sin entradas derivadas o sin poder reconstruir las tablas. Los scores publicados no corrigen la incertidumbre semántica de sus etiquetas. |
| **R1.7** | Definir fallo de transacción por perfil y reconciliar la afirmación sobre 21 728 transacciones con “cero excepciones/HTTP 5xx inesperados”. Incluir errores esperados no-5xx, timeouts y retransmisiones. | Definición por perfil y resultado por transacción; ledger de denominadores; resumen/resultado/conclusión usando la misma unidad y definición. | Convertir ausencia de un tipo de error en ausencia de todo fallo. Una retransmisión no es automáticamente un fallo, pero ignorar un timeout tampoco es aceptable. |

### Precisiones sobre R1.5 y R1.6

Average precision debe describirse como suma de la precisión ponderada por los incrementos de recall en los umbrales de ranking. No es precisión puntual ni exactitud global; tampoco debe intercambiarse sin explicación con el área trapezoidal bajo la curva PR. El signo y orientación de todos los scores deben verificarse antes de comparar rankings.

R1.5 pide cuantificar la incertidumbre de la diferencia, no solo añadir más decimales. Al tratarse de ventanas correlacionadas y resultados emparejados, el método no puede elegirse asumiendo independencia sin justificarla. La revisión no fija un único método estadístico: habrá que predefinirlo según la estructura real de captura/episodios.

## 3. Mapa completo: revisor 2

La numeración R2.1–R2.5 corresponde al orden de los cinco párrafos de solicitudes, después de su resumen.

| ID | Qué pide realmente | Tipo de trabajo y entregable verificable | Riesgo de una respuesta insuficiente |
|---|---|---|---|
| **R2.1** | Demostrar el valor adicional aprendido, ya que heurísticas e híbridos dan las mismas 20 detecciones oportunas citadas. Usar escenarios de evaluación independientes y ablación emparejada de heurística, modelo y fast path; distinguir detecciones adicionales, anticipación y carga extra. | Ablación emparejada sobre datos intactos, con escenarios y criterio elegidos de forma independiente; matriz de aportes por componente. | Responder solo con AUROC/AP offline. Mejor ranking no demuestra mejor cobertura/tiempo/carga en el sistema desplegado. No fabricar una campaña favorable tras observar las debilidades. |
| **R2.2** | Usar como principal el denominador de los 25 episodios ejecutados; dejar 20 de 21 elegibles como análisis condicionado secundario. Mostrar fast path aparte bajo el mismo protocolo en sombra. Dar incertidumbre por episodio o bloque y cualificar Wilson por dependencia. | Registro de los 25, razones de elegibilidad sin borrarlos del análisis principal, tablas durante/post, fast path separado e incertidumbre coherente con historias solapadas y repeticiones. | Ocultar los escaneos breves sin cierre de ventana; atribuir al detector por ventanas la rapidez de una ruta distinta; llamar independientes a repeticiones del mismo montaje. |
| **R2.3** | Separar 25 ensayos con despliegue/retirada de los cinco con efecto de tráfico demostrado. Comparar con controles emparejados sin respuesta y predefinir efecto por escenario. Medir impacto legítimo cuando comparta identidad de red con actividad sospechosa. | Diseño y ejecución de controles emparejados; medidas de efecto definidas antes; control de daño colateral con identidad compartida; afirmaciones limitadas mientras tanto. | Decir “25 mitigaciones efectivas” por 25 reglas instaladas. Tampoco puede concluirse que los otros 20 fueron ineficaces: lo informado es falta de efecto demostrado. |
| **R2.4** | Posicionar frente a [11] y [26], diferencias de supervisión/lugar de enforcement/verificación de efecto. Añadir comparador reciente viable o baseline defensivo ajustado con mismos datos, presupuesto de alertas y endpoints. Usar latencias publicadas heterogéneas solo como contexto. | Matriz de contribución frente a trabajos citados; comparador comparable o baseline ajustado con protocolo común; justificación de alcance. | Afirmar ventaja general por ganar a estimadores clásicos o a firmas por defecto; comparar milisegundos de papers con relojes/endpoints distintos. |
| **R2.5** | Mostrar heterogeneidad entre los seis perfiles, periodos independientes, carga legítima realista y deriva. Separar FPR por ventana, proporción de pares marcados y alertas accionables/hora. Atribuir la rapidez a la ruta correcta. | Métricas estratificadas por perfil/periodo y unidad; episodios de decisión accionable por hora según una regla de agrupación fijada; evidencia bajo carga/deriva y límites. | Promedio que oculta un perfil muy afectado; traducir ventanas marcadas directamente a carga del operador o a bloqueos; presentar fast path como latencia de IF. |

## 4. Qué significa el avance de R1.3

La [auditoría inicial](README.md) y la búsqueda ampliada, conservada en una copia privada, ya aportan una respuesta documental importante, pero **R1.3 todavía no está cerrado**.

| Parte de R1.3 | Estado defendible |
|---|---|
| Identificar las 84 positivas y las 74 referencias exactas | Hecho: claves, archivos fuente y hashes verificados. |
| Documentar evidencia y límites por ventana | Hecho: matriz de 158 filas y fuentes con cobertura/límites (copia privada). |
| Tratar historias de 10/30/60 s y fronteras | Se documentaron cruces nominales y mezclas. Falta decidir el tratamiento de evaluación; no se confundieron con atribuciones confirmadas. |
| Atribución independiente concluyente de cada positiva | No conseguida. La coincidencia temporal con bitácora no calibra relojes ni demuestra el par de tráfico por ventana. |
| Normalidad independiente de las 74 referencias | No conseguida. Veinte tienen paquetes/10 s; de las 54 con cero, ocho conservan actividad en contadores de historia examinados. |
| Agotar fuentes históricas pendientes | Parcial: quedan fuentes históricas pendientes según los límites registrados en la copia privada. |
| Recomputar Tabla 8 excluyendo o resolviendo las 74 | No ejecutado, por la instrucción vigente de no cambiar aún métricas ni manuscrito. |

Por tanto, una futura carta debe distinguir **qué evidencia se obtuvo**, **qué no pudo verificarse** y **qué población se usa al recalcular**. No sería correcto afirmar “se verificaron las 84 etiquetas” ni “las 74 eran normales” con lo recuperado.

Si se decide excluir las 74, se fijan sus claves de antemano y se explica que eso no resuelve la incertidumbre de las positivas. El revisor permite excluirlas o resolverlas: no obliga a demostrar normalidad si se adopta y reporta la exclusión. Sí pide que la Tabla 8 refleje esa decisión.

## 5. Trabajo documental frente a nueva evidencia

### A. Se puede empezar sin ejecutar más campañas

- Definir unidades, endpoints y cronología de selección; separar desarrollo de evaluación.
- Preparar el ledger de filas R1.2 y la matriz señal/decisión/acción de R1.5.
- Completar e inventariar la reproducibilidad; probar que los artefactos reconstruyen las tablas.
- Revisar las afirmaciones para no equiparar instalación con efecto, ni 5xx/excepciones con fallo de toda transacción.
- Preparar una carta de respuesta estructurada y registrar sus estados, sin declarar comentarios atendidos antes de verificarlo.

Estas tareas requieren contrastar artefactos; no se consideran hechas por mencionarlas en este análisis.

### B. Requiere reanálisis de datos existentes, si los registros lo permiten

- Denominador principal de 25 episodios y análisis condicionado secundario.
- Conteos durante/después y comparación con el mismo plazo; separación offline/live.
- Precisión/carga en el punto operativo y diferencias con incertidumbre apropiada.
- Recalcular Tabla 8 tras decidir las 74; mantener el alcance de las positivas explícito.
- Estratificación por perfil y agrupación en alertas accionables/hora.

No se han recalculado en esta intervención ni se presupone que todos los timestamps necesarios estén disponibles.

### C. Exige evidencia independiente o nuevos experimentos si los existentes no cumplen

- Bloque posterior intacto con configuración y criterio congelados antes de examinarlo: R1.1.
- Ablación emparejada que aísle modelo, heurísticas y fast path en escenarios independientes: R2.1/R2.2.
- Controles de respuesta deshabilitada y medidas de efecto por escenario; identidad compartida: R2.3.
- Comparador/baseline defensivo bajo protocolo común: R2.4.
- Carga/deriva y periodos independientes si no existen registros que satisfagan R2.5.

No basta renombrar una campaña ya usada como “evaluación independiente”. Tampoco debe seleccionarse un escenario nuevo solo porque favorece al componente aprendido.

## 6. Requisitos editoriales

1. **Manuscrito revisado y carta punto por punto para segunda revisión.** El editor pide marcar modificaciones y adiciones en **fuente roja** dentro del artículo. No equivale a aceptar cambios en silencio ni basta un informe externo.
2. **Conflicts of Interest:** declaración obligatoria, sea la ausencia o la existencia de conflictos. El texto “The authors declare no conflict of interest.” solo procede si los autores confirman que es veraz.
3. **Author Contributions:** párrafo obligatorio con contribuciones individuales. Los nombres/iniciales y roles deben confirmarse con los autores; no asignarlos por inferencia. El formulario sugiere conceptualización, metodología, software, validación, análisis, investigación, recursos, curación de datos, redacción, visualización, supervisión, administración y financiación, según corresponda.
4. **Publicación del código:** el editor la fomenta y describe contenido esperado: entrenamiento/evaluación, preprocesamiento, hiperparámetros, semillas y README. Modelos e instrucciones de preparación de datos se recomiendan si es posible. Tener repositorios públicos es un avance, pero falta demostrar correspondencia exacta con las tablas y citar la versión adecuada.

El requerimiento de reproducibilidad de R1.6 es más concreto que la invitación editorial a publicar código. En la respuesta conviene dar URLs permanentes por commit/release y un procedimiento comprobado, no solo el enlace a `main`.

## 7. Orden recomendado de resolución

1. **Definir el alcance científico y la evaluación intacta:** R1.1 y R2.1. Determina qué puede afirmarse y qué evidencia debe reservarse.
2. **Cerrar contabilidad y política de etiquetas/fronteras:** R1.2 y R1.3, antes de presentar métricas corregidas.
3. **Unificar eventos, plazos y denominadores:** R1.4 y R2.2. Sin esto, las comparaciones temporales no son equivalentes.
4. **Completar la evidencia del efecto y daño colateral:** R2.3 y R1.7.
5. **Cuantificar aporte, carga, incertidumbre y comparación defensiva:** R1.5, R2.4 y R2.5.
6. **Cerrar reproducibilidad y paquete editorial:** R1.6 y requisitos del editor, con las versiones finales de las tablas y sus fuentes.

Este orden no impide preparar documentación en paralelo; evita escribir conclusiones definitivas antes de decidir población, endpoints e independencia.

## 8. Valoración final

La revisión es exigente, pero concreta. **La contribución que los revisores ya reconocen es de integración y verificación operacional, no una demostración de superioridad universal de Isolation Forest o del híbrido.**

Para sostener la contribución actualmente enfatizada, hace falta evidencia de valor añadido del componente aprendido en una evaluación no utilizada para elegirlo. Si no se consigue, deben limitarse esas afirmaciones y posicionar los resultados como demostración de sistemas, explicando de forma transparente sus límites. Esto no autoriza omitir los experimentos pedidos ni garantiza que el editor acepte sustituirlos por redacción.

El avance R1.3 permite contestar con trazabilidad y honestidad; todavía requiere una decisión sobre las 74 y el recálculo solicitado antes de declararse resuelto. El análisis no cierra por extensión los demás comentarios.
