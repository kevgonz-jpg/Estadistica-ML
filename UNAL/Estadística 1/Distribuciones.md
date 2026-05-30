# Continuas
## Normal

```dataviewjs
const container = this.container;

// 1. Crear la estructura de controles (HTML)
const controls = container.createDiv();
controls.style.display = "flex";
controls.style.gap = "20px";
controls.style.marginBottom = "15px";
controls.style.padding = "10px";
controls.style.background = "#222";
controls.style.borderRadius = "8px";


function createSlider(label, min, max, step, initial) {
    const wrapper = controls.createDiv();
    wrapper.style.display = "flex";
    wrapper.style.flexDirection = "column";
    
    const labelEl = wrapper.createEl("label", { text: label });
    labelEl.style.fontSize = "0.8em";
    labelEl.style.color = "#aaa";
    
    const slider = wrapper.createEl("input");
    slider.type = "range";
    slider.min = min;
    slider.max = max;
    slider.step = step;
    slider.value = initial;
    
    const valueDisp = wrapper.createEl("span", { text: initial });
    valueDisp.style.textAlign = "center";
    valueDisp.style.color = "#fff";
    
    return { slider, valueDisp };
}

const muCtrl = createSlider("Media (μ)", -3, 3, 0.1, 0);
const sigmaCtrl = createSlider("Desviación (σ)", 0.1, 2, 0.05, 1);

// 2. Lógica matemática
const n = 200;
const x = Array.from({length: n}, (_, i) => -5 + (i * 10) / (n - 1));

function normalPDF(x, mu, sigma) {
    return (1 / (sigma * Math.sqrt(2 * Math.PI))) * Math.exp(-Math.pow(x - mu, 2) / (2 * Math.pow(sigma, 2)));
}

// 3. Renderizado Inicial
const plotDiv = container.createDiv();

const data = [{
    x: x,
    y: x.map(v => normalPDF(v, 0, 1)),
    type: 'scatter',
    mode: 'lines',
    fill: 'tozeroy',
    line: { color: '#00eeff' },
    fillcolor: 'rgba(0, 238, 255, 0.2)'
}];

const layout = {
    margin: { t: 30, b: 40, l: 40, r: 20 },
    paper_bgcolor: 'rgba(0,0,0,0)',
    plot_bgcolor: 'rgba(0,0,0,0)',
    xaxis: { range: [-5, 5], gridcolor: '#333', tickfont: {color: '#888'} },
    yaxis: { range: [0, 1.2], gridcolor: '#333', tickfont: {color: '#888'} },
    showlegend: false
};

// Render original
window.renderPlotly(plotDiv, data, layout);

// 4. Eventos de actualización (¡CORREGIDO!)
const updatePlot = () => {
    // Capturar nuevos valores de los sliders
    const mu = parseFloat(muCtrl.slider.value);
    const sigma = parseFloat(sigmaCtrl.slider.value);
    
    // Actualizar los números de texto en la pantalla
    muCtrl.valueDisp.innerText = mu;
    sigmaCtrl.valueDisp.innerText = sigma;

    // Recalcular la curva
    data[0].y = x.map(v => normalPDF(v, mu, sigma));
    
    // Forzar a Plotly a reaccionar a los nuevos datos desde el objeto global window
    if (window.Plotly && window.Plotly.react) {
        // Buscamos el div interno que Plotly realmente usa para dibujar
        const targetDiv = plotDiv.querySelector('.js-plotly-plot') || plotDiv;
        window.Plotly.react(targetDiv, data, layout);
    } else {
        // Plan B: Si por algún motivo react no está disponible, redibujamos
        plotDiv.empty(); 
        window.renderPlotly(plotDiv, data, layout);
    }
};

// Asignar los eventos para que escuchen cuando mueves la barra
muCtrl.slider.addEventListener("input", updatePlot);
sigmaCtrl.slider.addEventListener("input", updatePlot);
```

## Gamma
```dataviewjs
(() => {
    const container = this.container;
    container.empty(); // Limpiamos el contenedor

    // --- 1. INTERFAZ (UI) ---
    const headerDiv = container.createDiv();
    headerDiv.style.marginBottom = "15px";
    headerDiv.createEl("h3", { text: "Distribución Gamma: Función de Densidad", style: "margin: 0; color: #fff;" });

    const controlsDiv = container.createDiv();
    controlsDiv.style.display = "grid";
    controlsDiv.style.gridTemplateColumns = "1fr 1fr";
    controlsDiv.style.gap = "20px";
    controlsDiv.style.marginBottom = "10px";
    controlsDiv.style.background = "rgba(0,0,0,0.2)";
    controlsDiv.style.padding = "15px";
    controlsDiv.style.borderRadius = "8px";

    // Slider k (Forma / Shape)
    const divK = controlsDiv.createDiv();
    const labelK = divK.createEl("label", { text: "Forma (k): 2.0", style: "display: block; color: #00eeff; font-weight: bold; margin-bottom: 5px;" });
    const sliderK = divK.createEl("input", { type: "range", min: "1", max: "10", step: "0.1", value: "2" });
    sliderK.style.width = "100%";

    // Slider Theta (Escala / Scale)
    const divTheta = controlsDiv.createDiv();
    const labelTheta = divTheta.createEl("label", { text: "Escala (θ): 2.0", style: "display: block; color: #ffaa00; font-weight: bold; margin-bottom: 5px;" });
    const sliderTheta = divTheta.createEl("input", { type: "range", min: "0.5", max: "5", step: "0.1", value: "2" });
    sliderTheta.style.width = "100%";

    // Display de Estadísticas Matemáticas
    const statsDiv = container.createDiv();
    statsDiv.style.display = "flex";
    statsDiv.style.justifyContent = "space-around";
    statsDiv.style.marginBottom = "15px";
    statsDiv.style.padding = "10px";
    statsDiv.style.background = "rgba(255,255,255,0.05)";
    statsDiv.style.borderRadius = "5px";
    statsDiv.style.color = "#ccc";
    const textMean = statsDiv.createEl("span", { text: "Media (μ = kθ): 4.00" });
    const textVar = statsDiv.createEl("span", { text: "Varianza (σ² = kθ²): 8.00" });

    // --- 2. EL MOTOR GRÁFICO (Canvas Nativo) ---
    const canvas = container.createEl("canvas");
    canvas.width = 800;
    canvas.height = 350;
    canvas.style.width = "100%";
    canvas.style.backgroundColor = "transparent";
    canvas.style.borderBottom = "1px solid #555";
    canvas.style.borderLeft = "1px solid #555";
    const ctx = canvas.getContext("2d");

    // --- 3. MATEMÁTICAS ---
    // Aproximación de Lanczos para la Función Gamma Γ(z)
    function gammaFunc(z) {
        const p = [
            0.99999999999980993, 676.5203681218851, -1259.1392167224028, 
            771.32342877765313, -176.61502916214059, 12.507343278224757, 
            -0.13857109526572012, 9.9843695780195716e-6, 1.5056327351493116e-7
        ];
        if (z < 0.5) return Math.PI / (Math.sin(Math.PI * z) * gammaFunc(1 - z));
        z -= 1;
        let x = p[0];
        for (let i = 1; i < 9; i++) x += p[i] / (z + i);
        let t = z + 7.5;
        return Math.sqrt(2 * Math.PI) * Math.pow(t, z + 0.5) * Math.exp(-t) * x;
    }

    // Función de Densidad de Probabilidad (PDF) de la Gamma
    function gammaPDF(x, k, theta) {
        if (x <= 0) x = 0.0001; // Evitar divisiones por cero matemáticas
        return (Math.pow(x, k - 1) * Math.exp(-x / theta)) / (Math.pow(theta, k) * gammaFunc(k));
    }

    // --- 4. RENDERIZADOR ---
    function draw() {
        let k = parseFloat(sliderK.value);
        let theta = parseFloat(sliderTheta.value);

        // Actualizar UI
        labelK.innerText = `Forma (k): ${k.toFixed(1)}`;
        labelTheta.innerText = `Escala (θ): ${theta.toFixed(1)}`;
        textMean.innerHTML = `<b>Media</b> (μ = kθ): ${(k * theta).toFixed(2)}`;
        textVar.innerHTML = `<b>Varianza</b> (σ² = kθ²): ${(k * theta * theta).toFixed(2)}`;

        // Limpiar lienzo
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const maxX = 30; // Fijamos el eje X para notar cómo se "estira" la curva
        const points = [];
        let maxY = 0.1;

        // 4.1 Generar datos numéricos
        for(let x = 0; x <= maxX; x += 0.1) {
            let y = gammaPDF(x, k, theta);
            if(y > maxY) maxY = y; // Auto-escalar el eje Y
            points.push({x, y});
        }

        maxY = maxY * 1.15; // Dar un poco de margen visual arriba

        // 4.2 Dibujar grilla vertical y números en el eje X
        ctx.strokeStyle = "rgba(255, 255, 255, 0.1)";
        ctx.lineWidth = 1;
        ctx.fillStyle = "#888";
        ctx.font = "12px Arial";
        
        for(let i = 0; i <= maxX; i += 5) {
            let px = (i / maxX) * canvas.width;
            ctx.beginPath(); 
            ctx.moveTo(px, 0); 
            ctx.lineTo(px, canvas.height); 
            ctx.stroke();
            if(i > 0) ctx.fillText(i, px - 5, canvas.height - 5);
        }

        // 4.3 Dibujar la curva Gamma
        ctx.beginPath();
        ctx.moveTo(0, canvas.height); // Iniciar en la esquina inferior izquierda

        for (let i = 0; i < points.length; i++) {
            let px = (points[i].x / maxX) * canvas.width;
            let py = canvas.height - (points[i].y / maxY) * canvas.height;
            ctx.lineTo(px, py);
        }

        // Color de línea
        ctx.strokeStyle = "#00eeff";
        ctx.lineWidth = 3;
        ctx.stroke();

        // 4.4 Efecto de relleno (Gradiente)
        ctx.lineTo((maxX / maxX) * canvas.width, canvas.height); // Bajar
        ctx.lineTo(0, canvas.height); // Cerrar figura
        ctx.closePath();
        
        let gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
        gradient.addColorStop(0, "rgba(0, 238, 255, 0.4)");
        gradient.addColorStop(1, "rgba(0, 238, 255, 0.01)");
        ctx.fillStyle = gradient;
        ctx.fill();
    }

    // --- 5. EVENTOS ---
    // Repintar cada vez que movemos un slider (a 60 FPS nativos)
    sliderK.addEventListener("input", draw);
    sliderTheta.addEventListener("input", draw);

    // Dibujo inicial
    draw();
})();
```

## Beta
```dataviewjs
(() => {
    const container = this.container;
    container.empty();

    // --- 1. INTERFAZ (UI) ---
    const headerDiv = container.createDiv();
    headerDiv.style.marginBottom = "15px";
    headerDiv.createEl("h3", { text: "Distribución Beta: Función de Densidad", style: "margin: 0; color: #fff;" });

    const controlsDiv = container.createDiv();
    controlsDiv.style.display = "grid";
    controlsDiv.style.gridTemplateColumns = "1fr 1fr";
    controlsDiv.style.gap = "20px";
    controlsDiv.style.marginBottom = "10px";
    controlsDiv.style.background = "rgba(0,0,0,0.2)";
    controlsDiv.style.padding = "15px";
    controlsDiv.style.borderRadius = "8px";

    // Slider Alfa (α)
    const divAlpha = controlsDiv.createDiv();
    const labelAlpha = divAlpha.createEl("label", { text: "Alfa (α): 2.00", style: "display: block; color: #ff3366; font-weight: bold; margin-bottom: 5px;" });
    const sliderAlpha = divAlpha.createEl("input", { type: "range", min: "0.1", max: "10", step: "0.01", value: "2" });
    sliderAlpha.style.width = "100%";

    // Slider Beta (β)
    const divBeta = controlsDiv.createDiv();
    const labelBeta = divBeta.createEl("label", { text: "Beta (β): 2.00", style: "display: block; color: #00eeff; font-weight: bold; margin-bottom: 5px;" });
    const sliderBeta = divBeta.createEl("input", { type: "range", min: "0.1", max: "10", step: "0.01", value: "2" });
    sliderBeta.style.width = "100%";

    // Display de Estadísticas Matemáticas
    const statsDiv = container.createDiv();
    statsDiv.style.display = "flex";
    statsDiv.style.justifyContent = "space-around";
    statsDiv.style.marginBottom = "15px";
    statsDiv.style.padding = "10px";
    statsDiv.style.background = "rgba(255,255,255,0.05)";
    statsDiv.style.borderRadius = "5px";
    statsDiv.style.color = "#ccc";
    const textMean = statsDiv.createEl("span", { text: "Media (μ): 0.50" });
    const textVar = statsDiv.createEl("span", { text: "Varianza (σ²): 0.05" });

    // --- 2. MOTOR GRÁFICO ---
    const canvas = container.createEl("canvas");
    canvas.width = 800;
    canvas.height = 350;
    canvas.style.width = "100%";
    canvas.style.backgroundColor = "transparent";
    canvas.style.borderBottom = "1px solid #555";
    canvas.style.borderLeft = "1px solid #555";
    const ctx = canvas.getContext("2d");

    // --- 3. MATEMÁTICAS ---
    function gammaFunc(z) {
        const p = [
            0.99999999999980993, 676.5203681218851, -1259.1392167224028, 
            771.32342877765313, -176.61502916214059, 12.507343278224757, 
            -0.13857109526572012, 9.9843695780195716e-6, 1.5056327351493116e-7
        ];
        if (z < 0.5) return Math.PI / (Math.sin(Math.PI * z) * gammaFunc(1 - z));
        z -= 1;
        let x = p[0];
        for (let i = 1; i < 9; i++) x += p[i] / (z + i);
        let t = z + 7.5;
        return Math.sqrt(2 * Math.PI) * Math.pow(t, z + 0.5) * Math.exp(-t) * x;
    }

    // Función Beta: B(a,b) = (Γ(a)*Γ(b)) / Γ(a+b)
    function betaFunc(a, b) {
        return (gammaFunc(a) * gammaFunc(b)) / gammaFunc(a + b);
    }

    // PDF de la distribución Beta
    function betaPDF(x, a, b) {
        // Evitamos 0 y 1 exactos para que las matemáticas no exploten si a < 1 o b < 1
        if (x <= 0) x = 0.0001; 
        if (x >= 1) x = 0.9999;
        return (Math.pow(x, a - 1) * Math.pow(1 - x, b - 1)) / betaFunc(a, b);
    }

    // --- 4. RENDERIZADOR ---
    function draw() {
        let a = parseFloat(sliderAlpha.value);
        let b = parseFloat(sliderBeta.value);

        // Actualizar Textos
        labelAlpha.innerText = `Alfa (α): ${a.toFixed(2)}`;
        labelBeta.innerText = `Beta (β): ${b.toFixed(2)}`;
        
        let mean = a / (a + b);
        let variance = (a * b) / (Math.pow(a + b, 2) * (a + b + 1));
        
        textMean.innerHTML = `<b>Media</b>: ${mean.toFixed(3)}`;
        textVar.innerHTML = `<b>Varianza</b>: ${variance.toFixed(4)}`;

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        const points = [];
        let maxY = 0.1;

        // Generar puntos entre 0 y 1
        for(let x = 0.001; x < 1; x += 0.002) {
            let y = betaPDF(x, a, b);
            if(y > maxY && y < 10) maxY = y; // Topamos el máximo en 10 para gráficas tipo "U"
            points.push({x, y});
        }

        maxY = maxY * 1.15; // Margen superior
        if (maxY < 2) maxY = 2; // Mantener una escala mínima para que no salte mucho

        // Dibujar grilla vertical (Eje X de 0 a 1)
        ctx.strokeStyle = "rgba(255, 255, 255, 0.1)";
        ctx.lineWidth = 1;
        ctx.fillStyle = "#888";
        ctx.font = "12px Arial";
        
        for(let i = 0; i <= 10; i++) {
            let val = i / 10;
            let px = val * canvas.width;
            ctx.beginPath(); 
            ctx.moveTo(px, 0); 
            ctx.lineTo(px, canvas.height); 
            ctx.stroke();
            if(i > 0 && i < 10) ctx.fillText(val.toFixed(1), px - 10, canvas.height - 5);
        }

        // Dibujar curva Beta
        ctx.beginPath();
        // Empezamos desde abajo a la izquierda
        ctx.moveTo(0, canvas.height);

        for (let i = 0; i < points.length; i++) {
            let px = points[i].x * canvas.width;
            let py = canvas.height - (points[i].y / maxY) * canvas.height;
            // Si la curva se sale por arriba (cuando a o b < 1), la cortamos visualmente
            if (py < -50) py = -50; 
            ctx.lineTo(px, py);
        }

        ctx.strokeStyle = "#ff3366";
        ctx.lineWidth = 3;
        ctx.stroke();

        // Relleno Gradiente
        ctx.lineTo(canvas.width, canvas.height); // Esquina inferior derecha
        ctx.lineTo(0, canvas.height); // Esquina inferior izquierda
        ctx.closePath();
        
        let gradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
        gradient.addColorStop(0, "rgba(255, 51, 102, 0.5)");
        gradient.addColorStop(1, "rgba(255, 51, 102, 0.05)");
        ctx.fillStyle = gradient;
        ctx.fill();
    }

    // --- 5. EVENTOS ---
    sliderAlpha.addEventListener("input", draw);
    sliderBeta.addEventListener("input", draw);

    // Dibujo inicial
    draw();
})();
```

# Discreta

---
