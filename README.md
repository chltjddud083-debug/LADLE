<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>1제강 래들 수리장 3D 스마트 팩토리</title>
    <!-- Three.js 3D 엔진 및 OrbitControls -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/three@0.128.0/examples/js/controls/OrbitControls.js"></script>
    
    <!-- Firebase Realtime Database SDK (Compat v9) -->
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>

    <style>
        * { box-sizing: border-box; }
        body { margin: 0; padding: 0; overflow: hidden; font-family: 'Malgun Gothic', '맑은 고딕', sans-serif; background-color: #f1f5f9; color: #1e293b; user-select: none; }
        /* 상단 헤더 */
        #header { position: absolute; top: 0; left: 0; right: 0; height: 54px; background: #ffffff; padding: 0 16px; display: flex; align-items: center; gap: 7px; border-bottom: 2px solid #cbd5e1; z-index: 2000; }
        #header h2 { margin: 0 4px 0 0; font-size: 1.15rem; color: #0284c7; font-weight: 900; white-space: nowrap; }
        .header-group { display: flex; align-items: center; gap: 5px; }
        .header-divider { width: 1.5px; height: 22px; background-color: #cbd5e1; margin: 0 4px; }
        .btn { background-color: #ffffff; color: #334155; border: 1.5px solid #cbd5e1; padding: 5px 10px; font-size: 0.82rem; font-weight: bold; border-radius: 6px; cursor: pointer; transition: 0.2s; display: flex; align-items: center; gap: 4px; white-space: nowrap; }
        .btn:hover { background-color: #f8fafc; border-color: #94a3b8; }
        .btn:disabled { opacity: 0.38; cursor: not-allowed; border-color: #e2e8f0; }
        .btn-add { background-color: #0284c7; color: white; border: none; }
        .btn-add:hover { background-color: #0369a1; }
        .btn-edit-mode { background-color: #f59e0b; color: white; border: none; }
        .btn-edit-mode:hover { background-color: #d97706; }
        .btn-edit-active { background-color: #dc2626 !important; color: white !important; animation: pulse-btn 1.2s infinite; }
        @keyframes pulse-btn { 0% { transform: scale(1); } 50% { transform: scale(1.04); } 100% { transform: scale(1); } }
        #sync-badge { margin-left: auto; background: #dcfce7; border: 1.5px solid #22c55e; color: #15803d; padding: 4px 10px; border-radius: 20px; font-size: 0.78rem; font-weight: 900; display: flex; align-items: center; gap: 6px; white-space: nowrap; }
        .sync-dot { width: 7px; height: 7px; background-color: #22c55e; border-radius: 50%; animation: pulse 1.5s infinite; }
        @keyframes pulse { 0% { transform: scale(0.9); opacity: 0.7; } 50% { transform: scale(1.3); opacity: 1; } 100% { transform: scale(0.9); opacity: 0.7; } }

        /* 전체 레이아웃 */
        #main-layout { display: flex; width: 100vw; height: calc(100vh - 54px); margin-top: 54px; position: relative; overflow: hidden; }
        #webgl-container { width: 100%; height: 100%; position: relative; background-color: #f8fafc; }

        /* 상단 대형 LED 전광판 띠 (★수정: 사이드바가 열린 상태가 기본이므로 right: 394px) */
        #ticker-wrap { position: absolute; top: 4px; left: 14px; right: 394px; height: 44px; background: linear-gradient(180deg, #090d16 0%, #1e293b 100%); border: 2px solid #38bdf8; border-radius: 8px; display: flex; align-items: center; overflow: hidden; box-shadow: 0 4px 16px rgba(0,0,0,0.4); z-index: 500; pointer-events: none; transition: right 0.3s cubic-bezier(0.4, 0, 0.2, 1); }
        .ticker-badge { background: #dc2626; color: #ffffff; font-weight: 900; font-size: 0.88rem; padding: 0 16px; height: 100%; display: flex; align-items: center; gap: 6px; white-space: nowrap; letter-spacing: 1px; box-shadow: 4px 0 10px rgba(0,0,0,0.3); z-index: 2; }
        .ticker-track { flex: 1; overflow: hidden; white-space: nowrap; position: relative; }
        .ticker-content { display: inline-block; white-space: nowrap; padding-left: 100%; animation: ticker-anim 26s linear infinite; font-size: 1.25rem; font-weight: 900; color: #facc15; text-shadow: 0 0 10px rgba(250, 204, 21, 0.7); letter-spacing: 2px; font-family: 'Consolas', 'Malgun Gothic', monospace; }
        @keyframes ticker-anim { 0% { transform: translate3d(0, 0, 0); } 100% { transform: translate3d(-100%, 0, 0); } }

        /* 우측 상단 대형 디지털 시계 HUD (★수정: 사이드바가 열린 상태가 기본이므로 right: 398px) */
        #floating-hud-clock { position: absolute; top: 54px; right: 398px; background: rgba(15, 23, 42, 0.92); backdrop-filter: blur(8px); border: 2.5px solid #38bdf8; border-radius: 12px; padding: 8px 18px; color: #ffffff; text-align: center; box-shadow: 0 8px 24px rgba(0, 0, 0, 0.45); z-index: 600; pointer-events: none; display: flex; flex-direction: column; align-items: center; transition: right 0.3s cubic-bezier(0.4, 0, 0.2, 1); }
        #floating-hud-clock .hud-date { font-size: 0.92rem; color: #94a3b8; font-weight: 800; letter-spacing: 1px; }
        #floating-hud-clock .hud-time { font-size: 2.3rem; font-weight: 900; color: #38bdf8; font-family: 'Consolas', monospace; letter-spacing: 3px; text-shadow: 0 0 14px rgba(56, 189, 248, 0.6); line-height: 1.1; margin-top: 2px; }

        /* 우측 사이드 패널 */
        #right-sidebar { position: absolute; top: 0; right: 0; width: 380px; height: 100%; background: #ffffff; border-left: 2px solid #cbd5e1; display: flex; flex-direction: column; overflow-y: auto; padding: 16px; gap: 14px; box-shadow: -4px 0 16px rgba(0,0,0,0.08); transform: translateX(0); transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1); z-index: 800; box-sizing: border-box; }
        #right-sidebar.collapsed { transform: translateX(100%); }
        #sidebar-toggle-btn { position: absolute; top: 50%; right: 380px; transform: translateY(-50%); width: 28px; height: 68px; background: #ffffff; border: 2px solid #cbd5e1; border-right: none; border-radius: 8px 0 0 8px; display: flex; align-items: center; justify-content: center; cursor: pointer; box-shadow: -4px 2px 10px rgba(0,0,0,0.15); font-size: 14px; font-weight: 900; color: #0284c7; user-select: none; z-index: 1500; transition: right 0.3s cubic-bezier(0.4, 0, 0.2, 1), background-color 0.2s; }
        #sidebar-toggle-btn:hover { background-color: #f0f9ff; color: #0369a1; }
        #sidebar-toggle-btn.collapsed { right: 0px !important; border-right: 2px solid #cbd5e1; background-color: #e0f2fe; box-shadow: -4px 2px 12px rgba(2, 132, 199, 0.25); }

        .clock-card { background: #0f172a; border: 2px solid #38bdf8; border-radius: 10px; padding: 12px 16px; color: #ffffff; text-align: center; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .clock-card .c-date { font-size: 0.95rem; color: #94a3b8; font-weight: bold; }
        .clock-card .c-time { font-size: 2.2rem; font-weight: 900; color: #38bdf8; font-family: 'Consolas', monospace; letter-spacing: 2px; margin-top: 4px; }

        .dash-section { background: #f8fafc; border: 1.5px solid #e2e8f0; border-radius: 10px; padding: 14px; }
        .dash-section h4 { margin: 0 0 10px 0; font-size: 0.92rem; font-weight: 900; color: #0369a1; display: flex; align-items: center; justify-content: space-between; }
        .auto-badge { font-size: 0.72rem; background: #e0f2fe; color: #0284c7; padding: 2px 8px; border-radius: 12px; font-weight: bold; }
        .shift-display-box { background: #ffffff; border: 2px solid #0284c7; border-radius: 8px; padding: 12px; text-align: center; }
        .shift-main-text { font-size: 1.15rem; font-weight: 900; color: #0284c7; display: block; }

        .worker-tags-container { display: flex; flex-wrap: wrap; gap: 6px; margin-top: 6px; }
        .worker-tag { background: #e2e8f0; color: #1e293b; padding: 5px 10px; border-radius: 6px; font-size: 0.85rem; font-weight: bold; display: flex; align-items: center; gap: 4px; border: 1px solid #cbd5e1; }

        .dash-input, .dash-select, .dash-textarea { width: 100%; padding: 8px 10px; background: #ffffff; border: 1.5px solid #cbd5e1; border-radius: 6px; font-size: 0.9rem; font-family: inherit; color: #0f172a; }
        .dash-input:focus, .dash-select:focus, .dash-textarea:focus { outline: none; border-color: #0284c7; }
        .dash-textarea { height: 90px; resize: vertical; line-height: 1.4; }
        .notice-textarea { background: #fefce8; border-color: #facc15; color: #713f12; font-weight: 600; }

        /* 래들 우클릭 플로팅 카드 */
        #ladle-floating-card { display: none; position: fixed; width: 330px; max-height: calc(100vh - 75px); background: rgba(255, 255, 255, 0.98); backdrop-filter: blur(8px); border: 2.5px solid #0284c7; border-radius: 12px; box-shadow: 0 16px 40px rgba(0,0,0,0.35); padding: 16px; z-index: 3500; overflow-y: auto; box-sizing: border-box; }
        #ladle-floating-card h4 { margin: 0 0 12px 0; font-size: 1.05rem; font-weight: 900; color: #0284c7; display: flex; justify-content: space-between; align-items: center; border-bottom: 2px solid #f1f5f9; padding-bottom: 6px; }
        .card-row { margin-bottom: 9px; }
        .card-row label { display: block; font-size: 0.78rem; font-weight: bold; color: #64748b; margin-bottom: 3px; }

        /* 모달창 */
        .modal-overlay { display: none; position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: rgba(0, 0, 0, 0.45); backdrop-filter: blur(3px); z-index: 4000; justify-content: center; align-items: center; }
        .modal-box { background: #ffffff; border-radius: 12px; width: 460px; padding: 24px; box-shadow: 0 15px 35px rgba(0,0,0,0.2); }
        .modal-box h3 { margin-top: 0; color: #0284c7; font-size: 1.2rem; border-bottom: 2px solid #f1f5f9; padding-bottom: 12px; }
        .form-group { margin-bottom: 14px; }
        .form-group label { display: block; font-size: 0.85rem; font-weight: bold; margin-bottom: 6px; color: #475569; }
        .radio-group { display: flex; gap: 20px; }
        .radio-group label { display: flex; align-items: center; gap: 6px; font-weight: 600; cursor: pointer; }

        /* ★수정: 편집 모드 배너가 전광판을 가리지 않도록 아래로 이동 */
        #layout-edit-banner { display: none; position: absolute; top: 108px; left: 50%; transform: translateX(-50%); background: #dc2626; color: #ffffff; padding: 8px 24px; border-radius: 20px; font-weight: 900; font-size: 0.9rem; box-shadow: 0 4px 15px rgba(220, 38, 38, 0.35); z-index: 1000; pointer-events: none; }
    </style>
</head>
<body>

    <!-- 상단 헤더 -->
    <div id="header">
        <h2>🏗️ 1제강 래들 수리장</h2>
        
        <div class="header-divider"></div>

        <div class="header-group">
            <button class="btn btn-add" onclick="openAddModal()">➕ 래들 등록</button>
            <button class="btn" id="btn-undo" onclick="undo()" title="실행 취소 (Ctrl+Z)" disabled>↩️ 되돌리기</button>
            <button class="btn" id="btn-redo" onclick="redo()" title="다시 실행 (Ctrl+Y)" disabled>↪️ 다시 실행</button>
        </div>

        <div class="header-divider"></div>

        <div class="header-group">
            <button class="btn btn-edit-mode" id="btn-layout-mode" onclick="toggleLayoutEditMode()">🛠️ 시설/인원 배치</button>
            <button class="btn" onclick="saveCurrentCameraView()">💾 시점 저장</button>
            <button class="btn" onclick="resetCamera()">🎥 시점 초기화</button>
        </div>

        <div class="header-divider"></div>

        <div class="header-group">
            <button class="btn" onclick="exportDataBackup()" title="현재 설정을 JSON 파일로 다운로드">💾 백업</button>
            <button class="btn" onclick="document.getElementById('import-file-input').click()" title="JSON 백업 파일 불러오기">📂 복원</button>
            <input type="file" id="import-file-input" accept=".json" style="display:none;" onchange="importDataBackup(event)">
        </div>

        <div class="header-divider"></div>

        <div class="header-group">
            <button class="btn" id="btn-dash-toggle" onclick="toggleSidebar()">📊 대시보드 접기</button>
            <button class="btn" id="btn-fullscreen" onclick="toggleFullScreen()">⛶ 전체 화면</button>
        </div>
        
        <!-- 우측 끝: Firebase 실시간 동기화 상태 뱃지 -->
        <div id="sync-badge">
            <span class="sync-dot"></span>
            <span id="sync-status-text">동기화 준비중...</span>
        </div>
    </div>

    <div id="layout-edit-banner">🛠️ 레이아웃 편집 모드 : 믹서기, 굴착기, 사람, 부스를 마우스로 잡아 옮기세요!</div>

    <div id="main-layout">
        <div id="webgl-container">
            <!-- 상단 현장 지시사항 LED 전광판 -->
            <div id="ticker-wrap">
                <div class="ticker-badge">🚨 현장 지시사항</div>
                <div class="ticker-track">
                    <div class="ticker-content" id="ticker-text">※ 작업 공지사항을 불러오는 중입니다...</div>
                </div>
            </div>

            <!-- 대시보드를 안 펴도 우측 위에 크게 뜨는 네온 디지털 시계 HUD -->
            <div id="floating-hud-clock">
                <div class="hud-date" id="hud-date">2025. 00. 00 (월)</div>
                <div class="hud-time" id="hud-time">00:00:00</div>
            </div>
        </div>

        <!-- 독립 배치된 화살표 탭 버튼 -->
        <div id="sidebar-toggle-btn" onclick="toggleSidebar()" title="대시보드 접기/펼치기">▶</div>

        <!-- 우측 사이드 패널 -->
        <div id="right-sidebar">
            <div class="clock-card">
                <div class="c-date" id="clock-date">2025. 00. 00 (월)</div>
                <div class="c-time" id="clock-time">00:00:00</div>
            </div>

            <div class="dash-section">
                <h4>
                    <span>👷 금일 작업 조 (2일 맞교대)</span>
                    <span class="auto-badge" onclick="openShiftSettingModal()" style="cursor:pointer;" title="클릭하여 교대 설정">⚙️ 설정</span>
                </h4>
                <div class="shift-display-box">
                    <span id="cur-shift-name" class="shift-main-text">B조 근무 (07:00 ~ 19:00)</span>
                </div>
                <div id="shift-day-info" style="font-size:0.8rem; color:#64748b; margin-top:8px; font-weight:bold; text-align:center;">
                    (B조 근무중)
                </div>
            </div>

            <div class="dash-section">
                <h4>
                    <span>👥 금일 작업자 명단</span>
                    <span class="auto-badge" id="worker-count-badge">4명 근무중</span>
                </h4>
                <div id="worker-tags" class="worker-tags-container"></div>
            </div>

            <div class="dash-section">
                <h4>📢 현장 안전 및 작업 공지사항</h4>
                <textarea id="db-notice" class="dash-textarea notice-textarea" oninput="onNoticeEdit()" placeholder="금일 주요 작업 사항 및 안전 주의사항을 입력하세요."></textarea>
                <div style="font-size:0.75rem; color:#94a3b8; margin-top:4px;">* 여기에 적으면 화면 위 LED 전광판에 실시간으로 흐릅니다.</div>
            </div>
        </div>
    </div>

    <!-- 래들 마우스 우클릭 플로팅 카드 -->
    <div id="ladle-floating-card">
        <h4>
            <span id="fc-title">래들 정보 수정</span>
            <div style="display:flex; align-items:center; gap:8px;">
                <button class="btn" style="background:#fee2e2; color:#dc2626; border:1.5px solid #f87171; padding:3px 9px; font-size:0.75rem; border-radius:5px;" onclick="deleteCurrentLadle()" title="래들 삭제">🗑️ 삭제</button>
                <span style="cursor:pointer; color:#94a3b8; font-size:1.15rem; padding:2px 4px; font-weight:bold;" onclick="closeFloatingCard()">✕</span>
            </div>
        </h4>
        
        <div class="card-row">
            <label>래들 구분</label>
            <select id="fc-kind" class="dash-select" onchange="onLadleKindChange('fc')">
                <option value="티밍래들">🔵 티밍래들 (메탈릭 블루)</option>
                <option value="장입래들">🟠 장입래들 (메탈릭 오렌지)</option>
            </select>
        </div>

        <div class="card-row">
            <label>베셀 호수 (숫자만)</label>
            <input type="number" id="fc-name" class="dash-input" oninput="onFloatingCardEdit()" min="1" max="999">
        </div>

        <div class="card-row">
            <label>수리 구분</label>
            <select id="fc-repair" class="dash-select" onchange="onFloatingCardEdit()"></select>
        </div>

        <div class="card-row">
            <label>작업 상태</label>
            <select id="fc-status" class="dash-select" onchange="onFloatingCardEdit()">
                <option value="축조중">🔵 축조중</option>
                <option value="예열중">🟠 예열/건조</option>
                <option value="해체중">🔴 해체중</option>
                <option value="대기">⚪ 대기</option>
                <option value="완료">🟢 시공완료</option>
            </select>
        </div>

        <div class="card-row">
            <label>작업 및 조치사항</label>
            <textarea id="fc-note" class="dash-textarea" oninput="onFloatingCardEdit()" style="height:70px; background:#fef9c3; border-color:#ca8a04; color:#713f12; font-weight:bold;"></textarea>
        </div>

        <div style="display:flex; justify-content:center; margin-top:14px;">
            <button class="btn btn-add" style="width:100%; justify-content:center; padding:8px 0; font-size:0.92rem; font-weight:900;" onclick="closeFloatingCard()">✔️ 수정 완료</button>
        </div>
    </div>

    <!-- 조별 인원 설정 모달 -->
    <div id="shift-setting-modal" class="modal-overlay">
        <div class="modal-box">
            <h3>⚙️ 조별 작업자 및 교대 기준 설정</h3>
            <div class="form-group">
                <label>B조 1일차 기준일자</label>
                <input type="date" id="cfg-base-date" class="dash-input">
                <span style="font-size:0.75rem; color:#64748b;">* 설정된 기준일로부터 B조(2일) ➔ A조(2일) 맞교대가 자동 계산됩니다.</span>
            </div>
            <div class="form-group">
                <label>🅰️ A조 작업자 명단 (쉼표로 구분)</label>
                <input type="text" id="cfg-group-a" class="dash-input" placeholder="예: 김반장, 이축조, 박해체, 최보수">
            </div>
            <div class="form-group">
                <label>🅱️ B조 작업자 명단 (쉼표로 구분)</label>
                <input type="text" id="cfg-group-b" class="dash-input" placeholder="예: 정조장, 강연와, 조정비, 윤안전">
            </div>
            <div style="display:flex; justify-content:flex-end; gap:10px; margin-top:20px;">
                <button class="btn" onclick="closeShiftSettingModal()">취소</button>
                <button class="btn btn-add" onclick="saveShiftSettings()">설정 저장</button>
            </div>
        </div>
    </div>

    <!-- 신규 래들 등록 모달 -->
    <div id="ladle-modal" class="modal-overlay">
        <div class="modal-box">
            <h3>🏗️ 3D 래들 베셀 신규 등록</h3>
            <div class="form-group">
                <label>1. 래들 종류 선택</label>
                <div class="radio-group">
                    <label><input type="radio" name="m-kind" value="티밍래들" checked onchange="onModalKindChange('티밍래들')"> 🔵 티밍래들</label>
                    <label><input type="radio" name="m-kind" value="장입래들" onchange="onModalKindChange('장입래들')"> 🟠 장입래들</label>
                </div>
            </div>
            <div class="form-group">
                <label>2. 베셀 호수 (숫자만)</label>
                <input type="number" id="m-vessel-no" class="dash-input" placeholder="예: 5" min="1" max="999">
            </div>
            <div class="form-group">
                <label>3. 수리 종류 선택</label>
                <select id="m-repair-type" class="dash-select"></select>
            </div>
            <div class="form-group">
                <label>4. 현재 작업 상태</label>
                <select id="m-status" class="dash-select">
                    <option value="축조중">🔵 축조중</option>
                    <option value="예열중">🟠 예열/건조</option>
                    <option value="해체중">🔴 해체중</option>
                    <option value="대기">⚪ 대기</option>
                    <option value="완료">🟢 시공완료</option>
                </select>
            </div>
            <div class="form-group">
                <label>5. 작업해야 할 내용 및 비고</label>
                <textarea id="m-note" class="dash-textarea" placeholder="작업 내용 기입"></textarea>
            </div>
            <div style="display:flex; justify-content:flex-end; gap:10px; margin-top:16px;">
                <button class="btn" onclick="closeModal()">취소</button>
                <button class="btn btn-add" onclick="submitAddLadle()">등록 완료</button>
            </div>
        </div>
    </div>

    <script>
        /* -------------------------------------------------------------
           ★ VRAM 메모리 누수 방지 헬퍼 (GPU 리소스 강제 해제)
           (수정) Sprite 는 공유 geometry 를 쓰므로 dispose 하지 않음
        ------------------------------------------------------------- */
        function safeDispose3DObject(obj) {
            if (!obj) return;
            obj.traverse((child) => {
                if (child.geometry && !child.isSprite) {
                    child.geometry.dispose();
                }
                if (child.material) {
                    if (Array.isArray(child.material)) {
                        child.material.forEach(m => disposeMaterial(m));
                    } else {
                        disposeMaterial(child.material);
                    }
                }
            });
        }

        function disposeMaterial(mat) {
            if (!mat) return;
            if (mat.map) mat.map.dispose();
            mat.dispose();
        }

        /* -------------------------------------------------------------
           ★ (수정) 날짜 헬퍼 : UTC 가 아닌 로컬(한국) 시간 기준
        ------------------------------------------------------------- */
        function localDateStr(d) {
            d = d || new Date();
            const p = n => String(n).padStart(2, '0');
            return `${d.getFullYear()}-${p(d.getMonth() + 1)}-${p(d.getDate())}`;
        }

        function parseLocalDate(str) {
            const m = /^(\d{4})-(\d{2})-(\d{2})/.exec(str || "");
            if (!m) {
                const t = new Date();
                t.setHours(0, 0, 0, 0);
                return t;
            }
            return new Date(Number(m[1]), Number(m[2]) - 1, Number(m[3]));
        }

        /* -------------------------------------------------------------
           ★ Firebase Realtime Database 실시간 연동
           (수정) 내가 보낸 데이터의 에코 무시 / 저장 대기 중 원격 무시
        ------------------------------------------------------------- */
        const FIREBASE_URL = "https://ladle-72acf-default-rtdb.firebaseio.com/";
        let firebaseRef = null;
        const CLIENT_ID = Math.random().toString(36).slice(2) + Date.now().toString(36);
        let pendingSave = false;

        function setSyncText(text) {
            const el = document.getElementById('sync-status-text');
            if (el) el.innerText = text;
        }

        // 원격 데이터를 로컬에 캐시 (카메라 시점은 PC별 로컬 값 유지)
        function cacheRemoteToLocal(remoteData) {
            try {
                const merged = Object.assign({}, remoteData, { camera_view: defaultCameraView });
                localStorage.setItem(STORAGE_KEY, JSON.stringify(merged));
            } catch (e) {}
        }

        function initFirebaseConnection() {
            try {
                if (typeof firebase !== 'undefined' && firebase.initializeApp) {
                    if (!firebase.apps.length) {
                        firebase.initializeApp({ databaseURL: FIREBASE_URL });
                    }
                    const db = firebase.database();
                    firebaseRef = db.ref('factory_layout');

                    db.ref('.info/connected').on('value', (snap) => {
                        if (snap.val() === true) setSyncText("Firebase 실시간 동기화됨");
                        else setSyncText("연결 대기중 (로컬 저장)");
                    });

                    firebaseRef.on('value', (snapshot) => {
                        const remoteData = snapshot.val();
                        if (!remoteData) {
                            triggerAutoSave();
                            return;
                        }

                        cacheRemoteToLocal(remoteData);

                        // ★ 내가 보낸 데이터가 되돌아온 것(에코)이면 화면 재생성 없이 무시
                        if (remoteData.meta && remoteData.meta.by === CLIENT_ID) return;

                        // ★ 내 변경이 전송 대기 중이면 곧 내 데이터가 덮어쓰므로 무시
                        if (pendingSave) return;

                        // ★ 타이핑 중에는 서버 데이터로 화면을 덮어쓰지 않도록 보호
                        if (isEditingText) return;

                        applyLoadedData(remoteData, true);
                    });
                } else {
                    setSyncText("로컬 모드 (Firebase 미연결)");
                }
            } catch (err) {
                console.warn("Firebase 연동 대기 (로컬 모드 동작):", err);
                setSyncText("로컬 모드 (Firebase 미연결)");
            }
        }

        /* -------------------------------------------------------------
           ★ 로컬 스토리지 (localStorage) 데이터 시스템
        ------------------------------------------------------------- */
        const STORAGE_KEY = 'POSCO_LADLE_3D_LAYOUT_DATA';
        const today_str = localDateStr();

        const DEFAULT_DATA = {
            camera_view: { pos: [0, 85, 115], target: [0, 5, 0] },
            shift_settings: {
                base_date: today_str,
                group_a: "김반장, 이축조, 박해체, 최보수",
                group_b: "정조장, 강연와, 조정비, 윤안전"
            },
            dashboard: {
                notice: "※ 래들 3호 슬래그라인 연와 축조 후 14:00 예열 버너 점화 예정  |  ※ 작업장 바닥 크레인 이동 반경 안전 통로 확보 철저  |  ※ 안전모 및 방열 보호구 착용 필수"
            },
            facilities: {
                booth: { x: -50, z: -28 },
                mortar: { x: -34, z: -28 },
                vortex: { x: -16, z: -28 },
                ct: { x: 4, z: -28 },
                excavator: { x: 50, z: -7 }
            },
            vessels: [
                {"kind": "티밍래들", "name": "1", "repair": "1차 소수리", "status": "축조중", "note": "하부 노즐 교체 및 바닥 연와 보수", "x": -32, "z": 12},
                {"kind": "장입래들", "name": "2", "repair": "소수리", "status": "축조중", "note": "슬래그라인 연와 전면 교체", "x": -14, "z": 12},
                {"kind": "티밍래들", "name": "3", "repair": "대수리", "status": "예열중", "note": "버너 예열 점화 (3시간 경과)", "x": 4, "z": 12},
                {"kind": "장입래들", "name": "4", "repair": "대수리", "status": "대기", "note": "", "x": 22, "z": 12},
                {"kind": "티밍래들", "name": "5", "repair": "준대수리", "status": "해체중", "note": "", "x": 43, "z": 12}
            ]
        };

        function loadLocalData() {
            try {
                const stored = localStorage.getItem(STORAGE_KEY);
                if (stored) return JSON.parse(stored);
            } catch (e) {}
            return JSON.parse(JSON.stringify(DEFAULT_DATA));
        }

        // ★ (수정) Firebase 는 빈 배열을 지우거나 객체로 바꿀 수 있어 안전하게 정규화
        function normalizeVessels(v) {
            if (!v) return [];
            const arr = Array.isArray(v) ? v : Object.values(v);
            return arr.filter(x => x && typeof x === 'object').map(x => ({
                kind: (x.kind === "장입래들") ? "장입래들" : "티밍래들",
                name: String(x.name || "1"),
                repair: x.repair || "",
                status: x.status || "대기",
                note: x.note || "",
                x: Number(x.x) || 0,
                z: Number(x.z) || 0
            }));
        }

        function buildSaveData() {
            const noticeEl = document.getElementById('db-notice');
            return {
                camera_view: defaultCameraView,
                shift_settings: shiftSettings,
                dashboard: { notice: noticeEl ? noticeEl.value : "" },
                facilities: facilityPositions,
                vessels: getCurrentVesselsState()
            };
        }

        let saveDebounceTimer = null;
        function triggerAutoSave() {
            // 브라우저 로컬 저장 (즉시)
            try {
                localStorage.setItem(STORAGE_KEY, JSON.stringify(buildSaveData()));
            } catch (e) {}

            if (!firebaseRef) return;

            // Firebase 실시간 DB 디바운스 전송 (네트워크 스팸 방지)
            pendingSave = true;
            clearTimeout(saveDebounceTimer);
            saveDebounceTimer = setTimeout(() => {
                const payload = buildSaveData();
                delete payload.camera_view;   // ★ 카메라 시점은 PC별 로컬 값 (서로 덮어쓰지 않음)
                payload.meta = { by: CLIENT_ID, at: Date.now() };
                pendingSave = false;
                firebaseRef.set(payload).catch((err) => {
                    console.warn("Firebase 저장 에러:", err);
                });
            }, 300);
        }

        /* 💾 JSON 백업 파일 다운로드 & 📂 복원 기능 */
        function exportDataBackup() {
            const dataToSave = buildSaveData();
            const blob = new Blob([JSON.stringify(dataToSave, null, 2)], { type: 'application/json' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            const dateStr = localDateStr().replace(/-/g, '');
            a.href = url;
            a.download = `래들_레이아웃_백업_${dateStr}.json`;
            a.click();
            URL.revokeObjectURL(url);
        }

        function importDataBackup(event) {
            const file = event.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const imported = JSON.parse(e.target.result);
                    // ★ (수정) 형식 검증
                    if (!imported || typeof imported !== 'object' ||
                        !(imported.vessels || imported.facilities || imported.shift_settings || imported.dashboard)) {
                        throw new Error("invalid format");
                    }
                    if (imported.vessels) pushHistoryState();   // ★ 복원도 되돌리기 가능
                    applyLoadedData(imported);
                    triggerAutoSave();
                    alert("📂 백업 데이터를 성공적으로 불러왔습니다!");
                } catch (err) {
                    alert("올바른 JSON 백업 파일이 아닙니다.");
                }
            };
            reader.readAsText(file);
            event.target.value = '';
        }

        /* -------------------------------------------------------------
           1. 실시간 시계 & 날짜
        ------------------------------------------------------------- */
        function startRealtimeClock() {
            const dateEl = document.getElementById('clock-date');
            const timeEl = document.getElementById('clock-time');
            const hudDateEl = document.getElementById('hud-date');
            const hudTimeEl = document.getElementById('hud-time');
            const days = ['일', '월', '화', '수', '목', '금', '토'];

            function updateTime() {
                const now = new Date();
                const year = now.getFullYear();
                const month = String(now.getMonth() + 1).padStart(2, '0');
                const date = String(now.getDate()).padStart(2, '0');
                const day = days[now.getDay()];

                const hours = String(now.getHours()).padStart(2, '0');
                const minutes = String(now.getMinutes()).padStart(2, '0');
                const seconds = String(now.getSeconds()).padStart(2, '0');

                const dateStr = `${year}. ${month}. ${date} (${day})`;
                const timeStr = `${hours}:${minutes}:${seconds}`;

                if (dateEl) dateEl.innerText = dateStr;
                if (timeEl) timeEl.innerText = timeStr;
                if (hudDateEl) hudDateEl.innerText = dateStr;
                if (hudTimeEl) hudTimeEl.innerText = timeStr;
            }

            updateTime();
            setInterval(updateTime, 1000);
        }
        startRealtimeClock();

        /* -------------------------------------------------------------
           2. 전체 화면 & 대시보드 토글
        ------------------------------------------------------------- */
        function toggleFullScreen() {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch(err => {
                    alert("전체화면 전환 실패: " + err.message);
                });
            } else {
                if (document.exitFullscreen) document.exitFullscreen();
            }
        }
        document.addEventListener('fullscreenchange', () => {
            const btn = document.getElementById('btn-fullscreen');
            if (document.fullscreenElement) {
                btn.innerHTML = '🗗 전체화면 종료';
                btn.style.backgroundColor = '#fef08a';
            } else {
                btn.innerHTML = '⛶ 전체 화면';
                btn.style.backgroundColor = '#ffffff';
            }
            setTimeout(onWindowResize, 150);
        });

        let isSidebarCollapsed = false;
        function toggleSidebar() {
            isSidebarCollapsed = !isSidebarCollapsed;
            const sidebar = document.getElementById('right-sidebar');
            const toggleBtn = document.getElementById('sidebar-toggle-btn');
            const headToggleBtn = document.getElementById('btn-dash-toggle');
            const hudClock = document.getElementById('floating-hud-clock');
            const tickerWrap = document.getElementById('ticker-wrap');

            if (isSidebarCollapsed) {
                sidebar.classList.add('collapsed');
                toggleBtn.classList.add('collapsed');
                toggleBtn.innerText = '◀';
                toggleBtn.title = '대시보드 펼치기';
                headToggleBtn.innerHTML = '📊 대시보드 펼치기';
                headToggleBtn.style.backgroundColor = '#e0f2fe';
                headToggleBtn.style.borderColor = '#0284c7';
                headToggleBtn.style.color = '#0284c7';
                hudClock.style.right = "18px";
                tickerWrap.style.right = "14px";
            } else {
                sidebar.classList.remove('collapsed');
                toggleBtn.classList.remove('collapsed');
                toggleBtn.innerText = '▶';
                toggleBtn.title = '대시보드 접기';
                headToggleBtn.innerHTML = '📊 대시보드 접기';
                headToggleBtn.style.backgroundColor = '#ffffff';
                headToggleBtn.style.borderColor = '#cbd5e1';
                headToggleBtn.style.color = '#334155';
                hudClock.style.right = "398px";
                tickerWrap.style.right = "394px";
            }
            setTimeout(onWindowResize, 320);
        }

        /* -------------------------------------------------------------
           3. 래들 수리 유형
        ------------------------------------------------------------- */
        const REPAIR_OPTIONS_TEEMING = ["1차 소수리", "2차 소수리", "3차 소수리", "중수리", "준대수리", "대수리"];
        const REPAIR_OPTIONS_CHARGING = ["소수리", "대수리"];

        function populateRepairSelect(selectEl, kind, selectedVal) {
            selectEl.innerHTML = "";
            const list = (kind === "장입래들") ? REPAIR_OPTIONS_CHARGING : REPAIR_OPTIONS_TEEMING;
            list.forEach(item => {
                const opt = document.createElement('option');
                opt.value = item;
                opt.innerText = item;
                if (item === selectedVal) opt.selected = true;
                selectEl.appendChild(opt);
            });
            if (!list.includes(selectedVal)) {
                selectEl.value = list[0];
            }
        }

        function onLadleKindChange(prefix) {
            const kind = document.getElementById(`${prefix}-kind`).value;
            const repSelect = document.getElementById(`${prefix}-repair`);
            populateRepairSelect(repSelect, kind, repSelect.value);
            if (prefix === 'fc') onFloatingCardEdit();
        }

        function onModalKindChange(kind) {
            const repSelect = document.getElementById('m-repair-type');
            populateRepairSelect(repSelect, kind, repSelect.value);
        }

        function getVesselLabel(kind, rawNum) {
            const numOnly = String(rawNum).replace(/[^0-9]/g, '');
            const num = numOnly ? numOnly : "1";
            const shortKind = kind.includes("티밍") ? "티밍" : "장입";
            return `${shortKind} ${num}호`;
        }

        const SNAP_POINTS = [
            { x: -32, z: 12 },
            { x: -14, z: 12 },
            { x: 4, z: 12 },
            { x: 22, z: 12 },
            { x: 43, z: 12 },
            { x: 57, z: 12 }
        ];

        // ★ (수정) 비어 있는 스냅 자리 찾기 (신규 등록 시 겹침 방지)
        function findFreeSnap() {
            const free = SNAP_POINTS.find(sp =>
                !ladleObjects.some(o => Math.hypot(o.position.x - sp.x, o.position.z - sp.z) < 1)
            );
            if (free) return { x: free.x, z: free.z };
            const extra = Math.max(0, ladleObjects.length - SNAP_POINTS.length);
            return { x: -32 + extra * 18, z: 36 };
        }

        /* -------------------------------------------------------------
           4. 2조 2교대 자동 계산 & LED 전광판
        ------------------------------------------------------------- */
        let shiftSettings = {
            base_date: today_str,
            group_a: "김반장, 이축조, 박해체, 최보수",
            group_b: "정조장, 강연와, 조정비, 윤안전"
        };
        let currentActiveWorkers = [];
        let lastWorkerSig = null;

        function calculateTodayShift() {
            const today = new Date();
            today.setHours(0, 0, 0, 0);

            const base = parseLocalDate(shiftSettings.base_date);

            const diffDays = Math.round((today.getTime() - base.getTime()) / (1000 * 60 * 60 * 24));
            const cycleDay = ((diffDays % 4) + 4) % 4;

            const shiftNameEl = document.getElementById('cur-shift-name');
            const shiftInfoEl = document.getElementById('shift-day-info');
            const tagsContainer = document.getElementById('worker-tags');
            const badgeEl = document.getElementById('worker-count-badge');

            let isGroupB = (cycleDay === 0 || cycleDay === 1);
            let workerStr = (isGroupB ? shiftSettings.group_b : shiftSettings.group_a) || "";

            if (isGroupB) {
                shiftNameEl.innerText = "B조 근무 (07:00 ~ 19:00)";
                shiftNameEl.style.color = "#0284c7";
                if (cycleDay === 0) shiftInfoEl.innerText = "📅 B조 1일차 근무 (내일 B조 2일차)";
                else shiftInfoEl.innerText = "📅 B조 2일차 근무 (내일부터 A조 2일 근무)";
            } else {
                shiftNameEl.innerText = "A조 근무 (07:00 ~ 19:00)";
                shiftNameEl.style.color = "#ea580c";
                if (cycleDay === 2) shiftInfoEl.innerText = "📅 A조 1일차 근무 (내일 A조 2일차)";
                else shiftInfoEl.innerText = "📅 A조 2일차 근무 (내일부터 B조 2일 근무)";
            }

            currentActiveWorkers = workerStr.split(',')
                .map(s => s.trim())
                .filter(s => s.length > 0);

            badgeEl.innerText = `${currentActiveWorkers.length}명 근무중`;
            tagsContainer.innerHTML = "";
            currentActiveWorkers.forEach(name => {
                // ★ (수정) innerHTML 대신 textContent 사용 (XSS 방지)
                const tag = document.createElement('div');
                tag.className = 'worker-tag';
                const s1 = document.createElement('span');
                s1.textContent = '👷';
                const s2 = document.createElement('span');
                s2.textContent = name;
                tag.appendChild(s1);
                tag.appendChild(s2);
                tagsContainer.appendChild(tag);
            });

            // ★ (수정) 근무조/명단이 실제로 바뀐 경우에만 3D 작업자 재생성 (위치 초기화·유령 방지)
            const sig = (isGroupB ? 'B' : 'A') + '|' + currentActiveWorkers.join('|');
            if (sig !== lastWorkerSig) {
                lastWorkerSig = sig;
                rebuild3DWorkers(currentActiveWorkers);
            }
        }

        // ★ (수정) 자정이 지나면 근무조/명단 자동 갱신 (24시간 가동 전광판 대응)
        let lastShiftDay = localDateStr();
        setInterval(() => {
            const nowDay = localDateStr();
            if (nowDay !== lastShiftDay) {
                lastShiftDay = nowDay;
                calculateTodayShift();
            }
        }, 30000);

        function updateLedTicker(text) {
            const tickerEl = document.getElementById('ticker-text');
            if (!text || text.trim() === "") {
                tickerEl.innerText = "안전벨트 착용 철저히 합시다. 작업장 바닥 크레인 이동 반경 통로 확보 철저";
            } else {
                let clean = text.split('\n').join('   •   ').split('\r').join('').split('|').join('   •   ').trim();
                while (clean.startsWith('•')) clean = clean.substring(1).trim();
                while (clean.endsWith('•')) clean = clean.substring(0, clean.length - 1).trim();
                tickerEl.innerText = clean;
            }
            adjustTickerSpeed();
        }

        // ★ (수정) 글자 길이에 비례해 전광판 속도 조절 (긴 공지도 읽을 수 있게)
        function adjustTickerSpeed() {
            const el = document.getElementById('ticker-text');
            requestAnimationFrame(() => {
                const dur = Math.max(18, Math.round(el.scrollWidth / 110));
                if (el.dataset.dur !== String(dur)) {
                    el.dataset.dur = String(dur);
                    el.style.animationDuration = dur + 's';
                }
            });
        }

        /* -------------------------------------------------------------
           5. 실행 취소(Undo) & 다시 실행(Redo) 시스템
        ------------------------------------------------------------- */
        let undoStack = [];
        let redoStack = [];
        const MAX_HISTORY = 40;

        function getCurrentVesselsState() {
            return ladleObjects.map(obj => ({
                kind: obj.userData.kind,
                name: obj.userData.name,
                repair: obj.userData.repair,
                status: obj.userData.status,
                note: obj.userData.note,
                x: obj.position.x,
                z: obj.position.z
            }));
        }

        function pushHistoryState() {
            const state = getCurrentVesselsState();
            undoStack.push(JSON.stringify(state));
            if (undoStack.length > MAX_HISTORY) undoStack.shift();
            redoStack = [];
            updateHistoryButtons();
        }

        function updateHistoryButtons() {
            document.getElementById('btn-undo').disabled = (undoStack.length === 0);
            document.getElementById('btn-redo').disabled = (redoStack.length === 0);
        }

        function undo() {
            // ★ (수정) 카드에서 수정 중이던 내용도 이력에 반영한 뒤 되돌림
            if (floatingCard.style.display === 'block') closeFloatingCard();
            if (undoStack.length === 0) return;
            const currentState = getCurrentVesselsState();
            redoStack.push(JSON.stringify(currentState));

            const prevStateStr = undoStack.pop();
            const prevState = JSON.parse(prevStateStr);
            restoreVesselsState(prevState);
            updateHistoryButtons();
            triggerAutoSave();
        }

        function redo() {
            if (floatingCard.style.display === 'block') closeFloatingCard();
            if (redoStack.length === 0) return;
            const currentState = getCurrentVesselsState();
            undoStack.push(JSON.stringify(currentState));

            const nextStateStr = redoStack.pop();
            const nextState = JSON.parse(nextStateStr);
            restoreVesselsState(nextState);
            updateHistoryButtons();
            triggerAutoSave();
        }

        function restoreVesselsState(vesselsList) {
            hideFloatingCardSilently();
            selectedLadle = null;
            ladleObjects.forEach(obj => {
                safeDispose3DObject(obj);
                scene.remove(obj);
            });
            ladleObjects = [];
            hitMeshList = [];
            vesselsList.forEach(v => createLadleMesh(v));
        }

        // ★ (수정) 한글 입력 상태 / Mac(Cmd) 에서도 단축키 동작
        window.addEventListener('keydown', (e) => {
            if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
            const mod = e.ctrlKey || e.metaKey;
            if (mod && e.code === 'KeyZ') {
                e.preventDefault();
                if (e.shiftKey) redo();
                else undo();
            } else if (mod && e.code === 'KeyY') {
                e.preventDefault();
                redo();
            }
        });


        /* -------------------------------------------------------------
           6. Three.js 초기화
        ------------------------------------------------------------- */
        const clock = new THREE.Clock();
        let animTime = 0;
        let defaultCameraView = { pos: [0, 85, 115], target: [0, 5, 0] };

        let scene, camera, renderer, controls;
        const container = document.getElementById('webgl-container');
        const modal = document.getElementById('ladle-modal');
        const settingModal = document.getElementById('shift-setting-modal');
        const floatingCard = document.getElementById('ladle-floating-card');

        let ladleObjects = [];
        let hitMeshList = [];
        let facilityHitMeshList = [];
        let selectedObject = null;
        let selectedLadle = null;
        let cardSnapshot = null;   // ★ 카드 열 때의 상태 (닫을 때 변경됐으면 되돌리기 이력에 추가)

        let isDragging = false;
        let isEditingText = false;
        let isLayoutEditMode = false;
        let dragStartSnapshot = null;

        const dragPlane = new THREE.Plane(new THREE.Vector3(0, 1, 0), 0);
        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();
        const planeIntersect = new THREE.Vector3();
        const dragOffset = new THREE.Vector3();

        let boothMesh, mortarMesh, vortexMesh, ctMesh, excavatorGroup;
        let forkliftGroup;
        let excCabin, excBoom, excArm, excBreaker;
        let workers = [];

        let facilityPositions = {
            booth: { x: -50, z: -28 },
            mortar: { x: -34, z: -28 },
            vortex: { x: -16, z: -28 },
            ct: { x: 4, z: -28 },
            excavator: { x: 50, z: -7 }
        };

        init3DScene();
        buildFactoryEnvironment();
        buildExcavatorSimulation();
        buildForkliftSimulation();
        animate();

        function init3DScene() {
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0xf8fafc);

            const w = isSidebarCollapsed ? window.innerWidth : (window.innerWidth - 380);
            const h = window.innerHeight - 54;

            camera = new THREE.PerspectiveCamera(45, w / h, 1, 1000);
            camera.position.set(defaultCameraView.pos[0], defaultCameraView.pos[1], defaultCameraView.pos[2]);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.setSize(w, h);
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;
            container.appendChild(renderer.domElement);

            controls = new THREE.OrbitControls(camera, renderer.domElement);
            controls.enableDamping = true;
            controls.dampingFactor = 0.05;
            controls.maxPolarAngle = Math.PI / 2.05;
            controls.target.set(defaultCameraView.target[0], defaultCameraView.target[1], defaultCameraView.target[2]);

            // ★ (수정) LEFT=PAN 고정. Ctrl+드래그 회전은 OrbitControls 가 자체 처리
            controls.mouseButtons = {
                LEFT: THREE.MOUSE.PAN,
                MIDDLE: THREE.MOUSE.DOLLY,
                RIGHT: THREE.MOUSE.NONE
            };

            const hemiLight = new THREE.HemisphereLight(0xffffff, 0xe2e8f0, 0.85);
            hemiLight.position.set(0, 100, 0);
            scene.add(hemiLight);

            const dirLight = new THREE.DirectionalLight(0xffffff, 0.85);
            dirLight.position.set(60, 120, 60);
            dirLight.castShadow = true;
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            // ★ (수정) 그림자 범위를 공장 전체로 확장 (기본값 ±5 는 거의 안 보임)
            dirLight.shadow.camera.left = -110;
            dirLight.shadow.camera.right = 110;
            dirLight.shadow.camera.top = 110;
            dirLight.shadow.camera.bottom = -110;
            dirLight.shadow.camera.near = 10;
            dirLight.shadow.camera.far = 400;
            dirLight.shadow.camera.updateProjectionMatrix();
            scene.add(dirLight);

            const subLight = new THREE.DirectionalLight(0xbae6fd, 0.35);
            subLight.position.set(-60, 80, -60);
            scene.add(subLight);

            window.addEventListener('resize', onWindowResize);
            setupDragAndSelect();
        }

        function saveCurrentCameraView() {
            defaultCameraView = {
                pos: [camera.position.x, camera.position.y, camera.position.z],
                target: [controls.target.x, controls.target.y, controls.target.z]
            };
            triggerAutoSave();
            alert("🎥 현재 카메라 시점이 [기본 시점]으로 브라우저에 저장되었습니다!");
        }

        function resetCamera() {
            camera.position.set(defaultCameraView.pos[0], defaultCameraView.pos[1], defaultCameraView.pos[2]);
            controls.target.set(defaultCameraView.target[0], defaultCameraView.target[1], defaultCameraView.target[2]);
        }

        function onWindowResize() {
            const w = isSidebarCollapsed ? window.innerWidth : (window.innerWidth - 380);
            const h = window.innerHeight - 54;
            camera.aspect = w / h;
            camera.updateProjectionMatrix();
            renderer.setSize(w, h);
        }

        function toggleLayoutEditMode() {
            isLayoutEditMode = !isLayoutEditMode;
            const btn = document.getElementById('btn-layout-mode');
            const banner = document.getElementById('layout-edit-banner');

            if (isLayoutEditMode) {
                btn.innerHTML = "💾 배치 완료 (저장)";
                btn.classList.add('btn-edit-active');
                banner.style.display = "block";
            } else {
                btn.innerHTML = "🛠️ 시설/인원 배치";
                btn.classList.remove('btn-edit-active');
                banner.style.display = "none";
                triggerAutoSave();
            }
        }

        /* -------------------------------------------------------------
           7. 공장 설비 & 해체장/축조장 바닥 & 굴착기 & 지게차
        ------------------------------------------------------------- */
        function registerDraggableFacility(group, key, is2F, hitRadius, hitHeight) {
            group.userData.facilityKey = key;
            group.userData.is2F = is2F;

            const hit = new THREE.Mesh(
                new THREE.CylinderGeometry(hitRadius, hitRadius, hitHeight, 16),
                new THREE.MeshBasicMaterial({ visible: false })
            );
            hit.position.y = hitHeight / 2;
            hit.userData.parentGroup = group;
            group.add(hit);
            facilityHitMeshList.push(hit);
        }

        function buildFactoryEnvironment() {
            const floorGeo = new THREE.PlaneGeometry(165, 125);
            const floorMat = new THREE.MeshStandardMaterial({ color: 0xe2e8f0, roughness: 0.6 });
            const floor = new THREE.Mesh(floorGeo, floorMat);
            floor.rotation.x = -Math.PI / 2;
            floor.receiveShadow = true;
            scene.add(floor);

            const grid = new THREE.GridHelper(165, 33, 0x94a3b8, 0xcbd5e1);
            grid.position.y = 0.05;
            scene.add(grid);

            const safeLine = new THREE.Mesh(new THREE.BoxGeometry(75, 0.1, 0.8), new THREE.MeshBasicMaterial({ color: 0xfacc15 }));
            safeLine.position.set(-20, 0.1, 15);
            scene.add(safeLine);

            const zoneDivider = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.15, 36), new THREE.MeshBasicMaterial({ color: 0xdc2626 }));
            zoneDivider.position.set(32, 0.1, 12);
            scene.add(zoneDivider);

            const mezFloor = new THREE.Mesh(
                new THREE.BoxGeometry(90, 1.5, 40),
                new THREE.MeshStandardMaterial({ color: 0x94a3b8, metalness: 0.2, roughness: 0.4 })
            );
            mezFloor.position.set(-15, 18, -25);
            mezFloor.castShadow = true;
            mezFloor.receiveShadow = true;
            scene.add(mezFloor);

            const colMat = new THREE.MeshStandardMaterial({ color: 0x475569 });
            [[-55, -40], [25, -40], [-55, -10], [25, -10]].forEach(pos => {
                const col = new THREE.Mesh(new THREE.BoxGeometry(2, 18, 2), colMat);
                col.position.set(pos[0], 9, pos[1]);
                col.castShadow = true;
                scene.add(col);
            });

            const railMat = new THREE.MeshStandardMaterial({ color: 0xeab308 });
            const rail1 = new THREE.Mesh(new THREE.BoxGeometry(90, 0.4, 0.4), railMat);
            rail1.position.set(-15, 21.5, -5.2);
            scene.add(rail1);
            for(let rx = -58; rx <= 28; rx += 7) {
                const post = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 3), railMat);
                post.position.set(rx, 20, -5.2);
                scene.add(post);
            }

            boothMesh = new THREE.Group();
            const booth = new THREE.Mesh(
                new THREE.BoxGeometry(16, 12, 18),
                new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.3 })
            );
            booth.position.y = 6;
            booth.castShadow = true;
            boothMesh.add(booth);
            const win = new THREE.Mesh(new THREE.PlaneGeometry(10, 5), new THREE.MeshStandardMaterial({ color: 0x38bdf8, roughness: 0.1 }));
            win.position.set(0, 6.5, 9.1);
            boothMesh.add(win);
            boothMesh.position.set(facilityPositions.booth.x, 18.75, facilityPositions.booth.z);
            registerDraggableFacility(boothMesh, 'booth', true, 10, 13);
            scene.add(boothMesh);

            mortarMesh = new THREE.Group();
            const tank1 = new THREE.Mesh(new THREE.CylinderGeometry(3, 3, 5, 24), new THREE.MeshStandardMaterial({ color: 0xf59e0b, metalness: 0.4 }));
            tank1.position.y = 4.5;
            tank1.castShadow = true;
            mortarMesh.add(tank1);
            const motor1 = new THREE.Mesh(new THREE.BoxGeometry(2.5, 2.5, 2.5), new THREE.MeshStandardMaterial({ color: 0x475569 }));
            motor1.position.y = 8;
            mortarMesh.add(motor1);
            for(let a=0; a<3; a++) {
                const leg = new THREE.Mesh(new THREE.CylinderGeometry(0.3, 0.3, 3), new THREE.MeshStandardMaterial({ color: 0x334155 }));
                leg.position.set(Math.cos(a*2.1)*2.5, 1.5, Math.sin(a*2.1)*2.5);
                mortarMesh.add(leg);
            }
            mortarMesh.position.set(facilityPositions.mortar.x, 18.75, facilityPositions.mortar.z);
            registerDraggableFacility(mortarMesh, 'mortar', true, 4, 10);
            scene.add(mortarMesh);

            vortexMesh = new THREE.Group();
            const vTank = new THREE.Mesh(new THREE.CylinderGeometry(3.5, 3.5, 4.5, 24), new THREE.MeshStandardMaterial({ color: 0x0284c7, metalness: 0.5 }));
            vTank.position.y = 3;
            vTank.castShadow = true;
            vortexMesh.add(vTank);
            const vTop = new THREE.Mesh(new THREE.ConeGeometry(3.6, 2, 24), new THREE.MeshStandardMaterial({ color: 0x0369a1 }));
            vTop.position.y = 6;
            vortexMesh.add(vTop);
            vortexMesh.position.set(facilityPositions.vortex.x, 18.75, facilityPositions.vortex.z);
            registerDraggableFacility(vortexMesh, 'vortex', true, 4.5, 8);
            scene.add(vortexMesh);

            ctMesh = new THREE.Group();
            const ctBarrel = new THREE.Mesh(new THREE.CylinderGeometry(1.6, 1.6, 9, 16), new THREE.MeshStandardMaterial({ color: 0xea580c }));
            ctBarrel.rotation.z = Math.PI / 4;
            ctBarrel.position.set(0, 4, 0);
            ctBarrel.castShadow = true;
            ctMesh.add(ctBarrel);
            const ctHopper = new THREE.Mesh(new THREE.CylinderGeometry(2.5, 0.8, 3, 4), new THREE.MeshStandardMaterial({ color: 0x64748b }));
            ctHopper.position.set(-2.5, 7.5, 0);
            ctMesh.add(ctHopper);
            ctMesh.position.set(facilityPositions.ct.x, 18.75, facilityPositions.ct.z);
            registerDraggableFacility(ctMesh, 'ct', true, 5, 9);
            scene.add(ctMesh);

            const hoistBeam = new THREE.Mesh(new THREE.BoxGeometry(85, 2, 1.5), new THREE.MeshStandardMaterial({ color: 0x1d4ed8 }));
            hoistBeam.position.set(-15, 34, -25);
            scene.add(hoistBeam);
            const hoistTrolley = new THREE.Mesh(new THREE.BoxGeometry(6, 4, 5), new THREE.MeshStandardMaterial({ color: 0xf59e0b }));
            hoistTrolley.position.set(0, 31.5, -25);
            scene.add(hoistTrolley);
            const hook = new THREE.Mesh(new THREE.TorusGeometry(0.8, 0.3, 8, 16), new THREE.MeshStandardMaterial({ color: 0x334155 }));
            hook.position.set(0, 24, -25);
            scene.add(hook);

            const wallRail = new THREE.Mesh(new THREE.BoxGeometry(95, 1.2, 1.2), new THREE.MeshStandardMaterial({ color: 0x059669 }));
            wallRail.position.set(-15, 20, -4.5);
            scene.add(wallRail);
            const craneArm = new THREE.Mesh(new THREE.BoxGeometry(2, 2.5, 14), new THREE.MeshStandardMaterial({ color: 0x10b981 }));
            craneArm.position.set(10, 20, 2);
            scene.add(craneArm);

            // [축조장 스탠드 블록 1~4번]
            for(let i=0; i<4; i++) {
                const standX = -32 + (i * 18);
                const stand = new THREE.Mesh(
                    new THREE.CylinderGeometry(5.8, 6.2, 1.2, 32),
                    new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.5 })
                );
                stand.position.set(standX, 0.6, 12);
                stand.receiveShadow = true;
                scene.add(stand);

                const ring = new THREE.Mesh(new THREE.TorusGeometry(6.4, 0.25, 8, 32), new THREE.MeshBasicMaterial({ color: 0x0284c7 }));
                ring.rotation.x = Math.PI / 2;
                ring.position.set(standX, 0.7, 12);
                scene.add(ring);
            }

            const castTower = new THREE.Group();
            const post = new THREE.Mesh(new THREE.CylinderGeometry(1.2, 1.5, 12), new THREE.MeshStandardMaterial({ color: 0x0284c7 }));
            post.position.y = 6;
            post.castShadow = true;
            castTower.add(post);
            const arm = new THREE.Mesh(new THREE.BoxGeometry(10, 1.2, 1.2), new THREE.MeshStandardMaterial({ color: 0x38bdf8 }));
            arm.position.set(4, 11, 0);
            castTower.add(arm);
            castTower.position.set(-50, 0, 12);
            scene.add(castTower);

            // [해체장 전용 바닥판]
            const dFloor = new THREE.Mesh(
                new THREE.BoxGeometry(34, 1.5, 26),
                new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.8, metalness: 0.3 })
            );
            dFloor.position.set(50, 0.75, 12);
            dFloor.receiveShadow = true;
            scene.add(dFloor);

            const dBorder = new THREE.Mesh(
                new THREE.BoxGeometry(34.8, 0.2, 26.8),
                new THREE.MeshBasicMaterial({ color: 0xf59e0b })
            );
            dBorder.position.set(50, 0.1, 12);
            scene.add(dBorder);

            // [해체장 블록/스탠드 5~6번]
            [43, 57].forEach(dx => {
                const dStand = new THREE.Mesh(
                    new THREE.CylinderGeometry(5.2, 5.5, 1.6, 32),
                    new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.7, roughness: 0.3 })
                );
                dStand.position.set(dx, 1.2, 12);
                scene.add(dStand);

                const redRing = new THREE.Mesh(
                    new THREE.TorusGeometry(5.8, 0.35, 8, 32),
                    new THREE.MeshBasicMaterial({ color: 0xff1744 })
                );
                redRing.rotation.x = Math.PI / 2;
                redRing.position.set(dx, 1.35, 12);
                scene.add(redRing);
            });
        }

        function buildExcavatorSimulation() {
            excavatorGroup = new THREE.Group();

            const yellowMat = new THREE.MeshStandardMaterial({ color: 0xeab308, metalness: 0.3, roughness: 0.4 });
            const darkMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, metalness: 0.6, roughness: 0.5 });
            const steelMat = new THREE.MeshStandardMaterial({ color: 0x64748b, metalness: 0.8, roughness: 0.2 });

            [-2.2, 2.2].forEach(tx => {
                const track = new THREE.Mesh(new THREE.BoxGeometry(1.4, 1.6, 7.5), darkMat);
                track.position.set(tx, 0.8, 0);
                track.castShadow = true;
                excavatorGroup.add(track);
            });
            const baseCenter = new THREE.Mesh(new THREE.BoxGeometry(3.2, 1.2, 5.5), darkMat);
            baseCenter.position.set(0, 1.2, 0);
            excavatorGroup.add(baseCenter);

            excCabin = new THREE.Group();
            excCabin.position.set(0, 2.2, 0);

            const cabBody = new THREE.Mesh(new THREE.BoxGeometry(4.2, 2.8, 4.8), yellowMat);
            cabBody.position.set(0, 1.4, -0.2);
            cabBody.castShadow = true;
            excCabin.add(cabBody);

            const cabGlass = new THREE.Mesh(new THREE.BoxGeometry(2.2, 1.8, 2.2), new THREE.MeshStandardMaterial({ color: 0x38bdf8, roughness: 0.1 }));
            cabGlass.position.set(-1.1, 2.0, 1.2);
            excCabin.add(cabGlass);

            const counterWeight = new THREE.Mesh(new THREE.BoxGeometry(4.2, 2.2, 1.5), darkMat);
            counterWeight.position.set(0, 1.4, -2.4);
            excCabin.add(counterWeight);

            excBoom = new THREE.Group();
            excBoom.position.set(1.2, 1.6, 1.8);

            const boomArmMesh = new THREE.Mesh(new THREE.BoxGeometry(0.8, 5.5, 1.0), yellowMat);
            boomArmMesh.position.set(0, 2.5, 0.6);
            boomArmMesh.rotation.x = Math.PI / 5;
            boomArmMesh.castShadow = true;
            excBoom.add(boomArmMesh);

            excArm = new THREE.Group();
            excArm.position.set(0, 4.6, 2.0);

            const armMesh = new THREE.Mesh(new THREE.BoxGeometry(0.7, 4.5, 0.8), yellowMat);
            armMesh.position.set(0, 1.8, 0.8);
            armMesh.rotation.x = -Math.PI / 3;
            armMesh.castShadow = true;
            excArm.add(armMesh);

            excBreaker = new THREE.Group();
            excBreaker.position.set(0, 3.2, 3.2);

            const breakerBody = new THREE.Mesh(new THREE.BoxGeometry(0.8, 2.2, 0.8), darkMat);
            breakerBody.rotation.x = Math.PI / 4;
            excBreaker.add(breakerBody);

            const chisel = new THREE.Mesh(new THREE.CylinderGeometry(0.12, 0.25, 2.2, 16), steelMat);
            chisel.position.set(0, -1.2, 0.8);
            chisel.rotation.x = Math.PI / 4;
            chisel.castShadow = true;
            excBreaker.add(chisel);

            excArm.add(excBreaker);
            excBoom.add(excArm);
            excCabin.add(excBoom);
            excavatorGroup.add(excCabin);

            excavatorGroup.position.set(facilityPositions.excavator.x, 0, facilityPositions.excavator.z);
            excavatorGroup.rotation.y = Math.PI * 0.15;
            registerDraggableFacility(excavatorGroup, 'excavator', false, 5, 8);
            scene.add(excavatorGroup);

            for(let d=0; d<8; d++) {
                const brick = new THREE.Mesh(
                    new THREE.BoxGeometry(1.4, 0.7, 1.0),
                    new THREE.MeshStandardMaterial({ color: 0x991b1b, roughness: 0.9 })
                );
                brick.position.set(43 + Math.random()*12, 0.4, -1 + Math.random()*5);
                brick.rotation.set(Math.random(), Math.random(), Math.random());
                scene.add(brick);
            }
        }

        function buildForkliftSimulation() {
            forkliftGroup = new THREE.Group();

            const bodyMat = new THREE.MeshStandardMaterial({ color: 0xf59e0b, metalness: 0.3, roughness: 0.4 });
            const darkMat = new THREE.MeshStandardMaterial({ color: 0x1e293b, roughness: 0.6 });
            const steelMat = new THREE.MeshStandardMaterial({ color: 0x475569, metalness: 0.7 });
            const woodMat = new THREE.MeshStandardMaterial({ color: 0xb45309, roughness: 0.8 });

            const body = new THREE.Mesh(new THREE.BoxGeometry(2.8, 1.4, 4.2), bodyMat);
            body.position.y = 1.3;
            body.castShadow = true;
            forkliftGroup.add(body);

            const cageMat = new THREE.MeshStandardMaterial({ color: 0x0f172a });
            [[-1.2, -0.6], [1.2, -0.6], [-1.2, 1.2], [1.2, 1.2]].forEach(p => {
                const bar = new THREE.Mesh(new THREE.BoxGeometry(0.15, 2.5, 0.15), cageMat);
                bar.position.set(p[0], 2.8, p[1]);
                forkliftGroup.add(bar);
            });
            const roof = new THREE.Mesh(new THREE.BoxGeometry(2.6, 0.15, 2.0), cageMat);
            roof.position.set(0, 4.0, 0.3);
            forkliftGroup.add(roof);

            const beacon = new THREE.Mesh(new THREE.CylinderGeometry(0.2, 0.2, 0.35, 12), new THREE.MeshBasicMaterial({ color: 0xf97316 }));
            beacon.position.set(0, 4.25, 0.3);
            forkliftGroup.add(beacon);

            [[-1.5, 1.4], [1.5, 1.4], [-1.4, -1.4], [1.4, -1.4]].forEach(p => {
                const wheel = new THREE.Mesh(new THREE.CylinderGeometry(0.7, 0.7, 0.6, 16), darkMat);
                wheel.rotation.z = Math.PI / 2;
                wheel.position.set(p[0], 0.7, p[1]);
                wheel.castShadow = true;
                forkliftGroup.add(wheel);
            });

            const mast1 = new THREE.Mesh(new THREE.BoxGeometry(0.2, 3.8, 0.2), steelMat);
            mast1.position.set(-0.8, 2.1, 2.3);
            forkliftGroup.add(mast1);
            const mast2 = new THREE.Mesh(new THREE.BoxGeometry(0.2, 3.8, 0.2), steelMat);
            mast2.position.set(0.8, 2.1, 2.3);
            forkliftGroup.add(mast2);

            const forkMat = new THREE.MeshStandardMaterial({ color: 0x334155, metalness: 0.8 });
            [-0.5, 0.5].forEach(fx => {
                const forkV = new THREE.Mesh(new THREE.BoxGeometry(0.12, 1.6, 0.1), forkMat);
                forkV.position.set(fx, 1.0, 2.45);
                forkliftGroup.add(forkV);

                const forkH = new THREE.Mesh(new THREE.BoxGeometry(0.15, 0.08, 2.0), forkMat);
                forkH.position.set(fx, 0.25, 3.45);
                forkH.castShadow = true;
                forkliftGroup.add(forkH);
            });

            const pallet = new THREE.Mesh(new THREE.BoxGeometry(2.0, 0.2, 1.8), woodMat);
            pallet.position.set(0, 0.45, 3.45);
            forkliftGroup.add(pallet);

            const cargo = new THREE.Mesh(new THREE.BoxGeometry(1.6, 1.2, 1.5), new THREE.MeshStandardMaterial({ color: 0x0284c7, roughness: 0.5 }));
            cargo.position.set(0, 1.15, 3.45);
            cargo.castShadow = true;
            forkliftGroup.add(cargo);

            forkliftGroup.position.set(0, 0, 26);
            forkliftGroup.rotation.y = Math.PI / 2;   // ★ (수정) 처음부터 진행 방향(+X)을 바라봄
            // ★ (수정) speed 는 초당 이동 거리 (프레임 속도와 무관하게 동일한 속도)
            forkliftGroup.userData = { dir: 1, minX: -36, maxX: 32, speed: 4.8 };
            scene.add(forkliftGroup);
        }

        /* -------------------------------------------------------------
           8. 작업자 생성 & 명찰
        ------------------------------------------------------------- */
        function makeWorkerNameTag(name) {
            const canvas = document.createElement('canvas');
            canvas.width = 256;
            canvas.height = 96;
            const ctx = canvas.getContext('2d');

            drawRoundedRect(ctx, 6, 6, 244, 84, 16, "#0f172a", "#38bdf8", 6);

            ctx.fillStyle = "#ffffff";
            ctx.font = "900 44px 'Malgun Gothic'";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(name, 128, 48);

            const texture = new THREE.CanvasTexture(canvas);
            const spriteMat = new THREE.SpriteMaterial({ map: texture, depthTest: false });
            const sprite = new THREE.Sprite(spriteMat);
            sprite.scale.set(4.5, 1.7, 1);
            sprite.position.y = 4.2;
            return sprite;
        }

        function createHumanCharacter(name, roleIndex) {
            const humanGroup = new THREE.Group();

            const skinMat = new THREE.MeshStandardMaterial({ color: 0xffedd5, roughness: 0.5 });
            const clothMat = new THREE.MeshStandardMaterial({ color: 0x1e3a8a, roughness: 0.6 });
            const vestMat = new THREE.MeshStandardMaterial({ color: 0x84cc16, roughness: 0.4 });
            const hatColor = (roleIndex === 0) ? 0xffffff : (roleIndex % 2 === 0 ? 0xfacc15 : 0x0284c7);
            const hatMat = new THREE.MeshStandardMaterial({ color: hatColor, roughness: 0.3 });

            const body = new THREE.Mesh(new THREE.BoxGeometry(0.9, 1.3, 0.6), vestMat);
            body.position.y = 1.95;
            body.castShadow = true;
            humanGroup.add(body);

            const head = new THREE.Mesh(new THREE.SphereGeometry(0.32, 16, 16), skinMat);
            head.position.y = 2.85;
            humanGroup.add(head);

            const hat = new THREE.Mesh(new THREE.CylinderGeometry(0.38, 0.42, 0.3, 16), hatMat);
            hat.position.y = 3.1;
            humanGroup.add(hat);
            const hatBrim = new THREE.Mesh(new THREE.CylinderGeometry(0.5, 0.5, 0.06, 16), hatMat);
            hatBrim.position.y = 2.98;
            humanGroup.add(hatBrim);

            const leftArm = new THREE.Mesh(new THREE.BoxGeometry(0.26, 1.1, 0.26), clothMat);
            leftArm.position.set(-0.6, 1.95, 0);
            humanGroup.add(leftArm);

            const rightArm = new THREE.Mesh(new THREE.BoxGeometry(0.26, 1.1, 0.26), clothMat);
            rightArm.position.set(0.6, 1.95, 0);
            humanGroup.add(rightArm);

            const leftLeg = new THREE.Mesh(new THREE.BoxGeometry(0.32, 1.3, 0.32), clothMat);
            leftLeg.position.set(-0.25, 0.65, 0);
            leftLeg.castShadow = true;
            humanGroup.add(leftLeg);

            const rightLeg = new THREE.Mesh(new THREE.BoxGeometry(0.32, 1.3, 0.32), clothMat);
            rightLeg.position.set(0.25, 0.65, 0);
            rightLeg.castShadow = true;
            humanGroup.add(rightLeg);

            const nameTag = makeWorkerNameTag(name);
            humanGroup.add(nameTag);

            const hit = new THREE.Mesh(
                new THREE.CylinderGeometry(1.4, 1.4, 4.5, 12),
                new THREE.MeshBasicMaterial({ visible: false })
            );
            hit.position.y = 2.2;
            hit.userData.parentGroup = humanGroup;
            humanGroup.add(hit);
            facilityHitMeshList.push(hit);

            humanGroup.userData = { leftArm, rightArm, leftLeg, rightLeg, head, name, isWorker: true };
            return humanGroup;
        }

        function rebuild3DWorkers(nameList) {
            // ★ (수정) 삭제되는 작업자의 히트박스를 목록에서도 제거 (유령 히트박스 방지)
            facilityHitMeshList = facilityHitMeshList.filter(
                h => !(h.userData.parentGroup && h.userData.parentGroup.userData && h.userData.parentGroup.userData.isWorker)
            );
            workers.forEach(w => {
                safeDispose3DObject(w);
                scene.remove(w);
            });
            workers = [];

            const patterns = [
                { type: "operator", x: -34, y: 18.75, z: -21, rotY: 0 },
                { type: "patrol", x: -15, y: 18.75, z: -7.5, rotY: Math.PI/2, startX: -28, endX: 12, dir: 1 },
                { type: "inspector", x: 4, y: 18.75, z: -21, rotY: -Math.PI/4 },
                { type: "supervisor", x: -40, y: 18.75, z: -19, rotY: Math.PI * 0.6 },
                { type: "patrol_floor", x: -10, y: 0, z: 22, rotY: Math.PI/2, startX: -30, endX: 25, dir: 1 },
                { type: "inspector_floor", x: -20, y: 0, z: 4, rotY: 0 }
            ];

            nameList.forEach((name, idx) => {
                const pat = patterns[idx % patterns.length];
                const worker = createHumanCharacter(name, idx);
                worker.position.set(pat.x, pat.y, pat.z);
                worker.rotation.y = pat.rotY;
                worker.userData.type = pat.type;
                worker.userData.startX = pat.startX;
                worker.userData.endX = pat.endX;
                worker.userData.dir = pat.dir || 1;
                worker.userData.is2F = (pat.y > 10);
                scene.add(worker);
                workers.push(worker);
            });
        }

        /* -------------------------------------------------------------
           9. 3D 래들 모델링 & 전광판 표 (★ 지능형 자동 줄바꿈 장착)
        ------------------------------------------------------------- */
        function createLadleMesh(data) {
            const ladleGroup = new THREE.Group();

            const bodyColor = (data.kind === "티밍래들") ? 0x1d4ed8 : 0xea580c;
            const ladleMat = new THREE.MeshStandardMaterial({
                color: bodyColor,
                metalness: 0.55,
                roughness: 0.35
            });
            ladleMat.name = 'ladleBody';   // ★ (수정) 종류 변경 시 이 재질만 색을 바꾸기 위한 이름표

            const rim = new THREE.Mesh(new THREE.CylinderGeometry(4.8, 4.8, 1.2, 32), ladleMat);
            rim.position.y = 10.5;
            rim.castShadow = true;
            ladleGroup.add(rim);

            const body = new THREE.Mesh(new THREE.CylinderGeometry(4.3, 3.8, 8, 32), ladleMat);
            body.position.y = 6;
            body.castShadow = true;
            ladleGroup.add(body);

            const bottom = new THREE.Mesh(new THREE.SphereGeometry(3.8, 32, 16, 0, Math.PI*2, Math.PI/2, Math.PI/2), ladleMat);
            bottom.position.y = 2;
            bottom.rotation.x = Math.PI;
            ladleGroup.add(bottom);

            const plate = new THREE.Mesh(new THREE.BoxGeometry(5.2, 0.6, 3.2), new THREE.MeshStandardMaterial({ color: 0x334155 }));
            plate.position.y = 1.6;
            ladleGroup.add(plate);

            [-1.2, 1.2].forEach(nx => {
                const nozzle = new THREE.Mesh(new THREE.CylinderGeometry(0.8, 0.8, 1.5, 16), ladleMat);
                nozzle.position.set(nx, 0.8, 0);
                ladleGroup.add(nozzle);
            });

            [-4.7, 4.7].forEach(tx => {
                const trunnion = new THREE.Mesh(new THREE.CylinderGeometry(0.6, 0.6, 1.5, 16), new THREE.MeshStandardMaterial({ color: 0x1e293b, metalness: 0.8 }));
                trunnion.rotation.z = Math.PI / 2;
                trunnion.position.set(tx, 7.5, 0);
                ladleGroup.add(trunnion);
            });

            const bodyBadge = makeBodyNumberBadge(data.kind, data.name);
            bodyBadge.position.set(0, 6, 4.4);
            bodyBadge.name = "bodyBadge";
            ladleGroup.add(bodyBadge);

            const sprite = makeUnifiedLadleBoard(data);
            sprite.position.y = 23.5;
            sprite.name = "ladleSprite";
            ladleGroup.add(sprite);

            const hitBox = new THREE.Mesh(
                new THREE.CylinderGeometry(6, 6, 14, 16),
                new THREE.MeshBasicMaterial({ visible: false })
            );
            hitBox.position.y = 7;
            hitBox.userData = { parentGroup: ladleGroup, isLadle: true };
            ladleGroup.add(hitBox);
            hitMeshList.push(hitBox);

            ladleGroup.position.set(data.x, 0, data.z);
            ladleGroup.userData = data;
            scene.add(ladleGroup);
            ladleObjects.push(ladleGroup);
            return ladleGroup;
        }

        function drawRoundedRect(ctx, x, y, width, height, radius, fillColor, strokeColor, strokeWidth) {
            ctx.beginPath();
            ctx.moveTo(x + radius, y);
            ctx.lineTo(x + width - radius, y);
            ctx.quadraticCurveTo(x + width, y, x + width, y + radius);
            ctx.lineTo(x + width, y + height - radius);
            ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
            ctx.lineTo(x + radius, y + height);
            ctx.quadraticCurveTo(x, y + height, x, y + height - radius);
            ctx.lineTo(x, y + radius);
            ctx.quadraticCurveTo(x, y, x + radius, y);
            ctx.closePath();

            if (fillColor) { ctx.fillStyle = fillColor; ctx.fill(); }
            if (strokeColor && strokeWidth) { ctx.strokeStyle = strokeColor; ctx.lineWidth = strokeWidth; ctx.stroke(); }
        }

        function makeBodyNumberBadge(kind, rawNum) {
            const canvas = document.createElement('canvas');
            canvas.width = 512;
            canvas.height = 256;
            const ctx = canvas.getContext('2d');

            drawRoundedRect(ctx, 12, 12, 488, 232, 28, "#ffffff", "#000000", 12);

            const shortKind = kind.includes("티밍") ? "티밍" : "장입";
            const numOnly = String(rawNum).replace(/[^0-9]/g, '') || "1";

            ctx.fillStyle = (shortKind === "티밍") ? "#1d4ed8" : "#ea580c";
            ctx.font = "bold 62px 'Malgun Gothic', sans-serif";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(shortKind, 256, 75);

            ctx.fillStyle = "#000000";
            ctx.font = "900 115px 'Malgun Gothic', sans-serif";
            ctx.fillText(`${numOnly}호`, 256, 170);

            const texture = new THREE.CanvasTexture(canvas);
            const mat = new THREE.MeshBasicMaterial({ map: texture, transparent: true });
            return new THREE.Mesh(new THREE.PlaneGeometry(5.2, 2.6), mat);
        }

        /* ★ [캔버스 자동 줄바꿈 헬퍼 함수] 긴 글이나 엔터도 완벽하게 박스 안에 표시 */
        function wrapCanvasText(ctx, text, x, y, maxWidth, lineHeight, maxLines) {
            if (!text) return;
            const paragraphs = text.split('\n');
            let currentLine = 0;

            for (let p = 0; p < paragraphs.length; p++) {
                const words = paragraphs[p];
                let line = '';

                for (let n = 0; n < words.length; n++) {
                    const testLine = line + words[n];
                    const metrics = ctx.measureText(testLine);
                    const testWidth = metrics.width;
                    if (testWidth > maxWidth && n > 0) {
                        ctx.fillText(line, x, y + (currentLine * lineHeight));
                        line = words[n];
                        currentLine++;
                        if (currentLine >= maxLines) return;
                    } else {
                        line = testLine;
                    }
                }
                ctx.fillText(line, x, y + (currentLine * lineHeight));
                currentLine++;
                if (currentLine >= maxLines) return;
            }
        }

        /* ★ VRAM 크래시 없는 최적 해상도(720x1200) 및 자동 줄바꿈 텍스처 */
        function makeUnifiedLadleBoard(data) {
            const canvas = document.createElement('canvas');
            canvas.width = 720;
            canvas.height = 1200;
            const ctx = canvas.getContext('2d');

            // 1. 전체 보드 외곽 테두리
            drawRoundedRect(ctx, 14, 14, 692, 1172, 36, "#ffffff", "#0f172a", 16);

            // 2. 상단 헤더
            const headerColor = (data.kind === "티밍래들") ? "#1e3a8a" : "#9a3412";
            drawRoundedRect(ctx, 14, 14, 692, 216, 36, headerColor, null, 0);
            ctx.fillStyle = headerColor;
            ctx.fillRect(14, 120, 692, 110);

            const labelText = getVesselLabel(data.kind, data.name);
            ctx.fillStyle = "#ffffff";
            ctx.font = "900 88px 'Malgun Gothic', sans-serif";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(labelText, 360, 120);

            // 3. 작업 상태 대형 알림 블록
            let statusBg = "#0284c7";
            if (data.status === '예열중') statusBg = "#ea580c";
            else if (data.status === '해체중') statusBg = "#dc2626";
            else if (data.status === '완료') statusBg = "#16a34a";
            else if (data.status === '대기') statusBg = "#475569";

            drawRoundedRect(ctx, 36, 252, 648, 150, 28, statusBg, "#ffffff", 8);
            ctx.fillStyle = "#ffffff";
            ctx.font = "900 78px 'Malgun Gothic', sans-serif";
            ctx.textAlign = "center";
            ctx.textBaseline = "middle";
            ctx.fillText(`● ${data.status}`, 360, 327);

            // 4. 수리 구분 안내 영역
            drawRoundedRect(ctx, 36, 426, 648, 186, 22, "#f1f5f9", "#cbd5e1", 5);

            ctx.fillStyle = "#475569";
            ctx.font = "bold 52px 'Malgun Gothic', sans-serif";
            ctx.textAlign = "center";
            ctx.fillText("🔧 수리 구분", 360, 471);

            ctx.fillStyle = "#0284c7";
            ctx.font = "900 88px 'Malgun Gothic', sans-serif";
            ctx.fillText(data.repair, 360, 555);

            // 5. 작업 내용 및 비고 블록 (★ 지능형 자동 줄바꿈)
            drawRoundedRect(ctx, 36, 636, 648, 522, 28, "#fef08a", "#ca8a04", 10);

            ctx.fillStyle = "#854d0e";
            ctx.font = "bold 50px 'Malgun Gothic', sans-serif";
            ctx.textAlign = "left";
            ctx.textBaseline = "top";
            ctx.fillText("📝 작업 내용 / 비고:", 60, 666);

            if (data.note && data.note.trim() !== "") {
                ctx.fillStyle = "#451a03";
                ctx.font = "900 52px 'Malgun Gothic', sans-serif";
                wrapCanvasText(ctx, data.note.trim(), 60, 746, 560, 68, 6);
            }

            const texture = new THREE.CanvasTexture(canvas);
            texture.minFilter = THREE.LinearFilter;
            texture.magFilter = THREE.LinearFilter;

            const spriteMat = new THREE.SpriteMaterial({ map: texture });
            const sprite = new THREE.Sprite(spriteMat);
            sprite.scale.set(15, 23, 1);
            return sprite;
        }

        /* -------------------------------------------------------------
           10. 마우스 인터랙션 & Undo 연동
        ------------------------------------------------------------- */
        function setupDragAndSelect() {
            const dom = renderer.domElement;

            dom.addEventListener('contextmenu', (e) => {
                e.preventDefault();
                const rect = dom.getBoundingClientRect();
                mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
                mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;

                raycaster.setFromCamera(mouse, camera);
                const intersects = raycaster.intersectObjects(hitMeshList, false);

                if (intersects.length > 0) {
                    closeFloatingCard();   // 이미 열린 카드가 있으면 먼저 정리
                    selectedLadle = intersects[0].object.userData.parentGroup;
                    openFloatingCard(e.clientX, e.clientY, selectedLadle.userData);
                } else {
                    closeFloatingCard();
                }
            });

            // ★ (수정) 캡처 단계에서 처리 → OrbitControls 보다 먼저 컨트롤을 끌 수 있음
            //          (래들/설비 위에서 클릭할 때 화면이 같이 끌려가는 현상 방지, Ctrl 회전 꼬임 제거)
            container.addEventListener('pointerdown', (e) => {
                if (e.target !== dom || e.button !== 0) return;
                closeFloatingCard();

                const rect = dom.getBoundingClientRect();
                mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
                mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;

                raycaster.setFromCamera(mouse, camera);

                let candidateList = isLayoutEditMode ? [...hitMeshList, ...facilityHitMeshList] : hitMeshList;
                const intersects = raycaster.intersectObjects(candidateList, false);

                if (intersects.length > 0) {
                    const hitObj = intersects[0].object;
                    selectedObject = hitObj.userData.parentGroup;
                    isDragging = true;
                    controls.enabled = false;

                    if (selectedObject.userData.repair !== undefined) {
                        dragStartSnapshot = JSON.stringify(getCurrentVesselsState());
                    }

                    const planeHeight = (selectedObject.userData && selectedObject.userData.is2F) ? 18.75 : 0;
                    dragPlane.set(new THREE.Vector3(0, 1, 0), -planeHeight);

                    if (raycaster.ray.intersectPlane(dragPlane, planeIntersect)) {
                        dragOffset.copy(planeIntersect).sub(selectedObject.position);
                    }
                    dom.style.cursor = 'grabbing';
                }
            }, true);

            dom.addEventListener('pointermove', (e) => {
                const rect = dom.getBoundingClientRect();
                mouse.x = ((e.clientX - rect.left) / rect.width) * 2 - 1;
                mouse.y = -((e.clientY - rect.top) / rect.height) * 2 + 1;

                if (isDragging && selectedObject) {
                    raycaster.setFromCamera(mouse, camera);
                    if (raycaster.ray.intersectPlane(dragPlane, planeIntersect)) {
                        let curX = planeIntersect.x - dragOffset.x;
                        let curZ = planeIntersect.z - dragOffset.z;

                        if (selectedObject.userData.repair !== undefined) {
                            for (let sp of SNAP_POINTS) {
                                if (Math.hypot(curX - sp.x, curZ - sp.z) < 4.8) {
                                    curX = sp.x;
                                    curZ = sp.z;
                                    break;
                                }
                            }
                        }

                        selectedObject.position.x = curX;
                        selectedObject.position.z = curZ;

                        if (selectedObject.userData.repair !== undefined) {
                            selectedObject.userData.x = curX;
                            selectedObject.userData.z = curZ;
                        } else if (selectedObject.userData.facilityKey) {
                            facilityPositions[selectedObject.userData.facilityKey].x = curX;
                            facilityPositions[selectedObject.userData.facilityKey].z = curZ;
                        }
                    }
                } else {
                    raycaster.setFromCamera(mouse, camera);
                    let candidateList = isLayoutEditMode ? [...hitMeshList, ...facilityHitMeshList] : hitMeshList;
                    const intersects = raycaster.intersectObjects(candidateList, false);
                    dom.style.cursor = intersects.length > 0 ? 'grab' : 'default';
                }
            });

            // ★ (수정) 캔버스 밖에서 마우스를 놓아도 드래그가 해제되도록 window 에서 처리
            function endDrag() {
                if (!isDragging) return;
                isDragging = false;
                controls.enabled = true;
                dom.style.cursor = 'default';

                if (selectedObject && selectedObject.userData.repair !== undefined) {
                    for (let sp of SNAP_POINTS) {
                        if (Math.hypot(selectedObject.position.x - sp.x, selectedObject.position.z - sp.z) < 5.2) {
                            selectedObject.position.x = sp.x;
                            selectedObject.position.z = sp.z;
                            selectedObject.userData.x = sp.x;
                            selectedObject.userData.z = sp.z;
                            break;
                        }
                    }

                    const currentSnapshot = JSON.stringify(getCurrentVesselsState());
                    if (dragStartSnapshot && dragStartSnapshot !== currentSnapshot) {
                        undoStack.push(dragStartSnapshot);
                        if (undoStack.length > MAX_HISTORY) undoStack.shift();
                        redoStack = [];
                        updateHistoryButtons();
                    }
                }

                triggerAutoSave();
                selectedObject = null;
                dragStartSnapshot = null;
            }
            window.addEventListener('pointerup', endDrag);
            window.addEventListener('pointercancel', endDrag);

            // ★ 텍스트 필드 포커스 시 서버 덮어쓰기 방지 플래그 연동
            ['fc-name', 'fc-note', 'db-notice'].forEach(id => {
                const el = document.getElementById(id);
                if (el) {
                    el.addEventListener('focus', () => { isEditingText = true; });
                    el.addEventListener('blur', () => { 
                        isEditingText = false; 
                        triggerAutoSave();
                    });
                }
            });
        }

        /* -------------------------------------------------------------
           11. 래들 우클릭 플로팅 카드
        ------------------------------------------------------------- */
        function fillFloatingCardFields(data) {
            document.getElementById('fc-title').innerText = `📝 ${getVesselLabel(data.kind, data.name)} 수정`;
            document.getElementById('fc-kind').value = data.kind;
            const numOnly = String(data.name).replace(/[^0-9]/g, '') || "1";
            document.getElementById('fc-name').value = numOnly;

            const repSelect = document.getElementById('fc-repair');
            populateRepairSelect(repSelect, data.kind, data.repair);

            document.getElementById('fc-status').value = data.status;
            document.getElementById('fc-note').value = data.note || "";
        }

        function openFloatingCard(clientX, clientY, data) {
            fillFloatingCardFields(data);
            cardSnapshot = JSON.stringify(getCurrentVesselsState());

            floatingCard.style.display = 'block';
            const cardWidth = floatingCard.offsetWidth || 330;
            const cardHeight = floatingCard.offsetHeight || 500;

            let posX = clientX + 15;
            let posY = clientY - 30;

            const rightLimit = (isSidebarCollapsed ? window.innerWidth : (window.innerWidth - 380)) - 15;
            if (posX + cardWidth > rightLimit) {
                posX = clientX - cardWidth - 15;
            }
            if (posX < 15) posX = 15;

            if (posY + cardHeight > window.innerHeight - 20) {
                posY = window.innerHeight - cardHeight - 20;
            }
            if (posY < 65) posY = 65;

            floatingCard.style.left = `${posX}px`;
            floatingCard.style.top = `${posY}px`;
        }

        // 저장/이력 없이 카드만 조용히 숨김
        function hideFloatingCardSilently() {
            floatingCard.style.display = 'none';
            isEditingText = false;
            cardSnapshot = null;
        }

        // ★ (수정) 클릭할 때마다 저장하던 triggerAutoSave 제거.
        //          대신 카드에서 실제로 바뀐 경우에만 되돌리기 이력에 추가
        function closeFloatingCard() {
            const wasOpen = (floatingCard.style.display === 'block');
            floatingCard.style.display = 'none';
            isEditingText = false;

            if (wasOpen && cardSnapshot) {
                const cur = JSON.stringify(getCurrentVesselsState());
                if (cur !== cardSnapshot) {
                    undoStack.push(cardSnapshot);
                    if (undoStack.length > MAX_HISTORY) undoStack.shift();
                    redoStack = [];
                    updateHistoryButtons();
                }
            }
            cardSnapshot = null;
        }

        // ★ 입력 즉시 3D 래들 전광판 화면에 바로바로 반영
        function onFloatingCardEdit() {
            if (!selectedLadle || !selectedLadle.parent) return;

            const kind = document.getElementById('fc-kind').value;
            const rawName = document.getElementById('fc-name').value.trim();
            let n = parseInt(rawName, 10);
            if (isNaN(n) || n < 1) n = 1;            // ★ (수정) 0/음수/빈 값은 1호로 보정
            const name = String(n);
            const repair = document.getElementById('fc-repair').value;
            const status = document.getElementById('fc-status').value;
            const note = document.getElementById('fc-note').value;

            selectedLadle.userData.kind = kind;
            selectedLadle.userData.name = name;
            selectedLadle.userData.repair = repair;
            selectedLadle.userData.status = status;
            selectedLadle.userData.note = note;

            document.getElementById('fc-title').innerText = `📝 ${getVesselLabel(kind, name)} 수정`;

            // ★ (수정) 래들 본체 재질만 색 변경 (트러니언 등 다른 부품 색 유지)
            const bodyColor = (kind === "티밍래들") ? 0x1d4ed8 : 0xea580c;
            selectedLadle.children.forEach(child => {
                if (child.material && child.material.name === 'ladleBody') {
                    child.material.color.setHex(bodyColor);
                }
            });

            const oldBadge = selectedLadle.getObjectByName("bodyBadge");
            if (oldBadge) {
                safeDispose3DObject(oldBadge);
                selectedLadle.remove(oldBadge);
            }
            const newBadge = makeBodyNumberBadge(kind, name);
            newBadge.position.set(0, 6, 4.4);
            newBadge.name = "bodyBadge";
            selectedLadle.add(newBadge);

            const oldSprite = selectedLadle.getObjectByName("ladleSprite");
            if (oldSprite) {
                safeDispose3DObject(oldSprite);
                selectedLadle.remove(oldSprite);
            }
            const newSprite = makeUnifiedLadleBoard(selectedLadle.userData);
            newSprite.position.y = 23.5;
            newSprite.name = "ladleSprite";
            selectedLadle.add(newSprite);

            triggerAutoSave();
        }

        function deleteCurrentLadle() {
            if (!selectedLadle) return;
            const vesselName = getVesselLabel(selectedLadle.userData.kind, selectedLadle.userData.name);
            if (!confirm(`정말 [${vesselName}]을(를) 삭제하시겠습니까?\n(삭제 후에도 상단 '되돌리기' 버튼으로 복구 가능합니다)`)) {
                return;
            }

            // 카드에서 수정했던 내용까지 포함해 삭제 직전 상태를 이력에 저장
            pushHistoryState();
            hitMeshList = hitMeshList.filter(mesh => mesh.userData.parentGroup !== selectedLadle);
            safeDispose3DObject(selectedLadle);
            scene.remove(selectedLadle);
            ladleObjects = ladleObjects.filter(item => item !== selectedLadle);
            selectedLadle = null;
            hideFloatingCardSilently();   // 이력 중복 방지
            triggerAutoSave();
        }

        function onNoticeEdit() {
            const val = document.getElementById('db-notice').value;
            updateLedTicker(val);
            triggerAutoSave();
        }

        /* -------------------------------------------------------------
           12. 신규 래들 등록 & 중복 방지
        ------------------------------------------------------------- */
        function openShiftSettingModal() {
            document.getElementById('cfg-base-date').value = shiftSettings.base_date;
            document.getElementById('cfg-group-a').value = shiftSettings.group_a;
            document.getElementById('cfg-group-b').value = shiftSettings.group_b;
            settingModal.style.display = 'flex';
        }

        function closeShiftSettingModal() { settingModal.style.display = 'none'; }

        function saveShiftSettings() {
            shiftSettings.base_date = document.getElementById('cfg-base-date').value || localDateStr();
            shiftSettings.group_a = document.getElementById('cfg-group-a').value.trim();
            shiftSettings.group_b = document.getElementById('cfg-group-b').value.trim();
            closeShiftSettingModal();
            lastWorkerSig = null;   // 설정을 저장하면 3D 작업자도 새로 배치
            calculateTodayShift();
            triggerAutoSave();
        }

        function openAddModal() {
            let maxNo = 0;
            ladleObjects.forEach(obj => {
                const n = parseInt(obj.userData.name, 10);
                if (!isNaN(n) && n > maxNo) maxNo = n;
            });
            const nextNo = maxNo + 1;

            document.querySelector(`input[name="m-kind"][value="티밍래들"]`).checked = true;
            document.getElementById('m-vessel-no').value = nextNo;
            populateRepairSelect(document.getElementById('m-repair-type'), "티밍래들", "1차 소수리");
            document.getElementById('m-status').value = "축조중";
            document.getElementById('m-note').value = "";
            modal.style.display = 'flex';
        }

        function closeModal() { modal.style.display = 'none'; }

        function submitAddLadle() {
            const kind = document.querySelector('input[name="m-kind"]:checked').value;
            const rawNo = document.getElementById('m-vessel-no').value.trim();
            let n = parseInt(rawNo, 10);
            if (isNaN(n) || n < 1) n = 1;
            const name = String(n);

            const isDuplicate = ladleObjects.some(obj => {
                return obj.userData.kind === kind && String(obj.userData.name) === String(name);
            });

            if (isDuplicate) {
                alert(`⚠️ 이미 등록되어 있는 베셀 호수입니다!\n[${kind} ${name}호]\n중복되지 않는 다른 호수를 입력해주세요.`);
                return;
            }

            pushHistoryState();

            // ★ (수정) 비어 있는 스냅 자리에 배치 (기존 래들과 겹치지 않게)
            const pos = findFreeSnap();

            createLadleMesh({
                kind: kind,
                name: name,
                repair: document.getElementById('m-repair-type').value,
                status: document.getElementById('m-status').value,
                note: document.getElementById('m-note').value,
                x: pos.x,
                z: pos.z
            });

            closeModal();
            triggerAutoSave();
        }

        /* -------------------------------------------------------------
           13. 데이터 적용 & 초기 로드
           (수정) 원격 데이터가 현재 화면과 같으면 아무것도 재생성하지 않음
        ------------------------------------------------------------- */
        function applyLoadedData(data, fromRemote) {
            if (!data) return;
            // 드래그 중에는 원격 데이터로 화면을 건드리지 않음
            if (fromRemote && isDragging) return;

            // 1) 카메라 시점 (PC별 로컬 값 : 원격 데이터로는 바꾸지 않음)
            if (data.camera_view && !fromRemote) {
                defaultCameraView = data.camera_view;
                resetCamera();
            }

            // 2) 교대/작업자 설정
            if (data.shift_settings) {
                shiftSettings = Object.assign({}, shiftSettings, data.shift_settings);
            }
            calculateTodayShift();

            // 3) 공지사항 (입력 중이면 덮어쓰지 않음)
            if (data.dashboard && typeof data.dashboard.notice === 'string') {
                const noticeEl = document.getElementById('db-notice');
                if (!isEditingText || !fromRemote) noticeEl.value = data.dashboard.notice;
            }
            updateLedTicker(document.getElementById('db-notice').value);

            // 4) 시설 위치
            if (data.facilities) {
                const map = { booth: boothMesh, mortar: mortarMesh, vortex: vortexMesh, ct: ctMesh, excavator: excavatorGroup };
                Object.keys(map).forEach(key => {
                    const f = data.facilities[key];
                    if (f && isFinite(f.x) && isFinite(f.z) && map[key]) {
                        facilityPositions[key] = { x: Number(f.x), z: Number(f.z) };
                        map[key].position.x = Number(f.x);
                        map[key].position.z = Number(f.z);
                    }
                });
            }

            // 5) 래들 : 실제로 달라졌을 때만 다시 만듦
            if (data.vessels !== undefined || fromRemote) {
                const list = normalizeVessels(data.vessels);
                if (JSON.stringify(list) !== JSON.stringify(getCurrentVesselsState())) {
                    const cardOpen = (floatingCard.style.display === 'block');
                    const openKey = (selectedLadle && selectedLadle.userData)
                        ? selectedLadle.userData.kind + '|' + selectedLadle.userData.name : null;

                    restoreVesselsState(list);

                    // 카드가 열려 있었다면 같은 래들을 찾아 다시 연결
                    if (cardOpen && openKey) {
                        const matched = ladleObjects.find(o => (o.userData.kind + '|' + o.userData.name) === openKey);
                        if (matched) {
                            selectedLadle = matched;
                            floatingCard.style.display = 'block';
                            fillFloatingCardFields(matched.userData);
                            cardSnapshot = JSON.stringify(getCurrentVesselsState());
                        }
                    }
                }
            }
        }

        (function initLoad() {
            const local = loadLocalData();
            applyLoadedData(local);
            undoStack = [];
            redoStack = [];
            updateHistoryButtons();
            initFirebaseConnection();
        })();

        /* -------------------------------------------------------------
           14. 애니메이션 루프
           (수정) 프레임 속도와 무관하게 동일한 속도로 움직이도록 delta time 사용
        ------------------------------------------------------------- */
        function animate() {
            requestAnimationFrame(animate);

            const dt = Math.min(clock.getDelta(), 0.1);
            animTime += dt;
            const time = animTime;

            controls.update();

            // 굴착기 (브레이커 타격 / 캐빈 회전)
            if (excCabin && excBreaker && excavatorGroup) {
                excCabin.rotation.y = Math.sin(time * 0.6) * 0.3;
                excBreaker.position.y = 3.2 + Math.sin(time * 14) * 0.12;
                if (excBoom) excBoom.rotation.x = Math.sin(time * 0.9) * 0.05;
            }

            // 지게차 왕복
            if (forkliftGroup && selectedObject !== forkliftGroup) {
                const u = forkliftGroup.userData;
                forkliftGroup.position.x += u.dir * u.speed * dt;
                if (forkliftGroup.position.x >= u.maxX) {
                    forkliftGroup.position.x = u.maxX;
                    u.dir = -1;
                    forkliftGroup.rotation.y = -Math.PI / 2;
                } else if (forkliftGroup.position.x <= u.minX) {
                    forkliftGroup.position.x = u.minX;
                    u.dir = 1;
                    forkliftGroup.rotation.y = Math.PI / 2;
                }
            }

            // 작업자
            workers.forEach((w, i) => {
                const u = w.userData;
                const isPatrol = (u.type === 'patrol' || u.type === 'patrol_floor');

                if (isPatrol && !isLayoutEditMode && selectedObject !== w &&
                    u.startX !== undefined && u.endX !== undefined) {
                    w.position.x += u.dir * 2.7 * dt;
                    if (w.position.x >= u.endX) { w.position.x = u.endX; u.dir = -1; }
                    else if (w.position.x <= u.startX) { w.position.x = u.startX; u.dir = 1; }
                    w.rotation.y = (u.dir > 0) ? Math.PI / 2 : -Math.PI / 2;

                    const swing = Math.sin(time * 6 + i) * 0.6;
                    u.leftLeg.rotation.x = swing;
                    u.rightLeg.rotation.x = -swing;
                    u.leftArm.rotation.x = -swing * 0.8;
                    u.rightArm.rotation.x = swing * 0.8;
                } else {
                    // 제자리 작업 자세 (살짝 움직임)
                    const idle = Math.sin(time * 2 + i) * 0.12;
                    u.leftLeg.rotation.x = 0;
                    u.rightLeg.rotation.x = 0;
                    u.leftArm.rotation.x = idle;
                    u.rightArm.rotation.x = -idle;
                }
            });

            renderer.render(scene, camera);
        }
    </script>
</body>
</html>

