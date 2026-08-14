# Matriz de costos y decisión de modelos

**Proyecto:** CUI Student Ops AI Hub  
**Entrega:** Ecosistema IA Autónomo para Negocios  
**Estado:** Documento de respaldo para GitHub / entrega final  

---

## 1. Objetivo de la matriz

Esta matriz justifica qué modelo o estrategia conviene usar en cada parte del ecosistema, priorizando una solución funcional, económica y segura.

El objetivo no es usar el modelo más potente para todas las tareas, sino elegir el recurso adecuado según complejidad, costo, riesgo operativo y necesidad de revisión humana.

---

## 2. Criterio general de optimización

El sistema trabaja con consultas ficticias de alumnos. Las tareas principales son clasificación, resumen, priorización, generación de borradores internos y preparación de una salida simulada.

Para el MVP se prioriza:

- Modelo económico para tareas repetitivas.
- Límite de tokens de salida.
- Prompt estructurado.
- Respuesta obligatoria en JSON puro.
- Revisión humana antes de cualquier acción crítica.
- No uso de datos reales de alumnos.
- Escalamiento a modelos más potentes solo cuando el caso lo justifique.

---

## 3. Matriz de decisión por tarea

| Etapa del flujo | Tarea | Modelo / estrategia recomendada | Justificación | Costo relativo | Riesgo operativo | Cuándo escalar |
|---|---|---|---|---|---|---|
| Entrada de consulta | Validar datos mínimos | Filtros de Make + Airtable | No requiere IA. Se resuelve con lógica booleana y campos obligatorios. | Muy bajo | Bajo | No aplica |
| Procesamiento IA | Clasificar tipo de consulta | GPT-4o-mini | Tarea simple y repetitiva. Buen equilibrio entre costo y rendimiento. | Bajo | Medio | Escalar a GPT-4o si hay ambigüedad alta |
| Procesamiento IA | Generar resumen IA | GPT-4o-mini | Las consultas son breves y estructurables. No requiere razonamiento profundo. | Bajo | Medio | Escalar si el texto es largo o confuso |
| Procesamiento IA | Detectar prioridad | GPT-4o-mini + reglas del prompt | Se puede resolver con criterios simples: urgencia, reclamo, falta de datos o derivación humana. | Bajo | Medio | Escalar a revisión humana si hay riesgo institucional |
| Procesamiento IA | Generar respuesta sugerida | GPT-4o-mini | La salida es un borrador interno, no un envío automático. | Bajo | Alto si se enviara sin revisar | Siempre pasa por HITL |
| Estructuración | Convertir salida en campos | JSON Parse | Evita procesar texto libre y permite mapear variables dinámicas. | Muy bajo | Medio si el JSON falla | Reforzar prompt o derivar a error handler |
| Actualización de datos | Guardar resultado en Airtable | Airtable Update Record | No requiere IA. Solo persistencia de datos. | Muy bajo | Bajo | No aplica |
| Trazabilidad | Crear log de ejecución | Airtable Create Record | Permite auditar ejecuciones, errores y acciones tomadas. | Muy bajo | Bajo | No aplica |
| Salida final | Preparar salida simulada | Airtable + Make | Se evita envío real. La salida queda documentada y bloqueada por revisión humana. | Muy bajo | Bajo | En producción: Gmail/Slack/WhatsApp con aprobación |
| Acción crítica | Aprobar salida | HITL humano | Reduce riesgo de respuestas incorrectas, exposición de datos o confirmaciones indebidas. | Variable | Alto | Siempre requiere humano |
| Procesamiento masivo | Procesar histórico de consultas | Batch API / procesamiento por lotes | Recomendable cuando haya alto volumen y no se requiera respuesta inmediata. | Bajo por volumen | Medio | Usar solo para análisis masivo no urgente |
| Casos complejos | Lectura densa o reclamos extensos | GPT-4o / Claude | Útil si el caso requiere mejor comprensión contextual. | Medio / Alto | Alto | Solo en casos especiales |

---

## 4. Matriz de optimización aplicada

| Estrategia | Dónde se aplica | Impacto esperado | Decisión para el proyecto |
|---|---|---|---|
| Usar GPT-4o-mini | Clasificación, resumen, prioridad y borrador | Reduce costo frente a modelos más grandes | Se usa como modelo principal del MVP |
| Limitar Max Output Tokens | Módulo OpenAI de Make | Evita respuestas largas e innecesarias | Se configuró límite de salida en 800 tokens |
| JSON puro obligatorio | Prompt de OpenAI + JSON Parse | Reduce errores de parsing y reprocesamiento | Se reforzó el prompt después del error detectado |
| Variables dinámicas | Prompt con campos de Airtable | Evita hardcodear datos y permite reutilizar el flujo | Se usan campos dinámicos de cada consulta |
| Filtros previos a IA | Router de Make | Evita gastar tokens en registros incompletos | Se valida `Datos mínimos completos` antes de llamar al modelo |
| Logs de ejecución | Tabla `Logs de ejecución` | Permite medir errores y mejorar el sistema | Se registra resultado, módulo afectado y acción tomada |
| HITL | Vista `Aprobados para salida` | Evita acciones críticas sin aprobación | Es obligatorio antes de salida final |
| Batch API | Procesamiento histórico o alto volumen | Puede reducir costos en procesamiento masivo | Recomendado como mejora futura |
| Prompt Caching | Instrucciones repetidas del sistema | Reduce costo si se reutiliza un prompt largo y estable | Recomendado como mejora futura |

---

## 5. Estimación comparativa de volumen

Esta estimación es conceptual y sirve para comparar decisiones de arquitectura. No representa una cotización oficial de proveedores, ya que los precios pueden cambiar.

**Supuesto de trabajo:** 500 consultas procesadas.

| Escenario | Tokens estimados | Lectura |
|---|---:|---|
| Sin optimización | 4.750.000 | Flujo sin control de tokens, sin batching y con instrucciones repetidas completas |
| Con procesamiento por lotes | 2.375.000 | Menor costo relativo para ejecuciones masivas no urgentes |
| Con prompt caching | 1.150.000 | Ahorro por reutilizar instrucciones estables del sistema |
| Batch + prompt caching | 575.000 | Mejor escenario para volumen alto |

**Ahorro estimado del mejor escenario:** aproximadamente 87,9% frente al escenario sin optimización.

---

## 6. Decisión final para el MVP

Para la entrega final se decidió usar **GPT-4o-mini** como modelo principal porque el caso de uso requiere tareas breves, repetitivas y estructuradas: resumir, clasificar, priorizar y generar un borrador interno.

No se justifica usar un modelo más costoso para todos los casos porque:

- Las consultas son cortas.
- El resultado no se envía automáticamente.
- La respuesta queda bloqueada por revisión humana.
- El JSON permite controlar la estructura.
- Los errores pueden detectarse en JSON Parse y registrarse en logs.

Los modelos más potentes se reservan para una mejora futura, por ejemplo:

- Reclamos extensos.
- Casos ambiguos.
- Lectura de documentos largos.
- Necesidad de mayor razonamiento contextual.

---

## 7. Conclusión

La optimización de costos del proyecto no depende solamente de elegir un modelo barato. Se apoya en una arquitectura completa:

- Validar datos antes de llamar a la IA.
- Usar GPT-4o-mini para tareas simples.
- Limitar tokens de salida.
- Exigir JSON puro.
- Registrar errores.
- Usar revisión humana antes de acciones críticas.
- Reservar modelos más potentes y procesamiento por lotes para escenarios específicos.

De esta forma, el ecosistema mantiene equilibrio entre costo, calidad, trazabilidad y seguridad operativa.
