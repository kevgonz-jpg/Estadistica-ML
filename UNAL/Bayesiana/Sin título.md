{
  "content": "---
banner: \"https://images.unsplash.com/photo-1558494949-ef010cbdcc31?w=1200\"
banner_y: 0.4
tags:
  - bayesiana
  - redes-bayesianas
  - modelos-graficos
  - DAG
  - inferencia-causal
  - belief-propagation
  - unal
fecha: 2026-04-03
estado: completa
fuentes:
  - \"Pearl — Probabilistic Reasoning in Intelligent Systems (1988)\"
  - \"Koller & Friedman — Probabilistic Graphical Models (MIT Press, 2009)\"
  - \"Bishop — Pattern Recognition and Machine Learning — Cap. 8\"
  - \"Pearl, Glymour & Jewell — Causal Inference in Statistics: A Primer (2016)\"
  - \"Darwiche — Modeling and Reasoning with Bayesian Networks (Cambridge, 2009)\"
---

# 🔵 Redes Bayesianas

> [!abstract] Idea central
> Una **red bayesiana** es un grafo acíclico dirigido (DAG) donde los nodos representan variables aleatorias y las aristas codifican dependencias causales o probabilísticas directas. La red factoriza la distribución conjunta en un producto de condicionales locales — una por nodo — lo que convierte problemas de alta dimensión en colecciones de problemas pequeños y manejables. La red no es solo una herramienta de inferencia: es un **lenguaje para representar conocimiento sobre un dominio** y razonar con él en ambas direcciones — hacia adelante (*predicción*) y hacia atrás (*diagnóstico*). Esto las hace el modelo central en IA clásica, diagnóstico médico, análisis de riesgo y, combinadas con el do-calculus de Pearl, el fundamento matemático de la **inferencia causal**.

---

## 1. El problema: la maldición dimensional de las distribuciones conjuntas

Supón que quieres modelar el rendimiento académico de estudiantes de UNAL usando cinco variables: $N$ (nota bachillerato), $H$ (horas estudio), $A$ (asistencia), $P$ (nota primer parcial), $F$ (nota final). Si cada variable toma 10 valores discretos, la distribución conjunta completa requiere $10^5 - 1 = 99{,}999$ parámetros.

Las redes bayesianas resuelven esto explotando la **independencia condicional**: si $F$ depende directamente solo de $P$ y $H$, entonces especificar $P(F \\mid P, H)$ es suficiente — no necesitamos $P(F \\mid N, H, A, P)$.

<font color=\"#00b0f0\">Definición 1 — Red Bayesiana</font> Una **red bayesiana** es un par $(\\mathcal{G}, \\mathcal{P})$ donde:

- $\\mathcal{G} = (V, E)$ es un DAG con nodos $V = \\{X_1, \\ldots, X_n\\}$ y aristas dirigidas $E$
- $\\mathcal{P} = \\{P(X_i \\mid \	ext{Pa}(X_i))\\}_{i=1}^n$ es el conjunto de **distribuciones condicionales locales (CPDs)**, una por nodo

La distribución conjunta se factoriza como:

$$\\boxed{P(X_1, \\ldots, X_n) = \\prod_{i=1}^n P(X_i \\mid \	ext{Pa}(X_i))}$$

donde $\	ext{Pa}(X_i)$ son los **padres** de $X_i$ en el grafo — los nodos con arista directa hacia $X_i$.

> [!tip] La factorización como prior estructural
> En el marco bayesiano, el grafo $\\mathcal{G}$ es un prior sobre la estructura de dependencias. Las CPDs $P(X_i \\mid \	ext{Pa}(X_i))$ son los parámetros del modelo, con sus propios priors (típicamente Dirichlet para variables discretas). El aprendizaje de la red es inferencia bayesiana sobre $\\mathcal{G}$ y sobre las CPDs.

---

## 2. Independencia condicional y d-separación

El grafo codifica un conjunto de afirmaciones de independencia condicional. Saber leerlas es la habilidad central para trabajar con redes bayesianas.

### 2.1 Tres patrones de dependencia

Hay exactamente tres formas en que tres variables pueden conectarse en un DAG:

**Cadena** $X \	o Y \	o Z$: $X$ y $Z$ son independientes dado $Y$.
$$P(X, Y, Z) = P(X)\\, P(Y \\mid X)\\, P(Z \\mid Y) \\implies X \\perp Z \\mid Y$$
Ejemplo: bachillerato → parcial 1 → nota final. Dado el parcial, bachillerato no aporta info sobre la final.

**Fork** $X \\leftarrow Y \\rightarrow Z$: $X$ y $Z$ son independientes dado $Y$.
$$P(X, Y, Z) = P(Y)\\, P(X \\mid Y)\\, P(Z \\mid Y) \\implies X \\perp Z \\mid Y$$
Ejemplo: horas estudio → parcial 1, horas estudio → nota final. Dado horas, parcial y final son independientes.

**Collider** $X \	o Y \\leftarrow Z$: $X$ y $Z$ son **dependientes** dado $Y$ (aunque marginalmente independientes).
$$P(X, Y, Z) = P(X)\\, P(Z)\\, P(Y \\mid X, Z) \\implies X \
ot\\perp Z \\mid Y$$
Ejemplo: talento → admisión ← esfuerzo. Marginalmente independientes, pero condicionados en haber sido admitido, talento y esfuerzo se vuelven negativamente correlacionados (*explaining away*).

> [!warning] El collider: la trampa más común
> Condicionar en un colisionador (o en sus descendientes) **abre** un camino que estaba bloqueado — introduce una dependencia espuria. Este es el origen del **sesgo de collider** en epidemiología y el fenómeno de *explaining away* en diagnóstico. Es contraintuitivo y una fuente frecuente de errores en análisis observacional.

### 2.2 d-separación

<font color=\"#00b0f0\">Definición 2 — d-separación</font> Un conjunto $\\mathbf{Z}$ **d-separa** $\\mathbf{X}$ de $\\mathbf{Y}$ en $\\mathcal{G}$ si bloquea todos los caminos no dirigidos entre cualquier $X \\in \\mathbf{X}$ e $Y \\in \\mathbf{Y}$. Un camino está **bloqueado** por $\\mathbf{Z}$ si contiene:

- Una cadena $A \	o B \	o C$ o fork $A \\leftarrow B \\rightarrow C$ con $B \\in \\mathbf{Z}$, **o**
- Un collider $A \	o B \\leftarrow C$ con $B \
otin \\mathbf{Z}$ y ningún descendiente de $B$ en $\\mathbf{Z}$

Si $\\mathbf{Z}$ d-separa $\\mathbf{X}$ de $\\mathbf{Y}$, entonces $\\mathbf{X} \\perp \\mathbf{Y} \\mid \\mathbf{Z}$ en cualquier distribución compatible con $\\mathcal{G}$ (**Markov condition**).

---

## 3. Visualización: red bayesiana de rendimiento académico UNAL

```plotly
{
  \"data\": [
    {
      \"type\": \"scatter\",
      \"mode\": \"markers+text\",
      \"x\": [1, 3, 1, 3, 5, 3],
      \"y\": [4, 4, 2, 2, 2, 0],
      \"text\": [\"Nota<br>bachillerato\", \"Horas<br>estudio\", \"Asistencia\", \"Parcial 1\", \"Parcial 2\", \"Nota<br>final\"],
      \"textposition\": [\"top center\",\"top center\",\"bottom center\",\"bottom center\",\"bottom center\",\"bottom center\"],
      \"marker\": {
        \"size\": [40,40,40,45,45,50],
        \"color\": [\"#636EFA\",\"#636EFA\",\"#636EFA\",\"#00CC96\",\"#00CC96\",\"#FFA15A\"],
        \"line\": {\"color\": \"#fff\", \"width\": 2}
      },
      \"hovertemplate\": \"%{text}<extra></extra>\"
    },
    {
      \"type\": \"scatter\", \"mode\": \"lines\",
      \"x\": [1,3,null, 3,3,null, 1,3,null, 3,3,null, 1,3,null, 3,3,null, 3,5,null, 3,3,null, 5,3],
      \"y\": [4,2,null, 4,2,null, 2,2,null, 2,0,null, 4,2,null, 2,0,null, 2,2,null, 4,0,null, 2,0],
      \"line\": {\"color\": \"rgba(128,128,128,0.4)\", \"width\": 1.5},
      \"hoverinfo\": \"skip\"
    }
  ],
  \"layout\": {
    \"title\": {
      \"text\": \"Red bayesiana — rendimiento académico UNAL<br><sub>Azul = covariables | Verde = parciales | Naranja = variable objetivo</sub>\",
      \"font\": {\"size\": 14}
    },
    \"xaxis\": {\"visible\": false, \"range\": [0, 6]},
    \"yaxis\": {\"visible\": false, \"range\": [-0.8, 5.2]},
    \"plot_bgcolor\": \"#f8f9fa\",
    \"height\": 340,
    \"annotations\": [
      {\"x\":2,\"y\":3.3,\"text\":\"→\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(99,110,250,0.6)\"}},
      {\"x\":2,\"y\":1.5,\"text\":\"→\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(99,110,250,0.6)\"}},
      {\"x\":1.8,\"y\":2.7,\"text\":\"↘\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(99,110,250,0.6)\"}},
      {\"x\":3,\"y\":1.2,\"text\":\"↓\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(0,204,150,0.6)\"}},
      {\"x\":3.8,\"y\":1.3,\"text\":\"↙\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(0,204,150,0.6)\"}},
      {\"x\":2.8,\"y\":3.1,\"text\":\"↓\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(99,110,250,0.6)\"}},
      {\"x\":4,\"y\":2,\"text\":\"→\",\"showarrow\":false,\"font\":{\"size\":16,\"color\":\"rgba(0,204,150,0.6)\"}}
    ]
  }
}
```

La red captura que la nota final depende directamente de los dos parciales y de horas de estudio. La nota de bachillerato influye en parcial 1 y en asistencia. Horas de estudio afecta todos los resultados intermedios. Esta estructura implica, por d-separación: dados los parciales 1 y 2, la nota de bachillerato no aporta información adicional sobre la nota final.

---

## 4. Inferencia exacta: el algoritmo de propagación de creencias

Dado el grafo y las CPDs, la tarea central es calcular la distribución posterior de una variable dada evidencia sobre otras: $P(X_i \\mid \\mathbf{e})$ donde $\\mathbf{e}$ es el conjunto de variables observadas.

### 4.1 Eliminación de variables

El algoritmo más directo suma variables irrelevantes una a una:

$$P(F \\mid N=3.5) = \\sum_{H, A, P_1, P_2} P(N=3.5, H, A, P_1, P_2, F)$$

La clave es **ordenar las sumas** para evitar recomputar. Con la factorización de la red:

$$= \\sum_{P_1, P_2} P(F \\mid P_1, P_2, H) \\sum_H P(P_2 \\mid H) \\sum_A P(P_1 \\mid N=3.5, H, A) P(A \\mid N=3.5) P(H)$$

El costo depende del **ancho del árbol** (*treewidth*) del grafo — en árboles y politrees es lineal; en grafos densos puede ser exponencial.

### 4.2 Belief Propagation (BP) en árboles

En redes con estructura de árbol, el algoritmo **suma-producto** (belief propagation) calcula todos los marginales en tiempo lineal pasando mensajes entre nodos vecinos.

<font color=\"#00b0f0\">Definición 3 — Mensajes BP</font> Cada nodo $X_i$ envía un mensaje $\\mu_{i \	o j}(X_j)$ a su vecino $X_j$ que resume toda la información del subárbol separado al cortar la arista $(i, j)$:

$$\\mu_{i \	o j}(x_j) = \\sum_{x_i} P(x_i \\mid x_j) \\prod_{k \\in \	ext{Ne}(i) \\setminus j} \\mu_{k \	o i}(x_i)$$

La creencia marginal en $X_i$ tras observar evidencia $\\mathbf{e}$ es proporcional al producto de todos los mensajes entrantes y la evidencia local:

$$b_i(x_i) \\propto P(e_i \\mid x_i) \\prod_{j \\in \	ext{Ne}(i)} \\mu_{j \	o i}(x_i)$$

BP en árboles es **exacto**. En grafos con ciclos, ejecutar BP de todas formas (llamado *loopy BP*) es una aproximación que funciona sorprendentemente bien en la práctica — es el algoritmo detrás de la decodificación de códigos LDPC y de los modelos de campo aleatorio de Markov.

---

## 5. Simulador interactivo: propagación de evidencia en la red UNAL

```dataviewjs
(() => {
    const container = this.container;
    container.empty();

    const header = container.createDiv();
    header.style.cssText = \"display:flex;justify-content:space-between;align-items:center;margin-bottom:10px;flex-wrap:wrap;gap:8px;\";
    header.createEl(\"span\", {text: \"Propagación de creencias — red bayesiana UNAL\",
        style: \"font-size:13px;font-weight:500;color:var(--color-text-primary);\"});

    const btnReset = header.createEl(\"button\", {text: \"↺ Limpiar evidencia\"});
    btnReset.style.cssText = \"padding:5px 12px;background:var(--color-background-secondary);color:var(--color-text-secondary);border:0.5px solid var(--color-border-secondary);border-radius:6px;font-size:12px;cursor:pointer;\";

    // Descripción
    const desc = container.createDiv();
    desc.style.cssText = \"font-size:11px;color:var(--color-text-tertiary);margin-bottom:12px;\";
    desc.textContent = \"Haz clic en las barras de cada variable para fijar evidencia. La red propaga la actualización bayesiana al resto de nodos.\";

    const canvas = container.createEl(\"canvas\");
    canvas.width = 820; canvas.height = 480;
    canvas.style.cssText = \"width:100%;background:transparent;cursor:pointer;\";
    const ctx = canvas.getContext(\"2d\");

    // ── Definición de la red ──────────────────────────────────────────────────
    // Variables: cada una tiene valores discretos (bajo/medio/alto)
    // Representadas como distribuciones sobre 3 niveles
    const NIVELES = [\"bajo\",\"medio\",\"alto\"];
    const COLORES_NIV = [\"#EF553B\",\"#FFA15A\",\"#1D9E75\"];
    const COLORES_NOD = {
        Bach: \"#636EFA\", Horas: \"#636EFA\", Asist: \"#636EFA\",
        Parc1: \"#00CC96\", Parc2: \"#00CC96\", Final: \"#FFA15A\"
    };

    // Posiciones de los nodos en el canvas
    const nodos = {
        Bach:  {x: 130, y: 90,  label: \"Nota\
bachillerato\"},
        Horas: {x: 420, y: 90,  label: \"Horas\
estudio\"},
        Asist: {x: 130, y: 240, label: \"Asistencia\"},
        Parc1: {x: 280, y: 240, label: \"Parcial 1\"},
        Parc2: {x: 560, y: 240, label: \"Parcial 2\"},
        Final: {x: 420, y: 390, label: \"Nota\
final\"}
    };

    // Aristas
    const aristas = [
        [\"Bach\",\"Asist\"],[\"Bach\",\"Parc1\"],
        [\"Horas\",\"Parc1\"],[\"Horas\",\"Parc2\"],[\"Horas\",\"Final\"],
        [\"Asist\",\"Parc1\"],
        [\"Parc1\",\"Final\"],[\"Parc2\",\"Final\"]
    ];

    // Distribuciones marginales a priori (sin evidencia)
    // Cada distribución es [P(bajo), P(medio), P(alto)]
    const priors = {
        Bach:  [0.20, 0.50, 0.30],
        Horas: [0.30, 0.45, 0.25],
        Asist: [0.25, 0.45, 0.30],
        Parc1: [0.28, 0.42, 0.30],
        Parc2: [0.30, 0.42, 0.28],
        Final: [0.32, 0.40, 0.28]
    };

    // Efecto de la evidencia: tabla simplificada de actualización
    // evidencia[nodo][nivel] = {nodo_afectado: [delta_bajo, delta_medio, delta_alto]}
    // Valores aproximados basados en una red real
    const efectos = {
        Bach: {
            alto:  {Asist:[-.08,.00,.08],Parc1:[-.10,.02,.08],Final:[-.06,.01,.05]},
            bajo:  {Asist:[.10,-.02,-.08],Parc1:[.12,-.02,-.10],Final:[.08,-.01,-.07]},
            medio: {}
        },
        Horas: {
            alto:  {Parc1:[-.08,.01,.07],Parc2:[-.09,.01,.08],Final:[-.12,.02,.10]},
            bajo:  {Parc1:[.10,-.01,-.09],Parc2:[.11,-.01,-.10],Final:[.14,-.02,-.12]},
            medio: {}
        },
        Asist: {
            alto:  {Parc1:[-.06,.01,.05],Final:[-.04,.01,.03]},
            bajo:  {Parc1:[.07,-.01,-.06],Final:[.05,-.01,-.04]},
            medio: {}
        },
        Parc1: {
            alto:  {Final:[-.10,.02,.08],Bach:[-.05,.01,.04],Horas:[-.04,.01,.03]},
            bajo:  {Final:[.12,-.02,-.10],Bach:[.06,-.01,-.05],Horas:[.05,-.01,-.04]},
            medio: {}
        },
        Parc2: {
            alto:  {Final:[-.08,.01,.07],Horas:[-.04,.01,.03]},
            bajo:  {Final:[.10,-.01,-.09],Horas:[.05,-.01,-.04]},
            medio: {}
        },
        Final: {
            alto:  {Parc1:[-.07,.01,.06],Parc2:[-.06,.01,.05],Horas:[-.05,.01,.04]},
            bajo:  {Parc1:[.08,-.01,-.07],Parc2:[.07,-.01,-.06],Horas:[.06,-.01,-.05]},
            medio: {}
        }
    };

    // Estado actual
    let creencias = JSON.parse(JSON.stringify(priors));
    let evidencia = {}; // {nodo: nivel_idx}

    function aplicarEvidencia(nodoEv, nivelIdx) {
        // Reset
        creencias = JSON.parse(JSON.stringify(priors));
        evidencia[nodoEv] = nivelIdx;

        // Para cada evidencia activa, propagar
        for(const [nodo, idx] of Object.entries(evidencia)){
            const nivel = NIVELES[idx];
            // Fijar el nodo a la evidencia
            creencias[nodo] = [0,0,0];
            creencias[nodo][idx] = 1.0;

            // Actualizar nodos afectados
            const ef = efectos[nodo]?.[nivel] || {};
            for(const [afectado, deltas] of Object.entries(ef)){
                if(evidencia[afectado] !== undefined) continue; // ya fijado
                const dist = creencias[afectado];
                const nueva = dist.map((p,k) => Math.max(0.02, p + deltas[k]));
                const total = nueva.reduce((a,b)=>a+b,0);
                creencias[afectado] = nueva.map(p=>p/total);
            }
        }
    }

    // Zonas clickeables: barra de cada nivel para cada nodo
    const barZones = []; // {nodo, nivel, x1, y1, x2, y2}

    function drawNode(key, nodo, dist, evIdx) {
        const cx=nodo.x, cy=nodo.y;
        const W=110, H=80;
        const color = COLORES_NOD[key];

        // Fondo nodo
        ctx.fillStyle = evIdx !== undefined ? color+\"33\" : \"rgba(128,128,128,0.06)\";
        ctx.strokeStyle = evIdx !== undefined ? color : \"rgba(128,128,128,0.3)\";
        ctx.lineWidth = evIdx !== undefined ? 2 : 0.8;
        ctx.beginPath();
        ctx.roundRect(cx-W/2, cy-H/2, W, H, 8);
        ctx.fill(); ctx.stroke();

        // Label
        const lines = nodo.label.split(\"\
\");
        ctx.fillStyle = \"var(--color-text-primary,#333)\";
        ctx.font = \"bold 11px Arial\"; ctx.textAlign=\"center\";
        lines.forEach((l,i) => ctx.fillText(l, cx, cy-H/2+14+(i*13)));

        // Barras de distribución (3 barras horizontales pequeñas)
        const barH = 10, barMaxW = 80, barGap = 13;
        const barStartY = cy - H/2 + 38;
        dist.forEach((p, k) => {
            const bx = cx - barMaxW/2;
            const by = barStartY + k*barGap;
            const bw = p*barMaxW;

            // Fondo barra
            ctx.fillStyle = \"rgba(128,128,128,0.12)\";
            ctx.fillRect(bx, by, barMaxW, barH);
            // Barra coloreada
            ctx.fillStyle = evIdx===k ? COLORES_NIV[k] : COLORES_NIV[k]+\"99\";
            ctx.fillRect(bx, by, bw, barH);
            // Porcentaje
            ctx.fillStyle = \"var(--color-text-secondary,#666)\";
            ctx.font = \"9px Arial\"; ctx.textAlign=\"left\";
            ctx.fillText((p*100).toFixed(0)+\"%\", bx+barMaxW+3, by+8);
            // Label nivel
            ctx.font = \"9px Arial\"; ctx.textAlign=\"right\";
            ctx.fillText(NIVELES[k][0].toUpperCase(), bx-3, by+8);

            // Registrar zona clickeable
            barZones.push({nodo:key, nivel:k,
                x1:bx-15, y1:by-2, x2:bx+barMaxW+25, y2:by+barH+2});
        });

        // Indicador de evidencia
        if(evIdx !== undefined){
            ctx.fillStyle=COLORES_NIV[evIdx];
            ctx.font=\"bold 10px Arial\"; ctx.textAlign=\"center\";
            ctx.fillText(`★ ${NIVELES[evIdx]}`, cx, cy+H/2-4);
        }
    }

    function draw() {
        ctx.clearRect(0,0,820,480);
        barZones.length = 0;

        // Aristas
        aristas.forEach(([from, to]) => {
            const nf=nodos[from], nt=nodos[to];
            const dx=nt.x-nf.x, dy=nt.y-nf.y;
            const len=Math.sqrt(dx*dx+dy*dy);
            const ux=dx/len, uy=dy/len;
            const x1=nf.x+ux*58, y1=nf.y+uy*42;
            const x2=nt.x-ux*58, y2=nt.y-uy*42;

            ctx.strokeStyle=\"rgba(128,128,128,0.35)\"; ctx.lineWidth=1.5;
            ctx.beginPath(); ctx.moveTo(x1,y1); ctx.lineTo(x2,y2); ctx.stroke();
            // Punta de flecha
            const angle=Math.atan2(y2-y1,x2-x1);
            ctx.fillStyle=\"rgba(128,128,128,0.5)\";
            ctx.beginPath();
            ctx.moveTo(x2,y2);
            ctx.lineTo(x2-10*Math.cos(angle-0.4),y2-10*Math.sin(angle-0.4));
            ctx.lineTo(x2-10*Math.cos(angle+0.4),y2-10*Math.sin(angle+0.4));
            ctx.closePath(); ctx.fill();
        });

        // Nodos
        for(const [key, nodo] of Object.entries(nodos)){
            drawNode(key, nodo, creencias[key], evidencia[key]);
        }

        // Leyenda
        ctx.fillStyle=\"rgba(128,128,128,0.6)\"; ctx.font=\"10px Arial\"; ctx.textAlign=\"left\";
        ctx.fillText(\"Haz clic en las barras para fijar evidencia  |  ★ = evidencia observada\", 30, 460);
        COLORES_NIV.forEach((c,k)=>{
            ctx.fillStyle=c; ctx.fillRect(30+k*90, 468, 10, 10);
            ctx.fillStyle=\"rgba(128,128,128,0.7)\";
            ctx.fillText(NIVELES[k], 44+k*90, 477);
        });
    }

    canvas.addEventListener(\"click\", (e) => {
        const rect = canvas.getBoundingClientRect();
        const scaleX = canvas.width / rect.width;
        const scaleY = canvas.height / rect.height;
        const mx = (e.clientX - rect.left) * scaleX;
        const my = (e.clientY - rect.top)  * scaleY;

        for(const z of barZones){
            if(mx>=z.x1 && mx<=z.x2 && my>=z.y1 && my<=z.y2){
                if(evidencia[z.nodo]===z.nivel){
                    // Quitar evidencia
                    delete evidencia[z.nodo];
                    creencias = JSON.parse(JSON.stringify(priors));
                    // Re-aplicar evidencias restantes
                    const evCopy = {...evidencia};
                    evidencia = {};
                    for(const [n,idx] of Object.entries(evCopy)){
                        aplicarEvidencia(n,idx);
                    }
                } else {
                    aplicarEvidencia(z.nodo, z.nivel);
                }
                draw();
                return;
            }
        }
    });

    btnReset.addEventListener(\"click\", ()=>{
        evidencia={};
        creencias=JSON.parse(JSON.stringify(priors));
        draw();
    });

    draw();
})();
```

> [!tip] Qué explorar en la red
> - **Fijar Horas = alto**: observa cómo mejoran Parcial 1, Parcial 2 y Nota Final simultáneamente — propagación *hacia adelante*.
> - **Fijar Nota Final = alto**: observa cómo sube la probabilidad de Horas = alto y Parciales = alto — propagación *hacia atrás* (diagnóstico).
> - **Fijar Parcial 1 = alto y Horas = bajo**: el collider entre Bachillerato → Parcial 1 ← Horas activa *explaining away* — bachillerato sube para explicar la buena nota en parcial 1 a pesar de pocas horas de estudio.
> - **Fijar dos variables en conflicto**: la red redistribuye credibilidad de forma coherente.

---

## 6. Aprendizaje de redes bayesianas

Hasta ahora asumimos que el grafo $\\mathcal{G}$ y las CPDs son conocidos. En la práctica hay que aprenderlos de datos.

### 6.1 Aprendizaje de parámetros (CPDs dados $\\mathcal{G}$)

Con el grafo fijo, aprender las CPDs es inferencia bayesiana estándar. Para variables discretas, el prior conjugado de la CPD $P(X_i \\mid \	ext{Pa}(X_i) = \\mathbf{pa})$ es una **Dirichlet**:

$$P(X_i \\mid \\mathbf{pa}) \\sim \	ext{Dirichlet}(\\alpha_1, \\ldots, \\alpha_K)$$

El posterior tras observar $N_{ij}$ casos de $(X_i = j, \	ext{Pa}(X_i) = \\mathbf{pa})$ es:

$$P(X_i \\mid \\mathbf{pa}) \\mid \	ext{datos} \\sim \	ext{Dirichlet}(\\alpha_1 + N_{i1}, \\ldots, \\alpha_K + N_{iK})$$

El prior de Dirichlet actúa como **pseudo-conteos**: $\\alpha_j$ observaciones imaginarias de la clase $j$. Con $\\alpha_j = 1$ se tiene el prior uniforme (suavizado de Laplace).

### 6.2 Aprendizaje estructural (aprender $\\mathcal{G}$)

Aprender el grafo es más difícil: el espacio de DAGs sobre $n$ nodos crece super-exponencialmente. Hay tres familias de algoritmos:

**Algoritmos basados en score:** asignar una puntuación a cada grafo (BIC bayesiano, BDe score) y buscar el grafo con mayor score usando búsqueda greedy, hill-climbing o algoritmos evolutivos.

**Algoritmos basados en tests de independencia (constraint-based):** el algoritmo PC y FCI realizan tests de independencia condicional para inferir la estructura del grafo. Son más interpretables pero sensibles a errores en los tests.

**Aprendizaje bayesiano completo:** poner un prior sobre grafos $P(\\mathcal{G})$ y calcular el posterior $P(\\mathcal{G} \\mid \	ext{datos})$ usando MCMC sobre el espacio de DAGs. Es lo correcto pero computacionalmente muy costoso para $n > 20$.

> [!note] El BDe score — conexión con el Factor de Bayes
> El **BDe score** (Bayesian Dirichlet equivalent) de un grafo $\\mathcal{G}$ es exactamente el logaritmo de la evidencia marginal $\\log P(\	ext{datos} \\mid \\mathcal{G})$ bajo priors Dirichlet conjugados. Comparar dos grafos por su BDe score es equivalente a comparar sus Factores de Bayes — exactamente la misma idea de la [[9. Selección y Comparación de Modelos]].

---

## 7. Inferencia causal: el do-calculus de Pearl

Las redes bayesianas capturan dependencias probabilísticas. Pero la ciencia quiere más: **relaciones causales** — lo que pasaría si interviniéramos en el sistema, no solo si observáramos.

<font color=\"#00b0f0\">Definición 4 — Intervención y el operador do()</font> En el formalismo de Pearl, la **intervención** $do(X = x)$ modifica el grafo eliminando todas las aristas que entran a $X$ y fijando $X = x$:

$$P(Y \\mid do(X = x)) \
eq P(Y \\mid X = x) \\quad \	ext{en general}$$

La diferencia es la distinción entre **correlación y causalidad**:
- $P(Y \\mid X = x)$: distribución de $Y$ entre unidades donde $X$ fue observado en $x$ (puede haber confounding)
- $P(Y \\mid do(X = x))$: distribución de $Y$ si forzáramos $X = x$ mediante intervención externa (efecto causal puro)

### 7.1 La fórmula de ajuste (backdoor criterion)

Si existe un conjunto de variables de confusión $\\mathbf{Z}$ que bloquea todos los caminos de puerta trasera de $X$ a $Y$, el efecto causal es identificable:

$$P(Y \\mid do(X = x)) = \\sum_{\\mathbf{z}} P(Y \\mid X = x, \\mathbf{Z} = \\mathbf{z})\\, P(\\mathbf{Z} = \\mathbf{z})$$

Esta fórmula justifica el ajuste por covariables en regresión observacional — pero solo cuando las covariables correctas satisfacen el criterio de backdoor.

> [!example] Ejemplo 1 — Efecto causal de las horas de estudio en la nota final
> En la red UNAL, Horas → Final tiene un efecto causal directo. Pero observar $H = \	ext{alto}$ no es lo mismo que intervenir $do(H = \	ext{alto})$: en los datos observacionales, los estudiantes con muchas horas también tienden a tener mejor bachillerato (hay confounding por motivación no observada). Para estimar el efecto causal puro de aumentar horas de estudio, se necesita controlar por las variables que satisfacen el criterio de backdoor.

### 7.2 El modelo causal estructural (SCM)

Un **Structural Causal Model** extiende la red bayesiana con ecuaciones estructurales:

$$X_i := f_i(\	ext{Pa}(X_i), U_i), \\qquad U_i \\sim P(U_i)$$

donde $U_i$ son ruidos exógenos (causas no observadas). Los SCM permiten responder tres tipos de preguntas en orden de complejidad creciente (la \"escalera causal\" de Pearl):

| Nivel | Pregunta | Operación | Ejemplo |
|---|---|---|---|
| **Asociación** | ¿Qué ocurre si observo? | $P(Y \\mid X)$ | ¿Qué nota tienen los que estudian mucho? |
| **Intervención** | ¿Qué ocurre si hago? | $P(Y \\mid do(X))$ | ¿Qué nota tendrían si les obligáramos a estudiar más? |
| **Contrafactual** | ¿Qué habría ocurrido si? | $P(Y_x \\mid X', Y')$ | ¿Hubiera aprobado Juan si hubiera estudiado más? |

---

## 8. Redes bayesianas dinámicas (DBN)

Las **redes bayesianas dinámicas** modelan secuencias temporales añadiendo una dimensión de tiempo. Son la generalización de los Modelos Ocultos de Markov (HMM de la [[11. Modelos de Mezclas y Variables Latentes]]) a múltiples variables interrelacionadas.

<font color=\"#00b0f0\">Definición 5 — DBN</font> Una DBN define una distribución sobre secuencias $\\{X_1^{(t)}, \\ldots, X_n^{(t)}\\}_{t=1}^T$ mediante:

- Una red inicial $P(\\mathbf{X}^{(1)})$ para el tiempo $t=1$
- Una red de transición $P(\\mathbf{X}^{(t+1)} \\mid \\mathbf{X}^{(t)})$ que se repite en cada paso temporal

Las aristas dentro de un tiempo $t$ capturan dependencias sincrónicas; las aristas entre $t$ y $t+1$ capturan dependencias temporales.

> [!example] Ejemplo 2 — DBN para seguimiento del aprendizaje
> En una DBN para modelar el aprendizaje semestre a semestre en UNAL: las variables $\\{$Comprensión$_t$, Esfuerzo$_t$, Nota$_t\\}$ en cada semestre $t$ forman una red, y la comprensión del semestre $t$ influye en la comprensión del semestre $t+1$ — el aprendizaje es acumulativo. La inferencia en la DBN usa el **filtro bayesiano** (una extensión de la regla de Bayes a la actualización secuencial de estados) o el algoritmo de Viterbi para encontrar la secuencia de estados más probable.

---

## 9. Ideas para gráficos adicionales

> [!note] Gráfico 3 — D-separación interactiva: ¿están X e Y separados dado Z? (pendiente de implementar)
>
> **Tipo:** Grafo interactivo con tres modos de conexión (cadena, fork, collider) y un panel de preguntas.
>
> **Setup:** Tres nodos A, B, C con estructura seleccionable. Un selector de \"¿está B en el conjunto de evidencia?\".
>
> **Visualización:** Al cambiar la estructura y el estado de B, el camino entre A y C se pinta verde (abierto, dependientes) o rojo (bloqueado, independientes). Texto en tiempo real: \"A ⊥ C | {B}\" o \"A ⊬ C | {B}\".
>
> **Por qué vale la pena:** la d-separación es la regla más importante y más contraintuitiva de las redes bayesianas. El collider es el caso donde condicionar **abre** una dependencia que no existía. Ver esto animado con el camino cambiando de color es la forma más rápida de entenderlo.

> [!note] Gráfico 4 — Escalera causal: asociación vs. intervención vs. contrafactual (pendiente de implementar)
>
> **Tipo:** Tres paneles con distribuciones $P(Y|X=x)$, $P(Y|do(X=x))$ y $P(Y_x|X',Y')$.
>
> **Setup:** La red UNAL simplificada con un confundidor oculto (motivación, no observada) entre Horas y Nota. Slider de \"fuerza del confundidor\" (0 a 1).
>
> **Panel 1 (asociación):** histograma de Nota Final dado Horas = alto — incluye efecto causal + confounding.
>
> **Panel 2 (intervención):** distribución después de aplicar la fórmula de ajuste — solo efecto causal.
>
> **Panel 3 (contrafactual):** distribución de \"¿qué nota habría tenido el estudiante Juan si hubiera estudiado más?\" — usa el ruido exógeno observado de Juan.
>
> **Al mover el slider de confundidor:** los paneles 1 y 2 divergen más — el confundidor aumenta la brecha entre correlación y causalidad. El panel 3 permanece igual — los contrafactuales son independientes del confundidor observacional.
>
> **Por qué vale la pena:** la distinción entre los tres paneles es el corazón de la \"revolución causal\" de Pearl. Ver cómo divergen con el confundidor hace la diferencia visceral.

---

## 10. Código Python — red bayesiana con pgmpy

```python
# title=Redes Bayesianas — pgmpy: construcción, CPDs y propagación de creencias
import numpy as np
import pandas as pd
from pgmpy.models import BayesianNetwork
from pgmpy.factors.discrete import TabularCPD
from pgmpy.inference import VariableElimination, BeliefPropagation
import plotly.graph_objects as go
from plotly.subplots import make_subplots

np.random.seed(42)

# ─── Definición de la red ─────────────────────────────────────────────────────
# Variables: Bach(0=bajo,1=medio,2=alto), Horas, Asist, Parc1, Parc2, Final

modelo = BayesianNetwork([
    (\"Bach\",  \"Asist\"),
    (\"Bach\",  \"Parc1\"),
    (\"Horas\", \"Parc1\"),
    (\"Horas\", \"Parc2\"),
    (\"Horas\", \"Final\"),
    (\"Asist\", \"Parc1\"),
    (\"Parc1\", \"Final\"),
    (\"Parc2\", \"Final\")
])

# ─── Tablas de probabilidad condicional (CPDs) ────────────────────────────────
# P(Bach): marginal a priori
cpd_bach = TabularCPD(\"Bach\", 3,
    values=[[0.20], [0.50], [0.30]])

# P(Horas): marginal a priori
cpd_horas = TabularCPD(\"Horas\", 3,
    values=[[0.30], [0.45], [0.25]])

# P(Asist | Bach): bachillerato alto → más asistencia
cpd_asist = TabularCPD(\"Asist\", 3,
    values=[
        [0.35, 0.25, 0.15],  # P(Asist=bajo | Bach=bajo,medio,alto)
        [0.45, 0.45, 0.40],  # P(Asist=medio)
        [0.20, 0.30, 0.45]   # P(Asist=alto)
    ],
    evidence=[\"Bach\"], evidence_card=[3])

# P(Parc1 | Bach, Horas, Asist): 3x3x3=27 configuraciones
# Construir como tabla: Bach x Horas x Asist
vals_p1 = np.zeros((3, 27))
idx = 0
for b in range(3):
    for h in range(3):
        for a in range(3):
            base = (b + h + a) / 6  # índice de facilidad [0, 1]
            p_alto = 0.15 + base * 0.65
            p_bajo = max(0.05, 0.50 - base * 0.45)
            p_medio = max(0.05, 1 - p_alto - p_bajo)
            vals_p1[:, idx] = [p_bajo, p_medio, p_alto]
            idx += 1

cpd_parc1 = TabularCPD(\"Parc1\", 3, values=vals_p1,
    evidence=[\"Bach\",\"Horas\",\"Asist\"], evidence_card=[3,3,3])

# P(Parc2 | Horas): más simple
cpd_parc2 = TabularCPD(\"Parc2\", 3,
    values=[
        [0.45, 0.25, 0.10],
        [0.40, 0.45, 0.35],
        [0.15, 0.30, 0.55]
    ],
    evidence=[\"Horas\"], evidence_card=[3])

# P(Final | Parc1, Parc2, Horas): 3x3x3=27 configuraciones
vals_f = np.zeros((3, 27))
idx = 0
for p1 in range(3):
    for p2 in range(3):
        for h in range(3):
            base = (p1*0.45 + p2*0.35 + h*0.20) / 3
            p_alto = 0.10 + base * 0.75
            p_bajo = max(0.05, 0.55 - base * 0.50)
            vals_f[:, idx] = [p_bajo, max(0.05,1-p_alto-p_bajo), p_alto]
            idx += 1

cpd_final = TabularCPD(\"Final\", 3, values=vals_f,
    evidence=[\"Parc1\",\"Parc2\",\"Horas\"], evidence_card=[3,3,3])

# Añadir CPDs al modelo
modelo.add_cpds(cpd_bach, cpd_horas, cpd_asist, cpd_parc1, cpd_parc2, cpd_final)
assert modelo.check_model(), \"El modelo tiene errores en las CPDs\"
print(\"Modelo válido ✓\")

# ─── Inferencia por eliminación de variables ──────────────────────────────────
ie = VariableElimination(modelo)

print(\"\
=== INFERENCIA SIN EVIDENCIA ===\")
for var in [\"Bach\",\"Horas\",\"Parc1\",\"Parc2\",\"Final\"]:
    q = ie.query([var])
    print(f\"  P({var}): bajo={q.values[0]:.3f}  medio={q.values[1]:.3f}  alto={q.values[2]:.3f}\")

print(\"\
=== INFERENCIA CON EVIDENCIA: Horas=alto ===\")
ev = {\"Horas\": 2}
for var in [\"Parc1\",\"Parc2\",\"Final\",\"Asist\"]:
    q = ie.query([var], evidence=ev)
    print(f\"  P({var}|Horas=alto): bajo={q.values[0]:.3f}  medio={q.values[1]:.3f}  alto={q.values[2]:.3f}\")

print(\"\
=== DIAGNÓSTICO: Final=alto → causas más probables ===\")
ev_diag = {\"Final\": 2}
for var in [\"Horas\",\"Bach\",\"Parc1\"]:
    q_prior = ie.query([var])
    q_post  = ie.query([var], evidence=ev_diag)
    delta = q_post.values[2] - q_prior.values[2]
    print(f\"  P({var}=alto|Final=alto) = {q_post.values[2]:.3f}  (prior: {q_prior.values[2]:.3f}, Δ={delta:+.3f})\")

print(\"\
=== EXPLAINING AWAY: Parc1=alto, Horas=bajo ===\")
ev_ea = {\"Parc1\": 2, \"Horas\": 0}
q_bach_ea = ie.query([\"Bach\"], evidence=ev_ea)
q_bach_prior = ie.query([\"Bach\"])
print(f\"  P(Bach=alto) prior:                {q_bach_prior.values[2]:.3f}\")
print(f\"  P(Bach=alto|Parc1=alto):           {ie.query(['Bach'],evidence={'Parc1':2}).values[2]:.3f}\")
print(f\"  P(Bach=alto|Parc1=alto,Horas=bajo):{q_bach_ea.values[2]:.3f}\")
print(f\"  → Explaining away: al saber Horas=bajo, la explicación 'Bach=alto' se vuelve más necesaria\")

# ─── Aprendizaje de parámetros desde datos ────────────────────────────────────
from pgmpy.estimators import BayesianEstimator

# Simular datos
n_datos = 500
datos_sim = modelo.simulate(n_datos, seed=42)
print(f\"\
=== APRENDIZAJE DE CPDs DESDE {n_datos} OBSERVACIONES ===\")

# Aprender con prior Dirichlet(alpha=1) — suavizado de Laplace
modelo_aprendido = BayesianNetwork(modelo.edges())
modelo_aprendido.fit(datos_sim, estimator=BayesianEstimator,
                     prior_type=\"BDeu\", equivalent_sample_size=5)

# Comparar CPD aprendida vs. verdadera para Parc2
cpd_aprendida = modelo_aprendido.get_cpds(\"Parc2\")
print(\"  CPD Parc2 aprendida (vs. verdadera):\")
print(f\"    P(Parc2=alto|Horas=bajo):  {cpd_aprendida.values[2,0]:.3f}  (verdadera: 0.15)\")
print(f\"    P(Parc2=alto|Horas=medio): {cpd_aprendida.values[2,1]:.3f}  (verdadera: 0.30)\")
print(f\"    P(Parc2=alto|Horas=alto):  {cpd_aprendida.values[2,2]:.3f}  (verdadera: 0.55)\")

# ─── Visualización: comparación de distribuciones con y sin evidencia ─────────
escenarios = {
    \"Sin evidencia\": {},
    \"Horas = alto\": {\"Horas\": 2},
    \"Final = alto\": {\"Final\": 2},
    \"Parc1=alto, Horas=bajo\": {\"Parc1\": 2, \"Horas\": 0}
}
variables = [\"Bach\",\"Horas\",\"Parc1\",\"Parc2\",\"Final\"]
colores_e = [\"#888\",\"#636EFA\",\"#00CC96\",\"#EF553B\"]

fig = make_subplots(rows=1, cols=5, subplot_titles=variables,
                    shared_yaxes=True)

for j, var in enumerate(variables):
    for i, (escen, ev) in enumerate(escenarios.items()):
        if var in ev:
            continue  # es evidencia, no la ploteamos
        try:
            q = ie.query([var], evidence=ev)
            fig.add_trace(go.Bar(
                name=escen if j==0 else \"\",
                x=[\"bajo\",\"medio\",\"alto\"], y=q.values,
                marker_color=colores_e[i], opacity=0.75,
                showlegend=j==0,
                hovertemplate=f\"{var}=%{{x}}<br>P=%{{y:.3f}}<extra>{escen}</extra>\"),
                row=1, col=j+1)
        except:
            pass

fig.update_layout(
    title=dict(text=\"Red Bayesiana UNAL — distribuciones bajo distintas evidencias<br><sub>Sin evidencia | Horas=alto | Final=alto | Explaining away (Parc1=alto, Horas=bajo)</sub>\",
               font=dict(size=13)),
    plot_bgcolor=\"#f8f9fa\", height=360, barmode=\"group\", hovermode=\"closest\")
fig.update_yaxes(title_text=\"Probabilidad\", range=[0,1], col=1)
fig.show()
```

---

## 11. Código R — red bayesiana con bnlearn

```r
# title=Redes Bayesianas — bnlearn: aprendizaje estructural y propagación (R)
library(bnlearn)
library(tidyverse)
library(plotly)

set.seed(42)

# ─── Datos simulados: rendimiento académico UNAL ──────────────────────────────
n <- 600
bach  <- sample(c(\"bajo\",\"medio\",\"alto\"), n, replace=TRUE, prob=c(0.2,0.5,0.3))
horas <- sample(c(\"bajo\",\"medio\",\"alto\"), n, replace=TRUE, prob=c(0.3,0.45,0.25))

# Variables derivadas con dependencia
asist <- mapply(function(b) sample(c(\"bajo\",\"medio\",\"alto\"), 1,
    prob=list(bajo=c(0.35,0.45,0.20),medio=c(0.25,0.45,0.30),alto=c(0.15,0.40,0.45))[[b]]), bach)

base_p1 <- (as.integer(factor(bach,c(\"bajo\",\"medio\",\"alto\")))-1 +
             as.integer(factor(horas,c(\"bajo\",\"medio\",\"alto\")))-1 +
             as.integer(factor(asist,c(\"bajo\",\"medio\",\"alto\")))-1) / 6
parc1 <- sapply(base_p1, function(b) sample(c(\"bajo\",\"medio\",\"alto\"), 1,
    prob=c(max(0.05, 0.50-b*0.45), max(0.05, 0.35-b*0.10), min(0.90, 0.15+b*0.65))))

base_p2 <- (as.integer(factor(horas,c(\"bajo\",\"medio\",\"alto\")))-1) / 2
parc2 <- sapply(base_p2, function(b) sample(c(\"bajo\",\"medio\",\"alto\"), 1,
    prob=c(max(0.05, 0.45-b*0.35), max(0.05, 0.40-b*0.05), min(0.90, 0.15+b*0.40))))

base_f <- (as.integer(factor(parc1,c(\"bajo\",\"medio\",\"alto\")))-1)*0.45 +
          (as.integer(factor(parc2,c(\"bajo\",\"medio\",\"alto\")))-1)*0.35 +
          (as.integer(factor(horas,c(\"bajo\",\"medio\",\"alto\")))-1)*0.20
base_f <- base_f / 3
final <- sapply(base_f, function(b) sample(c(\"bajo\",\"medio\",\"alto\"), 1,
    prob=c(max(0.05, 0.50-b*0.45), max(0.05, 0.35-b*0.05), min(0.90, 0.15+b*0.65))))

datos <- data.frame(
    Bach=factor(bach,c(\"bajo\",\"medio\",\"alto\")),
    Horas=factor(horas,c(\"bajo\",\"medio\",\"alto\")),
    Asist=factor(asist,c(\"bajo\",\"medio\",\"alto\")),
    Parc1=factor(parc1,c(\"bajo\",\"medio\",\"alto\")),
    Parc2=factor(parc2,c(\"bajo\",\"medio\",\"alto\")),
    Final=factor(final,c(\"bajo\",\"medio\",\"alto\"))
)

cat(\"=== DATOS UNAL ===\
\")
cat(sprintf(\"n=%d | P(Final=alto)=%.3f\
\", n, mean(datos$Final==\"alto\")))

# ─── Definir estructura manualmente ──────────────────────────────────────────
dag <- model2network(\"[Bach][Horas][Asist|Bach][Parc1|Bach:Horas:Asist][Parc2|Horas][Final|Parc1:Parc2:Horas]\")
cat(\"\
Grafo manual: nodos=\", length(nodes(dag)), \"| aristas=\", nrow(arcs(dag)),\"\
\")

# ─── Aprendizaje de CPDs con prior BDeu ──────────────────────────────────────
bn_fit <- bn.fit(dag, data=datos, method=\"bayes\", iss=5)

cat(\"\
=== CPD APRENDIDA: P(Final | ...) ===\
\")
print(bn.fit.barchart(bn_fit$Final))

# ─── Aprendizaje estructural automático (Hill-Climbing + BIC) ─────────────────
dag_aprendido <- hc(datos, score=\"bic\")
cat(\"\
=== ESTRUCTURA APRENDIDA (Hill-Climbing + BIC) ===\
\")
cat(\"Aristas:\
\")
print(arcs(dag_aprendido))

# Comparar con grafo manual
diff_arist <- setdiff(arcs2str(arcs(dag)), arcs2str(arcs(dag_aprendido)))
cat(sprintf(\"Aristas en manual pero no en aprendido: %d\
\", length(diff_arist)))

# ─── Inferencia exacta ────────────────────────────────────────────────────────
library(gRain)
junction <- compile(as.grain(bn_fit))

cat(\"\
=== INFERENCIA SIN EVIDENCIA ===\
\")
q0 <- querygrain(junction, nodes=c(\"Final\",\"Parc1\"))
cat(\"P(Final):\", round(q0$Final,3), \"\
\")
cat(\"P(Parc1):\", round(q0$Parc1,3), \"\
\")

# Con evidencia
junction_ev1 <- setEvidence(junction, nodes=\"Horas\", states=\"alto\")
q1 <- querygrain(junction_ev1, nodes=c(\"Parc1\",\"Parc2\",\"Final\"))
cat(\"\
=== CON Horas=alto ===\
\")
cat(\"P(Final|Horas=alto):\", round(q1$Final,3), \"\
\")
cat(\"P(Parc1|Horas=alto):\", round(q1$Parc1,3), \"\
\")

# Explaining away
junction_ea <- setEvidence(junction, nodes=c(\"Parc1\",\"Horas\"), states=c(\"alto\",\"bajo\"))
q_ea <- querygrain(junction_ea, nodes=\"Bach\")
q_prior <- querygrain(junction, nodes=\"Bach\")
cat(\"\
=== EXPLAINING AWAY: Parc1=alto, Horas=bajo ===\
\")
cat(\"P(Bach=alto) prior:                     \", round(q_prior$Bach[\"alto\"],3), \"\
\")
cat(\"P(Bach=alto|Parc1=alto,Horas=bajo):     \", round(q_ea$Bach[\"alto\"],3), \"\
\")

# ─── Visualización ────────────────────────────────────────────────────────────
escenarios <- list(
    \"Sin evidencia\"        = list(nodes=NULL, states=NULL),
    \"Horas=alto\"           = list(nodes=\"Horas\", states=\"alto\"),
    \"Final=alto\"           = list(nodes=\"Final\", states=\"alto\"),
    \"Parc1=alto,Horas=bajo\"= list(nodes=c(\"Parc1\",\"Horas\"), states=c(\"alto\",\"bajo\"))
)
vars_plot <- c(\"Bach\",\"Horas\",\"Parc1\",\"Parc2\",\"Final\")
colores_e <- c(\"#888\",\"#636EFA\",\"#00CC96\",\"#EF553B\")

fig <- plot_ly()
niveles <- c(\"bajo\",\"medio\",\"alto\")

for(j in seq_along(vars_plot)){
    var <- vars_plot[j]
    for(i in seq_along(escenarios)){
        es <- escenarios[[i]]
        if(var %in% (es$nodes %||% \"\")) next
        junc_es <- if(is.null(es$nodes)) junction else
            setEvidence(junction, nodes=es$nodes, states=es$states)
        q <- querygrain(junc_es, nodes=var)[[var]]
        fig <- fig %>% add_trace(
            x=niveles, y=as.numeric(q[niveles]),
            type=\"bar\", name=names(escenarios)[i],
            marker=list(color=colores_e[i], opacity=0.75),
            showlegend=(j==1),
            hovertemplate=paste0(var,\"=%{x}<br>P=%{y:.3f}<extra>\",names(escenarios)[i],\"</extra>\"),
            offsetgroup=i, xaxis=paste0(\"x\",j), yaxis=\"y\"
        )
    }
}

fig <- fig %>% layout(
    title=list(text=\"Red Bayesiana — inferencia bajo distintas evidencias (bnlearn + gRain)<br><sub>Comparing marginals across evidence scenarios</sub>\",
               font=list(size=13)),
    barmode=\"group\", plot_bgcolor=\"#f8f9fa\", height=380,
    xaxis=list(domain=c(0,.18),title=\"Bach\"),
    xaxis2=list(domain=c(.2,.38),title=\"Horas\"),
    xaxis3=list(domain=c(.40,.58),title=\"Parc1\"),
    xaxis4=list(domain=c(.60,.78),title=\"Parc2\"),
    xaxis5=list(domain=c(.80,1),title=\"Final\"),
    yaxis=list(title=\"Probabilidad\",range=c(0,1))
)
print(fig)
```

---

## 12. Propiedades y consideraciones prácticas

> [!tip] Propiedad 1 — Compresión exponencial de la distribución conjunta
> Una distribución conjunta sobre $n$ variables binarias requiere $2^n - 1$ parámetros. Una red bayesiana con $k$ padres máximos por nodo requiere solo $O(n \\cdot 2^k)$. Para $n = 20$ y $k = 3$: de $10^6$ a $160$ parámetros. La independencia condicional es el ingrediente que hace tractable la probabilidad en alta dimensión.

> [!tip] Propiedad 2 — Inferencia bidireccional
> La misma red soporta inferencia en dos direcciones: predictiva (de causas a efectos) y diagnóstica (de efectos a causas). Un médico puede usar la misma red tanto para predecir síntomas dado un diagnóstico como para inferir diagnóstico probable dados los síntomas — algo que requeriría dos modelos separados en un enfoque frecuentista.

> [!warning] Limitación — identificación causal requiere supuestos no testeables
> El do-calculus permite estimar efectos causales de datos observacionales, pero solo bajo el supuesto de que el DAG es correcto y no hay confundidores no observados. Estos supuestos no son verificables desde los datos solos — requieren conocimiento de dominio. La estructura del grafo es una hipótesis sobre el mecanismo generador, no un resultado estadístico.

> [!warning] Aprendizaje estructural — espacio enorme
> El número de DAGs sobre $n$ nodos crece más que exponencialmente: $1, 3, 25, 543, 29281, \\ldots$ para $n = 1, 2, 3, 4, 5, \\ldots$ La búsqueda exhaustiva es imposible para $n > 5$. Los algoritmos de hill-climbing con restarts aleatorios son el estándar práctico, con la advertencia de que pueden quedarse en óptimos locales.

---

## 13. Mapa conceptual

```
REDES BAYESIANAS
│
├── El modelo
│   ├── DAG G = (V, E): nodos = variables, aristas = dependencias directas
│   ├── CPDs: P(Xᵢ | Pa(Xᵢ)) — una distribución condicional por nodo
│   └── Factorización: P(X₁,...,Xₙ) = ∏ᵢ P(Xᵢ | Pa(Xᵢ))
│
├── Independencia condicional — d-separación
│   ├── Cadena X→Y→Z: X ⊥ Z | Y  (Y bloquea el flujo de info)
│   ├── Fork X←Y→Z: X ⊥ Z | Y   (Y explica la correlación)
│   └── Collider X→Y←Z: X ⊬ Z | Y  (condicionar en Y ABRE la dependencia)
│
├── Inferencia
│   ├── Eliminación de variables → exacta, O(n · exp(treewidth))
│   ├── Belief Propagation → exacta en árboles, approx. en grafos con ciclos
│   └── MCMC (sampling) → cuando la exacta no escala
│
├── Aprendizaje
│   ├── De parámetros: Dirichlet conjugado → posterior Dirichlet
│   └── Estructural: BDe score (= Factor de Bayes sobre grafos), Hill-Climbing
│
├── Extensiones
│   ├── DBNs → redes en tiempo discreto (generalización de HMMs)
│   └── SCM + do-calculus → causalidad, intervención, contrafactuales
│       ├── Asociación: P(Y|X)
│       ├── Intervención: P(Y|do(X))
│       └── Contrafactual: P(Yx|X',Y')
│
└── Conexiones en el vault
    ├── [[3. Familias Conjugadas]]   — Dirichlet-Categorical: conjugado para CPDs
    ├── [[7. Modelos Jerárquicos]]   — el DAG del modelo jerárquico es una BN
    ├── [[9. Selección de Modelos]]  — BDe score = evidencia marginal = Factor de Bayes
    └── [[11. Mezclas y Latentes]]   — HMM = DBN con una sola cadena latente
```

---

## 14. Conexiones y referencias cruzadas

> [!note] Notas relacionadas
> - [[3. Familias Conjugadas]] — el prior Dirichlet sobre las CPDs es la aplicación del par Dirichlet-Categorical a cada tabla condicional de la red
> - [[7. Modelos Jerárquicos Bayesianos]] — el DAG de un modelo jerárquico (hiperprior → parámetros → datos) es una red bayesiana; toda la maquinaria de inferencia es la misma
> - [[9. Selección y Comparación de Modelos]] — el BDe score para aprendizaje estructural es exactamente la evidencia marginal bayesiana del grafo; comparar grafos = calcular Factores de Bayes
> - [[11. Modelos de Mezclas y Variables Latentes]] — los HMM son DBNs con una sola variable latente en cadena; LDA es una red bayesiana con variables de tema latentes
> - [[14. Diseño Bayesiano de Experimentos]] — el diseño óptimo de experimentos puede formularse como encontrar la intervención $do(X)$ que maximiza la ganancia de información en el grafo causal

---

## Fuentes

- Pearl, J. — *Probabilistic Reasoning in Intelligent Systems: Networks of Plausible Inference* — Morgan Kaufmann, 1988
- Koller, D. & Friedman, N. — *Probabilistic Graphical Models: Principles and Techniques* — MIT Press, 2009
- Bishop, C. M. — *Pattern Recognition and Machine Learning* — **Cap. 8: Graphical Models** — Springer, 2006
- Pearl, J., Glymour, M. & Jewell, N. P. — *Causal Inference in Statistics: A Primer* — Wiley, 2016
- Darwiche, A. — *Modeling and Reasoning with Bayesian Networks* — Cambridge University Press, 2009
- Scutari, M. — *Learning Bayesian Networks with the bnlearn R Package* — Journal of Statistical Software, 35(3): 1–22, 2010
",
  "path": "C:\\Users\\ADMON\\Documents\\Obsidian Vault\\UNAL\\Bayesiana\\15. Redes Bayesianas.md"
}
