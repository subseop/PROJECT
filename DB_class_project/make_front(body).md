# 프론트 구축하기

## 세부설명- body html 부분까지 

### 3. <body> - 실제 화면
    3-1. 상단 헤더
    ```
    <header class="top">
    <div class="top-inner">
        <!-- 왼쪽: 로고 + 서비스 이름 -->
        <div class="brand">
        <div class="logo">🛒</div>
        소상공인 판매관리
        </div>

        <!-- 오른쪽: 매장 선택 + 새로고침 -->
        <div class="actions">
        <label>매장</label>
        <!-- 매장 필터 (JS에서 옵션 채움) -->
        <select class="input" id="storeFilter">
            <option value="">전체</option>
        </select>
        <!-- 전체 데이터 재로드 -->
        <button class="btn primary" id="refreshBtn">새로고침</button>
        </div>
    </div>
    </header>
    ```
    - 왼쪽: 로고 + 서비스 이름
    - 오른쪽: 매장 선택 드롭다운 + 새로고침 버튼
    - id="storeFilter" → JS에서 선택된 매장 읽을 때 사용
    - id="refreshBtn" → 클릭 시 새로 API 불러오는 버튼

    3-2. 메인 컨텐츠 <main class="container">
    - (1) 매장 대시보드 섹션
    ```
    <main class="container">
    
    <!-- 1. 매장 대시보드 (요약 + 차트) -->
    <section class="dash-section">
        <h2 class="dash-title">📊 매장 대시보드</h2>
        <p class="dash-sub">상단 매장 / 월 / 검색 필터에 맞는 데이터를 요약해서 보여줍니다.</p>

        <!-- 상단 요약 카드: 총 매출, 총 수량, 평균 단가, 건수 -->
        <div class="dash-summary">
        <div class="dash-card">
            <p class="dash-label">총 매출액</p>
            <p class="dash-value" id="dashTotal">-</p>
        </div>
        <div class="dash-card">
            <p class="dash-label">총 수량</p>
            <p class="dash-value" id="dashQty">-</p>
        </div>
        <div class="dash-card">
            <p class="dash-label">평균 단가</p>
            <p class="dash-value" id="dashAvgPrice">-</p>
        </div>
        <div class="dash-card">
            <p class="dash-label">행 개수 (건수)</p>
            <p class="dash-value" id="dashCount">-</p>
        </div>
        </div>

        <!-- 2개의 차트: 월별 매출 추이(라인), 품목별 수량(도넛) -->
        <div class="dash-charts">
        <div class="dash-chart-card">
            <h3 class="dash-chart-title">월별 매출 추이</h3>
            <canvas id="lineChart"></canvas>
        </div>
        <div class="dash-chart-card">
            <h3 class="dash-chart-title">품목별 판매 수량</h3>
            <canvas id="donutChart"></canvas>
        </div>
        </div>
    </section>
    ```
    - 나중에 JS에서 updateDashboard()가 호출되면,
    dashTotal, dashQty 등에 숫자를 채워넣고,
    lineChart, donutChart 캔버스에 차트를 그림.

    - (2) 우리 매장만의 독점 상품
    ```
    <section class="dash-section">
        <h2 class="dash-title">💎 우리 매장만의 독점 상품</h2>
        <p class="dash-sub">인근 대형마트에는 없는, 우리 매장만의 차별화된 상품입니다.</p>
        <div id="martRecList" class="new-grid">
        <!-- JS에서 API 호출 결과로 내용을 채움 -->
        <div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">데이터 분석 중...</div>
        </div>
    </section>
    ```
    - 처음에는 “데이터 분석 중…” 이라고 뜸
    - JS에서 loadMartRecommendations()가 martRecList 안에 카드를 채워줌
    - 대형마트에는 없고 우리 매장에만 있는 상품들 목록

    - (3) 다음 달 발주 가이드
    ```
    <section class="dash-section">
        <h2 class="dash-title">📅 다음 달 발주 가이드</h2>
        <p class="dash-sub">과거 데이터를 분석하여 발주량을 늘려야 할 상품을 추천합니다.</p>
        <div id="orderGuideList" class="new-grid">
        <!-- JS에서 예측 API 호출 결과로 내용을 채움 -->
        <div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">데이터 분석 중...</div>
        </div>
    </section>
    ```
    - loadOrderGuide() 가 orderGuideList 안에
    - 발주 늘릴 상품(🔥 발주 증량)
    - 유지/감소 상품, 카드를 채워줌

    - (4) CRUD 그리드 (좌: 테이블, 우:폼) 
    ```
    <div class="crud-grid">
        <!-- 좌측: 판매 데이터 목록 테이블 -->
        <div class="card">
        <!-- 조회 조건 툴바 -->
        <div class="toolbar">
            <!-- 년-월 필터 -->
            <input type="month" id="qDate" class="input">
            <!-- 품목명 검색 필터 -->
            <input type="text" id="qKeyword" class="input" placeholder="품목 검색">
            <!-- 필터 적용 버튼 -->
            <button class="btn" id="btnSelect">조회</button>
            <!-- 필터 초기화 버튼 -->
            <button class="btn ghost" id="btnReset">초기화</button>
        </div>

        <!-- 판매 데이터 테이블 -->
        <div class="table-wrap">
            <table id="salesTable">
            <thead>
                <tr>
                <th>날짜</th>
                <th>품목</th>
                <th>수량</th>
                <th>단가</th>
                <th>매출액</th>
                <th>관리</th>
                </tr>
            </thead>
            <tbody></tbody>
            </table>
        </div>
        </div>

        <!-- 우측: 신규 등록 / 수정 폼 -->
        <div class="card">
        <h3 id="formTitle">신규 등록</h3>
        <div class="form-group">
            <label>날짜 (년-월 기준)</label>
            <input id="fDate" type="month" class="input">
        </div>
        <div class="form-group">
            <label>상품 선택</label>
            <!-- 상품 리스트는 JS에서 API 호출로 채움 -->
            <select id="fProduct" class="input"></select>
        </div>
        <!-- 수량/단가를 좌우 2열 레이아웃으로 배치 -->
        <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px">
            <div class="form-group">
            <label>수량</label>
            <input id="fQty" type="number" class="input" min="0">
            </div>
            <div class="form-group">
            <label>단가(원)</label>
            <input id="fPrice" type="number" class="input" min="0">
            </div>
        </div>
        <!-- 저장 / 취소 버튼 -->
        <div class="form-actions">
            <button class="btn primary" id="btnSave">저장</button>
            <button class="btn ghost" id="btnCancel">취소</button>
        </div>
        <!-- 현재 모드 안내 (INSERT / UPDATE) -->
        <p class="helper" id="formHint">모드: INSERT</p>
        </div>
    </div>
    
    </main>
    ```
    - 왼쪽
     - qDate : 월 필터
     - qKeyword : 품목 이름 검색
     - btnSelect : 조회 버튼 (필터 적용)
     - btnReset : 검색 조건 초기화
     - salesTable tbody : JS에서 데이터 채움

    - 오른쪽
     - fDate, fProduct, fQty, fPrice 입력
     - btnSave 누르면 INSERT 또는 UPDATE
     - btnCancel 누르면 폼 초기화 + 모드 리셋