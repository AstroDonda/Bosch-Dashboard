# 🎯 Quick Start: Complete Machine Switching

## What's New ✨

Your dashboard has been completely overhauled to support **true multi-machine KPI tracking**. Everything now changes dynamically when you switch machines:

✅ **Titles and descriptions** - Update to show current machine  
✅ **Stat cards** - Calculate from actual machine data  
✅ **Charts and tables** - Display machine-specific KPIs  
✅ **Alerts** - Show relevant issues for selected machine  
✅ **Empty machines** - Show helpful placeholders instead of confusing blank screens  
✅ **Excel import** - Bulk upload data from your PC for any machine  

---

## 🖥️ How to Switch Machines

### Using the Dropdown
```
1. Look at the topbar (blue toolbar above the main content)
2. Find machine selector: "F06" dropdown
3. Click dropdown
4. Select any machine: F01, F02, ..., F12
5. Everything updates automatically!
```

**What changes:**
- 📊 Sidebar shows "[Machine] Live KPIs"
- 📈 All charts show that machine's data (or empty state if no data)
- 📋 Tables display that machine's daily log
- 📌 Stat cards show calculated values from that machine
- 🔔 Alerts show relevant issues for that machine
- 🔤 All titles mention the selected machine

---

## 📤 How to Upload Excel Data

### Simple Excel Upload
```
1. Prepare Excel/CSV file on your PC (see format below)
2. Click "📤 Upload Excel" button in topbar
3. Select your file
4. Dashboard imports automatically
5. Green notification appears: "✓ Imported X entries from Excel"
6. Switch to different machines to see their data
```

### Excel File Format
Create spreadsheet with these columns:

| Column | Type | Example | Required? |
|--------|------|---------|-----------|
| Date | YYYY-MM-DD | 2026-03-14 | ✅ |
| Machine | F01-F12 | F06 | ✅ |
| OEE Day | 0-100 or text | 68 or "GEZA" | ❌ |
| OEE Night | 0-100 | 72 | ❌ |
| Volume Day | Tonnes | 28 | ❌ |
| Volume Night | Tonnes | 35 | ❌ |
| Breakdown | Text | "JAW stuck" | ❌ |
| Process Failure | Text | "Powder low" | ❌ |
| Sealing | Text | "Leakers detected" | ❌ |

### Example Data:
```
Date       | Machine | OEE Day | OEE Night | Volume Day | Volume Night | Breakdown      | Process Failure
-----------|---------|---------|-----------|-----------|--------------|----------------|------------------
2026-03-13 | F06     | 68      | 72        | 28        | 35           |                | 
2026-03-13 | F09     | 45      | 52        | 18        | 20           | JAW obstruction | No powder
2026-03-14 | F06     | 71      | 75        | 30        | 38           |                |
2026-03-14 | F07     | GEZA    | 68        | —         | 32           |                |
```

---

## 🔍 What Happens When Machine Has No Data

When you select an empty machine (F02, F07, F09 without data), dashboard shows:

```
Overview:
  📊 "No data available for [Machine]"
  "Data will appear when KPI entries are logged"

Stat Cards:
  Avg OEE (Day): —
  Avg OEE (Night): —
  Breakdowns: —
  Failures: —

Alerts:
  ℹ️ "No Data Available"
  "Start logging shift data via 'Log Data' view"

Charts & Tables:
  All blank/empty with helpful message

What to do:
  1. Upload Excel data, OR
  2. Click "+ Log Data" to manually add entries
```

---

## 📊 Understanding the Dashboard Now

### F06 (With Data - 37 Entries)
```
When you select F06:
✓ Overview shows "F06 post-upgrade performance"
✓ Stat cards show calculated values:
  - Avg OEE Day: 45% (calculated from 37 days)
  - Avg OEE Night: 53% (calculated from 37 nights)
  - Breakdowns: 4 (actual count from data)
  - Failures: 11 (actual count from data)
✓ Charts display F06's trend data
✓ Alerts show: "OEE Trending Positively - 15 days ≥65% target"
✓ Causes chart shows: Powder Supply (8), Mechanical (3), etc.
```

### F09 (Empty - No Data Yet)
```
When you select F09:
✓ Overview shows "F09 — Awaiting commissioning (scheduled: 2026 Mar 02)"
✓ Stat cards show: — (dashes)
✓ Charts are empty
✓ Alerts show: "F09 Upgrade In Progress - KPI tracking will begin on commissioning"
✓ Causes chart shows: "No events"
✓ All tables are empty

Ready to receive data via Excel upload or manual entry!
```

---

## 📁 Daily Workflow Examples

### Scenario 1: View F06 Post-Upgrade Performance
```
1. Load dashboard (defaults to F06)
2. See overview: OEE stats, trends, alerts
3. Click "OEE Trend Analysis" to see day/night trends
4. Click "Breakdowns & Failures" to analyze root causes
5. Use insights for improvement actions
```

### Scenario 2: Import Data for Multiple Machines
```
1. Prepare Excel file with all 12 machines' data
2. Click "📤 Upload Excel"
3. Select file → Dashboard imports all rows
4. Switch through machines (F01 → F02 → F03, etc.)
5. See insights for each machine
6. Identify which machines need attention
```

### Scenario 3: Monitor New Machine Post-Commissioning
```
1. Commission F09 (upgrade starts)
2. Operators manually log shift data via "+ Log Data"
3. Or admin uploads daily data via Excel
4. Switch to F09 from dropdown
5. Watch stat cards populate as data arrives
6. Monitor trends as F09 stabilizes
```

### Scenario 4: Add Today's Data
```
1. Click "+ Log Data"
2. Select today's date
3. Select current machine from dropdown
4. Enter OEE%, volume, breakdowns, notes
5. Click "✓ Save Entry"
6. Data appears in heatmap and table
7. Charts update automatically
```

---

## 🎯 Key Features Now Working

| Feature | F06 (with data) | F09 (empty) |
|---------|-----------------|------------|
| Switch dropdown | ✅ Shows F06 data | ✅ Shows placeholder |
| Stat cards | ✅ Calculated values | ✅ Shows dashes (—) |
| Charts | ✅ Display trends | ✅ Empty state message |
| Alerts | ✅ Machine-specific | ✅ Helpful guidance |
| Causes chart | ✅ Actual breakdown data | ✅ "No events" |
| Tables | ✅ All entries | ✅ Empty |
| Navigation | ✅ All 8 views work | ✅ All 8 views work |

---

## ✅ Verification Checklist

Test these to confirm everything works:

### Machine Switching
- [ ] Select F06 → See data (45% avg OEE)
- [ ] Select F09 → See empty state (dashes)
- [ ] Select F07 → See empty state
- [ ] Select back to F06 → Data reappears (no loss)

### Stat Cards
- [ ] F06: Shows "45%" for Day OEE (not "49.5%")
- [ ] F09: Shows "—" (dashes, not data)
- [ ] Switch between machines → Stats change

### Alerts
- [ ] F06: Shows "15 days meeting target"
- [ ] F09: Shows "Upgrade in progress"
- [ ] Machine-specific information

### Causes Chart
- [ ] F06: Shows "Powder Supply (8), Mechanical (3), etc."
- [ ] F09: Shows "No events"
- [ ] Updates when switching

### Excel Upload
- [ ] Click "📤 Upload Excel" → File picker appears
- [ ] Select Excel → "✓ Imported X entries" message
- [ ] Switch to imported machine → Shows new data

### Navigation
- [ ] All 8 views accessible
- [ ] Titles update per machine
- [ ] Charts don't crash on empty machines

---

## 🔧 Troubleshooting

### Problem: Charts are blank
**Solution**: Check if machine has data. F02-F12 are empty by default. Upload Excel or log data via form.

### Problem: Stat cards show "—"
**Solution**: Machine has no data. Upload Excel file with data, or use "+ Log Data" to add entries.

### Problem: Excel upload not working
**Solution**: Check Excel columns match exactly (case-sensitive):
- Date, Machine, OEE Day, OEE Night,  
- Volume Day, Volume Night, Breakdown,  
- Process Failure, Sealing

### Problem: Switching machines is slow
**Solution**: This is normal on first load. Subsequent switches are instant. Reload page if stuck (Ctrl+R).

### Problem: Data disappears when switching
**Solution**: This was the original bug—now fixed! All data persists. If still happening, reload page.

---

## 📖 Excel Import Detailed Example

### Create Excel File (3 simple steps):

**Step 1**: Open Excel/Google Sheets

**Step 2**: Add headers in Row 1:
```
A1: Date
B1: Machine
C1: OEE Day
D1: OEE Night
E1: Volume Day
F1: Volume Night
G1: Breakdown
H1: Process Failure
```

**Step 3**: Add data starting Row 2:
```
A2: 2026-03-13
B2: F06
C2: 68
D2: 72
E2: 28
F2: 35
G2: (leave blank)
H2: (leave blank)

A3: 2026-03-13
B3: F09
C3: 45
D3: 52
E3: 18
F3: 20
G3: JAW obstruction
H3: No powder supply
```

**Step 4**: Save as Excel (.xlsx) or CSV, then upload!

---

## 🎉 You're All Set!

Your dashboard is now ready for **multi-machine KPI tracking**. You can:

1. ✅ **Switch between machines** - Everything updates automatically
2. ✅ **Upload Excel data** - Bulk import for any machine
3. ✅ **View actual stats** - Calculated from real data, not hardcoded
4. ✅ **Get machine-specific alerts** - Relevant insights per machine
5. ✅ **Log manual data** - Daily entry form still available
6. ✅ **Monitor trends** - Charts update as data changes

**Next steps:**
- Prepare Excel file with your machine data
- Upload it using "📤 Upload Excel"
- Switch through machines to verify
- Start monitoring KPIs!

---

If you have questions or find issues, check:
- IMPLEMENTATION_COMPLETE.md (detailed technical guide)
- CODE_FIXES_GUIDE.md (code-level details)
- TESTING_GUIDE.md (validation procedures)
