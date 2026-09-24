<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SROS // TRIADE SYSTEM - Módulo MMU</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body, html { width: 100%; height: 100%; overflow: hidden; background-color: #030308; font-family: 'Courier New', Courier, monospace; color: #ff9900; }
        #canvas-container { width: 100%; height: 100%; position: absolute; top: 0; left: 0; z-index: 1; }
        canvas { display: block; width: 100%; height: 100%; }
        .hud-overlay { position: absolute; top: 20px; left: 20px; z-index: 10; pointer-events: none; text-shadow: 0 0 5px #ff9900; background: rgba(12, 8, 4, 0.9); padding: 15px; border: 1px solid #ff9900; border-radius: 4px; box-shadow: 0 0 15px rgba(255, 153, 0, 0.2); max-width: 320px; }
        h1 { font-size: 14px; margin-bottom: 8px; border-bottom: 1px solid #ff9900; padding-bottom: 4px; text-transform: uppercase; }
        .telemetry-item { font-size: 11px; margin: 5px 0; display: flex; justify-content: space-between; }
        .status-active { color: #ff9900; animation: blink 1.5s infinite; }
        .status-alert { color: #ff3333; animation: blink 0.5s infinite; }
        .btn-action { position: absolute; bottom: 20px; left: 20px; z-index: 10; background: #221407; border: 1px solid #ff9900; color: #ff9900; padding: 10px 20px; font-family: inherit; font-size: 11px; cursor: pointer; text-shadow: 0 0 3px #ff9900; box-shadow: 0 0 10px rgba(255,153,0,0.1); border-radius: 4px; pointer-events: auto; }
        .btn-action:hover { background: #ff9900; color: #030308; font-weight: bold; }
        .legend { position: absolute; top: 20px; right: 20px; z-index: 10; background: rgba(12, 8, 4, 0.9); border: 1px solid #ff9900; padding: 10px; font-size: 10px; border-radius: 4px; }
        .legend-item { margin: 4px 0; display: flex; align-items: center; }
        .dot { width: 8px; height: 8px; border-radius: 50%; margin-right: 8px; display: inline-block; }
        @keyframes blink { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
    </style>
</head>
<body>

    <div id="canvas-container">
        <canvas id="mmuCanvas"></canvas>
    </div>

    <div class="hud-overlay">
        <h1>SROS // MMU PROPULSION</h1>
        <div class="telemetry-item"><span>ESTADO DO VETOR:</span> <span id="status-mmu" class="status-active">RCS STANDBY</span></div>
        <div class="telemetry-item"><span>PRESSÃO N₂:</span> <span id="pressao-val">3200 PSI</span></div>
        <div class="telemetry-item"><span>PROPORÇÃO PROPELENTE:</span> <span id="prop-val">100%</span></div>
        <div class="telemetry-item"><span>Δv DISPONÍVEL:</span> <span id="dv-val">24.5 m/s</span></div>
        <div class="telemetry-item"><span>VETOR SELECIONADO:</span> <span id="vetor-val">NENHUM</span></div>
    </div>

    <div class="legend">
        <div class="legend-item"><span class="dot" style="background:#ff9900;"></span>Sensores de Atitude/Giro</div>
        <div class="legend-item"><span class="dot" style="background:#ffffff;"></span>Jato de Expansão N₂ (Gás Frio)</div>
        <div class="legend-item"><span class="dot" style="background:#4444ff;"></span>Estabilizadores de Órbita</div>
    </div>

    <button class="btn-action" id="trigger-thrust">DISPARAR PULSO PROPULSOR (+X / LATERAL)</button>

    <script>
        const canvas = document.getElementById('mmuCanvas');
        const ctx = canvas.getContext('2d');

        function redimensionar() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', redimensionar);
        redimensionar();

        // Parâmetros do MMU
        let propulsionAtiva = false;
        let pressaoN2 = 3200;
        let propelentePct = 100.0;
        let deltaV = 24.5;
        let temporizadorJato = 0;

        // Partículas do escapamento de gás frio N2
        const particulasPlume = [];

        const pressaoEl = document.getElementById('pressao-val');
        const propEl = document.getElementById('prop-val');
        const dvEl = document.getElementById('dv-val');
        const vetorEl = document.getElementById('vetor-val');
        const statusEl = document.getElementById('status-mmu');

        function draw() {
            // Fundo espacial profundo e limpo
            ctx.fillStyle = '#03030d';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;

            // Lógica de consumo de propelente sob empuxo ativo
            if (propulsionAtiva && propelentePct > 0) {
                temporizadorJato++;
                pressaoN2 = Math.max(0, pressaoN2 - 8);
                propelentePct = (pressaoN2 / 3200) * 100;
                deltaV = (pressaoN2 / 3200) * 24.5;

                statusEl.innerText = "EMPULO RCS ATIVO";
                statusEl.className = "status-active";
                vetorEl.innerText = "+X (TRANS-LATERAL)";

                // Injeta partículas contínuas simulando a expansão mecânica do nitrogênio no vácuo
                for (let i = 0; i < 4; i++) {
                    particulasPlume.push({
                        x: centerX - 65, // Propulsor esquerdo acendendo para empurrar o traje para a direita (+X)
                        y: centerY + 20 + (Math.random() - 0.5) * 10,
                        vx: -4 - Math.random() * 4, // Gás expelido em alta velocidade para o lado oposto
                        vy: (Math.random() - 0.5) * 1.5,
                        vida: 1.0,
                        tamanho: 2 + Math.random() * 3
                    });
                }

                if (temporizadorJato > 45) { // Duração do pulso programado
                    propulsionAtiva = false;
                    temporizadorJato = 0;
                }
            } else {
                if (propelentePct <= 0) {
                    statusEl.innerText = "ALERTA: PROPELENTE ESGOTADO";
                    statusEl.className = "status-alert";
                    vetorEl.innerText = "NENHUM";
                } else {
                    statusEl.innerText = "RCS READY";
                    statusEl.className = "status-active";
                    statusEl.style.color = "#00ff66";
                    vetorEl.innerText = "NENHUM";
                }
            }

            // 1. DESENHAR INTERFACE VETORIAL DE ATITUDE (Giroscópio Circular)
            ctx.strokeStyle = 'rgba(255, 153, 0, 0.2)';
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.arc(centerX, centerY, 120, 0, Math.PI * 2);
            ctx.stroke();

            // Linhas de horizonte e mira artificial
            ctx.beginPath();
            ctx.moveTo(centerX - 140, centerY); ctx.lineTo(centerX + 140, centerY);
            ctx.moveTo(centerX, centerY - 140); ctx.lineTo(centerX, centerY + 140);
            ctx.stroke();

            // 2. SILHUETA DORSAL DA MOCHILA DE PROPULSÃO MMU
            ctx.strokeStyle = '#ff9900';
            ctx.lineWidth = 2;
            ctx.fillStyle = 'rgba(255, 153, 0, 0.05)';
            // Desenho simplificado da unidade dorsal traseira do traje
            ctx.fillRect(centerX - 60, centerY - 50, 120, 100);
            ctx.strokeRect(centerX - 60, centerY - 50, 120, 100);

            // Bocais dos propulsores laterais (Thruster blocks)
            ctx.fillRect(centerX - 70, centerY + 15, 10, 15);
            ctx.strokeRect(centerX - 70, centerY + 15, 10, 15);
            ctx.fillRect(centerX + 60, centerY + 15, 10, 15);
            ctx.strokeRect(centerX + 60, centerY + 15, 10, 15);

            // 3. ANIMAÇÃO DE EXPANSÃO DO EXPULSO DE GÁS COMPRIMIDO (NUVEM DE PLUME)
            particulasPlume.forEach((p, idx) => {
                p.x += p.vx;
                p.y += p.vy;
                p.vida -= 0.025; // Dissipa muito rápido simétrica ao vácuo

                // Tons de branco translúcido brilhante sumindo no escuro
                ctx.fillStyle = `rgba(255, 255, 255, ${p.vida * 0.8})`;
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.tamanho * (2 - p.vida), 0, Math.PI * 2); // O jato expande em diâmetro enquanto se dissipa
                ctx.fill();

                if (p.vida <= 0) {
                    particulasPlume.splice(idx, 1);
                }
            });

            // 4. ATUALIZAÇÃO DA TELEMETRIA TEXTUAL DO HUD
            pressaoEl.innerText = Math.floor(pressaoN2) + " PSI";
            propEl.innerText = propelentePct.toFixed(1) + "%";
            dvEl.innerText = deltaV.toFixed(2) + " m/s";

            requestAnimationFrame(draw);
        }

        // Gatilho do comando manual: Executa a injeção do pulso N2 estabilizado eletronicamente
        document.getElementById('trigger-thrust').addEventListener('click', () => {
            if (propelentePct > 0 && !propulsionAtiva) {
                propulsionAtiva = true;
                temporizadorJato = 0;
            }
        });

        draw();
    </script>
</body>
</html>
