# imanbaytemirlan3-cyber.github.io
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SmartFloodControl | Оперативная сводка</title>
    
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    
    <style>
        :root {
            --bg-base: #0b1120;        
            --bg-panel: #1e293b;      
            --bg-panel-hover: #334155;
            --text-main: #f8fafc;
            --text-muted: #94a3b8;
            --primary: #3b82f6;        
            --primary-glow: rgba(59, 130, 246, 0.5);
            --danger: #ef4444;
            --warning: #f59e0b;
            --success: #10b981;
        }
        
        * { box-sizing: border-box; margin: 0; padding: 0; }
        
        body { 
            font-family: 'Inter', sans-serif; 
            background-color: var(--bg-base); 
            color: var(--text-main); 
            overflow-x: hidden;
            display: flex;
            flex-direction: column;
            height: 100vh;
        }

        /* --- ШАПКА --- */
        header { 
            padding: 15px 30px; 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            background: rgba(11, 17, 32, 0.8);
            backdrop-filter: blur(12px);
            border-bottom: 1px solid rgba(255,255,255,0.05);
            z-index: 100;
        }
        
        .logo { font-size: 1.5rem; font-weight: 700; color: var(--text-main); display: flex; align-items: center; gap: 10px; }
        .logo i { color: var(--primary); text-shadow: 0 0 15px var(--primary-glow); }
        
        .header-actions { display: flex; align-items: center; gap: 15px; }
        .live-status { display: flex; align-items: center; gap: 8px; font-size: 0.85rem; color: var(--success); background: rgba(16, 185, 129, 0.1); padding: 8px 14px; border-radius: 8px; border: 1px solid rgba(16, 185, 129, 0.2); font-weight: 600; }
        .live-dot { width: 8px; height: 8px; background-color: var(--success); border-radius: 50%; animation: pulse 2s infinite; }
        
        /* Кнопка меню справа */
        .menu-btn { background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); color: var(--text-main); font-size: 1.4rem; cursor: pointer; transition: 0.3s; padding: 6px 14px; border-radius: 8px; display: flex; align-items: center; justify-content: center; }
        .menu-btn:hover { background: rgba(59, 130, 246, 0.1); border-color: var(--primary); color: var(--primary); box-shadow: 0 0 10px var(--primary-glow); }

        @keyframes pulse { 0% { box-shadow: 0 0 0 0 rgba(16, 185, 129, 0.7); } 70% { box-shadow: 0 0 0 6px rgba(16, 185, 129, 0); } 100% { box-shadow: 0 0 0 0 rgba(16, 185, 129, 0); } }

        /* --- БОКОВАЯ ШТОРКА СПРАВА (SIDEBAR) --- */
        .sidebar-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.6); backdrop-filter: blur(4px); z-index: 2000; opacity: 0; visibility: hidden; transition: all 0.3s ease; }
        .sidebar-overlay.active { opacity: 1; visibility: visible; }
        
        .right-sidebar { position: fixed; top: 0; right: -350px; width: 320px; height: 100%; background: var(--bg-panel); box-shadow: -5px 0 25px rgba(0,0,0,0.5); z-index: 2001; transition: right 0.4s cubic-bezier(0.4, 0, 0.2, 1); display: flex; flex-direction: column; }
        .right-sidebar.active { right: 0; }
        
        .sidebar-header { padding: 25px; border-bottom: 1px solid rgba(255,255,255,0.05); display: flex; justify-content: space-between; align-items: center; font-size: 1.2rem; font-weight: 600; }
        .close-sidebar { background: rgba(255,255,255,0.05); border: none; color: var(--text-muted); width: 35px; height: 35px; border-radius: 50%; cursor: pointer; transition: 0.2s; display: flex; align-items: center; justify-content: center; font-size: 1.2rem; }
        .close-sidebar:hover { background: var(--danger); color: white; }
        
        .sidebar-menu { list-style: none; padding: 15px; overflow-y: auto; flex-grow: 1; }
        .sidebar-menu li { margin-bottom: 8px; }
        .sidebar-menu button { width: 100%; background: transparent; border: none; color: var(--text-main); font-size: 1rem; padding: 15px 20px; border-radius: 12px; text-align: left; cursor: pointer; display: flex; align-items: center; gap: 15px; transition: 0.2s; font-family: 'Inter', sans-serif; }
        .sidebar-menu button:hover { background: rgba(255,255,255,0.05); color: var(--primary); transform: translateX(-5px); }
        .sidebar-menu button i { font-size: 1.2rem; width: 25px; text-align: center; color: var(--text-muted); transition: 0.2s; }
        .sidebar-menu button:hover i { color: var(--primary); }
        
        .sidebar-menu .highlight-btn { background: rgba(59, 130, 246, 0.1); border: 1px solid rgba(59, 130, 246, 0.2); }
        .sidebar-menu .highlight-btn i { color: var(--primary); }

        .sidebar-footer { padding: 20px; border-top: 1px solid rgba(255,255,255,0.05); text-align: center; color: var(--text-muted); font-size: 0.8rem; }

        /* --- ЭКРАНЫ --- */
        .screen { flex-grow: 1; display: none; opacity: 0; transform: translateY(15px); transition: all 0.4s ease; height: calc(100vh - 70px); }
        .screen.active { display: flex; flex-direction: column; }
        .screen.visible { opacity: 1; transform: translateY(0); }

        /* --- ГЛАВНЫЙ ДАШБОРД --- */
        .dashboard-container { max-width: 1200px; margin: 0 auto; padding: 30px 20px; width: 100%; overflow-y: auto; }
        
        .top-section { display: flex; justify-content: space-between; align-items: flex-end; margin-bottom: 30px; flex-wrap: wrap; gap: 20px; }
        .top-section h1 { font-size: 2.2rem; font-weight: 700; }
        .top-section p { color: var(--text-muted); margin-top: 5px; }
        
        .btn-primary { background: var(--primary); color: white; border: none; padding: 12px 25px; border-radius: 8px; font-size: 1rem; font-weight: 600; cursor: pointer; transition: 0.3s; box-shadow: 0 4px 15px rgba(59, 130, 246, 0.4); display: flex; align-items: center; gap: 10px; }
        .btn-primary:hover { background: #2563eb; transform: translateY(-2px); box-shadow: 0 6px 20px rgba(59, 130, 246, 0.6); }

        /* Кликабельная Статистика */
        .stats-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr)); gap: 20px; margin-bottom: 30px; }
        .stat-card { background: var(--bg-panel); padding: 25px; border-radius: 16px; border: 1px solid rgba(255,255,255,0.05); position: relative; overflow: hidden; cursor: pointer; transition: all 0.3s; }
        .stat-card:hover { transform: translateY(-5px); box-shadow: 0 10px 25px rgba(0,0,0,0.3); border-color: rgba(59, 130, 246, 0.4); }
        .stat-card::before { content: ''; position: absolute; top: 0; left: 0; width: 4px; height: 100%; background: var(--success); }
        
        .stat-title { color: var(--text-muted); font-size: 0.9rem; text-transform: uppercase; letter-spacing: 1px; margin-bottom: 10px; }
        .stat-value { font-size: 2.2rem; font-weight: 700; color: var(--text-main); }
        .stat-desc { font-size: 0.85rem; color: var(--text-muted); margin-top: 5px; display: flex; align-items: center; justify-content: space-between; gap: 5px; }
        
        .stat-chart-hint { font-size: 0.8rem; color: var(--primary); opacity: 0; transition: 0.3s; display: flex; align-items: center; gap: 5px; }
        .stat-card:hover .stat-chart-hint { opacity: 1; }

        /* Информационные панели */
        .content-grid { display: grid; grid-template-columns: 2fr 1fr; gap: 20px; }
        @media (max-width: 900px) { .content-grid { grid-template-columns: 1fr; } }
        
        .panel { background: var(--bg-panel); border-radius: 16px; border: 1px solid rgba(255,255,255,0.05); padding: 25px; display: flex; flex-direction: column; }
        .panel h3 { font-size: 1.2rem; margin-bottom: 20px; display: flex; align-items: center; gap: 10px; border-bottom: 1px solid rgba(255,255,255,0.05); padding-bottom: 15px; }
        
        /* Лента событий (Динамичная) */
        #dchs-feed { overflow-y: auto; max-height: 400px; padding-right: 10px; scrollbar-width: thin; scrollbar-color: var(--primary) rgba(255,255,255,0.05); }
        #dchs-feed::-webkit-scrollbar { width: 6px; }
        #dchs-feed::-webkit-scrollbar-track { background: rgba(255,255,255,0.05); border-radius: 4px; }
        #dchs-feed::-webkit-scrollbar-thumb { background: var(--primary); border-radius: 4px; }

        .feed-item { display: flex; gap: 15px; margin-bottom: 20px; }
        .feed-icon { width: 40px; height: 40px; border-radius: 50%; background: rgba(16, 185, 129, 0.1); color: var(--success); display: flex; align-items: center; justify-content: center; font-size: 1.1rem; flex-shrink: 0; }
        .feed-icon.info { background: rgba(59, 130, 246, 0.1); color: var(--primary); }
        .feed-icon.warning { background: rgba(245, 158, 11, 0.1); color: var(--warning); }
        .feed-content h4 { font-size: 1rem; margin-bottom: 5px; }
        .feed-content p { font-size: 0.9rem; color: var(--text-muted); line-height: 1.5; }
        .feed-time { font-size: 0.8rem; color: #64748b; margin-top: 5px; }

        /* Погода */
        .weather-panel { cursor: pointer; transition: all 0.3s; position: relative; }
        .weather-panel:hover { transform: translateY(-5px); box-shadow: 0 10px 25px rgba(0,0,0,0.3); border-color: rgba(59, 130, 246, 0.4); }
        
        .weather-box { text-align: center; padding: 20px 0; }
        .weather-icon { font-size: 3rem; color: #93c5fd; margin-bottom: 15px; }
        .weather-temp { font-size: 2rem; font-weight: 700; }
        .weather-desc { color: var(--text-muted); font-size: 0.95rem; margin-top: 5px; }
        
        .weather-hint { text-align: center; font-size: 0.85rem; color: var(--primary); margin-top: auto; opacity: 0; transition: 0.3s; display: flex; align-items: center; justify-content: center; gap: 8px; padding-top: 15px; }
        .weather-panel:hover .weather-hint { opacity: 1; }

        .forecast-container { display: flex; gap: 15px; overflow-x: auto; padding: 10px 0 20px 0; scrollbar-width: thin; scrollbar-color: var(--primary) rgba(255,255,255,0.05); }
        .forecast-container::-webkit-scrollbar { height: 8px; }
        .forecast-container::-webkit-scrollbar-track { background: rgba(255,255,255,0.05); border-radius: 4px; }
        .forecast-container::-webkit-scrollbar-thumb { background: var(--primary); border-radius: 4px; }
        
        .forecast-card { background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.1); border-radius: 12px; padding: 20px 15px; min-width: 120px; text-align: center; flex-shrink: 0; transition: 0.3s; }
        .forecast-card:hover { background: rgba(59, 130, 246, 0.1); border-color: var(--primary); transform: translateY(-3px); }
        .forecast-date { font-size: 0.9rem; color: var(--text-muted); margin-bottom: 12px; font-weight: 600; }
        .forecast-icon { font-size: 2.2rem; margin-bottom: 12px; }
        .forecast-temp { font-size: 1.4rem; font-weight: 700; color: var(--text-main); }
        .forecast-desc { font-size: 0.8rem; color: var(--text-muted); margin-top: 8px; }

        /* --- ЭКРАН КАРТЫ --- */
        .map-header { padding: 15px 20px; background: var(--bg-panel); display: flex; align-items: center; gap: 15px; z-index: 10; border-bottom: 1px solid rgba(255,255,255,0.05); }
        .btn-back { background: transparent; border: 1px solid rgba(255,255,255,0.1); color: var(--text-main); padding: 8px 15px; border-radius: 8px; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 8px; }
        .btn-back:hover { background: rgba(255,255,255,0.1); }
        .map-container { flex-grow: 1; position: relative; }
        #map { width: 100%; height: 100%; background: #e5e7eb; } 

        /* --- МОДАЛЬНЫЕ ОКНА --- */
        .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.7); backdrop-filter: blur(5px); z-index: 3000; display: flex; align-items: center; justify-content: center; opacity: 0; visibility: hidden; transition: all 0.3s ease; }
        .modal-overlay.active { opacity: 1; visibility: visible; }
        .modal-content { background: var(--bg-panel); width: 90%; max-width: 500px; border-radius: 16px; padding: 30px; border: 1px solid rgba(255,255,255,0.1); transform: translateY(30px); transition: all 0.3s ease; position: relative; max-height: 90vh; overflow-y: auto; }
        .modal-overlay.active .modal-content { transform: translateY(0); }
        .close-modal { position: absolute; top: 15px; right: 20px; background: none; border: none; color: var(--text-muted); font-size: 1.5rem; cursor: pointer; transition: 0.2s; z-index: 10; }
        .close-modal:hover { color: white; }
        
        .chart-container { position: relative; height: 300px; width: 100%; margin-top: 20px; }
    </style>
</head>
<body>

    <header>
        <div class="logo"><i class="fa-solid fa-water"></i> SmartFlood</div>
        <div class="header-actions">
            <div class="live-status"><div class="live-dot"></div> Live</div>
            <button class="menu-btn" onclick="toggleSidebar()"><i class="fa-solid fa-bars"></i> Меню</button>
        </div>
    </header>

    <div class="sidebar-overlay" id="sidebar-overlay" onclick="toggleSidebar()"></div>
    <div class="right-sidebar" id="right-sidebar">
        <div class="sidebar-header">
            Меню управления
            <button class="close-sidebar" onclick="toggleSidebar()"><i class="fa-solid fa-xmark"></i></button>
        </div>
        <ul class="sidebar-menu">
            <li><button class="highlight-btn" onclick="openScreen('screen-map'); toggleSidebar();"><i class="fa-solid fa-map-location-dot"></i> Интерактивная карта</button></li>
            
            <li><button onclick="window.open('https://t.me/smartfloodcontrolhelp_bot', '_blank'); toggleSidebar();"><i class="fa-solid fa-camera"></i> Сообщить о подтоплении</button></li>
            
            <li><button onclick="window.open('https://t.me/smartfloodcontrol_bot', '_blank'); toggleSidebar();"><i class="fa-brands fa-telegram"></i> Telegram-бот (Поддержка)</button></li>
            <li><button onclick="openModal('modal-ikomek'); toggleSidebar();"><i class="fa-solid fa-headset"></i> Служба 109 iKomek</button></li>
            <li><button onclick="openModal('modal-rules'); toggleSidebar();"><i class="fa-solid fa-suitcase-medical"></i> Инструкции при эвакуации</button></li>
            <li><button onclick="alert('Push-уведомления успешно активированы!');"><i class="fa-solid fa-bell"></i> Включить уведомления</button></li>
        </ul>
        <div class="sidebar-footer">
            SmartFloodControl © 2026<br>СКО, Казахстан
        </div>
    </div>

    <main id="screen-home" class="screen active visible">
        <div class="dashboard-container">
            
            <div class="top-section">
                <div>
                    <h1>Оперативная сводка СКО</h1>
                    <p>Анализ состояния рек и водохранилищ на <b>28 февраля 2026 г.</b></p>
                </div>
                <button class="btn-primary" onclick="openScreen('screen-map')">
                    <i class="fa-solid fa-satellite-dish"></i> Открыть Карту
                </button>
            </div>

            <div class="stats-grid">
                <div class="stat-card" onclick="openChartModal('Сергеевка', 800)">
                    <div class="stat-title">Сергеевское вдхр.</div>
                    <div class="stat-value">620 см</div>
                    <div class="stat-desc">
                        <span><i class="fa-solid fa-snowflake" style="color: var(--text-muted);"></i> Зимний режим</span>
                        <span class="stat-chart-hint"><i class="fa-solid fa-chart-line"></i> График</span>
                    </div>
                </div>
                <div class="stat-card" onclick="openChartModal('Петропавловск', 1050)">
                    <div class="stat-title">г. Петропавловск</div>
                    <div class="stat-value">363 см</div>
                    <div class="stat-desc">
                        <span><i class="fa-solid fa-minus" style="color: var(--text-muted);"></i> Без изменений</span>
                        <span class="stat-chart-hint"><i class="fa-solid fa-chart-line"></i> График</span>
                    </div>
                </div>
                <div class="stat-card" onclick="openChartModal('Общая динамика', 0)">
                    <div class="stat-title">Активные гидропосты</div>
                    <div class="stat-value">5 / 5</div>
                    <div class="stat-desc">
                        <span><i class="fa-solid fa-check" style="color: var(--success);"></i> Штатно</span>
                        <span class="stat-chart-hint"><i class="fa-solid fa-chart-line"></i> Сводный график</span>
                    </div>
                </div>
            </div>

            <div class="content-grid">
                <div class="panel">
                    <h3 style="justify-content: space-between;">
                        <span style="display: flex; align-items: center; gap: 10px;"><i class="fa-solid fa-tower-broadcast"></i> Журнал событий ДЧС</span>
                        <button onclick="renderDchsEvents()" title="Обновить ленту" style="background: none; border: none; color: var(--text-muted); cursor: pointer; font-size: 1rem; transition: 0.2s;"><i class="fa-solid fa-rotate-right"></i></button>
                    </h3>
                    
                    <div id="dchs-feed"></div>

                </div>

                <div class="panel weather-panel" onclick="openModal('modal-weather')">
                    <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid rgba(255,255,255,0.05); padding-bottom: 15px; margin-bottom: 20px;">
                        <h3 style="margin-bottom: 0; border: none; padding: 0;"><i class="fa-solid fa-cloud-sun"></i> Метеоусловия</h3>
                        <i class="fa-solid fa-chevron-right" style="color: var(--text-muted); font-size: 0.9rem;"></i>
                    </div>
                    <div class="weather-box">
                        <i class="fa-solid fa-cloud-showers-heavy weather-icon" style="color: #60a5fa;"></i>
                        <div class="weather-temp">-18°C</div>
                        <div class="weather-desc">Петропавловск, Пасмурно, дождь со снегом</div>
                    </div>
                    <div style="margin-top: 15px; border-top: 1px solid rgba(255,255,255,0.05); padding-top: 15px;">
                        <p style="font-size: 0.9rem; color: var(--text-muted); line-height: 1.5;">
                            <b>Текущая обстановка:</b> В регионе наблюдается потепление и смешанные осадки. Ветер юго-западный, 6-10 м/с.
                        </p>
                    </div>
                    <div class="weather-hint"><i class="fa-regular fa-hand-pointer"></i> Нажмите для прогноза на 10 дней</div>
                </div>
            </div>
        </div>
    </main>

    <main id="screen-map" class="screen">
        <div class="map-header">
            <button class="btn-back" onclick="openScreen('screen-home')">
                <i class="fa-solid fa-arrow-left"></i> Вернуться к сводке
            </button>
        </div>
        <div class="map-container">
            <div id="map"></div>
        </div>
    </main>

    <div class="modal-overlay" id="modal-weather" onclick="closeModalOutside(event, 'modal-weather')">
        <div class="modal-content" style="max-width: 850px;">
            <button class="close-modal" onclick="closeModal('modal-weather')">&times;</button>
            <h2 style="margin-bottom: 5px;"><i class="fa-solid fa-calendar-days" style="color: var(--primary);"></i> Прогноз погоды на 10 дней</h2>
            <p style="color: var(--text-muted); font-size: 0.9rem; margin-bottom: 20px;">Петропавловск (с 28.02.2026)</p>
            
            <div class="forecast-container" id="forecast-container"></div>
        </div>
    </div>

    <div class="modal-overlay" id="modal-chart" onclick="closeModalOutside(event, 'modal-chart')">
        <div class="modal-content" style="max-width: 700px;">
            <button class="close-modal" onclick="closeModal('modal-chart')">&times;</button>
            <h2 id="chartModalTitle" style="margin-bottom: 5px;"><i class="fa-solid fa-chart-line" style="color: var(--primary);"></i> Динамика уровня воды</h2>
            <p id="chartModalDesc" style="color: var(--text-muted); font-size: 0.9rem;">Архивные данные за последнюю неделю февраля 2026</p>
            <div class="chart-container">
                <canvas id="floodChart"></canvas>
            </div>
        </div>
    </div>

    <div class="modal-overlay" id="modal-rules" onclick="closeModalOutside(event, 'modal-rules')">
        <div class="modal-content">
            <button class="close-modal" onclick="closeModal('modal-rules')">&times;</button>
            <h2 style="margin-bottom: 20px; color: var(--warning);"><i class="fa-solid fa-suitcase-medical"></i> Правила эвакуации</h2>
            <div class="modal-body">
                <p>При звуке сирен <b>«Внимание всем»</b> включите ТВ или радио.</p>
                <p><b>Перед уходом из дома:</b></p>
                <ul style="padding-left: 20px; color: var(--text-muted); margin-bottom: 15px;">
                    <li>Отключите электричество, газ и воду.</li>
                    <li>Перенесите ценные вещи на верхние этажи.</li>
                    <li>Отвяжите животных, откройте загоны.</li>
                </ul>
                <p><b>Взять с собой (Тревожный чемоданчик):</b></p>
                <ul style="padding-left: 20px; color: var(--text-muted);">
                    <li>Документы в водонепроницаемом пакете.</li>
                    <li>Деньги, аптечку, теплые вещи, сапоги.</li>
                    <li>Запас еды и воды на 3 суток.</li>
                </ul>
            </div>
        </div>
    </div>

    <div class="modal-overlay" id="modal-ikomek" onclick="closeModalOutside(event, 'modal-ikomek')">
        <div class="modal-content">
            <button class="close-modal" onclick="closeModal('modal-ikomek')">&times;</button>
            <h2 style="margin-bottom: 20px;"><i class="fa-solid fa-headset" style="color: var(--success);"></i> 109 iKomek</h2>
            <div class="modal-body">
                <p>Единый контакт-центр. Оставьте заявку на откачку талых вод или сообщите о перекрытых дорогах.</p>
                <div style="background: rgba(16, 185, 129, 0.1); padding: 15px; border-radius: 8px; border-left: 4px solid var(--success); font-size: 1.5rem; font-weight: bold; margin: 15px 0; text-align: center;">Тел: <span style="color: var(--success);">109</span></div>
                <a href="https://ikomekastana.kz/" target="_blank" class="btn-primary" style="width: 100%; justify-content: center; background: #0f766e; box-shadow: none;"><i class="fa-solid fa-globe"></i> Перейти на сайт ikomekastana.kz</a>
            </div>
        </div>
    </div>

    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script>
        // --- 1. УПРАВЛЕНИЕ БОКОВОЙ ШТОРКОЙ ---
        function toggleSidebar() {
            document.getElementById('right-sidebar').classList.toggle('active');
            document.getElementById('sidebar-overlay').classList.toggle('active');
        }

        // --- 2. ПЕРЕКЛЮЧЕНИЕ ЭКРАНОВ ---
        function openScreen(screenId) {
            const current = document.querySelector('.screen.active');
            if(current && current.id !== screenId) {
                current.classList.remove('visible');
                setTimeout(() => {
                    current.classList.remove('active');
                    showNewScreen(screenId);
                }, 300);
            } else if (!current) { showNewScreen(screenId); }
        }

        function showNewScreen(screenId) {
            const next = document.getElementById(screenId);
            next.classList.add('active');
            if(screenId === 'screen-map') setTimeout(() => map.invalidateSize(), 50);
            setTimeout(() => next.classList.add('visible'), 50);
        }

        // --- 3. МОДАЛЬНЫЕ ОКНА ---
        function openModal(id) { document.getElementById(id).classList.add('active'); }
        function closeModal(id) { document.getElementById(id).classList.remove('active'); }
        function closeModalOutside(e, id) { if(e.target.id === id) closeModal(id); }

        // --- 4. КАРТА ---
        const boundsСКО = [[53.10, 65.20], [55.45, 71.30]];
        const map = L.map('map', { center: [54.5, 68.5], zoom: 8, minZoom: 8, maxZoom: 12, maxBounds: boundsСКО, maxBoundsViscosity: 1.0 });
        L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png', { attribution: '© OpenStreetMap, © CartoDB', subdomains: 'abcd', maxZoom: 20 }).addTo(map);

        const winterData = [
            { name: "Петропавловск", lat: 54.8655, lon: 69.1351, current: 363, critical: 1050 },
            { name: "Бесколь", lat: 54.7800, lon: 69.1100, current: 280, critical: 900 },
            { name: "Соколовка", lat: 55.0450, lon: 69.1510, current: 250, critical: 850 },
            { name: "Сергеевка", lat: 53.8833, lon: 67.4167, current: 620, critical: 800 },
            { name: "Тепличное", lat: 54.8510, lon: 69.1835, current: 310, critical: 850 }
        ];

        winterData.forEach(p => {
            const marker = L.circleMarker([p.lat, p.lon], { radius: 10, fillColor: "#10b981", color: "#fff", weight: 2, fillOpacity: 0.8 }).addTo(map);
            marker.bindPopup(`<div style="font-family: 'Inter'; background: #1e293b; color: #f8fafc; padding: 5px; border-radius: 8px;"><h3 style="margin: 0 0 10px; font-size: 1.1rem; border-bottom: 1px solid rgba(255,255,255,0.1); padding-bottom: 5px;">${p.name}</h3>Текущий: <b>${p.current} см</b><br>Критический: <b style="color:#ef4444">${p.critical} см</b></div>`);
        });

        // --- 5. ДИНАМИЧНЫЙ ЖУРНАЛ ДЧС ---
        const dchsEvents = [
            { title: "Штормовое предупреждение", desc: "Ожидается усиление ветра до 15-20 м/с, низовая метель. Просим соблюдать осторожность на дорогах.", time: "Сегодня, 11:20", icon: "fa-wind", type: "warning" },
            { title: "Мониторинг водоемов", desc: "Службы завершили утренний облет русла реки Есиль. Заторов льда на данный момент не зафиксировано.", time: "Сегодня, 09:15", icon: "fa-helicopter", type: "info" },
            { title: "Профилактические работы", desc: "Начата расчистка водопропускных труб в районе села Бесколь. Техника работает в штатном режиме.", time: "Вчера, 16:00", icon: "fa-person-digging", type: "normal" },
            { title: "Замеры толщины льда", desc: "Толщина ледяного покрова в районе Сергеевки составляет 70-85 см. Показатели в пределах нормы для конца февраля.", time: "26 Фев, 14:30", icon: "fa-snowflake", type: "normal" }
        ];

        function renderDchsEvents() {
            const container = document.getElementById('dchs-feed');
            
            // Добавим небольшую анимацию загрузки/обновления
            container.style.opacity = '0.5';
            
            setTimeout(() => {
                container.innerHTML = dchsEvents.map(ev => {
                    let iconClass = ev.type === 'info' ? 'info' : (ev.type === 'warning' ? 'warning' : '');
                    return `
                        <div class="feed-item">
                            <div class="feed-icon ${iconClass}"><i class="fa-solid ${ev.icon}"></i></div>
                            <div class="feed-content">
                                <h4>${ev.title}</h4>
                                <p>${ev.desc}</p>
                                <div class="feed-time">${ev.time}</div>
                            </div>
                        </div>
                    `;
                }).join('');
                container.style.opacity = '1';
            }, 300);
        }

        // --- 6. ГРАФИКИ (CHART.JS) ---
        const historyDB = [
            { date: "22 Фев", data: { "Сергеевка": 618, "Петропавловск": 361 } },
            { date: "23 Фев", data: { "Сергеевка": 619, "Петропавловск": 362 } },
            { date: "24 Фев", data: { "Сергеевка": 619, "Петропавловск": 362 } },
            { date: "25 Фев", data: { "Сергеевка": 619, "Петропавловск": 363 } },
            { date: "26 Фев", data: { "Сергеевка": 620, "Петропавловск": 363 } },
            { date: "27 Фев", data: { "Сергеевка": 620, "Петропавловск": 363 } },
            { date: "28 Фев", data: { "Сергеевка": 620, "Петропавловск": 363 } }
        ];

        let floodChart = null;

        function openChartModal(cityName, criticalLevel) {
            document.getElementById('chartModalTitle').innerHTML = `<i class="fa-solid fa-chart-line" style="color: var(--primary);"></i> ${cityName}: Уровень воды`;
            
            const labels = historyDB.map(d => d.date);
            let datasets = [];

            if (cityName === 'Общая динамика') {
                datasets = [
                    { label: 'Сергеевка', data: historyDB.map(d => d.data["Сергеевка"]), borderColor: '#f59e0b', backgroundColor: 'rgba(245, 158, 11, 0.1)', fill: true, tension: 0.3 },
                    { label: 'Петропавловск', data: historyDB.map(d => d.data["Петропавловск"]), borderColor: '#3b82f6', backgroundColor: 'rgba(59, 130, 246, 0.1)', fill: true, tension: 0.3 }
                ];
            } else {
                datasets = [{
                    label: cityName + ' (Крит: ' + criticalLevel + ')',
                    data: historyDB.map(d => d.data[cityName]),
                    borderColor: '#10b981', backgroundColor: 'rgba(16, 185, 129, 0.1)', fill: true, tension: 0.3, borderWidth: 3
                }];
            }

            if (floodChart) floodChart.destroy();

            const ctx = document.getElementById('floodChart').getContext('2d');
            Chart.defaults.color = '#94a3b8';
            Chart.defaults.font.family = "'Inter', sans-serif";

            floodChart = new Chart(ctx, {
                type: 'line',
                data: { labels: labels, datasets: datasets },
                options: {
                    responsive: true, maintainAspectRatio: false,
                    scales: {
                        y: { beginAtZero: false, grid: { color: 'rgba(255,255,255,0.05)' } },
                        x: { grid: { color: 'rgba(255,255,255,0.05)' } }
                    },
                    plugins: { legend: { display: true, position: 'top' } }
                }
            });

            openModal('modal-chart');
        }

        // --- 7. ПРОГНОЗ ПОГОДЫ (АКТУАЛИЗИРОВАННЫЙ) ---
        // Данные приближены к реальному прогнозу для Петропавловска на рубеже февраля/марта
        const forecastData = [
            { date: "28 Фев", icon: "fa-cloud-showers-heavy", temp: "-18°C", desc: "Пасмурно", color: "#60a5fa" },
            { date: "01 Мар", icon: "fa-cloud", temp: "-16°C", desc: "Пасмурно", color: "#cbd5e1" },
            { date: "02 Мар", icon: "fa-cloud-sun", temp: "-8°C", desc: "Переменная", color: "#fde047" },
            { date: "03 Мар", icon: "fa-snowflake", temp: "-8°C", desc: "Небольшой снег", color: "#93c5fd" },
            { date: "04 Мар", icon: "fa-cloud", temp: "-3°C", desc: "Облачно", color: "#cbd5e1" },
            { date: "05 Мар", icon: "fa-sun", temp: "-7°C", desc: "Ясно", color: "#fcd34d" },
            { date: "06 Мар", icon: "fa-cloud-sun", temp: "-5°C", desc: "Переменная", color: "#fde047" },
            { date: "07 Мар", icon: "fa-cloud-rain", temp: "-4°C", desc: "Слабый дождь", color: "#60a5fa" },
            { date: "08 Мар", icon: "fa-sun", temp: "-8°C", desc: "Ясно", color: "#fcd34d" },
            { date: "09 Мар", icon: "fa-cloud", temp: "-12°C", desc: "Облачно", color: "#cbd5e1" }
        ];

        function renderForecast() {
            const container = document.getElementById('forecast-container');
            container.innerHTML = forecastData.map(day => `
                <div class="forecast-card">
                    <div class="forecast-date">${day.date}</div>
                    <div class="forecast-icon"><i class="fa-solid ${day.icon}" style="color: ${day.color};"></i></div>
                    <div class="forecast-temp">${day.temp}</div>
                    <div class="forecast-desc">${day.desc}</div>
                </div>
            `).join('');
        }

        // Инициализация при загрузке
        renderDchsEvents();
        renderForecast();
    </script>
</body>
</html>
