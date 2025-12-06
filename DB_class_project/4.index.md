# index.php 풀 코드
```
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>소상공인 판매관리</title>

<style>
/* ================================
   1. 공통 스타일 및 색상 변수
   - 전체 페이지 톤, 폰트, 레이아웃 기본값
================================== */
:root {
  /* 색상 팔레트 (필요시 한 번에 수정 가능) */
  --bg: #ffffff;
  --text: #111827;
  --muted: #6b7280;
  --border: #e5e7eb;
  --accent: #2563eb;
}

/* 모든 요소에 box-sizing 지정 (padding, border 포함한 width 계산) */
* { box-sizing: border-box; }

/* body 공통 스타일 */
body {
  margin: 0;
  background: #f3f4f6;  /* 회색 배경 (카드들이 떠 있는 느낌) */
  color: var(--text);
  font: 14px/1.6 system-ui, -apple-system, sans-serif;
}

/* ================================
   상단 헤더 영역 (매장 선택, 새로고침)
================================== */
.top {
  position: sticky;        /* 스크롤 시 상단에 고정 */
  top: 0;
  z-index: 10;
  background: #ffffff;
  border-bottom: 1px solid var(--border);
}

.top-inner {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between; /* 좌: 로고, 우: 액션 버튼들 */
  padding: 10px 20px;
}

/* 로고 + 서비스 이름 영역 */
.brand {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: 700;
  font-size: 15px;
}

/* 로고 박스 (이모지 포함 동그란 박스) */
.logo {
  width: 28px;
  height: 28px;
  border-radius: 8px;
  display: grid;
  place-items: center;  /* 가로/세로 가운데 정렬 */
  background: var(--accent);
  color: #fff;
  font-size: 18px;
}

/* 상단 우측 액션 영역 (매장 선택, 새로고침 버튼 등) */
.actions {
  display: flex;
  align-items: center;
  gap: 8px;
}

.actions label {
  font-size: 12px;
  color: var(--muted);
}

/* 공통 인풋 스타일 (select, input 등) */
.input {
  padding: 6px 10px;
  border-radius: 8px;
  border: 1px solid var(--border);
  background: #f9fafb;
  font-size: 13px;
  min-height: 32px;
}

/* 공통 버튼 스타일 */
.btn {
  border-radius: 999px;
  border: 1px solid var(--border);
  background: #ffffff;
  padding: 6px 12px;
  font-size: 13px;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  cursor: pointer;
}

/* 파란색(강조) 버튼 */
.btn.primary {
  background: var(--accent);
  border-color: var(--accent);
  color: #ffffff;
}

/* 연한 회색 배경 버튼 (보조 버튼 느낌) */
.btn.ghost {
  background: #f9fafb;
}

/* 비활성화된 버튼 상태 */
.btn:disabled {
  opacity: .5;
  cursor: not-allowed;
}

/* ================================
   메인 컨테이너 (전체 레이아웃의 폭)
================================== */
.container {
  max-width: 1200px;
  margin: 18px auto 32px;
  padding: 0 20px;
}

h2 { margin: 0 0 12px; font-size: 20px; }

.subtext {
  font-size: 12px;
  color: var(--muted);
  margin-bottom: 16px;
}

/* ================================
   카드 공통 스타일
================================== */
.card {
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 18px 40px rgba(15, 23, 42, 0.08);
  padding: 18px 20px 20px;
}

.card h3 { margin: 0 0 12px; font-size: 16px; }

/* ================================
   대시보드 전체 섹션 박스
================================== */
.dash-section {
  margin-top: 28px;
  padding: 24px 28px 30px;
  background: #ffffff;
  border-radius: 16px;
  border: 1px solid #e5e7eb;
  box-shadow: 0 18px 40px rgba(15, 23, 42, 0.08);
}

.dash-title {
  margin: 0 0 4px;
  font-size: 18px;
  font-weight: 700;
}

.dash-sub {
  margin: 0 0 18px;
  font-size: 12px;
  color: var(--muted);
}

/* ================================
   대시보드 상단 요약 카드 (총 매출액, 수량 등)
================================== */
.dash-summary {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-bottom: 20px;
}

.dash-card {
  flex: 1 1 180px;
  min-width: 180px;
  background: #f9fafb;
  border-radius: 12px;
  padding: 12px 14px;
  border: 1px solid #e5e7eb;
}

.dash-value {
  margin: 0;
  font-size: 18px;
  font-weight: 700;
}

/* ================================
   대시보드 차트 영역 (월별 매출 / 품목별 수량)
================================== */
.dash-charts {
  display: grid;
  grid-template-columns: minmax(0, 1.6fr) minmax(0, 1.2fr); /* 왼쪽이 좀 더 넓게 */
  gap: 18px;
}

.dash-chart-card {
  background: #f9fafb;
  border-radius: 12px;
  padding: 14px 16px 18px;
  border: 1px solid #e5e7eb;
}

/* 캔버스는 width 100%로, 높이는 최대 280px */
canvas { width: 100%; max-height: 280px; }

/* ================================
   반응형 스타일 (화면이 좁아졌을 때)
================================== */
@media(max-width: 960px) {
  /* 차트를 세로로 쌓기 */
  .dash-charts { grid-template-columns: minmax(0, 1fr); }
  /* CRUD 그리드도 한 열로 쌓기 */
  .crud-grid { grid-template-columns: 1fr; }
}

/* ================================
   CRUD 레이아웃 (좌: 테이블, 우: 입력폼)
================================== */
.crud-grid {
  display: grid;
  gap: 20px;
  grid-template-columns: 2.2fr 1fr; /* 왼쪽 크게, 오른쪽 작게 */
}

/* 테이블 상단 툴바 (조회, 초기화, 필터) */
.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  margin-bottom: 12px;
}

.table-wrap { margin-top: 8px; }

/* 기본 테이블 스타일 */
table {
  width: 100%;
  border-collapse: collapse; /* 테두리 겹침 제거 */
  font-size: 13px;
}

th, td {
  padding: 8px 10px;
  border-bottom: 1px solid #e5e7eb; /* 행 구분선 */
  text-align: left;
  vertical-align: middle;
}

thead th {
  background: #f9fafb;
  font-weight: 600;
}

/* 입력폼 그룹 (라벨 + 인풋) */
.form-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 10px;
  gap: 4px;
}

/* 입력폼 하단 버튼 영역 (저장/취소) */
.form-actions {
  display: flex;
  gap: 8px;
  margin-top: 4px;
}

.form-actions .btn {
  flex: 1;  /* 버튼 2개를 1:1 비율로 채우기 */
}

/* ================================
   신규 기능: 독점상품 / 발주가이드 카드 레이아웃
================================== */
.new-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(240px, 1fr)); /* 카드 자동 배치 */
  gap: 12px;
}

/* 독점상품 / 발주가이드 카드 기본 스타일 */
.new-card {
  background: #ffffff;
  border: 1px solid #e5e7eb;
  border-radius: 12px;
  padding: 16px;
  display: flex;
  flex-direction: column;
}

/* 카드 상단에 붙는 작은 뱃지 (카테고리, 상태 등) */
.nc-badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 999px;
  font-size: 11px;
  font-weight: 700;
  margin-bottom: 6px;
  width: fit-content;
}

/* 카드 타이틀(상품명 등) */
.nc-title {
  font-size: 15px;
  font-weight: 700;
  margin-bottom: 4px;
  color: var(--text);
}

/* 카드 설명 텍스트 */
.nc-desc {
  font-size: 12px;
  color: var(--muted);
  line-height: 1.4;
}

/* 카드 안의 주요 수치(예상 매출 등) */
.nc-val {
  font-size: 13px;
  font-weight: 600;
  margin-top: 8px;
  color: #111827;
}

/* 파란 계열 뱃지 */
.theme-blue {
  background: #eff6ff;
  color: var(--accent);
}

/* 보라 계열 뱃지 */
.theme-purple {
  background: #f3e8ff;
  color: #7c3aed;
}
</style>

<!-- 차트 라이브러리: Chart.js CDN -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>

<!-- ================================
     상단 고정 헤더 (서비스 로고 + 매장 선택)
================================== -->
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

<!-- ================================
     메인 컨텐츠 영역
================================== -->
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

  <!-- 2. 대형마트와 비교한 우리 매장 독점 상품 리스트 -->
  <section class="dash-section">
    <h2 class="dash-title">💎 우리 매장만의 독점 상품</h2>
    <p class="dash-sub">인근 대형마트에는 없는, 우리 매장만의 차별화된 상품입니다.</p>
    <div id="martRecList" class="new-grid">
      <!-- JS에서 API 호출 결과로 내용을 채움 -->
      <div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">데이터 분석 중...</div>
    </div>
  </section>

  <!-- 3. 다음 달 발주 가이드 (예측 결과 기반 추천) -->
  <section class="dash-section">
    <h2 class="dash-title">📅 다음 달 발주 가이드</h2>
    <p class="dash-sub">과거 데이터를 분석하여 발주량을 늘려야 할 상품을 추천합니다.</p>
    <div id="orderGuideList" class="new-grid">
      <!-- JS에서 예측 API 호출 결과로 내용을 채움 -->
      <div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">데이터 분석 중...</div>
    </div>
  </section>

  <!-- ================================
       CRUD 영역 (좌: 테이블, 우: 입력폼)
  ================================== -->
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

<script>
// ==========================================
/* 전역 상태 관리 변수
   - SALES      : 서버에서 가져온 전체 매출 데이터
   - SALES_VIEW : 필터가 적용된 후 화면에 보여주는 데이터
   - PRODUCTS   : 상품 리스트 (상품 + 매장 정보)
   - editId     : 수정 중인 행의 ID (null이면 신규등록 모드)
   - lineChart / donutChart : Chart.js 차트 인스턴스
*/
// ==========================================
let SALES = [];
let SALES_VIEW = [];
let PRODUCTS = [];
let editId = null;

let lineChart = null;
let donutChart = null;

/* 숫자 포맷 함수
   - 입력 숫자를 한국식 천단위 콤마로 변환
   - Number(n) || 0 : 숫자 변환 실패 시 0 처리
*/
const fmt = n => new Intl.NumberFormat('ko-KR').format(Number(n) || 0);

// ==========================================
// 1. 대형마트 비교 (독점 상품) 데이터 로드
//    - 대형마트에 없는 우리 매장만의 상품 목록
// ==========================================
async function loadMartRecommendations() {
  const storeFilter = document.getElementById('storeFilter');
  const storeName = storeFilter.options[storeFilter.selectedIndex].text; // 선택된 매장 이름
  const storeVal = storeFilter.value; // 선택된 매장 value (빈 값이면 전체)
  const box = document.getElementById('martRecList');

  // 매장이 선택되지 않은 경우 안내 메시지 출력
  if (storeVal === "") {
    box.innerHTML = '<div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">매장을 선택하면 분석 결과가 나옵니다.</div>';
    return;
  }

  try {
    // PHP API 호출: 대형마트 비교 분석 결과
    const res = await fetch(`sales_api.php?action=mart_recommendation&store=${encodeURIComponent(storeName)}`);
    const data = await res.json();
    box.innerHTML = '';

    // 독점 상품이 없는 경우
    if (!data.length) {
      box.innerHTML = '<div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">독점 상품이 없습니다. (대형마트와 모두 겹침)</div>';
      return;
    }

    // 상위 6개만 카드로 표시
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

// ==========================================
// 2. 발주 가이드 로드
//    - 다음 달 매출 예측 결과 기반으로 발주 추천
// ==========================================
async function loadOrderGuide() {
  const storeFilter = document.getElementById('storeFilter');
  const storeName = storeFilter.options[storeFilter.selectedIndex].text;
  const storeVal = storeFilter.value;
  const box = document.getElementById('orderGuideList');

  // 매장을 선택하지 않은 경우
  if (storeVal === "") {
    box.innerHTML = '<div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">매장을 선택하면 발주 가이드가 나옵니다.</div>';
    return;
  }

  try {
    // PHP API 호출: 예측 + 트렌드 정보 (trend: up/down 등)
    const res = await fetch(`sales_api.php?action=predict&store=${encodeURIComponent(storeName)}`);
    const data = await res.json();
    box.innerHTML = '';

    // 데이터가 없는 경우
    if (!data.length) {
      box.innerHTML = '<div style="grid-column:1/-1; text-align:center; padding:10px; color:#999;">데이터 부족으로 분석 불가</div>';
      return;
    }

    // 상승 추세(Up) 상품과 나머지 분리
    const upList = data.filter(i => i.trend === 'up');
    const downList = data.filter(i => i.trend !== 'up');
    // 우선 순위: 상승 추세 → 나머지, 총 6개까지
    const list = [...upList, ...downList].slice(0, 6);

    list.forEach(item => {
      const isUp = item.trend === 'up';
      // 추세에 따라 뱃지와 문구 변경
      const badgeHtml = isUp 
        ? '<span class="nc-badge theme-blue">🔥 발주 증량</span>' 
        : '<span class="nc-badge" style="background:#f3f4f6; color:#6b7280;">유지/감소</span>';
      
      const borderColor = isUp ? '#2563eb' : '#e5e7eb';
      const bgStyle = isUp ? 'background:#fdfdff;' : '';

      // 카드 UI 렌더링
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

// ==========================================
// 3. 기본 화면 렌더링 (테이블 + 대시보드 차트)
// ==========================================

/* 판매 데이터 테이블 렌더링
   - SALES_VIEW 배열을 기반으로 <tbody> 내용 생성
*/
function renderTable() {
  const tb = document.querySelector('#salesTable tbody');
  tb.innerHTML = '';

  // 표시할 데이터가 없을 때
  if (!SALES_VIEW.length) {
    tb.innerHTML = '<tr><td colspan="6" style="text-align:center;color:#999">데이터가 없습니다.</td></tr>';
    return;
  }
  
  SALES_VIEW.forEach(r => {
    // revenue가 있으면 사용, 없으면 qty * price로 계산
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
          <!-- 행 수정 버튼: onEdit(id) 호출 -->
          <button class="btn ghost" onclick="onEdit(${r.id})">수정</button>
          <!-- 행 삭제 버튼: onDel(id) 호출 -->
          <button class="btn ghost" onclick="onDel(${r.id})">삭제</button>
        </td>
      </tr>`;
  });
}

/* 대시보드 상단 요약 + 차트 업데이트
   - SALES_VIEW 기준으로 계산
   - 총 매출, 총 수량, 평균 단가, 건수
   - 월별 매출 라인차트, 품목별 수량 도넛차트
*/
function updateDashboard() {
  const totalEl = document.getElementById('dashTotal');
  const qtyEl = document.getElementById('dashQty');
  const avgEl = document.getElementById('dashAvgPrice');
  const countEl = document.getElementById('dashCount');
  
  // 엘리먼트가 없으면 종료 (안전장치)
  if (!totalEl) return;
  
  let totalRevenue = 0, totalQty = 0, totalPriceSum = 0;
  const byMonth = {}; // { '2024-01': 매출합, ... }
  const byItem = {};  // { '바나나': 수량합, ... }

  SALES_VIEW.forEach(r => {
    const qty = Number(r.qty) || 0;
    const price = Number(r.price) || 0;
    const rev = r.revenue ? Number(r.revenue) : qty * price;

    totalRevenue += rev;
    totalQty += qty;
    totalPriceSum += price;

    // 날짜에서 "년-월"만 추출 (YYYY-MM 포맷)
    const ym = (r.date || '').slice(0, 7);
    byMonth[ym] = (byMonth[ym] || 0) + rev;

    // 품목별(아이템명) 수량 집계
    const itemName = r.item || '기타';
    byItem[itemName] = (byItem[itemName] || 0) + qty;
  });

  // 상단 요약 카드 텍스트 세팅
  totalEl.innerText = '₩' + fmt(totalRevenue);
  qtyEl.innerText = fmt(totalQty) + ' 개';
  avgEl.innerText = '₩' + fmt(SALES_VIEW.length ? Math.round(totalPriceSum / SALES_VIEW.length) : 0);
  countEl.innerText = fmt(SALES_VIEW.length) + ' 건';

  // ------------------------------
  // 라인 차트: 월별 매출 추이
  // ------------------------------
  const monthLabels = Object.keys(byMonth).sort();

  // 기존 차트가 있다면 먼저 제거
  if (lineChart) lineChart.destroy();

  // 새로 라인차트 생성
  lineChart = new Chart(document.getElementById('lineChart'), {
    type: 'line',
    data: {
      labels: monthLabels,
      datasets: [{
        label: '매출',
        data: monthLabels.map(k => byMonth[k]),
        borderColor: '#2563eb',
        tension: 0.25  // 곡선 정도
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: { y: { beginAtZero: true } }
    }
  });

  // ------------------------------
  // 도넛 차트: 품목별 판매 수량
  // ------------------------------
  const itemEntries = Object.entries(byItem)
    .sort((a, b) => b[1] - a[1]) // 수량 기준 내림차순
    .slice(0, 10);               // 상위 10개만 표시

  // 기존 도넛차트 제거
  if (donutChart) donutChart.destroy();

  // 새 도넛차트 생성
  donutChart = new Chart(document.getElementById('donutChart'), {
    type: 'doughnut',
    data: {
      labels: itemEntries.map(e => e[0]),
      datasets: [{
        data: itemEntries.map(e => e[1]),
        // 간단히 5가지 색 반복 (데모용)
        backgroundColor: ['#3b82f6', '#10b981', '#f59e0b', '#ef4444', '#8b5cf6']
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      cutout: '60%' // 가운데 구멍 크기
    }
  });
}

/* 필터 조건(매장, 날짜, 검색어)을 적용하고
   - SALES_VIEW를 갱신한 뒤
   - 테이블 + 대시보드 + 독점상품 + 발주가이드를 모두 재렌더링
*/
function filterAndRender() {
  const store = document.getElementById('storeFilter').value;
  const date = document.getElementById('qDate').value;
  const keyword = document.getElementById('qKeyword').value.trim().toLowerCase();

  // SALES에서 필터 조건을 만족하는 데이터만 추출
  SALES_VIEW = SALES.filter(r => {
    const okStore = store ? r.store === store : true;                       // 매장 필터
    const okDate = date ? (r.date || '').startsWith(date) : true;           // 년-월 필터
    const okKey = keyword ? (r.item || '').toLowerCase().includes(keyword) : true; // 품목 검색
    return okStore && okDate && okKey;
  });

  // 테이블 + 대시보드 재렌더링
  renderTable();
  updateDashboard();
  // 독점상품/발주가이드는 매장별 결과이므로 같이 업데이트
  loadMartRecommendations();
  loadOrderGuide();
}

// ==========================================
// 4. API 통신 및 CRUD 이벤트 핸들러
// ==========================================

/* 상품 목록 로드
   - sales_api.php?action=products
   - 결과로 PRODUCTS 배열 채우고
   - 상품 선택 select(fProduct) + 매장 필터(storeFilter)에 옵션 추가
*/
async function loadProducts() {
  const res = await fetch('sales_api.php?action=products');
  PRODUCTS = await res.json();
  
  const sel = document.getElementById('fProduct');
  const fil = document.getElementById('storeFilter');
  sel.innerHTML = '';
  fil.innerHTML = '<option value="">전체</option>';
  
  const storeSet = new Set(); // 매장 이름 중복 제거용

  PRODUCTS.forEach(p => {
    // 상품 선택 select: [매장명] 상품명 형태로 표시
    sel.innerHTML += `<option value="${p.id}">[${p.store}] ${p.name}</option>`;
    storeSet.add(p.store); // 매장 이름 수집
  });
  
  // storeFilter에 매장 목록 추가
  storeSet.forEach(s => fil.innerHTML += `<option value="${s}">${s}</option>`);
}

/* 판매 데이터 목록 로드
   - sales_api.php (GET)
   - SALES를 채우고, 필터 적용 함수 호출
*/
async function fetchList() {
  const res = await fetch('sales_api.php');
  const data = await res.json();
  SALES = data || [];
  // revenue 필드는 숫자로 강제 변환 (NaN 방지용)
  SALES.forEach(r => r.revenue = Number(r.revenue));
  filterAndRender();
}

/* 수정 버튼 클릭 시 호출되는 전역 함수
   - 선택한 id에 해당하는 행을 SALES에서 찾고
   - 우측 폼에 값을 채운 뒤 UPDATE 모드로 전환
*/
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

/* 삭제 버튼 클릭 시 호출되는 전역 함수
   - confirm으로 사용자 확인 후
   - sendAPI에 mode: DELETE 요청
*/
window.onDel = async (id) => {
  if (confirm('삭제하시겠습니까?')) {
    await sendAPI({ mode: 'DELETE', id });
  }
};

/* 공통 API 전송 함수(INSERT, UPDATE, DELETE)
   - 모두 POST JSON 형태로 sales_api.php에 전송
   - 완료 후 목록 재조회 + 폼 초기화
*/
async function sendAPI(payload) {
  await fetch('sales_api.php', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
  });
  await fetchList();                // 데이터 다시 불러오기
  document.getElementById('btnCancel').click(); // 폼 초기화
}

/* 저장 버튼 클릭 시
   - 폼에서 값 읽어와서 유효성 체크
   - INSERT 또는 UPDATE 모드에 따라 mode 필드 변경
*/
document.getElementById('btnSave').onclick = () => {
  const d = document.getElementById('fDate').value;
  const q = document.getElementById('fQty').value;
  const p = document.getElementById('fPrice').value;
  const pd = document.getElementById('fProduct').value;
  
  // 필수값들이 모두 입력되었는지 체크
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

/* 취소 버튼 클릭 시
   - editId 초기화
   - 폼의 모든 입력값 비우기
   - 폼 타이틀/힌트를 INSERT 모드로 되돌리기
*/
document.getElementById('btnCancel').onclick = () => {
  editId = null;
  document.getElementById('formTitle').innerText = '신규 등록';
  document.getElementById('formHint').innerText = '모드: INSERT';
  document.getElementById('fDate').value = '';
  document.getElementById('fQty').value = '';
  document.getElementById('fPrice').value = '';
};

// ==========================================
// 5. 페이지 로드 시 초기화
//    - 상품 목록, 판매 목록 로드
//    - 버튼/필터 이벤트 연결
// ==========================================
window.onload = async () => {
  // 1) 상품 목록 + 매장 목록 로드
  await loadProducts();
  // 2) 판매 데이터 로드
  await fetchList();

  // 조회 버튼: 현재 필터 적용
  document.getElementById('btnSelect').onclick = filterAndRender;
  // 새로고침 버튼: 서버에서 다시 전체 목록 가져오기
  document.getElementById('refreshBtn').onclick = fetchList;
  // 매장 선택 변경 시 필터 재적용
  document.getElementById('storeFilter').onchange = filterAndRender;
  
  // 초기화 버튼: 날짜/키워드 비우고 필터 재적용
  document.getElementById('btnReset').onclick = () => {
    document.getElementById('qDate').value = '';
    document.getElementById('qKeyword').value = '';
    filterAndRender();
  };
};
</script>
</body>
</html>

```