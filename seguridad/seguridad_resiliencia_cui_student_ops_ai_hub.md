# Seguridad y resiliencia — CUI Student Ops AI Hub

## 1. Objetivo del documento

Este documento describe las decisiones de seguridad, privacidad y resiliencia aplicadas al proyecto **CUI Student Ops AI Hub**, un ecosistema de automatización con inteligencia artificial para asistir procesos de atención y operación de consultas de alumnos.

El objetivo es demostrar que el sistema no solo automatiza un flujo, sino que también incorpora controles para reducir riesgos, evitar envíos no revisados, minimizar exposición de datos y dejar trazabilidad sobre el comportamiento del proceso.

---

## 2. Principios de seguridad aplicados

### 2.1 Uso de datos ficticios

El proyecto trabaja únicamente con datos ficticios. No se cargaron datos reales de alumnos, correos reales, consultas reales ni información institucional sensible.

Esto permite probar el flujo completo sin exponer información personal ni comprometer datos internos.

### 2.2 Minimización de datos

El sistema utiliza solo los campos necesarios para procesar una consulta ficticia:

- ID de consulta
- Nombre ficticio
- Email ficticio
- Canal
- Tipo de consulta
- Texto de la consulta
- Contexto interno ficticio
- Estado del caso
- Campos de revisión humana
- Salida simulada

No se incluyen datos sensibles como DNI, teléfonos reales, direcciones, legajos, pagos reales, credenciales, contraseñas ni información privada de alumnos.

### 2.3 Separación entre prueba y operación real

La automatización fue diseñada como una prueba funcional y documentada. Las salidas generadas por IA no se envían a personas reales.

El sistema trabaja con el campo **Salida simulada**, lo que permite demostrar el comportamiento del ecosistema sin ejecutar una acción externa riesgosa.

### 2.4 No exposición de credenciales

Durante la documentación y el video demo se deben ocultar:

- API Keys
- Credenciales de Make
- Tokens de Airtable
- Configuración sensible de cuentas conectadas
- Datos internos no necesarios para la evaluación

Las capturas utilizadas como evidencia muestran el flujo, los módulos y los resultados, pero no deben exponer secretos técnicos.

---

## 3. Human-in-the-loop, control humano y prevención del “efecto metralleta”

Uno de los controles principales del proyecto es el enfoque **Human-in-the-loop (HITL)**.

El sistema no envía respuestas finales automáticamente. Primero procesa la consulta con IA y deja el caso en estado de revisión humana.

Para que una consulta avance hacia una salida final, una persona debe revisar el contenido y aprobarlo manualmente.

### Campos involucrados

En la tabla **Consultas** de Airtable se utilizan campos de control como:

- `Estado`
- `Aprobado`
- `Estado final`
- `Estado salida`
- `Observaciones humanas`
- `Última modificación aprobación`

### Vista de control

La vista **Aprobados para salida** funciona como filtro de seguridad. Solo permite que el segundo escenario de Make detecte casos que cumplan estas condiciones:

- `Aprobado` marcado
- `Estado = Aprobado`
- `Estado salida = Bloqueado por HITL`

Esto evita que una respuesta generada por IA avance si no fue revisada por una persona.

---

## 4. Resiliencia del flujo

La resiliencia del sistema se trabaja mediante validaciones, filtros, registros de ejecución, control de estados y corrección de errores detectados durante la prueba.

### 4.1 Validación de datos mínimos

Antes de enviar una consulta al modelo de IA, el flujo verifica si el caso tiene los datos mínimos completos.

Campo utilizado:

- `Datos mínimos completos`

Si el caso no cumple las condiciones necesarias, el flujo puede evitar procesarlo y dejar registro del motivo en:

- `Motivo datos faltantes`
- `Datos incompletos detectados`

### 4.2 Uso de filtros y vistas específicas

El proyecto evita disparadores amplios o ambiguos usando vistas específicas de Airtable:

- `Por generar`
- `Aprobados para salida`

Esto reduce el riesgo de procesar registros incorrectos, duplicar operaciones o ejecutar el flujo sobre casos que no corresponden.

### 4.3 Control de triggers

Los escenarios de Make utilizan disparadores controlados desde Airtable. En el caso del segundo escenario, se usa el campo técnico:

- `Última modificación aprobación`

Este campo permite detectar cambios relevantes de aprobación sin depender de cualquier modificación menor del registro.

### 4.4 Salida estructurada en JSON

El procesamiento de IA devuelve una respuesta estructurada en JSON. Esto permite que Make pueda transformar la salida del modelo en campos concretos de Airtable.

El prompt obliga al modelo a responder con JSON puro, sin markdown ni texto adicional.

Reglas aplicadas en el prompt:

- Responder solamente con JSON puro.
- No usar bloques markdown.
- No escribir ```json.
- No agregar explicación antes ni después.
- El primer carácter debe ser `{`.
- El último carácter debe ser `}`.

---

## 5. Error real detectado y solución aplicada

Durante la prueba apareció un error real en el módulo **JSON Parse**.

### Error detectado

El módulo no podía interpretar correctamente la salida de OpenAI porque la respuesta no llegaba como JSON puro.

### Riesgo del error

Si el JSON no se interpreta correctamente:

- Airtable no puede actualizar los campos esperados.
- El flujo puede quedar interrumpido.
- La información generada por IA no queda estructurada.
- Se pierde trazabilidad del procesamiento.

### Solución aplicada

Se reforzó el prompt del módulo OpenAI con reglas estrictas de formato para asegurar una salida compatible con JSON Parse.

Después de esta corrección, el escenario volvió a ejecutarse correctamente y el módulo JSON Parse pudo interpretar la respuesta del modelo.

### Evidencias asociadas

- `EF_03_error_json_parse_openai_output.png`
- `EF_01_openai_prompt_formato_json_puro.png`
- `EF_02_json_parse_result_mapping.png`
- `EF_04_make_escenario_procesar_consulta_ok.png`

---

## 6. Logs de ejecución

El ecosistema incluye una tabla específica de trazabilidad llamada **Logs de ejecución**.

Esta tabla permite registrar:

- ID del log
- Consulta relacionada
- Escenario de Make ejecutado
- Fecha y hora
- Resultado
- Módulo afectado
- Mensaje de error
- Acción tomada
- Reintento
- Detección de datos incompletos

Los logs permiten auditar el flujo y entender qué ocurrió en cada ejecución.

---

## 7. Estados de control del sistema

El flujo no depende de un único campo. Usa varios estados para controlar el avance del caso.

Ejemplos:

- `Generando`
- `En revisión humana`
- `Aprobado`
- `Pendiente`
- `Listo para enviar`
- `Bloqueado por HITL`
- `Salida simulada`
- `Error`

Esta separación ayuda a evitar confusiones entre procesamiento IA, revisión humana y salida final.

---

## 8. Acciones críticas bloqueadas

El sistema considera acción crítica cualquier salida que pueda interpretarse como contacto final con un alumno o cliente.

Por eso, la salida final queda bloqueada hasta aprobación humana.

En esta entrega, la acción final se mantiene como **salida simulada**. No se envían correos reales, mensajes reales de Slack ni mensajes reales de WhatsApp.

---

## 9. Limitaciones reconocidas

El proyecto ya incluye controles de seguridad, filtros, HITL, logs y evidencia de corrección de errores.

Sin embargo, para una versión productiva sería recomendable sumar una ruta formal de **Error Handler** en Make con directivas como `Resume` o `Break`, especialmente para:

- Fallos de OpenAI
- Fallos de JSON Parse
- Fallos de actualización en Airtable
- Registros con datos incompletos
- Reintentos automáticos controlados

Esta limitación queda identificada como mejora futura, pero el proyecto ya documenta el error real encontrado, la corrección aplicada y la trazabilidad mediante la tabla de logs.

---

## 10. Mejoras futuras de resiliencia

Para escalar el ecosistema, se podrían agregar:

1. Error Handler formal en Make para OpenAI y JSON Parse.
2. Ruta de error que cree automáticamente un registro en `Logs de ejecución`.
3. Reintentos controlados ante fallos temporales de API.
4. Alertas internas por Slack ante errores críticos.
5. Validación más estricta de campos obligatorios antes del procesamiento IA.
6. Dashboard con tasa de errores por escenario.
7. Separación entre entorno de prueba y entorno productivo.
8. Control de versiones para prompts y blueprints.

---

## 11. Conclusión

El proyecto incorpora controles de seguridad y resiliencia acordes a una entrega final de automatización IA:

- Usa datos ficticios.
- Minimiza la información procesada.
- No expone credenciales.
- No ejecuta envíos reales.
- Bloquea acciones críticas mediante HITL.
- Usa filtros y vistas específicas.
- Registra trazabilidad mediante logs.
- Documenta un error real y su solución.
- Mantiene la salida final como simulada y controlada.

Esto permite presentar el sistema como un ecosistema de automatización IA funcional, seguro para pruebas y defendible como proyecto profesional.
