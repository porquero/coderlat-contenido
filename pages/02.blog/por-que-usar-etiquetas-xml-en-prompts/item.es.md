---
title: '¿Por qué usar etiquetas XML en prompts?'
date: '16-10-2026 10:00'
author: porquero
taxonomy:
    tool:
        - ia
    resource:
        - tutorial
    country:
        - chile
metadata:
    description: 'Un prompt es una instrucción en lenguaje natural, y como tal, puede ser ambiguo. Las etiquetas XML resuelven esa ambigüedad.'
faq:
    -
        q: '¿Cómo escribir un buen prompt?'
        a: 'Redacta de forma instruccional, definiendo rol, contexto y formato de salida.'
sources:
    -
        title: 'Prompting best practices'
        url: 'https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices'
---

Un prompt es una instrucción en lenguaje natural, y como tal, puede ser ambiguo. Las etiquetas XML resuelven esa ambigüedad delimitando con precisión qué parte del mensaje es la instrucción, qué es el contenido a procesar y qué formato debe tener la respuesta.'

===

## ¿Por qué usar etiquetas XML en prompts?
 
Un prompt es una instrucción en lenguaje natural, y como tal, puede ser ambiguo. Las etiquetas XML resuelven esa ambigüedad delimitando con precisión qué parte del mensaje es la instrucción, qué es el contenido a procesar y qué formato debe tener la respuesta.
 
A diferencia del formato Markdown —que usa `###` o `**negritas**` para dar estructura visual—, las etiquetas tienen apertura y cierre explícitos. Eso significa que el motor IA sabe exactamente dónde termina cada sección, sin inferirlo por contexto.
 
Las ventajas concretas:
 
- **Delimitación inequívoca**: el cierre `</etiqueta>` elimina ambigüedad, especialmente en contenido largo o con saltos de línea.
- **Aislamiento del contenido externo**: si se pega un texto, informe o dato dentro de una etiqueta, el motor IA lo trata como material a procesar y no como parte de la instrucción. Esto reduce el riesgo de que el contenido pegado altere el comportamiento del prompt.
- **Referenciabilidad**: es posible mencionar `<instruccion>` o `<salida>` en la línea de instrucción final, creando una estructura cohesiva y explícita.
- **Escalabilidad**: en prompts largos, con múltiples secciones o enviados vía API con contenido dinámico, las etiquetas mantienen la estructura intacta independientemente del volumen o variabilidad del contenido.
Las etiquetas no están predefinidas; se nombran según lo que contienen. Cualquier nombre descriptivo funciona: `<contrato>`, `<contexto>`, `<restricciones>`. Si el nombre le diría a un humano qué hay adentro, le sirve igual a el motor IA.
 
---

Plantillas base con etiquetas XML para cada entorno de el motor IA.
La línea de conjunción final es **opcional en la mayoría de casos**, pero se incluye donde agrega valor real.

---

## 1. Chat (el motor IA)

Uso conversacional. La línea de conjunción es opcional; incluirla solo si el prompt es largo o el formato de salida es estricto.

```xml
<instruccion>
[Rol y tarea. Ej: Eres un editor. Resume el texto en 3 puntos clave.]
</instruccion>

<contenido>
[Pega aquí el texto, datos o material a procesar]
</contenido>

<salida>
[Estructura esperada. Puede ser plantilla vacía o ejemplo relleno]
</salida>
```
---

Si el prompt es muy extenso se recomienda terminar con lo siguiente:
```xml
Según <instruccion>, procesa el <contenido> y responde en el formato de <salida>
```
---

## 2. Proyecto (Claude Projects)

La instrucción se define **una sola vez** en la configuración del proyecto. El usuario solo envía el contenido variable en cada chat.

Las etiquetas `<output_guardrails>` aseguran precisión, consistencia y manejo explícito de la incertidumbre en cada respuesta.

### Instrucciones del proyecto

```xml
<instruccion>
[Rol permanente y reglas fijas del proyecto.
Ej: Eres un analista financiero especializado en mercados latinoamericanos.]
</instruccion>

<salida>
[Formato de respuesta que se aplicará en cada interacción]
</salida>

<output_guardrails>
  <accuracy>
    - Si no tienes certeza sobre un dato, decláralo explícitamente en lugar de inferirlo.
    - Indica el nivel de confianza cuando la información sea incierta o parcial.
    - Basa tus respuestas únicamente en la información provista en este proyecto; no uses conocimiento general externo salvo que se indique lo contrario.
    - Cita o referencia la fuente específica de cada afirmación factual relevante.
  </accuracy>
  <consistency>
    - Mantén el formato de salida definido en las instrucciones del proyecto en cada respuesta.
    - Aplica el mismo nivel de detalle, tono y estructura en todas las interacciones.
    - Ante tareas complejas, descompónlas en subtareas consistentes antes de resolverlas.
    - Ante escenarios no cubiertos por las instrucciones, señálalo explícitamente en lugar de improvisar.
  </consistency>
  <uncertainty_handling>
    - Ante preguntas fuera del alcance o sin suficiente contexto: responde "No tengo suficiente información para responder esto con certeza."
    - No completes datos faltantes con suposiciones implícitas.
    - Si una solicitud es ambigua, solicita clarificación antes de proceder.
  </uncertainty_handling>
</output_guardrails>
```

---

## 3. Cowork (Claude Cowork)

Orientado a **objetivos y archivos**. Claude planifica y ejecuta tareas de forma autónoma en el sistema de archivos y aplicaciones. La línea de conjunción se reemplaza por una instrucción de supervisión.

```xml
<instruccion>
[Objetivo concreto orientado a un entregable.
Ej: Organiza los archivos de /Descargas por tipo y fecha, renombrando según convención YYYY-MM-DD_nombre.]
</instruccion>

<contexto>
[Carpetas, apps o conectores disponibles.
Ej: Acceso a /Proyectos, Google Drive y Slack]
</contexto>

<restricciones>
[Límites de la tarea. Ej: No eliminar archivos, solo mover. No modificar archivos anteriores a 2023.]
</restricciones>

<salida>
[Entregable esperado.
Ej: Carpetas reorganizadas + informe de cambios en PDF con listado de archivos movidos]
</salida>

Muéstrame el plan antes de ejecutar. Solicita confirmación antes de cualquier acción irreversible.
```

---

## 4. Claude Code

Orientado a **tareas de código desde terminal**. El prompt describe el objetivo técnico con contexto del stack. La estructura es suficientemente clara sin línea de conjunción.

```xml
<instruccion>
[Tarea de código específica.
Ej: Refactoriza utils.py eliminando funciones duplicadas y mejorando legibilidad.]
</instruccion>

<contexto>
[Stack, lenguaje, versión y restricciones.
Ej: Python 3.11, sin dependencias externas nuevas, mantener compatibilidad con tests existentes]
</contexto>

<restricciones>
[Opcional. Ej: No modificar la interfaz pública de las funciones. Sin cambios en archivos de test.]
</restricciones>

<salida>
[Resultado esperado.
Ej: Archivo modificado + comentarios explicando cada cambio + resumen de funciones eliminadas]
</salida>
```

---

## 5. API (Anthropic API)

El prompt se construye en código. Las etiquetas XML son **más importantes** aquí porque el contenido es dinámico e impredecible. La línea de conjunción **sí se incluye** para dar instrucción explícita al modelo.

```python
## ── Plantilla reutilizable ─────────────────────────────────────────────────────
PROMPT_TEMPLATE = """
<instruccion>{instruccion}</instruccion>
<contenido>{contenido}</contenido>
<restricciones>{restricciones}</restricciones>
<salida>{salida}</salida>
Según <instruccion>, procesa el <contenido> respetando <restricciones>,
y responde en el formato de <salida>.
""".strip()

def build_prompt(**kwargs) -> str:
    return PROMPT_TEMPLATE.format(**kwargs)


## ── Configuraciones específicas (separadas de la plantilla) ────────────────────
CONFIGS = {
    "analista_financiero": {
        "instruccion": "Eres un analista financiero experto...",
        "restricciones": "No uses conocimiento externo. Si falta información, indícalo.",
        "salida": "Riesgos:\nOportunidades:\nRecomendación:",
    },
    "revisor_legal": {
        "instruccion": "Eres un revisor legal especializado en contratos...",
        "restricciones": "Cita solo el texto provisto. No interpretes más allá.",
        "salida": "Cláusulas críticas:\nRiesgos legales:\nSugerencias:",
    },
}


## ── Uso ────────────────────────────────────────────────────────────────────────
config = CONFIGS["analista_financiero"]
prompt = build_prompt(**config, contenido=texto_externo)
```

---

## Referencia rápida

| Entorno | `<contenido>` | `<restricciones>` | Línea de conjunción |
|---------|:---:|:---:|:---:|
| Chat | Según necesidad | Opcional | Opcional |
| Proyecto | No (usa variables) | En guardrails | No |
| Cowork | No (usa archivos) | Sí | Reemplazar por supervisión |
| Claude Code | No (usa archivos) | Sí | No |
| API | Sí, siempre | Según necesidad | **Sí, siempre** |