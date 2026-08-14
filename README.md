# CUI Student Ops AI Hub

**Entrega Final — Ecosistema IA Autónomo para Negocios**

Proyecto desarrollado por **Matías Algañaraz** como entrega final del módulo integrador de automatización con inteligencia artificial.

---

## 1. Resumen del proyecto

**CUI Student Ops AI Hub** es un ecosistema de automatización con inteligencia artificial orientado a procesos de atención y operación de consultas de alumnos.

El sistema recibe consultas ficticias, las procesa mediante IA, genera una respuesta sugerida en formato estructurado, registra trazabilidad operativa, bloquea la salida mediante revisión humana y prepara una salida simulada solo después de aprobación.

El objetivo no es reemplazar a una persona, sino asistir tareas operativas repetitivas con mayor orden, control, trazabilidad y documentación.

---

## 2. Problema que resuelve

En procesos de atención al alumno suelen aparecer consultas repetitivas que requieren:

- Clasificación del tipo de consulta.
- Resumen operativo.
- Detección de prioridad.
- Generación de una primera respuesta orientativa.
- Revisión humana antes de cualquier acción crítica.
- Registro de trazabilidad y errores.

Este proyecto automatiza el procesamiento inicial de esas consultas sin enviar respuestas reales ni usar datos personales reales.

---

## 3. Herramientas utilizadas

- **Airtable**: base de datos, memoria operativa, logs y métricas.
- **Make**: orquestador principal del flujo.
- **OpenAI / GPT-4o-mini**: análisis de lenguaje natural, resumen, clasificación y generación de borradores.
- **JSON Parse**: transformación de la salida de IA en campos estructurados.
- **Google Drive**: documentación, evidencias, blueprint y video demo.
- **GitHub**: repositorio final de entrega y respaldo técnico.

---

## 4. Arquitectura general

Flujo principal:

```text
Consulta ficticia en Airtable
        ↓
Make detecta registros en vista “Por generar”
        ↓
Router valida datos mínimos completos
        ↓
OpenAI procesa la consulta
        ↓
La IA devuelve JSON estructurado
        ↓
JSON Parse transforma la salida
        ↓
Airtable actualiza el caso
        ↓
El caso queda bloqueado por HITL
        ↓
Revisión humana aprueba o detiene
        ↓
Make procesa salida aprobada
        ↓
Estado final: Listo para enviar / Salida simulada
```

---

## 5. Base de datos

La base principal en Airtable se llama:

```text
CUI Reply Assistant HITL
```

Tablas principales:

| Tabla | Función |
|---|---|
| Consultas | Registro de casos ficticios, estados, respuesta IA, aprobación y salida simulada. |
| Base de conocimiento | Reglas ficticias y criterios operativos para orientar respuestas. |
| Logs de ejecución | Registro de escenarios, resultados, errores y acciones tomadas. |
| Métricas Dashboard | Indicadores generales del sistema para monitoreo. |

---

## 6. Escenarios de Make

### Escenario 1 — Procesar consulta

```text
EF - CUI Student Ops AI Hub - Procesar consulta
```

Función:

- Detecta consultas nuevas en Airtable.
- Valida datos mínimos.
- Envía la consulta a OpenAI.
- Recibe una respuesta en JSON puro.
- Parsea la salida con JSON Parse.
- Actualiza el registro en Airtable.
- Crea un log de ejecución.
- Deja el caso en revisión humana.

### Escenario 2 — Salida aprobada

```text
EF - CUI Student Ops AI Hub - Salida aprobada
```

Función:

- Detecta casos aprobados en Airtable.
- Verifica que pasaron por revisión humana.
- Actualiza el estado final.
- Marca la salida como simulada.
- Evita envíos automáticos reales.

---

## 7. Human-in-the-loop

El sistema incorpora un punto de validación humana antes de cualquier acción crítica.

La IA puede sugerir una respuesta, resumir la consulta y preparar una salida simulada, pero no puede enviar una comunicación real de forma autónoma.

Condiciones para avanzar:

- El caso debe estar revisado.
- El campo **Aprobado** debe estar marcado.
- El estado debe ser **Aprobado**.
- El estado de salida debe estar bloqueado previamente por HITL.

---

## 8. Seguridad y privacidad

Medidas aplicadas:

- Uso exclusivo de datos ficticios.
- No se utilizan datos reales de alumnos.
- No se exponen API keys ni credenciales.
- No se confirman cupos, pagos, fechas, precios ni disponibilidad real.
- No se envían correos ni mensajes reales.
- La salida final queda simulada.
- Se documenta el error real de JSON Parse y su corrección.
- Se registran logs de ejecución.

Documento específico:

```text
/seguridad/seguridad_resiliencia_cui_student_ops_ai_hub.md
```

---

## 9. Optimización de costos

El proyecto utiliza **GPT-4o-mini** para tareas simples y repetitivas de clasificación, resumen y generación de borradores orientativos.

Criterios de optimización:

- Uso de modelo económico para tareas de bajo riesgo.
- Límite de tokens en Make.
- Filtros previos antes de llamar a la IA.
- JSON estructurado para evitar reprocesamiento.
- Uso de HITL para casos críticos.
- Batch API y prompt caching considerados como mejoras para procesamiento masivo.

Documento específico:

```text
/costos/matriz_costos_cui_student_ops_ai_hub.md
```

---

## 10. Dashboard de control

Vista pública de Airtable con KPIs del sistema:

```text
https://airtable.com/appv4HQuLoznGMA1Z/shr6pjBuPnE5lzkuR
```

KPIs incluidos:

- Total de consultas.
- Procesadas por IA.
- En revisión humana.
- Aprobadas.
- Listas para enviar.
- Errores.
- Tasa de error.
- Tasa de aprobación.

Documento específico:

```text
/dashboard/dashboard_control_cui_student_ops_ai_hub.md
```

---

## 11. Entregables de la rúbrica

| Criterio | Evidencia |
|---|---|
| Mapa de arquitectura | PDF final con arquitectura, flujo, herramientas, IA, routers, outputs y destino de datos. |
| Estructuras de datos + JSON | Documentación de tablas Airtable + blueprint JSON del flujo. |
| Optimización de costos | Matriz de costos y decisión de modelos. |
| Seguridad y resiliencia | Documento de seguridad, HITL, logs, error JSON Parse y medidas de control. |
| Dashboard de control | Airtable Shared View con KPIs y tasa de errores. |

---

## 12. Estructura sugerida del repositorio

```text
cui-student-ops-ai-hub/

README.md

/docs/
  Entrega_Final_CUI_Student_Ops_AI_Hub.pdf

/blueprint/
  blueprint_ef_cui_student_ops_ai_hub.json

/evidencias/
  EF_01_openai_prompt_formato_json_puro.png
  EF_02_json_parse_result_mapping.png
  EF_03_error_json_parse_openai_output.png
  EF_04_make_escenario_procesar_consulta_ok.png
  EF_05_airtable_consultas_y_salida_human_review.png
  EF_06_airtable_vista_aprobados_para_salida_filtros_hitl.png
  EF_07_make_trigger_salida_aprobada_configurado.png
  EF_08_make_escenario_salida_aprobada_configurado.png
  EF_09_make_escenario_salida_aprobada_ok.png
  EF_10_airtable_caso_aprobado_listo_para_enviar.png

/costos/
  matriz_costos_cui_student_ops_ai_hub.md

/seguridad/
  seguridad_resiliencia_cui_student_ops_ai_hub.md

/dashboard/
  dashboard_control_cui_student_ops_ai_hub.md

/video/
  link_video_demo.txt
```

---

## 13. Video demo

Estado: **pendiente de grabación**.

El video demo mostrará:

- Carpeta de entrega final.
- PDF y documentación.
- Base Airtable.
- Escenario 1 de Make.
- Error JSON Parse y solución.
- HITL y vista de aprobación.
- Escenario 2 de salida aprobada.
- Resultado final en Airtable.

Archivo final sugerido:

```text
Demo_Final_CUI_Student_Ops_AI_Hub.mp4
```

---

## 14. Estado del proyecto

```text
Estado: funcionando / en cierre de entrega
```

Componentes ya implementados:

- Base Airtable.
- Escenario Make de procesamiento IA.
- Escenario Make de salida aprobada.
- HITL.
- Logs.
- Métricas dashboard.
- Evidencias.
- PDF final.
- Blueprint JSON.
- Matriz de costos.
- Seguridad y resiliencia.
- Dashboard compartido.

Pendientes:

- Grabar video demo.
- Unir clips.
- Subir video final.
- Crear repositorio GitHub final.
- Copiar este README en el repositorio.

---

## 15. Valor profesional del proyecto

Este proyecto puede presentarse como un caso aplicado de:

- Automatización de procesos.
- Customer Experience.
- Back office operativo.
- Atención al alumno.
- Trazabilidad de flujos.
- Uso de IA con revisión humana.
- Arquitectura low-code/no-code.
- Documentación técnica para portfolio profesional.

