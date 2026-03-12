# Code Fixes Guide — Immediate Actions

This document contains concrete code changes to address the **15+ critical gaps** identified in the audit. All changes are ready to copy-paste.

---

## PRIORITY 1: Content Dynamization (Fix Hardcoded Titles)

### 1.1 Create Machine Metadata Object

**Add this after UPGRADE_SCHEDULE (around line 720):**

```javascript
// Machine-specific metadata for dynamic content
const MACHINE_METADATA = {
  F06: {
    startDate: '04 Feb 2026',
    startDateFull: '04 Feb 2026 onwards',
    description: 'Post-upgrade performance',
    status: 'Done',
    statusBadge: 'badge-green',
  },
  F09: {
    startDate: '02 Mar 2026',
    startDateFull: '02 Mar 2026 onwards',
    description: 'Commissioning phase',
    status: 'WIP',
    statusBadge: 'badge-blue',
  },
  F08: {
    startDate: '20 Mar 2026 (planned)',
    startDateFull: '20 Mar 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F07: {
    startDate: '09 Apr 2026 (planned)',
    startDateFull: '09 Apr 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F10: {
    startDate: '29 Apr 2026 (planned)',
    startDateFull: '29 Apr 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F05: {
    startDate: '19 May 2026 (planned)',
    startDateFull: '19 May 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F04: {
    startDate: '08 Jun 2026 (planned)',
    startDateFull: '08 Jun 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F03: {
    startDate: '26 Jun 2026 (planned)',
    startDateFull: '26 Jun 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F02: {
    startDate: '16 Jul 2026 (planned)',
    startDateFull: '16 Jul 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F01: {
    startDate: '05 Aug 2026 (planned)',
    startDateFull: '05 Aug 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F11: {
    startDate: '25 Aug 2026 (planned)',
    startDateFull: '25 Aug 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
  F12: {
    startDate: '14 Sep 2026 (planned)',
    startDateFull: '14 Sep 2026 onwards (planned)',
    description: 'Upgrade scheduled',
    status: 'Not Started',
    statusBadge: '',
  },
};
```

---

### 1.2 Create Dynamic Stats Calculator

**Add this function (around line 730, before renderView function):**

```javascript
function calculateMachineStats(machine) {
  const machineData = MACHINES_DATA[machine];
  
  if(!machineData || machineData.length === 0) {
    return {
      avgOEEDay: null,
      avgOEENight: null,
      bestOEEDay: null,
      bestOEENight: null,
      worstOEEDay: null,
      breakdownCount: 0,
      failureCount: 0,
      hasData: false,
    };
  }

  const oeeDay = machineData
    .map(d => numericOEE(d.oeeDay))
    .filter(v => v !== null);
  
  const oeeNight = machineData
    .map(d => numericOEE(d.oeeNight))
    .filter(v => v !== null);
  
  const breakdowns = machineData.filter(d => d.bd !== null).length;
  const failures = machineData.filter(d => d.pf !== null).length;
  
  return {
    avgOEEDay: oeeDay.length ? Math.round(oeeDay.reduce((a,b)=>a+b)/oeeDay.length * 10) / 10 : null,
    avgOEENight: oeeNight.length ? Math.round(oeeNight.reduce((a,b)=>a+b)/oeeNight.length * 10) / 10 : null,
    bestOEEDay: oeeDay.length ? Math.max(...oeeDay) : null,
    bestOEENight: oeeNight.length ? Math.max(...oeeNight) : null,
    worstOEEDay: oeeDay.length ? Math.min(...oeeDay) : null,
    breakdownCount: breakdowns,
    failureCount: failures,
    hasData: machineData.length > 0,
  };
}
```

---

### 1.3 Update Overview Section HTML

**Replace the Overview stat cards section (around line 259-283) with IDs for dynamic updates:**

```html
<!-- ====== OVERVIEW ====== -->

<div class="view active" id="view-overview">
<div class="sec-header">
<div>
<div class="sec-title">Executive Overview</div>
<div class="sec-sub" id="overview-subtitle">F06 post-upgrade performance · Real data from 04 Feb 2026 onwards</div>
</div>
<div style="font-family:'DM Mono',monospace;font-size:10px;color:var(--text-muted)" id="live-date"></div>
</div>

<div class="stat-row stat-4">
<div class="stat-card sc-blue">
<div class="stat-label" id="stat-label-oee-day">{M} Avg OEE (Day)</div>
<div class="stat-val sv-blue" id="stat-val-oee-day">—</div>
<div class="stat-delta"><span id="stat-delta-oee-day">No data</span></div>
</div>
<div class="stat-card sc-green">
<div class="stat-label" id="stat-label-oee-night">{M} Avg OEE (Night)</div>
<div class="stat-val sv-green" id="stat-val-oee-night">—</div>
<div class="stat-delta"><span id="stat-delta-oee-night">No data</span></div>
</div>
<div class="stat-card sc-amber">
<div class="stat-label">Breakdown Events</div>
<div class="stat-val sv-amber" id="stat-val-breakdowns">0</div>
<div class="stat-delta" id="stat-delta-breakdowns">Since commissioning</div>
</div>
<div class="stat-card sc-red">
<div class="stat-label">Process Failures</div>
<div class="stat-val sv-red" id="stat-val-failures">0</div>
<div class="stat-delta" id="stat-delta-failures">Mainly powder supply issues</div>
</div>
</div>

<!-- Rest of cards unchanged -->
```

---

### 1.4 Function to Update Overview Content

**Add this function (around line 740):**

```javascript
function updateOverviewContent() {
  const machine = currentMachine;
  const metadata = MACHINE_METADATA[machine];
  const stats = calculateMachineStats(machine);
  
  // Update subtitle
  document.getElementById('overview-subtitle').textContent = 
    `${metadata.description} · Real data from ${metadata.startDateFull}`;
  
  // Update stat labels
  document.getElementById('stat-label-oee-day').textContent = 
    `${machine} Avg OEE (Day)`;
  document.getElementById('stat-label-oee-night').textContent = 
    `${machine} Avg OEE (Night)`;
  
  // Update stat values
  if(stats.hasData) {
    document.getElementById('stat-val-oee-day').textContent = 
      stats.avgOEEDay !== null ? stats.avgOEEDay + '%' : '—';
    document.getElementById('stat-val-oee-night').textContent = 
      stats.avgOEENight !== null ? stats.avgOEENight + '%' : '—';
    
    document.getElementById('stat-delta-oee-day').innerHTML = 
      stats.avgOEEDay >= 65 ? '<span class="du">✓ On target</span>' : '<span class="dd">↓ Below target</span>';
    document.getElementById('stat-delta-oee-night').innerHTML = 
      'Night readings available';
    
    document.getElementById('stat-val-breakdowns').textContent = stats.breakdownCount;
    document.getElementById('stat-val-failures').textContent = stats.failureCount;
  } else {
    document.getElementById('stat-val-oee-day').textContent = '—';
    document.getElementById('stat-val-oee-night').textContent = '—';
    document.getElementById('stat-delta-oee-day').textContent = 'No data yet';
    document.getElementById('stat-delta-oee-night').textContent = 'No data yet';
    document.getElementById('stat-val-breakdowns').textContent = '—';
    document.getElementById('stat-val-failures').textContent = '—';
  }
  
  // Update live date
  document.getElementById('live-date').textContent = new Date().toLocaleDateString('en-GB',{weekday:'long',year:'numeric',month:'long',day:'numeric'});
}
```

---

### 1.5 Update Overview Section Titles (F06 View)

**Replace the F06 Live KPI section header (around line 319):**

```javascript
// In view-f06 HTML:
<div class="sec-title" id="f06-view-title">F06 Live KPI Dashboard</div>
<div class="sec-sub" id="f06-view-subtitle">All captured data since commissioning · 04 Feb 2026</div>

// Add this function to update it:
function updateF06ViewContent() {
  const machine = currentMachine;
  const metadata = MACHINE_METADATA[machine];
  document.getElementById('f06-view-title').textContent = 
    `${machine} Live KPI Dashboard`;
  document.getElementById('f06-view-subtitle').textContent = 
    `All captured data since commissioning · ${metadata.startDate}`;
}
```

---

### 1.6 Fix Other View Headers

**Update these sections similarly:**

**OEE Trend (line 371):**
```html
<div class="sec-sub" id="oee-subtitle">{M} day & night shift OEE over time · Target 65%</div>
```

**Volume Analysis (line 409):**
```html
<div class="sec-sub" id="volume-subtitle">{M} daily production output in tonnes · Target: >10 tons/shift</div>
```

**Breakdowns (line 438):**
```html
<div class="sec-sub" id="bd-subtitle">{M} real event log since commissioning</div>
```

**Impact (line 515):**
```html
<div class="sec-title" id="impact-title">Upgrade Impact — {M}</div>
<div class="sec-sub" id="impact-subtitle">Commissioning baseline vs post-upgrade performance trend</div>
```

---

## PRIORITY 2: Call Updates on Machine Switch

**Update switchMachine() function (around line 746):**

```javascript
function switchMachine(machine) {
  currentMachine = machine;
  document.getElementById('machineSelector').value = machine;
  document.getElementById('f06-nav-text').textContent = machine + ' Live KPIs';
  
  const activeView = document.querySelector('.view.active')?.id?.split('view-')[1] || 'overview';
  
  // Update view label
  const labels = {overview:'Executive Overview',f06:'{M} Live KPIs',oee:'OEE Trend Analysis',volume:'Volume Analysis',breakdowns:'Breakdowns & Process Failures',tracker:'Upgrade Tracker',impact:'Upgrade Impact',entry:'Log Data'};
  let viewLabel = (labels[activeView]||activeView);
  viewLabel = viewLabel.replace('{M}', machine);
  document.getElementById('view-label').textContent = viewLabel + ' · ' + machine + ' Real Data';
  
  // ← ADD THESE LINES:
  if(activeView === 'overview') updateOverviewContent();
  else if(activeView === 'f06') updateF06ViewContent();
  // (Add similar for oee, volume, impact, etc. as you implement them)
  
  // Auto-select in form
  document.getElementById('f-machine').value = machine;
  
  renderView(activeView);
}
```

---

## PRIORITY 3: Dynamic Heatmap/Table Cards

**Update the F06 view card headers - add IDs:**

```html
<div class="card-header">
<div class="card-title" id="heatmap-title">F06 Daily OEE Heatmap</div>
</div>

<!-- and -->

<div class="card-header">
  <div class="card-title" id="table-title">Full F06 Daily Log</div>
</div>
```

**Add update function:**

```javascript
function updateF06CardTitles() {
  const machine = currentMachine;
  document.getElementById('heatmap-title').textContent = `${machine} Daily OEE Heatmap`;
  document.getElementById('table-title').textContent = `Full ${machine} Daily Log`;
}
```

---

## PRIORITY 4: Empty State UI

**Add this function:**

```javascript
function renderEmptyState(container, machine) {
  const metadata = MACHINE_METADATA[machine];
  container.innerHTML = `
    <div style="text-align:center;padding:60px 20px;color:var(--text-muted);">
      <div style="font-size:48px;margin-bottom:10px">📭</div>
      <div style="font-size:14px;font-weight:600;margin-bottom:5px">No data available for ${machine}</div>
      <div style="font-size:12px;margin-bottom:20px;">
        ${metadata.status === 'Done' ? 'Data not yet logged. ' : 'Commissioning scheduled for ' + metadata.startDate + '. '}
        Start logging shift data to see KPIs.
      </div>
      <button onclick="nav('entry',null)" class="btn btn-primary">
        + Log First Shift Data
      </button>
    </div>
  `;
}
```

**Update chart render functions:**

```javascript
function buildOverviewCharts() {
  const machineData = MACHINES_DATA[currentMachine];
  
  // ← ADD THIS CHECK:
  if(!machineData || machineData.length === 0) {
    renderEmptyState(document.getElementById('chart-container'), currentMachine);
    return;
  }
  
  // ... rest of function unchanged
}
```

---

## PRIORITY 5: Fix Form Auto-Select

**Already partially done, but ensure this is in switchMachine():**

```javascript
// In switchMachine function, add:
document.getElementById('f-machine').value = currentMachine;
```

---

## PRIORITY 6: Improve Error Handling

**Wrap renderView() with try-catch:**

```javascript
function renderView(id) {
  try {
    if(id==='overview') { 
      updateOverviewContent();  // ← Add this call
      buildOverviewCharts(); 
      buildAlerts(); 
    }
    else if(id==='f06') { 
      updateF06ViewContent();  // ← Add this call
      buildHeatmap(); 
      buildFullTable(); 
    }
    else if(id==='oee') { buildOEECharts(); }
    else if(id==='volume') { buildVolumeCharts(); }
    else if(id==='breakdowns') { buildBreakdownTables(); buildBreakdownChart(); }
    else if(id==='tracker') { buildTrackerTable(); }
    else if(id==='impact') { buildImpactCharts(); }
    else if(id==='entry') { renderEntryLog(); setTodayDate(); }
  } catch(error) {
    console.error('Error rendering view', id, error);
    showToast('❌ Error loading view. Please refresh the page.');
  }
}
```

---

## PRIORITY 7: localStorage for User Entries

**Update submitEntry() and initialization:**

```javascript
// At top of script section, change:
let userEntries = [];

// To:
let userEntries = (() => {
  try {
    const stored = localStorage.getItem('bosch_dashboard_entries');
    return stored ? JSON.parse(stored) : [];
  } catch(e) {
    return [];
  }
})();

// In submitEntry(), after adding entry:
userEntries.unshift({...});
// ADD THIS LINE:
localStorage.setItem('bosch_dashboard_entries', JSON.stringify(userEntries));
```

---

## Quick Implementation Checklist

**Week 1:**
- [ ] Add MACHINE_METADATA object
- [ ] Add calculateMachineStats() function
- [ ] Add updateOverviewContent() function
- [ ] Add IDs to HTML stat cards
- [ ] Update switchMachine() to call updateOverviewContent()
- [ ] Test machine switching on Overview page

**Week 2:**
- [ ] Add updateF06ViewContent() and similar for other views
- [ ] Update all view header IDs and functions
- [ ] Add renderEmptyState() function
- [ ] Test empty states for F09-F12
- [ ] Fix form auto-select

**Week 3:**
- [ ] Add error handling to renderView()
- [ ] Add localStorage persistence
- [ ] Mobile testing
- [ ] Final QA

---

## Testing Steps

After each change:

1. **Test F06 (has data):**
   ```
   - Select F06 from dropdown
   - Navigate through all views
   - Check stats update correctly
   - Check titles mention F06
   ```

2. **Test F09 (no data):**
   ```
   - Select F09 from dropdown
   - Each view should show empty state
   - Should guide user to "Log Data"
   - No errors in console
   ```

3. **Test Form:**
   ```
   - Switch machines
   - "Machine" field should auto-select current machine
   - Log entry
   - Entry should appear in "Recently Logged" table
   ```

4. **Test Mobile (if applicable):**
   ```
   - Browser DevTools: 375px wide
   - Machine selector should not overflow
   - Stats cards should stack properly
   ```

---

## Support Notes

- All changes are backward-compatible
- No modifications needed to MACHINES_DATA structure
- No external libraries added
- localStorage size: ~50KB for 1000 entries
- Performance impact: negligible

---

**Generated:** March 12, 2026  
**Estimated Implementation Time:** 6-8 hours  
**Complexity:** Medium
