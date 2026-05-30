---
tags:
  - meta
  - estándares
  - plantilla
fecha: 2026-03-22
---

# 📋 Estándares de Notas — UNAL Kevin

> [!abstract] Propósito
> Este archivo es la **guía de referencia** para Claude (y para mí) al crear o editar cualquier nota del vault. En una nueva conversación, Claude debe leer este archivo primero para mantener consistencia total en estilo, estructura y herramientas.

---

## 1. Contexto del vault

- **Propietario:** Kevin — estudiante UNAL Medellín
- **Vault:** `C:\Users\ADMON\Documents\Obsidian Vault`
- **Obsidian versión:** Desktop (Windows)
- **Objetivo de las notas:** Aprendizaje profundo propio + compartible con otros estudiantes

### Estructura de carpetas activa

```
UNAL/
├── Bayesiana/           ← Estadística Bayesiana (en construcción activa)
├── Estadística 1/       ← Organizada por parciales
├── Fundamentos de Analitica/
└── Procesos Estocasticos/
```

### Fuentes de conocimiento disponibles (para Claude)
1. **Google Drive** — carpeta `Libros_Kevin` con libros organizados por tema
2. **Google Drive** — carpeta `estadistica UNAL MED` con contenido de materias UNAL
3. **Búsqueda web** — para complementar con fuentes académicas actualizadas
4. **Vault mismo** — notas existentes para mantener consistencia de notación

> ⚠️ **Instrucción para Claude:** Al iniciar una sesión nueva, siempre:
> 1. Leer este archivo
> 2. Listar el directorio de la materia en cuestión para ver qué notas ya existen
> 3. Buscar en Drive los libros relevantes al tema
> 4. Revisar una nota existente de la materia para mantener consistencia de notación

---

## 2. Plugins activos (relevantes para notas)

| Plugin | Sintaxis | Uso |
|---|---|---|
| **execute-code** | ` ```python ` / ` ```r ` | Código ejecutable con botón Run |
| **obsidian-plotly** | ` ```plotly ` | Gráficos interactivos con JSON de Plotly |
| **obsidian-latex-suite** | `$...$` / `$$...$$` | LaTeX inline y en bloque |
| **dataview** | ` ```dataview ` | Consultas dinámicas del vault |
| **obsidian-banners** | `banner:` en frontmatter | Imagen de cabecera visual |
| **code-styler** | `title=` en bloques de código | Título decorativo en bloques |
| **highlightr** | `==texto==` | Resaltado de texto |
| **table-editor** | Tablas markdown | Tablas editables |

### Rutas de ejecutables confirmadas
- **Python:** `C:/Python313/python.exe` (versión 3.13)
- **R:** `C:/Program Files/R/R-4.5.1/bin/x64/Rscript.exe` (versión 4.5.1)
- Ambos tienen `EmbedPlots: true` → los gráficos se renderizan inline al ejecutar

---

## 3. Anatomía de una nota perfecta

Toda nota de materia debe tener estas capas, **en este orden:**

```
1. Frontmatter YAML        → metadata, tags, banner
2. Título + callout resumen → idea central en 2-3 líneas
3. Contexto / motivación   → por qué importa este tema
4. Teoría                  → definiciones, teoremas, demostraciones con LaTeX
5. Ejemplos numéricos      → mínimo 2-3, resueltos paso a paso
6. Visualización Plotly    → gráfico interactivo (bloque ```plotly```)
7. Código Python ejecutable → implementación + gráfico Plotly con fig.show()
8. Código R ejecutable     → implementación + gráfico Plotly con print(fig)
9. Propiedades / teoremas  → en callouts [!tip]
10. Conexiones             → links internos [[nota]] + tags
11. Fuentes                → libros usados al final
```

---

## 4. Plantilla de frontmatter

```yaml
---
banner: "URL_de_imagen_unsplash"
banner_y: 0.4
tags:
  - tema-principal
  - subtema
  - materia
  - unal
fecha: YYYY-MM-DD
estado: borrador | en-progreso | completa
fuentes:
  - "Autor — Título del libro (edición)"
---
```

---

## 5. Estilo de escritura teórica

### Definiciones y teoremas
Usar colores consistentes con el estilo de Procesos Estocásticos:

```markdown
<font color="#00b0f0">Definición N</font> Texto de la definición...

<font color="#00b050">Ejemplo N</font> Texto del ejemplo...

<font color="#ff6b6b">Teorema N</font> Enunciado del teorema...
```

### Callouts de Obsidian (preferidos para notas nuevas)

```markdown
> [!abstract] Idea central
> Resumen ejecutivo de la nota.

> [!tip] Propiedad importante
> Contenido de la propiedad.

> [!warning] Cuidado / Paradoja
> Algo contraintuitivo o error común.

> [!note] Conexión con otra idea
> Links y referencias cruzadas.

> [!example] Ejemplo numérico
> Desarrollo paso a paso.

> [!quote] Cita memorable
> Autor — Fuente
```

### LaTeX — convenciones

- **Inline:** `$X \sim \mathcal{N}(\mu, \sigma^2)$`
- **Bloque centrado:** `$$...$$` para fórmulas importantes
- **Boxed para resultados clave:** `\boxed{P(H|E) = \frac{P(E|H)P(H)}{P(E)}}`
- Notación vectorial: `\boldsymbol{\theta}`, `\mathbf{X}`
- Conjuntos: `\mathbb{R}`, `\mathbb{N}`, `\Omega`
- Distribuciones: `\mathcal{N}`, `\text{Beta}`, `\text{Bin}`, `\text{Poisson}`

---

## 6. Estándar de código ejecutable

### Python
```python
# Siempre incluir:
# 1. Imports al inicio
# 2. Separadores de sección con ─── comentarios
# 3. Print de resultados con formato legible
# 4. fig.show() al final de cada gráfico Plotly

import numpy as np
import plotly.graph_objects as go

# ─── Descripción de la sección ────────────────────────────
# código aquí...
fig.show()
```

### R
```r
# Siempre incluir:
# 1. library() al inicio
# 2. Comentarios descriptivos con ─── 
# 3. print(fig) para Plotly
# 4. cat() para outputs formateados

library(plotly)

# ─── Descripción ──────────────────────────────────────────
fig <- plot_ly() %>% ...
print(fig)
```

---

## 7. Estándar de gráficos Plotly

### Bloques ```plotly (JSON estático — siempre visible sin ejecutar)
- Usar para visualizaciones conceptuales clave
- Siempre incluir `"hovertemplate"` en cada trace
- `"plot_bgcolor": "#f8f9fa"` como fondo estándar
- `"hovermode": "x unified"` para comparaciones
- Títulos con subtítulo via `"<br><sub>texto</sub>"`

### Gráficos en código Python/R (dinámicos — requieren ejecución)
- Para visualizaciones que dependen de cálculos
- Subplots cuando hay más de una perspectiva del mismo concepto
- Siempre incluir anotaciones explicativas en el gráfico

### Paleta de colores estándar
```
Azul principal:  #636EFA
Verde:           #00CC96
Naranja:         #FFA15A
Rojo:            #EF553B
Morado:          #AB63FA
Cyan:            #19D3F3
```

---

## 8. Nivel de profundidad por materia

| Materia | Profundidad teoría | Énfasis |
|---|---|---|
| **Bayesiana** | Alta — con demostraciones | Intuición + rigor matemático + aplicaciones |
| **Estadística 1** | Media — definiciones claras | Ejemplos con datos reales colombianos/ICFES |
| **Procesos Estocásticos** | Alta — mantener estilo del libro Dobrow | Consistencia con notación del profesor |
| **Fundamentos de Analítica** | Media-baja | Código y aplicación práctica |

---

## 9. Libros de referencia por materia (buscar en Drive → Libros_Kevin)

| Materia | Libros clave a buscar en Drive |
|---|---|
| **Bayesiana** | Gelman "Bayesian Data Analysis", Kruschke "Doing Bayesian Data Analysis", Bolstad |
| **Procesos Estocásticos** | Dobrow "Introduction to Stochastic Processes" (ya en vault), Ross |
| **Estadística 1** | Wackerly, Mendenhall & Scheaffer; Montgomery & Runger |
| **Machine Learning / Analítica** | Bishop "PRML", Hastie "ESL" |

> **Nota para Claude:** Siempre buscar primero en `Libros_Kevin` con el nombre del libro. Si está, leer el capítulo relevante antes de escribir la teoría. Si no está, usar búsqueda web con fuentes académicas (preferir arXiv, libros clásicos, Wikipedia matemática).

---

## 10. Flujo de trabajo completo para una nota nueva

```
1. Leer este archivo (_ESTÁNDARES DE NOTAS.md)
2. Listar directorio de la materia → ver notas existentes
3. Buscar en Drive (Libros_Kevin + estadistica UNAL MED) los libros del tema
4. Leer capítulos relevantes del libro (índice → secciones clave)
5. Buscar web para complementar (ejemplos adicionales, aplicaciones)
6. Redactar la nota siguiendo la anatomía de la Sección 3
7. Escribir con obsidian:write_file en la ruta correcta
8. Confirmar con obsidian:get_file_info que quedó bien guardada
```

---

## 11. Notas existentes y su estado

### UNAL/Bayesiana/
| Nota | Estado | Descripción |
|---|---|---|
| `Teorema de Bayes.md` | ✅ Completa | 31KB — teoría completa, 4 ejemplos numéricos, Python+R+3 Plotly |
| `Introducción.md` | ⬜ Vacía | Pendiente de desarrollar |

### UNAL/Procesos Estocásticos/
| Nota | Estado | Descripción |
|---|---|---|
| `0. Motivación.md` | 🔶 Parcial | Solo texto, sin código ni gráficos |
| `1. Introducción y Primeras definiciones.md` | 🔶 Parcial | Buen contenido teórico, sin Plotly |
| `2. Cadenas de Márkov en Tiempo Discreto.md` | 🔶 Parcial | Por revisar |
| `4. Proceso Poisson.md` | 🔶 Parcial | Por revisar |

---

*Archivo de estándares — actualizar cuando se acuerden nuevas convenciones*
*Creado: 2026-03-22*


---
