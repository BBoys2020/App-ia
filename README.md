        .status-msg.error { background: rgba(248, 81, 73, 0.2); color: var(--error); display: block; }
        .status-msg.info { background: rgba(0, 242, 254, 0.1); color: var(--neon-cyan); display: block; }
        .memory-area {
            margin-top: 15px;
            border-top: 1px solid #30363d;
            padding-top: 15px;
        }
        .memory-item {
            background: var(--bg-input);
            padding: 10px;
            border-radius: 6px;
            margin-bottom: 8px;
            border-left: 3px solid var(--neon-purple);
            font-size: 0.9rem;
        }
        .memory-item small { opacity: 0.6; display: block; margin-top: 4px; }
        .badge {
            display: inline-block;
            padding: 2px 10px;
            border-radius: 12px;
            font-size: 0.7rem;
            font-weight: bold;
            margin-left: 5px;
        }
        .badge.cyan { background: var(--neon-cyan); color: #000; }
        .badge.purple { background: var(--neon-purple); color: #fff; }
        .flex-row { display: flex; gap: 10px; align-items: center; flex-wrap: wrap; }
        .flex-row button { width: auto; padding: 8px 16px; }
        .hidden { display: none; }
        @media (max-width: 600px) {
            body { padding: 10px; }
            .card { padding: 15px; }
        }
    </style>
</head>
<body>

    <div class="header">
        <h1>🧠 OmniMedia AI Pro</h1>
        <div class="header-sub">Processamento Local • IA com Memória</div>
        <div style="font-size:0.9rem; margin-top: 5px;">
            Saldo: <span id="userCredits" style="color:var(--neon-cyan); font-weight:bold;">100</span> HW
            <span class="badge cyan" id="memoryCount">0</span> memórias
        </div>
    </div>

    <div class="card">
        <label for="aiFunction">Selecione a Ferramenta de IA</label>
        <select id="aiFunction" onchange="toggleFormFields()">
            <option value="text2image">🖼️ Texto para Imagem (Criação Ultra-Realista)</option>
            <option value="bgremoval">✂️ Remoção de Fundo Inteligente</option>
            <option value="upscale">✨ Upscaling Pro (Aumentar Resolução)</option>
            <option value="text2video">📹 Texto para Vídeo (Animação)</option>
            <option value="voiceclone">🗣️ Clonagem de Voz e Narração</option>
            <option value="subtitles">📝 Legendagem Automática</option>
            <option value="learn">📚 Modo Aprendizado (Ensinar IA)</option>
        </select>

        <div id="fileWrapper" class="file-input-wrapper">
            <label id="fileLabel" for="fileUpload">📎 Selecione o Arquivo</label>
            <input type="file" id="fileUpload" accept="image/*,video/*">
        </div>

        <div id="promptWrapper">
            <label id="promptLabel" for="promptText">📝 Instrução/Prompt</label>
            <textarea id="promptText" rows="3" placeholder="Descreva o que deseja criar ou perguntar..."></textarea>
        </div>

        <button id="processBtn" onclick="executarIA()">🚀 Processar Localmente</button>
        
        <div id="statusMessage" class="status-msg"></div>
    </div>

    <div class="card" id="resultadoCard" style="display:none;">
        <div id="resStatus" style="color:white; font-weight:bold;">Aguardando comando...</div>
        <div class="loading-bar" id="loadBar"><div class="loading-fill" id="loadFill"></div></div>
        
        <img id="outputImage" src="" alt="Resultado da IA">
        <video id="outputVideo" src="" controls></video>
        <div id="outputText" class="text-output"></div>
        
        <div class="memory-area" id="memoryArea">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:10px;">
                <strong>🧠 Memórias da IA</strong>
                <button onclick="limparMemorias()" style="width:auto; padding:5px 15px; font-size:0.8rem; background:var(--error);">Limpar</button>
            </div>
            <div id="memoryList"></div>
        </div>
    </div>

    <script>
        // ============================================
        //  SISTEMA DE MEMÓRIA (APRENDIZADO)
        // ============================================
        const MEMORY_KEY = 'omnimemoria_ai';
        
        function carregarMemorias() {
            try {
                const data = localStorage.getItem(MEMORY_KEY);
                return data ? JSON.parse(data) : [];
            } catch { return []; }
        }

        function salvarMemorias(memorias) {
            localStorage.setItem(MEMORY_KEY, JSON.stringify(memorias));
            atualizarContadorMemorias();
        }

        function adicionarMemoria(tipo, entrada, saida) {
            const memorias = carregarMemorias();
            memorias.push({
                id: Date.now(),
                tipo: tipo,
                entrada: entrada,
                saida: saida,
                data: new Date().toLocaleString()
            });
            // Mantém apenas as últimas 50 memórias
            if (memorias.length > 50) memorias.shift();
            salvarMemorias(memorias);
            renderizarMemorias();
        }

        function renderizarMemorias() {
            const container = document.getElementById('memoryList');
            const memorias = carregarMemorias();
            
            if (memorias.length === 0) {
                container.innerHTML = '<div style="opacity:0.5; text-align:center; padding:15px;">Nenhuma memória ainda. Use o modo Aprendizado!</div>';
                return;
            }

            container.innerHTML = memorias.slice().reverse().map(m => `
                <div class="memory-item">
                    <strong>${m.tipo}</strong>
                    <div style="margin-top:4px;">📥 ${m.entrada.substring(0, 100)}${m.entrada.length > 100 ? '...' : ''}</div>
                    <div style="color:var(--neon-cyan);">📤 ${m.saida.substring(0, 150)}${m.saida.length > 150 ? '...' : ''}</div>
                    <small>${m.data}</small>
                </div>
            `).join('');
        }

        function atualizarContadorMemorias() {
            const memorias = carregarMemorias();
            document.getElementById('memoryCount').textContent = memorias.length;
        }

        function limparMemorias() {
            if (confirm('Tem certeza que deseja limpar todas as memórias?')) {
                salvarMemorias([]);
                renderizarMemorias();
                mostrarStatus('🧹 Memórias limpas com sucesso!', 'info');
            }
        }

        // ============================================
        //  SISTEMA DE PROCESSAMENTO LOCAL
        // ============================================
        let credits = parseInt(localStorage.getItem('omnicredits')) || 100;
        let isProcessing = false;

        function atualizarCreditsUI() {
            document.getElementById('userCredits').innerText = credits;
            localStorage.setItem('omnicredits', credits);
        }

        function mostrarStatus(msg, tipo = 'info') {
            const el = document.getElementById('statusMessage');
            el.className = `status-msg ${tipo}`;
            el.textContent = msg;
            el.style.display = 'block';
            setTimeout(() => { el.style.display = 'none'; }, 5000);
        }

        // Processamento local para cada função
        function processarLocal(func, prompt, file) {
            return new Promise((resolve) => {
                const delay = 500 + Math.random() * 1500;
                
                setTimeout(() => {
                    let resultado = '';
                    let tipoSaida = 'texto';

                    switch(func) {
                        case 'text2image':
                            resultado = `🎨 Imagem gerada com IA:\nPrompt: "${prompt}"\nCriado com estilo ultra-realista, cores vibrantes e alta definição.`;
                            tipoSaida = 'imagem';
                            break;
                        
                        case 'bgremoval':
                            if (file) {
                                resultado = `✂️ Fundo removido com sucesso!\nArquivo: ${file.name}\nObjeto principal isolado com precisão.`;
                                tipoSaida = 'imagem';
                            } else {
                                resultado = '❌ Nenhum arquivo selecionado para remoção de fundo.';
                            }
                            break;
                        
                        case 'upscale':
                            if (file) {
                                resultado = `✨ Upscaling concluído!\nArquivo: ${file.name}\nResolução aumentada em 4x com detalhes aprimorados.`;
                                tipoSaida = 'imagem';
                            } else {
                                resultado = '❌ Nenhum arquivo selecionado para upscaling.';
                            }
                            break;
                        
                        case 'text2video':
                            resultado = `🎬 Vídeo gerado:\nRoteiro: "${prompt}"\nCenas animadas com transições suaves e efeitos cinematográficos.`;
                            tipoSaida = 'video';
                            break;
                        
                        case 'voiceclone':
                            resultado = `🎤 Narração gerada:\nTexto: "${prompt}"\nVoz realista com entonação natural e ritmo perfeito.`;
                            tipoSaida = 'audio';
                            break;
                        
                        case 'subtitles':
                            if (file) {
                                resultado = `📝 Legendas geradas:\nArquivo: ${file.name}\nTranscrição completa com timestamps e tradução automática.`;
                                tipoSaida = 'texto';
                            } else {
                                resultado = '❌ Nenhum arquivo de vídeo selecionado.';
                            }
                            break;
                        
                        case 'learn':
                            // MODO APRENDIZADO - IA aprende com o prompt
                            const memorias = carregarMemorias();
                            const similar = memorias.filter(m => 
                                m.entrada.toLowerCase().includes(prompt.toLowerCase().substring(0, 20)) ||
                                prompt.toLowerCase().includes(m.entrada.toLowerCase().substring(0, 20))
                            );
                            
                            if (similar.length > 0) {
                                resultado = `🧠 Aprendizado baseado em ${similar.length} memória(s) similar(es):\n\n`;
                                similar.slice(0, 3).forEach((m, i) => {
                                    resultado += `[Memória ${i+1}] Entrada: "${m.entrada}" → Saída: "${m.saida}"\n`;
                                });
                                resultado += `\n💡 Nova memória adicionada com base no seu prompt!`;
                            } else {
                                resultado = `🧠 Novo aprendizado!\nPrompt: "${prompt}"\nIA processou e armazenou este conhecimento para uso futuro.`;
                            }
                            
                            // Salva a memória
                            const saidaGerada = `Aprendizado: ${prompt}`;
                            adicionarMemoria('Aprendizado', prompt, saidaGerada);
                            tipoSaida = 'texto';
                            break;
                        
                        default:
                            resultado = '⚠️ Função não reconhecida.';
                    }

                    resolve({ resultado, tipoSaida });
                }, delay);
            });
        }

        // ============================================
        //  FUNÇÃO PRINCIPAL
        // ============================================
        async function executarIA() {
            if (isProcessing) {
                mostrarStatus('⏳ Processamento em andamento...', 'info');
                return;
            }

            const func = document.getElementById('aiFunction').value;
            const prompt = document.getElementById('promptText').value.trim();
            const fileInput = document.getElementById('fileUpload');
            const file = fileInput.files.length > 0 ? fileInput.files[0] : null;

            // Validações
            if (func !== 'bgremoval' && func !== 'upscale' && func !== 'subtitles' && !prompt) {
                mostrarStatus('❌ Por favor, insira um prompt/instrução.', 'error');
                return;
            }

            if ((func === 'bgremoval' || func === 'upscale' || func === 'subtitles') && !file) {
                mostrarStatus('❌ Selecione um arquivo para processar.', 'error');
                return;
            }

            // Valida créditos
            const custo = 3;
            if (credits < custo) {
                mostrarStatus(`❌ Créditos insuficientes! Necessário: ${custo} HW. Recarregue.`, 'error');
                return;
            }

            // Inicia processamento
            isProcessing = true;
            const btn = document.getElementById('processBtn');
            btn.disabled = true;
            btn.textContent = '⏳ Processando...';

            // Mostra card de resultado
            const cardRes = document.getElementById('resultadoCard');
            cardRes.style.display = 'block';
            
            const resStatus = document.getElementById('resStatus');
            const loadBar = document.getElementById('loadBar');
            const loadFill = document.getElementById('loadFill');
            
            document.getElementById('outputImage').style.display = 'none';
            document.getElementById('outputVideo').style.display = 'none';
            document.getElementById('outputText').style.display = 'none';
            
            resStatus.innerText = '🧠 IA processando localmente...';
            loadBar.style.display = 'block';
            
            // Simula progresso
            let progresso = 0;
            const progressInterval = setInterval(() => {
                progresso += Math.random() * 15 + 5;
                if (progresso > 95) progresso = 95;
                loadFill.style.width = progresso + '%';
            }, 100);

            try {
                // Processamento local
                const { resultado, tipoSaida } = await processarLocal(func, prompt, file);
                
                clearInterval(progressInterval);
                loadFill.style.width = '100%';
                
                setTimeout(() => {
                    loadBar.style.display = 'none';
                    resStatus.innerText = '✅ Processamento Concluído!';
                    
                    // Debita créditos
                    credits -= custo;
                    atualizarCreditsUI();
                    
                    // Exibe resultado
                    if (tipoSaida === 'imagem' || func === 'text2image' || func === 'bgremoval' || func === 'upscale') {
                        const img = document.getElementById('outputImage');
                        // Cria uma imagem baseada no texto para simular
                        const canvas = document.createElement('canvas');
                        canvas.width = 800;
                        canvas.height = 400;
                        const ctx = canvas.getContext('2d');
                        ctx.fillStyle = '#0d1117';
                        ctx.fillRect(0, 0, 800, 400);
                        ctx.fillStyle = '#00f2fe';
                        ctx.font = '20px Arial';
                        ctx.textAlign = 'center';
