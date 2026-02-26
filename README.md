---
---
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>EMA Step Up — Vendor Dashboard</title>
<script src="https://cdn.tailwindcss.com"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<link rel="preconnect" href="https://fonts.googleapis.com" />
<link href="https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500;600&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet" />
<style>
  :root {
    --bg:#0e1117; --surface:#161b27; --surface2:#1e2535; --border:#2a3249;
    --amber:#f59e0b; --green:#10b981; --red:#ef4444; --blue:#3b82f6;
    --purple:#8b5cf6; --text:#e2e8f0; --muted:#64748b;
  }
  *{box-sizing:border-box;margin:0;padding:0;}
  body{font-family:'DM Sans',sans-serif;background:var(--bg);color:var(--text);min-height:100vh;}
  ::-webkit-scrollbar{width:6px;height:6px;}
  ::-webkit-scrollbar-track{background:var(--bg);}
  ::-webkit-scrollbar-thumb{background:var(--border);border-radius:3px;}

  .nav-tab{padding:7px 16px;border-radius:8px;font-size:0.8rem;font-weight:500;cursor:pointer;border:1px solid transparent;background:none;color:var(--muted);transition:all 0.15s;display:inline-flex;align-items:center;gap:6px;}
  .nav-tab.active{background:var(--surface2);color:var(--text);border-color:var(--border);}
  .nav-tab:hover:not(.active){color:var(--text);}

  #drop-zone{border:2px dashed var(--border);transition:border-color 0.2s,background 0.2s;}
  #drop-zone.drag-over{border-color:var(--amber);background:rgba(245,158,11,0.05);}

  .data-table{width:100%;border-collapse:collapse;}
  .data-table th{background:var(--surface2);color:var(--muted);font-size:0.7rem;font-weight:500;letter-spacing:0.1em;text-transform:uppercase;padding:10px 16px;text-align:left;cursor:pointer;user-select:none;white-space:nowrap;}
  .data-table th:hover{color:var(--text);}
  .data-table td{padding:11px 16px;font-size:0.875rem;border-bottom:1px solid var(--border);vertical-align:middle;}
  .data-table tr:hover td{background:rgba(255,255,255,0.025);cursor:pointer;}

  .badge{display:inline-flex;align-items:center;gap:4px;padding:2px 9px;border-radius:999px;font-size:0.72rem;font-weight:500;font-family:'DM Mono',monospace;letter-spacing:0.02em;}
  .badge-paid{background:rgba(16,185,129,0.12);color:#34d399;}
  .badge-pending{background:rgba(245,158,11,0.12);color:#fbbf24;}
  .badge-failed{background:rgba(239,68,68,0.12);color:#f87171;}
  .badge-default{background:rgba(100,116,139,0.15);color:#94a3b8;}
  .badge-vip{background:rgba(139,92,246,0.15);color:#a78bfa;}

  #crm-panel{position:fixed;top:0;right:0;width:380px;height:100vh;background:var(--surface);border-left:1px solid var(--border);transform:translateX(100%);transition:transform 0.3s cubic-bezier(0.4,0,0.2,1);z-index:50;overflow-y:auto;display:flex;flex-direction:column;}
  #crm-panel.open{transform:translateX(0);}
  #crm-overlay{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.45);z-index:49;}
  #crm-overlay.open{display:block;}

  #print-modal{display:none;position:fixed;inset:0;background:rgba(0,0,0,0.7);z-index:60;align-items:center;justify-content:center;padding:20px;}
  #print-modal.open{display:flex;}
  #print-modal-inner{background:var(--surface);border:1px solid var(--border);border-radius:16px;width:100%;max-width:700px;max-height:90vh;overflow-y:auto;}

  .form-input{width:100%;background:var(--bg);border:1px solid var(--border);border-radius:8px;color:var(--text);padding:9px 12px;font-family:'DM Sans',sans-serif;font-size:0.875rem;transition:border-color 0.2s;outline:none;}
  .form-input:focus{border-color:var(--amber);}
  textarea.form-input{resize:vertical;min-height:90px;}

  .btn{display:inline-flex;align-items:center;gap:6px;padding:8px 16px;border-radius:8px;font-size:0.875rem;font-weight:500;cursor:pointer;border:none;transition:opacity 0.15s,transform 0.1s;}
  .btn:active{transform:scale(0.97);}
  .btn-amber{background:var(--amber);color:#0e1117;}.btn-amber:hover{opacity:0.9;}
  .btn-ghost{background:var(--surface2);color:var(--text);border:1px solid var(--border);}.btn-ghost:hover{border-color:var(--amber);color:var(--amber);}
  .btn-green{background:rgba(16,185,129,0.15);color:#34d399;border:1px solid rgba(16,185,129,0.3);}.btn-green:hover{background:rgba(16,185,129,0.25);}
  .btn-blue{background:rgba(59,130,246,0.15);color:#93c5fd;border:1px solid rgba(59,130,246,0.3);}.btn-blue:hover{background:rgba(59,130,246,0.25);}
  .btn-red{background:rgba(239,68,68,0.15);color:#fca5a5;border:1px solid rgba(239,68,68,0.3);}.btn-red:hover{background:rgba(239,68,68,0.25);}

  .stat-card{background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:20px 24px;position:relative;overflow:hidden;}
  .stat-card::before{content:'';position:absolute;top:0;left:0;right:0;height:2px;}
  .stat-card.card-amber::before{background:var(--amber);}
  .stat-card.card-green::before{background:var(--green);}
  .stat-card.card-blue::before{background:var(--blue);}
  .stat-card.card-red::before{background:var(--red);}

  .pipeline-bar{height:6px;border-radius:3px;background:var(--border);overflow:hidden;margin-top:8px;}
  .pipeline-fill{height:100%;border-radius:3px;transition:width 0.6s ease;}

  input[type="date"]{background:var(--surface2);border:1px solid var(--border);border-radius:8px;color:var(--text);padding:7px 10px;font-family:'DM Mono',monospace;font-size:0.8rem;outline:none;transition:border-color 0.2s;color-scheme:dark;}
  input[type="date"]:focus{border-color:var(--amber);}
  input[type="date"]::-webkit-calendar-picker-indicator{filter:invert(0.7);cursor:pointer;}

  .chart-tab{padding:5px 14px;border-radius:6px;font-size:0.75rem;font-weight:500;cursor:pointer;border:1px solid transparent;background:none;color:var(--muted);transition:all 0.15s;}
  .chart-tab.active{background:var(--surface2);border-color:var(--border);color:var(--text);}
  .chart-tab:hover:not(.active){color:var(--text);}

  #toast{position:fixed;bottom:24px;right:24px;background:var(--surface2);border:1px solid var(--amber);color:var(--text);padding:12px 20px;border-radius:10px;font-size:0.875rem;transform:translateY(80px);opacity:0;transition:all 0.3s;z-index:100;}
  #toast.show{transform:translateY(0);opacity:1;}

  .mono{font-family:'DM Mono',monospace;}
  .section-view{display:none;}
  .section-view.active{display:block;}

  @media print{
    body *{visibility:hidden;}
    #printable,#printable *{visibility:visible;}
    #printable{position:fixed;inset:0;background:white;color:black;padding:32px;font-family:sans-serif;}
  }
  /* Family accordion */
  .family-row{background:var(--surface);border:1px solid var(--border);border-radius:10px;margin-bottom:8px;overflow:hidden;}
  .family-header{display:flex;align-items:center;justify-content:space-between;padding:13px 16px;cursor:pointer;gap:12px;transition:background 0.15s;}
  .family-header:hover{background:rgba(255,255,255,0.03);}
  .family-header.expanded{border-bottom:1px solid var(--border);}
  .family-chevron{transition:transform 0.2s;flex-shrink:0;}
  .family-header.expanded .family-chevron{transform:rotate(90deg);}
  .student-row{display:flex;align-items:center;gap:12px;padding:10px 16px 10px 36px;border-bottom:1px solid rgba(42,50,73,0.5);font-size:0.85rem;}
  .student-row:last-child{border-bottom:none;}
  .student-row:hover{background:rgba(255,255,255,0.02);}

  /* Invoice queue */
  .invoice-item{background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:14px 16px;margin-bottom:8px;display:flex;align-items:center;gap:12px;flex-wrap:wrap;}
  .invoice-item.ready{border-left:3px solid var(--amber);}
  .invoice-item.submitted{border-left:3px solid var(--blue);}
  .invoice-item.processing{border-left:3px solid var(--purple);}
  .urgency-dot{width:8px;height:8px;border-radius:50%;flex-shrink:0;}

  @media(max-width:640px){#crm-panel{width:100vw;}}
</style>
</head>
<body>

<!-- HEADER -->
<header style="background:var(--surface);border-bottom:1px solid var(--border);" class="px-5 py-3 flex items-center justify-between sticky top-0 z-40 flex-wrap gap-3">
  <div class="flex items-center gap-3">
    <div style="background:var(--amber);border-radius:8px;width:32px;height:32px;flex-shrink:0;" class="flex items-center justify-center">
      <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="#0e1117" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/><polyline points="17 6 23 6 23 12"/></svg>
    </div>
    <div>
      <div class="font-semibold text-sm">EMA Step Up</div>
      <div class="text-xs" style="color:var(--muted);">Vendor Dashboard</div>
    </div>
  </div>

  <div id="main-nav" style="display:none;" class="flex items-center gap-1 flex-wrap">
    <button class="nav-tab active" id="nav-overview" onclick="switchView('overview')">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>
      Overview
    </button>
    <button class="nav-tab" id="nav-actions" onclick="switchView('actions')">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 17H2a3 3 0 0 0 3-3V9a7 7 0 0 1 14 0v5a3 3 0 0 0 3 3zm-8.27 4a2 2 0 0 1-3.46 0"/></svg>
      Chase Unpaid
      <span id="nav-actions-badge" style="display:none;background:var(--red);color:white;border-radius:999px;font-size:0.65rem;padding:1px 6px;font-family:'DM Mono',monospace;"></span>
    </button>
    <button class="nav-tab" id="nav-customers" onclick="switchView('customers')">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
      Customers
    </button>
    <button class="nav-tab" id="nav-transactions" onclick="switchView('transactions')">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="8" y1="6" x2="21" y2="6"/><line x1="8" y1="12" x2="21" y2="12"/><line x1="8" y1="18" x2="21" y2="18"/><line x1="3" y1="6" x2="3.01" y2="6"/><line x1="3" y1="12" x2="3.01" y2="12"/><line x1="3" y1="18" x2="3.01" y2="18"/></svg>
      Transactions
    </button>
    <button class="nav-tab" id="nav-families" onclick="switchView('families')">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/></svg>
      Families
    </button>
    <button class="nav-tab" id="nav-invoice" onclick="switchView('invoice')">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>
      Invoice Queue
      <span id="nav-invoice-badge" style="display:none;background:var(--amber);color:#0e1117;border-radius:999px;font-size:0.65rem;padding:1px 6px;font-family:'DM Mono',monospace;font-weight:700;"></span>
    </button>
  </div>

  <div class="flex items-center gap-2 flex-wrap">
    <span id="file-name-label" class="text-xs mono" style="color:var(--muted);display:none;"></span>
    <button class="btn btn-ghost text-xs" id="report-btn" style="display:none;" onclick="openReport()">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/></svg>
      Report
    </button>
    <button class="btn btn-ghost text-xs" id="export-btn" style="display:none;" onclick="exportCSV()">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
      Export CSV
    </button>
    <button class="btn btn-ghost text-xs" id="reset-btn" style="display:none;" onclick="resetDashboard()">
      <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
      New File
    </button>
  </div>
</header>

<!-- MAIN -->
<main class="px-4 py-6 max-w-7xl mx-auto">

  <!-- UPLOAD -->
  <section id="upload-section">
    <div class="flex flex-col items-center justify-center" style="min-height:60vh;">
      <div id="drop-zone" class="rounded-2xl p-12 text-center w-full max-w-xl cursor-pointer" onclick="document.getElementById('csv-input').click()">
        <div style="background:rgba(245,158,11,0.1);border-radius:50%;width:72px;height:72px;" class="flex items-center justify-center mx-auto mb-5">
          <svg xmlns="http://www.w3.org/2000/svg" width="34" height="34" viewBox="0 0 24 24" fill="none" stroke="#f59e0b" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="12" y1="18" x2="12" y2="12"/><polyline points="9 15 12 12 15 15"/></svg>
        </div>
        <h2 class="text-lg font-semibold mb-2">Drop your CSV file here</h2>
        <p class="text-sm mb-6" style="color:var(--muted);">Drag & drop or click to browse. Exported from the EMA portal.</p>
        <span class="btn btn-amber text-sm">
          <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z"/></svg>
          Browse File
        </span>
        <p class="text-xs mt-5" style="color:var(--muted);">Supports: Date, Customer Name, Amount, Status columns</p>
      </div>
      <input type="file" id="csv-input" accept=".csv" class="hidden" onchange="handleFileInput(event)" />
      <button onclick="loadDemo()" class="mt-4 text-xs underline" style="color:var(--muted);background:none;border:none;cursor:pointer;">Load demo data instead</button>
    </div>
  </section>

  <!-- DASHBOARD -->
  <section id="dashboard-section" style="display:none;">

    <!-- VIEW: OVERVIEW -->
    <div id="view-overview" class="section-view active">
      <!-- 4 stat cards -->
      <div class="grid grid-cols-1 gap-4 mb-6" style="grid-template-columns:repeat(auto-fit,minmax(180px,1fr));">
        <div class="stat-card card-amber">
          <div class="flex items-center justify-between mb-3">
            <span class="text-xs uppercase tracking-widest" style="color:var(--muted);">Total Revenue</span>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="1" x2="12" y2="23"/><path d="M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
          </div>
          <div class="text-2xl font-semibold mono" id="stat-revenue">—</div>
          <div class="text-xs mt-1" style="color:var(--muted);" id="stat-revenue-sub"></div>
        </div>
        <div class="stat-card card-green">
          <div class="flex items-center justify-between mb-3">
            <span class="text-xs uppercase tracking-widest" style="color:var(--muted);">Active Customers</span>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#10b981" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>
          </div>
          <div class="text-2xl font-semibold mono" id="stat-customers">—</div>
          <div class="text-xs mt-1" style="color:var(--muted);" id="stat-customers-sub"></div>
        </div>
        <div class="stat-card card-blue">
          <div class="flex items-center justify-between mb-3">
            <span class="text-xs uppercase tracking-widest" style="color:var(--muted);">Pending Payments</span>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#3b82f6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><polyline points="12 6 12 12 16 14"/></svg>
          </div>
          <div class="text-2xl font-semibold mono" id="stat-pending">—</div>
          <div class="text-xs mt-1" style="color:var(--muted);" id="stat-pending-sub"></div>
        </div>
        <div class="stat-card card-red">
          <div class="flex items-center justify-between mb-3">
            <span class="text-xs uppercase tracking-widest" style="color:var(--muted);">At Risk / Failed</span>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#ef4444" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
          </div>
          <div class="text-2xl font-semibold mono" id="stat-failed">—</div>
          <div class="text-xs mt-1" style="color:var(--muted);" id="stat-failed-sub"></div>
        </div>
      </div>

      <!-- Payment Pipeline -->
      <div style="background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:20px 24px;" class="mb-6">
        <div class="flex items-center justify-between mb-4">
          <div>
            <h3 class="font-semibold text-sm">Payment Pipeline</h3>
            <p class="text-xs mt-0.5" style="color:var(--muted);">Where your money is right now</p>
          </div>
          <span class="text-xs mono" style="color:var(--muted);" id="pipeline-total"></span>
        </div>
        <div class="grid gap-3" id="pipeline-bars" style="grid-template-columns:repeat(auto-fit,minmax(140px,1fr));"></div>
      </div>

      <!-- Revenue Trend Chart -->
      <div style="background:var(--surface);border:1px solid var(--border);border-radius:12px;padding:20px 24px;" class="mb-6">
        <div class="flex items-center justify-between mb-4 flex-wrap gap-3">
          <div>
            <h3 class="font-semibold text-sm">Revenue Trend</h3>
            <p class="text-xs mt-0.5" style="color:var(--muted);" id="chart-subtitle">Paid transactions over time</p>
          </div>
          <div class="flex items-center gap-1" style="background:var(--bg);border:1px solid var(--border);border-radius:8px;padding:3px;">
            <button class="chart-tab active" id="tab-monthly" onclick="setChartGroup('monthly')">Monthly</button>
            <button class="chart-tab" id="tab-weekly" onclick="setChartGroup('weekly')">Weekly</button>
            <button class="chart-tab" id="tab-daily" onclick="setChartGroup('daily')">Daily</button>
          </div>
        </div>
        <div style="position:relative;height:220px;"><canvas id="revenue-chart"></canvas></div>
        <div class="flex gap-6 mt-4 pt-4" style="border-top:1px solid var(--border);">
          <div><div class="text-xs" style="color:var(--muted);">Peak Period</div><div class="font-semibold text-sm mono mt-0.5" id="chart-peak">—</div></div>
          <div><div class="text-xs" style="color:var(--muted);">Avg per Period</div><div class="font-semibold text-sm mono mt-0.5" id="chart-avg">—</div></div>
          <div><div class="text-xs" style="color:var(--muted);">Trend</div><div class="font-semibold text-sm mt-0.5" id="chart-trend">—</div></div>
        </div>
      </div>
    </div>

    <!-- VIEW: CHASE UNPAID -->
    <div id="view-actions" class="section-view">
      <div class="flex items-start justify-between mb-5 flex-wrap gap-3">
        <div>
          <h2 class="font-semibold text-base">Chase Unpaid Customers</h2>
          <p class="text-sm mt-1" style="color:var(--muted);">All pending &amp; failed payments. Contact them in bulk or one by one.</p>
        </div>
        <div class="flex gap-2 flex-wrap">
          <button class="btn btn-green text-xs" onclick="bulkSMS()">
            <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
            Bulk SMS All
          </button>
          <button class="btn btn-blue text-xs" onclick="bulkEmail()">
            <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
            Bulk Email All
          </button>
        </div>
      </div>
      <!-- Summary strip -->
      <div class="grid gap-3 mb-5" style="grid-template-columns:repeat(auto-fit,minmax(140px,1fr));">
        <div style="background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Pending Value</div>
          <div class="text-xl font-semibold mono mt-1" id="action-pending-val">—</div>
        </div>
        <div style="background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Failed Value</div>
          <div class="text-xl font-semibold mono mt-1" style="color:#f87171;" id="action-failed-val">—</div>
        </div>
        <div style="background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Customers to Chase</div>
          <div class="text-xl font-semibold mono mt-1" id="action-count">—</div>
        </div>
        <div style="background:var(--surface);border:1px solid var(--border);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">With Saved Contact</div>
          <div class="text-xl font-semibold mono mt-1" style="color:#34d399;" id="action-contact-count">—</div>
        </div>
      </div>
      <div style="background:var(--surface);border:1px solid var(--border);border-radius:12px;overflow:hidden;">
        <div style="overflow-x:auto;">
          <table class="data-table">
            <thead><tr>
              <th style="cursor:default;">Customer</th>
              <th style="cursor:default;">Amount</th>
              <th style="cursor:default;">Status</th>
              <th style="cursor:default;">Date</th>
              <th style="cursor:default;">Saved Contact</th>
              <th style="cursor:default;">Action</th>
            </tr></thead>
            <tbody id="action-table-body"></tbody>
          </table>
        </div>
        <div id="action-empty" class="py-12 text-center" style="color:var(--muted);display:none;">
          <div style="font-size:2rem;margin-bottom:8px;">🎉</div>
          <p class="text-sm font-medium">No unpaid customers — you're all caught up!</p>
        </div>
      </div>
    </div>

    <!-- VIEW: CUSTOMERS LTV -->
    <div id="view-customers" class="section-view">
      <div class="flex items-center justify-between mb-5 flex-wrap gap-2">
        <div>
          <h2 class="font-semibold text-base">Customer Lifetime Value</h2>
          <p class="text-sm mt-1" style="color:var(--muted);">Repeat buyers, VIPs, and your most valuable relationships.</p>
        </div>
        <div class="flex items-center gap-3 text-xs" style="color:var(--muted);">
          <span><span class="badge badge-vip">VIP</span> 3+ transactions</span>
          <span><span class="badge badge-paid">Repeat</span> 2+ transactions</span>
        </div>
      </div>
      <div style="background:var(--surface);border:1px solid var(--border);border-radius:12px;overflow:hidden;">
        <div style="overflow-x:auto;">
          <table class="data-table">
            <thead><tr>
              <th style="cursor:default;">Customer</th>
              <th style="cursor:default;">Total Paid</th>
              <th style="cursor:default;">Visits</th>
              <th style="cursor:default;">Pending</th>
              <th style="cursor:default;">Last Seen</th>
              <th style="cursor:default;">Tier</th>
              <th style="cursor:default;">CRM</th>
            </tr></thead>
            <tbody id="ltv-table-body"></tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- VIEW: TRANSACTIONS -->
    <div id="view-transactions" class="section-view">
      <div class="flex flex-wrap items-center gap-3 mb-4">
        <div class="flex items-center gap-2 flex-1" style="min-width:200px;">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="#64748b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="flex-shrink:0;"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
          <input type="text" id="search-input" placeholder="Search customer or status…" class="form-input" style="background:var(--surface2);" oninput="applyFilters()" />
        </div>
        <div class="flex items-center gap-2 flex-wrap">
          <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#64748b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/></svg>
          <input type="date" id="date-from" onchange="applyFilters()" title="From date" />
          <span class="text-xs" style="color:var(--muted);">to</span>
          <input type="date" id="date-to" onchange="applyFilters()" title="To date" />
          <button class="btn btn-ghost text-xs" onclick="clearDates()">Clear</button>
        </div>
        <div class="text-xs mono" style="color:var(--muted);" id="row-count"></div>
      </div>
      <div style="background:var(--surface);border:1px solid var(--border);border-radius:12px;overflow:hidden;">
        <div style="overflow-x:auto;">
          <table class="data-table">
            <thead><tr>
              <th onclick="sortBy('date')">Date <span id="sort-date"></span></th>
              <th onclick="sortBy('name')">Customer Name <span id="sort-name"></span></th>
              <th onclick="sortBy('amount')">Amount <span id="sort-amount"></span></th>
              <th onclick="sortBy('status')">Status <span id="sort-status"></span></th>
              <th style="cursor:default;">CRM</th>
            </tr></thead>
            <tbody id="table-body"></tbody>
          </table>
        </div>
        <div id="empty-state" class="py-16 text-center" style="color:var(--muted);display:none;">
          <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#64748b" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" style="margin:0 auto 8px;display:block;"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/><line x1="8" y1="8" x2="14" y2="14"/><line x1="14" y1="8" x2="8" y2="14"/></svg>
          <p class="text-sm">No transactions match your filters.</p>
        </div>
      </div>
    </div>

    <!-- VIEW: FAMILIES -->
    <div id="view-families" class="section-view">
      <div class="flex items-start justify-between mb-4 flex-wrap gap-3">
        <div>
          <h2 class="font-semibold text-base">Family Accounts</h2>
          <p class="text-sm mt-1" style="color:var(--muted);">Grouped by parent — every student and their balance in one place. Click a family to expand.</p>
        </div>
        <span class="text-xs mono" style="color:var(--muted);" id="families-summary"></span>
      </div>
      <div class="flex items-center gap-2 mb-4" style="max-width:360px;">
        <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#64748b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="flex-shrink:0;"><circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/></svg>
        <input type="text" id="family-search" placeholder="Search family or student…" class="form-input" style="background:var(--surface2);" oninput="renderFamiliesView()" />
      </div>
      <div id="families-list"></div>
      <p class="text-xs mt-4" style="color:var(--muted);line-height:1.7;"><strong style="color:var(--amber);">How grouping works:</strong> If your CSV has a "Parent Name" or "Family" column it will be used directly. Otherwise students are grouped by matching last names. You can also override the family name for any customer inside their CRM panel using the notes field.</p>
    </div>

    <!-- VIEW: INVOICE QUEUE -->
    <div id="view-invoice" class="section-view">
      <div class="flex items-start justify-between mb-4 flex-wrap gap-3">
        <div>
          <h2 class="font-semibold text-base">Invoice Queue</h2>
          <p class="text-sm mt-1" style="color:var(--muted);">Your prioritised action list — know exactly where to start when you open the EMA portal.</p>
        </div>
        <div class="flex items-center gap-3 text-xs" style="color:var(--muted);">
          <span><span style="width:8px;height:8px;background:var(--amber);border-radius:50%;display:inline-block;margin-right:4px;"></span>Ready</span>
          <span><span style="width:8px;height:8px;background:var(--blue);border-radius:50%;display:inline-block;margin-right:4px;"></span>Submitted</span>
          <span><span style="width:8px;height:8px;background:var(--purple);border-radius:50%;display:inline-block;margin-right:4px;"></span>Processing</span>
        </div>
      </div>
      <div class="grid gap-3 mb-5" style="grid-template-columns:repeat(auto-fit,minmax(140px,1fr));">
        <div style="background:var(--surface);border:1px solid var(--border);border-top:2px solid var(--amber);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Ready to Invoice</div>
          <div class="text-xl font-semibold mono mt-1" id="iq-ready-count">—</div>
          <div class="text-xs mt-0.5" style="color:var(--muted);" id="iq-ready-val"></div>
        </div>
        <div style="background:var(--surface);border:1px solid var(--border);border-top:2px solid var(--blue);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Submitted / Waiting</div>
          <div class="text-xl font-semibold mono mt-1" id="iq-submitted-count">—</div>
          <div class="text-xs mt-0.5" style="color:var(--muted);" id="iq-submitted-val"></div>
        </div>
        <div style="background:var(--surface);border:1px solid var(--border);border-top:2px solid var(--purple);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Processing</div>
          <div class="text-xl font-semibold mono mt-1" id="iq-processing-count">—</div>
          <div class="text-xs mt-0.5" style="color:var(--muted);" id="iq-processing-val"></div>
        </div>
        <div style="background:var(--surface);border:1px solid var(--border);border-top:2px solid var(--green);border-radius:10px;padding:14px 18px;">
          <div class="text-xs" style="color:var(--muted);">Completed</div>
          <div class="text-xl font-semibold mono mt-1" id="iq-done-count">—</div>
          <div class="text-xs mt-0.5" style="color:var(--muted);" id="iq-done-val"></div>
        </div>
      </div>
      <div style="background:rgba(245,158,11,0.06);border:1px solid rgba(245,158,11,0.15);border-radius:10px;padding:12px 16px;margin-bottom:16px;" class="flex items-start gap-3">
        <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="#f59e0b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="flex-shrink:0;margin-top:1px;"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>
        <p class="text-xs" style="color:var(--muted);line-height:1.7;"><strong style="color:var(--text);">How to use:</strong> Open your EMA portal alongside this list. Start at the top — <span style="color:var(--amber);">Ready to Invoice</span> rows need you to raise an invoice now. <span style="color:#93c5fd;">Submitted</span> rows just need a confirmation check. <span style="color:#c4b5fd;">Processing</span> rows are in the system — monitor for stalls. Tick off each one as you go.</p>
      </div>
      <div id="invoice-queue-list"></div>
    </div>

  </section>
</main>

<!-- CRM Overlay + Panel -->
<div id="crm-overlay" onclick="closeCRM()"></div>
<aside id="crm-panel">
  <div class="p-5 flex flex-col gap-5 flex-1">
    <div class="flex items-start justify-between">
      <div>
        <h2 class="font-semibold text-base" id="crm-name">Customer</h2>
        <p class="text-xs mt-0.5" style="color:var(--muted);" id="crm-meta"></p>
      </div>
      <button onclick="closeCRM()" class="btn btn-ghost" style="padding:6px;">
        <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/></svg>
      </button>
    </div>
    <div style="background:var(--bg);border:1px solid var(--border);border-radius:10px;padding:14px;" id="crm-tx-info"></div>
    <div id="crm-ltv-info" style="display:none;background:rgba(139,92,246,0.08);border:1px solid rgba(139,92,246,0.2);border-radius:10px;padding:14px;">
      <div class="text-xs uppercase tracking-wider mb-2" style="color:#a78bfa;">Customer History</div>
      <div class="flex gap-5" id="crm-ltv-stats"></div>
    </div>
    <div>
      <label class="text-xs uppercase tracking-wider block mb-1.5" style="color:var(--muted);">Phone Number</label>
      <input type="tel" id="crm-phone" class="form-input" placeholder="+1 (555) 000-0000" />
    </div>
    <div>
      <label class="text-xs uppercase tracking-wider block mb-1.5" style="color:var(--muted);">Email Address</label>
      <input type="email" id="crm-email" class="form-input" placeholder="customer@example.com" />
    </div>
    <div>
      <label class="text-xs uppercase tracking-wider block mb-1.5" style="color:var(--muted);">Personal Notes</label>
      <textarea id="crm-notes" class="form-input" placeholder="Add notes about this customer…"></textarea>
    </div>
    <button class="btn btn-amber w-full justify-center" onclick="saveCRM()">
      <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M19 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11l5 5v11a2 2 0 0 1-2 2z"/><polyline points="17 21 17 13 7 13 7 21"/><polyline points="7 3 7 8 15 8"/></svg>
      Save to CRM
    </button>
    <div style="border-top:1px solid var(--border);padding-top:16px;">
      <p class="text-xs uppercase tracking-wider mb-3" style="color:var(--muted);">Quick Actions</p>
      <div class="flex gap-2">
        <button class="btn btn-green flex-1 justify-center text-xs" onclick="sendSMS()">
          <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>
          Send SMS
        </button>
        <button class="btn btn-blue flex-1 justify-center text-xs" onclick="sendEmail()">
          <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 4h16c1.1 0 2 .9 2 2v12c0 1.1-.9 2-2 2H4c-1.1 0-2-.9-2-2V6c0-1.1.9-2 2-2z"/><polyline points="22,6 12,13 2,6"/></svg>
          Email
        </button>
      </div>
    </div>
    <div id="crm-saved-badge" style="display:none;background:rgba(16,185,129,0.1);border:1px solid rgba(16,185,129,0.2);border-radius:8px;padding:10px 14px;" class="flex items-center gap-2">
      <svg xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#34d399" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="flex-shrink:0;"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/></svg>
      <span class="text-xs" style="color:#34d399;">CRM data saved for this customer</span>
    </div>
  </div>
</aside>

<!-- Print Report Modal -->
<div id="print-modal">
  <div id="print-modal-inner">
    <div class="p-5 flex items-center justify-between" style="border-bottom:1px solid var(--border);">
      <h2 class="font-semibold">Business Summary Report</h2>
      <div class="flex gap-2">
        <button class="btn btn-amber text-xs" onclick="printReport()">
          <svg xmlns="http://www.w3.org/2000/svg" width="13" height="13" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 6 2 18 2 18 9"/><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"/><rect x="6" y="14" width="12" height="8"/></svg>
          Print / Save PDF
        </button>
        <button class="btn btn-ghost text-xs" onclick="closeReport()">Close</button>
      </div>
    </div>
    <div id="report-content" class="p-6"></div>
  </div>
</div>

<div id="toast"></div>

<script>
// ─── State ────────────────────────────────────────────────────────────────────
let allRows=[], filteredRows=[];
let sortKey='date', sortDir=1;
let activeCRMKey=null, activeCRMName=null;
let revenueChart=null, chartGroup='monthly';
let currentView='overview';

// ─── Upload / CSV ─────────────────────────────────────────────────────────────
const dropZone=document.getElementById('drop-zone');
dropZone.addEventListener('dragover',e=>{e.preventDefault();dropZone.classList.add('drag-over');});
dropZone.addEventListener('dragleave',()=>dropZone.classList.remove('drag-over'));
dropZone.addEventListener('drop',e=>{
  e.preventDefault();dropZone.classList.remove('drag-over');
  const f=e.dataTransfer.files[0];
  if(f&&f.name.endsWith('.csv'))parseCSV(f);else showToast('Please drop a valid CSV file.');
});
function handleFileInput(e){const f=e.target.files[0];if(f)parseCSV(f);}
function parseCSV(file){
  document.getElementById('file-name-label').textContent=file.name;
  document.getElementById('file-name-label').style.display='inline';
  Papa.parse(file,{header:true,skipEmptyLines:true,complete:r=>initDashboard(r.data,r.meta.fields)});
}
function normalizeRow(row,fields){
  const find=(...keys)=>{
    for(const k of keys){const m=fields.find(f=>f.toLowerCase().replace(/[\s_-]/g,'').includes(k));if(m)return row[m]||'';}
    return '';
  };
  const dateRaw=find('date','time','created');
  const name=find('customername','studentname','student','name','client','fullname')||'Unknown';
  const parentName=find('parentname','parent','guardian','family','accountholder')||'';
  const amtRaw=find('amount','total','price','sum','revenue','payment');
  const status=find('status','state');
  const amount=parseFloat(amtRaw.replace(/[^0-9.\-]/g,''))||0;
  let dateObj=null;
  try{const n=dateRaw.trim().replace(/\//g,'-');dateObj=new Date(n.includes('T')?n:n+'T12:00:00');if(isNaN(dateObj))dateObj=null;}catch{}
  return{dateRaw,dateObj,name,parentName,amount,status:status||'',raw:row};
}
function initDashboard(data,fields){
  allRows=data.map(r=>normalizeRow(r,fields||Object.keys(r)));
  persistData();
  showDashboard();
}

// ─── Demo Data ────────────────────────────────────────────────────────────────
function loadDemo(){
  // Family-aware demo: parent + student + realistic EMA invoice statuses
  const families=[
    {parent:'Linda Sanchez',   students:['Billy Sanchez','Elena Sanchez','Sofia Sanchez']},
    {parent:'James Okafor',    students:['James Okafor Jr.']},
    {parent:'Priya Patel',     students:['Aarav Patel','Meera Patel']},
    {parent:'Tom Nguyen',      students:['Tom Nguyen Jr.']},
    {parent:'Aisha Williams',  students:['Maya Williams','Zoe Williams']},
    {parent:'Derek Kim',       students:['Kai Kim']},
    {parent:'Sofia Rossi',     students:['Luca Rossi','Mia Rossi','Nico Rossi']},
    {parent:'Marcus Brown',    students:['Marcus Brown Jr.']},
    {parent:'Lily Chen',       students:['Oliver Chen','Emma Chen']},
    {parent:'Alex Torres',     students:['Alex Torres Jr.']},
  ];
  // EMA-realistic statuses including invoice-queue stages
  const statusPool=[
    'Paid','Paid','Paid',
    'Pending','Pending',
    'Submitted','Submitted',
    'Processing','Processing',
    'Approved','Approved',
    'Failed','Denied',
  ];
  const dates=[];
  for(let m=0;m<10;m++){for(const d of[3,8,14,21,27]){dates.push('2024-'+String(m+1).padStart(2,'0')+'-'+String(d).padStart(2,'0'));if(dates.length>=55)break;}if(dates.length>=55)break;}
  let i=0;
  allRows=[];
  families.forEach(fam=>{
    fam.students.forEach(student=>{
      const numOrders=Math.floor(Math.random()*4)+1;
      for(let o=0;o<numOrders&&i<dates.length;o++,i++){
        allRows.push({
          dateRaw:dates[i],
          dateObj:new Date(dates[i]+'T12:00:00'),
          name:student,
          parentName:fam.parent,
          amount:parseFloat((Math.random()*600+80).toFixed(2)),
          status:statusPool[Math.floor(Math.random()*statusPool.length)],
          raw:{}
        });
      }
    });
  });
  document.getElementById('file-name-label').textContent='demo-data.csv';
  document.getElementById('file-name-label').style.display='inline';
  persistData();
  showDashboard();
}
function showDashboard(){
  document.getElementById('upload-section').style.display='none';
  document.getElementById('dashboard-section').style.display='block';
  ['reset-btn','report-btn','export-btn'].forEach(id=>document.getElementById(id).style.display='inline-flex');
  document.getElementById('main-nav').style.display='flex';
  applyFilters();
}

// ─── Navigation ───────────────────────────────────────────────────────────────
function switchView(view){
  currentView=view;
  ['overview','actions','customers','transactions','families','invoice'].forEach(v=>{
    document.getElementById('view-'+v).classList.toggle('active',v===view);
    document.getElementById('nav-'+v).classList.toggle('active',v===view);
  });
  if(view==='actions')renderActionView();
  if(view==='customers')renderLTVView();
  if(view==='families')renderFamiliesView();
  if(view==='invoice')renderInvoiceQueue();
}

// ─── Helpers ──────────────────────────────────────────────────────────────────
const fmtAmt=n=>'$'+n.toLocaleString('en-US',{minimumFractionDigits:2,maximumFractionDigits:2});
const fmtShort=n=>'$'+(n>=1000?(n/1000).toFixed(1)+'k':n.toFixed(0));
const escHtml=s=>(s||'').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;');
const normKey=n=>n.toLowerCase().replace(/\s+/g,'_');
const hasCRM=name=>!!localStorage.getItem('crm_'+normKey(name));
const getCRM=name=>JSON.parse(localStorage.getItem('crm_'+normKey(name))||'{}');
const isPaid=r=>{const s=r.status.toLowerCase();return s.includes('paid')&&!s.includes('unpaid');};
const isPending=r=>r.status.toLowerCase().includes('pending');
const isFailed=r=>{const s=r.status.toLowerCase();return s.includes('fail')||s.includes('declined')||s.includes('cancel');};
const isUnpaid=r=>isPending(r)||isFailed(r);
function badgeHtml(status){
  const s=(status||'').toLowerCase();
  let cls='badge-default';
  if(s.includes('paid')&&!s.includes('unpaid'))cls='badge-paid';
  else if(s.includes('pending'))cls='badge-pending';
  else if(s.includes('fail')||s.includes('declined'))cls='badge-failed';
  return`<span class="badge ${cls}">${escHtml(status)||'—'}</span>`;
}
const crmDot=name=>hasCRM(name)
  ?'<span style="width:6px;height:6px;background:#34d399;border-radius:50%;display:inline-block;" title="CRM saved"></span>'
  :'<span style="width:6px;height:6px;background:var(--border);border-radius:50%;display:inline-block;"></span>';

// ─── Filters & Sort ───────────────────────────────────────────────────────────
function applyFilters(){
  const q=(document.getElementById('search-input').value||'').toLowerCase();
  const from=document.getElementById('date-from').value;
  const to=document.getElementById('date-to').value;
  filteredRows=allRows.filter(r=>{
    const mQ=!q||r.name.toLowerCase().includes(q)||r.status.toLowerCase().includes(q)||r.dateRaw.toLowerCase().includes(q);
    const mF=!from||(r.dateObj&&r.dateObj>=new Date(from));
    const mT=!to||(r.dateObj&&r.dateObj<=new Date(to+'T23:59:59'));
    return mQ&&mF&&mT;
  });
  sortRows();renderTable();renderStats();renderPipeline();renderChart();updateActionBadge();updateInvoiceBadge();
  if(currentView==='actions')renderActionView();
  if(currentView==='customers')renderLTVView();
  if(currentView==='families')renderFamiliesView();
  if(currentView==='invoice')renderInvoiceQueue();
}
function clearDates(){document.getElementById('date-from').value='';document.getElementById('date-to').value='';applyFilters();}
function sortBy(key){if(sortKey===key)sortDir*=-1;else{sortKey=key;sortDir=1;}sortRows();renderTable();updateSortIndicators();}
function sortRows(){
  filteredRows.sort((a,b)=>{
    let av,bv;
    if(sortKey==='date'){av=a.dateObj||0;bv=b.dateObj||0;}
    else if(sortKey==='amount'){av=a.amount;bv=b.amount;}
    else if(sortKey==='name'){av=a.name;bv=b.name;}
    else{av=a.status;bv=b.status;}
    return av<bv?-1*sortDir:av>bv?sortDir:0;
  });
}
function updateSortIndicators(){
  ['date','name','amount','status'].forEach(k=>{
    const el=document.getElementById('sort-'+k);
    if(el)el.textContent=sortKey===k?(sortDir===1?' ↑':' ↓'):'';
  });
}

// ─── Stats ────────────────────────────────────────────────────────────────────
function renderStats(){
  const paid=filteredRows.filter(isPaid);
  const pending=filteredRows.filter(isPending);
  const failed=filteredRows.filter(isFailed);
  const rev=paid.reduce((s,r)=>s+r.amount,0);
  const pVal=pending.reduce((s,r)=>s+r.amount,0);
  const fVal=failed.reduce((s,r)=>s+r.amount,0);
  const uniq=new Set(filteredRows.map(r=>r.name)).size;
  document.getElementById('stat-revenue').textContent=fmtAmt(rev);
  document.getElementById('stat-revenue-sub').textContent=`${paid.length} paid transaction${paid.length!==1?'s':''}`;
  document.getElementById('stat-customers').textContent=uniq.toLocaleString();
  document.getElementById('stat-customers-sub').textContent=`${filteredRows.length} total transactions`;
  document.getElementById('stat-pending').textContent=fmtAmt(pVal);
  document.getElementById('stat-pending-sub').textContent=`${pending.length} awaiting payment`;
  document.getElementById('stat-failed').textContent=fmtAmt(fVal);
  document.getElementById('stat-failed-sub').textContent=`${failed.length} failed / at risk`;
}

// ─── Pipeline ─────────────────────────────────────────────────────────────────
function renderPipeline(){
  const total=filteredRows.reduce((s,r)=>s+r.amount,0);
  const stages=[
    {label:'Paid',filter:isPaid,color:'#10b981'},
    {label:'Pending',filter:isPending,color:'#f59e0b'},
    {label:'Failed',filter:isFailed,color:'#ef4444'},
    {label:'Other',filter:r=>!isPaid(r)&&!isPending(r)&&!isFailed(r),color:'#64748b'},
  ];
  document.getElementById('pipeline-total').textContent=`${filteredRows.length} transactions · ${fmtAmt(total)} total`;
  document.getElementById('pipeline-bars').innerHTML=stages.map(st=>{
    const rows=filteredRows.filter(st.filter);
    const val=rows.reduce((s,r)=>s+r.amount,0);
    const pct=total>0?(val/total*100).toFixed(1):0;
    return`<div style="background:var(--surface2);border:1px solid var(--border);border-radius:10px;padding:14px 16px;">
      <div class="flex items-center justify-between mb-1">
        <span class="text-xs font-medium">${st.label}</span>
        <span class="text-xs mono" style="color:var(--muted);">${pct}%</span>
      </div>
      <div class="text-base font-semibold mono">${fmtAmt(val)}</div>
      <div class="text-xs mt-1" style="color:var(--muted);">${rows.length} transaction${rows.length!==1?'s':''}</div>
      <div class="pipeline-bar"><div class="pipeline-fill" style="width:${pct}%;background:${st.color};"></div></div>
    </div>`;
  }).join('');
}

// ─── Chart ────────────────────────────────────────────────────────────────────
function setChartGroup(g){
  chartGroup=g;
  ['monthly','weekly','daily'].forEach(x=>document.getElementById('tab-'+x).classList.toggle('active',x===g));
  renderChart();
}
function getPeriodKey(d,g){
  if(!d)return null;
  if(g==='monthly')return d.toLocaleDateString('en-US',{year:'numeric',month:'short'});
  if(g==='weekly'){const x=new Date(d);const day=x.getDay()||7;x.setDate(x.getDate()-day+1);return x.toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'});}
  return d.toLocaleDateString('en-US',{month:'short',day:'numeric'});
}
function renderChart(){
  const canvas=document.getElementById('revenue-chart');if(!canvas)return;
  const paid=filteredRows.filter(r=>isPaid(r)&&r.dateObj);
  const map=new Map();
  [...paid].sort((a,b)=>a.dateObj-b.dateObj).forEach(r=>{const k=getPeriodKey(r.dateObj,chartGroup);if(k)map.set(k,(map.get(k)||0)+r.amount);});
  const labels=[...map.keys()],values=[...map.values()];
  if(values.length>0){
    const mx=Math.max(...values),mi=values.indexOf(mx),avg=values.reduce((s,v)=>s+v,0)/values.length;
    document.getElementById('chart-peak').textContent=labels[mi]+' ('+fmtShort(mx)+')';
    document.getElementById('chart-avg').textContent=fmtAmt(avg);
    if(values.length>=2){
      const mid=Math.floor(values.length/2)||1;
      const f=values.slice(0,mid).reduce((s,v)=>s+v,0)/mid;
      const s=values.slice(mid).reduce((s,v)=>s+v,0)/(values.length-mid);
      const p=f>0?((s-f)/f*100).toFixed(1):0;
      document.getElementById('chart-trend').innerHTML=p>0?`<span style="color:#34d399;">↑ ${p}%</span>`:p<0?`<span style="color:#f87171;">↓ ${Math.abs(p)}%</span>`:`<span style="color:var(--muted);">— Flat</span>`;
    }
  }else['chart-peak','chart-avg','chart-trend'].forEach(id=>document.getElementById(id).textContent='—');
  document.getElementById('chart-subtitle').textContent=`${paid.length} paid transaction${paid.length!==1?'s':''} · grouped by ${chartGroup==='monthly'?'month':chartGroup==='weekly'?'week':'day'}`;
  const ctx=canvas.getContext('2d');
  const grad=ctx.createLinearGradient(0,0,0,220);
  grad.addColorStop(0,'rgba(245,158,11,0.25)');grad.addColorStop(1,'rgba(245,158,11,0.01)');
  if(revenueChart){revenueChart.data.labels=labels;revenueChart.data.datasets[0].data=values;revenueChart.data.datasets[0].backgroundColor=grad;revenueChart.update('active');return;}
  revenueChart=new Chart(ctx,{
    type:'line',
    data:{labels,datasets:[{label:'Revenue',data:values,borderColor:'#f59e0b',borderWidth:2,backgroundColor:grad,fill:true,tension:0.4,pointBackgroundColor:'#f59e0b',pointBorderColor:'#0e1117',pointBorderWidth:2,pointRadius:values.length>60?0:4,pointHoverRadius:6}]},
    options:{responsive:true,maintainAspectRatio:false,interaction:{mode:'index',intersect:false},
      plugins:{legend:{display:false},tooltip:{backgroundColor:'#1e2535',borderColor:'#2a3249',borderWidth:1,titleColor:'#94a3b8',bodyColor:'#e2e8f0',titleFont:{family:'DM Sans',size:11},bodyFont:{family:'DM Mono',size:13},padding:10,callbacks:{label:c=>' '+fmtAmt(c.parsed.y)}}},
      scales:{x:{grid:{color:'rgba(42,50,73,0.6)'},ticks:{color:'#64748b',font:{family:'DM Mono',size:10},maxRotation:0,maxTicksLimit:10},border:{display:false}},y:{grid:{color:'rgba(42,50,73,0.6)'},ticks:{color:'#64748b',font:{family:'DM Mono',size:10},callback:v=>'$'+(v>=1000?(v/1000).toFixed(1)+'k':v)},border:{display:false}}}}
  });
}

// ─── Transactions Table ───────────────────────────────────────────────────────
function renderTable(){
  const tbody=document.getElementById('table-body'),empty=document.getElementById('empty-state');
  document.getElementById('row-count').textContent=`${filteredRows.length} transaction${filteredRows.length!==1?'s':''}`;
  if(filteredRows.length===0){tbody.innerHTML='';empty.style.display='block';return;}
  empty.style.display='none';
  tbody.innerHTML=filteredRows.map((r,i)=>{
    const ds=r.dateObj?r.dateObj.toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}):r.dateRaw||'—';
    return`<tr onclick="openCRM(${i})">
      <td class="mono text-xs" style="color:var(--muted);">${ds}</td>
      <td class="font-medium">${escHtml(r.name)}</td>
      <td class="mono font-medium">${fmtAmt(r.amount)}</td>
      <td>${badgeHtml(r.status)}</td>
      <td>${crmDot(r.name)}</td>
    </tr>`;
  }).join('');
}

// ─── Chase Unpaid View ────────────────────────────────────────────────────────
function updateActionBadge(){
  const n=allRows.filter(isUnpaid).length;
  const b=document.getElementById('nav-actions-badge');
  b.textContent=n;b.style.display=n>0?'inline':'none';
}
function renderActionView(){
  const unpaid=filteredRows.filter(isUnpaid);
  const pRows=unpaid.filter(isPending),fRows=unpaid.filter(isFailed);
  const uniqNames=[...new Set(unpaid.map(r=>r.name))];
  const withContact=uniqNames.filter(n=>{const c=getCRM(n);return c.phone||c.email;});
  document.getElementById('action-pending-val').textContent=fmtAmt(pRows.reduce((s,r)=>s+r.amount,0));
  document.getElementById('action-failed-val').textContent=fmtAmt(fRows.reduce((s,r)=>s+r.amount,0));
  document.getElementById('action-count').textContent=uniqNames.length;
  document.getElementById('action-contact-count').textContent=withContact.length+' / '+uniqNames.length;
  const tbody=document.getElementById('action-table-body'),empty=document.getElementById('action-empty');
  if(unpaid.length===0){tbody.innerHTML='';empty.style.display='block';return;}
  empty.style.display='none';
  // Latest row per customer
  const byCustomer=new Map();
  unpaid.forEach(r=>{const ex=byCustomer.get(r.name);if(!ex||(r.dateObj&&(!ex.dateObj||r.dateObj>ex.dateObj)))byCustomer.set(r.name,r);});
  tbody.innerHTML=[...byCustomer.entries()].map(([name,r])=>{
    const crm=getCRM(name);
    const ds=r.dateObj?r.dateObj.toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}):r.dateRaw||'—';
    const contactHtml=crm.phone||crm.email
      ?`<span style="color:#34d399;" class="text-xs mono">${escHtml(crm.phone||crm.email)}</span>`
      :`<span style="color:var(--muted);" class="text-xs">No contact</span>`;
    const safeN=encodeURIComponent(name);
    const actionHtml=crm.phone||crm.email
      ?`<div class="flex gap-1">${crm.phone?`<button class="btn btn-green text-xs" style="padding:4px 10px;" onclick="quickSMS('${safeN}')">SMS</button>`:''}${crm.email?`<button class="btn btn-blue text-xs" style="padding:4px 10px;" onclick="quickEmail('${safeN}')">Email</button>`:''}</div>`
      :`<button class="btn btn-ghost text-xs" style="padding:4px 10px;" onclick="openCRMByName('${safeN}')">Add contact</button>`;
    return`<tr>
      <td class="font-medium" style="cursor:pointer;" onclick="openCRMByName('${safeN}')">${escHtml(name)}</td>
      <td class="mono">${fmtAmt(r.amount)}</td>
      <td>${badgeHtml(r.status)}</td>
      <td class="mono text-xs" style="color:var(--muted);">${ds}</td>
      <td>${contactHtml}</td>
      <td>${actionHtml}</td>
    </tr>`;
  }).join('');
}
function bulkSMS(){
  const names=[...new Set(filteredRows.filter(isUnpaid).map(r=>r.name))].filter(n=>getCRM(n).phone);
  if(!names.length){showToast('No unpaid customers have a phone saved in CRM.');return;}
  window.location.href=`sms:${names.map(n=>getCRM(n).phone).join(',')}?body=${encodeURIComponent('Hi, this is a reminder about your outstanding EMA Step Up payment. Please get in touch.')}`;
}
function bulkEmail(){
  const names=[...new Set(filteredRows.filter(isUnpaid).map(r=>r.name))].filter(n=>getCRM(n).email);
  if(!names.length){showToast('No unpaid customers have an email saved in CRM.');return;}
  window.location.href=`mailto:${names.map(n=>getCRM(n).email).join(',')}?subject=${encodeURIComponent('Outstanding Payment Reminder — EMA Step Up')}&body=${encodeURIComponent('Hi,\n\nThis is a friendly reminder that you have an outstanding payment on your EMA Step Up account.\n\nPlease contact us at your earliest convenience.\n\nThank you.')}`;
}
function quickSMS(encodedName){
  const name=decodeURIComponent(encodedName);
  const crm=getCRM(name);if(!crm.phone)return;
  window.location.href=`sms:${crm.phone}?body=${encodeURIComponent(`Hi ${name.split(' ')[0]}, this is a reminder about your outstanding EMA Step Up payment. Please get in touch.`)}`;
}
function quickEmail(encodedName){
  const name=decodeURIComponent(encodedName);
  const crm=getCRM(name);if(!crm.email)return;
  window.location.href=`mailto:${crm.email}?subject=${encodeURIComponent('Payment Reminder — EMA Step Up')}&body=${encodeURIComponent(`Hi ${name.split(' ')[0]},\n\nThis is a friendly reminder about your outstanding payment on your EMA Step Up account.\n\nPlease get in touch at your earliest convenience.\n\nThank you.`)}`;
}

// ─── LTV View ─────────────────────────────────────────────────────────────────
function buildCustomerMap(rows){
  const map=new Map();
  rows.forEach(r=>{
    if(!map.has(r.name))map.set(r.name,{name:r.name,totalPaid:0,totalPending:0,txCount:0,lastDate:null});
    const c=map.get(r.name);c.txCount++;
    if(isPaid(r))c.totalPaid+=r.amount;
    if(isPending(r))c.totalPending+=r.amount;
    if(r.dateObj&&(!c.lastDate||r.dateObj>c.lastDate))c.lastDate=r.dateObj;
  });
  return map;
}
function renderLTVView(){
  const map=buildCustomerMap(filteredRows);
  const customers=[...map.values()].sort((a,b)=>b.totalPaid-a.totalPaid);
  document.getElementById('ltv-table-body').innerHTML=customers.map(c=>{
    const tier=c.txCount>=3?'<span class="badge badge-vip">⭐ VIP</span>':c.txCount>=2?'<span class="badge badge-paid">Repeat</span>':'<span class="badge badge-default">New</span>';
    const lastDs=c.lastDate?c.lastDate.toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}):'—';
    return`<tr onclick="openCRMByName('${encodeURIComponent(c.name)}')" style="cursor:pointer;">
      <td class="font-medium">${escHtml(c.name)}</td>
      <td class="mono font-semibold">${fmtAmt(c.totalPaid)}</td>
      <td class="mono">${c.txCount}</td>
      <td class="mono" style="color:${c.totalPending>0?'#fbbf24':'var(--muted)'};">${c.totalPending>0?fmtAmt(c.totalPending):'—'}</td>
      <td class="mono text-xs" style="color:var(--muted);">${lastDs}</td>
      <td>${tier}</td>
      <td>${crmDot(c.name)}</td>
    </tr>`;
  }).join('');
}

// ─── Invoice status helpers ───────────────────────────────────────────────────
// EMA uses: Submitted, Processing, Approved, Ordering, Service Order → stages before Paid
// We map these to queue categories:
const isInvoiceReady    =r=>{const s=r.status.toLowerCase();return s.includes('approved')||s.includes('service order')||s.includes('ordering');};
const isInvoiceSubmitted=r=>{const s=r.status.toLowerCase();return s.includes('submitted');};
const isInvoiceProcess  =r=>{const s=r.status.toLowerCase();return s.includes('processing')||s.includes('in process')||s.includes('in-process');};
const isInvoiceActive   =r=>isInvoiceReady(r)||isInvoiceSubmitted(r)||isInvoiceProcess(r);

function updateInvoiceBadge(){
  const n=allRows.filter(isInvoiceActive).length;
  const b=document.getElementById('nav-invoice-badge');
  if(!b)return;
  b.textContent=n;b.style.display=n>0?'inline':'none';
}

// ─── Family Grouping ──────────────────────────────────────────────────────────
function getFamilyKey(r){
  // Priority: explicit parentName field > last name match
  if(r.parentName&&r.parentName.trim()) return r.parentName.trim();
  const parts=r.name.trim().split(/\s+/);
  return parts.length>1?parts[parts.length-1]+' Family':r.name+' (Individual)';
}

function buildFamilyMap(rows){
  const map=new Map();
  rows.forEach(r=>{
    const key=getFamilyKey(r);
    if(!map.has(key))map.set(key,{familyKey:key,students:new Map(),totalPaid:0,totalPending:0,totalActive:0,allRows:[]});
    const fam=map.get(key);
    fam.allRows.push(r);
    if(!fam.students.has(r.name))fam.students.set(r.name,{name:r.name,rows:[],paid:0,pending:0,active:0});
    const st=fam.students.get(r.name);
    st.rows.push(r);
    if(isPaid(r)){fam.totalPaid+=r.amount;st.paid+=r.amount;}
    if(isPending(r)){fam.totalPending+=r.amount;st.pending+=r.amount;}
    if(isInvoiceActive(r)){fam.totalActive+=r.amount;st.active+=r.amount;}
  });
  return map;
}

function renderFamiliesView(){
  const q=(document.getElementById('family-search')||{}).value||'';
  const qLow=q.toLowerCase();
  const map=buildFamilyMap(filteredRows);
  let families=[...map.values()].sort((a,b)=>b.totalPaid-a.totalPaid);
  if(qLow) families=families.filter(f=>f.familyKey.toLowerCase().includes(qLow)||[...f.students.keys()].some(n=>n.toLowerCase().includes(qLow)));

  const totalFamilies=families.length;
  const multiStudent=families.filter(f=>f.students.size>1).length;
  document.getElementById('families-summary').textContent=`${totalFamilies} famil${totalFamilies!==1?'ies':'y'} · ${multiStudent} with multiple students`;

  if(families.length===0){
    document.getElementById('families-list').innerHTML=`<div class="py-12 text-center" style="color:var(--muted);"><p class="text-sm">No families match your search.</p></div>`;
    return;
  }

  document.getElementById('families-list').innerHTML=families.map(fam=>{
    const studentCount=fam.students.size;
    const hasPending=fam.totalPending>0;
    const hasActive=fam.totalActive>0;
    const crmSaved=hasCRM(fam.familyKey);

    // Student rows HTML
    const studentRowsHtml=[...fam.students.values()].map(st=>{
      // Latest status for this student
      const latestRow=st.rows.sort((a,b)=>(b.dateObj||0)-(a.dateObj||0))[0];
      const pendingStr=st.pending>0?`<span class="mono text-xs" style="color:#fbbf24;">+${fmtAmt(st.pending)} pending</span>`:'';
      const activeStr=st.active>0?`<span class="mono text-xs" style="color:#a78bfa;">+${fmtAmt(st.active)} queued</span>`:'';
      return`<div class="student-row">
        <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" viewBox="0 0 24 24" fill="none" stroke="#64748b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="flex-shrink:0;"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>
        <span class="flex-1 font-medium text-sm">${escHtml(st.name)}</span>
        <span class="mono text-xs" style="color:#34d399;">${fmtAmt(st.paid)} paid</span>
        ${pendingStr}${activeStr}
        <span>${badgeHtml(latestRow?latestRow.status:'')}</span>
        <button class="btn btn-ghost text-xs" style="padding:3px 10px;font-size:0.7rem;" onclick="openCRMByName('${encodeURIComponent(st.name)}')">CRM</button>
      </div>`;
    }).join('');

    return`<div class="family-row">
      <div class="family-header" onclick="toggleFamily(this)" id="fam-${encodeURIComponent(fam.familyKey)}">
        <div class="flex items-center gap-3 flex-1 min-w-0">
          <svg class="family-chevron" xmlns="http://www.w3.org/2000/svg" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="#64748b" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="9 18 15 12 9 6"/></svg>
          <div>
            <div class="font-semibold text-sm flex items-center gap-2">
              ${escHtml(fam.familyKey)}
              ${studentCount>1?`<span class="badge badge-default">${studentCount} students</span>`:''}
              ${crmSaved?'<span style="width:6px;height:6px;background:#34d399;border-radius:50%;display:inline-block;" title="CRM saved"></span>':''}
            </div>
            <div class="text-xs mt-0.5" style="color:var(--muted);">${[...fam.students.keys()].map(n=>n.split(' ')[0]).join(', ')}</div>
          </div>
        </div>
        <div class="flex items-center gap-4 flex-shrink-0">
          <div class="text-right">
            <div class="mono font-semibold text-sm">${fmtAmt(fam.totalPaid)}</div>
            <div class="text-xs" style="color:var(--muted);">paid</div>
          </div>
          ${hasPending?`<div class="text-right"><div class="mono font-semibold text-sm" style="color:#fbbf24;">${fmtAmt(fam.totalPending)}</div><div class="text-xs" style="color:var(--muted);">pending</div></div>`:''}
          ${hasActive?`<div class="text-right"><div class="mono font-semibold text-sm" style="color:#a78bfa;">${fmtAmt(fam.totalActive)}</div><div class="text-xs" style="color:var(--muted);">queued</div></div>`:''}
          <button class="btn btn-ghost text-xs" style="padding:4px 12px;" onclick="openCRMByName('${encodeURIComponent(fam.familyKey)}');event.stopPropagation();">CRM</button>
        </div>
      </div>
      <div class="family-students" style="display:none;">${studentRowsHtml}</div>
    </div>`;
  }).join('');
}

function toggleFamily(header){
  header.classList.toggle('expanded');
  const body=header.nextElementSibling;
  body.style.display=body.style.display==='none'?'block':'none';
}

// ─── Invoice Queue ─────────────────────────────────────────────────────────────
function renderInvoiceQueue(){
  const ready      =filteredRows.filter(isInvoiceReady);
  const submitted  =filteredRows.filter(isInvoiceSubmitted);
  const processing =filteredRows.filter(isInvoiceProcess);
  const done       =filteredRows.filter(isPaid);

  const sum=arr=>arr.reduce((s,r)=>s+r.amount,0);
  document.getElementById('iq-ready-count').textContent=ready.length;
  document.getElementById('iq-ready-val').textContent=ready.length?fmtAmt(sum(ready)):'Nothing here';
  document.getElementById('iq-submitted-count').textContent=submitted.length;
  document.getElementById('iq-submitted-val').textContent=submitted.length?fmtAmt(sum(submitted)):'Nothing here';
  document.getElementById('iq-processing-count').textContent=processing.length;
  document.getElementById('iq-processing-val').textContent=processing.length?fmtAmt(sum(processing)):'Nothing here';
  document.getElementById('iq-done-count').textContent=done.length;
  document.getElementById('iq-done-val').textContent=done.length?fmtAmt(sum(done)):'';

  const sections=[
    {label:'Ready to Invoice',sublabel:'Action required — raise invoice in EMA portal now',rows:ready,color:'var(--amber)',cls:'ready',priority:1},
    {label:'Submitted — Awaiting Confirmation',sublabel:'Check EMA portal to confirm receipt',rows:submitted,color:'var(--blue)',cls:'submitted',priority:2},
    {label:'Processing / In Progress',sublabel:'In the system — monitor for stalls',rows:processing,color:'var(--purple)',cls:'processing',priority:3},
  ];

  const container=document.getElementById('invoice-queue-list');
  if(ready.length+submitted.length+processing.length===0){
    container.innerHTML=`<div class="py-12 text-center" style="color:var(--muted);"><div style="font-size:2rem;margin-bottom:8px;">✅</div><p class="text-sm font-medium">Invoice queue is clear — nothing needs action right now.</p></div>`;
    return;
  }

  container.innerHTML=sections.filter(s=>s.rows.length>0).map(sec=>{
    const rowsHtml=sec.rows
      .sort((a,b)=>(b.dateObj||0)-(a.dateObj||0))
      .map(r=>{
        const ds=r.dateObj?r.dateObj.toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}):r.dateRaw||'—';
        const fam=getFamilyKey(r);
        const crm=getCRM(r.name);
        return`<div class="invoice-item ${sec.cls}">
          <span class="urgency-dot" style="background:${sec.color};"></span>
          <div style="flex:1;min-width:0;">
            <div class="font-medium text-sm">${escHtml(r.name)}</div>
            <div class="text-xs mt-0.5" style="color:var(--muted);">${escHtml(fam)} · ${ds}</div>
          </div>
          <div class="mono font-semibold text-sm">${fmtAmt(r.amount)}</div>
          ${badgeHtml(r.status)}
          <div class="flex gap-1.5">
            ${crm.phone?`<button class="btn btn-green text-xs" style="padding:3px 10px;" onclick="quickSMS('${encodeURIComponent(r.name)}')">SMS</button>`:''}
            ${crm.email?`<button class="btn btn-blue text-xs" style="padding:3px 10px;" onclick="quickEmail('${encodeURIComponent(r.name)}')">Email</button>`:''}
            <button class="btn btn-ghost text-xs" style="padding:3px 10px;" onclick="openCRMByName('${encodeURIComponent(r.name)}')">CRM</button>
          </div>
        </div>`;
      }).join('');

    return`<div class="mb-5">
      <div class="flex items-center gap-2 mb-2">
        <span style="width:10px;height:10px;background:${sec.color};border-radius:50%;display:inline-block;flex-shrink:0;"></span>
        <div>
          <span class="font-semibold text-sm">${sec.label}</span>
          <span class="text-xs ml-2" style="color:var(--muted);">${sec.rows.length} order${sec.rows.length!==1?'s':''} · ${fmtAmt(sum(sec.rows))}</span>
        </div>
      </div>
      <p class="text-xs mb-3 ml-4" style="color:var(--muted);">${sec.sublabel}</p>
      ${rowsHtml}
    </div>`;
  }).join('');
}

// ─── CRM Panel ────────────────────────────────────────────────────────────────
function populateCRMPanel(r,headingMeta){
  activeCRMKey='crm_'+normKey(r.name);activeCRMName=r.name;
  document.getElementById('crm-name').textContent=r.name;
  document.getElementById('crm-meta').textContent=headingMeta||'';
  document.getElementById('crm-tx-info').innerHTML=`
    <div class="flex justify-between items-center">
      <div><div class="text-xs" style="color:var(--muted);">Transaction Amount</div><div class="mono font-semibold text-lg">${fmtAmt(r.amount)}</div></div>
      <div class="text-right"><div class="text-xs" style="color:var(--muted);">Status</div><div class="mt-1">${badgeHtml(r.status)}</div></div>
    </div>`;
  const custRows=allRows.filter(x=>x.name===r.name);
  const ltvEl=document.getElementById('crm-ltv-info');
  if(custRows.length>1){
    const tp=custRows.filter(isPaid).reduce((s,x)=>s+x.amount,0);
    document.getElementById('crm-ltv-stats').innerHTML=`
      <div><div class="text-xs" style="color:#a78bfa;">Lifetime Value</div><div class="mono font-semibold text-sm mt-0.5">${fmtAmt(tp)}</div></div>
      <div><div class="text-xs" style="color:#a78bfa;">Total Visits</div><div class="mono font-semibold text-sm mt-0.5">${custRows.length}</div></div>
      <div><div class="text-xs" style="color:#a78bfa;">Tier</div><div class="font-semibold text-sm mt-0.5">${custRows.length>=3?'⭐ VIP':'Repeat'}</div></div>`;
    ltvEl.style.display='block';
  }else{ltvEl.style.display='none';}
  const saved=getCRM(r.name);
  document.getElementById('crm-phone').value=saved.phone||'';
  document.getElementById('crm-email').value=saved.email||'';
  document.getElementById('crm-notes').value=saved.notes||'';
  document.getElementById('crm-saved-badge').style.display=(saved.phone||saved.email||saved.notes)?'flex':'none';
  document.getElementById('crm-panel').classList.add('open');
  document.getElementById('crm-overlay').classList.add('open');
}
function openCRM(idx){
  const r=filteredRows[idx];if(!r)return;
  const ds=r.dateObj?r.dateObj.toLocaleDateString('en-US',{month:'long',day:'numeric',year:'numeric'}):r.dateRaw;
  populateCRMPanel(r,ds);
}
function openCRMByName(encodedName){
  const name=decodeURIComponent(encodedName);
  const r=allRows.filter(x=>x.name===name).sort((a,b)=>(b.dateObj||0)-(a.dateObj||0))[0];
  if(!r)return;
  const ds=r.dateObj?'Last transaction: '+r.dateObj.toLocaleDateString('en-US',{month:'long',day:'numeric',year:'numeric'}):'';
  populateCRMPanel(r,ds);
}
function closeCRM(){document.getElementById('crm-panel').classList.remove('open');document.getElementById('crm-overlay').classList.remove('open');activeCRMKey=null;activeCRMName=null;}
function saveCRM(){
  if(!activeCRMKey)return;
  localStorage.setItem(activeCRMKey,JSON.stringify({phone:document.getElementById('crm-phone').value.trim(),email:document.getElementById('crm-email').value.trim(),notes:document.getElementById('crm-notes').value.trim()}));
  document.getElementById('crm-saved-badge').style.display='flex';
  showToast('CRM data saved ✓');
  renderTable();
  if(currentView==='actions')renderActionView();
  if(currentView==='customers')renderLTVView();
}
function sendSMS(){
  const p=document.getElementById('crm-phone').value.trim(),n=document.getElementById('crm-name').textContent;
  if(!p){showToast('Add a phone number first.');return;}
  window.location.href=`sms:${p}?body=${encodeURIComponent(`Hi ${n.split(' ')[0]}, this is a message regarding your EMA Step Up account.`)}`;
}
function sendEmail(){
  const e=document.getElementById('crm-email').value.trim(),n=document.getElementById('crm-name').textContent;
  if(!e){showToast('Add an email address first.');return;}
  window.location.href=`mailto:${e}?subject=${encodeURIComponent('Your EMA Step Up Account — '+n)}&body=${encodeURIComponent(`Hi ${n.split(' ')[0]},\n\nI'm reaching out regarding your EMA Step Up account.\n\nBest regards`)}`;
}

// ─── Export CSV ───────────────────────────────────────────────────────────────
function exportCSV(){
  if(!filteredRows.length){showToast('No data to export.');return;}
  const rows=[['Date','Customer Name','Amount','Status']];
  filteredRows.forEach(r=>{
    const ds=r.dateObj?r.dateObj.toLocaleDateString('en-US'):r.dateRaw;
    rows.push([ds,r.name,r.amount.toFixed(2),r.status]);
  });
  const csv=rows.map(r=>r.map(v=>`"${String(v).replace(/"/g,'""')}"`).join(',')).join('\n');
  const a=document.createElement('a');
  a.href=URL.createObjectURL(new Blob([csv],{type:'text/csv'}));
  a.download=`ema-export-${new Date().toISOString().split('T')[0]}.csv`;
  a.click();
  showToast(`Exported ${filteredRows.length} rows ✓`);
}

// ─── Print Report ─────────────────────────────────────────────────────────────
function openReport(){
  const paid=filteredRows.filter(isPaid),pending=filteredRows.filter(isPending),failed=filteredRows.filter(isFailed);
  const rev=paid.reduce((s,r)=>s+r.amount,0),pVal=pending.reduce((s,r)=>s+r.amount,0),fVal=failed.reduce((s,r)=>s+r.amount,0);
  const uniq=new Set(filteredRows.map(r=>r.name)).size;
  const totalAll=rev+pVal+fVal;
  const map=buildCustomerMap(filteredRows);
  const top=[...map.values()].sort((a,b)=>b.totalPaid-a.totalPaid).slice(0,10);
  const from=document.getElementById('date-from').value,to=document.getElementById('date-to').value;
  const period=from&&to?`${from} – ${to}`:from?`From ${from}`:to?`To ${to}`:'All time';

  document.getElementById('report-content').innerHTML=`<div id="printable">
    <!-- Header -->
    <div style="display:flex;align-items:center;justify-content:space-between;margin-bottom:24px;padding-bottom:16px;border-bottom:2px solid #f59e0b;">
      <div>
        <div style="font-size:1.2rem;font-weight:600;">EMA Step Up — Business Summary</div>
        <div style="font-size:0.78rem;color:var(--muted);margin-top:3px;">Period: ${escHtml(period)} &nbsp;·&nbsp; Generated ${new Date().toLocaleDateString('en-US',{month:'long',day:'numeric',year:'numeric'})}</div>
      </div>
      <div style="background:#f59e0b;border-radius:8px;padding:6px 14px;font-size:0.72rem;font-weight:700;color:#0e1117;letter-spacing:0.05em;">VENDOR REPORT</div>
    </div>
    <!-- KPIs -->
    <div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(130px,1fr));gap:10px;margin-bottom:20px;">
      ${[
        {l:'Total Revenue',v:fmtAmt(rev),c:'#f59e0b'},
        {l:'Paid Transactions',v:paid.length,c:'#10b981'},
        {l:'Unique Customers',v:uniq,c:'#3b82f6'},
        {l:'Pending Value',v:fmtAmt(pVal),c:'#f59e0b'},
        {l:'Failed / At Risk',v:fmtAmt(fVal),c:'#ef4444'},
        {l:'Collection Rate',v:(totalAll>0?(rev/totalAll*100):0).toFixed(1)+'%',c:'#10b981'},
      ].map(k=>`<div style="background:var(--surface2);border:1px solid var(--border);border-radius:10px;padding:14px;border-top:2px solid ${k.c};">
        <div style="font-size:0.68rem;color:var(--muted);text-transform:uppercase;letter-spacing:0.08em;">${k.l}</div>
        <div style="font-size:1.05rem;font-weight:600;font-family:'DM Mono',monospace;margin-top:4px;">${k.v}</div>
      </div>`).join('')}
    </div>
    <!-- Pipeline -->
    <div style="background:var(--surface2);border:1px solid var(--border);border-radius:10px;padding:16px 20px;margin-bottom:16px;">
      <div style="font-size:0.72rem;font-weight:600;text-transform:uppercase;letter-spacing:0.08em;color:var(--muted);margin-bottom:12px;">Payment Status Breakdown</div>
      ${[{l:'Paid',v:rev,n:paid.length,c:'#10b981'},{l:'Pending',v:pVal,n:pending.length,c:'#f59e0b'},{l:'Failed',v:fVal,n:failed.length,c:'#ef4444'}].map(s=>{
        const pct=totalAll>0?(s.v/totalAll*100).toFixed(1):0;
        return`<div style="display:flex;align-items:center;gap:10px;margin-bottom:9px;">
          <div style="width:60px;font-size:0.8rem;">${s.l}</div>
          <div style="flex:1;background:var(--border);border-radius:3px;height:6px;overflow:hidden;"><div style="width:${pct}%;background:${s.c};height:100%;border-radius:3px;"></div></div>
          <div style="font-family:'DM Mono',monospace;font-size:0.78rem;width:90px;text-align:right;">${fmtAmt(s.v)}</div>
          <div style="font-size:0.72rem;color:var(--muted);width:50px;">${s.n} txns</div>
          <div style="font-size:0.72rem;color:var(--muted);width:36px;">${pct}%</div>
        </div>`;
      }).join('')}
    </div>
    <!-- Top customers -->
    <div style="background:var(--surface2);border:1px solid var(--border);border-radius:10px;padding:16px 20px;margin-bottom:16px;">
      <div style="font-size:0.72rem;font-weight:600;text-transform:uppercase;letter-spacing:0.08em;color:var(--muted);margin-bottom:12px;">Top Customers by Revenue</div>
      <table style="width:100%;border-collapse:collapse;font-size:0.78rem;">
        <thead><tr style="border-bottom:1px solid var(--border);">
          <th style="text-align:left;padding:5px 8px;color:var(--muted);font-weight:500;">#</th>
          <th style="text-align:left;padding:5px 8px;color:var(--muted);font-weight:500;">Customer</th>
          <th style="text-align:right;padding:5px 8px;color:var(--muted);font-weight:500;">Revenue</th>
          <th style="text-align:right;padding:5px 8px;color:var(--muted);font-weight:500;">Visits</th>
          <th style="text-align:right;padding:5px 8px;color:var(--muted);font-weight:500;">Pending</th>
          <th style="text-align:center;padding:5px 8px;color:var(--muted);font-weight:500;">Tier</th>
        </tr></thead>
        <tbody>${top.map((c,i)=>`<tr style="border-bottom:1px solid var(--border);">
          <td style="padding:7px 8px;color:var(--muted);">${i+1}</td>
          <td style="padding:7px 8px;font-weight:500;">${escHtml(c.name)}</td>
          <td style="padding:7px 8px;text-align:right;font-family:'DM Mono',monospace;">${fmtAmt(c.totalPaid)}</td>
          <td style="padding:7px 8px;text-align:right;font-family:'DM Mono',monospace;">${c.txCount}</td>
          <td style="padding:7px 8px;text-align:right;font-family:'DM Mono',monospace;color:${c.totalPending>0?'#fbbf24':'var(--muted)'};">${c.totalPending>0?fmtAmt(c.totalPending):'—'}</td>
          <td style="padding:7px 8px;text-align:center;">${c.txCount>=3?'⭐ VIP':c.txCount>=2?'Repeat':'New'}</td>
        </tr>`).join('')}</tbody>
      </table>
    </div>
    <!-- Outstanding -->
    ${pending.length>0?`<div style="background:rgba(245,158,11,0.06);border:1px solid rgba(245,158,11,0.2);border-radius:10px;padding:16px 20px;">
      <div style="font-size:0.72rem;font-weight:600;text-transform:uppercase;letter-spacing:0.08em;color:#fbbf24;margin-bottom:12px;">Outstanding Payments to Chase (${pending.length})</div>
      <table style="width:100%;border-collapse:collapse;font-size:0.78rem;">
        <thead><tr style="border-bottom:1px solid rgba(245,158,11,0.2);">
          <th style="text-align:left;padding:5px 8px;color:var(--muted);font-weight:500;">Customer</th>
          <th style="text-align:right;padding:5px 8px;color:var(--muted);font-weight:500;">Amount</th>
          <th style="text-align:left;padding:5px 8px;color:var(--muted);font-weight:500;">Date</th>
          <th style="text-align:left;padding:5px 8px;color:var(--muted);font-weight:500;">Saved Contact</th>
        </tr></thead>
        <tbody>${pending.map(r=>{const crm=getCRM(r.name);const ds=r.dateObj?r.dateObj.toLocaleDateString('en-US',{month:'short',day:'numeric',year:'numeric'}):r.dateRaw||'—';return`<tr style="border-bottom:1px solid rgba(245,158,11,0.1);">
          <td style="padding:7px 8px;font-weight:500;">${escHtml(r.name)}</td>
          <td style="padding:7px 8px;text-align:right;font-family:'DM Mono',monospace;color:#fbbf24;">${fmtAmt(r.amount)}</td>
          <td style="padding:7px 8px;font-family:'DM Mono',monospace;color:var(--muted);">${ds}</td>
          <td style="padding:7px 8px;color:var(--muted);">${escHtml(crm.phone||crm.email||'—')}</td>
        </tr>`;}).join('')}</tbody>
      </table>
    </div>`:''}
  </div>`;
  document.getElementById('print-modal').classList.add('open');
}
function closeReport(){document.getElementById('print-modal').classList.remove('open');}
function printReport(){window.print();}

// ─── Persistence ─────────────────────────────────────────────────────────────
function persistData(){
  try{
    // Store rows without the live dateObj (it's not serializable cleanly); we re-hydrate on load
    const slim=allRows.map(r=>({dateRaw:r.dateRaw,name:r.name,parentName:r.parentName||'',amount:r.amount,status:r.status}));
    localStorage.setItem('ema_rows',JSON.stringify(slim));
    localStorage.setItem('ema_filename',document.getElementById('file-name-label').textContent||'');
  }catch(e){/* storage full or private mode */}
}

function rehydrateRows(slim){
  return slim.map(r=>{
    let dateObj=null;
    try{const n=(r.dateRaw||'').trim().replace(/\//g,'-');dateObj=new Date(n.includes('T')?n:n+'T12:00:00');if(isNaN(dateObj))dateObj=null;}catch{}
    return{...r,dateObj,raw:{}};
  });
}

// Auto-restore on page load
(function restoreOnLoad(){
  try{
    const saved=localStorage.getItem('ema_rows');
    if(!saved)return;
    const slim=JSON.parse(saved);
    if(!slim||!slim.length)return;
    allRows=rehydrateRows(slim);
    const fname=localStorage.getItem('ema_filename')||'saved-data.csv';
    document.getElementById('file-name-label').textContent=fname;
    document.getElementById('file-name-label').style.display='inline';
    showDashboard();
  }catch(e){/* corrupt data — ignore and show upload screen */}
})();

// ─── Reset ────────────────────────────────────────────────────────────────────
function resetDashboard(){
  allRows=[];filteredRows=[];
  localStorage.removeItem('ema_rows');
  localStorage.removeItem('ema_filename');
  if(revenueChart){revenueChart.destroy();revenueChart=null;}
  document.getElementById('upload-section').style.display='block';
  document.getElementById('dashboard-section').style.display='none';
  ['reset-btn','report-btn','export-btn'].forEach(id=>document.getElementById(id).style.display='none');
  document.getElementById('main-nav').style.display='none';
  document.getElementById('file-name-label').style.display='none';
  document.getElementById('csv-input').value='';
  document.getElementById('search-input').value='';
  document.getElementById('date-from').value='';
  document.getElementById('date-to').value='';
  closeCRM();closeReport();currentView='overview';
  ['overview','actions','customers','transactions','families','invoice'].forEach(v=>{
    document.getElementById('view-'+v).classList.toggle('active',v==='overview');
    document.getElementById('nav-'+v).classList.toggle('active',v==='overview');
  });
}

// ─── Toast ────────────────────────────────────────────────────────────────────
let toastTimer;
function showToast(msg){const t=document.getElementById('toast');t.textContent=msg;t.classList.add('show');clearTimeout(toastTimer);toastTimer=setTimeout(()=>t.classList.remove('show'),2800);}
</script>
</body>
</html>
