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
    </style>
</head>
<body class="p-4 md:p-8">

    <!-- Loader de Inicialização -->
    <div id="initLoader" class="loading-overlay">
        <div class="text-center">
            <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600 mx-auto mb-4"></div>
            <p class="text-slate-500 animate-pulse">Conectando ao sistema seguro...</p>
        </div>
    </div>

    <!-- Tela de Login -->
    <div id="loginScreen">
        <div class="bg-white p-8 rounded-2xl shadow-xl border border-slate-200 w-full max-w-md">
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
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">Senha</label>
                    <input type="password" id="password" required placeholder="Digite a senha" 
                        class="w-full px-4 py-2 rounded-lg border border-slate-300 focus:ring-2 focus:ring-blue-500 outline-none">
                </div>
                <div id="loginError" class="text-red-500 text-sm hidden font-medium">Usuário ou senha incorretos.</div>
                <button type="submit" id="btnLogin" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 rounded-lg transition-all shadow-lg active:scale-95 disabled:opacity-50">
                    Entrar no Sistema
                </button>
            </form>
        </div>
    </div>

    <!-- Painel Principal -->
    <div id="mainContent" class="max-w-5xl mx-auto">
        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200 flex flex-col md:flex-row md:items-center justify-between gap-4">
            <div>
                <h1 class="text-2xl font-bold text-slate-800">📊 Controle de Roupas</h1>
                <p class="text-slate-500 text-xs font-bold uppercase">Painel de Gestão</p>
            </div>
            
            <div class="flex items-center gap-2 flex-wrap">
                <button id="exportExcel" class="flex items-center gap-2 px-4 py-2 bg-green-600 hover:bg-green-700 text-white rounded-lg transition-colors font-semibold text-sm">
                    Excel
                </button>
                <button id="openSettings" class="p-2 bg-slate-100 hover:bg-slate-200 rounded-lg text-slate-600">
                    ⚙️
                </button>
                <button id="btnLogoutAction" class="px-4 py-2 bg-red-50 hover:bg-red-100 text-red-600 rounded-lg border border-red-100 font-semibold text-sm">
                    Sair
                </button>
                <div class="bg-blue-50 p-2 rounded-lg border border-blue-100">
                    <label class="text-[10px] font-bold text-blue-600 uppercase block">Preço Un.</label>
                    <input type="number" id="unitPriceInput" step="0.01" value="70.00" class="w-16 bg-transparent font-bold text-slate-700 outline-none">
                </div>
            </div>
        </div>

        <div class="bg-white rounded-xl shadow-sm p-6 mb-6 border border-slate-200">
            <form id="entryForm" class="grid grid-cols-1 md:grid-cols-4 gap-4">
                <input type="text" id="patientName" required placeholder="Paciente" class="px-4 py-2 rounded-lg border outline-none">
                <input type="date" id="entryDate" required class="px-4 py-2 rounded-lg border outline-none">
                <input type="number" id="clothingQty" required min="1" value="1" class="px-4 py-2 rounded-lg border outline-none">
                <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 rounded-lg shadow-md">Adicionar</button>
            </form>
        </div>

        <div class="bg-white rounded-xl shadow-sm border border-slate-200 overflow-hidden">
            <div class="table-container">
                <table class="w-full text-left">
                    <thead class="bg-slate-50">
                        <tr>
                            <th class="px-6 py-3 text-sm font-semibold">Paciente</th>
                            <th class="px-6 py-3 text-sm font-semibold">Data</th>
                            <th class="px-6 py-3 text-sm font-semibold text-center">Qtd</th>
                            <th class="px-6 py-3 text-sm font-semibold text-right">Total</th>
                            <th class="px-6 py-3 text-sm font-semibold text-center">Ações</th>
                        </tr>
                    </thead>
                    <tbody id="tableBody"></tbody>
                    <tfoot class="bg-slate-50 font-bold border-t">
                        <tr>
                            <td colspan="3" class="px-6 py-4 text-right">TOTAL GERAL:</td>
                            <td id="grandTotal" class="px-6 py-4 text-right text-green-600 text-lg">R$ 0,00</td>
                            <td></td>
                        </tr>
                    </tfoot>
                </table>
            </div>
        </div>
    </div>

    <!-- Modal Configurações -->
    <div id="settingsModal">
        <div class="bg-white rounded-2xl shadow-2xl w-full max-w-2xl overflow-hidden max-h-[90vh] flex flex-col">
            <div class="p-6 border-b border-slate-100 flex justify-between items-center bg-slate-50">
                <h3 class="text-xl font-bold text-slate-800">Configurações de Acesso</h3>
                <button id="closeSettings" class="text-slate-400 hover:text-slate-600 text-2xl">&times;</button>
            </div>
            <div class="p-6 overflow-y-auto space-y-6">
                <form id="newUserForm" class="grid grid-cols-1 md:grid-cols-3 gap-3">
                    <input type="text" id="newUsername" required placeholder="Novo Usuário" class="px-3 py-2 border rounded-lg">
                    <input type="text" id="newPassword" required placeholder="Senha" class="px-3 py-2 border rounded-lg">
                    <button type="submit" class="bg-blue-600 text-white font-bold py-2 rounded-lg">Adicionar</button>
                </form>
                <div class="border rounded-lg overflow-hidden">
                    <table class="w-full text-sm">
                        <thead class="bg-slate-50"><tr><th class="p-2 text-left">Usuário</th><th class="p-2 text-left">Senha</th><th class="p-2">Ação</th></tr></thead>
                        <tbody id="userTableBody"></tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, deleteDoc, getDocs, writeBatch, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // Configuração
        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        let fbUser = null;
        let isAppStarted = false;

        // --- GESTÃO DE TELAS ---
        function showScreen(screenId) {
            document.getElementById('initLoader').style.display = 'none';
            document.getElementById('loginScreen').style.display = screenId === 'login' ? 'flex' : 'none';
            document.getElementById('mainContent').style.display = screenId === 'app' ? 'block' : 'none';
        }

        // --- AUTENTICAÇÃO E PERMISSÕES (Regra 3) ---
        const startAuth = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (err) {
                console.error("Erro na autenticação:", err);
            }
        };

        // Escutador de estado de autenticação garante que o Firestore só é acessado após o login
        onAuthStateChanged(auth, (user) => {
            if (user) {
                fbUser = user;
                const logged = localStorage.getItem('clothes_logged') === 'true';
                if (logged) {
                    showScreen('app');
                    if (!isAppStarted) startApp();
                } else {
                    showScreen('login');
                }
            }
        });

        // --- LOGIN ---
        document.getElementById('loginForm').onsubmit = async (e) => {
            e.preventDefault();
            if (!fbUser) {
                alert("O sistema ainda está conectando. Tente novamente em 2 segundos.");
                return;
            }

            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            
            document.getElementById('btnLogin').disabled = true;
            document.getElementById('loginError').classList.add('hidden');

            try {
                // Verificação de Usuários (Apenas após fbUser estar pronto)
                const authColl = collection(db, 'artifacts', appId, 'public', 'data', 'auth');
                const snap = await getDocs(authColl);
                let valid = (u === "CLX" && p === "02072007");

                if (!valid) {
                    snap.forEach(d => {
                        const data = d.data();
                        if (data.username === u && data.password === p) valid = true;
                    });
                }

                if (valid) {
                    localStorage.setItem('clothes_logged', 'true');
                    showScreen('app');
                    if (!isAppStarted) startApp();
                } else {
                    document.getElementById('loginError').classList.remove('hidden');
                }
            } catch (err) {
                console.error("Erro ao validar login:", err);
                alert("Erro de permissão no servidor. Tente atualizar a página.");
            } finally {
                document.getElementById('btnLogin').disabled = false;
            }
        };

        document.getElementById('btnLogoutAction').onclick = () => {
            localStorage.removeItem('clothes_logged');
            showScreen('login');
        };

        // --- APLICAÇÃO ---
        function startApp() {
            if (isAppStarted) return;
            isAppStarted = true;
            loadPrice();

            // Snapshot Registros
            const regRef = collection(db, 'artifacts', appId, 'public', 'data', 'registros');
            onSnapshot(regRef, (snap) => {
                const records = [];
                snap.forEach(d => records.push({ id: d.id, ...d.data() }));
                // Filtro em memória para evitar erros de índice (Regra 2)
                records.sort((a, b) => (b.createdAt || 0) - (a.createdAt || 0));
                renderTable(records);
            }, (err) => {
                console.error("Erro ao ler registros:", err);
            });

            // Snapshot Usuários
            const userRef = collection(db, 'artifacts', appId, 'public', 'data', 'auth');
            onSnapshot(userRef, (snap) => {
                const body = document.getElementById('userTableBody');
                body.innerHTML = '<tr><td class="p-2 font-bold">CLX (Master)</td><td class="p-2">********</td><td class="text-center">-</td></tr>';
                snap.forEach(d => {
                    const tr = document.createElement('tr');
                    tr.className = "border-t";
                    tr.innerHTML = `<td class="p-2">${d.data().username}</td><td class="p-2">${d.data().password}</td><td class="p-2 text-center"><button class="text-red-500 del-u" data-id="${d.id}">Remover</button></td>`;
                    body.appendChild(tr);
                });
                document.querySelectorAll('.del-u').forEach(btn => {
                    btn.onclick = async () => {
                        if (confirm("Apagar este usuário?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'auth', btn.dataset.id));
                    }
                });
            }, (err) => console.error("Erro ao ler usuários:", err));
        }

        async function loadPrice() {
            if (!fbUser) return;
            try {
                const snap = await getDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'price'));
                if (snap.exists()) document.getElementById('unitPriceInput').value = snap.data().value.toFixed(2);
            } catch (err) { console.error(err); }
        }

        document.getElementById('unitPriceInput').onchange = async (e) => {
            if (!fbUser) return;
            await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'config', 'price'), { value: parseFloat(e.target.value) || 0 });
        };

        function renderTable(recs) {
            const body = document.getElementById('tableBody');
            body.innerHTML = '';
            let total = 0;
            recs.forEach(r => {
                const sub = r.qty * (r.price || 70);
                total += sub;
                const tr = document.createElement('tr');
                tr.className = "border-b hover:bg-slate-50";
                tr.innerHTML = `
                    <td class="px-6 py-4 font-medium">${r.name}</td>
                    <td class="px-6 py-4">${r.date.split('-').reverse().join('/')}</td>
                    <td class="px-6 py-4 text-center">${r.qty}</td>
                    <td class="px-6 py-4 text-right font-bold">R$ ${sub.toFixed(2).replace('.', ',')}</td>
                    <td class="px-6 py-4 text-center"><button class="text-red-400 del-rec" data-id="${r.id}">Excluir</button></td>
                `;
                body.appendChild(tr);
            });
            document.getElementById('grandTotal').innerText = `R$ ${total.toFixed(2).replace('.', ',')}`;
            document.querySelectorAll('.del-rec').forEach(btn => {
                btn.onclick = async () => { if(confirm("Excluir registro?")) await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'registros', btn.dataset.id)); }
            });
        }

        document.getElementById('entryForm').onsubmit = async (e) => {
            e.preventDefault();
            if (!fbUser) return;
            const btn = e.target.querySelector('button');
            btn.disabled = true;
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'registros'), {
                    name: document.getElementById('patientName').value,
                    date: document.getElementById('entryDate').value,
                    qty: parseInt(document.getElementById('clothingQty').value),
                    price: parseFloat(document.getElementById('unitPriceInput').value),
                    createdAt: Date.now()
                });
                e.target.reset();
                document.getElementById('entryDate').valueAsDate = new Date();
            } catch (err) {
                console.error(err);
            } finally { btn.disabled = false; }
        };

        document.getElementById('newUserForm').onsubmit = async (e) => {
            e.preventDefault();
            if (!fbUser) return;
            await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'auth'), {
                username: document.getElementById('newUsername').value,
                password: document.getElementById('newPassword').value
            });
            e.target.reset();
        };

        // Exportar Excel simplificado
        document.getElementById('exportExcel').onclick = () => {
            const data = [['Paciente', 'Data', 'Qtd', 'Total']];
            const rows = document.querySelectorAll("#tableBody tr");
            rows.forEach(row => {
                const cols = row.querySelectorAll("td");
                data.push([cols[0].innerText, cols[1].innerText, cols[2].innerText, cols[3].innerText]);
            });
            const ws = XLSX.utils.aoa_to_sheet(data);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, "Relatório");
            XLSX.writeFile(wb, "Controle_Roupas.xlsx");
        };

        // Modais
        document.getElementById('openSettings').onclick = () => document.getElementById('settingsModal').style.display = 'flex';
        document.getElementById('closeSettings').onclick = () => document.getElementById('settingsModal').style.display = 'none';
        document.getElementById('entryDate').valueAsDate = new Date();

        // Inicializar Autenticação
        startAuth();
    </script>
</body>
</html>
