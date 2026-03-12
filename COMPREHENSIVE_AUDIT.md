# Bosch Dashboard — Comprehensive Technical Audit
**Date:** March 12, 2026 | **Status:** Version 1.0 with Multi-Machine Foundation

---

## EXECUTIVE SUMMARY

The dashboard is **functionally complete** for F06 with a scalable multi-machine architecture. However, there are **20+ critical gaps** preventing other machines from displaying properly. The system needs content dynamization and error handling before production use with additional machines.

**Priority Level:** 🔴 **HIGH** - Immediate fixes needed for machine-switching feature to work fully.

---

## 🔴 CRITICAL ISSUES (Must Fix)

### 1. **Hardcoded View Titles & Descriptions** 
**Impact:** When switching machines, all view page headers still say "F06"
**Locations:**
- Line 252: `F06 post-upgrade performance · Real data from 04 Feb 2026 onwards`
- Line 319: `F06 Live KPI Dashboard`
- Line 346: `F06 Daily OEE Heatmap`
- Line 355: `Full F06 Daily Log`
- Line 371: `F06 day & night shift OEE over time`
- Line 409: `F06 daily production output in tonnes`
- Line 515: `Upgrade Impact — F06`

**Fix Approach:**
```javascript
// Create metadata for each machine
const MACHINE_METADATA = {
  F06: {
    startDate: '04 Feb 2026',
    startDateFull: '04 Feb 2026 onwards',
    description: 'Post-upgrade performance',
    subtitle: 'Real data',
  },
  F09: {
    startDate: '02 Mar 2026',
    startDateFull: '02 Mar 2026 onwards',
    description: 'Commissioning phase',
    subtitle: 'Starting',
  },
  // ... etc
};
```

Then inject these into view headers dynamically when machine changes.

---

### 2. **Empty Data Arrays for F09-F12**
**Impact:** Switching to F09+ shows empty charts with no user guidance
**Current State:** 
```javascript
F09: [],
F08: [],
// ... all empty
```

**Problems:**
- Charts render but show no data
- No error message or "No data available" indicator
- Users don't know if data is missing or app is broken
- Heatmap will be empty table with just headers

**Fix Required:**
1. Add `dataAvailable` flag to MACHINES_DATA or check array length
2. Create "Empty State" UI component for all views
3. Show: `"No data available for F09 yet. Commissioning started 02 Mar 2026."`
4. Suggest action: `"Log your first shift data in the 'Log Data' view"`

---

### 3. **Hardcoded Stat Card Labels**
**Impact:** Stat cards say "F06 Avg OEE (Day)" even when viewing F08
**Locations:**
- Line 259: `F06 Avg OEE (Day)`
- Line 264: `F06 Avg OEE (Night)`
- Lines 268-276: `Breakdown Events`, `Process Failures` are generic but context says F06

**Current:**
```html
<div class="stat-label">F06 Avg OEE (Day)</div>
```

**Should Be:**
```javascript
const statsLabels = {
  avgOEEDay: `${currentMachine} Avg OEE (Day)`,
  avgOEENight: `${currentMachine} Avg OEE (Night)`,
  // ... etc
};
// Then in HTML: <div class="stat-label" id="stat-avg-oee-day"></div>
// And in JS: document.getElementById('stat-avg-oee-day').textContent = statsLabels.avgOEEDay;
```

---

### 4. **Hardcoded Alerts (buildAlerts function)**
**Impact:** Active Alerts always refer to F06 regardless of selected machine
**Current (Line 854-880):**
```javascript
const alerts = [
  {title:'Low OEE Days Detected', desc:'Multiple days below 50% — primarily driven by powder supply failures (upstream issue).'},
  {title:'F09 Upgrade In Progress', desc:'F09 upgrade started 02 Mar 2026...'},
  {title:'Sealing Event — 23 Feb', desc:'Leaking packs detected on 23 Feb 2026...'},
  {title:'F06 Commissioned', desc:'F06 achieved 1kg: 50bpm...'},
  {title:'OEE Trending Up', desc:'Recent readings: 75% and 90% on 11 Mar...'},
];
```

**Issues:**
- All alerts are static F06 data
- Should show alerts relevant to selected machine
- For empty machines, should show different messages

**Fix Required:**
```javascript
function buildAlerts() {
  const machineData = MACHINES_DATA[currentMachine];
  if(!machineData || machineData.length === 0) {
    // Show "No data yet" message
    return;
  }
  // Generate alerts from machineData dynamically
  // E.g., calculate low OEE % from actual data
}
```

---

### 5. **Hardcoded Impact Statistics & Charts**
**Impact:** "Upgrade Impact" page shows only F06 data even when F09 selected
**Issues:**
- Line 515: Title is hardcoded to `Upgrade Impact — F06`
- Lines 523-531: The 4 impact cards show F06-specific values:
  - Speed: "50 bpm ✓"
  - OEE: "~35%" early, "~65%+" now
  - BTB: "0 ✓"  
  - Sealing: "1 event"
- Lines 1033-1076: buildImpactCharts() has hardcoded weekly data:

```javascript
const weeks = ['W1 (Feb 11-15)','W2 (Feb 16-22)',...];
const weekData = [
  [76,73,20,24,12], // W1
  [28,46,37,29,24], // W2
  // ... static data
];
```

**Fix Required:**
- Make title dynamic: `Upgrade Impact — ${currentMachine}`
- Calculate impact stats from actual MACHINES_DATA
- For empty machines, show comparative metrics instead (planned vs actual)

---

### 6. **Hard-Coded Breakdown Data**
**Impact:** Breakdowns & Process Failures tab shows only F06 events
**Current:** Lines in buildBreakdownTables() reference fixed event data:
```javascript
const bdData = [
  {date:'2026-02-05',desc:'Temperature for Jaws not increasing',type:'Mechanical'},
  // ... 3 more hardcoded events
];
```

**Should Be:**
```javascript
function buildBreakdownTables() {
  const machineData = MACHINES_DATA[currentMachine];
  const bdData = machineData.filter(d => d.bd !== null).map(d => ({
    date: d.date,
    desc: d.bd,
    type: 'Mechanical'
  }));
  // ... render
}
```

---

## 🟡 MAJOR GAPS (Should Fix Soon)

### 7. **No Live-Date Updates on View Switch**
**Impact:** The `live-date` display shows when page loads but may not update properly
**Current:** Line 1145:
```javascript
document.getElementById('live-date').textContent = new Date().toLocaleDateString(...);
```
**Issue:** This runs once on page init, not on view change. Should update in main render loop.

---

### 8. **No Loading States or Error Handling**
**Impact:** If Chart.js fails or data is corrupted, users see blank pages with no indication
**Missing:**
- Try-catch blocks around renderView()
- Loading spinners during render
- Error messages for failed chart creation
- Fallback UI for missing data

**Suggested Pattern:**
```javascript
async function renderView(id) {
  try {
    showLoadingState(true);
    if(id==='overview') { buildOverviewCharts(); buildAlerts(); }
    else if(id==='f06') { buildHeatmap(); buildFullTable(); }
    // ...
    showLoadingState(false);
  } catch(error) {
    console.error('View render error:', error);
    showToast('❌ Error loading view: ' + error.message);
    showLoadingState(false);
  }
}
```

---

### 9. **User Entry Form Has No Machine Binding**
**Impact:** When user logs data and switches machines, form still shows previous machine
**Current:** Line 614+
```html
<select class="fi" id="f-machine">
  <option value="">Select...</option>
  <option>F06</option><option>F09</option>...
</select>
```

**Issue:** Form doesn't auto-set to currentMachine
**Fix:** 
```javascript
function switchMachine(machine) {
  currentMachine = machine;
  document.getElementById('f-machine').value = machine; // ← Add this
  // ...
}
```

---

### 10. **Data Persistence Lost on Refresh**
**Impact:** User-logged entries with `submitEntry()` are stored in `userEntries = []` (memory only)
**Current:** Line 710
```javascript
let userEntries = [];
```

**Problem:** Refreshing page loses all logged data
**Solutions:**
- Use localStorage: `localStorage.setItem('entries', JSON.stringify(userEntries))`
- Load on init: `userEntries = JSON.parse(localStorage.getItem('entries') || '[]')`
- Or: Server-sync via API

---

### 11. **No Empty State UI**
**Impact:** When viewing empty machines (F09-F12), user sees blank charts with no explanation
**Missing Components:**
- Empty state illustration or message
- Loading states while rendering
- "No data" indicators in heatmaps
- Helpful text: "Start logging data to see KPIs"

**Pattern Needed:**
```javascript
function renderEmptyState(container, machine) {
  container.innerHTML = `
    <div style="text-align:center;padding:60px 20px;color:var(--text-muted);">
      <div style="font-size:48px;margin-bottom:10px">📭</div>
      <div style="font-size:14px;font-weight:600;margin-bottom:5px">No data for ${machine} yet</div>
      <div style="font-size:12px;">Commissioning scheduled for ...  </div>
      <button onclick="nav('entry',null)" style="margin-top:15px" class="btn btn-primary">
        + Log First Shift Data
      </button>
    </div>
  `;
}
```

---

### 12. **Stat Values Are Hardcoded**
**Impact:** Stat card values (49.5%, 52.2%, 4 events, 11 failures) don't update when machine changes
**Current:** Lines 259-283
```html
<div class="stat-val sv-blue">49.5%</div>
<div class="stat-label">F06 Avg OEE (Day)</div>
<div class="stat-delta">Target: 65% | <span class="dd">↓ Below target</span></div>
```

**Fix Required:**
Calculate dynamically:
```javascript
function calculateMachineStats(machineData) {
  const oeeValues = machineData
    .map(d => numericOEE(d.oeeDay))
    .filter(v => v !== null);
  
  const avgOEEDay = oeeValues.length 
    ? Math.round(oeeValues.reduce((a,b)=>a+b)/oeeValues.length * 10) / 10
    : null;
  
  return { avgOEEDay, avgOEENight, breakdownCount, failureCount };
}

// Then in renderView:
const stats = calculateMachineStats(MACHINES_DATA[currentMachine]);
document.querySelector('.stat-val').textContent = stats.avgOEEDay + '%';
```

---

### 13. **OEE Trend Sub-text Hardcoded**
**Impact:** View subtitles like "F06 day & night shift OEE over time" don't include current machine
**Current:** Line 371
```html
<div class="sec-sub">F06 day & night shift OEE over time · Target 65%</div>
```

**Should Be:**
```html
<div class="sec-sub" id="oee-subtex">${currentMachine} day & night shift OEE over time · Target 65%</div>
```

---

### 14. **Mobile Responsiveness Issues**
**Impact:** Machine selector dropdown and stat cards may break on mobile
**Current State:**
- Topbar uses `gap:10px` which may be too tight
- Machine selector dropdown at inline style may overflow
- Stat cards with stat-4 might stack poorly

**Suggested Media Query:**
```css
@media(max-width:768px) {
  .stat-4 { grid-template-columns:repeat(2,1fr); }
  .topbar-sub { flex-direction:column; gap:5px; }
  #machineSelector { margin-left:0; margin-top:5px; }
  .sidebar { width:200px; } 
}
```

---

### 15. **Print/PDF Export Still References F06**
**Impact:** Exporting PDF includes "F06 Real Data" in all pages even with F09 selected
**Current:** Line 1101 in exportAllPagesToPDF()
```javascript
// Page title shows 'F06 Real Data' in topbar
```

**Issue:** Print dialog uses hardcoded topbar text
**Fix:** Pre-process exportAllPagesToPDF to update topbar text before rendering

---

## 🟢 WORKING WELL

✅ **Multi-machine data structure** - MACHINES_DATA organized cleanly  
✅ **Dynamic machine selector** - Dropdown works properly  
✅ **Nav sync** - Sidebar "F06 Live KPIs" updates correctly  
✅ **View label updates** - Topbar title changes with machine  
✅ **Chart rendering** - All chart types work (line, doughnut, bar, scatter)  
✅ **Chart.js integration** - Good use of helper functions and destruction  
✅ **Data entry form** - Form logic and validation sound  
✅ **Styling** - CSS variables and layout very clean  
✅ **Color system** - Consistent palette, accessible colors  
✅ **Responsive grid layouts** - g2, g3, g21 breakpoints good  

---

## 📋 IMPLEMENTATION ROADMAP

### **Phase 1: Content Dynamization** (2-3 hours)
1. Create `MACHINE_METADATA` object with per-machine dates/descriptions
2. Fix 6 hardcoded view titles/subtitles
3. Dynamically calculate and render stat card values
4. Update alert generation to be data-driven

### **Phase 2: Empty State Handling** (1-2 hours)
1. Add `renderEmptyState()` function
2. Check for empty data in each render function
3. Show helpful placeholder messages
4. Add "Log Data" action buttons in empty states

### **Phase 3: Error Handling & Robustness** (1-2 hours)
1. Wrap renderView() and chart functions in try-catch
2. Add loading states
3. Add error toast messages
4. Test with corrupted/missing data

### **Phase 4: Mobile & Polish** (1 hour)
1. Fix mobile responsive issues
2. Test on tablet sizes
3. Improve dropdown styling
4. Add keyboard navigation

### **Phase 5: Data Persistence** (optional) (30 mins)
1. Add localStorage for user entries
2. Test data survival on refresh
3. Add clear/export options

---

## 📊 CHECKLIST FOR PRODUCTION

Before deploying with multiple machines:

- [ ] All view titles/subtitles dynamic based on currentMachine
- [ ] Stat card values calculated from MACHINES_DATA
- [ ] Empty states show for machines with no data
- [ ] Alerts generated from actual machine data
- [ ] Breakdown table populated from d.bd field
- [ ] Impact stats calculated, not hardcoded
- [ ] Error handling in all render functions
- [ ] Loading states during chart creation
- [ ] Mobile tested at 375px, 768px, 1024px
- [ ] PDF export shows correct machine name
- [ ] User form auto-selects current machine
- [ ] localStorage backup for user entries
- [ ] Data validation in submitEntry()
- [ ] No console errors on machine switch

---

## 🔧 QUICK WINS (Easy Fixes)

**30-minute fixes:**
1. Add `document.getElementById('f-machine').value = machine` to switchMachine()
2. Wrap empty data arrays check: `if(!MACHINES_DATA[machine] || MACHINES_DATA[machine].length === 0)`
3. Add `id="view-title"` and `id="view-subtitle"` to render dynamically
4. Create `calculateMachineStats()` function
5. Call `live-date` update in renderView()

---

## 💡 FUTURE ENHANCEMENTS

### **Data Management:**
- Bulk import CSV for historical data
- Edit/delete capability for entries
- Data validation rules per machine
- Automatic backup to cloud

### **Analytics:**
- Trend lines and forecasting
- Performance benchmarking between machines
- Anomaly detection (sudden OEE drops)
- Root cause suggestions via rule engine

### **UI/UX:**
- Dark mode toggle
- Custom date range filters
- Comparison view (F06 vs F09 side-by-side)
- Drill-down detail views
- Machine health score dashboard
- Shift handover notes section

### **Integrations:**
- Slack notifications for low OEE
- Email reports (weekly summary)
- API for ERP integration
- Real-time data sync from sensors

### **Admin:**
- User management (view-only vs edit rights)
- Audit log of all entries
- Machine configuration panel
- KPI target management per machine
- Holiday/planned downtime calendar

---

## 🎯 PRIORITY SUMMARY

| Issue | Severity | Impact | Effort | Priority |
|-------|----------|--------|--------|----------|
| Hardcoded titles | 🔴 HIGH | Confusing UX | 1h | P0 |
| Empty data states | 🔴 HIGH | Looks broken | 1.5h | P0 |
| Stat card values | 🔴 HIGH | Wrong data shown | 1.5h | P0 |
| Hardcoded alerts | 🟡 MEDIUM | Misleading | 1h | P1 |
| No error handling | 🟡 MEDIUM | Silent failures | 1.5h | P1 |
| Mobile issues | 🟡 MEDIUM | Bad experience | 1h | P1 |
| Data persistence | 🟢 LOW | Lost on refresh | 30m | P2 |
| Form auto-select | 🟢 LOW | UX friction | 10m | P2 |
| Loading states | 🟢 LOW | User clarity | 1h | P2 |

---

## 📝 SUMMARY

**Current State:** 95% foundational work complete. Machine switching architecture is solid.

**Gap:** Missing content dynamization. Works perfectly for F06 but reveals data for other machines.

**Risk:** If you add F09 data now and users switch, they'll see misleading F06-specific alerts, titles, and stats.

**Timeline to Fix:** 6-8 hours for Phase 1 + 2 (critical path).

**Recommendation:** Complete hardcoding fixes before sharing with team using multiple machines.

---

**Generated:** March 12, 2026 | **Dashboard Version:** 1.0-multi-machine  
**Next Review:** After Phase 1 completion
