<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PORTADORES v2.0 - Multiplayer & Visibilidade</title>
    <!-- Firebase SDK (Compat para arquivo único em HTML) -->
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore-compat.js"></script>
    <style>
        :root {
            --bg-color: #0b0b0e;
            --card-bg: #14141c;
            --modal-bg: #1a1a26;
            --accent-red: #8b0000;
            --accent-purple: #6a1b9a;
            --accent-orange: #ff5500;
            --accent-gold: #d4af37;
            --accent-green: #10b981;
            --text-main: #e0e0e0;
            --text-muted: #a0a0b0;
            --border-color: #2a2a3d;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: 'Segoe UI', system-ui, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-main);
            padding: 20px;
            min-height: 100vh;
        }

        header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 2px solid var(--accent-red);
            padding-bottom: 15px;
            margin-bottom: 20px;
            flex-wrap: wrap;
            gap: 15px;
        }

        .role-selector {
            background-color: #1a1a28;
            border: 1px solid var(--accent-gold);
            padding: 8px 12px;
            border-radius: 6px;
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 0.9rem;
        }

        .role-selector select {
            background-color: #0d0d14;
            color: var(--accent-gold);
            border: 1px solid var(--border-color);
            padding: 4px 8px;
            border-radius: 4px;
            font-weight: bold;
            cursor: pointer;
        }

        .btn {
            background-color: var(--border-color);
            color: var(--text-main);
            border: 1px solid var(--border-color);
            padding: 8px 14px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 4px;
        }
        .btn:hover { filter: brightness(1.2); }
        .btn-main { background-color: var(--accent-red); color: white; border: none; }
        .btn-gold { background-color: var(--accent-gold); color: #000; border: none; }
        .btn-green { background-color: var(--accent-green); color: #000; border: none; }

        .tracker-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(360px, 1fr));
            gap: 20px;
        }

        .char-card {
            background-color: var(--card-bg);
            border: 1px solid var(--border-color);
            border-radius: 8px;
            padding: 15px;
            display: flex;
            flex-direction: column;
            gap: 10px;
        }

        .char-card.inimigo { border-color: var(--accent-red); }
        .char-card.oculto { opacity: 0.6; border-style: dashed; }

        .char-header {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 8px;
        }

        .badge {
            font-size: 0.7rem;
            padding: 2px 6px;
            border-radius: 3px;
            font-weight: bold;
            text-transform: uppercase;
        }
        .badge-aliado { background-color: var(--accent-green); color: #000; }
        .badge-inimigo { background-color: var(--accent-red); color: #fff; }
        
        .stat-bar { margin: 4px 0; }
        .stat-header { display: flex; justify-content: space-between; font-size: 0.85rem; margin-bottom: 4px; }
        .boxes { display: flex; gap: 4px; }
        .box {
            flex: 1;
            height: 18px;
            border: 1px solid var(--border-color);
            border-radius: 3px;
            background-color: #08080c;
        }
        .box.filled-vida { background-color: var(--accent-red); }
        .box.filled-sobrecarga { background-color: var(--accent-purple); }

        .visibility-controls {
            background-color: #0d0d14;
            border: 1px dashed var(--accent-gold);
            padding: 8px;
            border-radius: 6px;
            display: flex;
            flex-direction: column;
            gap: 6px;
            font-size: 0.78rem;
        }
        .vis-btn {
            background-color: #1a1a26;
            color: var(--text-main);
            border: 1px solid var(--border-color);
            padding: 4px 6px;
            border-radius: 3px;
            cursor: pointer;
            text-align: left;
        }
        .vis-btn.ativo { background-color: var(--accent-gold); color: #000; font-weight: bold; }

        .secret-info {
            background-color: #1a0000;
            border: 1px solid var(--accent-red);
            padding: 6px;
            border-radius: 4px;
            font-size: 0.8rem;
            color: #ffaaaa;
        }
    </style>
</head>
<body>

    <header>
        <div>
            <h1>PORTADORES <small style="font-size: 0.9rem; color: var(--accent-red);">v2.0 Online</small></h1>
            <p style="font-size: 0.8rem; color: var(--text-muted);">Modo Multiplayer com Controle de Visibilidade</p>
        </div>

        <!-- Seletor de Papel para testes -->
        <div class="role-selector">
            <label><strong>Visão Atual:</strong></label>
            <select id="roleSelect" onchange="mudarPapel(this.value)">
                <option value="mestre">👑 Mestre (Visão Total)</option>
                <option value="jogador">⚔️ Jogador (Visão Limitada)</option>
            </select>
        </div>

        <div>
            <button class="btn btn-green" id="btnCriar" onclick="adicionarFichaExemplo()">+ Criar Ficha Teste</button>
        </div>
    </header>

    <div class="tracker-grid" id="main-container"></div>

    <script>
        // 1. CONFIGURAÇÃO DO FIREBASE (Substitua pelos dados do seu projeto depois)
        const firebaseConfig = {
  		apiKey: "AIzaSyCbI6EawXqvDLY5oj8-DwrcOV88wGWTG_Y",
  		authDomain: "ferramenta-portadores.firebaseapp.com",
 		projectId: "ferramenta-portadores",
  		storageBucket: "ferramenta-portadores.firebasestorage.app",
  		messagingSenderId: "100507788348",
  		appId: "1:100507788348:web:9c4cbfd99322e05c134f39",
 		measurementId: "G-3Q9LF2RHXF"

        };

        // Inicializador em modo Local (se não houver Firebase configurado ainda)
        let db = null;
        let modoOnline = false;

        try {
            if (firebaseConfig.apiKey !== "SUA_API_KEY") {
                firebase.initializeApp(firebaseConfig);
                db = firebase.firestore();
                modoOnline = true;
                console.log("Conectado ao Firebase com sucesso!");
            }
        } catch (e) {
            console.warn("Rodando em modo Local / Demonstração.");
        }

        // State local para demonstração imediata
        let papelAtual = 'mestre'; // 'mestre' ou 'jogador'
        let fichas = JSON.parse(localStorage.getItem('portadores_online_mock')) || [
            {
                id: "1",
                nome: "Leo (Jogador)",
                tipo: "aliado",
                vida: 6,
                sobrecarga: 1,
                detalhes: "Faca de Caça, Lanterna",
                revelado: { tudo: true, nome: true, vida: true }
            },
            {
                id: "2",
                nome: "Criatura das Sombras",
                tipo: "inimigo",
                vida: 4,
                sobrecarga: 2,
                detalhes: "Fraqueza: Fogo. Ataque causa +2 de Sobrecarga.",
                revelado: { tudo: false, nome: false, vida: false }
            }
        ];

        function salvarLocal() {
            localStorage.setItem('portadores_online_mock', JSON.stringify(fichas));
            render();
        }

        function mudarPapel(novoPapel) {
            papelAtual = novoPapel;
            document.getElementById('btnCriar').style.display = (papelAtual === 'mestre') ? 'block' : 'none';
            render();
        }

        function toggleRevelacao(id, campo) {
            const char = fichas.find(f => f.id === id);
            if (!char) return;

            char.revelado[campo] = !char.revelado[campo];

            if (modoOnline && db) {
                db.collection("fichas").doc(id).update({ revelado: char.revelado });
            } else {
                salvarLocal();
            }
        }

        function adicionarFichaExemplo() {
            const novoInimigo = {
                id: Date.now().toString(),
                nome: "Infectado Nível " + Math.floor(Math.random() * 10),
                tipo: "inimigo",
                vida: 6,
                sobrecarga: 0,
                detalhes: "Movimentos erráticos no escuro.",
                revelado: { tudo: false, nome: false, vida: false }
            };
            fichas.push(novoInimigo);
            salvarLocal();
        }

        function render() {
            const container = document.getElementById('main-container');
            container.innerHTML = '';

            fichas.forEach(char => {
                const isMestre = (papelAtual === 'mestre');
                const isAliado = (char.tipo === 'aliado');
                
                // Regras de visibilidade para Jogadores
                if (!isMestre && !isAliado && !char.revelado.nome && !char.revelado.vida && !char.revelado.tudo) {
                    // Inimigo totalmente oculto para o jogador não aparece na tela!
                    return;
                }

                const nomeExibido = (isMestre || char.revelado.nome || isAliado) ? char.nome : "❓ Entidade Desconhecida";
                
                let cardHTML = `
                    <div class="char-card ${char.tipo} ${(!char.revelado.tudo && !isAliado) ? 'oculto' : ''}">
                        <div class="char-header">
                            <div>
                                <h3 style="color: #fff;">${nomeExibido}</h3>
                                <span class="badge ${isAliado ? 'badge-aliado' : 'badge-inimigo'}">${char.tipo}</span>
                            </div>
                        </div>

                        <!-- Barra de Vida -->
                        ${(isMestre || char.revelado.vida || isAliado) ? `
                            <div class="stat-bar">
                                <div class="stat-header"><span>VIDA</span><span>${char.vida} / 6</span></div>
                                <div class="boxes">
                                    ${Array.from({length: 6}, (_, i) => `<div class="box ${i < char.vida ? 'filled-vida' : ''}"></div>`).join('')}
                                </div>
                            </div>
                        ` : `
                            <div class="stat-bar" style="color: var(--text-muted); font-size: 0.8rem; font-style: italic;">
                                🔒 Condição física oculta pelo Mestre
                            </div>
                        `}

                        <!-- Detalhes da Ficha / Segredos -->
                        ${(isMestre || char.revelado.tudo || isAliado) ? `
                            <div style="font-size: 0.8rem; background-color: #0d0d14; padding: 6px; border-radius: 4px;">
                                <strong>Info:</strong> ${char.detalhes}
                            </div>
                        ` : ''}

                        <!-- Controles de Visibilidade exclusivos do Mestre -->
                        ${isMestre ? `
                            <div class="visibility-controls">
                                <strong>👁️ Liberar para os Jogadores:</strong>
                                <button class="vis-btn ${char.revelado.nome ? 'ativo' : ''}" onclick="toggleRevelacao('${char.id}', 'nome')">
                                    ${char.revelado.nome ? '✓ Nome Revelado' : '✗ Nome Oculto'}
                                </button>
                                <button class="vis-btn ${char.revelado.vida ? 'ativo' : ''}" onclick="toggleRevelacao('${char.id}', 'vida')">
                                    ${char.revelado.vida ? '✓ Vida Revelada' : '✗ Vida Oculta'}
                                </button>
                                <button class="vis-btn ${char.revelado.tudo ? 'ativo' : ''}" onclick="toggleRevelacao('${char.id}', 'tudo')">
                                    ${char.revelado.tudo ? '✓ Ficha Completa Revelada' : '✗ Ficha Oculta'}
                                </button>
                            </div>
                        ` : ''}
                    </div>
                `;
                container.innerHTML += cardHTML;
            });
        }

        // Conectividade em tempo real com Firebase (se ativado)
        if (modoOnline && db) {
            db.collection("fichas").onSnapshot((snapshot) => {
                fichas = [];
                snapshot.forEach((doc) => {
                    fichas.push({ id: doc.id, ...doc.data() });
                });
                render();
            });
        } else {
            render();
        }
    </script>
</body>
</html>
