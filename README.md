<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mapa de Batalhas do DF e Entorno</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" crossorigin=""></script>
    <style>
        body { font-family: 'Poppins', sans-serif; background-color: #f0f4f8; }
        .map-container { position: relative; width: 100%; height: 70vh; border-radius: 12px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); overflow: hidden; }
        #map { width: 100%; height: 100%; }
        .info-card { position: absolute; bottom: 20px; left: 20px; width: 320px; background-color: white; border-radius: 12px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15); padding: 20px; z-index: 1000; transform: translateY(20px); opacity: 0; transition: all 0.3s ease; pointer-events: none; }
        .info-card.active { transform: translateY(0); opacity: 1; pointer-events: all; }
        .close-btn { position: absolute; top: 10px; right: 10px; cursor: pointer; font-size: 18px; color: #555; }
        .admin-panel { display: none; background-color: white; border-radius: 12px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1); padding: 20px; margin-top: 20px; }
        .admin-panel.active { display: block; }
        .marker-icon { width: 36px; height: 36px; background-color: #ff4757; border: 3px solid white; border-radius: 50%; box-shadow: 0 0 10px rgba(0, 0, 0, 0.3); display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; transition: all 0.3s ease; }
        .marker-icon:hover { animation: markerPulse 2s infinite; transform: scale(1.1); }
        @keyframes markerPulse { 0% { box-shadow: 0 0 0 0 rgba(255, 71, 87, 0.7); } 70% { box-shadow: 0 0 0 10px rgba(255, 71, 87, 0); } 100% { box-shadow: 0 0 0 0 rgba(255, 71, 87, 0); } }
        .leaflet-popup-content-wrapper { border-radius: 12px; box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15); }
        .leaflet-popup-content { margin: 15px; }
        .instructions { background-color: #fff3cd; border-left: 4px solid #ffc107; padding: 10px 15px; margin-bottom: 15px; border-radius: 0 8px 8px 0; }
        .custom-marker { background-color: transparent; }
        .map-legend { position: absolute; bottom: 20px; right: 20px; background-color: white; padding: 10px; border-radius: 8px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.2); z-index: 1000; }
        .legend-item { display: flex; align-items: center; margin-bottom: 5px; }
        .legend-color { width: 20px; height: 20px; border-radius: 50%; margin-right: 8px; border: 2px solid white; box-shadow: 0 0 5px rgba(0, 0, 0, 0.2); }
        .credit-banner { position: absolute; bottom: 20px; left: 20px; background-color: rgba(255, 255, 255, 0.9); padding: 6px 10px; border-radius: 6px; box-shadow: 0 0 10px rgba(0, 0, 0, 0.15); font-size: 11px; z-index: 999; max-width: 200px; border-left: 3px solid #6c5ce7; }
        .credit-banner a { color: #6c5ce7; font-weight: 600; text-decoration: none; }
        .credit-banner a:hover { text-decoration: underline; }
        .battle-tooltip { background-color: white; border-radius: 8px; box-shadow: 0 2px 10px rgba(0, 0, 0, 0.15); padding: 10px; border-left: 4px solid #6c5ce7; max-width: 220px; }
        .battle-tooltip h3 { font-weight: 600; margin-bottom: 5px; font-size: 14px; }
        .battle-tooltip p { margin: 0 0 8px 0; font-size: 12px; color: #555; }
        .battle-tooltip .click-more { font-size: 10px; color: #888; font-style: italic; text-align: right; margin-top: 5px; }
        .leaflet-tooltip { background: transparent; border: none; box-shadow: none; padding: 0; }
        .leaflet-tooltip-top:before, .leaflet-tooltip-bottom:before, .leaflet-tooltip-left:before, .leaflet-tooltip-right:before { display: none; }
        .expired-event { opacity: 0.5; text-decoration: line-through; }
        .event-date-field { display: none; }
        .event-date-field.active { display: block; }
    </style>
</head>
<body>
    <div class="container mx-auto px-4 py-8">
        <header class="mb-8">
            <h1 class="text-4xl font-bold text-center text-gray-800">Mapa de Batalhas de Rima</h1>
            <p class="text-center text-gray-600 mt-2">Distrito Federal e Entorno</p>
        </header>
        
        <div class="map-container">
            <div id="map"></div>
            
            <div class="credit-banner">
                Em caso de alteração, reportar a <a href="https://instagram.com/joanamarcelly" target="_blank">@joanamarcelly</a>
            </div>
            
            <div class="map-legend">
                <h4 class="font-bold mb-2">Legenda</h4>
                <div class="legend-item"><div class="legend-color" style="background-color: #ff4757;"></div><span>Semanais</span></div>
                <div class="legend-item"><div class="legend-color" style="background-color: #ffa502;"></div><span>Quinzenais</span></div>
                <div class="legend-item"><div class="legend-color" style="background-color: #2ed573;"></div><span>Mensais</span></div>
                <div class="legend-item"><div class="legend-color" style="background-color: #1e90ff;"></div><span>Eventos Especiais</span></div>
            </div>
            
            <div class="info-card" id="infoCard">
                <span class="close-btn" id="closeBtn">&times;</span>
                <h3 class="text-xl font-bold mb-2" id="locationTitle">Nome da Batalha</h3>
                <div class="mb-3 flex items-center">
                    <svg class="w-5 h-5 mr-2 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z"></path><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z"></path></svg>
                    <p class="text-gray-600" id="locationAddress">Endereço</p>
                </div>
                <div class="mb-3 flex items-center">
                    <svg class="w-5 h-5 mr-2 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"></path></svg>
                    <p class="text-gray-600" id="locationTime">Horário</p>
                </div>
                <div class="mb-4 flex items-center">
                    <svg class="w-5 h-5 mr-2 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.828 10.172a4 4 0 00-5.656 0l-4 4a4 4 0 105.656 5.656l1.102-1.101m-.758-4.899a4 4 0 005.656 0l4-4a4 4 0 00-5.656-5.656l-1.1 1.1"></path></svg>
                    <a href="#" class="text-blue-500 hover:underline" id="locationIG" target="_blank">Instagram</a>
                </div>
                <p class="text-gray-700 text-sm" id="locationDesc">Descrição</p>
            </div>
        </div>
        
        <div class="mt-8 bg-white p-6 rounded-lg shadow-md">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-2xl font-bold text-gray-800">Batalhas do DF e Entorno</h2>
                <div class="space-x-2">
                    <button id="addPointBtn" class="bg-green-600 hover:bg-green-700 text-white py-2 px-4 rounded transition duration-300">Adicionar Ponto</button>
                    <button id="adminBtn" class="bg-purple-600 hover:bg-purple-700 text-white py-2 px-4 rounded transition duration-300">Gerenciar Batalhas</button>
                </div>
            </div>
            
            <div class="instructions" id="addPointInstructions" style="display: none;">
                <p class="font-medium">Modo de adição de ponto ativado!</p>
                <p>Clique em qualquer lugar no mapa para adicionar um novo ponto.</p>
            </div>
            
            <div class="overflow-x-auto">
                <table class="min-w-full bg-white" id="battlesTable">
                    <thead>
                        <tr>
                            <th class="py-3 px-4 border-b-2 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase">Batalha</th>
                            <th class="py-3 px-4 border-b-2 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase">Local</th>
                            <th class="py-3 px-4 border-b-2 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase">Dia/Horário</th>
                            <th class="py-3 px-4 border-b-2 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase">Ação</th>
                        </tr>
                    </thead>
                    <tbody id="battlesTableBody"></tbody>
                </table>
            </div>
        </div>
        
        <div class="admin-panel" id="adminPanel">
            <h2 class="text-2xl font-bold mb-4 text-gray-800">Adicionar/Editar Batalha</h2>
            <form id="battleForm" class="space-y-4">
                <input type="hidden" id="battleId">
                <input type="hidden" id="battleLat">
                <input type="hidden" id="battleLng">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div><label class="block text-sm font-medium mb-1">Nome da Batalha</label><input type="text" id="battleName" required class="w-full px-3 py-2 border rounded-md"></div>
                    <div><label class="block text-sm font-medium mb-1">Endereço</label><input type="text" id="battleAddress" required class="w-full px-3 py-2 border rounded-md"></div>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div><label class="block text-sm font-medium mb-1">Dia e Horário</label><input type="text" id="battleTime" required class="w-full px-3 py-2 border rounded-md"></div>
                    <div><label class="block text-sm font-medium mb-1">Instagram</label><input type="text" id="battleIG" class="w-full px-3 py-2 border rounded-md"></div>
                </div>
                <div><label class="block text-sm font-medium mb-1">Descrição</label><textarea id="battleDesc" rows="3" class="w-full px-3 py-2 border rounded-md"></textarea></div>
                <div>
                    <label class="block text-sm font-medium mb-1">Tipo de Batalha</label>
                    <select id="battleType" class="w-full px-3 py-2 border rounded-md">
                        <option value="weekly">Semanal</option>
                        <option value="biweekly">Quinzenal</option>
                        <option value="monthly">Mensal</option>
                        <option value="special">Evento Especial</option>
                    </select>
                </div>
                <div id="eventDateField" class="event-date-field">
                    <label class="block text-sm font-medium mb-1">Data do Evento (DD/MM/AAAA)</label>
                    <input type="date" id="eventDate" class="w-full px-3 py-2 border rounded-md">
                </div>
                <div class="flex justify-end space-x-3">
                    <button type="button" id="cancelBtn" class="px-4 py-2 bg-gray-300 rounded-md">Cancelar</button>
                    <button type="submit" class="px-4 py-2 bg-blue-600 text-white rounded-md">Salvar</button>
                </div>
            </form>
        </div>
    </div>

    <script>
        let battles = [];
        const infoCard = document.getElementById('infoCard');
        const closeBtn = document.getElementById('closeBtn');
        const locationTitle = document.getElementById('locationTitle');
        const locationAddress = document.getElementById('locationAddress');
        const locationTime = document.getElementById('locationTime');
        const locationIG = document.getElementById('locationIG');
        const locationDesc = document.getElementById('locationDesc');
        const battlesTableBody = document.getElementById('battlesTableBody');
        const adminBtn = document.getElementById('adminBtn');
        const addPointBtn = document.getElementById('addPointBtn');
        const addPointInstructions = document.getElementById('addPointInstructions');
        const adminPanel = document.getElementById('adminPanel');
        const battleForm = document.getElementById('battleForm');
        const cancelBtn = document.getElementById('cancelBtn');
        const battleType = document.getElementById('battleType');
        const eventDateField = document.getElementById('eventDateField');

        const map = L.map('map').setView([-15.7801, -47.9292], 11);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors',
            maxZoom: 19
        }).addTo(map);
        
        let markers = {};
        let addPointMode = false;
        
        closeBtn.addEventListener('click', () => infoCard.classList.remove('active'));

        adminBtn.addEventListener('click', () => {
            adminPanel.classList.toggle('active');
            if (adminPanel.classList.contains('active')) {
                adminBtn.textContent = 'Fechar Gerenciamento';
                adminBtn.classList.replace('bg-purple-600', 'bg-red-600');
                adminBtn.classList.replace('hover:bg-purple-700', 'hover:bg-red-700');
            } else {
                adminBtn.textContent = 'Gerenciar Batalhas';
                adminBtn.classList.replace('bg-red-600', 'bg-purple-600');
                adminBtn.classList.replace('hover:bg-red-700', 'hover:bg-purple-700');
                clearForm();
            }
        });

        battleType.addEventListener('change', () => {
            battleType.value === 'special' ? eventDateField.classList.add('active') : eventDateField.classList.remove('active');
        });

        addPointBtn.addEventListener('click', () => {
            addPointMode = !addPointMode;
            if (addPointMode) {
                addPointBtn.textContent = 'Cancelar Adição';
                addPointBtn.classList.replace('bg-green-600', 'bg-red-600');
                addPointBtn.classList.replace('hover:bg-green-700', 'hover:bg-red-700');
                addPointInstructions.style.display = 'block';
                map.getContainer().style.cursor = 'crosshair';
            } else {
                addPointBtn.textContent = 'Adicionar Ponto';
                addPointBtn.classList.replace('bg-red-600', 'bg-green-600');
                addPointBtn.classList.replace('hover:bg-red-700', 'hover:bg-green-700');
                addPointInstructions.style.display = 'none';
                map.getContainer().style.cursor = '';
            }
        });

        map.on('click', function(e) {
            if (addPointMode) {
                addPointMode = false;
                addPointBtn.textContent = 'Adicionar Ponto';
                addPointBtn.classList.replace('bg-red-600', 'bg-green-600');
                addPointBtn.classList.replace('hover:bg-red-700', 'hover:bg-green-700');
                addPointInstructions.style.display = 'none';
                map.getContainer().style.cursor = '';
                
                clearForm();
                document.getElementById('battleLat').value = e.latlng.lat;
                document.getElementById('battleLng').value = e.latlng.lng;
                
                adminPanel.classList.add('active');
                adminBtn.textContent = 'Fechar Gerenciamento';
                adminBtn.classList.replace('bg-purple-600', 'bg-red-600');
                adminBtn.classList.replace('hover:bg-purple-700', 'hover:bg-red-700');
                adminPanel.scrollIntoView({ behavior: 'smooth' });
            }
        });

        cancelBtn.addEventListener('click', () => {
            clearForm();
            adminPanel.classList.remove('active');
            adminBtn.textContent = 'Gerenciar Batalhas';
            adminBtn.classList.replace('bg-red-600', 'bg-purple-600');
            adminBtn.classList.replace('hover:bg-red-700', 'hover:bg-purple-700');
        });

        battleForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const id = document.getElementById('battleId').value;
            const battle = {
                id: id ? parseInt(id) : Date.now(),
                name: document.getElementById('battleName').value,
                address: document.getElementById('battleAddress').value,
                time: document.getElementById('battleTime').value,
                instagram: document.getElementById('battleIG').value,
                description: document.getElementById('battleDesc').value,
                lat: parseFloat(document.getElementById('battleLat').value),
                lng: parseFloat(document.getElementById('battleLng').value),
                type: document.getElementById('battleType').value
            };
            
            if (battle.type === 'special' && document.getElementById('eventDate').value) {
                battle.eventDate = document.getElementById('eventDate').value;
            }
            
            if (id) {
                const index = battles.findIndex(b => b.id === parseInt(id));
                if (index !== -1) battles[index] = battle;
            } else {
                battles.push(battle);
            }
            
            renderBattles();
            clearForm();
            adminPanel.classList.remove('active');
            adminBtn.textContent = 'Gerenciar Batalhas';
            adminBtn.classList.replace('bg-red-600', 'bg-purple-600');
            adminBtn.classList.replace('hover:bg-red-700', 'hover:bg-purple-700');
        });

        function clearForm() {
            document.getElementById('battleForm').reset();
            document.getElementById('battleId').value = '';
            document.getElementById('battleLat').value = '';
            document.getElementById('battleLng').value = '';
            eventDateField.classList.remove('active');
        }

        window.showBattleInfo = function(id) {
            const battle = battles.find(b => b.id === parseInt(id));
            if (!battle) return;
            
            locationTitle.textContent = battle.name;
            locationAddress.textContent = battle.address;
            locationTime.textContent = battle.time;
            
            let igLink = battle.instagram;
            if (igLink && !igLink.startsWith('http')) {
                 igLink = 'https://instagram.com/' + igLink.replace('@', '');
            }
            locationIG.href = igLink || '#';
            locationIG.textContent = battle.instagram ? '@' + battle.instagram.split('/').pop().replace('@', '') : 'Sem Instagram';
            
            locationDesc.textContent = battle.description;
            infoCard.classList.add('active');
            map.setView([battle.lat, battle.lng], 15);
            if (markers[battle.id]) markers[battle.id].openPopup();
        };

        window.editBattle = function(id) {
            const battle = battles.find(b => b.id === parseInt(id
