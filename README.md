<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>OPALBUD - Serwis v2.8</title>
    
    <!-- Podstawowe biblioteki -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700;900&display=swap');
        :root { --opal-green: #064e3b; --opal-bg: #f8fafc; }
        body { background-color: var(--opal-bg); font-family: 'Inter', sans-serif; margin: 0; -webkit-tap-highlight-color: transparent; }
        .card { background: white; border-radius: 24px; padding: 20px; border: 1px solid #e2e8f0; box-shadow: 0 4px 12px rgba(0,0,0,0.05); }
        .btn-primary { background: var(--opal-green); color: white; padding: 16px; border-radius: 16px; font-weight: bold; width: 100%; transition: transform 0.1s; }
        .btn-primary:active { transform: scale(0.98); }
        #signature-pad { border: 2px dashed #cbd5e1; background: #fff; touch-action: none; width: 100%; height: 240px; border-radius: 20px; }
        .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.8); z-index: 1000; align-items: center; justify-content: center; padding: 20px; backdrop-filter: blur(4px); }
        .modal.active { display: flex; }
    </style>
</head>
<body class="pb-20">

    <!-- Nagłówek -->
    <header class="bg-[#064e3b] text-white p-5 sticky top-0 z-50 rounded-b-[2rem] shadow-lg">
        <div class="max-w-xl mx-auto flex justify-between items-center">
            <div class="flex items-center gap-3">
                <div class="bg-white p-1 rounded-xl shadow-inner flex items-center justify-center h-12 min-w-[60px]">
                    <!-- Próba załadowania logo z obsługą spacji w nazwie pliku -->
                    <img src="agata%20logo%20(1).jpg" alt="Logo" style="max-height: 40px;" onerror="this.parentElement.innerHTML='<i class=\'fas fa-tools text-emerald-800 text-xl\'></i>'">
                </div>
                <div>
                    <h1 class="font-black text-xl leading-none uppercase tracking-tighter">Opalbud</h1>
                    <p class="text-[9px] text-emerald-300 font-bold uppercase tracking-widest mt-1">Panel Serwisowy</p>
                </div>
            </div>
            <button onclick="toggleModal('modal-mgr')" class="bg-emerald-900/50 w-10 h-10 rounded-full flex items-center justify-center text-emerald-200">
                <i class="fas fa-cog"></i>
            </button>
        </div>
    </header>

    <main class="max-w-xl mx-auto p-4 space-y-6">
        
        <!-- Formularz Dodawania -->
        <div class="card animate-fade-in">
            <div class="flex justify-between items-center mb-4">
                <h2 class="font-black text-slate-800 flex items-center gap-2"><i class="fas fa-edit text-emerald-500"></i> Nowy Raport</h2>
                <span id="clock" class="text-[10px] font-bold text-slate-400"></span>
            </div>

            <div class="space-y-4">
                <div>
                    <label class="text-[10px] font-black text-slate-400 uppercase ml-1">Wybierz Obiekt</label>
                    <select id="sel-obj" class="w-full p-4 bg-slate-50 rounded-2xl border-none outline-none font-bold text-slate-700 appearance-none shadow-inner"></select>
                </div>

                <div>
                    <label class="text-[10px] font-black text-slate-400 uppercase ml-1">Opis wykonanych prac</label>
                    <textarea id="txt-act" rows="4" placeholder="Opisz czynności serwisowe..." class="w-full p-4 bg-slate-50 rounded-2xl border-none outline-none text-sm shadow-inner resize-none"></textarea>
                </div>

                <div>
                    <label class="text-[10px] font-black text-slate-400 uppercase ml-1">Zdjęcia</label>
                    <div id="grid-photos" class="grid grid-cols-4 gap-2 my-2"></div>
                    <label class="flex flex-col items-center justify-center p-6 border-2 border-dashed border-slate-200 rounded-2xl bg-slate-50 cursor-pointer active:bg-emerald-50 transition-colors">
                        <input type="file" multiple accept="image/*" class="hidden" onchange="processImages(event)">
                        <i class="fas fa-camera text-2xl text-emerald-700 mb-1"></i>
                        <span class="text-[10px] font-black text-emerald-800">DODAJ ZDJĘCIA</span>
                    </label>
                </div>

                <button onclick="saveReport()" class="btn-primary flex items-center justify-center gap-2 shadow-xl shadow-emerald-900/20">
                    <i class="fas fa-check-circle"></i> ZAPISZ W PAMIĘCI
                </button>
            </div>
        </div>

        <!-- Lista Raportów -->
        <div class="space-y-4">
            <div class="flex justify-between items-center px-2">
                <h3 class="text-[10px] font-black text-slate-400 uppercase tracking-widest text-emerald-900">Ostatnie Wpisy</h3>
                <span id="count-badge" class="bg-emerald-100 text-emerald-700 text-[10px] font-black px-2 py-0.5 rounded-full">0</span>
            </div>
            <div id="list-reports" class="space-y-4"></div>
        </div>
    </main>

    <!-- Modal: Zarządzanie Obiektami -->
    <div id="modal-mgr" class="modal">
        <div class="bg-white w-full max-w-sm p-6 rounded-[2rem] shadow-2xl">
            <h3 class="font-black text-slate-800 mb-4">Lista Obiektów</h3>
            <div class="flex gap-2 mb-4">
                <input type="text" id="in-new-obj" placeholder="Nazwa..." class="flex-1 p-3 bg-slate-100 rounded-xl outline-none text-sm">
                <button onclick="addObject()" class="bg-emerald-600 text-white px-4 rounded-xl font-bold"><i class="fas fa-plus"></i></button>
            </div>
            <div id="list-mgr" class="space-y-2 max-h-60 overflow-y-auto pr-2 mb-4"></div>
            <button onclick="toggleModal('modal-mgr')" class="w-full p-4 bg-slate-100 text-slate-400 font-black text-[10px] rounded-2xl uppercase tracking-widest">Zamknij</button>
        </div>
    </div>

    <!-- Modal: Podpis -->
    <div id="modal-sig" class="modal">
        <div class="bg-white w-full max-w-sm p-6 rounded-[2rem] shadow-2xl text-center">
            <h3 class="font-black text-slate-800 mb-4 uppercase tracking-tighter">Podpis Zleceniodawcy</h3>
            <canvas id="signature-pad"></canvas>
            <div class="grid grid-cols-2 gap-3 mt-4">
                <button onclick="clearSig()" class="p-4 bg-slate-100 text-slate-400 font-bold rounded-2xl text-xs uppercase">Czyść</button>
                <button onclick="saveSig()" class="p-4 bg-emerald-600 text-white font-bold rounded-2xl text-xs uppercase shadow-lg shadow-emerald-600/30">Zatwierdź</button>
            </div>
        </div>
    </div>

    <!-- Template PDF (Ukryty) -->
    <div id="pdf-box" style="position: absolute; left: -9999px; width: 800px; padding: 60px; background: white; font-family: sans-serif; color: #000;">
        <div style="border-bottom: 4px solid #064e3b; padding-bottom: 20px; margin-bottom: 40px; display: flex; justify-content: space-between; align-items: center;">
            <div style="display: flex; align-items: center; gap: 20px;">
                <img src="agata%20logo%20(1).jpg" style="height: 80px;" onerror="this.style.display='none'">
                <div>
                    <h1 style="margin:0; font-size: 36px; font-weight: 900; color: #064e3b;">OPALBUD</h1>
                    <p style="margin:0; font-size: 14px; color: #666;">PROTOKÓŁ KONSERWACJI TECHNICZNEJ</p>
                </div>
            </div>
            <div style="text-align: right;"><p style="margin:0; font-size: 10px; color: #999;">NR RAPORTU:</p><p style="margin:0; font-weight: bold;" id="p-rid"></p></div>
        </div>
        <p><strong>DATA SERWISU:</strong> <span id="p-date"></span></p>
        <p><strong>OBIEKT:</strong> <span id="p-obj" style="font-size: 20px;"></span></p>
        <div style="background: #f8fafc; padding: 30px; border-radius: 20px; margin: 30px 0; border: 1px solid #eee;">
            <p style="font-size: 12px; color: #666; margin-bottom: 10px; font-weight: bold;">OPIS WYKONANYCH CZYNNOŚCI:</p>
            <p id="p-task" style="line-height: 1.6; font-size: 16px;"></p>
        </div>
        <div id="p-images" style="display: grid; grid-template-columns: 1fr 1fr; gap: 15px; margin-bottom: 40px;"></div>
        <div style="display: flex; justify-content: space-between; margin-top: 80px;">
            <div style="text-align: center; width: 45%; border-top: 1px solid #ccc; padding-top: 10px; font-size: 12px; font-weight: bold; color: #064e3b;">Pieczęć i podpis Konserwatora</div>
            <div style="text-align: center; width: 45%; border-top: 1px solid #ccc; padding-top: 10px; font-size: 12px; font-weight: bold; color: #064e3b;">
                <img id="p-sig" src="" style="max-height: 100px; display: block; margin: 0 auto 10px;">
                Podpis Zleceniodawcy
            </div>
        </div>
    </div>

    <div id="toast" class="fixed bottom-10 left-1/2 -translate-x-1/2 bg-emerald-950 text-white px-8 py-4 rounded-3xl text-xs font-black opacity-0 transition-all z-[2000] shadow-2xl pointer-events-none"></div>

    <script>
        // --- LOGIKA APLIKACJI v2.8 ---
        let data = {
            objs: JSON.parse(localStorage.getItem('opal_o')) || [{id:1, name:'Budynek Administracyjny'}],
            reps: JSON.parse(localStorage.getItem('opal_r')) || []
        };

        const sync = () => {
            localStorage.setItem('opal_o', JSON.stringify(data.objs));
            localStorage.setItem('opal_r', JSON.stringify(data.reps));
            render();
        };

        let currentPhotos = [], currentId = null;

        // Przetwarzanie zdjęć
        function processImages(e) {
            for (let f of e.target.files) {
                const reader = new FileReader();
                reader.onload = (ev) => {
                    const img = new Image();
                    img.src = ev.target.result;
                    img.onload = () => {
                        const canvas = document.createElement('canvas');
                        const scale = 600 / img.width;
                        canvas.width = 600; canvas.height = img.height * scale;
                        canvas.getContext('2d').drawImage(img, 0, 0, 600, canvas.height);
                        currentPhotos.push(canvas.toDataURL('image/jpeg', 0.6));
                        renderPhotos();
                    };
                };
                reader.readAsDataURL(f);
            }
        }

        function renderPhotos() {
            document.getElementById('grid-photos').innerHTML = currentPhotos.map((p,i) => `
                <div class="relative"><img src="${p}" class="w-full aspect-square object-cover rounded-xl border border-slate-200 shadow-sm">
                <button onclick="currentPhotos.splice(${i},1);renderPhotos()" class="absolute -top-1 -right-1 bg-red-500 text-white w-5 h-5 rounded-full text-[8px] flex items-center justify-center shadow-lg">X</button></div>`).join('');
        }

        // Akcje Raportu
        function saveReport() {
            const o = document.getElementById('sel-obj').value;
            const t = document.getElementById('txt-act').value;
            if(!o || !t) return showToast("⚠️ Uzupełnij opis!");
            
            data.reps.unshift({
                id: Math.random().toString(36).substr(2, 6).toUpperCase(),
                obj: o, task: t, photos: [...currentPhotos], date: new Date().toLocaleString(), signed: false
            });
            
            document.getElementById('txt-act').value = ""; currentPhotos = []; renderPhotos();
            sync();
            showToast("✅ Zapisano pomyślnie!");
        }

        function render() {
            document.getElementById('sel-obj').innerHTML = data.objs.map(o => `<option value="${o.name}">${o.name}</option>`).join('');
            document.getElementById('count-badge').innerText = data.reps.length;
            document.getElementById('list-reports').innerHTML = data.reps.map(r => `
                <div class="card p-6 flex flex-col gap-3">
                    <div class="flex justify-between items-start">
                        <div><h4 class="font-black text-emerald-950 text-sm uppercase leading-tight">${r.obj}</h4><p class="text-[10px] font-bold text-slate-400 mt-1">${r.date}</p></div>
                        <span class="text-[8px] font-black px-2 py-1 rounded-md ${r.signed ? 'bg-emerald-100 text-emerald-700':'bg-amber-100 text-amber-700'}">${r.signed ? 'PODPISANO':'CZEKA'}</span>
                    </div>
                    <p class="text-xs text-slate-600 italic">"${r.task.substring(0, 100)}${r.task.length > 100 ? '...':''}"</p>
                    <div class="flex gap-2">
                        ${r.signed ? `<button onclick="downloadPDF('${r.id}')" class="flex-1 bg-emerald-800 text-white py-4 rounded-2xl text-[10px] font-black uppercase shadow-lg shadow-emerald-900/20">Pobierz PDF</button>` : `<button onclick="openSignature('${r.id}')" class="flex-1 bg-white text-emerald-800 py-4 rounded-2xl text-[10px] font-black border-2 border-emerald-800">Podpis Klienta</button>`}
                        <button onclick="if(confirm('Usunąć?')){data.reps=data.reps.filter(x=>x.id!='${r.id}');sync()}" class="w-12 bg-red-50 text-red-500 rounded-2xl flex items-center justify-center"><i class="fas fa-trash"></i></button>
                    </div>
                </div>`).join('');
            
            document.getElementById('list-mgr').innerHTML = data.objs.map(o => `<div class="flex justify-between p-3 bg-slate-50 rounded-xl mb-1 text-sm font-bold"><span>${o.name}</span><button onclick="data.objs=data.objs.filter(x=>x.id!=${o.id});sync()" class="text-red-400"><i class="fas fa-times"></i></button></div>`).join('');
        }

        async function downloadPDF(id) {
            const r = data.reps.find(x => x.id === id);
            document.getElementById('p-rid').innerText = r.id;
            document.getElementById('p-date').innerText = r.date;
            document.getElementById('p-obj').innerText = r.obj;
            document.getElementById('p-task').innerText = r.task;
            document.getElementById('p-sig').src = r.signature || '';
            document.getElementById('p-images').innerHTML = r.photos.map(p => `<img src="${p}" style="width:100%; border-radius:10px; border:1px solid #eee;">`).join('');
            
            showToast("⚙️ Budowanie PDF...");
            setTimeout(async () => {
                const canv = await html2canvas(document.getElementById('pdf-box'), {scale: 2});
                const pdf = new jspdf.jsPDF('p', 'mm', 'a4');
                pdf.addImage(canv.toDataURL('image/jpeg', 0.9), 'JPEG', 0, 0, 210, (canv.height * 210) / canv.width);
                pdf.save(`RAPORT_${r.obj.replace(/\s/g, '_')}.pdf`);
            }, 600);
        }

        // Obsługa Podpisu (Canvas)
        const can = document.getElementById('signature-pad'); const ctx = can.getContext('2d'); let drawing = false;
        function openSignature(id) { currentId = id; toggleModal('modal-sig'); can.width = can.offsetWidth; can.height = can.offsetHeight; ctx.lineWidth = 3; ctx.lineCap = 'round'; ctx.strokeStyle = '#064e3b'; }
        const getPos = (e) => { const r = can.getBoundingClientRect(); const c = e.touches ? e.touches[0] : e; return { x: c.clientX - r.left, y: c.clientY - r.top }; };
        can.onmousedown = (e) => { drawing = true; ctx.beginPath(); const p = getPos(e); ctx.moveTo(p.x, p.y); };
        can.onmousemove = (e) => { if(drawing) { const p = getPos(e); ctx.lineTo(p.x, p.y); ctx.stroke(); } };
        window.onmouseup = () => drawing = false;
        can.ontouchstart = (e) => { e.preventDefault(); can.onmousedown(e); };
        can.ontouchmove = (e) => { e.preventDefault(); can.onmousemove(e); };

        function saveSig() { const r = data.reps.find(x => x.id === currentId); r.signed = true; r.signature = can.toDataURL(); sync(); toggleModal('modal-sig'); }
        function clearSig() { ctx.clearRect(0,0,can.width,can.height); }
        function addObject() { const v = document.getElementById('in-new-obj').value; if(v){ data.objs.push({id:Date.now(), name:v}); document.getElementById('in-new-obj').value=""; sync(); } }
        function toggleModal(id) { document.getElementById(id).classList.toggle('active'); }
        function showToast(m) { const t = document.getElementById('toast'); t.innerText = m; t.style.opacity = '1'; t.style.bottom = '30px'; setTimeout(() => { t.style.opacity = '0'; t.style.bottom = '10px'; }, 3000); }
        
        setInterval(() => { document.getElementById('clock').innerText = new Date().toLocaleTimeString(); }, 1000);
        window.onload = render;
    </script>
</body>
</html>
