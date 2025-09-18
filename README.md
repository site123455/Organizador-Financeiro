<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Organizador Financeiro</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
body { font-family: Arial; margin:0; padding:0; background:#f2f2f2;}
header { background:#4CAF50; color:white; text-align:center; padding:1rem;}
.container { max-width:800px; margin:20px auto; background:white; padding:20px; border-radius:10px; box-shadow:0 3px 8px rgba(0,0,0,.2);}
form { display:flex; flex-wrap:wrap; gap:10px; margin-bottom:20px;}
input, select, button { padding:10px; font-size:1rem; border:1px solid #ccc; border-radius:5px;}
button { background:#4CAF50; color:white; cursor:pointer; border:none; transition:.3s;}
button:hover { background:#45a049;}
table { width:100%; border-collapse:collapse; margin-top:15px;}
table, th, td { border:1px solid #ddd;}
th, td { text-align:center; padding:8px;}
th { background:#f4f4f4;}
.resumo { display:flex; justify-content:space-around; margin:20px 0;}
.resumo div { background:#eee; padding:15px; border-radius:10px; flex:1; margin:0 5px; text-align:center; font-weight:bold;}
canvas { margin-top:20px;}
</style>
</head>
<body>
<header><h1>💰 Organizador Financeiro</h1></header>
<div class="container">
<form id="form-transacao">
<select id="tipo">
<option value="ganho">Ganho</option>
<option value="despesa">Despesa</option>
</select>
<input type="text" id="descricao" placeholder="Descrição" required>
<input type="number" id="valor" placeholder="Valor (R$)" step="0.01" required>
<input type="text" id="categoria" placeholder="Categoria" required>
<button type="submit">Adicionar</button>
</form>
<div class="resumo">
<div id="ganhos">Ganhos: R$ 0,00</div>
<div id="despesas">Despesas: R$ 0,00</div>
<div id="saldo">Saldo: R$ 0,00</div>
</div>
<table>
<thead>
<tr>
<th>Tipo</th>
<th>Descrição</th>
<th>Categoria</th>
<th>Valor</th>
<th>Ação</th>
</tr>
</thead>
<tbody id="lista-transacoes"></tbody>
</table>
<canvas id="grafico"></canvas>
</div>

<script type="module">
  // Firebase SDK
  import { initializeApp } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-app.js";
  import { getDatabase, ref, push, onValue, remove } from "https://www.gstatic.com/firebasejs/9.22.2/firebase-database.js";

  // Firebase config
  const firebaseConfig = {
    apiKey: "AIzaSyC59lD35NPP7NnF37JZdHioX5ZQFtH1x18",
    authDomain: "organizacao-financeira-22cbd.firebaseapp.com",
    databaseURL: "https://organizacao-financeira-22cbd-default-rtdb.firebaseio.com",
    projectId: "organizacao-financeira-22cbd",
    storageBucket: "organizacao-financeira-22cbd.firebasestorage.app",
    messagingSenderId: "1094597462153",
    appId: "1:1094597462153:web:92bbb8b9ccc43f4732de58"
  };

  const app = initializeApp(firebaseConfig);
  const db = getDatabase(app);

  const form = document.getElementById('form-transacao');
  const lista = document.getElementById('lista-transacoes');
  const ganhosEl = document.getElementById('ganhos');
  const despesasEl = document.getElementById('despesas');
  const saldoEl = document.getElementById('saldo');
  const ctx = document.getElementById('grafico').getContext('2d');

  let transacoes = [];

  let grafico = new Chart(ctx, {
    type: 'pie',
    data: {
      labels: [],
      datasets: [{
        label: 'Distribuição por Categoria',
        data: [],
        backgroundColor: ['#4CAF50','#FF6384','#36A2EB','#FFCE56','#8E44AD','#E67E22']
      }]
    }
  });

  function atualizarResumo() {
    let ganhos = transacoes.filter(t => t.tipo === 'ganho').reduce((acc,t)=>acc+t.valor,0);
    let despesas = transacoes.filter(t => t.tipo === 'despesa').reduce((acc,t)=>acc+t.valor,0);
    let saldo = ganhos - despesas;
    ganhosEl.textContent = `Ganhos: R$ ${ganhos.toFixed(2)}`;
    despesasEl.textContent = `Despesas: R$ ${despesas.toFixed(2)}`;
    saldoEl.textContent = `Saldo: R$ ${saldo.toFixed(2)}`;
  }

  function atualizarLista() {
    lista.innerHTML = "";
    transacoes.forEach((t) => {
      let tr = document.createElement('tr');
      tr.innerHTML = `
        <td>${t.tipo}</td>
        <td>${t.descricao}</td>
        <td>${t.categoria}</td>
        <td>R$ ${t.valor.toFixed(2)}</td>
        <td><button onclick="remover('${t.id}')">❌</button></td>
      `;
      lista.appendChild(tr);
    });
  }

  function atualizarGrafico() {
    let categorias = {};
    transacoes.forEach(t => {
      if(t.tipo === 'despesa'){
        categorias[t.categoria] = (categorias[t.categoria] || 0) + t.valor;
      }
    });
    grafico.data.labels = Object.keys(categorias);
    grafico.data.datasets[0].data = Object.values(categorias);
    grafico.update();
  }

  function salvarFirebase(transacao){
    push(ref(db, 'transacoes'), transacao);
  }

  form.addEventListener('submit', e => {
    e.preventDefault();
    let tipo = document.getElementById('tipo').value;
    let descricao = document.getElementById('descricao').value;
    let valor = parseFloat(document.getElementById('valor').value);
    let categoria = document.getElementById('categoria').value;

    let transacao = {tipo, descricao, valor, categoria};
    salvarFirebase(transacao);
    form.reset();
  });

  window.remover = function(id){
    remove(ref(db, 'transacoes/' + id));
  }

  // Escuta mudanças em tempo real
  onValue(ref(db, 'transacoes'), (snapshot) => {
    const data = snapshot.val() || {};
    transacoes = Object.entries(data).map(([id,t]) => ({id, ...t}));
    atualizarResumo();
    atualizarLista();
    atualizarGrafico();
  });
</script>
</body>
</html>
