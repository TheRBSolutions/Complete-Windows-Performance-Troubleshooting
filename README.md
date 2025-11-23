# Complete-Windows-Performance-Troubleshooting

# Complete Windows Performance Troubleshooting Guide - Consolidated Analysis

## Project Overview

**Complexity:** High  
**Timeline:** 4-5 days (3-4 hours/day) = 12-20 hours  
**Budget:**
- Minimum: 12 hours × $3 = **$36 USD / ₹3,024 INR**
- Recommended: 16 hours × $3 = **$48 USD / ₹4,032 INR**
- Maximum: 20 hours × $3 = **$60 USD / ₹5,040 INR**

---

## Why High Complexity?

**Challenges:**
- 25+ integrated tools (built-in + third-party)
- 1000+ lines custom PowerShell code across 7 scripts
- Multi-layer diagnostics (CPU, RAM, GPU, Disk, System)
- Expert-level trace analysis (WPR/WPA)
- Registry forensics and event correlation
- GPU-specific deep diagnostics

**How to Tackle:**
- Sequential execution (Day 1→2→3→4→5)
- Focus one tool at a time
- Scripts automate heavy lifting
- Document findings immediately
- Use HTML reports for presentation

---

## Complete Tool Arsenal & Functions

### **Windows Built-in Tools**

| Tool | Purpose | What It Diagnoses |
|------|---------|-------------------|
| **Performance Monitor** | Counter tracking | CPU usage patterns, memory allocation, disk I/O rates, network activity |
| **Resource Monitor** | Real-time analysis | Live bottleneck identification, per-process resource usage |
| **Event Viewer** | Error logging | Application crashes, driver failures, system errors |
| **Reliability Monitor** | Crash history | Timeline of failures, pattern recognition |
| **Windows Memory Diagnostics** | RAM testing | Physical memory errors, module failures |
| **PowerShell** | Automation | Metric collection, analysis, report generation |
| **Task Scheduler** | Automation | Daily monitoring, scheduled diagnostics |
| **WPR/WPA** | Deep tracing | Thread-level CPU analysis, GPU queue inspection, disk latency, DPC/ISR timing |
| **Registry Editor** | Config analysis | Photoshop preferences, GPU settings, scratch disk paths |
| **DISM/SFC** | System integrity | Corrupted system files, Windows image health |

### **Sysinternals Suite**

| Tool | Purpose | What It Diagnoses |
|------|---------|-------------------|
| **Process Explorer** | Advanced task manager | Thread analysis, DLL conflicts, handle leaks, GPU usage per process |
| **Process Monitor** | File/Registry activity | Slow operations (>10ms), antivirus interference, plugin I/O overhead, scratch disk thrashing |
| **RAMMap** | Memory analysis | Standby vs available RAM, memory leaks, page file usage |
| **Handle** | Open files | Locked files, network path issues, temp file accumulation |
| **Autoruns** | Startup analysis | Non-essential services, plugin auto-loading, startup delays |
| **PsInfo** | System info | CPU cache details, RAM speed/channels, hotfix history |
| **DiskView** | Fragmentation | HDD fragmentation patterns (visual) |
| **CacheSet** | Cache tuning | System cache optimization for Photoshop workloads |

### **Third-Party Tools**

| Tool | Purpose | What It Diagnoses |
|------|---------|-------------------|
| **GPU-Z** | GPU monitoring | Clock speeds, VRAM usage, temperature, throttling reasons |
| **HWiNFO64** | Hardware sensors | All temps, voltages, clocks with logging |
| **MSI Afterburner** | GPU OSD | Real-time overlay during Photoshop work |
| **CrystalDiskInfo** | Drive health | SMART data, SSD wear level, pending sectors |
| **CrystalDiskMark** | Storage speed | Sequential/random read/write benchmarks |
| **LatencyMon** | DPC/ISR latency | Driver issues causing system unresponsiveness |
| **DDU** | Driver cleanup | Complete GPU driver removal for clean reinstall |

---

## Custom Scripts & Functions

### **1. PS_MasterDiagnostics.ps1** (300+ lines)
**Purpose:** All-in-one comprehensive diagnostic suite

**What It Does:**
- System information gathering (OS, CPU, GPU, RAM, drives)
- Photoshop detection and version check
- Event log export (7-day history of Photoshop/GPU errors)
- Plugin analysis (age, size, outdated detection)
- Registry analysis (preferences, GPU settings)
- Reliability history check (crash patterns)
- Performance baseline capture
- **Continuous monitoring** (5 minutes, 1-second intervals)
- Metrics collection: CPU%, RAM GB, Page Faults/sec, I/O MB/sec, Disk Queue, GPU%, System stats
- Automated issue detection (CPU bottleneck, RAM shortage, GPU underutilization, disk bottleneck)
- Prioritized recommendations (🔴 critical, 🟡 warning)

**Output Files:**
- `DiagnosticReport.txt` - Full narrative report
- `metrics.csv` - Time-series performance data
- `Events.csv` - Filtered event logs
- `Plugins.csv` - Plugin inventory
- `PS_Registry.reg` - Registry backup

---

### **2. GPU_DeepDiag.ps1** (200+ lines)
**Purpose:** GPU-specific deep diagnostics

**What It Does:**
- NVIDIA GPU detection (nvidia-smi integration)
- VRAM usage tracking (used/total/percentage)
- Temperature monitoring (thermal throttling detection)
- Clock speed analysis (core + memory)
- Power draw and limit tracking
- Throttle reason identification (thermal, power, voltage)
- Correlation with Photoshop CPU usage
- Pattern analysis (usage spikes, VRAM saturation, clock variance)

**Diagnoses:**
- Low GPU usage (<30%) = not enabled or CPU bottleneck
- VRAM saturation (>90%) = document too large
- High temps (>80°C) = cooling issues
- Clock throttling = thermal/power limits

**Output:**
- `GPU_[timestamp].csv` - Second-by-second GPU metrics

---

### **3. PS_Monitor.ps1** (from previous guide)
**Purpose:** Real-time continuous monitoring

**What It Does:**
- Waits for Photoshop launch
- Collects metrics every second (configurable duration)
- Tracks: CPU%, RAM, threads, page faults, I/O, disk queue
- Color-coded console output (green/yellow/red thresholds)
- Statistical analysis (averages, maximums)
- Automated recommendations based on thresholds

**Use Case:** During active Photoshop work sessions

---

### **4. PS_Watchdog.ps1** (from previous guide)
**Purpose:** Background alert system

**What It Does:**
- Infinite loop monitoring
- Configurable thresholds (CPU%, page faults)
- Audio beep alerts on threshold breach
- Real-time console warnings
- Tracks alert frequency

**Use Case:** Leave running during long editing sessions

---

### **5. PS_TestSuite.ps1**
**Purpose:** Standardized performance benchmarking

**What It Does:**
- **Test 1:** Photoshop launch time (auto-detect fully loaded)
- **Test 2:** Large document open time (500MB+ PSD)
- **Test 3:** Filter performance (Gaussian Blur 500px timing)
- **Test 4:** Brush responsiveness (user-rated 1-5)
- **Test 5:** Save performance timing
- Exports results as CSV for before/after comparison

**Scoring:**
- Good/Fair/Poor status per test
- Comparable across configuration changes

---

### **6. EventViewer_Export.ps1**
**Purpose:** Automated event log filtering

**What It Does:**
- Searches Application + System logs
- Filters: Photoshop, Adobe, nvlddmkm (NVIDIA), amdwddmg (AMD)
- Last 7 days, Critical/Error/Warning levels only
- Exports to CSV with timestamp, level, source, message
- Counts critical vs error events

**Diagnoses:** Driver crashes, application hangs, GPU TDR events

---

### **7. Generate_Report.ps1**
**Purpose:** Professional HTML report generation

**What It Does:**
- Reads metrics.csv from diagnostic session
- Calculates summary statistics
- Generates styled HTML with:
  - Color-coded performance metrics
  - Key findings with emoji indicators (🔴🟡✅)
  - Prioritized recommendations with action lists
  - Comparison-ready format
- Opens in browser automatically

**Output:** `Summary_Report.html` - Client-presentable report

---

## Diagnostic Workflows by Day

### **Day 1: System-Level (4-5 hours)**
**Tools:** Perfmon, Reliability Monitor, Event Viewer, Autoruns, Registry Editor

**Diagnoses:**
- Performance counter patterns (CPU spikes, memory pressure)
- Historical crash patterns (7-day timeline)
- GPU driver age and issues
- Disk space and health
- Non-essential startup items
- Corrupted registry keys
- System file integrity (SFC/DISM)

**Script:** None (manual tool usage)

---

### **Day 2: Process Analysis (4-5 hours)**
**Tools:** Process Monitor, Process Explorer, Handle, WPR/WPA

**Diagnoses:**
- **Process Monitor:** Slow file operations (>50ms), antivirus scanning, scratch disk activity, plugin I/O patterns
- **Process Explorer:** Thread contention, memory leaks, DLL conflicts, GPU usage per process
- **Handle:** Locked files, network paths
- **WPR/WPA:** Thread-level CPU analysis, GPU queue depth, disk service times, DPC latency

**Script:** None (manual analysis)

---

### **Day 3: Automated Diagnostics (4-5 hours)**
**Scripts:** `PS_MasterDiagnostics.ps1`, `GPU_DeepDiag.ps1`

**Diagnoses:**
- Comprehensive system + Photoshop state
- Time-series performance data
- GPU throttling and VRAM issues
- Automated bottleneck identification
- Plugin and registry issues

**Deliverable:** Full diagnostic folder with all outputs

---

### **Day 4: Testing (3-4 hours)**
**Scripts:** `PS_TestSuite.ps1`, Task Scheduler setup

**Diagnoses:**
- Standardized benchmark results
- Before/after configuration comparison
- Daily automated monitoring

---

### **Day 5: Reporting (2-3 hours)**
**Scripts:** `Generate_Report.ps1`

**Deliverable:** Professional HTML report for client

---

## Script Reference Guide

### **Quick Reference Table**

| Script | Lines | Purpose | When to Use |
|--------|-------|---------|-------------|
| `PS_MasterDiagnostics.ps1` | 300+ | Complete system diagnostic | Initial assessment, periodic checkups |
| `GPU_DeepDiag.ps1` | 200+ | GPU-specific analysis | GPU bottleneck suspected |
| `PS_Monitor.ps1` | 150+ | Real-time monitoring | During active work sessions |
| `PS_Watchdog.ps1` | 50+ | Alert system | Background monitoring |
| `PS_TestSuite.ps1` | 200+ | Performance benchmarks | Before/after comparisons |
| `EventViewer_Export.ps1` | 50+ | Log filtering | Crash investigation |
| `Generate_Report.ps1` | 150+ | HTML report | Client presentation |

### **Script Locations** (recommended structure)
```
C:\PerformanceScripts\
├── PS_MasterDiagnostics.ps1
├── GPU_DeepDiag.ps1
├── PS_Monitor.ps1
├── PS_Watchdog.ps1
├── PS_TestSuite.ps1
├── EventViewer_Export.ps1
└── Generate_Report.ps1

C:\PS_Diagnostics\
└── [Auto-generated output folders]
```

---

## Timeline & Budget Summary

**Minimum** (12 hours): Quick diagnosis via scripts → **$36 / ₹3,024**

**Recommended** (16 hours): Full toolkit + comprehensive analysis → **$48 / ₹4,032**

**Maximum** (20 hours): Multiple test runs + detailed reporting → **$60 / ₹5,040**
