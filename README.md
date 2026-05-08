<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>OPALBUD - Panel Konserwatora (Lokalny)</title>
    
    <!-- Biblioteki zewnętrzne -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap');
        
        :root { 
            --opal-green: #064e3b; 
            --opal-bg: #f8fafc; 
        }

        body { 
            background-color: var(--opal-bg); 
            font-family: 'Inter', sans-serif; 
            -webkit-tap-highlight-color: transparent;
            overscroll-behavior-y: contain;
        }

        #signature-pad { 
            border: 2px dashed #cbd5e1; 
            background: #fff; 
            touch-action: none; 
            cursor: crosshair; 
        }

        .modal { 
            display: none; 
            position: fixed; 
            inset: 0; 
            background: rgba(0,0,0,0.8); 
            z-index: 1000; 
            align-items: center; 
            justify-content: center; 
            padding: 20px;
            backdrop-filter: blur(4px);
        }

        .modal.active { display: flex; }

        .btn-primary { 
            background-color: var(--opal-green); 
            color: white; 
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1); 
        }

        .btn-primary:active { 
            transform: scale(0.95); 
            background-color: #043d2e; 
        }

        .glass-card {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(6, 78, 59, 0.05);
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in { animation: fadeIn 0.3s ease-out forwards; }

        .ai-loading {
            background: linear-gradient(90deg, #ecfdf5 0%, #d1fae5 50%, #ecfdf5 100%);
            background-size: 200% 100%;
            animation: shimmer 1.5s infinite;
        }

        @keyframes shimmer {
            0% { background-position: 200% 0; }
            100% { background-position: -200% 0; }
        }

        .company-logo-img {
            max-height: 52px;
            width: auto;
            object-fit: contain;
        }
    </style>
</head>
<body class="pb-24">

    <!-- Ekran ładowania -->
    <div id="loading-overlay" class="fixed inset-0 bg-white z-[9999] flex flex-col items-center justify-center transition-opacity duration-500">
        <div class="relative w-20 h-20 mb-4">
            <div class="absolute inset-0 border-4 border-emerald-100 rounded-full"></div>
            <div class="absolute inset-0 border-4 border-emerald-800 rounded-full border-t-transparent animate-spin"></div>
        </div>
        <p class="font-bold text-emerald-900 tracking-tight">System OPALBUD v2.4 ✨</p>
    </div>

    <!-- Header -->
    <header class="bg-[#064e3b] text-white p-6 shadow-xl sticky top-0 z-50 rounded-b-[2.5rem]">
        <div class="max-w-xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-4">
                <div class="bg-white p-1.5 rounded-2xl shadow-lg flex items-center justify-center overflow-hidden h-14 w-auto min-w-[70px]">
                    <!-- Poprawiona nazwa pliku z kodowaniem spacji -->
                    <img src="agata%20logo%20(1).jpg" alt="Logo OPALBUD" class="company-logo-img" onerror="this.src='https://via.placeholder.com/100x60?text=LOGO';">
                </div>
                <div>
                    <h1 class="font-black text-2xl tracking-tighter leading-none uppercase">Opalbud</h1>
                    <p class="text-[10px] text-emerald-300 uppercase font-black tracking-[0.2em] mt-1">Tryb Lokalny</p>
                </div>
            </div>
            <div class="flex items-center gap-3">
                <button onclick="openManager()" class="w-10 h-10 flex items-center justify-center bg-emerald-900/40 rounded-full text-emerald-200 active:scale-90 transition-transform">
                    <i class="fas fa-list-check"></i>
                </button>
                <div id="status-dot" class="w-4 h-4 rounded-full bg-emerald-500 border-2 border-[#064e3b] shadow-inner"></div>
            </div>
        </div>
    </header>

    <main class="max-w-xl mx-auto p-5 space-y-8 mt-4">
        
        <!-- Sekcja: Nowy Protokół -->
        <section class="glass-card rounded-[2.5rem] shadow-xl p-7 animate-fade-in">
            <div class="flex items-center justify-between mb-6">
                <h2 class="text-emerald-900 font-black text-lg flex items-center gap-3">
                    <span class="w-8 h-8 bg-emerald-100 text-emerald-600 rounded-lg flex items-center justify-center text-sm">
                        <i class="fas fa-plus"></i>
                    </span>
                    Nowy Protokół
                </h2>
                <span id="current-time" class="text-[10px] font-bold text-slate-400 uppercase"></span>
            </div>

            <div class="space-y-5">
                <div class="space-y-1.5">
                    <label class="text-[11px] font-black text-slate-500 uppercase ml-1">Obiekt / Lokalizacja</label>
                    <div class="relative">
                        <select id="sel-object" class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none appearance-none font-bold text-slate-800 focus:border-emerald-500 transition-colors">
                            <option value="" disabled selected>Wybierz z listy...</option>
                        </select>
                        <i class="fas fa-chevron-down absolute right-4 top-1/2 -translate-y-1/2 text-slate-400 pointer-events-none text-xs"></i>
                    </div>
                </div>

                <div class="space-y-1.5">
                    <div class="flex justify-between items-center mb-1">
                        <label class="text-[11px] font-black text-slate-500 uppercase ml-1">Opis prac</label>
                        <button onclick="aiPolishDescription()" class="text-[9px] font-black bg-emerald-100 text-emerald-700 px-3 py-1.5 rounded-full hover:bg-emerald-200 transition-colors flex items-center gap-1">
                            <i class="fas fa-magic"></i> ✨ PROFESJONALNY OPIS
                        </button>
                    </div>
                    <textarea id="txt-activity" rows="4" placeholder="Opisz krótko czynności..." class="w-full p-4 bg-slate-50 border border-slate-200 rounded-2xl outline-none resize-none text-sm font-medium focus:border-emerald-500 transition-colors"></textarea>
                </div>

                <div class="space-y-3">
                    <div class="flex justify-between items-center mb-1">
                        <label class="text-[11px] font-black text-slate-500 uppercase ml-1">Zdjęcia</label>
                        <button id="ai-photo-btn" onclick="aiAnalyzePhotos()" class="hidden text-[9px] font-black bg-emerald-100 text-emerald-700 px-3 py-1.5 rounded-full hover:bg-emerald-200 transition-colors flex items-center gap-1">
                            <i class="fas fa-eye"></i> ✨ ANALIZA AI
                        </button>
                    </div>
                    <div id="photo-grid" class="grid grid-cols-4 gap-2 mb-2"></div>
                    <label class="flex items-center justify-center w-full py-6 border-2 border-dashed border-slate-200 rounded-2xl bg-slate-50 cursor-pointer active:bg-emerald-50 transition-all">
                        <input type="file" id="file-input" multiple accept="image/*" class="hidden" onchange="handleImg(event)">
                        <div class="text-center">
                            <i class="fas fa-camera text-emerald-700 text-2xl mb-2"></i>
                            <p class="text-[10px] font-black text-emerald-800 uppercase tracking-wider">Dodaj zdjęcia</p>
                        </div>
                    </label>
                </div>

                <button id="btn-submit" onclick="submitReport()" class="btn-primary w-full py-5 rounded-2xl font-black text-lg shadow-xl shadow-emerald-900/20 flex items-center justify-center gap-3 mt-4">
                    <i class="fas fa-save"></i> ZAPISZ RAPORT
                </button>
            </div>
        </section>

        <!-- Sekcja: Historia -->
        <section class="space-y-4 animate-fade-in" style="animation-delay: 0.1s">
            <div class="flex justify-between items-center px-4">
                <h3 class="font-black text-emerald-900 text-xs uppercase tracking-[0.15em]">Zapisane raporty</h3>
                <span id="badge-count" class="bg-emerald-800 text-white w-6 h-6 flex items-center justify-center rounded-full text-[10px] font-bold">0</span>
            </div>
            <div id="reports-list" class="space-y-4"></div>
        </section>
    </main>

    <!-- Modale -->
    <div id="modal-mgr" class="modal">
        <div class="bg-white w-full max-w-md rounded-[2.5rem] p-8 shadow-2xl flex flex-col max-h-[80vh]">
            <h3 class="font-black text-emerald-900 text-xl mb-6">Lista Obiektów</h3>
            <div class="flex gap-2 mb-6">
                <input type="text" id="new-obj" placeholder="Nowa lokalizacja..." class="flex-1 p-4 bg-slate-50 border border-slate-200 rounded-xl text-sm outline-none">
                <button onclick="addObj()" class="bg-emerald-700 text-white w-14 rounded-xl font-bold active:scale-90 transition-transform"><i class="fas fa-plus"></i></button>
            </div>
            <div id="mgr-list" class="space-y-3 overflow-y-auto no-scrollbar pr-1"></div>
            <button onclick="closeMod('modal-mgr')" class="w-full py-4 bg-slate-100 text-slate-500 rounded-2xl font-black text-[10px] mt-6 uppercase tracking-widest">Zamknij</button>
        </div>
    </div>

    <div id="modal-sig" class="modal">
        <div class="bg-white w-full max-w-md rounded-[2.5rem] p-8 shadow-2xl">
            <div class="text-center mb-6">
                <h3 class="font-black text-emerald-900 text-lg uppercase">Podpis Zleceniodawcy</h3>
                <p class="text-[10px] text-slate-400 font-bold mt-1 uppercase">Zatwierdzenie przeglądu</p>
            </div>
            <canvas id="signature-pad" class="w-full h-64 rounded-3xl mb-6 border-2 border-slate-100"></canvas>
            <div class="grid grid-cols-2 gap-4">
                <button onclick="clearSig()" class="py-4 border-2 border-slate-100 rounded-2xl text-slate-400 font-black text-[10px] uppercase">Wyczyść</button>
                <button onclick="saveSig()" class="py-4 bg-emerald-700 text-white rounded-2xl font-black text-[10px] uppercase">Zatwierdź</button>
            </div>
        </div>
    </div>

    <!-- Szablon PDF -->
    <div id="pdf-template" style="position: absolute; left: -9999px; width: 800px; padding: 60px; background: white; font-family: sans-serif;">
        <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 4px solid #064e3b; padding-bottom: 20px; margin-bottom: 40px;">
            <div style="display: flex; align-items: center; gap: 20px;">
                <img src="agata%20logo%20(1).jpg" alt="Logo" style="height: 90px; width: auto;">
                <div>
                    <h1 style="color: #064e3b; margin: 0; font-size: 38px; font-weight: 900;">OPALBUD</h1>
                    <p style="margin: 0; color: #666; letter-spacing: 2px; font-size: 14px;">SPÓŁKA Z OGRANICZONĄ ODPOWIEDZIALNOŚCIĄ</p>
                </div>
            </div>
            <div style="text-align: right;">
                <p style="margin: 0; font-size: 10px; color: #999; font-weight: bold; text-transform: uppercase;">Protokół Serwisowy</p>
                <p style="margin: 0; font-weight: bold; font-size: 14px; color: #064e3b;" id="pdf-rid"></p>
            </div>
        </div>
        <table style="width: 100%; border-collapse: collapse; margin-bottom: 30px;">
            <tr><td style="padding: 10px 0; font-size: 12px; color: #999; width: 150px;">DATA SERWISU</td><td style="padding: 10px 0; font-weight: bold;" id="pdf-date"></td></tr>
            <tr><td style="padding: 10px 0; font-size: 12px; color: #999;">OBIEKT / LOKALIZACJA</td><td style="padding: 10px 0; font-weight: bold; font-size: 18px;" id="pdf-obj"></td></tr>
        </table>
        <div style="background: #f8fafc; padding: 30px; border-radius: 20px; margin-bottom: 40px; border: 1px solid #eef2f6;">
            <p style="margin: 0 0 10px; font-size: 12px; color: #999; font-weight: bold;">Opis wykonanych czynności:</p>
            <p style="margin: 0; line-height: 1.6; color: #333; font-size: 15px;" id="pdf-task"></p>
        </div>
        <div id="pdf-photos" style="display: grid; grid-template-columns: repeat(2, 1fr); gap: 15px; margin-bottom: 80px;"></div>
        <div style="display: flex; justify-content: space-between; align-items: flex-end; gap: 60px; margin-top: 60px;">
            <div style="text-align: center; flex: 1;">
                <div style="height: 120px; border-bottom: 2px solid #064e3b; display: flex; align-items: center; justify-content: center; margin-bottom: 15px;">
                    <p style="color: #f1f5f9; font-size: 40px; font-weight: 900; transform: rotate(-5deg);">OPALBUD</p>
                </div>
                <p style="font-size: 11px; color: #064e3b; font-weight: 900; text-transform: uppercase;">Konserwator</p>
            </div>
            <div style="text-align: center; flex: 1;">
                <div style="height: 120px; border-bottom: 2px solid #064e3b; display: flex; align-items: center; justify-content: center; margin-bottom: 15px;">
                    <img id="pdf-sig-img" src="" style="max-height: 110px; max-width: 100%;">
                </div>
                <p style="font-size: 11px; color: #064e3b; font-weight: 900; text-transform: uppercase;">Zleceniodawca</p>
            </div>
        </div>
    </div>

    <div id="toast" class="fixed bottom-10 left-1/2 -translate-x-1/2 bg-emerald-950 text-white px-8 py-4 rounded-3xl text-xs font-black opacity-0 transition-all z-[2000] shadow-2xl pointer-events-none"></div>

    <script>
        // --- NOWA LOGIKA BEZ ZALEŻNOŚCI ---
        const apiKeyGemini = ""; 

        let state = {
            objects: JSON.parse(localStorage.getItem('opal_objects')) || [
                { id: '1', name: 'Kotłownia' },
                { id: '2', name: 'Winda' }
            ],
            reports: JSON.parse(localStorage.getItem('opal_reports')) || []
        };

        const saveState = () => {
            localStorage.setItem('opal_objects', JSON.stringify(state.objects));
            localStorage.setItem('opal_reports', JSON.stringify(state.reports));
            renderObjects();
            renderReports();
        };

        let tempPhotos = [], activeId = null;

        // Force hide loader even if images fail
        const hideLoader = () => {
            const loader = document.getElementById('loading-overlay');
            if (loader) {
                loader.classList.add('opacity-0');
                setTimeout(() => loader.style.display = 'none', 500);
            }
        };

        window.addEventListener('DOMContentLoaded', () => {
            renderObjects();
            renderReports();
            setTimeout(hideLoader, 1000);
        });

        // Gemini AI 
        async function callGemini(prompt, systemPrompt, imageData = null) {
            const endpoint = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKeyGemini}`;
            const payload = {
                contents: [{
                    parts: [
                        { text: prompt },
                        ...(imageData ? imageData.map(data => ({ inlineData: { mimeType: "image/jpeg", data: data.split(',')[1] } })) : [])
                    ]
                }],
                systemInstruction: { parts: [{ text: systemPrompt }] }
            };
            try {
                const response = await fetch(endpoint, { method: 'POST', body: JSON.stringify(payload) });
                const result = await response.json();
                return result.candidates?.[0]?.content?.parts?.[0]?.text || null;
            } catch (e) { return null; }
        }

        window.aiPolishDescription = async () => {
            const textarea = document.getElementById('txt-activity');
            if (!textarea.value.trim()) return showToast("⚠️ Wpisz tekst.");
            textarea.classList.add('ai-loading');
            showToast("✨ AI pracuje...");
            const polished = await callGemini(`Zamień na techniczny opis serwisu: "${textarea.value}"`, "Jesteś technikiem OPALBUD.");
            textarea.classList.remove('ai-loading');
            if (polished) { textarea.value = polished.trim(); showToast("✨ Gotowe!"); }
        };

        window.aiAnalyzePhotos = async () => {
            const textarea = document.getElementById('txt-activity');
            showToast("✨ AI analizuje zdjęcia...");
            const analysis = await callGemini("Co widzisz na zdjęciach? Opisz usterki.", "Jesteś ekspertem.", tempPhotos.slice(0, 3));
            if (analysis) { textarea.value += "\n\nAI: " + analysis; showToast("✨ Analiza dodana!"); }
        };

        // CRUD
        window.addObj = () => {
            const el = document.getElementById('new-obj');
            if (!el.value) return;
            state.objects.push({ id: Date.now().toString(), name: el.value.trim() });
            el.value = '';
            saveState();
        };

        window.delObj = (id) => {
            if(confirm("Usunąć?")) {
                state.objects = state.objects.filter(o => o.id !== id);
                saveState();
            }
        };

        window.handleImg = (e) => {
            const aiBtn = document.getElementById('ai-photo-btn');
            for (let f of e.target.files) {
                const rd = new FileReader();
                rd.onload = (ev) => { tempPhotos.push(ev.target.result); renderTempGrid(); aiBtn.classList.remove('hidden'); };
                rd.readAsDataURL(f);
            }
        };

        function renderTempGrid() {
            document.getElementById('photo-grid').innerHTML = tempPhotos.map((p, idx) => `
                <div class="relative animate-fade-in"><img src="${p}" class="w-full aspect-square object-cover rounded-xl border border-slate-200">
                <button onclick="removeTempPhoto(${idx})" class="absolute -top-1 -right-1 bg-red-500 text-white w-5 h-5 rounded-full text-[8px] flex items-center justify-center"><i class="fas fa-times"></i></button></div>`).join('');
        }

        window.removeTempPhoto = (idx) => {
            tempPhotos.splice(idx, 1);
            renderTempGrid();
            if (tempPhotos.length === 0) document.getElementById('ai-photo-btn').classList.add('hidden');
        };

        window.submitReport = () => {
            const obj = document.getElementById('sel-object').value;
            const tsk = document.getElementById('txt-activity').value;
            if(!obj || !tsk) return showToast("⚠️ Wypełnij formularz!");
            
            const newReport = {
                id: 'REP-' + Date.now().toString().slice(-6),
                object: obj, tsk: tsk, photos: tempPhotos, ts: Date.now(),
                date: new Date().toLocaleString('pl-PL'), signed: false
            };
            state.reports.unshift(newReport);
            document.getElementById('txt-activity').value = ''; 
            tempPhotos = []; 
            renderTempGrid();
            saveState();
            showToast("✅ Zapisano!");
        };

        window.openSig = (id) => { activeId = id; document.getElementById('modal-sig').classList.add('active'); setTimeout(initPad, 300); };
        
        window.saveSig = () => {
            const sig = canvas.toDataURL('image/png');
            const report = state.reports.find(r => r.id === activeId);
            if (report) { report.signed = true; report.signature = sig; saveState(); }
            closeMod('modal-sig');
            showToast("✍️ Podpisano.");
        };

        window.genPDF = async (id) => {
            const r = state.reports.find(x => x.id === id);
            document.getElementById('pdf-rid').innerText = r.id;
            document.getElementById('pdf-date').innerText = r.date;
            document.getElementById('pdf-obj').innerText = r.object;
            document.getElementById('pdf-task').innerText = r.tsk;
            document.getElementById('pdf-sig-img').src = r.signature || '';
            document.getElementById('pdf-photos').innerHTML = (r.photos || []).map(p => `<img src="${p}" style="width:100%; border-radius:10px;">`).join('');

            showToast("⚙️ Składanie PDF...");
            setTimeout(async () => {
                const canv = await html2canvas(document.getElementById('pdf-template'), { scale: 2 });
                const pdf = new jspdf.jsPDF('p', 'mm', 'a4');
                pdf.addImage(canv.toDataURL('image/jpeg', 0.95), 'JPEG', 0, 0, pdf.internal.pageSize.getWidth(), (canv.height * pdf.internal.pageSize.getWidth()) / canv.width);
                pdf.save(`PROTOKOL_${r.object.replace(/\s/g, '_')}.pdf`);
            }, 800);
        };

        function renderObjects() {
            const sel = document.getElementById('sel-object');
            const cur = sel.value;
            sel.innerHTML = '<option value="" disabled selected>Wybierz...</option>' + 
                state.objects.map(o => `<option value="${o.name}" ${cur === o.name ? 'selected' : ''}>${o.name}</option>`).join('');
            document.getElementById('mgr-list').innerHTML = state.objects.map(o => `
                <div class="flex justify-between items-center p-4 bg-slate-50 rounded-2xl border border-slate-100 animate-fade-in">
                    <span class="font-bold text-slate-700 text-sm">${o.name}</span>
                    <button onclick="window.delObj('${o.id}')" class="text-red-400 p-2"><i class="fas fa-trash"></i></button></div>`).join('');
        }

        function renderReports() {
            document.getElementById('badge-count').innerText = state.reports.length;
            document.getElementById('reports-list').innerHTML = state.reports.map(r => `
                <div class="glass-card p-6 rounded-[2.5rem] shadow-sm border border-slate-100 flex flex-col gap-4 animate-fade-in">
                    <div class="flex justify-between items-start">
                        <div><h4 class="font-black text-emerald-950 text-sm uppercase">${r.object}</h4><p class="text-[10px] text-slate-400 font-bold uppercase">${r.date}</p></div>
                        ${r.signed ? '<span class="bg-emerald-100 text-emerald-700 text-[8px] font-black px-2 py-1 rounded-md">OK</span>' : '<span class="bg-amber-100 text-amber-700 text-[8px] font-black px-2 py-1 rounded-md">BRAK PODPISU</span>'}
                    </div>
                    <p class="text-xs text-slate-600 italic">"${r.tsk}"</p>
                    <div class="flex gap-2">
                        ${r.signed ? `<button onclick="window.genPDF('${r.id}')" class="flex-1 bg-emerald-800 text-white py-4 rounded-2xl text-[10px] font-black uppercase"><i class="fas fa-file-pdf mr-2"></i> PDF</button>` : `<button onclick="window.openSig('${r.id}')" class="flex-1 bg-white text-emerald-800 py-4 rounded-2xl text-[10px] font-black uppercase border-2 border-emerald-800">PODPISZ</button>`}
                        <button onclick="deleteReport('${r.id}')" class="bg-red-50 text-red-500 w-12 rounded-2xl flex items-center justify-center"><i class="fas fa-trash"></i></button>
                    </div></div>`).join('');
        }

        window.deleteReport = (id) => { if(confirm("Usunąć?")) { state.reports = state.reports.filter(r => r.id !== id); saveState(); } };
        function showToast(m) { const t = document.getElementById('toast'); t.innerText = m; t.style.opacity = '1'; t.style.bottom = '40px'; setTimeout(() => { t.style.opacity = '0'; t.style.bottom = '10px'; }, 3000); }
        const canvas = document.getElementById('signature-pad'); const ctx = canvas.getContext('2d'); let drw = false;
        function initPad() {
            canvas.width = canvas.offsetWidth; canvas.height = canvas.offsetHeight;
            ctx.lineWidth = 3; ctx.strokeStyle = '#064e3b'; ctx.lineJoin = 'round'; ctx.lineCap = 'round';
            const getP = (e) => { const r = canvas.getBoundingClientRect(); const cx = e.touches ? e.touches[0].clientX : e.clientX; const cy = e.touches ? e.touches[0].clientY : e.clientY; return { x: cx - r.left, y: cy - r.top }; };
            const start = (e) => { drw = true; ctx.beginPath(); const p = getP(e); ctx.moveTo(p.x, p.y); e.preventDefault(); };
            const move = (e) => { if(!drw) return; const p = getP(e); ctx.lineTo(p.x, p.y); ctx.stroke(); e.preventDefault(); };
            canvas.onmousedown = start; canvas.onmousemove = move; window.onmouseup = () => drw = false;
            canvas.ontouchstart = start; canvas.ontouchmove = move; canvas.ontouchend = () => drw = false;
        }
        window.clearSig = () => ctx.clearRect(0, 0, canvas.width, canvas.height);
        window.openManager = () => document.getElementById('modal-mgr').classList.add('active');
        window.closeMod = (id) => document.getElementById(id).classList.remove('active');
        setInterval(() => { document.getElementById('current-time').innerText = new Date().toLocaleTimeString('pl-PL', { hour: '2-digit', minute: '2-digit' }); }, 1000);
    </script>
</body>
</html>
