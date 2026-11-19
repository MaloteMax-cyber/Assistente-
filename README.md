<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Meu Assistente</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; font-family: system-ui, sans-serif; }
    body { background: #e2e8f0; display: flex; justify-content: center; }
    .app { width: 100%; max-width: 400px; min-height: 100vh; background: #fff; display: flex; flex-direction: column; box-shadow: 0 0 20px rgba(0,0,0,0.1); }
    header { background: #2563eb; color: #fff; padding: 18px; text-align: center; font-weight: bold; font-size: 1.2rem; }
    main { flex: 1; padding: 16px; display: flex; flex-direction: column; gap: 12px; }
    .chat-box { flex: 1; background: #f8fafc; border-radius: 12px; padding: 12px; overflow-y: auto; border: 1px solid #e2e8f0; display: flex; flex-direction: column; gap: 8px; max-height: 300px; }
    .msg { padding: 10px 14px; border-radius: 12px; max-width: 80%; font-size: 0.95rem; }
    .bot { background: #e2e8f0; align-self: flex-start; color: #1e293b; }
    .user { background: #2563eb; color: #fff; align-self: flex-end; }
    .tools { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
    .tool-btn { background: #f1f5f9; color: #0f172a; border: 1px solid #cbd5e1; padding: 10px; border-radius: 8px; font-size: 0.85rem; cursor: pointer; font-weight: 600; }
    .input-area { display: flex; gap: 8px; }
    input { flex: 1; padding: 12px; border: 1px solid #cbd5e1; border-radius: 10px; outline: none; }
    .send-btn { padding: 12px 18px; background: #2563eb; color: #fff; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }
  </style>
</head>
<body>
  <div class="app">
    <header>Assistente Pessoal</header>
    <main>
      <div class="chat-box" id="chat">
        <div class="msg bot">Olá! Como posso ajudar? Use os botões abaixo ou digite um comando (ex: "hora", "calcular 15*4" ou "anotar comprar pão").</div>
      </div>

      <div class="tools">
        <button class="tool-btn" onclick="executarFerramenta('hora')">🕒 Ver Hora</button>
        <button class="tool-btn" onclick="executarFerramenta('notas')">📝 Ver Notas</button>
        <button class="tool-btn" onclick="executarFerramenta('bateria')">🔋 Bateria</button>
        <button class="tool-btn" onclick="executarFerramenta('limpar')">🧹 Limpar Chat</button>
      </div>

      <div class="input-area">
        <input type="text" id="comando" placeholder="Digite algo..." onkeypress="if(event.key==='Enter') enviar()">
        <button class="send-btn" onclick="enviar()">Enviar</button>
      </div>
    </main>
  </div>

  <script>
    const chat = document.getElementById('chat');
    const input = document.getElementById('comando');

    function adicionarMensagem(texto, tipo) {
      const div = document.createElement('div');
      div.className = `msg ${tipo}`;
      div.innerText = texto;
      chat.appendChild(div);
      chat.scrollTop = chat.scrollHeight;
    }

    function enviar() {
      const texto = input.value.trim();
      if (!texto) return;
      adicionarMensagem(texto, 'user');
      input.value = '';
      processarComando(texto.toLowerCase());
    }

    function processarComando(cmd) {
      if (cmd.includes('hora') || cmd.includes('data')) {
        const agora = new Date();
        adicionarMensagem(`Agora são ${agora.toLocaleTimeString('pt-BR')} do dia ${agora.toLocaleDateString('pt-BR')}.`, 'bot');
      } 
      else if (cmd.startsWith('calcular ')) {
        try {
          const conta = cmd.replace('calcular ', '');
          const resultado = Function('"use strict";return (' + conta + ')')();
          adicionarMensagem(`O resultado é: ${resultado}`, 'bot');
        } catch {
          adicionarMensagem('Não consegui calcular. Tente algo como: calcular 25 * 4', 'bot');
        }
      } 
      else if (cmd.startsWith('anotar ')) {
        const nota = cmd.replace('anotar ', '');
        let notas = JSON.parse(localStorage.getItem('minhas_notas') || '[]');
        notas.push(nota);
        localStorage.setItem('minhas_notas', JSON.stringify(notas));
        adicionarMensagem(`Anotado: "${nota}"`, 'bot');
      } 
      else if (cmd.includes('notas')) {
        const notas = JSON.parse(localStorage.getItem('minhas_notas') || '[]');
        adicionarMensagem(notas.length ? 'Suas notas: ' + notas.join(' | ') : 'Nenhuma nota salva.', 'bot');
      } 
      else {
        adicionarMensagem('Comando não reconhecido. Tente "hora", "calcular 10+10" ou "anotar texto".', 'bot');
      }
    }

    async function executarFerramenta(acao) {
      if (acao === 'hora') processarComando('hora');
      if (acao === 'notas') processarComando('notas');
      if (acao === 'limpar') chat.innerHTML = '';
      if (acao === 'bateria') {
        if ('getBattery' in navigator) {
          const bat = await navigator.getBattery();
          adicionarMensagem(`Nível da bateria: ${Math.round(bat.level * 100)}%`, 'bot');
        } else {
          adicionarMensagem('Seu navegador não permite ler a bateria.', 'bot');
        }
      }
    }
  </script>
</body>
</html>
