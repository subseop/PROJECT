# 프론트 구축하기

## 세부설명- SCRIPT 부분 
코드를 크게 나누면

1. 전역 변수 & 공용 함수
2. “독점 상품” / “발주 가이드” 가져오는 부분 (API 통신)
3. 테이블/대시보드/차트 그리는 부분
4. INSERT / UPDATE / DELETE 하는 부분
5. 페이지 처음 로드될 때 동작


### 4. <script> – 자바스크립트 로직
    4-1. 전역 상태 & 유틸 함수
    ```
    let SALES = [];
    let SALES_VIEW = [];
    let PRODUCTS = [];
    let editId = null;

    let lineChart = null;
    let donutChart = null;

    // 숫자 포맷 함수 (천단위 콤마)
    const fmt = n => new Intl.NumberFormat('ko-KR').format(Number(n) || 0);
    ```
    - let SALES = [] : 판매 데이터를 저장하는 배열 / slaes_api.php에서 받아온 JSON들이 여기 들어감
    - let SALES_VIEW = [] : 현재 필터(매장/날짜/검색어)에 맞게 걸러진 데이터 / 화면에 실제로 보여줄 것만 이 배열에 담김
    - let PRODUCTS = [] : 상품 목록 배열(상품 선택 드롭다운에 쓰임)
    - let editId = null; : 지금 폼이 '수정모드'인지 '신규등록모드'인지 구분하는 용도
        - editId = null -> 새로 INSERT
        - editId = 5 -> id가 5인 행을 UPDATE
    - lineChart, donutChart : Chart.js 차트 객체를 저장해두는 변수(나중에 다시 그릴 때 기존 차트 destory()하려고)
    - fmt 함수: 
        - n을 숫자로 바꾸고(Number(n))
        - Intl.NumberFormat('ko-KR')로 "1,234" 이런 식의 천단위 콤마를 찍어주는 포맷터
        - ex) fmt(123456) -> "123,456"  

    4-2 독점 상품 / 발주 가이드 - API 불러오기
    - 독점 상품 (대형마트 비교)
    - "선택된 매장을 기준으로 PHP API를 호출해서 결과(JSON)를 받아서, 그걸 HTML로 만들어서 화면에 뿌려주는 함수"
    ```
    async function loadMartRecommendations() {
    const storeFilter = document.getElementById('storeFilter');
    const storeName = storeFilter.options[storeFilter.selectedIndex].text;
    const storeVal = storeFilter.value;
    const box = document.getElementById('martRecList');

    if (storeVal === "") {
        box.innerHTML = '<div ...>매장을 선택하면 분석 결과가 나옵니다.</div>';
        return;
    }

    try {
        const res = await fetch(`sales_api.php?action=mart_recommendation&store=${encodeURIComponent(storeName)}`);
        const data = await res.json();
        box.innerHTML = '';

        if (!data.length) {
        box.innerHTML = '<div ...>독점 상품이 없습니다. (대형마트와 모두 겹침)</div>';
        return;
        }

        data.slice(0, 6).forEach(item => {
        box.innerHTML += `
            <div class="new-card" style="border-left:4px solid #7c3aed;">
            <span class="nc-badge theme-purple">${item.category}</span>
            <div class="nc-title">${item.name}</div>
            <div class="nc-desc">대형마트 미취급 품목<br>경쟁력 확보 가능</div>
            </div>`;
        });
    } catch (e) {
        console.error(e);
    }
    }
    ```
    - async function: 비동기 함수 안에서 await를 쓸 수 있음 / "기다렸다가 결과 오면 이어서 실행해" 의미
    - document.getElementById('storeFilter') : HTML에서 <select id="storeFilter"> 요소를 가져옴 -> 거기서 선택된 매장 이름/값을 읽음
    - if (storeVal === ""): "전체"거나 아무 매장도 선택 안했으면 그냥 안내 문구만 보여주고 종료
    - fetch( ... ): sales_api.php?action=mart_recommendation&store=매장이름 으로 HTTP 요청
    - const res = await fetch(...); : 서버 응답을 기다렸다가 받음
    - const data = await res.json(); : 응답을 JSON으로 파싱 -> 배열/객체로 변환
    - box.innerHTML = '' : 기존에 써있던 "데이터 분석 중..." 이런 텍스트 지우고
    - data.slice(0, 6).forEach(item => { ... }): 최대 6개까지만 잘라서 -> 각 상품에 대해 <div class="new-card">...</div> HTML 문자열을 붙임

    - 발주 가이드(다음 달 예측)
    ```
    async function loadOrderGuide() {
    const storeFilter = document.getElementById('storeFilter');
    const storeName = storeFilter.options[storeFilter.selectedIndex].text;
    const storeVal = storeFilter.value;
    const box = document.getElementById('orderGuideList');

    if (storeVal === "") {
        box.innerHTML = '<div ...>매장을 선택하면 발주 가이드가 나옵니다.</div>';
        return;
    }

    try {
        const res = await fetch(`sales_api.php?action=predict&store=${encodeURIComponent(storeName)}`);
        const data = await res.json();
        box.innerHTML = '';

        if (!data.length) {
        box.innerHTML = '<div ...>데이터 부족으로 분석 불가</div>';
        return;
        }

        const upList = data.filter(i => i.trend === 'up');
        const downList = data.filter(i => i.trend !== 'up');
        const list = [...upList, ...downList].slice(0, 6);

        list.forEach(item => {
        const isUp = item.trend === 'up';
        const badgeHtml = isUp 
            ? '<span class="nc-badge theme-blue">🔥 발주 증량</span>' 
            : '<span class="nc-badge" style="background:#f3f4f6; color:#6b7280;">유지/감소</span>';
        
        const borderColor = isUp ? '#2563eb' : '#e5e7eb';
        const bgStyle = isUp ? 'background:#fdfdff;' : '';

        box.innerHTML += `
            <div class="new-card" style="border-top:4px solid ${borderColor}; ${bgStyle}">
            ${badgeHtml}
            <div class="nc-title">${item.name}</div>
            <div class="nc-desc">${isUp ? '매출 상승 예상' : '매출 감소 예상'}</div>
            <div class="nc-val">예상: ₩${fmt(item.predicted_revenue)}</div>
            </div>`;
        });

    } catch (e) {
        console.error(e);
    }
    }
    ```
    - 독점 상품과 비슷한 역할과 형태
    - action=predict: API 호출
    - trend === 'up' 이면 불꽃🔥뱃지, 파란색 강조(발주 늘리기) / 아니면 회색 뱃지 (유지/감소)
    - predicted_revenue 를 fmt()로 포맷해 보여줌  

    4-3. 테이블 / 대시보드 / 차트
    - 테이블 랜더링
    ```
    function renderTable() {
    const tb = document.querySelector('#salesTable tbody');
    tb.innerHTML = '';

    if (!SALES_VIEW.length) {
        tb.innerHTML = '<tr><td colspan="6" ...>데이터가 없습니다.</td></tr>';
        return;
    }
    
    SALES_VIEW.forEach(r => {
        const revenue = r.revenue ? r.revenue : (Number(r.qty) || 0) * (Number(r.price) || 0);
        tb.innerHTML += `
        <tr>
            <td>${r.date}</td>
            <td>
            <div style="font-weight:600;">${r.item}</div>
            <div style="font-size:12px;color:#888">${r.store}</div>
            </td>
            <td>${fmt(r.qty)}</td>
            <td>₩${fmt(r.price)}</td>
            <td>₩${fmt(revenue)}</td>
            <td class="actions-cell">
            <button class="btn ghost" onclick="onEdit(${r.id})">수정</button>
            <button class="btn ghost" onclick="onDel(${r.id})">삭제</button>
            </td>
        </tr>`;
    });
    }
    ```
    - document.querySelector('#salesTable tbody'): <tbody>부분에만 데이터를 그리겠다는 뜻
    - tb.innerHTML = '' : 기존 행들 전체 삭제
    - SALES_VIEW가 비었으면 → “데이터가 없습니다.” 한 줄만 출력
    - 아니면 forEach로 한 행씩 <tr>...</tr> 문자열을 누적
    - onclick="onEdit(${r.id})" -> 버튼을 누르면 전역 함수 onEdit실행 / window.onEdit = ... 부분에 정의돼 있음

    - 대시보드 숫자 + 차트
    ```
    function updateDashboard() {
    const totalEl = document.getElementById('dashTotal');
    const qtyEl = document.getElementById('dashQty');
    const avgEl = document.getElementById('dashAvgPrice');
    const countEl = document.getElementById('dashCount');
    
    if (!totalEl) return;
    
    let totalRevenue = 0, totalQty = 0, totalPriceSum = 0;
    const byMonth = {}, byItem = {};

    SALES_VIEW.forEach(r => {
        const qty = Number(r.qty) || 0;
        const price = Number(r.price) || 0;
        const rev = r.revenue ? Number(r.revenue) : qty * price;

        totalRevenue += rev;
        totalQty += qty;
        totalPriceSum += price;

        // 월별 매출
        byMonth[(r.date || '').slice(0, 7)] =
        (byMonth[(r.date || '').slice(0, 7)] || 0) + rev;

        // 품목별 수량
        byItem[r.item || '기타'] =
        (byItem[r.item || '기타'] || 0) + qty;
    });

    // 요약 숫자 표시
    totalEl.innerText = '₩' + fmt(totalRevenue);
    qtyEl.innerText = fmt(totalQty) + ' 개';
    avgEl.innerText = '₩' + fmt(SALES_VIEW.length ? Math.round(totalPriceSum / SALES_VIEW.length) : 0);
    countEl.innerText = fmt(SALES_VIEW.length) + ' 건';
    ```
    - SALES_VIEW 기준으로 
        - 총 매출액(totalRevenue)
        - 총 수량(totalQty)
        - 평균 단가(전체 가격 합 / 행 수)
        - 행 개수
        계산하고, 
    - byMonth : { '2024-01': 매출합, '2024-02': 매출합, ... }
    - byItem : { '바나나': 수량합, '우유': 수량합, ... }

    - 그 다음 차트 그리기:
    ```
    // 라인 차트 (월별 매출)
    const monthLabels = Object.keys(byMonth).sort();
    if (lineChart) lineChart.destroy();
    lineChart = new Chart(document.getElementById('lineChart'), {
        type: 'line',
        data: {
        labels: monthLabels,
        datasets: [{
            label: '매출',
            data: monthLabels.map(k => byMonth[k]),
            borderColor: '#2563eb',
            tension: 0.25
        }]
        },
        options: {
        responsive: true,
        maintainAspectRatio: false,
        scales: { y: { beginAtZero: true } }
        }
    });

    // 도넛 차트 (품목별 수량)
    const itemEntries = Object.entries(byItem).sort((a, b) => b[1] - a[1]).slice(0, 10);
    if (donutChart) donutChart.destroy();
    donutChart = new Chart(document.getElementById('donutChart'), {
        type: 'doughnut',
        data: {
        labels: itemEntries.map(e => e[0]),
        datasets: [{
            data: itemEntries.map(e => e[1]),
            backgroundColor: ['#3b82f6', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6']
        }]
        },
        options: {
        responsive: true,
        maintainAspectRatio: false,
        cutout: '60%'
        }
    });
    }
    ```
    - new Chart(캔버스DOM, {옵션}) - Chart.js 사용법
    - 라인 차트: 
        - x축: 월(monthLabels)
        - y축: 해당 월 매출(byMonth[k])
    - 도넛 차트:
        - 라벨: 품목 이름
        - 값: 판매 수량
    - 매번 새 필터 적용할 때마다 -> 기존 lineChart, donutChart 있으면 destroy() 하고 새로 만듦

    - 필터 적용 & 다시 그리기
    ```
    function filterAndRender() {
    const store = document.getElementById('storeFilter').value;
    const date = document.getElementById('qDate').value;
    const keyword = document.getElementById('qKeyword').value.trim().toLowerCase();

    SALES_VIEW = SALES.filter(r => {
        const okStore = store ? r.store === store : true;
        const okDate = date ? (r.date || '').startsWith(date) : true;
        const okKey = keyword ? (r.item || '').toLowerCase().includes(keyword) : true;
        return okStore && okDate && okKey;
    });

    renderTable();
    updateDashboard();
    loadMartRecommendations(); // 독점상품
    loadOrderGuide();          // 발주가이드
    }
    ```
    - SALES 전체에서 다시 필터해서 SALES_VIEW 만들고, 
    - 그걸 기준으로
        - 테이블 다시 그리기
        - 요약 + 차트 업데이트
        - 독점상품 / 발주가이드도 다시 불러옴


    4-4 상품 목록 / 판매 목록 / CRUD API
    - 상품 목록 로드
    ```
    async function loadProducts() {
    const res = await fetch('sales_api.php?action=products');
    PRODUCTS = await res.json();
    
    const sel = document.getElementById('fProduct');
    const fil = document.getElementById('storeFilter');
    sel.innerHTML = '';
    fil.innerHTML = '<option value="">전체</option>';
    
    const storeSet = new Set();
    PRODUCTS.forEach(p => {
        sel.innerHTML += `<option value="${p.id}">[${p.store}] ${p.name}</option>`;
        storeSet.add(p.store);
    });
    
    storeSet.forEach(s => fil.innerHTML += `<option value="${s}">${s}</option>`);
    }
    ```
    - action=products API 호출 → 상품 목록, 매장정보 리턴
    - 오른쪽 폼의 <select id="fProduct"> 옵션 채우고, 상단의 매장 필터 <select id="storeFilter"> 도 여기서 같이 채움

    - 판매 목록 로드
    ```
    async function fetchList() {
    const res = await fetch('sales_api.php');
    const data = await res.json();
    SALES = data || [];
    SALES.forEach(r => r.revenue = Number(r.revenue));
    filterAndRender();
    }
    ```
    - sales_api.php 을 기본 호출 → 판매 기록 전체 JSON 응답
    - SALES에 저장한 후 filterAndRender() 호출해서 화면에 반영

    - 수정/삭제/저장
    ```
    window.onEdit = (id) => {
    editId = id;
    const row = SALES.find(x => x.id == id);
    document.getElementById('formTitle').innerText = '행 수정';
    document.getElementById('formHint').innerText = '모드: UPDATE';
    
    document.getElementById('fDate').value = row.date;
    document.getElementById('fQty').value = row.qty;
    document.getElementById('fPrice').value = row.price;
    document.getElementById('fProduct').value = row.product_id;
    };
    ```
    - 전역 window.onEdit 로 등록 → HTML onclick="onEdit(id)" 에서 호출됨
    - 수정할 id를 기준으로 SALES에서 해당 행 찾음
    - 폼에 기존 값을 채워 넣고, “모드: UPDATE” 표시
    - editId 에 수정 대상 id 저장

    ```
    window.onDel = async (id) => {
    if (confirm('삭제하시겠습니까?')) {
        await sendAPI({ mode: 'DELETE', id });
    }
    };
    ```
    - 삭제 버튼 누르면 confirm()으로 확인 → OK일 때만 DELETE 요청 보내기

    ```
    async function sendAPI(payload) {
    await fetch('sales_api.php', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
    });
    await fetchList();
    document.getElementById('btnCancel').click();
    }
    ```
    - INSERT / UPDATE / DELETE 모두 이 함수 사용
    - payload 예:
        - INSERT: { mode:'INSERT', date:..., qty:..., price:..., product_id:... }
        - UPDATE: { mode:'UPDATE', id:..., date:..., ... }
        - DELETE: { mode:'DELETE', id:... }
    - 서버에 반영된 뒤 다시 목록 불러오기 + 폼 리셋

    ```
    document.getElementById('btnSave').onclick = () => {
    const d = document.getElementById('fDate').value;
    const q = document.getElementById('fQty').value;
    const p = document.getElementById('fPrice').value;
    const pd = document.getElementById('fProduct').value;
    
    if (d && q && p && pd) {
        sendAPI({
        mode: editId ? 'UPDATE' : 'INSERT',
        id: editId,
        date: d, qty: q, price: p, product_id: pd
        });
    } else {
        alert('모든 값을 입력해주세요.');
    }
    };
    ```
    - 저장 버튼 클릭 시
        - 값이 모두 채워졌는지 확인
        - editId가 있으면 UPDATE, 없으면 INSERT
        - 나머지는 sendAPI()한테 맡김
    
    ```
    document.getElementById('btnCancel').onclick = () => {
    editId = null;
    document.getElementById('formTitle').innerText = '신규 등록';
    document.getElementById('formHint').innerText = '모드: INSERT';
    document.getElementById('fDate').value = '';
    document.getElementById('fQty').value = '';
    document.getElementById('fPrice').value = '';
    };
    ```
    - 취소 버튼 -> 폼 비우고 모드 다시 INSERT로

    4-5. 페이지가 처음 로드될 때(window.onload)
    ```
    window.onload = async () => {
    await loadProducts();
    await fetchList();

    document.getElementById('btnSelect').onclick = filterAndRender;
    document.getElementById('refreshBtn').onclick = fetchList;
    document.getElementById('storeFilter').onchange = filterAndRender;
    
    document.getElementById('btnReset').onclick = () => {
        document.getElementById('qDate').value = '';
        document.getElementById('qKeyword').value = '';
        filterAndRender();
        };
    };
    ```
    - 페이지가 딱 처음 열렸을 때 실행되는 부분
    - loadProducts() → 상품 목록 + 매장 목록 불러오기
    - fetchList() → 판매 기록 전체 불러오기 (테이블, 차트, 카드에 반영)
    - 버튼/셀렉트에 이벤트 연결:
        - 조회 버튼 → filterAndRender()
        - 새로고침 버튼 → DB에서 전체 목록 다시 (fetchList)
        - 매장 변경 → 필터 적용 + 대시보드/추천 갱신
        - 초기화 버튼 → 날짜/검색어 비우고 다시 필터