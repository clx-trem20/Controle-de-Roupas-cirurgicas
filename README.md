<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Controle de Roupas - Centro Cirúrgico</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f8fafc;
        }
        .table-container {
            overflow-x: auto;
        }
        .loading-overlay {
            position: fixed;
            inset: 0;
            background: white;
            display: flex;
            justify-content: center;
            align-items: center;
            z-index: 2000;
        }
        #loginScreen {
            position: fixed;
            inset: 0;
            background-color: #f1f5f9;
            display: none;
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        #mainContent {
            display: none;
        }
        #settingsModal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.5);
            z-index: 1100;
            justify-content: center;
            align-items: center;
        }
        /* Estilo premium para inputs */
        input:focus {
            border-color: #2563eb;
            box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1);
        }
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- Loader de Inicialização -->
    <div id="initLoader" class="loading-overlay">
        <div class="text-center">
            <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
            <p class="text-slate-500 animate-pulse font-semibold">Conectando ao servidor seguro...</p>
        </div>
    </div>

    <!-- Tela de Login -->
    <div id="loginScreen">
        <div class="bg-white p-8 rounded-2xl shadow-xl border border-slate-200 w-full max-w-md transform transition-all">
            <div class="text-center mb-8">
                <div class="bg-blue-100 w-16 h-16 rounded-full flex items-center justify-center mx-auto mb-4">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                    </svg>
                </div>
                <h2 class="text-2xl font-bold text-slate-800">Acesso Restrito</h2>
                <p class="text-slate-500 text-sm">Controle de Roupas Cirúrgicas</p>
            </div>
            <form id="loginForm" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Usuário</label>
                    <input type="text" id="username" required placeholder="Digite o usuário" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 outline-none transition-all">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Senha</label>
                    <input type="password" id="password" required placeholder="Digite a senha" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 outline-none transition-all">
                </div>
                <div id="loginError" class="text-red-500 text-sm hidden font-medium text-center">Usuário ou senha incorretos.</div>
                <button type="submit" id="btnLogin" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 rounded-lg transition-all shadow-lg active:scale-95 disabled:opacity-50">
                    Entrar no Sistema
                </button>
            </form>
        </div>
    </div>

    <!-- Painel Principal -->
    <div id="mainContent" class="max-w-5xl mx-auto">
        <!-- Cabeçalho -->
        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200 flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">📊 Controle de Roupas</h1>
                <p class="text-slate-500 text-xs font-bold uppercase tracking-wider">Centro Cirúrgico - Gestão de Insumos</p>
            </div>
            
            <div class="flex items-center gap-2 flex-wrap">
                <button id="exportExcel" class="flex items-center gap-2 px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-lg transition-colors font-semibold text-sm shadow-sm">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" /></svg>
                    Exportar Excel
                </button>
                <button id="openSettings" class="p-2 bg-slate-100 hover:bg-slate-200 rounded-lg text-slate-600 transition-colors" title="Configurações">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z" /><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" /></svg>
                </button>
                <button id="btnLogoutAction" class="px-4 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg border border-red-100 font-semibold text-sm transition-colors">
                    Sair
                </button>
                <div class="bg-blue-50 p-2 rounded-lg border border-blue-100 flex flex-col justify-center">
                    <label class="text-[10px] font-bold text-blue-600 uppercase block leading-none mb-1">Preço Unitário</label>
                    <div class="flex items-center">
                        <span class="text-xs font-bold text-slate-500 mr-1">R$</span>
                        <input type="number" id="unitPriceInput" step="0.01" value="70.00" class="w-16 bg-transparent font-bold text-slate-700 outline-none">
                    </div>
                </div>
            </div>
        </div>

        <!-- Formulário de Entrada -->
        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200">
            <h3 class="text-sm font-bold text-slate-700 mb-4 uppercase tracking-tight">Novo Registro</h3>
            <form id="entryForm" class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <div class="flex flex-col">
                    <label class="text-[10px] font-bold text-slate-400 uppercase mb-1 ml-1">Paciente</label>
                    <input type="text" id="patientName" required placeholder="Nome completo" class="px-4 py-2 rounded-lg border border-slate-200 outline-none w-full">
                </div>
                <div class="flex flex-col">
                    <label class="text-[10px] font-bold text-slate-400 uppercase mb-1 ml-1">Data</label>
                    <input type="date" id="entryDate" required class="px-4 py-2 rounded-lg border border-slate-200 outline-none w-full">
                </div>
                <div class="flex flex-col">
                    <label class="text-[10px] font-bold text-slate-400 uppercase mb-1 ml-1">Quantidade</label>
                    <input type="number" id="clothingQty" required min="1" value="1" class="px-4 py-2 rounded-lg border border-slate-200 outline-none w-full">
                </div>
                <div class="flex flex-col justify-end">
                    <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-lg shadow-md transition-all active:scale-95 h-[42px]">
                        Adicionar Registro
                    </button>
                </div>
            </form>
        </div>

        <!-- Tabela de Dados -->
        <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
            <div class="table-container">
                <table class="w-full text-left border-collapse" id="mainDataTable">
                    <thead>
                        <tr class="bg-slate-50 border-b border-slate-200">
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider">Paciente</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider">Data do Procedimento</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider text-center">Qtd</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider text-right">Valor Total</th>
                            <th class="px-6 py-4 text-xs font-bold text-slate-500 uppercase tracking-wider text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody" class="divide-y divide-slate-100">
                        <!-- Dados dinâmicos -->
                    </tbody>
                    <tfoot>
                        <tr class="bg-slate-50 font-bold border-t-2 border-slate-200">
                            <td colspan="3" class="px-6 py-5 text-right text-slate-600 uppercase text-xs tracking-widest">Total Geral Acumulado:</td>
                            <td id="grandTotal" class="px-6 py-5 text-right text-green-600 text-xl font-bold tracking-tight">R$ 0,00</td>
                            <td class="bg-slate-50"></td>
                        </tr>
                    </tfoot>
                </table>
            </div>
        </div>

        <!-- Rodapé de Ações -->
        <div class="mt-6 flex justify-between items-center">
            <p class="text-xs text-slate-400 font-bold uppercase tracking-tight">© 2026 – Criado por CLX</p>
            <button id="btnClearAll" class="text-xs font-bold text-red-400 hover:text-red-600 transition-colors flex items-center gap-1 uppercase tracking-tighter">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                Limpar Banco de Dados
            </button>
        </div>
    </div>

    <!-- Modal de Configurações (Gestão de Usuários) -->
    <div id="settingsModal">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl overflow-hidden max-h-[90vh] flex flex-col border border-slate-200">
            <div class="p-6 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                <div>
                    <h3 class="text-xl font-bold text-slate-800">Painel de Configurações</h3>
                    <p class="text-xs text-slate-500">Gestão de acessos ao sistema</p>
                </div>
                <button id="closeSettings" class="text-slate-400 hover:text-slate-600 transition-colors text-3xl font-light">&times;</button>
            </div>
            
            <div class="p-6 overflow-y-auto space-y-8">
                <!-- Adicionar Usuário -->
                <div>
                    <h4 class="text-sm font-bold text-slate-700 mb-4 uppercase tracking-wider flex items-center gap-2">
                        <span class="w-2 h-2 bg-blue-500 rounded-full"></span>
                        Cadastrar Novo Acesso
                    </h4>
                    <form id="newUserForm" class="grid grid-cols-1 md:grid-cols-3 gap-3">
                        <input type="text" id="newUsername" required placeholder="Nome de Usuário" class="px-3 py-2 border border-slate-200 rounded-lg outline-none text-sm">
                        <input type="text" id="newPassword" required placeholder="Senha de Acesso" class="px-3 py-2 border border-slate-200 rounded-lg outline-none text-sm">
                        <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 rounded-lg transition-all text-sm shadow-sm active:scale-95">
                            Salvar Usuário
                        </button>
                    </form>
                </div>

                <!-- Lista de Usuários -->
                <div>
                    <h4 class="text-sm font-bold text-slate-700 mb-4 uppercase tracking-wider flex items-center gap-2">
                        <span class="w-2 h-2 bg-green-500 rounded-full"></span>
                        Usuários com Acesso
                    </h4>
                    <div class="border border-slate-100 rounded-xl overflow-hidden shadow-sm">
                        <table class="w-full text-sm">
                            <thead class="bg-slate-50">
                                <tr class="text-slate-500 border-b border-slate-100">
                                    <th class="p-3 text-left font-semibold">Login</th>
                                    <th class="p-3 text-left font-semibold">Senha</th>
                                    <th class="p-3 text-center font-semibold">Ação</th>
                                </tr>
                            </thead>
                            <tbody id="userTableBody" class="divide-y divide-slate-50">
                                <!-- Users dinâmicos -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
            
            <div class="p-4 bg-slate-50 border-t border-slate-100 text-center">
                <p class="text-[10px] text-slate-400 uppercase font-bold tracking-widest">Sistema de Segurança Firebase Ativo</p>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, writeBatch, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // Prevenção de erro de inicialização se as globais não estiverem prontas
        const getFirebaseConfig = () => {
            try {
                return typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
            } catch (e) {
                return null;
            }
        };

        const config = getFirebaseConfig();
        if (!config) {
            console.error("Firebase config is missing.");
            document.body.innerHTML = "<div class='p-10 text-center text-red-600 font-bold'>Erro Crítico: Configuração do Banco de Dados não encontrada. Contate o administrador.</div>";
        } else {
            const app = initializeApp(config);
            const db = getFirestore(app);
            const auth = getAuth(app);
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

            let fbUser = null;
            let isAppStarted = false;
            let recordsData = [];

            // --- CONTROLE DE INTERFACE ---
            function setView(view) {
                document.getElementById('initLoader').style.display = 'none';
                document.getElementById('loginScreen').style.display = view === 'login' ? 'flex' : 'none';
                document.getElementById('mainContent').style.display = view === 'app' ? 'block' : 'none';
            }

            // --- SISTEMA DE AUTENTICAÇÃO ---
            const handleAuth = async () => {
                try {
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch (e) {
                    console.error("Auth Error:", e);
                    // Fallback para usuários em ambiente sem token customizado (retry manual)
                    setTimeout(handleAuth, 2000);
                }
            };

            onAuthStateChanged(auth, (user) => {
                if (user) {
                    fbUser = user;
                    const isLogged = localStorage.getItem('clothes_system_auth') === 'true';
                    if (isLogged) {
                        setView('app');
                        startApp();
                    } else {
                        setView('login');
                    }
                }
            });

            // --- LOGIN ---
            document.getElementById('loginForm').onsubmit = async (e) => {
                e.preventDefault();
                if (!fbUser) return;

                const u = document.getElementById('username').value;
                const p = document.getElementById('password').value;
                const btn = document.getElementById('btnLogin');
                const err = document.getElementById('loginError');

                btn.disabled = true;
                err.classList.add('hidden');

                try {
                    const authRef = collection(db, 'artifacts', appId, 'public', 'data', 'auth');
                    const snap = await getDocs(authRef);
                    let access = (u === "CLX" && p === "02072007");

                    if (!access) {
                        snap.forEach(d => {
                            if (d.data().username === u && d.data().password === p) access = true;
                        });
                    }

                    if (access) {
                        localStorage.setItem('clothes_system_auth', 'true');
                        setView('app');
                        startApp();
                    } else {
                        err.classList.remove('hidden');
                    }
                } catch (err) {
                    console.error(err);
                    alert("Erro ao verificar credenciais. Verifique sua conexão.");
                } finally {
                    btn.disabled = false;
                }
            };

            document.getElementById('btnLogoutAction').onclick = () => {
                localStorage.removeItem('clothes_system_auth');
                setView('login');
            };

            // --- NÚCLEO DA APLICAÇÃO ---
            function startApp() {
                if (isAppStarted) return;
                isAppStarted = true;
                
                loadSettings();

                // Registros
                const regRef = collection(db, 'artifacts', appId, 'public', 'data', 'registros');
                onSnapshot(regRef, (snap) => {
                    const list = [];
                    snap.forEach(d => list.push({ id: d.id, ...d.data() }));
                    list.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));
                    recordsData = list;
                    renderMainTable(list);
                }, (err) => console.error("Snapshot error:", err));

                // Usuários
                const userRef = collection(db, 'artifacts', appId, 'public', 'data', 'auth');
                onSnapshot(userRef, (snap) => {
                    const body = document.getElementById('userTableBody');
                    body.innerHTML = `
                        <tr class="bg-blue-50/30">
                            <td class="p-3 font-bold text-blue-700">CLX (Administrador)</td>
                            <td class="p-3 text-slate-400 font-mono italic">Protegido</td>
                            <td class="p-3 text-center text-slate-300">-</td>
                        </tr>
                    `;
                    snap.forEach(d => {
                        const tr = document.createElement('tr');
                        tr.innerHTML = `
                            <td class="p-3 font-medium text-slate-700">${d.data().username}</td>
                            <td class="p-3 text-slate-500 font-mono">${d.data().password}</td>
                            <td class="p-3 text-center">
                                <button class="text-red-500 hover:text-red-700 transition-colors del-u-btn" data-id="${d.id}">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                                </button>
                            </td>
                        `;
                        body.appendChild(tr);
                    });
                    
                    document.querySelectorAll('.del-u-btn').forEach(b => {
                        b.onclick = async () => {
                            if (confirm("Revogar acesso deste usuário?")) {
                                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'auth', b.dataset.id));
                            }
                        };
                    });
                });
            }

            async function loadSettings() {
                try {
                    const snap = await getDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'main'));
                    if (snap.exists()) {
                        document.getElementById('unitPriceInput').value = snap.data().price.toFixed(2);
                    }
                } catch (e) { console.error("Config Load Error", e); }
            }

            document.getElementById('unitPriceInput').onchange = async (e) => {
                const val = parseFloat(e.target.value) || 0;
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'main'), { price: val });
            };

            function renderMainTable(data) {
                const body = document.getElementById('tableBody');
                const totalDisplay = document.getElementById('grandTotal');
                body.innerHTML = '';
                let grandTotal = 0;

                data.forEach(item => {
                    const price = item.priceAtTime || 70;
                    const subtotal = item.qty * price;
                    grandTotal += subtotal;

                    const tr = document.createElement('tr');
                    tr.className = "hover:bg-slate-50/50 transition-colors group";
                    tr.innerHTML = `
                        <td class="px-6 py-4 font-semibold text-slate-700">${item.name}</td>
                        <td class="px-6 py-4 text-slate-500">${item.date.split('-').reverse().join('/')}</td>
                        <td class="px-6 py-4 text-center font-bold text-slate-600">${item.qty}</td>
                        <td class="px-6 py-4 text-right font-bold text-slate-800">R$ ${subtotal.toLocaleString('pt-BR', {minimumFractionDigits: 2})}</td>
                        <td class="px-6 py-4 text-center">
                            <button class="opacity-0 group-hover:opacity-100 transition-opacity text-red-400 hover:text-red-600 del-rec-btn p-1" data-id="${item.id}">
                                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mx-auto" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                            </button>
                        </td>
                    `;
                    body.appendChild(tr);
                });

                totalDisplay.innerText = `R$ ${grandTotal.toLocaleString('pt-BR', {minimumFractionDigits: 2})}`;

                document.querySelectorAll('.del-rec-btn').forEach(btn => {
                    btn.onclick = async () => {
                        if (confirm("Deseja realmente excluir este registro?")) {
                            await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', btn.dataset.id));
                        }
                    };
                });
            }

            document.getElementById('entryForm').onsubmit = async (e) => {
                e.preventDefault();
                const btn = e.target.querySelector('button');
                btn.disabled = true;
                
                try {
                    await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), {
                        name: document.getElementById('patientName').value,
                        date: document.getElementById('entryDate').value,
                        qty: parseInt(document.getElementById('clothingQty').value),
                        priceAtTime: parseFloat(document.getElementById('unitPriceInput').value),
                        createdAt: Date.now()
                    });
                    
                    e.target.reset();
                    document.getElementById('entryDate').valueAsDate = new Date();
                } catch (err) {
                    console.error(err);
                } finally {
                    btn.disabled = false;
                }
            };

            document.getElementById('newUserForm').onsubmit = async (e) => {
                e.preventDefault();
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), {
                    username: document.getElementById('newUsername').value,
                    password: document.getElementById('newPassword').value
                });
                e.target.reset();
            };

            document.getElementById('exportExcel').onclick = () => {
                if (recordsData.length === 0) return alert("Não existem dados para exportar.");
                const table = document.getElementById("mainDataTable");
                const wb = XLSX.utils.table_to_book(table, { sheet: "Controle de Roupas" });
                XLSX.writeFile(wb, `Controle_Roupas_CC_${new Date().toLocaleDateString().replace(/\//g, '-')}.xlsx`);
            };

            document.getElementById('btnClearAll').onclick = async () => {
                if (confirm("CUIDADO: Esta ação apagará TODOS os registros permanentemente. Deseja continuar?")) {
                    const snap = await getDocs(collection(db, 'artifacts', appId, 'public', 'data', 'registros'));
                    const batch = writeBatch(db);
                    snap.forEach(d => batch.delete(d.ref));
                    await batch.commit();
                }
            };

            document.getElementById('openSettings').onclick = () => document.getElementById('settingsModal').style.display = 'flex';
            document.getElementById('closeSettings').onclick = () => document.getElementById('settingsModal').style.display = 'none';
            document.getElementById('entryDate').valueAsDate = new Date();

            handleAuth();
        }
    </script>
</body>
</html>
