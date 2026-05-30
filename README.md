# 📚 Estadística & ML — Vault de Notas UNAL

Vault de Obsidian con notas de posgrado de la **Universidad Nacional de Colombia (Medellín)** en estadística, machine learning, deep learning, series de tiempo y procesos estocásticos.

## 📂 Contenido

| Carpeta | Notas | Descripción |
|---|---|---|
| `UNAL/Bayesiana/` | 17 notas | Estadística Bayesiana — desde Teorema de Bayes hasta Procesos Gaussianos |
| `UNAL/IA/Machine Learning/` | 9 notas | ML clásico — regresión, árboles, SVM, clustering, RL, MLOps |
| `UNAL/IA/Deep Learning/` | 9 notas | DL — redes neuronales, CNNs, Transformers, GNNs, transfer learning |
| `UNAL/Series de Tiempo/` | 11 notas | ARIMA/GARCH hasta DL para series de tiempo |
| `UNAL/Procesos Estocasticos/` | 16 notas | Cadenas de Márkov, Browniano, Itô, Martingalas + Finanzas Cuantitativas |
| `UNAL/Análisis Multivariado/` | En progreso | PCA, LDA, clustering multivariado, cópulas |
| `UNAL/Inferencia Causal/` | 3 notas | DAGs, do-Calculus, marco contrafactual |

## ⚡ Instalación rápida (para visualizar igual que el autor)

### 1. Clonar el repositorio como vault de Obsidian

```bash
git clone https://github.com/kevgonz-jpg/Estadistica-ML.git
```

### 2. Abrir en Obsidian

1. Abre **Obsidian**
2. Click en **"Open folder as vault"**
3. Selecciona la carpeta `Estadistica-ML/` clonada
4. Cuando Obsidian pregunte si confías en los plugins de la carpeta, selecciona **"Trust author and enable plugins"**

### 3. Instalar plugins de comunidad

Los `manifest.json` de los plugins están incluidos, pero los archivos `main.js` NO (por tamaño). Obsidian los descargará automáticamente al activarlos. Los plugins clave son:

| Plugin | Función |
|---|---|
| **Dataview** | Simuladores interactivos en JavaScript |
| **Plotly** | Gráficas estáticas interactivas dentro de las notas |
| **Execute Code** | Ejecutar código Python/R directamente en la nota |
| **Banners** | Imágenes de banner en el encabezado de cada nota |
| **Latex Suite** | Snippets de LaTeX para escribir matemáticas más rápido |
| **Quick Latex** | Fracciones automáticas y shortcuts LaTeX |
| **Editing Toolbar** | Barra de herramientas tipo Word |
| **Advanced Tables** | Edición avanzada de tablas Markdown |
| **Code Styler** | Estilizado de bloques de código |

Para instalarlos: `Settings → Community plugins → Browse` y busca cada uno por nombre.

### 4. Activar CSS Snippets

Los snippets de estilo están en `.obsidian/snippets/`. Para activarlos:
`Settings → Appearance → CSS Snippets` → activa **Destacado**, **mi-callout-rojo** y **simple**.

### 5. Configuración de fuentes (opcional)

El vault usa **Times New Roman** como fuente de texto y **Cambria Math** como fuente de interfaz. Puedes cambiarlas en `Settings → Appearance → Font`.

## 🗒️ Sobre las notas

Cada nota sigue un estándar riguroso definido en `UNAL/_ESTÁNDARES DE NOTAS.md`:
- YAML frontmatter con tags, fecha y fuentes
- Definiciones con cabeceras coloreadas en LaTeX
- Callouts: abstract, tip, warning, example
- **1 gráfica Plotly estática** + **1 simulador dataviewjs interactivo**
- Código en Python y R reproducible
- Mapa conceptual ASCII y sección de referencias cruzadas entre vaults

## 📖 Referencias principales

- Hastie, Tibshirani & Friedman — *The Elements of Statistical Learning* (ESL)
- Bishop — *Pattern Recognition and Machine Learning* (PRML)
- Gelman et al. — *Bayesian Data Analysis* (BDA3)
- Dobrow — *Introduction to Stochastic Processes with R* (Wiley, 2016)
- Shreve — *Stochastic Calculus for Finance I & II* (Springer)
- Goodfellow et al. — *Deep Learning*

---
*Vault mantenido por Kevin González — UNAL Medellín*
