# Windows Performance Troubleshooting - Part One

## Table of Contents
1. [Tool Installation & Setup](#1-tool-installation--setup)
2. [Understanding Performance Metrics](#2-understanding-performance-metrics)
3. [Complete Diagnostic Workflows](#3-complete-diagnostic-workflows)
4. [Tool-Specific Deep Dives](#4-tool-specific-deep-dives)
5. [Scenario-Based Troubleshooting](#5-scenario-based-troubleshooting)
6. [Advanced Analysis Techniques](#6-advanced-analysis-techniques)
7. [Automation & Scripting](#7-automation--scripting)

---

# 1. Tool Installation & Setup

## 1.1 Windows Performance Toolkit (WPT)

### Installation Steps

**Method 1: Standalone (Recommended)**
1. Visit: https://learn.microsoft.com/windows-hardware/get-started/adk-install
2. Download "Windows ADK Installer"
3. Run installer
4. Select ONLY "Windows Performance Toolkit" (~50MB)
5. Complete installation

**Verify Installation:**
```cmd
wpr -help
wpa
```

**Add to PATH (if needed):**
```cmd
setx PATH "%PATH%;C:\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit"
```

### Post-Installation Configuration

**Run as Administrator:**
- Always run WPR/WPA with admin rights
- Right-click → "Run as administrator"

**Create Desktop Shortcuts:**
```powershell
$WshShell = New-Object -comObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$Home\Desktop\WPA.lnk")
$Shortcut.TargetPath = "C:\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit\wpa.exe"
$Shortcut.Save()
```

---

## 1.2 Sysinternals Suite Installation

### Complete Suite Setup

**Download:**
1. Visit: https://learn.microsoft.com/sysinternals/downloads/
2. Download "Sysinternals Suite" ZIP
3. Extract to: `C:\Tools\Sysinternals`

**Add to System PATH:**
```cmd
setx PATH "%PATH%;C:\Tools\Sysinternals" /M
```

**Accept EULA for All Tools:**
```cmd
cd C:\Tools\Sysinternals
for %f in (*.exe) do %f -accepteula
```

### Individual Tool Setup

**Process Explorer Configuration:**
1. Launch as Administrator
2. **Options → Replace Task Manager** (recommended)
3. **View → Select Columns:**
   - Process Performance tab: CPU Usage, Private Bytes
   - Process GPU tab: GPU Usage, GPU Dedicated Bytes, GPU System Bytes
   - Process I/O tab: I/O Total, I/O Reads, I/O Writes
4. **View → System Information** (verify GPU detection)
5. **File → Save As** (saves configuration)

**Process Monitor Configuration:**
1. Launch as Administrator
2. **Filter → Filter:**
   ```
   Process Name | is | Photoshop.exe | Include | Add
   Duration | is greater than | 0.100 | Include | Add
   ```
3. **Filter → Save Filter** → Name: "Photoshop_Slow_Ops.pms"
4. Configure columns:
   - Right-click header → Add: Duration, Path, Result, Detail
5. **File → Backing Files** → Set to SSD location (for large captures)

**RAMMap Setup:**
- No configuration needed
- Always run as Administrator
- Use F5 to refresh

---

## 1.3 Hardware Monitoring Tools

### MSI Afterburner + RivaTuner Setup

**Installation:**
1. Download from MSI website
2. Install with RivaTuner Statistics Server (RTSS) - check during install
3. Launch MSI Afterburner

**Configuration:**
1. **Settings (gear icon) → Monitoring tab:**
   - GPU usage: ☑ Show in On-Screen Display
   - GPU temperature: ☑ Show in On-Screen Display
   - Memory usage: ☑ Show in On-Screen Display
   - Core clock: ☑ Show in On-Screen Display
   - Frame rate: ☑ Show in On-Screen Display

2. **RivaTuner Statistics Server:**
   - Launch RTSS from system tray
   - Application detection level: High
   - On-Screen Display position: Top right
   - Update rate: 1000 ms (1 second)

**Test Display:**
- Launch Photoshop
- Overlay should appear showing GPU stats

---

### HWiNFO64 Setup

**Installation:**
1. Download from hwinfo.com
2. Install or use portable version
3. Launch → Sensors-only mode

**Configure Sensors:**
1. **Settings → Layout:**
   - Show sensors: GPU, CPU, Motherboard
2. **Right-click sensor window → Customize:**
   - Add to view:
     - GPU Core Temperature
     - GPU Core Clock (all)
     - GPU Core Load
     - GPU Memory Used/Free
     - GPU Power
     - CPU Package Temperature
     - CPU Core Clocks (all cores)

**Enable Logging:**
1. Click log icon (disk with arrow)
2. Location: `C:\HWiNFO_Logs\`
3. Format: CSV
4. Interval: 1000 ms
5. Start logging

---

### GPU-Z Setup

**Installation:**
1. Download from techpowerup.com
2. No installation needed (portable)
3. Launch GPU-Z.exe

**Configure Logging:**
1. **Sensors tab**
2. **Click dropdown (bottom right) → Log to file**
3. Location: `C:\GPU_Logs\gpuz_log.txt`
4. Logging active (shows in title bar)

**Useful Sensors:**
- GPU Load
- Memory Used
- Temperature
- Core Clock
- Memory Clock
- GPU Power
- PerfCap Reason (shows throttling cause)

---

## 1.4 PowerShell Environment Setup

### Enable Script Execution

**Run PowerShell as Administrator:**
```powershell
# Allow scripts for current user
Set-ExecutionPolicy Bypass -Scope CurrentUser

# Verify
Get-ExecutionPolicy -List
```

### Create Scripts Directory

```powershell
# Create folder structure
New-Item -Path "C:\PerformanceScripts" -ItemType Directory -Force
New-Item -Path "C:\PS_Diagnostics" -ItemType Directory -Force
New-Item -Path "C:\PS_Traces" -ItemType Directory -Force

# Set permissions (optional)
icacls "C:\PerformanceScripts" /grant ${env:USERNAME}:F
```

### Test Basic Commands

```powershell
# Test process monitoring
Get-Process Photoshop -ErrorAction SilentlyContinue | Format-List *

# Test performance counters
Get-Counter "\Processor(_Total)\% Processor Time"

# Test GPU counters (may not work on all systems)
Get-Counter "\GPU Engine(*)\Utilization Percentage" -ErrorAction SilentlyContinue

# List available GPU counters
Get-Counter -ListSet "GPU*" | Select-Object CounterSetName, Paths
```

### Create Reusable Functions

**Save as: `C:\PerformanceScripts\PS_Functions.ps1`**

```powershell
# Reusable Photoshop monitoring functions

function Get-PhotoshopStatus {
    $ps = Get-Process Photoshop -ErrorAction SilentlyContinue
    if($ps) {
        [PSCustomObject]@{
            Running = $true
            PID = $ps.Id
            CPU_Seconds = [math]::Round($ps.CPU, 2)
            RAM_GB = [math]::Round($ps.WorkingSet64/1GB, 2)
            Threads = $ps.Threads.Count
            StartTime = $ps.StartTime
        }
    } else {
        [PSCustomObject]@{
            Running = $false
        }
    }
}

function Get-PhotoshopCPU {
    $counter = Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue
    if($counter) {
        [math]::Round($counter.CounterSamples.CookedValue, 2)
    } else {
        "N/A"
    }
}

function Get-PhotoshopPageFaults {
    $counter = Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue
    if($counter) {
        [math]::Round($counter.CounterSamples.CookedValue, 2)
    } else {
        "N/A"
    }
}

function Get-PhotoshopIO {
    $counter = Get-Counter "\Process(Photoshop)\IO Data Bytes/sec" -ErrorAction SilentlyContinue
    if($counter) {
        [math]::Round($counter.CounterSamples.CookedValue/1MB, 2)
    } else {
        "N/A"
    }
}

# Usage:
# . C:\PerformanceScripts\PS_Functions.ps1
# Get-PhotoshopStatus
# Get-PhotoshopCPU
```

---

# 2. Understanding Performance Metrics

## 2.1 CPU Metrics Deep Dive

### CPU Usage Percentage

**What it Measures:**
- Percentage of time CPU spends executing non-idle threads
- Aggregated across all cores OR per-core

**How to Read:**
```powershell
# Total CPU usage
Get-Counter "\Processor(_Total)\% Processor Time"

# Per-core usage
Get-Counter "\Processor(*)\% Processor Time"
```

**Interpretation:**

| Range | Status | Action |
|-------|--------|--------|
| 0-40% | Normal/Idle | No action needed |
| 40-70% | Moderate load | Monitor during tasks |
| 70-85% | High load | Check if sustained |
| 85-100% | Bottleneck | Investigate cause |

**Photoshop-Specific:**
- Filters: 70-100% CPU normal during processing
- Brush: 30-60% CPU normal, >80% indicates issue
- Idle: Should be <10% CPU

**Single-Core vs Multi-Core:**

Check individual cores:
```powershell
Get-Counter "\Processor(*)\% Processor Time" | 
    Select-Object -ExpandProperty CounterSamples | 
    Format-Table InstanceName, CookedValue -AutoSize
```

**Pattern Recognition:**
```
One core at 100%, others at 10-20%:
→ Single-threaded bottleneck
→ Photoshop operation not multi-threaded

All cores at 90-100%:
→ True CPU bottleneck
→ Consider CPU upgrade or reduce workload

All cores at 30-50%:
→ Not CPU bottlenecked
→ Look for GPU or I/O issues
```

---

### Thread Count

**What it Measures:**
- Number of execution threads in Photoshop process

**How to Check:**
```powershell
(Get-Process Photoshop).Threads.Count
```

**Normal Ranges:**
- Idle: 20-40 threads
- Active work: 40-80 threads
- Heavy processing: 80-150 threads

**High Thread Count Issues:**

If threads >150:
- Possible thread leak
- Plugin creating excessive threads
- Check with Process Explorer → Properties → Threads tab
- Sort by CPU Time to find busy threads

---

### Context Switches

**What it Measures:**
- How often CPU switches between threads
- High rate indicates thread contention

**How to Check:**
```powershell
Get-Counter "\Thread(*photoshop*)\Context Switches/sec"
```

**Interpretation:**
- <5,000/sec: Normal
- 5,000-10,000/sec: Moderate contention
- >10,000/sec: High contention (performance impact)

**Causes of High Context Switches:**
- Too many threads competing for CPU
- Lock contention (threads waiting for resources)
- I/O blocking (threads waiting for disk/network)

---

## 2.2 Memory Metrics Deep Dive

### Working Set (RAM Usage)

**What it Measures:**
- Physical RAM currently used by Photoshop

**How to Check:**
```powershell
$ps = Get-Process Photoshop
"Working Set: $([math]::Round($ps.WorkingSet64/1GB, 2)) GB"
"Private Bytes: $([math]::Round($ps.PrivateMemorySize64/1GB, 2)) GB"
```

**Difference Explained:**
- **Working Set**: RAM currently in physical memory
- **Private Bytes**: Total committed memory (may include paged)
- **Virtual Bytes**: Total virtual address space

**Normal Values (32GB RAM system):**
```
Photoshop Preferences set to 70% (22GB):
Working Set: 18-22 GB (normal for large docs)
Private Bytes: Should be close to Working Set
Difference >4GB: Heavy paging occurring
```

---

### Page Faults

**What it Measures:**
- Accesses to memory not currently in RAM

**Types:**
1. **Soft Page Fault**: Data in RAM cache (fast, <1μs)
2. **Hard Page Fault**: Data on disk (slow, 1-10ms)

**How to Check:**
```powershell
# Total page faults
Get-Counter "\Process(Photoshop)\Page Faults/sec"

# Hard faults (disk access)
Get-Counter "\Memory\Page Reads/sec"
```

**Interpretation:**

| Page Faults/sec | Status | Meaning |
|----------------|--------|---------|
| 0-50 | Excellent | All data in RAM |
| 50-100 | Good | Minimal paging |
| 100-500 | Warning | Some RAM pressure |
| >500 | Critical | Severe RAM shortage |

**Action Based on Faults:**
```
>100 hard faults/sec:
1. Check RAM allocation in Photoshop
2. Close other applications
3. Reduce document size
4. Add more physical RAM
```

---

### Photoshop Efficiency Indicator

**Location:** Bottom-left of document window (click dropdown)

**What it Measures:**
- Percentage of operations completed in RAM vs scratch disk

**Values:**
```
100%: Perfect - all in RAM
95-99%: Excellent - minimal scratch use
90-94%: Good - some scratch use
85-89%: Fair - noticeable slowdown
<85%: Poor - heavy scratch disk usage
```

**Troubleshooting Low Efficiency:**

**Step 1: Check RAM allocation**
```
Edit → Preferences → Performance
Let Photoshop Use: Increase to 75-85%
```

**Step 2: Check scratch disk**
```
Preferences → Performance → Scratch Disks
Ensure fastest drive is selected
Ensure 20GB+ free space
```

**Step 3: Reduce memory usage**
```
Edit → Purge → All
Reduce History States
Close unused documents
```

---

## 2.3 GPU Metrics Deep Dive

### GPU Utilization

**What it Measures:**
- Percentage of GPU compute units in use

**How to Check:**

**Method 1: Task Manager**
- Performance tab → GPU → 3D graph
- Shows Photoshop's GPU usage

**Method 2: PowerShell**
```powershell
Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage"
```

**Method 3: GPU-Z**
- Sensors tab → GPU Load value

**Method 4: NVIDIA (nvidia-smi)**
```cmd
nvidia-smi --query-gpu=utilization.gpu --format=csv --loop=1
```

**Interpretation:**

| GPU Usage | CPU Usage | Diagnosis |
|-----------|-----------|-----------|
| 80-100% | 40-60% | GPU bottleneck |
| 20-40% | 80-100% | CPU bottleneck |
| <20% | <50% | Neither bottleneck (check disk) |
| Fluctuating | Stable | GPU memory thrashing |

---

### VRAM (Video Memory) Usage

**What it Measures:**
- GPU memory used for textures, layer cache, acceleration

**How to Check:**

**Task Manager:**
```
Performance → GPU → Dedicated GPU Memory
Shows: X.X GB / Y.Y GB
```

**GPU-Z:**
```
Sensors tab → Memory Used
Also shows: Memory Free, Memory Load %
```

**NVIDIA:**
```cmd
nvidia-smi --query-gpu=memory.used,memory.total --format=csv
```

**Thresholds:**

| VRAM Usage | Status | Action |
|------------|--------|--------|
| <70% | Healthy | No action |
| 70-85% | Elevated | Monitor |
| 85-95% | High | Reduce document size |
| >95% | Critical | VRAM bottleneck |

**VRAM Bottleneck Symptoms:**
- GPU usage fluctuates wildly (20% → 80% → 30%)
- Stuttering during GPU-accelerated operations
- "Out of GPU memory" errors
- Performance worse than CPU mode

---

### GPU Clock Speed

**What it Measures:**
- Actual GPU core frequency (MHz or GHz)

**Typical Behavior:**
- **Base Clock**: Minimum guaranteed (e.g., 1,350 MHz)
- **Boost Clock**: Maximum (e.g., 1,900 MHz)
- **Actual Clock**: Real-time (varies with load/temp)

**How to Check:**

**GPU-Z:**
```
Graphics Card tab: GPU Clock (current)
Sensors tab: GPU Clock (real-time monitoring)
```

**HWiNFO:**
```
GPU Core Clock sensor
```

**NVIDIA:**
```cmd
nvidia-smi --query-gpu=clocks.sm --format=csv --loop=1
```

**Throttling Detection:**

```
Normal behavior:
Idle: 300-600 MHz (power saving)
Load: Boost to 1800-1900 MHz
Sustained: Stays at 1750-1850 MHz

Thermal throttling:
Load: Boosts to 1900 MHz
Temperature rises to 83C
Clock drops to 1650 MHz
Further temp rise → 1500 MHz

Power limit throttling:
Load: Boosts to 1900 MHz
Hits 100% TDP (e.g., 220W)
Clock drops to 1750 MHz to stay under limit
```

**Check Throttle Reason (GPU-Z):**
```
Sensors → PerfCap Reason:
- Idle: Normal (not under load)
- VRel: Voltage reliability limit
- Pwr: Power limit
- Thrm: Thermal limit
- vOp: Operating voltage limit
```

---

### GPU Temperature

**Ranges:**

| Temperature | Status | Action |
|-------------|--------|--------|
| 30-50°C | Idle | Normal |
| 50-70°C | Light load | Normal |
| 70-80°C | Heavy load | Normal |
| 80-85°C | Hot | Monitor |
| 85-90°C | Throttling | Improve cooling |
| >90°C | Critical | Immediate action |

**How to Check:**

**GPU-Z:** Sensors → GPU Temperature
**HWiNFO:** GPU Temperature sensor
**NVIDIA:** `nvidia-smi --query-gpu=temperature.gpu`

**Cooling Solutions:**

```
Progressive actions:
1. Clean dust from GPU cooler (compressed air)
2. Increase case fan speeds (BIOS/software)
3. Improve case airflow (cable management, fan placement)
4. Reapply thermal paste (advanced)
5. Aftermarket cooler (advanced)
6. Undervolt GPU (reduces heat without much performance loss)
```

---

## 2.4 Disk Metrics Deep Dive

### Disk Queue Length

**What it Measures:**
- Number of I/O requests waiting to be processed

**How to Check:**

**Resource Monitor:**
```
Win+R → resmon → Disk tab → Queue Length column
```

**PowerShell:**
```powershell
Get-Counter "\PhysicalDisk(*)\Current Disk Queue Length"
```

**Interpretation:**

| Queue Length | Status | Meaning |
|--------------|--------|---------|
| 0-1 | Optimal | No waiting |
| 1-2 | Normal | Minimal wait |
| 2-5 | Elevated | Some contention |
| >5 | Bottleneck | Disk can't keep up |

**Photoshop Context:**
```
During save (expected high queue):
Queue: 3-6 (normal, sequential write)
Active Time: 80-100%

During scratch disk use (concerning):
Queue: 8-15 (bottleneck)
Active Time: 100%
Duration: Sustained minutes
→ RAM shortage + slow disk
```

---

### Active Time Percentage

**What it Measures:**
- Percentage of time disk is processing requests

**How to Check:**

**Task Manager:**
```
Performance → Disk → Active time %
```

**Resource Monitor:**
```
Disk tab → Disk Activity graph
```

**Interpretation:**

| Active Time | Status | Action |
|-------------|--------|--------|
| 0-30% | Light use | Normal |
| 30-70% | Moderate | Normal |
| 70-90% | Heavy | Monitor |
| 90-100% | Saturated | Bottleneck |

**Pattern Analysis:**
```
Spiky (0% → 100% → 0%):
→ Normal (saves, cache writes)
→ No action needed

Sustained 100% for minutes:
→ Bottleneck (scratch disk or HDD)
→ Check which process
→ If Photoshop: RAM shortage likely
→ If other process: Background task
```

---

### Transfer Rate

**What it Measures:**
- Data throughput (MB/s or GB/s)

**How to Check:**

**Task Manager:**
```
Performance → Disk → Read/Write speed
```

**Resource Monitor:**
```
Disk tab → Disk Activity → Read/Write B/sec
```

**Expected Speeds:**

| Drive Type | Sequential Read | Sequential Write | Random 4K |
|------------|----------------|------------------|-----------|
| HDD 7200RPM | 100-200 MB/s | 100-180 MB/s | 0.5-2 MB/s |
| SATA SSD | 500-550 MB/s | 450-520 MB/s | 20-40 MB/s |
| NVMe Gen3 | 2000-3500 MB/s | 1500-3000 MB/s | 40-80 MB/s |
| NVMe Gen4 | 5000-7000 MB/s | 4000-6500 MB/s | 80-150 MB/s |

**Photoshop Scratch Disk Requirements:**
```
Minimum: 200 MB/s (fast HDD acceptable but slow)
Recommended: 500 MB/s (SATA SSD minimum)
Optimal: 2000+ MB/s (NVMe for large files)
```

**Test Disk Speed (CrystalDiskMark):**
```
1. Download CrystalDiskMark
2. Select scratch disk drive
3. Run test (5 runs, 1GB)
4. Check SEQ1M Q8T1 (sequential speed)

Results interpretation:
<200 MB/s: Too slow for scratch, upgrade to SSD
200-500 MB/s: Acceptable, but limiting
>500 MB/s: Good for Photoshop scratch
>2000 MB/s: Excellent, no disk bottleneck
```

---

## 2.5 System Latency Metrics

### DPC (Deferred Procedure Call) Latency

**What it is:**
- Kernel-mode functions executed at high priority
- Drivers use DPCs for interrupt handling
- High DPC latency = system unresponsiveness

**How to Check:**

**LatencyMon:**
```
1. Download from resplendence.com
2. Launch LatencyMon
3. Click "Start"
4. Let run during Photoshop use
5. Check results after 2-5 minutes
```

**Interpretation:**

| Highest DPC | Status | Impact |
|-------------|--------|--------|
| <100 μs | Excellent | No issues |
| 100-500 μs | Good | Minor impact |
| 500-1000 μs | Warning | Noticeable lag |
| >1000 μs | Critical | Severe problems |

**Results Display:**
```
LatencyMon shows:
"Your system appears suitable for real-time audio"
→ DPC latency is fine

"Your system is having trouble..."
→ DPC latency is problematic
→ Check Drivers tab for culprit
```

**Common Culprits:**

| Driver | Typical Issue | Solution |
|--------|---------------|----------|
| Netwtw10.sys | Intel Wi-Fi driver | Update or disable Wi-Fi |
| nvlddmkm.sys | NVIDIA driver | Update GPU driver |
| HDAudBus.sys | Audio driver | Update audio driver |
| storport.sys | Storage driver | Update chipset drivers |
| tcpip.sys | Network stack | Check network adapter |

**Fix High DPC Latency:**
```
1. Identify driver from LatencyMon "Drivers" tab
2. Device Manager → Find device
3. Update driver (Windows Update or manufacturer)
4. If still high: Roll back to previous version
5. Last resort: Disable device temporarily
```

---

### ISR (Interrupt Service Routine) Time

**What it is:**
- Time spent handling hardware interrupts
- Should be minimal (<1% CPU time)

**How to Check:**
- LatencyMon → Stats tab → ISR execution time

**Interpretation:**
```
ISR time <1% CPU: Normal
ISR time 1-5%: Elevated (monitor)
ISR time >5%: Problem (identify device)
```

**High ISR Causes:**
- Malfunctioning hardware
- Driver bugs
- Excessive interrupts (e.g., high polling rate mouse at 8000Hz)
- USB device issues

---

### Frame Time Analysis

**What it Measures:**
- Time from frame render to display (input lag)
- Critical for smooth brush response

**Target:**
- 60 FPS = 16.67ms per frame
- Consistent frame time = smooth feel
- Variable frame time = stuttery

**How to Measure:**

**PresentMon (Microsoft tool):**
```cmd
# Download from GitHub: GameTechDev/PresentMon
PresentMon-x64.exe -process_name Photoshop.exe -timed 60 -output_file frames.csv
```

**During capture:**
- Use Photoshop normally
- Paint with brush
- Apply filters
- Switch tools

**Analysis (Excel):**
```
Open frames.csv
Key column: MsBetweenPresents (frame time in ms)

Smooth performance:
16.5, 16.7, 16.6, 16.8, 16.5
Avg: 16.6ms (60 FPS)
StdDev: 0.1ms (very consistent)

Stuttery performance:
16.5, 16.7, 45.2, 16.6, 62.1, 16.5
Avg: 28.9ms (still ~35 FPS average)
StdDev: 18.5ms (HIGH variance = stutter)
```

**High Variance Causes:**
- DPC latency spikes
- Disk I/O blocking
- GPU driver timeout
- Background process CPU bursts
- Memory paging

---

# 3. Complete Diagnostic Workflows

## 3.1 Quick Check (30 Seconds)

### Step-by-Step Immediate Assessment

**A. Photoshop Efficiency Indicator**

```
1. Open any document in Photoshop
2. Look at bottom-left of window
3. Click dropdown menu
4. Select "Efficiency"

Reading:
100%: ✓ All operations in RAM (perfect)
95-99%: ✓ Minimal scratch use (good)
90-94%: ⚠ Some scratch use (monitor)
<90%: ✗ Heavy scratch use (problem)

If <100%: RAM or scratch disk issue
```

**B. System Info Check**

```
Help → System Info (or Ctrl+Alt+Shift+I)

Check GPU section:
GPU: [Should show your GPU model]
OpenCL: Available [Should say "Yes"]
OpenGL Drawing: Enabled
Metal/DirectX: Enabled

If GPU shows "Unavailable":
→ GPU not detected
→ Driver issue or GPU disabled
```

**C. Performance Preferences**

```
Edit → Preferences → Performance

Memory Usage:
Slider should be at 70-85%
If lower: Wasting available RAM
If higher: System instability risk

Graphics Processor:
☑ Use Graphics Processor (should be checked)
Advanced Settings: Normal or Advanced mode

Scratch Disks:
First disk: Should be fastest SSD
Free space: Should show 20GB+ available
```

**D. Quick Brush Test**

```
1. File → New: 3000x3000px, RGB, 8-bit
2. Brush Tool (B)
3. Size: 500px, Hardness: 0%
4. Paint 5-6 quick strokes

Observe:
Smooth, no lag: ✓ System performing well
Slight lag: ⚠ Investigate further
Severe lag: ✗ Significant bottleneck present
```

---

## 3.2 Resource Overview (1-2 Minutes)

### Multi-Tool Quick Scan

**A. Task Manager Overview**

```
Ctrl+Shift+Esc → Performance tab

CPU:
Check utilization %
If >80% sustained: CPU bottleneck

Memory:
Check "In use" vs "Available"
If Available <4GB: RAM shortage

Disk:
Check Active time %
If 100% sustained: Disk bottleneck

GPU (if available):
Check 3D utilization
Check Dedicated GPU memory
If VRAM maxed: VRAM bottleneck
```

**B. Resource Monitor Deep Dive**

```
Win+R → resmon → Enter
(Or: Task Manager → Performance → Open Resource Monitor)

CPU Tab:
Find Photoshop.exe
Note CPU % and Threads count

Memory Tab:
Find Photoshop.exe
Check Working Set (RAM usage)
Check Hard Faults/sec
>100 faults/sec = RAM problem

Disk Tab (MOST IMPORTANT):
Watch during Photoshop operation
Disk Queue Length: Should be <2
Active Time %: Sustained 100% = problem
Note which physical disk is busy

Network Tab:
Usually not relevant for Photoshop
Unless using cloud storage or network files
```

**C. Quick PowerShell Check**

```powershell
# Open PowerShell, run:
$ps = Get-Process Photoshop -ErrorAction SilentlyContinue
if($ps) {
    Write-Host "Photoshop Status:" -ForegroundColor Cyan
    Write-Host "CPU Time: $([math]::Round($ps.CPU, 2))s total"
    Write-Host "RAM: $([math]::Round($ps.WorkingSet64/1GB, 2)) GB"
    Write-Host "Threads: $($ps.Threads.Count)"
    
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    Write-Host "CPU%: $([math]::Round($cpu, 2))%"
    
    $pf = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    Write-Host "Page Faults: $([math]::Round($pf, 2))/sec" -ForegroundColor $(if($pf -gt 100){"Red"}else{"Green"})
} else {
    Write-Host "Photoshop not running" -ForegroundColor Red
}
```

**D. GPU Monitoring Setup**

```
Launch GPU-Z:
1. Sensors tab
2. Observe while using Photoshop:
   - GPU Load %
   - Memory Used / Total
   - Temperature
   - Core Clock

Or MSI Afterburner:
Enable OSD to see live stats overlay

During Photoshop work, note:
GPU high (>70%): GPU being utilized
GPU low (<30%): CPU bottleneck or GPU not enabled
VRAM >90%: VRAM bottleneck
Temp >80°C: Thermal issue
```

---

## 3.3 WPR Trace Capture (2-5 Minutes)

### Complete Trace Workflow

**Step 1: Preparation**

```
1. Close unnecessary applications
2. Ensure 2GB+ free disk space for trace
3. Have Photoshop open with problem document
4. Know exact steps to reproduce issue
```

**Step 2: Start Trace**

```cmd
# Open Command Prompt as Administrator
Win+X → Command Prompt (Admin)

# Start recording
wpr -start CPU -start GPU -start DiskIO -filemode

Expected output:
"Microsoft Windows Performance Recorder
Recording started."
```

**Alternative: Comprehensive Trace**

```cmd
# For more detailed analysis
wpr -start GeneralProfile -start GPU -filemode

GeneralProfile includes:
- CPU scheduling
- Disk I/O
- File I/O
- Registry access
- Network activity
```

**Step 3: Reproduce Issue**

```
1. Switch to Photoshop
2. Perform exact operation that causes lag:
   - Apply specific filter
   - Use laggy brush
   - Open/save large file
   - Any reproducible problem
3. Continue for 10-60 seconds
   (Longer capture = larger file)
4. Note exact time of worst lag
```

**Step 4: Stop Trace**

```cmd
# Return to Command Prompt
wpr -stop C:\PS_Traces\photoshop_lag.etl

Process:
- "Merging traces..." (10-30 seconds)
- "100% complete"
- "Saving merged trace..."
Done!
```

**File Size Expectations:**
```
30-second trace: ~200-500 MB
1-minute trace: ~500 MB - 1 GB
5-minute trace: ~2-5 GB
```

**Step 5: Open in WPA**

```cmd
# Launch Windows Performance Analyzer
wpa C:\PS_Traces\photoshop_lag.etl

Loading:
- Parsing... (10-60 seconds depending on size)
- Applying symbols (optional, can skip)
- Ready for analysis
```

---

## 3.4 WPA Analysis

### Essential Graphs for Photoshop

**Graph 1: CPU Usage (Precise)**

```
1. Graph Explorer (left panel) → Computation
2. Drag "CPU Usage (Precise)" to analysis area

Configure:
- Preset: "Utilization by Process, Thread, Stack"
- Find Photoshop.exe in process list
- Expand to see threads

Look for:
- Thread with high CPU% during lag
- Stack trace showing which function
- Long-running operations (>100ms)

Interpretation:
High CPU in single thread: Single-threaded bottleneck
High CPU in multiple threads: Multi-threaded, might be normal
Low CPU with lag: Not CPU bottleneck
```

**Graph 2: GPU Hardware Queue**

```
Computation → GPU Hardware Queue

Shows:
- GPU command submission
- Queue depth (pending commands)
- Packet duration (execution time)

Look for:
Queue depth 0-1 + gaps: CPU not feeding GPU
Queue depth 10+: GPU overloaded
Long packet duration: GPU-intensive operation

Zoom to lag timeframe:
- Select time range
- Ctrl+Z to zoom
- Examine pattern during problem
```

**Graph 3: Disk Usage**

```
Storage → Disk Usage

Preset: "Disk Offset by Process, Thread, Path, Stack"

Find Photoshop.exe:
- Expand to see disk operations
- Check "Disk Service Time" column
- Look at "Path Name" (which file EOF
wc -l /home/claude/Windows_Performance_Detailed_COMPLETE.md
# Windows Performance Troubleshooting - Complete Detailed Guide (Continued)

## 3.4 WPA Analysis (Continued)

**Graph 3: Disk Usage**
```
Storage → Disk Usage

Preset: "Disk Offset by Process, Thread, Path, Stack"

Find Photoshop.exe:
- Expand to see disk operations
- Check "Disk Service Time" column
- Look at "Path Name" (which files accessed)
- Sort by "Size" to find large operations

Interpretation:
Many small reads (<1MB): Cache misses or fragmentation
Large sequential writes (>100MB): Save operations (normal)
High service time (>100ms): Slow disk or queue backlog
Path shows scratch disk: RAM shortage forcing disk use
```

**Pattern Recognition in Disk Graph:**
```
Normal save operation:
├─ Sequential writes to .psd file
├─ Service time: 10-50ms per operation
├─ Total duration: 2-10 seconds
└─ Pattern: Continuous writes, no gaps

RAM shortage pattern:
├─ Random reads/writes to scratch disk
├─ Service time: 50-200ms per operation
├─ Total duration: Throughout work session
├─ Pattern: Constant activity, small chunks
└─ Action: Increase RAM allocation

Disk bottleneck:
├─ Queue length >5 visible in timeline
├─ Service time >500ms
├─ Multiple processes competing
└─ Action: Upgrade to SSD or reduce background I/O
```

**Graph 4: File I/O**
```
Storage → File I/O

Shows higher-level file operations:
- CreateFile, ReadFile, WriteFile
- Useful for identifying which files cause delays

Configure:
- Group by: Process Name, File Path
- Sort by: Duration (descending)

Look for:
- Operations >1 second duration
- Repeated access to same file (inefficiency)
- Network paths (UNC: \\server\share)
- Antivirus scanning (Windows Defender paths)
```

**Graph 5: Memory Usage**
```
Memory → Virtual Memory Snapshots

Shows memory allocation over time

Key columns:
- Process: Photoshop.exe
- Commit Size: Allocated memory
- Working Set: Physical RAM in use

Timeline analysis:
Working Set increasing steadily: Normal usage
Working Set dropping suddenly: Windows reclaimed RAM
Commit Size >> Working Set: Heavy paging occurring

Cross-reference with Page Fault graph:
High page faults + dropping working set = RAM pressure
```

**Graph 6: Context Switches**
```
Computation → CPU Usage (Sampled)
Right-click header → Add Column → Context Switch Count

High context switches indicate:
- Thread contention
- Lock waiting
- I/O blocking

Photoshop context switches:
<5,000/sec per thread: Normal
5,000-10,000/sec: Moderate contention
>10,000/sec: Severe contention (investigate)

Check "Wait Reason" column:
- WaitFreePage: Memory shortage
- WaitForSingleObject: Waiting on sync object
- WaitForMultipleObjects: Complex synchronization
- Suspended: Thread blocked
```

### Advanced WPA Techniques

**Comparing Two Traces:**
```
Scenario: Want to compare slow vs. fast operation

1. Capture "good" trace: wpr -start CPU -start GPU -filemode
2. Perform fast operation (e.g., small file save)
3. Stop: wpr -stop good_trace.etl
4. Capture "bad" trace: Repeat with slow operation
5. Stop: wpr -stop bad_trace.etl

Analysis:
1. Open both in WPA
2. Window → New Comparison Window
3. Select both traces
4. View differences side-by-side

Look for:
- Operations present in bad trace only
- Longer duration in bad trace
- Different code paths taken
```

**Symbol Loading for Stack Traces:**
```
To see function names instead of memory addresses:

1. WPA → Trace → Configure Symbol Paths
2. Add Microsoft symbol server:
   SRV*C:\Symbols*https://msdl.microsoft.com/download/symbols
3. Trace → Load Symbols
4. Wait for download (can take 5-10 minutes first time)

Benefits:
- See actual function names in Photoshop
- Identify which Photoshop subsystem is slow
- Example: "Adobe::GPU::Renderer::Process" vs. "0x7FF8C4B2"
```

**Filtering Noise:**
```
Problem: Trace contains too much irrelevant data

Solutions:

1. Time-based filtering:
   - Click and drag on timeline to select range
   - Right-click → Zoom to Selection
   - Focus only on problem timeframe

2. Process filtering:
   - Find Photoshop.exe in process list
   - Right-click → Filter to Process
   - Removes all other processes from view

3. Duration filtering:
   - Right-click column header → View Filter
   - Set Duration: "is greater than" 100ms
   - Shows only long-running operations

4. Stack filtering:
   - Right-click stack column → Filter
   - Include only: Photoshop.exe modules
   - Excludes system DLLs for cleaner view
```

---

## 3.5 Process Monitor Deep Dive

### Capturing Useful Data

**Step 1: Configure Filters BEFORE Capture**
```
Launch Procmon as Administrator

Essential filters:
1. Process Name | is | Photoshop.exe | Include
2. Duration | is greater than | 0.010 | Include (10ms threshold)
3. Result | is not | SUCCESS | Include (catch errors)

Optional performance filters:
4. Path | ends with | .tmp | Include (temp file activity)
5. Path | contains | scratch | Include (scratch disk)
6. Operation | is | WriteFile | Include (focus on writes)

Apply filters: Ctrl+L or Filter menu
```

**Why Filter First:**
- Unfiltered Procmon can capture 10,000+ events/second
- File size grows rapidly (1GB+ in minutes)
- Filtering reduces noise by 95%+

**Step 2: Enable Backing File (Important!)**
```
File → Backing Files

Settings:
☑ Use file named: C:\PS_Traces\procmon_capture.pml
Maximum file size: 2048 MB (2GB)
☑ Delete when closed (if temporary capture)

Why important:
- Without backing file: RAM fills quickly
- Long captures require backing file
- Prevents Out of Memory errors
```

**Step 3: Start Capture**
```
1. Ctrl+E to start (or click Capture icon)
2. Status bar shows: "Events: X, Captured: Y, Filtered: Z"
3. Perform slow Photoshop operation
4. Capture for 30-60 seconds
5. Ctrl+E to stop

Expected results:
Total events: 10,000-50,000 (unfiltered)
Filtered events: 500-2,000 (with good filters)
```

**Step 4: Save Capture**
```
File → Save
Format: Native Process Monitor Format (.PML)
Location: C:\PS_Traces\photoshop_analysis.PML

PML vs CSV:
PML: Keeps all metadata, stacktraces, can re-filter
CSV: Lightweight, for Excel analysis, loses some data
```

### Analysis Techniques

**Technique 1: Sort by Duration**
```
Click "Duration" column header to sort descending

Top operations are your bottlenecks:

Example findings:
1. WriteFile to D:\PS_Scratch\temp.tmp - 2.5 seconds
   → Slow scratch disk write
   
2. ReadFile from C:\Users\...\texture.jpg - 1.8 seconds
   → Large texture load, consider faster storage
   
3. CreateFile for output.psd - 0.8 seconds
   → Antivirus scanning on save?

Action items prioritized by impact (longest durations first)
```

**Technique 2: Stack Trace Analysis**
```
Double-click any event → Stack tab

Shows call chain:
Photoshop.exe+0x4B2C10    ← Photoshop code
Photoshop.exe+0x8A1234
kernel32.dll!WriteFile     ← System call
ntdll.dll!NtWriteFile      ← Kernel entry

If symbols loaded (see Setup section):
Adobe::Document::Save()    ← Readable function names
Adobe::FileIO::Write()

Use stack to understand:
- Why operation occurred
- Which Photoshop subsystem initiated it
- Whether it's expected or anomalous
```

**Technique 3: Path Analysis**
```
Tools → File Summary (Ctrl+Alt+F)

Groups all file activity by path:

Example results:
D:\PhotoshopScratch\
  ├─ Events: 12,453
  ├─ Read: 2.1 GB
  ├─ Write: 4.8 GB
  └─ Duration: 45.2 seconds total
  
C:\Users\User\Documents\project.psd
  ├─ Events: 234
  ├─ Read: 512 MB
  ├─ Write: 0 bytes (read-only access)
  └─ Duration: 1.2 seconds

Insights:
Heavy scratch disk use: RAM shortage
Excessive reads of same file: Caching issue
Writes to unusual paths: Plugin behavior
```

**Technique 4: Timeline View**
```
View → Show Timeline (Ctrl+T)

Shows event distribution over time:

Patterns to recognize:

Even distribution:
████████████████████████ 
→ Continuous background activity

Bursts:
█    ████    ██    ███████
→ Periodic operations (autosave, cache flush)

Sustained block:
          ████████████████
          ↑
          User action (save, filter apply)

Gap then activity:
████          ████████
    ↑ 
    User pause, then continued work
```

**Technique 5: Process Tree**
```
Tools → Process Tree (Ctrl+T)

Shows parent-child relationships:

Photoshop.exe (PID: 4512)
├─ Adobe CEF Helper.exe (PID: 5104) ← Extensions/panels
├─ Adobe QT Server.exe (PID: 5200) ← QuickTime codec
└─ Adobe Synchronizer.exe (PID: 5312) ← Cloud sync

If a child process has high I/O:
- May be plugin causing slowdown
- Consider disabling to test
- Check if necessary for work
```

### Common Patterns and Solutions

**Pattern 1: Excessive Scratch Disk Activity**
```
Symptoms in Procmon:
- 1000+ WriteFile operations to scratch location
- Duration of writes: 50-200ms each
- Continuous throughout session

Root cause: Insufficient RAM

Solution steps:
1. Edit → Preferences → Performance
2. Let Photoshop Use: Increase to 80%
3. Close other applications
4. Reduce document size if possible
5. If still occurs: Add physical RAM
```

**Pattern 2: Antivirus Scanning on Save**
```
Symptoms:
- CreateFile operation duration: 500-2000ms
- Stack trace shows: FLTMGR.SYS (filter manager)
- Path: Final output file

Root cause: Real-time antivirus scans file on creation

Solutions:
1. Add Photoshop to antivirus exclusions (safe if trusted)
2. Exclude output folder from real-time scanning
3. Adjust antivirus scan settings (reduce aggressiveness)

Steps (Windows Defender):
Settings → Update & Security → Windows Security
→ Virus & threat protection → Manage settings
→ Exclusions → Add folder → Select Photoshop output folder
```

**Pattern 3: Network Storage Latency**
```
Symptoms:
- Operations to \\server\share\ paths
- Duration: 500-5000ms per operation
- Multiple retries visible

Root cause: Working with files over network

Solutions:
1. Copy project files to local SSD
2. Work locally, sync back when done
3. If must use network:
   - Verify network speed (should be 1Gbps+)
   - Check network storage performance
   - Consider mapped drive vs. UNC path
   - Enable offline files (Windows feature)
```

**Pattern 4: Plugin I/O Overhead**
```
Symptoms:
- Procmon shows Adobe CEF Helper.exe
- File access to %AppData%\Adobe\CEP\extensions
- High frequency (100+ events/second)

Root cause: Poorly optimized plugin

Solutions:
1. Identify plugin from path name
2. Window → Extensions → Disable suspected plugin
3. Test performance without plugin
4. If improved: Contact plugin developer or find alternative
5. Remove plugin: C:\Program Files\Adobe\Adobe Photoshop\Required\CEP\extensions
```

**Pattern 5: Font Cache Thrashing**
```
Symptoms:
- Repeated access to Windows\Fonts folder
- Operations to font cache files
- Occurs when using Text tool

Root cause: Font cache corruption or too many fonts

Solutions:
1. Clear font cache:
   - Close Photoshop
   - Delete: C:\Windows\ServiceProfiles\LocalService\AppData\Local\FontCache*
   - Restart Windows
   
2. Reduce installed fonts:
   - Control Panel → Fonts
   - Move rarely-used fonts to backup folder
   - Keep <500 fonts installed

3. Use font manager:
   - NexusFont or FontBase
   - Load fonts on-demand
```

---

## 4. Tool-Specific Deep Dives

## 4.1 Process Explorer Mastery

### Setup for Photoshop Analysis

**Essential Columns:**
```
View → Select Columns

Process Performance tab:
☑ CPU Usage
☑ CPU Time
☑ Context Switch Delta/sec
☑ Private Bytes
☑ Working Set
☑ Peak Working Set
☑ Page Faults Delta/sec

Process GPU tab:
☑ GPU Usage
☑ GPU Dedicated Bytes
☑ GPU System Bytes
☑ GPU Commit

Process I/O tab:
☑ I/O Total
☑ I/O Reads
☑ I/O Writes
☑ I/O Read Bytes
☑ I/O Write Bytes
```

**Visual Enhancements:**
```
Options → Difference Highlight Duration: 2 seconds

Effect:
- Values changing rapidly: Highlighted colors
- CPU spikes: Bright red highlight
- Memory changes: Color flash
- Makes bottlenecks visually obvious
```

**Performance Graphs:**
```
View → System Information (Ctrl+I)

CPU tab: Overall system CPU usage over time
GPU tab: GPU utilization over time
Memory tab: RAM usage, commit charge
Network tab: Bandwidth usage
Disk tab: I/O activity

Keep this window open during Photoshop work
Watch for correlation between actions and spikes
```

### Real-Time Monitoring Workflow

**Step 1: Baseline Measurement**
```
1. Launch Photoshop (no documents open)
2. Find Photoshop.exe in Process Explorer
3. Right-click → Properties

Note baseline values:
CPU: 0-2% (idle should be very low)
Working Set: 500-800 MB (without documents)
I/O: Minimal (<1 MB/sec)
GPU: 0-5% (idle should be near zero)

If baseline is high:
- Background plugin activity
- Corrupted preferences
- Extension panel loading
```

**Step 2: Document Load Analysis**
```
File → Open large document (e.g., 500MB PSD)

Watch Process Explorer during load:

Expected behavior:
CPU: Spike to 40-80% (parsing file)
Working Set: Increase by ~file size (e.g., +500MB)
I/O Reads: Sustained high (reading file)
GPU: Brief spike at end (rendering preview)

Problem indicators:
CPU at 100% for >30 seconds: Slow single-core parsing
Working Set increases slowly: Disk bottleneck
I/O Read Bytes/sec low (<50 MB/sec): Slow storage
Page Faults spiking: RAM shortage
```

**Step 3: Filter Application Test**
```
Filter → Blur → Gaussian Blur (large radius)

Observe:

GPU-accelerated:
CPU: 30-50% (feeding GPU)
GPU Usage: 80-100% (doing work)
Working Set: Minimal increase
Pattern: Quick completion

CPU-only:
CPU: 100% on multiple cores
GPU Usage: <10%
Working Set: May increase
Pattern: Slower completion

If GPU not used when it should be:
Edit → Preferences → Performance
Advanced Settings: Check settings
Try "Basic" or "Advanced" modes
```

**Step 4: Brush Lag Investigation**
```
Use Brush tool with large soft brush

Expected (smooth):
CPU: 40-60% per stroke
GPU: 50-70% (if GPU preview enabled)
Context Switches: <5,000/sec
Page Faults: <50/sec

Laggy behavior:
CPU: Maxed at 100% (CPU bottleneck)
GPU: Low <20% (not helping)
Context Switches: >10,000/sec (thread contention)
Page Faults: >100/sec (RAM thrashing)

Click on Photoshop.exe → Properties → Threads tab:
Sort by CPU Time: Find busiest thread
If one thread >>: Single-threaded bottleneck
If spread evenly: Good multi-threading
```

### Process Explorer Advanced Features

**DLL View:**
```
View → Lower Pane View → DLLs

Shows all loaded libraries:

Look for:
- Third-party DLLs: Plugins, codecs
- Version info: Outdated components
- Path location: Non-standard plugin directories

Problem DLL indicators:
- Very old version number (pre-2020)
- Non-Adobe DLLs in Photoshop folder
- Multiple versions of same DLL

Example issue:
Old QuickTime DLL: Can cause file format issues
→ Update QuickTime or remove if unused
```

**Handles View:**
```
View → Lower Pane View → Handles

Shows open files, registry keys, events

File handles:
- See which files Photoshop has open
- Useful for "File in use" errors
- Find leaked file handles (not closed properly)

Search (Ctrl+F):
Type filename to find which process has it open
```

**Performance Graph Annotations:**
```
Right-click graph → Add Annotation

Mark key events:
- "Started large filter" at 10:23:45
- "Photoshop froze" at 10:24:12
- "Resumed" at 10:24:30

Compare annotations to graph spikes:
Correlate user actions with resource usage
```

**Process Comparison:**
```
File → Save As: Save current snapshot
Do something in Photoshop (e.g., apply effect)
File → Compare: Compare to saved snapshot

Shows differences:
- Memory increase
- CPU time consumed
- I/O generated
- New threads created

Useful for measuring exact impact of operations
```

---

## 4.2 RAMMap for Memory Analysis

### Understanding Memory Types

**Launch RAMMap as Administrator:**
```
Key tabs and their meanings:

Use Counts:
- Total: Sum of all RAM usage
- Available: RAM that can be freed instantly
- Standby: File cache (can be repurposed)
- Modified: Dirty pages waiting to write to disk

Processes:
- Shows RAM usage per process
- Private: Memory exclusively for that process
- Working Set: Currently in physical RAM

Physical Pages:
- Map of actual RAM chips
- Color-coded by use
- Visual representation of fragmentation
```

### Photoshop Memory Issues

**Scenario 1: Photoshop Says "Not Enough RAM"**
```
Check RAMMap:

1. Use Counts tab:
   Available: 8 GB shown
   
2. Empty → Empty Working Sets
   This frees standby cache
   
3. Check Available again:
   If Available increases: Confirm RAM was available
   
4. If Photoshop still complains:
   Issue: Photoshop's preference setting
   Fix: Edit → Preferences → Performance
   Increase "Let Photoshop Use" slider
```

**Scenario 2: System Slow Despite Available RAM**
```
RAMMap → Use Counts:

Modified: 4 GB (high)
Standby: 12 GB

Problem: Modified pages not flushing to disk
Cause: Slow storage or background disk activity

Solution:
1. Empty → Empty Modified Page List (forces flush)
2. Check Disk activity in Task Manager
3. If slow disk: Upgrade to SSD
4. If fast disk: Disable background processes (Windows Search, etc.)
```

**Scenario 3: Memory Leak Detection**
```
Workflow:
1. Launch RAMMap before starting Photoshop
2. Note Photoshop RAM: ~800 MB (baseline)
3. Use Photoshop normally for 30 minutes
4. Refresh RAMMap
5. Check Photoshop RAM: Should be ~work + baseline

Memory leak indicators:
Photoshop RAM: 8 GB (with only 2 GB project)
Constantly increasing even when idle
Not released after closing documents

To confirm:
Close all documents in Photoshop
Edit → Purge → All
Wait 1 minute
RAM should drop back near baseline
If stays high: Likely memory leak (plugin or bug)
```

### Optimizing RAM for Photoshop

**Pre-Work Optimization:**
```
Before starting heavy Photoshop work:

1. RAMMap → Empty → Empty Working Sets
   Frees RAM from background apps
   
2. Empty → Empty Standby List
   Clears file cache
   
3. Empty → Empty Modified Page List
   Flushes pending writes
   
4. Result: Maximum free RAM for Photoshop

Note: System will automatically repopulate these
This is safe and temporary
Gives Photoshop first access to most RAM
```

**Monitoring During Work:**
```
Keep RAMMap open:

Every 5-10 minutes, check:
- Processes tab → Find Photoshop
- Working Set: RAM currently used
- Private: Memory allocated

Warning signs:
Working Set increasing steadily: Normal if actively working
Working Set >> file size: Possible inefficiency
Private much larger than Working Set: Heavy paging
```

---

## 5. Scenario-Based Troubleshooting

## 5.1 Slow Brush Performance

### Symptom
```
Brush strokes lag behind cursor
Noticeable delay between input and screen update
Brush feels "heavy" or unresponsive
```

### Diagnostic Steps

**Step 1: Quick Checks**
```
1. Check brush size:
   Large brushes (>1000px): Expected to be slower
   Solution: Reduce size or use smaller screen resolution
   
2. Check brush hardness:
   Soft brushes (0% hardness): More CPU intensive
   Solution: Increase hardness to 50-100% for better performance
   
3. Check document size:
   4K+ canvas with many layers: Expected lag
   Solution: Work at lower resolution, upscale at end
   
4. Check layer count:
   50+ layers: Slows brush preview
   Solution: Merge non-essential layers
```

**Step 2: GPU Check**
```
Edit → Preferences → Performance

Graphics Processor Settings:
☑ Use Graphics Processor (should be checked)

Advanced Settings:
Mode: Advanced (try this first)
If lagging: Try Normal or Basic

Test each mode:
Basic: CPU-only rendering
Normal: Standard GPU acceleration
Advanced: Maximum GPU use

If Advanced is slower than Basic:
→ GPU driver issue
→ Update GPU drivers
→ Check GPU temperature (might be throttling)
```

**Step 3: Resource Monitoring**
```
Launch Process Explorer:
Find Photoshop.exe

While using brush:
Watch CPU%: Should be 40-70%
Watch GPU Usage: Should be 50-90%
Watch Page Faults/sec: Should be <50

Problem patterns:

CPU 100%, GPU <20%:
→ CPU bottleneck or GPU not engaged
→ Enable GPU in preferences
→ Update GPU drivers

CPU 40%, GPU 100%:
→ GPU bottleneck
→ Reduce brush size
→ Lower document resolution
→ Check VRAM usage

CPU 100%, Page Faults >100/sec:
→ RAM shortage
→ Close other documents
→ Increase RAM allocation
→ Add physical RAM
```

**Step 4: WPR Trace Analysis**
```
Capture trace during brush lag:

wpr -start CPU -start GPU -filemode
[Use brush for 30 seconds]
wpr -stop brush_lag.etl
wpa brush_lag.etl

Analysis:

GPU Hardware Queue graph:
Queue depth 0 during lag: CPU not feeding GPU fast enough
Queue depth 10+: GPU overloaded
Long packet duration (>50ms): Complex shader or GPU overload

CPU Usage (Precise):
Find Photoshop thread with high CPU
Check stack trace
If in: Adobe::Brush::Render(): Expected
If in: Adobe::Layer::Composite(): Layer blend overhead
If in: System DLLs: Driver overhead
```

### Solutions by Cause

**Cause: GPU Not Enabled**
```
Solution:
Edit → Preferences → Performance
☑ Use Graphics Processor
Advanced Settings: Try Advanced mode

Restart Photoshop

Test brush: Should be smoother

If still not working:
Help → System Info
Check GPU section: Should say "Available"
If Unavailable: Driver issue or unsupported GPU
```

**Cause: GPU Driver Issue**
```
Solution:

1. Identify GPU:
   Device Manager → Display adapters

2. Update driver:
   NVIDIA: GeForce Experience or website
   AMD: AMD Software or website
   Intel: Intel Driver & Support Assistant

3. Clean install (if update doesn't help):
   Download DDU (Display Driver Uninstaller)
   Boot to Safe Mode
   Run DDU to remove old driver
   Reboot
   Install latest driver clean

4. Test in Photoshop
```

**Cause: CPU Bottleneck**
```
Solution:

Short-term:
- Reduce brush complexity (fewer dynamics, simpler tip)
- Lower document resolution
- Flatten layers when possible
- Close other applications
- Disable unnecessary Photoshop panels

Long-term:
- CPU upgrade (focus on single-core speed)
- Overclock CPU (if capable and cooled)
- Work on smaller documents
- Use proxy workflow (low-res edit, high-res final)
```

**Cause: RAM Shortage**
```
Solution:

Immediate:
Edit → Purge → All
Close unused documents
Close other applications

Settings:
Edit → Preferences → Performance
Let Photoshop Use: 75-80% (not higher)
History States: 20 (reduce from default 50)

System-level:
Close browser tabs
Disable background apps:
  Task Manager → Startup → Disable unnecessary
  Settings → Privacy → Background apps → Disable

Ultimate:
Add more physical RAM (16GB → 32GB+ recommended)
```

**Cause: Disk Bottleneck (Scratch)**
```
Solution:

Check scratch disk usage:
Efficiency indicator: Should be >95%
If <90%: Scratch disk in use

Immediate fix:
Edit → Purge → All
Close and reopen document

Scratch disk settings:
Edit → Preferences → Performance → Scratch Disks
☑ Select fastest SSD
Ensure 50GB+ free space

Verify disk speed:
Use CrystalDiskMark to test scratch disk
Should be >500 MB/s for SATA SSD
Should be >2000 MB/s for NVMe

If slow HDD:
Urgent: Upgrade to SSD
Temporary: Free up more space (reduces fragmentation)
```

---

## 5.2 Filter Processing Takes Forever

### Symptom
```
Applying filters (Blur, Sharpen, etc.) is extremely slow
Progress bar barely moves
Photoshop becomes unresponsive during filter
```

### Diagnostic Workflow

**Step 1: Identify If It's Truly Slow**
```
Baseline expectations:

Gaussian Blur (500px radius) on 4000x3000 image:
Fast system (i7 10th gen + RTX 3060): 3-8 seconds
Average system (i5 8th gen + GTX 1660): 8-15 seconds
Slow system (i5 6th gen + no GPU): 30-60 seconds

If taking 2-3 minutes: Definitely a problem
If taking 15-20 seconds: Depends on hardware (might be normal)
```

**Step 2: Check GPU Acceleration**
```
While filter is processing:

Task Manager → Performance → GPU:
GPU should be 70-100%
If <30%: GPU not accelerating filter

Filters that support GPU:
- Gaussian Blur
- Motion Blur
- Smart Sharpen
- Liquify
- Neural Filters

Filters that don't:
- Add Noise (CPU only)
- Some third-party filters

If GPU not engaged:
Edit → Preferences → Performance
☑ Use Graphics Processor must be on
Advanced Settings: Try Advanced mode
```

**Step 3: Document Size Assessment**
```
Image → Image Size

Check dimensions and resolution:

Problem indicators:
Width/Height: >8000px
Resolution: >300 DPI
Layers: 50+
Bit depth: 16-bit or 32-bit

Why it matters:
Filter processing scales with pixels
8000x8000 = 64 megapixels
4000x4000 = 16 megapixels
4x reduction = 4x faster processing

Solution for huge documents:
1. Work on smaller proxy version
2. Apply filters at lower resolution
3. Save filter settings
4. Apply to full-res version at end when needed
```

**Step 4: WPR Capture During Filter**
```
wpr -start CPU -start GPU -filemode
[Apply slow filter]
wpr -stop filter_slow.etl
wpa filter_slow.etl

In WPA:

GPU Hardware Queue:
Look at queue depth during filter
Consistent 8-10 packets: GPU working hard (expected)
Fluctuating 0-2-5-0: CPU-GPU sync issues
Empty queue with low GPU%: GPU not used

CPU Usage (Precise):
All cores >80%: Multi-threaded filter (good)
One core 100%, others low: Single-threaded (bad)
All cores <40%: GPU doing the work (expected)

If CPU and GPU both low:
Check Disk Usage graph
High disk activity: Paging to scratch disk (RAM shortage)
```

### Solutions

**Solution 1: Enable GPU Acceleration**
```
Edit → Preferences → Performance

Graphics Processor Settings:
☑ Use Graphics Processor (enable)
Advanced Settings: Select "Advanced"

For filters specifically:
Edit → Preferences → Technology Previews
☑ Enable experimental features (if available)

After changes:
Completely restart Photoshop
Test filter again
```

**Solution 2: Optimize RAM Usage**
```
Check Efficiency indicator during filter:
<95%: RAM shortage causing slowdown

Fixes:

A. Close other documents:
File → Close All except current
Edit → Purge → All

B. Increase RAM allocation:
Edit → Preferences → Performance
Let Photoshop Use: Increase to 75-80%

C. Reduce History States:
Preferences → Performance
History States: 20 (down from 50)

D. Flatten layers before filter (if safe):
Layer → Flatten Image (or Merge Visible)
Filters process faster on fewer layers
```

**Solution 3: Reduce Document Complexity**
```
Before applying filter:

1. Merge layers when possible:
   Select layers → Ctrl+E (Merge Down)
   Or: Create merged copy (Ctrl+Alt+Shift+E)

2. Reduce bit depth (if acceptable):
   Image → Mode → 8 Bits/Channel
   (From 16-bit or 32-bit)
   Significantly faster processing

3. Crop to selection:
   If only filtering part of image
   Make selection → Image → Crop

4. Work on smaller copy:
   Image → Image Size → Reduce to 50%
   Apply filter
   Note settings
   Undo size reduction
   Apply filter to full size (it uses cached computation)
```

**Solution 4: Scratch Disk Optimization**
```
If Efficiency <90%:

A. Free up scratch disk space:
   Delete temp files
   Empty Recycle Bin
   Keep 100GB+ free

B. Change scratch disk to faster drive:
   Edit → Preferences → Performance → Scratch Disks
   Uncheck slow drives (HDDs)
   ☑ Only fastest SSD
   
C. Use multiple scratch disks:
   ☑ Check multiple fast drives
   Photoshop will distribute load

D. Defragment scratch disk (HDDs only):
   Windows: Defragment and Optimize Drives utility
   (Don't defrag SSDs - not needed and reduces lifespan)
```

**Solution 5: Smart Filters Instead**
```
Convert layer to Smart Object first:

1. Select layer
2. Filter → Convert for Smart Filters
3. Apply filter (same speed initially)

Benefits:
- Filter is non-destructive
- Can adjust settings without re-applying
- Can disable/re-enable instantly (no re-processing)
- Can apply to multiple layers efficiently

When to use:
- Experimenting with settings
- Might need to change later
- Applying same filter to multiple images

When NOT to use:
- Need absolute maximum speed
- Working with very large files (Smart Objects add overhead)
```

---

## 5.3 Photoshop Startup is Slow

### Symptom
```
Photoshop takes 30+ seconds to launch
Splash screen hangs on "Initializing..." steps
Long delay before main window appears
```

### Diagnostic Approach

**Step 1: Measure Baseline**
```
Time from double-click to main window:

Fast launch: 5-10 seconds
Average launch: 10-20 seconds
Slow launch: 20-40 seconds
Very slow: 40+ seconds

If >30 seconds: Problem exists
If <15 seconds: Acceptable given Photoshop's complexity
```

**Step 2: Watch Splash Screen**
```
Pay attention to where it pauses:

"Loading plugins...": Plugin issue
"Initializing fonts...": Font cache problem
"Reading preferences...": Corrupt preferences
"Initializing GPU...": GPU driver delay
"Loading workspaces...": Corrupt workspace

Note the longest pause to focus troubleshooting
```

**Step 3: Process Monitor Capture**
```
Launch Procmon as Admin

Filters:
Process Name | is | Photoshop.exe | Include
Duration | is greater than | 0.100 | Include

Start capture (Ctrl+E)
Launch Photoshop
Stop capture when main window appears

Analysis:
Sort by Duration (descending)
Look for operations >1 second

Common slow operations:
- Reading thousands of fonts
- Loading many plugins
- Accessing network paths
- Antivirus scanning files
```

**Step 4: Clean Boot Test**
```
Test if third-party services/startups interfere:

msconfig → Services tab
☑ Hide all Microsoft services
Disable All

msconfig → Startup tab
Open Task Manager → Startup tab
Disable all third-party items

Restart computer
Launch Photoshop
Time startup

If faster: Third-party conflict
Enable services/startups one by one to find culprit
```

### Solutions

**Solution 1: Font Cache Reset**
```
If pauses on "Initializing fonts...":

A. Clear Windows font cache:
   Close Photoshop
   Open CMD as Admin:
   
   cd %WinDir%\ServiceProfiles\LocalService\AppData\Local
   attrib -h FontCache*.dat
   del FontCache*.dat
   del GDIP*.dat
   
   Restart computer

B. Clear Photoshop font cache:
   Navigate to:
   C:\Users\[Username]\AppData\Roaming\Adobe\Adobe Photoshop [Version]\
   Delete: Adobe Photoshop [Version] Prefs.psp
   (Backup first!)

C. Reduce installed fonts:
   Control Panel → Fonts
   Move rarely-used fonts to backup folder
   Keep <500 fonts for best performance

Test launch after each step
```

**Solution 2: Disable Plugins**
```
If pauses on "Loading plugins...":

A. Find plugins folder:
   C:\Program Files\Adobe\Adobe Photoshop [Version]\Plug-ins

B. Temporarily move plugins:
   Create folder: C:\PS_Plugins_Disabled
   Move contents of Plug-ins to disabled folder
   Keep only:
   - Filters (Adobe's built-in)
   - File Formats (Adobe's built-in)

C. Launch Photoshop:
   Should be much faster

D. Re-enable plugins one by one:
   Move plugin back to Plug-ins folder
   Launch Photoshop
   Note startup time
   If dramatically slower: That plugin is the problem

E. Problematic plugins:
   Old/outdated plugins
   Network license validators
   Plugins that load large data on startup
```

**Solution 3: Reset Preferences**
```
If pauses on "Reading preferences...":

A. Safe launch (resets preferences):
   Hold Ctrl+Alt+Shift while launching Photoshop
   Dialog: "Delete Adobe Photoshop Settings?"
   Click Yes

B. Manual preference delete:
   Close Photoshop
   Navigate to:
   C:\Users\[Username]\AppData\Roaming\Adobe\Adobe Photoshop [Version]\
   Delete:
   - Adobe Photoshop [Version] Prefs.psp
   - Adobe Photoshop [Version] X64 Prefs.psp
   
   Launch Photoshop (recreates defaults)

C. Reconfigure preferences:
   Edit → Preferences
   Set RAM allocation: 70-80%
   Enable GPU
   Set scratch disks
   (Previous settings lost, need to reapply)
```

**Solution 4: GPU Initialization Delay**
```
If pauses on "Initializing GPU...":

A. Update GPU drivers:
   NVIDIA: Latest GeForce/Studio driver
   AMD: Latest Adrenalin driver
   Intel: Latest Graphics Driver

B. Temporarily disable GPU in Photoshop:
   Safe launch (Ctrl+Alt+Shift)
   Edit → Preferences → Performance
   ☐ Use Graphics Processor (uncheck)
   Restart Photoshop normally
   
   If now fast: GPU driver issue
   Update/reinstall driver
   Re-enable GPU

C. Check for driver conflicts:
   Device Manager → Display adapters
   Should show only one adapter (or iGPU + dGPU)
   Multiple identical entries: Driver corruption
   Solution: Use DDU to clean reinstall
```

**Solution 5: Disable Cloud Sync**
```
If network activity during startup:

Edit → Preferences → File Handling
☐ Disable "Automatically Sync Settings"

Help → Sign Out (if using Creative Cloud)
Test launch (should be faster without cloud sync delay)

Sign back in after testing if needed
```

**Solution 6: Antivirus Exclusions**
```
Procmon shows antivirus scanning Photoshop files:

Windows Defender:
Settings → Update & Security → Windows Security
→ Virus & threat protection → Manage settings
→ Exclusions → Add folder

Add:
- C:\Program Files\Adobe
- C:\Users\[Username]\AppData\Roaming\Adobe
- C:\Users\[Username]\AppData\Local\Adobe

Third-party antivirus:
Add same paths to exclusions
Refer to antivirus documentation

Security note:
This is safe for Adobe applications
Reduces startup time by 30-50%
```

---

## 6. Advanced Analysis Techniques

## 6.1 Bottleneck Identification Matrix

### Methodology

**Multi-Tool Correlation:**
```
Simultaneously monitor:
1. Task Manager (CPU, RAM, Disk, GPU)
2. Process Explorer (detailed process metrics)
3. GPU-Z or MSI Afterburner (GPU specifics)
4. HWiNFO64 (temperatures, clocks)

Perform slow operation in Photoshop
Observe all tools during problem
```

**Decision Matrix:**

| CPU% | GPU% | RAM Free | Disk Active | Page Faults/sec | Bottleneck | Primary Solution |
|------|------|----------|-------------|-----------------|------------|------------------|
| >90% | <30% | >4GB | <50% | <50 | CPU | Upgrade CPU or reduce workload |
| <50% | >90% | >4GB | <50% | <50 | GPU | Upgrade GPU or reduce GPU load |
| <70% | <70% | <2GB | >80% | >100 | RAM | Add RAM or close applications |
| <70% | <70% | >4GB | 100% | <50 | Disk | Upgrade to SSD or reduce I/O |
| <70% | <70% | >4GB | <50% | <50 | None (Software) | Driver update or Photoshop issue |
| >80% | >80% | <2GB | 100% | >100 | Multiple | System-wide upgrade needed |

**Example Diagnosis:**
```
Scenario: Filter application is slow

Observations:
CPU: 45%
GPU: 25%
RAM Free: 1.5GB
Disk: 100% active
Page Faults: 150/sec
Efficiency: 82%

Matrix match: RAM bottleneck
Root cause: Insufficient RAM → Heavy paging to disk
Solutions:
1. Increase RAM allocation in Photoshop
2. Close other applications
3. Add physical RAM (long-term)
```

---

## 6.2 Performance Profiling Script

### Automated Monitoring

**PowerShell Script: `C:\PerformanceScripts\PS_Monitor.ps1`**
```powershell
# Comprehensive Photoshop Performance Monitor
# Logs metrics every second to CSV

param(
    [int]$Duration = 300,  # 5 minutes default
    [string]$OutputPath = "C:\PS_Diagnostics\metrics_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
)

Write-Host "Starting Photoshop Performance Monitor" -ForegroundColor Cyan
Write-Host "Duration: $Duration seconds" -ForegroundColor Yellow
Write-Host "Output: $OutputPath" -ForegroundColor Yellow
Write-Host "`nWaiting for Photoshop to start..." -ForegroundColor Gray

# Wait for Photoshop process
while (-not (Get-Process Photoshop -ErrorAction SilentlyContinue)) {
    Start-Sleep -Seconds 1
}

$photoshop = Get-Process Photoshop
Write-Host "Photoshop detected (PID: $($photoshop.Id))" -ForegroundColor Green
Write-Host "Collecting metrics...`n" -ForegroundColor Green

# Initialize CSV
$headers = "Timestamp,CPU_Percent,RAM_GB,RAM_Private_GB,Threads,PageFaults_Sec,IO_MB_Sec,DiskQueue,GPU_Percent,GPU_VRAM_GB"
$headers | Out-File -FilePath $OutputPath -Encoding UTF8

$startTime = Get-Date

for ($i = 0; $i -lt $Duration; $i++) {
    $photoshop = Get-Process Photoshop -ErrorAction SilentlyContinue
    
    if (-not $photoshop) {
        Write-Host "Photoshop closed. Stopping monitoring." -ForegroundColor Red
        break
    }
    
    # Collect metrics
    $timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
    
    # CPU
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $cpuPercent = [math]::Round($cpu, 2)
    
    # Memory
    $ramGB = [math]::Round($photoshop.WorkingSet64 / 1GB, 2)
    $ramPrivateGB = [math]::Round($photoshop.PrivateMemorySize64 / 1GB, 2)
    $threads = $photoshop.Threads.Count
    
    # Page Faults
    $pageFaults = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $pageFaultsRounded = [math]::Round($pageFaults, 2)
    
    # I/O
    $ioBytes = (Get-Counter "\Process(Photoshop)\IO Data Bytes/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $ioMB = [math]::Round($ioBytes / 1MB, 2)
    
    # Disk Queue
    $diskQueue = (Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $diskQueueRounded = [math]::Round($diskQueue, 2)
    
    # GPU (may not work on all systems)
    $gpuPercent = 0
    $gpuVRAM = 0
    try {
        $gpuCounter = Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage" -ErrorAction Stop
        $gpuPercent = [math]::Round(($gpuCounter.CounterSamples | Measure-Object -Property CookedValue -Average).Average, 2)
    } catch {}
    
    # Build CSV line
    $line = "$timestamp,$cpuPercent,$ramGB,$ramPrivateGB,$threads,$pageFaultsRounded,$ioMB,$diskQueueRounded,$gpuPercent,$gpuVRAM"
    $line | Out-File -FilePath $OutputPath -Encoding UTF8 -Append
    
    # Console output (every 5 seconds)
    if ($i % 5 -eq 0) {
        Write-Host "[$timestamp] CPU: $cpuPercent% | RAM: $ramGB GB | PageFaults: $pageFaultsRounded/s | Disk Q: $diskQueueRounded" -ForegroundColor White
    }
    
    Start-Sleep -Seconds 1
}

Write-Host "`nMonitoring complete!" -ForegroundColor Green
Write-Host "Data saved to: $OutputPath" -ForegroundColor Cyan
Write-Host "`nAnalyzing results..." -ForegroundColor Yellow

# Simple analysis
$data = Import-Csv $OutputPath
$avgCPU = ($data | Measure-Object -Property CPU_Percent -Average).Average
$maxCPU = ($data | Measure-Object -Property CPU_Percent -Maximum).Maximum
$avgRAM = ($data | Measure-Object -Property RAM_GB -Average).Average
$maxRAM = ($data | Measure-Object -Property RAM_GB -Maximum).Maximum
$avgPageFaults = ($data | Measure-Object -Property PageFaults_Sec -Average).Average
$maxPageFaults = ($data | Measure-Object -Property PageFaults_Sec -Maximum).Maximum

Write-Host "`nSummary:" -ForegroundColor Cyan
Write-Host "CPU: Avg=$([math]::Round($avgCPU,2))% Max=$([math]::Round($maxCPU,2))%" -ForegroundColor $(if($avgCPU -gt 80){"Red"}else{"Green"})
Write-Host "RAM: Avg=$([math]::Round($avgRAM,2))GB Max=$([math]::Round($maxRAM,2))GB" -ForegroundColor $(if($maxRAM -gt 16){"Yellow"}else{"Green"})
Write-Host "Page Faults: Avg=$([math]::Round($avgPageFaults,2))/s Max=$([math]::Round($maxPageFaults,2))/s" -ForegroundColor $(if($avgPageFaults -gt 100){"Red"}elseif($avgPageFaults -gt 50){"Yellow"}else{"Green"})

# Recommendations
Write-Host "`nRecommendations:" -ForegroundColor Cyan
if ($avgCPU -gt 85) {
    Write-Host "- CPU bottleneck detected. Consider CPU upgrade or reducing workload." -ForegroundColor Red
}
if ($maxRAM -gt 20 -or $avgPageFaults -gt 100) {
    Write-Host "- RAM shortage detected. Increase RAM allocation or add physical RAM." -ForegroundColor Red
}
if ($avgPageFaults -lt 50 -and $avgCPU -lt 70) {
    Write-Host "- System resources look healthy!" -ForegroundColor Green
}
```

**Usage:**
```powershell
# Monitor for 5 minutes (default)
.\PS_Monitor.ps1

# Monitor for 10 minutes
.\PS_Monitor.ps1 -Duration 600

# Custom output path
.\PS_Monitor.ps1 -Duration 300 -OutputPath "C:\Logs\test_session.csv"
```

**Analysis in Excel:**
```
1. Open CSV in Excel
2. Create line charts:
   - CPU_Percent over time
   - RAM_GB over time
   - PageFaults_Sec over time
3. Identify spikes correlating with user actions
4. Look for sustained high values (bottlenecks)
5. Compare multiple sessions to track improvements
```

---

## 7. Automation & Scripting

## 7.1 Automated Diagnostic Suite

**PowerShell Script: `C:\PerformanceScripts\PS_Diagnostics.ps1`**
```powershell
# Automated Photoshop Diagnostics Suite
# Runs multiple checks and generates report

$reportPath = "C:\PS_Diagnostics\Report_$(Get-Date -Format 'yyyyMMdd_HHmmss').txt"

function Write-Report {
    param([string]$Text, [string]$Color = "White")
    Write-Host $Text -ForegroundColor $Color
    $Text | Out-File -FilePath $reportPath -Append -Encoding UTF8
}

Write-Report "======================================" "Cyan"
Write-Report "PHOTOSHOP PERFORMANCE DIAGNOSTICS" "Cyan"
Write-Report "======================================" "Cyan"
Write-Report "Date: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')" "Gray"
Write-Report ""

# System Info
Write-Report "[1] SYSTEM INFORMATION" "Yellow"
$os = Get-CimInstance Win32_OperatingSystem
$cpu = Get-CimInstance Win32_Processor | Select-Object -First 1
$ram = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)
$freeRAM = [math]::Round($os.FreePhysicalMemory / 1MB, 2)

Write-Report "OS: $($os.Caption) $($os.Version)"
Write-Report "CPU: $($cpu.Name)"
Write-Report "CPU Cores: $($cpu.NumberOfCores) cores, $($cpu.NumberOfLogicalProcessors) threads"
Write-Report "RAM: $ram GB total, $freeRAM GB free"
Write-Report ""

# GPU Detection
Write-Report "[2] GPU DETECTION" "Yellow"
$gpu = Get-CimInstance Win32_VideoController | Where-Object { $_.Name -notlike "*Microsoft*" }
if ($gpu) {
    Write-Report "GPU: $($gpu.Name)" "Green"
    $vram = [math]::Round($gpu.AdapterRAM / 1GB, 2)
    if ($vram -gt 0) {
        Write-Report "VRAM: $vram GB"
    }
    Write-Report "Driver: $($gpu.DriverVersion)"
} else {
    Write-Report "GPU: Not detected or integrated only" "Red"
}
Write-Report ""

# Storage Info
Write-Report "[3] STORAGE ANALYSIS" "Yellow"
$drives = Get-CimInstance Win32_LogicalDisk | Where-Object { $_.DriveType -eq 3 }
foreach ($drive in $drives) {
    $freeGB = [math]::Round($drive.FreeSpace / 1GB, 2)
    $totalGB = [math]::Round($drive.Size / 1GB, 2)
    $percentFree = [math]::Round(($drive.FreeSpace / $drive.Size) * 100, 1)
    
    $color = "Green"
    if ($percentFree -lt 20) { $color = "Red" }
    elseif ($percentFree -lt 30) { $color = "Yellow" }
    
    Write-Report "$($drive.DeviceID) $freeGB GB free / $totalGB GB total ($percentFree% free)" $color
}
Write-Report ""

# Photoshop Status
Write-Report "[4] PHOTOSHOP STATUS" "Yellow"
$ps = Get-Process Photoshop -ErrorAction SilentlyContinue
if ($ps) {
    Write-Report "Status: Running (PID: $($ps.Id))" "Green"
    Write-Report "RAM Usage: $([math]::Round($ps.WorkingSet64/1GB, 2)) GB"
    Write-Report "CPU Time: $([math]::Round($ps.CPU, 2)) seconds"
    Write-Report "Threads: $($ps.Threads.Count)"
    
    # Real-time CPU
    Start-Sleep -Seconds 1
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    if ($cpu) {
        Write-Report "Current CPU: $([math]::Round($cpu, 2))%"
    }
    
    # Page Faults
    $pf = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    if ($pf) {
        $pfColor = if ($pf -gt 100) { "Red" } elseif ($pf -gt 50) { "Yellow" } else { "Green" }
        Write-Report "Page Faults: $([math]::Round($pf, 2))/sec" $pfColor
    }
} else {
    Write-Report "Status: Not running" "Red"
}
Write-Report ""

# Performance Counters
Write-Report "[5] SYSTEM PERFORMANCE" "Yellow"
$totalCPU = (Get-Counter "\Processor(_Total)\% Processor Time").CounterSamples.CookedValue
Write-Report "System CPU: $([math]::Round($totalCPU, 2))%"

$memoryPressure = [math]::Round((1 - ($freeRAM / $ram)) * 100, 1)
$memColor = if ($memoryPressure -gt 90) { "Red" } elseif ($memoryPressure -gt 80) { "Yellow" } else { "Green" }
Write-Report "Memory Pressure: $memoryPressure%" $memColor

$diskQueue = (Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length").CounterSamples.CookedValue
$diskColor = if ($diskQueue -gt 5) { "Red" } elseif ($diskQueue -gt 2) { "Yellow" } else { "Green" }
Write-Report "Disk Queue: $([math]::Round($diskQueue, 2))" $diskColor
Write-Report ""

# Recommendations
Write-Report "[6] RECOMMENDATIONS" "Yellow"
$issues = @()

if ($ram -lt 16) {
    $issues += "- RAM is below 16GB. Recommended: 32GB+ for Photoshop."
}
if ($freeRAM -lt 4) {
    $issues += "- Free RAM is low (<4GB). Close unnecessary applications."
}
if (-not $gpu -or $gpu.Name -like "*Intel*") {
    $issues += "- No dedicated GPU detected. Consider adding NVIDIA/AMD GPU."
}
$scratchDrive = $drives | Where-Object { $_.DeviceID -eq "C:" }
if ($scratchDrive -and $scratchDrive.FreeSpace / 1GB -lt 50) {
    $issues += "- C: drive has <50GB free. Free up space for scratch disk."
}
if ($ps -and $pf -gt 100) {
    $issues += "- High page faults detected. Increase RAM allocation in Photoshop."
}

if ($issues.Count -eq 0) {
    Write-Report "No critical issues detected! System is well-configured for Photoshop." "Green"
} else {
    foreach ($issue in $issues) {
        Write-Report $issue "Red"
    }
}

Write-Report ""
Write-Report "======================================" "Cyan"
Write-Report "Report saved to: $reportPath" "Cyan"
Write-Report "======================================" "Cyan"
```

**Usage:**
```powershell
# Run diagnostics
.\PS_Diagnostics.ps1

# Output saved to timestamped report file
# Review recommendations at bottom
```

---

## 7.2 Batch Performance Testing

**Script: Test filter performance across settings**
```powershell
# Photoshop Filter Performance Test
# Tests Gaussian Blur at different settings

$testResults = @()

Write-Host "Photoshop Performance Test Suite" -ForegroundColor Cyan
Write-Host "Testing Gaussian Blur performance...`n"

# Test configurations
$configs = @(
    @{ Name = "GPU Advanced"; GPU = $true; Mode = "Advanced" },
    @{ Name = "GPU Normal"; GPU = $true; Mode = "Normal" },
    @{ Name = "GPU Basic"; GPU = $true; Mode = "Basic" },
    @{ Name = "CPU Only"; GPU = $false; Mode = "Basic" }
)

foreach ($config in $configs) {
    Write-Host "Testing: $($config.Name)" -ForegroundColor Yellow
    
    # User must manually change Photoshop settings
    Write-Host "Please configure Photoshop:" -ForegroundColor Gray
    Write-Host "  Edit → Preferences → Performance" -ForegroundColor Gray
    if ($config.GPU) {
        Write-Host "  ☑ Use Graphics Processor" -ForegroundColor Gray
        Write-Host "  Advanced Settings → $($config.Mode)" -ForegroundColor Gray
    } else {
        Write-Host "  ☐ Use Graphics Processor (disable)" -ForegroundColor Gray
    }
    Write-Host "  Press Enter when ready..." -ForegroundColor Cyan
    Read-Host
    
    # Monitor during test
    Write-Host "Apply Filter → Blur → Gaussian Blur (500px radius)" -ForegroundColor Yellow
    Write-Host "Press Enter when filter completes..." -ForegroundColor Cyan
    
    $startTime = Get-Date
    Read-Host
    $endTime = Get-Date
    
    $duration = ($endTime - $startTime).TotalSeconds
    
    # Collect resource usage
    $ps = Get-Process Photoshop -ErrorAction SilentlyContinue
    $peakRAM = if ($ps) { [math]::Round($ps.PeakWorkingSet64 / 1GB, 2) } else { 0 }
    
    $result = [PSCustomObject]@{
        Configuration = $config.Name
        Duration_Seconds = [math]::Round($duration, 2)
        Peak_RAM_GB = $peakRAM
    }
    
    $testResults += $result
    Write-Host "Duration: $($result.Duration_Seconds)s | Peak RAM: $($result.Peak_RAM_GB)GB`n" -ForegroundColor Green
}

# Display results
Write-Host "`nTest Results Summary:" -ForegroundColor Cyan
$testResults | Format-Table -AutoSize

# Find fastest
$fastest = $testResults | Sort-Object Duration_Seconds | Select-Object -First 1
Write-Host "Fastest configuration: $($fastest.Configuration) ($($fastest.Duration_Seconds)s)" -ForegroundColor Green

# Export results
$csvPath = "C:\PS_Diagnostics\Performance_Test_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
$testResults | Export-Csv -Path $csvPath -NoTypeInformation
Write-Host "`nResults exported to: $csvPath" -ForegroundColor Cyan
```

---

## 7.3 Watchdog Script

**Background monitoring with alerts**
```powershell
# Photoshop Performance Watchdog
# Monitors and alerts on performance issues

param(
    [int]$CPUThreshold = 95,
    [int]$PageFaultThreshold = 100,
    [int]$CheckIntervalSeconds = 5
)

Write-Host "Photoshop Watchdog Started" -ForegroundColor Cyan
Write-Host "CPU Threshold: $CPUThreshold%" -ForegroundColor Yellow
Write-Host "Page Fault Threshold: $PageFaultThreshold/sec" -ForegroundColor Yellow
Write-Host "Monitoring every $CheckIntervalSeconds seconds`n" -ForegroundColor Yellow

$alertCount = 0

while ($true) {
    $ps = Get-Process Photoshop -ErrorAction SilentlyContinue
    
    if ($ps) {
        $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
        $pageFaults = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
        
        $timestamp = Get-Date -Format "HH:mm:ss"
        $alert = $false
        $message = "[$timestamp] "
        
        if ($cpu -gt $CPUThreshold) {
            $message += "⚠ CPU HIGH ($([math]::Round($cpu,1))%) "
            $alert = $true
        }
        
        if ($pageFaults -gt $PageFaultThreshold) {
            $message += "⚠ PAGE FAULTS HIGH ($([math]::Round($pageFaults,1))/s) "
            $alert = $true
        }
        
        if ($alert) {
            Write-Host $message -ForegroundColor Red
            $alertCount++
            
            # Play system beep
            [console]::beep(1000, 200)
            
            # Detailed info
            $ram = [math]::Round($ps.WorkingSet64 / 1GB, 2)
            Write-Host "  RAM: $ram GB | Threads: $($ps.Threads.Count)" -ForegroundColor Yellow
        } else {
            Write-Host "[$timestamp] OK - CPU: $([math]::Round($cpu,1))% | PageFaults: $([math]::Round($pageFaults,1))/s" -ForegroundColor Green
        }
    } else {
        Write-Host "[$((Get-Date).ToString('HH:mm:ss'))] Photoshop not running" -ForegroundColor Gray
    }
    
    Start-Sleep -Seconds $CheckIntervalSeconds
}
```

**Usage:**
```powershell
# Default thresholds
.\PS_Watchdog.ps1

# Custom thresholds
.\PS_Watchdog.ps1 -CPUThreshold 90 -PageFaultThreshold 50 -CheckIntervalSeconds 3

# Run in background (separate PowerShell window)
Start-Process powershell -ArgumentList "-NoExit", "-File", "C:\PerformanceScripts\PS_Watchdog.ps1"
```

---

## 8. Quick Reference

### Essential Commands Cheat Sheet
```powershell
# QUICK CHECKS

# Photoshop status
Get-Process Photoshop | Select-Object CPU, WorkingSet64, Threads

# Current CPU usage
Get-Counter "\Process(Photoshop)\% Processor Time"

# Page faults
Get-Counter "\Process(Photoshop)\Page Faults/sec"

# Disk queue
Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length"

# GPU usage (if available)
Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage"

# Free RAM
[math]::Round((Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory/1MB,2)

# TRACES

# Start WPR trace
wpr -start CPU -start GPU -start DiskIO -filemode

# Stop WPR trace
wpr -stop output.etl

# Open in WPA
wpa output.etl

# PROCESS MONITOR

# Launch and filter
procmon /AcceptEula /Minimized /LoadConfig filter.pms /BackingFile capture.pml

# CLEANUP

# Photoshop temp files
Remove-Item "$env:LOCALAPPDATA\Temp\Photoshop*" -Recurse -Force

# Clear font cache
Remove-Item "$env:WinDir\ServiceProfiles\LocalService\AppData\Local\FontCache*" -Force
```

### Troubleshooting Flow Diagram (Text)
```
SLOW PERFORMANCE
       |
       v
Check Efficiency Indicator
       |
       |---> <95%? -----> [RAM ISSUE]
       |                      |
       |                      |---> Increase RAM allocation
       |                      |---> Close other apps
       |                      `---> Check scratch disk space
       |
       `---> >95%
               |
               v
        Check GPU Usage
               |
               |---> <30%? -----> [GPU NOT USED]
               |                       |
               |                       |---> Enable in Preferences
               |                       `---> Update GPU drivers
               |
               `---> >30%
                       |
                       v
                Check CPU Usage
                       |
                       |---> >85%? -----> [CPU BOTTLENECK]
                       |                       |
                       |                       |---> Reduce workload
                       |                       `---> Upgrade CPU
                       |
                       `---> <85%
                               |
                               v
                        Check Disk Activity
                               |
                               |---> 100%? -----> [DISK BOTTLENECK]
                               |                       |
                               |                       `---> Upgrade to SSD
                               |
                               `---> Normal
                                       |
                                       v
                                [SOFTWARE ISSUE]
                                       |
                                       |---> Update Photoshop
                                       |---> Reset preferences
                                       `---> Check for plugin conflicts
```

### Common Issues Quick Fix Table

| Symptom | Quick Fix | Time to Implement |
|---------|-----------|-------------------|
| Brush lag | GPU preferences → Advanced | 30 seconds |
| Low efficiency | Increase RAM allocation to 75% | 30 seconds |
| Startup slow | Reset preferences (Ctrl+Alt+Shift) | 1 minute |
| Filter slow | Enable GPU acceleration | 30 seconds |
| "Out of RAM" error | Purge (Edit → Purge → All) | 10 seconds |
| Scratch disk full | Free up 50GB+ on scratch drive | 5 minutes |
| GPU not detected | Update GPU drivers | 10 minutes |
| High page faults | Close other applications | 1 minute |

---

## Appendix: Tool Download Links

- **Windows Performance Toolkit**: https://learn.microsoft.com/windows-hardware/get-started/adk-install
- **Sysinternals Suite**: https://learn.microsoft.com/sysinternals/downloads/
- **Process Explorer**: https://learn.microsoft.com/sysinternals/downloads/process-explorer
- **Process Monitor**: https://learn.microsoft.com/sysinternals/downloads/procmon
- **RAMMap**: https://learn.microsoft.com/sysinternals/downloads/rammap
- **MSI Afterburner**: https://www.msi.com/Landing/afterburner
- **GPU-Z**: https://www.techpowerup.com/gpuz/
- **HWiNFO64**: https://www.hwinfo.com/download/
- **LatencyMon**: https://www.resplendence.com/latencymon
- **CrystalDiskMark**: https://crystalmark.info/en/software/crystaldiskmark/
- **DDU (Display Driver Uninstaller)**: https://www.guru3d.com/files-details/display-driver-uninstaller-download.html

---

## END OF GUIDE

This comprehensive guide covers Windows performance troubleshooting for Photoshop from installation through advanced analysis. Use the table of contents to navigate to specific sections as needed.

For automated diagnostics, use the PowerShell scripts in Section 7.

For quick troubleshooting, refer to Section 8 Quick Reference.

---

**Document Version**: 1.0 Complete
**Last Updated**: 2025
**Total Sections**: 8
**Total Word Count**: ~28,000 words
# Windows Performance Troubleshooting - Part One

## Table of Contents
1. [Tool Installation & Setup](#1-tool-installation--setup)
2. [Understanding Performance Metrics](#2-understanding-performance-metrics)
3. [Complete Diagnostic Workflows](#3-complete-diagnostic-workflows)
4. [Tool-Specific Deep Dives](#4-tool-specific-deep-dives)
5. [Scenario-Based Troubleshooting](#5-scenario-based-troubleshooting)
6. [Advanced Analysis Techniques](#6-advanced-analysis-techniques)
7. [Automation & Scripting](#7-automation--scripting)

---

# 1. Tool Installation & Setup

## 1.1 Windows Performance Toolkit (WPT)

### Installation Steps

**Method 1: Standalone (Recommended)**
1. Visit: https://learn.microsoft.com/windows-hardware/get-started/adk-install
2. Download "Windows ADK Installer"
3. Run installer
4. Select ONLY "Windows Performance Toolkit" (~50MB)
5. Complete installation

**Verify Installation:**
```cmd
wpr -help
wpa
```

**Add to PATH (if needed):**
```cmd
setx PATH "%PATH%;C:\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit"
```

### Post-Installation Configuration

**Run as Administrator:**
- Always run WPR/WPA with admin rights
- Right-click → "Run as administrator"

**Create Desktop Shortcuts:**
```powershell
$WshShell = New-Object -comObject WScript.Shell
$Shortcut = $WshShell.CreateShortcut("$Home\Desktop\WPA.lnk")
$Shortcut.TargetPath = "C:\Program Files (x86)\Windows Kits\10\Windows Performance Toolkit\wpa.exe"
$Shortcut.Save()
```

---

## 1.2 Sysinternals Suite Installation

### Complete Suite Setup

**Download:**
1. Visit: https://learn.microsoft.com/sysinternals/downloads/
2. Download "Sysinternals Suite" ZIP
3. Extract to: `C:\Tools\Sysinternals`

**Add to System PATH:**
```cmd
setx PATH "%PATH%;C:\Tools\Sysinternals" /M
```

**Accept EULA for All Tools:**
```cmd
cd C:\Tools\Sysinternals
for %f in (*.exe) do %f -accepteula
```

### Individual Tool Setup

**Process Explorer Configuration:**
1. Launch as Administrator
2. **Options → Replace Task Manager** (recommended)
3. **View → Select Columns:**
   - Process Performance tab: CPU Usage, Private Bytes
   - Process GPU tab: GPU Usage, GPU Dedicated Bytes, GPU System Bytes
   - Process I/O tab: I/O Total, I/O Reads, I/O Writes
4. **View → System Information** (verify GPU detection)
5. **File → Save As** (saves configuration)

**Process Monitor Configuration:**
1. Launch as Administrator
2. **Filter → Filter:**
   ```
   Process Name | is | Photoshop.exe | Include | Add
   Duration | is greater than | 0.100 | Include | Add
   ```
3. **Filter → Save Filter** → Name: "Photoshop_Slow_Ops.pms"
4. Configure columns:
   - Right-click header → Add: Duration, Path, Result, Detail
5. **File → Backing Files** → Set to SSD location (for large captures)

**RAMMap Setup:**
- No configuration needed
- Always run as Administrator
- Use F5 to refresh

---

## 1.3 Hardware Monitoring Tools

### MSI Afterburner + RivaTuner Setup

**Installation:**
1. Download from MSI website
2. Install with RivaTuner Statistics Server (RTSS) - check during install
3. Launch MSI Afterburner

**Configuration:**
1. **Settings (gear icon) → Monitoring tab:**
   - GPU usage: ☑ Show in On-Screen Display
   - GPU temperature: ☑ Show in On-Screen Display
   - Memory usage: ☑ Show in On-Screen Display
   - Core clock: ☑ Show in On-Screen Display
   - Frame rate: ☑ Show in On-Screen Display

2. **RivaTuner Statistics Server:**
   - Launch RTSS from system tray
   - Application detection level: High
   - On-Screen Display position: Top right
   - Update rate: 1000 ms (1 second)

**Test Display:**
- Launch Photoshop
- Overlay should appear showing GPU stats

---

### HWiNFO64 Setup

**Installation:**
1. Download from hwinfo.com
2. Install or use portable version
3. Launch → Sensors-only mode

**Configure Sensors:**
1. **Settings → Layout:**
   - Show sensors: GPU, CPU, Motherboard
2. **Right-click sensor window → Customize:**
   - Add to view:
     - GPU Core Temperature
     - GPU Core Clock (all)
     - GPU Core Load
     - GPU Memory Used/Free
     - GPU Power
     - CPU Package Temperature
     - CPU Core Clocks (all cores)

**Enable Logging:**
1. Click log icon (disk with arrow)
2. Location: `C:\HWiNFO_Logs\`
3. Format: CSV
4. Interval: 1000 ms
5. Start logging

---

### GPU-Z Setup

**Installation:**
1. Download from techpowerup.com
2. No installation needed (portable)
3. Launch GPU-Z.exe

**Configure Logging:**
1. **Sensors tab**
2. **Click dropdown (bottom right) → Log to file**
3. Location: `C:\GPU_Logs\gpuz_log.txt`
4. Logging active (shows in title bar)

**Useful Sensors:**
- GPU Load
- Memory Used
- Temperature
- Core Clock
- Memory Clock
- GPU Power
- PerfCap Reason (shows throttling cause)

---

## 1.4 PowerShell Environment Setup

### Enable Script Execution

**Run PowerShell as Administrator:**
```powershell
# Allow scripts for current user
Set-ExecutionPolicy Bypass -Scope CurrentUser

# Verify
Get-ExecutionPolicy -List
```

### Create Scripts Directory

```powershell
# Create folder structure
New-Item -Path "C:\PerformanceScripts" -ItemType Directory -Force
New-Item -Path "C:\PS_Diagnostics" -ItemType Directory -Force
New-Item -Path "C:\PS_Traces" -ItemType Directory -Force

# Set permissions (optional)
icacls "C:\PerformanceScripts" /grant ${env:USERNAME}:F
```

### Test Basic Commands

```powershell
# Test process monitoring
Get-Process Photoshop -ErrorAction SilentlyContinue | Format-List *

# Test performance counters
Get-Counter "\Processor(_Total)\% Processor Time"

# Test GPU counters (may not work on all systems)
Get-Counter "\GPU Engine(*)\Utilization Percentage" -ErrorAction SilentlyContinue

# List available GPU counters
Get-Counter -ListSet "GPU*" | Select-Object CounterSetName, Paths
```

### Create Reusable Functions

**Save as: `C:\PerformanceScripts\PS_Functions.ps1`**

```powershell
# Reusable Photoshop monitoring functions

function Get-PhotoshopStatus {
    $ps = Get-Process Photoshop -ErrorAction SilentlyContinue
    if($ps) {
        [PSCustomObject]@{
            Running = $true
            PID = $ps.Id
            CPU_Seconds = [math]::Round($ps.CPU, 2)
            RAM_GB = [math]::Round($ps.WorkingSet64/1GB, 2)
            Threads = $ps.Threads.Count
            StartTime = $ps.StartTime
        }
    } else {
        [PSCustomObject]@{
            Running = $false
        }
    }
}

function Get-PhotoshopCPU {
    $counter = Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue
    if($counter) {
        [math]::Round($counter.CounterSamples.CookedValue, 2)
    } else {
        "N/A"
    }
}

function Get-PhotoshopPageFaults {
    $counter = Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue
    if($counter) {
        [math]::Round($counter.CounterSamples.CookedValue, 2)
    } else {
        "N/A"
    }
}

function Get-PhotoshopIO {
    $counter = Get-Counter "\Process(Photoshop)\IO Data Bytes/sec" -ErrorAction SilentlyContinue
    if($counter) {
        [math]::Round($counter.CounterSamples.CookedValue/1MB, 2)
    } else {
        "N/A"
    }
}

# Usage:
# . C:\PerformanceScripts\PS_Functions.ps1
# Get-PhotoshopStatus
# Get-PhotoshopCPU
```

---

# 2. Understanding Performance Metrics

## 2.1 CPU Metrics Deep Dive

### CPU Usage Percentage

**What it Measures:**
- Percentage of time CPU spends executing non-idle threads
- Aggregated across all cores OR per-core

**How to Read:**
```powershell
# Total CPU usage
Get-Counter "\Processor(_Total)\% Processor Time"

# Per-core usage
Get-Counter "\Processor(*)\% Processor Time"
```

**Interpretation:**

| Range | Status | Action |
|-------|--------|--------|
| 0-40% | Normal/Idle | No action needed |
| 40-70% | Moderate load | Monitor during tasks |
| 70-85% | High load | Check if sustained |
| 85-100% | Bottleneck | Investigate cause |

**Photoshop-Specific:**
- Filters: 70-100% CPU normal during processing
- Brush: 30-60% CPU normal, >80% indicates issue
- Idle: Should be <10% CPU

**Single-Core vs Multi-Core:**

Check individual cores:
```powershell
Get-Counter "\Processor(*)\% Processor Time" | 
    Select-Object -ExpandProperty CounterSamples | 
    Format-Table InstanceName, CookedValue -AutoSize
```

**Pattern Recognition:**
```
One core at 100%, others at 10-20%:
→ Single-threaded bottleneck
→ Photoshop operation not multi-threaded

All cores at 90-100%:
→ True CPU bottleneck
→ Consider CPU upgrade or reduce workload

All cores at 30-50%:
→ Not CPU bottlenecked
→ Look for GPU or I/O issues
```

---

### Thread Count

**What it Measures:**
- Number of execution threads in Photoshop process

**How to Check:**
```powershell
(Get-Process Photoshop).Threads.Count
```

**Normal Ranges:**
- Idle: 20-40 threads
- Active work: 40-80 threads
- Heavy processing: 80-150 threads

**High Thread Count Issues:**

If threads >150:
- Possible thread leak
- Plugin creating excessive threads
- Check with Process Explorer → Properties → Threads tab
- Sort by CPU Time to find busy threads

---

### Context Switches

**What it Measures:**
- How often CPU switches between threads
- High rate indicates thread contention

**How to Check:**
```powershell
Get-Counter "\Thread(*photoshop*)\Context Switches/sec"
```

**Interpretation:**
- <5,000/sec: Normal
- 5,000-10,000/sec: Moderate contention
- >10,000/sec: High contention (performance impact)

**Causes of High Context Switches:**
- Too many threads competing for CPU
- Lock contention (threads waiting for resources)
- I/O blocking (threads waiting for disk/network)

---

## 2.2 Memory Metrics Deep Dive

### Working Set (RAM Usage)

**What it Measures:**
- Physical RAM currently used by Photoshop

**How to Check:**
```powershell
$ps = Get-Process Photoshop
"Working Set: $([math]::Round($ps.WorkingSet64/1GB, 2)) GB"
"Private Bytes: $([math]::Round($ps.PrivateMemorySize64/1GB, 2)) GB"
```

**Difference Explained:**
- **Working Set**: RAM currently in physical memory
- **Private Bytes**: Total committed memory (may include paged)
- **Virtual Bytes**: Total virtual address space

**Normal Values (32GB RAM system):**
```
Photoshop Preferences set to 70% (22GB):
Working Set: 18-22 GB (normal for large docs)
Private Bytes: Should be close to Working Set
Difference >4GB: Heavy paging occurring
```

---

### Page Faults

**What it Measures:**
- Accesses to memory not currently in RAM

**Types:**
1. **Soft Page Fault**: Data in RAM cache (fast, <1μs)
2. **Hard Page Fault**: Data on disk (slow, 1-10ms)

**How to Check:**
```powershell
# Total page faults
Get-Counter "\Process(Photoshop)\Page Faults/sec"

# Hard faults (disk access)
Get-Counter "\Memory\Page Reads/sec"
```

**Interpretation:**

| Page Faults/sec | Status | Meaning |
|----------------|--------|---------|
| 0-50 | Excellent | All data in RAM |
| 50-100 | Good | Minimal paging |
| 100-500 | Warning | Some RAM pressure |
| >500 | Critical | Severe RAM shortage |

**Action Based on Faults:**
```
>100 hard faults/sec:
1. Check RAM allocation in Photoshop
2. Close other applications
3. Reduce document size
4. Add more physical RAM
```

---

### Photoshop Efficiency Indicator

**Location:** Bottom-left of document window (click dropdown)

**What it Measures:**
- Percentage of operations completed in RAM vs scratch disk

**Values:**
```
100%: Perfect - all in RAM
95-99%: Excellent - minimal scratch use
90-94%: Good - some scratch use
85-89%: Fair - noticeable slowdown
<85%: Poor - heavy scratch disk usage
```

**Troubleshooting Low Efficiency:**

**Step 1: Check RAM allocation**
```
Edit → Preferences → Performance
Let Photoshop Use: Increase to 75-85%
```

**Step 2: Check scratch disk**
```
Preferences → Performance → Scratch Disks
Ensure fastest drive is selected
Ensure 20GB+ free space
```

**Step 3: Reduce memory usage**
```
Edit → Purge → All
Reduce History States
Close unused documents
```

---

## 2.3 GPU Metrics Deep Dive

### GPU Utilization

**What it Measures:**
- Percentage of GPU compute units in use

**How to Check:**

**Method 1: Task Manager**
- Performance tab → GPU → 3D graph
- Shows Photoshop's GPU usage

**Method 2: PowerShell**
```powershell
Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage"
```

**Method 3: GPU-Z**
- Sensors tab → GPU Load value

**Method 4: NVIDIA (nvidia-smi)**
```cmd
nvidia-smi --query-gpu=utilization.gpu --format=csv --loop=1
```

**Interpretation:**

| GPU Usage | CPU Usage | Diagnosis |
|-----------|-----------|-----------|
| 80-100% | 40-60% | GPU bottleneck |
| 20-40% | 80-100% | CPU bottleneck |
| <20% | <50% | Neither bottleneck (check disk) |
| Fluctuating | Stable | GPU memory thrashing |

---

### VRAM (Video Memory) Usage

**What it Measures:**
- GPU memory used for textures, layer cache, acceleration

**How to Check:**

**Task Manager:**
```
Performance → GPU → Dedicated GPU Memory
Shows: X.X GB / Y.Y GB
```

**GPU-Z:**
```
Sensors tab → Memory Used
Also shows: Memory Free, Memory Load %
```

**NVIDIA:**
```cmd
nvidia-smi --query-gpu=memory.used,memory.total --format=csv
```

**Thresholds:**

| VRAM Usage | Status | Action |
|------------|--------|--------|
| <70% | Healthy | No action |
| 70-85% | Elevated | Monitor |
| 85-95% | High | Reduce document size |
| >95% | Critical | VRAM bottleneck |

**VRAM Bottleneck Symptoms:**
- GPU usage fluctuates wildly (20% → 80% → 30%)
- Stuttering during GPU-accelerated operations
- "Out of GPU memory" errors
- Performance worse than CPU mode

---

### GPU Clock Speed

**What it Measures:**
- Actual GPU core frequency (MHz or GHz)

**Typical Behavior:**
- **Base Clock**: Minimum guaranteed (e.g., 1,350 MHz)
- **Boost Clock**: Maximum (e.g., 1,900 MHz)
- **Actual Clock**: Real-time (varies with load/temp)

**How to Check:**

**GPU-Z:**
```
Graphics Card tab: GPU Clock (current)
Sensors tab: GPU Clock (real-time monitoring)
```

**HWiNFO:**
```
GPU Core Clock sensor
```

**NVIDIA:**
```cmd
nvidia-smi --query-gpu=clocks.sm --format=csv --loop=1
```

**Throttling Detection:**

```
Normal behavior:
Idle: 300-600 MHz (power saving)
Load: Boost to 1800-1900 MHz
Sustained: Stays at 1750-1850 MHz

Thermal throttling:
Load: Boosts to 1900 MHz
Temperature rises to 83C
Clock drops to 1650 MHz
Further temp rise → 1500 MHz

Power limit throttling:
Load: Boosts to 1900 MHz
Hits 100% TDP (e.g., 220W)
Clock drops to 1750 MHz to stay under limit
```

**Check Throttle Reason (GPU-Z):**
```
Sensors → PerfCap Reason:
- Idle: Normal (not under load)
- VRel: Voltage reliability limit
- Pwr: Power limit
- Thrm: Thermal limit
- vOp: Operating voltage limit
```

---

### GPU Temperature

**Ranges:**

| Temperature | Status | Action |
|-------------|--------|--------|
| 30-50°C | Idle | Normal |
| 50-70°C | Light load | Normal |
| 70-80°C | Heavy load | Normal |
| 80-85°C | Hot | Monitor |
| 85-90°C | Throttling | Improve cooling |
| >90°C | Critical | Immediate action |

**How to Check:**

**GPU-Z:** Sensors → GPU Temperature
**HWiNFO:** GPU Temperature sensor
**NVIDIA:** `nvidia-smi --query-gpu=temperature.gpu`

**Cooling Solutions:**

```
Progressive actions:
1. Clean dust from GPU cooler (compressed air)
2. Increase case fan speeds (BIOS/software)
3. Improve case airflow (cable management, fan placement)
4. Reapply thermal paste (advanced)
5. Aftermarket cooler (advanced)
6. Undervolt GPU (reduces heat without much performance loss)
```

---

## 2.4 Disk Metrics Deep Dive

### Disk Queue Length

**What it Measures:**
- Number of I/O requests waiting to be processed

**How to Check:**

**Resource Monitor:**
```
Win+R → resmon → Disk tab → Queue Length column
```

**PowerShell:**
```powershell
Get-Counter "\PhysicalDisk(*)\Current Disk Queue Length"
```

**Interpretation:**

| Queue Length | Status | Meaning |
|--------------|--------|---------|
| 0-1 | Optimal | No waiting |
| 1-2 | Normal | Minimal wait |
| 2-5 | Elevated | Some contention |
| >5 | Bottleneck | Disk can't keep up |

**Photoshop Context:**
```
During save (expected high queue):
Queue: 3-6 (normal, sequential write)
Active Time: 80-100%

During scratch disk use (concerning):
Queue: 8-15 (bottleneck)
Active Time: 100%
Duration: Sustained minutes
→ RAM shortage + slow disk
```

---

### Active Time Percentage

**What it Measures:**
- Percentage of time disk is processing requests

**How to Check:**

**Task Manager:**
```
Performance → Disk → Active time %
```

**Resource Monitor:**
```
Disk tab → Disk Activity graph
```

**Interpretation:**

| Active Time | Status | Action |
|-------------|--------|--------|
| 0-30% | Light use | Normal |
| 30-70% | Moderate | Normal |
| 70-90% | Heavy | Monitor |
| 90-100% | Saturated | Bottleneck |

**Pattern Analysis:**
```
Spiky (0% → 100% → 0%):
→ Normal (saves, cache writes)
→ No action needed

Sustained 100% for minutes:
→ Bottleneck (scratch disk or HDD)
→ Check which process
→ If Photoshop: RAM shortage likely
→ If other process: Background task
```

---

### Transfer Rate

**What it Measures:**
- Data throughput (MB/s or GB/s)

**How to Check:**

**Task Manager:**
```
Performance → Disk → Read/Write speed
```

**Resource Monitor:**
```
Disk tab → Disk Activity → Read/Write B/sec
```

**Expected Speeds:**

| Drive Type | Sequential Read | Sequential Write | Random 4K |
|------------|----------------|------------------|-----------|
| HDD 7200RPM | 100-200 MB/s | 100-180 MB/s | 0.5-2 MB/s |
| SATA SSD | 500-550 MB/s | 450-520 MB/s | 20-40 MB/s |
| NVMe Gen3 | 2000-3500 MB/s | 1500-3000 MB/s | 40-80 MB/s |
| NVMe Gen4 | 5000-7000 MB/s | 4000-6500 MB/s | 80-150 MB/s |

**Photoshop Scratch Disk Requirements:**
```
Minimum: 200 MB/s (fast HDD acceptable but slow)
Recommended: 500 MB/s (SATA SSD minimum)
Optimal: 2000+ MB/s (NVMe for large files)
```

**Test Disk Speed (CrystalDiskMark):**
```
1. Download CrystalDiskMark
2. Select scratch disk drive
3. Run test (5 runs, 1GB)
4. Check SEQ1M Q8T1 (sequential speed)

Results interpretation:
<200 MB/s: Too slow for scratch, upgrade to SSD
200-500 MB/s: Acceptable, but limiting
>500 MB/s: Good for Photoshop scratch
>2000 MB/s: Excellent, no disk bottleneck
```

---

## 2.5 System Latency Metrics

### DPC (Deferred Procedure Call) Latency

**What it is:**
- Kernel-mode functions executed at high priority
- Drivers use DPCs for interrupt handling
- High DPC latency = system unresponsiveness

**How to Check:**

**LatencyMon:**
```
1. Download from resplendence.com
2. Launch LatencyMon
3. Click "Start"
4. Let run during Photoshop use
5. Check results after 2-5 minutes
```

**Interpretation:**

| Highest DPC | Status | Impact |
|-------------|--------|--------|
| <100 μs | Excellent | No issues |
| 100-500 μs | Good | Minor impact |
| 500-1000 μs | Warning | Noticeable lag |
| >1000 μs | Critical | Severe problems |

**Results Display:**
```
LatencyMon shows:
"Your system appears suitable for real-time audio"
→ DPC latency is fine

"Your system is having trouble..."
→ DPC latency is problematic
→ Check Drivers tab for culprit
```

**Common Culprits:**

| Driver | Typical Issue | Solution |
|--------|---------------|----------|
| Netwtw10.sys | Intel Wi-Fi driver | Update or disable Wi-Fi |
| nvlddmkm.sys | NVIDIA driver | Update GPU driver |
| HDAudBus.sys | Audio driver | Update audio driver |
| storport.sys | Storage driver | Update chipset drivers |
| tcpip.sys | Network stack | Check network adapter |

**Fix High DPC Latency:**
```
1. Identify driver from LatencyMon "Drivers" tab
2. Device Manager → Find device
3. Update driver (Windows Update or manufacturer)
4. If still high: Roll back to previous version
5. Last resort: Disable device temporarily
```

---

### ISR (Interrupt Service Routine) Time

**What it is:**
- Time spent handling hardware interrupts
- Should be minimal (<1% CPU time)

**How to Check:**
- LatencyMon → Stats tab → ISR execution time

**Interpretation:**
```
ISR time <1% CPU: Normal
ISR time 1-5%: Elevated (monitor)
ISR time >5%: Problem (identify device)
```

**High ISR Causes:**
- Malfunctioning hardware
- Driver bugs
- Excessive interrupts (e.g., high polling rate mouse at 8000Hz)
- USB device issues

---

### Frame Time Analysis

**What it Measures:**
- Time from frame render to display (input lag)
- Critical for smooth brush response

**Target:**
- 60 FPS = 16.67ms per frame
- Consistent frame time = smooth feel
- Variable frame time = stuttery

**How to Measure:**

**PresentMon (Microsoft tool):**
```cmd
# Download from GitHub: GameTechDev/PresentMon
PresentMon-x64.exe -process_name Photoshop.exe -timed 60 -output_file frames.csv
```

**During capture:**
- Use Photoshop normally
- Paint with brush
- Apply filters
- Switch tools

**Analysis (Excel):**
```
Open frames.csv
Key column: MsBetweenPresents (frame time in ms)

Smooth performance:
16.5, 16.7, 16.6, 16.8, 16.5
Avg: 16.6ms (60 FPS)
StdDev: 0.1ms (very consistent)

Stuttery performance:
16.5, 16.7, 45.2, 16.6, 62.1, 16.5
Avg: 28.9ms (still ~35 FPS average)
StdDev: 18.5ms (HIGH variance = stutter)
```

**High Variance Causes:**
- DPC latency spikes
- Disk I/O blocking
- GPU driver timeout
- Background process CPU bursts
- Memory paging

---

# 3. Complete Diagnostic Workflows

## 3.1 Quick Check (30 Seconds)

### Step-by-Step Immediate Assessment

**A. Photoshop Efficiency Indicator**

```
1. Open any document in Photoshop
2. Look at bottom-left of window
3. Click dropdown menu
4. Select "Efficiency"

Reading:
100%: ✓ All operations in RAM (perfect)
95-99%: ✓ Minimal scratch use (good)
90-94%: ⚠ Some scratch use (monitor)
<90%: ✗ Heavy scratch use (problem)

If <100%: RAM or scratch disk issue
```

**B. System Info Check**

```
Help → System Info (or Ctrl+Alt+Shift+I)

Check GPU section:
GPU: [Should show your GPU model]
OpenCL: Available [Should say "Yes"]
OpenGL Drawing: Enabled
Metal/DirectX: Enabled

If GPU shows "Unavailable":
→ GPU not detected
→ Driver issue or GPU disabled
```

**C. Performance Preferences**

```
Edit → Preferences → Performance

Memory Usage:
Slider should be at 70-85%
If lower: Wasting available RAM
If higher: System instability risk

Graphics Processor:
☑ Use Graphics Processor (should be checked)
Advanced Settings: Normal or Advanced mode

Scratch Disks:
First disk: Should be fastest SSD
Free space: Should show 20GB+ available
```

**D. Quick Brush Test**

```
1. File → New: 3000x3000px, RGB, 8-bit
2. Brush Tool (B)
3. Size: 500px, Hardness: 0%
4. Paint 5-6 quick strokes

Observe:
Smooth, no lag: ✓ System performing well
Slight lag: ⚠ Investigate further
Severe lag: ✗ Significant bottleneck present
```

---

## 3.2 Resource Overview (1-2 Minutes)

### Multi-Tool Quick Scan

**A. Task Manager Overview**

```
Ctrl+Shift+Esc → Performance tab

CPU:
Check utilization %
If >80% sustained: CPU bottleneck

Memory:
Check "In use" vs "Available"
If Available <4GB: RAM shortage

Disk:
Check Active time %
If 100% sustained: Disk bottleneck

GPU (if available):
Check 3D utilization
Check Dedicated GPU memory
If VRAM maxed: VRAM bottleneck
```

**B. Resource Monitor Deep Dive**

```
Win+R → resmon → Enter
(Or: Task Manager → Performance → Open Resource Monitor)

CPU Tab:
Find Photoshop.exe
Note CPU % and Threads count

Memory Tab:
Find Photoshop.exe
Check Working Set (RAM usage)
Check Hard Faults/sec
>100 faults/sec = RAM problem

Disk Tab (MOST IMPORTANT):
Watch during Photoshop operation
Disk Queue Length: Should be <2
Active Time %: Sustained 100% = problem
Note which physical disk is busy

Network Tab:
Usually not relevant for Photoshop
Unless using cloud storage or network files
```

**C. Quick PowerShell Check**

```powershell
# Open PowerShell, run:
$ps = Get-Process Photoshop -ErrorAction SilentlyContinue
if($ps) {
    Write-Host "Photoshop Status:" -ForegroundColor Cyan
    Write-Host "CPU Time: $([math]::Round($ps.CPU, 2))s total"
    Write-Host "RAM: $([math]::Round($ps.WorkingSet64/1GB, 2)) GB"
    Write-Host "Threads: $($ps.Threads.Count)"
    
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    Write-Host "CPU%: $([math]::Round($cpu, 2))%"
    
    $pf = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    Write-Host "Page Faults: $([math]::Round($pf, 2))/sec" -ForegroundColor $(if($pf -gt 100){"Red"}else{"Green"})
} else {
    Write-Host "Photoshop not running" -ForegroundColor Red
}
```

**D. GPU Monitoring Setup**

```
Launch GPU-Z:
1. Sensors tab
2. Observe while using Photoshop:
   - GPU Load %
   - Memory Used / Total
   - Temperature
   - Core Clock

Or MSI Afterburner:
Enable OSD to see live stats overlay

During Photoshop work, note:
GPU high (>70%): GPU being utilized
GPU low (<30%): CPU bottleneck or GPU not enabled
VRAM >90%: VRAM bottleneck
Temp >80°C: Thermal issue
```

---

## 3.3 WPR Trace Capture (2-5 Minutes)

### Complete Trace Workflow

**Step 1: Preparation**

```
1. Close unnecessary applications
2. Ensure 2GB+ free disk space for trace
3. Have Photoshop open with problem document
4. Know exact steps to reproduce issue
```

**Step 2: Start Trace**

```cmd
# Open Command Prompt as Administrator
Win+X → Command Prompt (Admin)

# Start recording
wpr -start CPU -start GPU -start DiskIO -filemode

Expected output:
"Microsoft Windows Performance Recorder
Recording started."
```

**Alternative: Comprehensive Trace**

```cmd
# For more detailed analysis
wpr -start GeneralProfile -start GPU -filemode

GeneralProfile includes:
- CPU scheduling
- Disk I/O
- File I/O
- Registry access
- Network activity
```

**Step 3: Reproduce Issue**

```
1. Switch to Photoshop
2. Perform exact operation that causes lag:
   - Apply specific filter
   - Use laggy brush
   - Open/save large file
   - Any reproducible problem
3. Continue for 10-60 seconds
   (Longer capture = larger file)
4. Note exact time of worst lag
```

**Step 4: Stop Trace**

```cmd
# Return to Command Prompt
wpr -stop C:\PS_Traces\photoshop_lag.etl

Process:
- "Merging traces..." (10-30 seconds)
- "100% complete"
- "Saving merged trace..."
Done!
```

**File Size Expectations:**
```
30-second trace: ~200-500 MB
1-minute trace: ~500 MB - 1 GB
5-minute trace: ~2-5 GB
```

**Step 5: Open in WPA**

```cmd
# Launch Windows Performance Analyzer
wpa C:\PS_Traces\photoshop_lag.etl

Loading:
- Parsing... (10-60 seconds depending on size)
- Applying symbols (optional, can skip)
- Ready for analysis
```

---

## 3.4 WPA Analysis

### Essential Graphs for Photoshop

**Graph 1: CPU Usage (Precise)**

```
1. Graph Explorer (left panel) → Computation
2. Drag "CPU Usage (Precise)" to analysis area

Configure:
- Preset: "Utilization by Process, Thread, Stack"
- Find Photoshop.exe in process list
- Expand to see threads

Look for:
- Thread with high CPU% during lag
- Stack trace showing which function
- Long-running operations (>100ms)

Interpretation:
High CPU in single thread: Single-threaded bottleneck
High CPU in multiple threads: Multi-threaded, might be normal
Low CPU with lag: Not CPU bottleneck
```

**Graph 2: GPU Hardware Queue**

```
Computation → GPU Hardware Queue

Shows:
- GPU command submission
- Queue depth (pending commands)
- Packet duration (execution time)

Look for:
Queue depth 0-1 + gaps: CPU not feeding GPU
Queue depth 10+: GPU overloaded
Long packet duration: GPU-intensive operation

Zoom to lag timeframe:
- Select time range
- Ctrl+Z to zoom
- Examine pattern during problem
```

**Graph 3: Disk Usage**

```
Storage → Disk Usage

Preset: "Disk Offset by Process, Thread, Path, Stack"

Find Photoshop.exe:
- Expand to see disk operations
- Check "Disk Service Time" column
- Look at "Path Name" (which file EOF
wc -l /home/claude/Windows_Performance_Detailed_COMPLETE.md
# Windows Performance Troubleshooting - Complete Detailed Guide (Continued)

## 3.4 WPA Analysis (Continued)

**Graph 3: Disk Usage**
```
Storage → Disk Usage

Preset: "Disk Offset by Process, Thread, Path, Stack"

Find Photoshop.exe:
- Expand to see disk operations
- Check "Disk Service Time" column
- Look at "Path Name" (which files accessed)
- Sort by "Size" to find large operations

Interpretation:
Many small reads (<1MB): Cache misses or fragmentation
Large sequential writes (>100MB): Save operations (normal)
High service time (>100ms): Slow disk or queue backlog
Path shows scratch disk: RAM shortage forcing disk use
```

**Pattern Recognition in Disk Graph:**
```
Normal save operation:
├─ Sequential writes to .psd file
├─ Service time: 10-50ms per operation
├─ Total duration: 2-10 seconds
└─ Pattern: Continuous writes, no gaps

RAM shortage pattern:
├─ Random reads/writes to scratch disk
├─ Service time: 50-200ms per operation
├─ Total duration: Throughout work session
├─ Pattern: Constant activity, small chunks
└─ Action: Increase RAM allocation

Disk bottleneck:
├─ Queue length >5 visible in timeline
├─ Service time >500ms
├─ Multiple processes competing
└─ Action: Upgrade to SSD or reduce background I/O
```

**Graph 4: File I/O**
```
Storage → File I/O

Shows higher-level file operations:
- CreateFile, ReadFile, WriteFile
- Useful for identifying which files cause delays

Configure:
- Group by: Process Name, File Path
- Sort by: Duration (descending)

Look for:
- Operations >1 second duration
- Repeated access to same file (inefficiency)
- Network paths (UNC: \\server\share)
- Antivirus scanning (Windows Defender paths)
```

**Graph 5: Memory Usage**
```
Memory → Virtual Memory Snapshots

Shows memory allocation over time

Key columns:
- Process: Photoshop.exe
- Commit Size: Allocated memory
- Working Set: Physical RAM in use

Timeline analysis:
Working Set increasing steadily: Normal usage
Working Set dropping suddenly: Windows reclaimed RAM
Commit Size >> Working Set: Heavy paging occurring

Cross-reference with Page Fault graph:
High page faults + dropping working set = RAM pressure
```

**Graph 6: Context Switches**
```
Computation → CPU Usage (Sampled)
Right-click header → Add Column → Context Switch Count

High context switches indicate:
- Thread contention
- Lock waiting
- I/O blocking

Photoshop context switches:
<5,000/sec per thread: Normal
5,000-10,000/sec: Moderate contention
>10,000/sec: Severe contention (investigate)

Check "Wait Reason" column:
- WaitFreePage: Memory shortage
- WaitForSingleObject: Waiting on sync object
- WaitForMultipleObjects: Complex synchronization
- Suspended: Thread blocked
```

### Advanced WPA Techniques

**Comparing Two Traces:**
```
Scenario: Want to compare slow vs. fast operation

1. Capture "good" trace: wpr -start CPU -start GPU -filemode
2. Perform fast operation (e.g., small file save)
3. Stop: wpr -stop good_trace.etl
4. Capture "bad" trace: Repeat with slow operation
5. Stop: wpr -stop bad_trace.etl

Analysis:
1. Open both in WPA
2. Window → New Comparison Window
3. Select both traces
4. View differences side-by-side

Look for:
- Operations present in bad trace only
- Longer duration in bad trace
- Different code paths taken
```

**Symbol Loading for Stack Traces:**
```
To see function names instead of memory addresses:

1. WPA → Trace → Configure Symbol Paths
2. Add Microsoft symbol server:
   SRV*C:\Symbols*https://msdl.microsoft.com/download/symbols
3. Trace → Load Symbols
4. Wait for download (can take 5-10 minutes first time)

Benefits:
- See actual function names in Photoshop
- Identify which Photoshop subsystem is slow
- Example: "Adobe::GPU::Renderer::Process" vs. "0x7FF8C4B2"
```

**Filtering Noise:**
```
Problem: Trace contains too much irrelevant data

Solutions:

1. Time-based filtering:
   - Click and drag on timeline to select range
   - Right-click → Zoom to Selection
   - Focus only on problem timeframe

2. Process filtering:
   - Find Photoshop.exe in process list
   - Right-click → Filter to Process
   - Removes all other processes from view

3. Duration filtering:
   - Right-click column header → View Filter
   - Set Duration: "is greater than" 100ms
   - Shows only long-running operations

4. Stack filtering:
   - Right-click stack column → Filter
   - Include only: Photoshop.exe modules
   - Excludes system DLLs for cleaner view
```

---

## 3.5 Process Monitor Deep Dive

### Capturing Useful Data

**Step 1: Configure Filters BEFORE Capture**
```
Launch Procmon as Administrator

Essential filters:
1. Process Name | is | Photoshop.exe | Include
2. Duration | is greater than | 0.010 | Include (10ms threshold)
3. Result | is not | SUCCESS | Include (catch errors)

Optional performance filters:
4. Path | ends with | .tmp | Include (temp file activity)
5. Path | contains | scratch | Include (scratch disk)
6. Operation | is | WriteFile | Include (focus on writes)

Apply filters: Ctrl+L or Filter menu
```

**Why Filter First:**
- Unfiltered Procmon can capture 10,000+ events/second
- File size grows rapidly (1GB+ in minutes)
- Filtering reduces noise by 95%+

**Step 2: Enable Backing File (Important!)**
```
File → Backing Files

Settings:
☑ Use file named: C:\PS_Traces\procmon_capture.pml
Maximum file size: 2048 MB (2GB)
☑ Delete when closed (if temporary capture)

Why important:
- Without backing file: RAM fills quickly
- Long captures require backing file
- Prevents Out of Memory errors
```

**Step 3: Start Capture**
```
1. Ctrl+E to start (or click Capture icon)
2. Status bar shows: "Events: X, Captured: Y, Filtered: Z"
3. Perform slow Photoshop operation
4. Capture for 30-60 seconds
5. Ctrl+E to stop

Expected results:
Total events: 10,000-50,000 (unfiltered)
Filtered events: 500-2,000 (with good filters)
```

**Step 4: Save Capture**
```
File → Save
Format: Native Process Monitor Format (.PML)
Location: C:\PS_Traces\photoshop_analysis.PML

PML vs CSV:
PML: Keeps all metadata, stacktraces, can re-filter
CSV: Lightweight, for Excel analysis, loses some data
```

### Analysis Techniques

**Technique 1: Sort by Duration**
```
Click "Duration" column header to sort descending

Top operations are your bottlenecks:

Example findings:
1. WriteFile to D:\PS_Scratch\temp.tmp - 2.5 seconds
   → Slow scratch disk write
   
2. ReadFile from C:\Users\...\texture.jpg - 1.8 seconds
   → Large texture load, consider faster storage
   
3. CreateFile for output.psd - 0.8 seconds
   → Antivirus scanning on save?

Action items prioritized by impact (longest durations first)
```

**Technique 2: Stack Trace Analysis**
```
Double-click any event → Stack tab

Shows call chain:
Photoshop.exe+0x4B2C10    ← Photoshop code
Photoshop.exe+0x8A1234
kernel32.dll!WriteFile     ← System call
ntdll.dll!NtWriteFile      ← Kernel entry

If symbols loaded (see Setup section):
Adobe::Document::Save()    ← Readable function names
Adobe::FileIO::Write()

Use stack to understand:
- Why operation occurred
- Which Photoshop subsystem initiated it
- Whether it's expected or anomalous
```

**Technique 3: Path Analysis**
```
Tools → File Summary (Ctrl+Alt+F)

Groups all file activity by path:

Example results:
D:\PhotoshopScratch\
  ├─ Events: 12,453
  ├─ Read: 2.1 GB
  ├─ Write: 4.8 GB
  └─ Duration: 45.2 seconds total
  
C:\Users\User\Documents\project.psd
  ├─ Events: 234
  ├─ Read: 512 MB
  ├─ Write: 0 bytes (read-only access)
  └─ Duration: 1.2 seconds

Insights:
Heavy scratch disk use: RAM shortage
Excessive reads of same file: Caching issue
Writes to unusual paths: Plugin behavior
```

**Technique 4: Timeline View**
```
View → Show Timeline (Ctrl+T)

Shows event distribution over time:

Patterns to recognize:

Even distribution:
████████████████████████ 
→ Continuous background activity

Bursts:
█    ████    ██    ███████
→ Periodic operations (autosave, cache flush)

Sustained block:
          ████████████████
          ↑
          User action (save, filter apply)

Gap then activity:
████          ████████
    ↑ 
    User pause, then continued work
```

**Technique 5: Process Tree**
```
Tools → Process Tree (Ctrl+T)

Shows parent-child relationships:

Photoshop.exe (PID: 4512)
├─ Adobe CEF Helper.exe (PID: 5104) ← Extensions/panels
├─ Adobe QT Server.exe (PID: 5200) ← QuickTime codec
└─ Adobe Synchronizer.exe (PID: 5312) ← Cloud sync

If a child process has high I/O:
- May be plugin causing slowdown
- Consider disabling to test
- Check if necessary for work
```

### Common Patterns and Solutions

**Pattern 1: Excessive Scratch Disk Activity**
```
Symptoms in Procmon:
- 1000+ WriteFile operations to scratch location
- Duration of writes: 50-200ms each
- Continuous throughout session

Root cause: Insufficient RAM

Solution steps:
1. Edit → Preferences → Performance
2. Let Photoshop Use: Increase to 80%
3. Close other applications
4. Reduce document size if possible
5. If still occurs: Add physical RAM
```

**Pattern 2: Antivirus Scanning on Save**
```
Symptoms:
- CreateFile operation duration: 500-2000ms
- Stack trace shows: FLTMGR.SYS (filter manager)
- Path: Final output file

Root cause: Real-time antivirus scans file on creation

Solutions:
1. Add Photoshop to antivirus exclusions (safe if trusted)
2. Exclude output folder from real-time scanning
3. Adjust antivirus scan settings (reduce aggressiveness)

Steps (Windows Defender):
Settings → Update & Security → Windows Security
→ Virus & threat protection → Manage settings
→ Exclusions → Add folder → Select Photoshop output folder
```

**Pattern 3: Network Storage Latency**
```
Symptoms:
- Operations to \\server\share\ paths
- Duration: 500-5000ms per operation
- Multiple retries visible

Root cause: Working with files over network

Solutions:
1. Copy project files to local SSD
2. Work locally, sync back when done
3. If must use network:
   - Verify network speed (should be 1Gbps+)
   - Check network storage performance
   - Consider mapped drive vs. UNC path
   - Enable offline files (Windows feature)
```

**Pattern 4: Plugin I/O Overhead**
```
Symptoms:
- Procmon shows Adobe CEF Helper.exe
- File access to %AppData%\Adobe\CEP\extensions
- High frequency (100+ events/second)

Root cause: Poorly optimized plugin

Solutions:
1. Identify plugin from path name
2. Window → Extensions → Disable suspected plugin
3. Test performance without plugin
4. If improved: Contact plugin developer or find alternative
5. Remove plugin: C:\Program Files\Adobe\Adobe Photoshop\Required\CEP\extensions
```

**Pattern 5: Font Cache Thrashing**
```
Symptoms:
- Repeated access to Windows\Fonts folder
- Operations to font cache files
- Occurs when using Text tool

Root cause: Font cache corruption or too many fonts

Solutions:
1. Clear font cache:
   - Close Photoshop
   - Delete: C:\Windows\ServiceProfiles\LocalService\AppData\Local\FontCache*
   - Restart Windows
   
2. Reduce installed fonts:
   - Control Panel → Fonts
   - Move rarely-used fonts to backup folder
   - Keep <500 fonts installed

3. Use font manager:
   - NexusFont or FontBase
   - Load fonts on-demand
```

---

## 4. Tool-Specific Deep Dives

## 4.1 Process Explorer Mastery

### Setup for Photoshop Analysis

**Essential Columns:**
```
View → Select Columns

Process Performance tab:
☑ CPU Usage
☑ CPU Time
☑ Context Switch Delta/sec
☑ Private Bytes
☑ Working Set
☑ Peak Working Set
☑ Page Faults Delta/sec

Process GPU tab:
☑ GPU Usage
☑ GPU Dedicated Bytes
☑ GPU System Bytes
☑ GPU Commit

Process I/O tab:
☑ I/O Total
☑ I/O Reads
☑ I/O Writes
☑ I/O Read Bytes
☑ I/O Write Bytes
```

**Visual Enhancements:**
```
Options → Difference Highlight Duration: 2 seconds

Effect:
- Values changing rapidly: Highlighted colors
- CPU spikes: Bright red highlight
- Memory changes: Color flash
- Makes bottlenecks visually obvious
```

**Performance Graphs:**
```
View → System Information (Ctrl+I)

CPU tab: Overall system CPU usage over time
GPU tab: GPU utilization over time
Memory tab: RAM usage, commit charge
Network tab: Bandwidth usage
Disk tab: I/O activity

Keep this window open during Photoshop work
Watch for correlation between actions and spikes
```

### Real-Time Monitoring Workflow

**Step 1: Baseline Measurement**
```
1. Launch Photoshop (no documents open)
2. Find Photoshop.exe in Process Explorer
3. Right-click → Properties

Note baseline values:
CPU: 0-2% (idle should be very low)
Working Set: 500-800 MB (without documents)
I/O: Minimal (<1 MB/sec)
GPU: 0-5% (idle should be near zero)

If baseline is high:
- Background plugin activity
- Corrupted preferences
- Extension panel loading
```

**Step 2: Document Load Analysis**
```
File → Open large document (e.g., 500MB PSD)

Watch Process Explorer during load:

Expected behavior:
CPU: Spike to 40-80% (parsing file)
Working Set: Increase by ~file size (e.g., +500MB)
I/O Reads: Sustained high (reading file)
GPU: Brief spike at end (rendering preview)

Problem indicators:
CPU at 100% for >30 seconds: Slow single-core parsing
Working Set increases slowly: Disk bottleneck
I/O Read Bytes/sec low (<50 MB/sec): Slow storage
Page Faults spiking: RAM shortage
```

**Step 3: Filter Application Test**
```
Filter → Blur → Gaussian Blur (large radius)

Observe:

GPU-accelerated:
CPU: 30-50% (feeding GPU)
GPU Usage: 80-100% (doing work)
Working Set: Minimal increase
Pattern: Quick completion

CPU-only:
CPU: 100% on multiple cores
GPU Usage: <10%
Working Set: May increase
Pattern: Slower completion

If GPU not used when it should be:
Edit → Preferences → Performance
Advanced Settings: Check settings
Try "Basic" or "Advanced" modes
```

**Step 4: Brush Lag Investigation**
```
Use Brush tool with large soft brush

Expected (smooth):
CPU: 40-60% per stroke
GPU: 50-70% (if GPU preview enabled)
Context Switches: <5,000/sec
Page Faults: <50/sec

Laggy behavior:
CPU: Maxed at 100% (CPU bottleneck)
GPU: Low <20% (not helping)
Context Switches: >10,000/sec (thread contention)
Page Faults: >100/sec (RAM thrashing)

Click on Photoshop.exe → Properties → Threads tab:
Sort by CPU Time: Find busiest thread
If one thread >>: Single-threaded bottleneck
If spread evenly: Good multi-threading
```

### Process Explorer Advanced Features

**DLL View:**
```
View → Lower Pane View → DLLs

Shows all loaded libraries:

Look for:
- Third-party DLLs: Plugins, codecs
- Version info: Outdated components
- Path location: Non-standard plugin directories

Problem DLL indicators:
- Very old version number (pre-2020)
- Non-Adobe DLLs in Photoshop folder
- Multiple versions of same DLL

Example issue:
Old QuickTime DLL: Can cause file format issues
→ Update QuickTime or remove if unused
```

**Handles View:**
```
View → Lower Pane View → Handles

Shows open files, registry keys, events

File handles:
- See which files Photoshop has open
- Useful for "File in use" errors
- Find leaked file handles (not closed properly)

Search (Ctrl+F):
Type filename to find which process has it open
```

**Performance Graph Annotations:**
```
Right-click graph → Add Annotation

Mark key events:
- "Started large filter" at 10:23:45
- "Photoshop froze" at 10:24:12
- "Resumed" at 10:24:30

Compare annotations to graph spikes:
Correlate user actions with resource usage
```

**Process Comparison:**
```
File → Save As: Save current snapshot
Do something in Photoshop (e.g., apply effect)
File → Compare: Compare to saved snapshot

Shows differences:
- Memory increase
- CPU time consumed
- I/O generated
- New threads created

Useful for measuring exact impact of operations
```

---

## 4.2 RAMMap for Memory Analysis

### Understanding Memory Types

**Launch RAMMap as Administrator:**
```
Key tabs and their meanings:

Use Counts:
- Total: Sum of all RAM usage
- Available: RAM that can be freed instantly
- Standby: File cache (can be repurposed)
- Modified: Dirty pages waiting to write to disk

Processes:
- Shows RAM usage per process
- Private: Memory exclusively for that process
- Working Set: Currently in physical RAM

Physical Pages:
- Map of actual RAM chips
- Color-coded by use
- Visual representation of fragmentation
```

### Photoshop Memory Issues

**Scenario 1: Photoshop Says "Not Enough RAM"**
```
Check RAMMap:

1. Use Counts tab:
   Available: 8 GB shown
   
2. Empty → Empty Working Sets
   This frees standby cache
   
3. Check Available again:
   If Available increases: Confirm RAM was available
   
4. If Photoshop still complains:
   Issue: Photoshop's preference setting
   Fix: Edit → Preferences → Performance
   Increase "Let Photoshop Use" slider
```

**Scenario 2: System Slow Despite Available RAM**
```
RAMMap → Use Counts:

Modified: 4 GB (high)
Standby: 12 GB

Problem: Modified pages not flushing to disk
Cause: Slow storage or background disk activity

Solution:
1. Empty → Empty Modified Page List (forces flush)
2. Check Disk activity in Task Manager
3. If slow disk: Upgrade to SSD
4. If fast disk: Disable background processes (Windows Search, etc.)
```

**Scenario 3: Memory Leak Detection**
```
Workflow:
1. Launch RAMMap before starting Photoshop
2. Note Photoshop RAM: ~800 MB (baseline)
3. Use Photoshop normally for 30 minutes
4. Refresh RAMMap
5. Check Photoshop RAM: Should be ~work + baseline

Memory leak indicators:
Photoshop RAM: 8 GB (with only 2 GB project)
Constantly increasing even when idle
Not released after closing documents

To confirm:
Close all documents in Photoshop
Edit → Purge → All
Wait 1 minute
RAM should drop back near baseline
If stays high: Likely memory leak (plugin or bug)
```

### Optimizing RAM for Photoshop

**Pre-Work Optimization:**
```
Before starting heavy Photoshop work:

1. RAMMap → Empty → Empty Working Sets
   Frees RAM from background apps
   
2. Empty → Empty Standby List
   Clears file cache
   
3. Empty → Empty Modified Page List
   Flushes pending writes
   
4. Result: Maximum free RAM for Photoshop

Note: System will automatically repopulate these
This is safe and temporary
Gives Photoshop first access to most RAM
```

**Monitoring During Work:**
```
Keep RAMMap open:

Every 5-10 minutes, check:
- Processes tab → Find Photoshop
- Working Set: RAM currently used
- Private: Memory allocated

Warning signs:
Working Set increasing steadily: Normal if actively working
Working Set >> file size: Possible inefficiency
Private much larger than Working Set: Heavy paging
```

---

## 5. Scenario-Based Troubleshooting

## 5.1 Slow Brush Performance

### Symptom
```
Brush strokes lag behind cursor
Noticeable delay between input and screen update
Brush feels "heavy" or unresponsive
```

### Diagnostic Steps

**Step 1: Quick Checks**
```
1. Check brush size:
   Large brushes (>1000px): Expected to be slower
   Solution: Reduce size or use smaller screen resolution
   
2. Check brush hardness:
   Soft brushes (0% hardness): More CPU intensive
   Solution: Increase hardness to 50-100% for better performance
   
3. Check document size:
   4K+ canvas with many layers: Expected lag
   Solution: Work at lower resolution, upscale at end
   
4. Check layer count:
   50+ layers: Slows brush preview
   Solution: Merge non-essential layers
```

**Step 2: GPU Check**
```
Edit → Preferences → Performance

Graphics Processor Settings:
☑ Use Graphics Processor (should be checked)

Advanced Settings:
Mode: Advanced (try this first)
If lagging: Try Normal or Basic

Test each mode:
Basic: CPU-only rendering
Normal: Standard GPU acceleration
Advanced: Maximum GPU use

If Advanced is slower than Basic:
→ GPU driver issue
→ Update GPU drivers
→ Check GPU temperature (might be throttling)
```

**Step 3: Resource Monitoring**
```
Launch Process Explorer:
Find Photoshop.exe

While using brush:
Watch CPU%: Should be 40-70%
Watch GPU Usage: Should be 50-90%
Watch Page Faults/sec: Should be <50

Problem patterns:

CPU 100%, GPU <20%:
→ CPU bottleneck or GPU not engaged
→ Enable GPU in preferences
→ Update GPU drivers

CPU 40%, GPU 100%:
→ GPU bottleneck
→ Reduce brush size
→ Lower document resolution
→ Check VRAM usage

CPU 100%, Page Faults >100/sec:
→ RAM shortage
→ Close other documents
→ Increase RAM allocation
→ Add physical RAM
```

**Step 4: WPR Trace Analysis**
```
Capture trace during brush lag:

wpr -start CPU -start GPU -filemode
[Use brush for 30 seconds]
wpr -stop brush_lag.etl
wpa brush_lag.etl

Analysis:

GPU Hardware Queue graph:
Queue depth 0 during lag: CPU not feeding GPU fast enough
Queue depth 10+: GPU overloaded
Long packet duration (>50ms): Complex shader or GPU overload

CPU Usage (Precise):
Find Photoshop thread with high CPU
Check stack trace
If in: Adobe::Brush::Render(): Expected
If in: Adobe::Layer::Composite(): Layer blend overhead
If in: System DLLs: Driver overhead
```

### Solutions by Cause

**Cause: GPU Not Enabled**
```
Solution:
Edit → Preferences → Performance
☑ Use Graphics Processor
Advanced Settings: Try Advanced mode

Restart Photoshop

Test brush: Should be smoother

If still not working:
Help → System Info
Check GPU section: Should say "Available"
If Unavailable: Driver issue or unsupported GPU
```

**Cause: GPU Driver Issue**
```
Solution:

1. Identify GPU:
   Device Manager → Display adapters

2. Update driver:
   NVIDIA: GeForce Experience or website
   AMD: AMD Software or website
   Intel: Intel Driver & Support Assistant

3. Clean install (if update doesn't help):
   Download DDU (Display Driver Uninstaller)
   Boot to Safe Mode
   Run DDU to remove old driver
   Reboot
   Install latest driver clean

4. Test in Photoshop
```

**Cause: CPU Bottleneck**
```
Solution:

Short-term:
- Reduce brush complexity (fewer dynamics, simpler tip)
- Lower document resolution
- Flatten layers when possible
- Close other applications
- Disable unnecessary Photoshop panels

Long-term:
- CPU upgrade (focus on single-core speed)
- Overclock CPU (if capable and cooled)
- Work on smaller documents
- Use proxy workflow (low-res edit, high-res final)
```

**Cause: RAM Shortage**
```
Solution:

Immediate:
Edit → Purge → All
Close unused documents
Close other applications

Settings:
Edit → Preferences → Performance
Let Photoshop Use: 75-80% (not higher)
History States: 20 (reduce from default 50)

System-level:
Close browser tabs
Disable background apps:
  Task Manager → Startup → Disable unnecessary
  Settings → Privacy → Background apps → Disable

Ultimate:
Add more physical RAM (16GB → 32GB+ recommended)
```

**Cause: Disk Bottleneck (Scratch)**
```
Solution:

Check scratch disk usage:
Efficiency indicator: Should be >95%
If <90%: Scratch disk in use

Immediate fix:
Edit → Purge → All
Close and reopen document

Scratch disk settings:
Edit → Preferences → Performance → Scratch Disks
☑ Select fastest SSD
Ensure 50GB+ free space

Verify disk speed:
Use CrystalDiskMark to test scratch disk
Should be >500 MB/s for SATA SSD
Should be >2000 MB/s for NVMe

If slow HDD:
Urgent: Upgrade to SSD
Temporary: Free up more space (reduces fragmentation)
```

---

## 5.2 Filter Processing Takes Forever

### Symptom
```
Applying filters (Blur, Sharpen, etc.) is extremely slow
Progress bar barely moves
Photoshop becomes unresponsive during filter
```

### Diagnostic Workflow

**Step 1: Identify If It's Truly Slow**
```
Baseline expectations:

Gaussian Blur (500px radius) on 4000x3000 image:
Fast system (i7 10th gen + RTX 3060): 3-8 seconds
Average system (i5 8th gen + GTX 1660): 8-15 seconds
Slow system (i5 6th gen + no GPU): 30-60 seconds

If taking 2-3 minutes: Definitely a problem
If taking 15-20 seconds: Depends on hardware (might be normal)
```

**Step 2: Check GPU Acceleration**
```
While filter is processing:

Task Manager → Performance → GPU:
GPU should be 70-100%
If <30%: GPU not accelerating filter

Filters that support GPU:
- Gaussian Blur
- Motion Blur
- Smart Sharpen
- Liquify
- Neural Filters

Filters that don't:
- Add Noise (CPU only)
- Some third-party filters

If GPU not engaged:
Edit → Preferences → Performance
☑ Use Graphics Processor must be on
Advanced Settings: Try Advanced mode
```

**Step 3: Document Size Assessment**
```
Image → Image Size

Check dimensions and resolution:

Problem indicators:
Width/Height: >8000px
Resolution: >300 DPI
Layers: 50+
Bit depth: 16-bit or 32-bit

Why it matters:
Filter processing scales with pixels
8000x8000 = 64 megapixels
4000x4000 = 16 megapixels
4x reduction = 4x faster processing

Solution for huge documents:
1. Work on smaller proxy version
2. Apply filters at lower resolution
3. Save filter settings
4. Apply to full-res version at end when needed
```

**Step 4: WPR Capture During Filter**
```
wpr -start CPU -start GPU -filemode
[Apply slow filter]
wpr -stop filter_slow.etl
wpa filter_slow.etl

In WPA:

GPU Hardware Queue:
Look at queue depth during filter
Consistent 8-10 packets: GPU working hard (expected)
Fluctuating 0-2-5-0: CPU-GPU sync issues
Empty queue with low GPU%: GPU not used

CPU Usage (Precise):
All cores >80%: Multi-threaded filter (good)
One core 100%, others low: Single-threaded (bad)
All cores <40%: GPU doing the work (expected)

If CPU and GPU both low:
Check Disk Usage graph
High disk activity: Paging to scratch disk (RAM shortage)
```

### Solutions

**Solution 1: Enable GPU Acceleration**
```
Edit → Preferences → Performance

Graphics Processor Settings:
☑ Use Graphics Processor (enable)
Advanced Settings: Select "Advanced"

For filters specifically:
Edit → Preferences → Technology Previews
☑ Enable experimental features (if available)

After changes:
Completely restart Photoshop
Test filter again
```

**Solution 2: Optimize RAM Usage**
```
Check Efficiency indicator during filter:
<95%: RAM shortage causing slowdown

Fixes:

A. Close other documents:
File → Close All except current
Edit → Purge → All

B. Increase RAM allocation:
Edit → Preferences → Performance
Let Photoshop Use: Increase to 75-80%

C. Reduce History States:
Preferences → Performance
History States: 20 (down from 50)

D. Flatten layers before filter (if safe):
Layer → Flatten Image (or Merge Visible)
Filters process faster on fewer layers
```

**Solution 3: Reduce Document Complexity**
```
Before applying filter:

1. Merge layers when possible:
   Select layers → Ctrl+E (Merge Down)
   Or: Create merged copy (Ctrl+Alt+Shift+E)

2. Reduce bit depth (if acceptable):
   Image → Mode → 8 Bits/Channel
   (From 16-bit or 32-bit)
   Significantly faster processing

3. Crop to selection:
   If only filtering part of image
   Make selection → Image → Crop

4. Work on smaller copy:
   Image → Image Size → Reduce to 50%
   Apply filter
   Note settings
   Undo size reduction
   Apply filter to full size (it uses cached computation)
```

**Solution 4: Scratch Disk Optimization**
```
If Efficiency <90%:

A. Free up scratch disk space:
   Delete temp files
   Empty Recycle Bin
   Keep 100GB+ free

B. Change scratch disk to faster drive:
   Edit → Preferences → Performance → Scratch Disks
   Uncheck slow drives (HDDs)
   ☑ Only fastest SSD
   
C. Use multiple scratch disks:
   ☑ Check multiple fast drives
   Photoshop will distribute load

D. Defragment scratch disk (HDDs only):
   Windows: Defragment and Optimize Drives utility
   (Don't defrag SSDs - not needed and reduces lifespan)
```

**Solution 5: Smart Filters Instead**
```
Convert layer to Smart Object first:

1. Select layer
2. Filter → Convert for Smart Filters
3. Apply filter (same speed initially)

Benefits:
- Filter is non-destructive
- Can adjust settings without re-applying
- Can disable/re-enable instantly (no re-processing)
- Can apply to multiple layers efficiently

When to use:
- Experimenting with settings
- Might need to change later
- Applying same filter to multiple images

When NOT to use:
- Need absolute maximum speed
- Working with very large files (Smart Objects add overhead)
```

---

## 5.3 Photoshop Startup is Slow

### Symptom
```
Photoshop takes 30+ seconds to launch
Splash screen hangs on "Initializing..." steps
Long delay before main window appears
```

### Diagnostic Approach

**Step 1: Measure Baseline**
```
Time from double-click to main window:

Fast launch: 5-10 seconds
Average launch: 10-20 seconds
Slow launch: 20-40 seconds
Very slow: 40+ seconds

If >30 seconds: Problem exists
If <15 seconds: Acceptable given Photoshop's complexity
```

**Step 2: Watch Splash Screen**
```
Pay attention to where it pauses:

"Loading plugins...": Plugin issue
"Initializing fonts...": Font cache problem
"Reading preferences...": Corrupt preferences
"Initializing GPU...": GPU driver delay
"Loading workspaces...": Corrupt workspace

Note the longest pause to focus troubleshooting
```

**Step 3: Process Monitor Capture**
```
Launch Procmon as Admin

Filters:
Process Name | is | Photoshop.exe | Include
Duration | is greater than | 0.100 | Include

Start capture (Ctrl+E)
Launch Photoshop
Stop capture when main window appears

Analysis:
Sort by Duration (descending)
Look for operations >1 second

Common slow operations:
- Reading thousands of fonts
- Loading many plugins
- Accessing network paths
- Antivirus scanning files
```

**Step 4: Clean Boot Test**
```
Test if third-party services/startups interfere:

msconfig → Services tab
☑ Hide all Microsoft services
Disable All

msconfig → Startup tab
Open Task Manager → Startup tab
Disable all third-party items

Restart computer
Launch Photoshop
Time startup

If faster: Third-party conflict
Enable services/startups one by one to find culprit
```

### Solutions

**Solution 1: Font Cache Reset**
```
If pauses on "Initializing fonts...":

A. Clear Windows font cache:
   Close Photoshop
   Open CMD as Admin:
   
   cd %WinDir%\ServiceProfiles\LocalService\AppData\Local
   attrib -h FontCache*.dat
   del FontCache*.dat
   del GDIP*.dat
   
   Restart computer

B. Clear Photoshop font cache:
   Navigate to:
   C:\Users\[Username]\AppData\Roaming\Adobe\Adobe Photoshop [Version]\
   Delete: Adobe Photoshop [Version] Prefs.psp
   (Backup first!)

C. Reduce installed fonts:
   Control Panel → Fonts
   Move rarely-used fonts to backup folder
   Keep <500 fonts for best performance

Test launch after each step
```

**Solution 2: Disable Plugins**
```
If pauses on "Loading plugins...":

A. Find plugins folder:
   C:\Program Files\Adobe\Adobe Photoshop [Version]\Plug-ins

B. Temporarily move plugins:
   Create folder: C:\PS_Plugins_Disabled
   Move contents of Plug-ins to disabled folder
   Keep only:
   - Filters (Adobe's built-in)
   - File Formats (Adobe's built-in)

C. Launch Photoshop:
   Should be much faster

D. Re-enable plugins one by one:
   Move plugin back to Plug-ins folder
   Launch Photoshop
   Note startup time
   If dramatically slower: That plugin is the problem

E. Problematic plugins:
   Old/outdated plugins
   Network license validators
   Plugins that load large data on startup
```

**Solution 3: Reset Preferences**
```
If pauses on "Reading preferences...":

A. Safe launch (resets preferences):
   Hold Ctrl+Alt+Shift while launching Photoshop
   Dialog: "Delete Adobe Photoshop Settings?"
   Click Yes

B. Manual preference delete:
   Close Photoshop
   Navigate to:
   C:\Users\[Username]\AppData\Roaming\Adobe\Adobe Photoshop [Version]\
   Delete:
   - Adobe Photoshop [Version] Prefs.psp
   - Adobe Photoshop [Version] X64 Prefs.psp
   
   Launch Photoshop (recreates defaults)

C. Reconfigure preferences:
   Edit → Preferences
   Set RAM allocation: 70-80%
   Enable GPU
   Set scratch disks
   (Previous settings lost, need to reapply)
```

**Solution 4: GPU Initialization Delay**
```
If pauses on "Initializing GPU...":

A. Update GPU drivers:
   NVIDIA: Latest GeForce/Studio driver
   AMD: Latest Adrenalin driver
   Intel: Latest Graphics Driver

B. Temporarily disable GPU in Photoshop:
   Safe launch (Ctrl+Alt+Shift)
   Edit → Preferences → Performance
   ☐ Use Graphics Processor (uncheck)
   Restart Photoshop normally
   
   If now fast: GPU driver issue
   Update/reinstall driver
   Re-enable GPU

C. Check for driver conflicts:
   Device Manager → Display adapters
   Should show only one adapter (or iGPU + dGPU)
   Multiple identical entries: Driver corruption
   Solution: Use DDU to clean reinstall
```

**Solution 5: Disable Cloud Sync**
```
If network activity during startup:

Edit → Preferences → File Handling
☐ Disable "Automatically Sync Settings"

Help → Sign Out (if using Creative Cloud)
Test launch (should be faster without cloud sync delay)

Sign back in after testing if needed
```

**Solution 6: Antivirus Exclusions**
```
Procmon shows antivirus scanning Photoshop files:

Windows Defender:
Settings → Update & Security → Windows Security
→ Virus & threat protection → Manage settings
→ Exclusions → Add folder

Add:
- C:\Program Files\Adobe
- C:\Users\[Username]\AppData\Roaming\Adobe
- C:\Users\[Username]\AppData\Local\Adobe

Third-party antivirus:
Add same paths to exclusions
Refer to antivirus documentation

Security note:
This is safe for Adobe applications
Reduces startup time by 30-50%
```

---

## 6. Advanced Analysis Techniques

## 6.1 Bottleneck Identification Matrix

### Methodology

**Multi-Tool Correlation:**
```
Simultaneously monitor:
1. Task Manager (CPU, RAM, Disk, GPU)
2. Process Explorer (detailed process metrics)
3. GPU-Z or MSI Afterburner (GPU specifics)
4. HWiNFO64 (temperatures, clocks)

Perform slow operation in Photoshop
Observe all tools during problem
```

**Decision Matrix:**

| CPU% | GPU% | RAM Free | Disk Active | Page Faults/sec | Bottleneck | Primary Solution |
|------|------|----------|-------------|-----------------|------------|------------------|
| >90% | <30% | >4GB | <50% | <50 | CPU | Upgrade CPU or reduce workload |
| <50% | >90% | >4GB | <50% | <50 | GPU | Upgrade GPU or reduce GPU load |
| <70% | <70% | <2GB | >80% | >100 | RAM | Add RAM or close applications |
| <70% | <70% | >4GB | 100% | <50 | Disk | Upgrade to SSD or reduce I/O |
| <70% | <70% | >4GB | <50% | <50 | None (Software) | Driver update or Photoshop issue |
| >80% | >80% | <2GB | 100% | >100 | Multiple | System-wide upgrade needed |

**Example Diagnosis:**
```
Scenario: Filter application is slow

Observations:
CPU: 45%
GPU: 25%
RAM Free: 1.5GB
Disk: 100% active
Page Faults: 150/sec
Efficiency: 82%

Matrix match: RAM bottleneck
Root cause: Insufficient RAM → Heavy paging to disk
Solutions:
1. Increase RAM allocation in Photoshop
2. Close other applications
3. Add physical RAM (long-term)
```

---

## 6.2 Performance Profiling Script

### Automated Monitoring

**PowerShell Script: `C:\PerformanceScripts\PS_Monitor.ps1`**
```powershell
# Comprehensive Photoshop Performance Monitor
# Logs metrics every second to CSV

param(
    [int]$Duration = 300,  # 5 minutes default
    [string]$OutputPath = "C:\PS_Diagnostics\metrics_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
)

Write-Host "Starting Photoshop Performance Monitor" -ForegroundColor Cyan
Write-Host "Duration: $Duration seconds" -ForegroundColor Yellow
Write-Host "Output: $OutputPath" -ForegroundColor Yellow
Write-Host "`nWaiting for Photoshop to start..." -ForegroundColor Gray

# Wait for Photoshop process
while (-not (Get-Process Photoshop -ErrorAction SilentlyContinue)) {
    Start-Sleep -Seconds 1
}

$photoshop = Get-Process Photoshop
Write-Host "Photoshop detected (PID: $($photoshop.Id))" -ForegroundColor Green
Write-Host "Collecting metrics...`n" -ForegroundColor Green

# Initialize CSV
$headers = "Timestamp,CPU_Percent,RAM_GB,RAM_Private_GB,Threads,PageFaults_Sec,IO_MB_Sec,DiskQueue,GPU_Percent,GPU_VRAM_GB"
$headers | Out-File -FilePath $OutputPath -Encoding UTF8

$startTime = Get-Date

for ($i = 0; $i -lt $Duration; $i++) {
    $photoshop = Get-Process Photoshop -ErrorAction SilentlyContinue
    
    if (-not $photoshop) {
        Write-Host "Photoshop closed. Stopping monitoring." -ForegroundColor Red
        break
    }
    
    # Collect metrics
    $timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
    
    # CPU
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $cpuPercent = [math]::Round($cpu, 2)
    
    # Memory
    $ramGB = [math]::Round($photoshop.WorkingSet64 / 1GB, 2)
    $ramPrivateGB = [math]::Round($photoshop.PrivateMemorySize64 / 1GB, 2)
    $threads = $photoshop.Threads.Count
    
    # Page Faults
    $pageFaults = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $pageFaultsRounded = [math]::Round($pageFaults, 2)
    
    # I/O
    $ioBytes = (Get-Counter "\Process(Photoshop)\IO Data Bytes/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $ioMB = [math]::Round($ioBytes / 1MB, 2)
    
    # Disk Queue
    $diskQueue = (Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $diskQueueRounded = [math]::Round($diskQueue, 2)
    
    # GPU (may not work on all systems)
    $gpuPercent = 0
    $gpuVRAM = 0
    try {
        $gpuCounter = Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage" -ErrorAction Stop
        $gpuPercent = [math]::Round(($gpuCounter.CounterSamples | Measure-Object -Property CookedValue -Average).Average, 2)
    } catch {}
    
    # Build CSV line
    $line = "$timestamp,$cpuPercent,$ramGB,$ramPrivateGB,$threads,$pageFaultsRounded,$ioMB,$diskQueueRounded,$gpuPercent,$gpuVRAM"
    $line | Out-File -FilePath $OutputPath -Encoding UTF8 -Append
    
    # Console output (every 5 seconds)
    if ($i % 5 -eq 0) {
        Write-Host "[$timestamp] CPU: $cpuPercent% | RAM: $ramGB GB | PageFaults: $pageFaultsRounded/s | Disk Q: $diskQueueRounded" -ForegroundColor White
    }
    
    Start-Sleep -Seconds 1
}

Write-Host "`nMonitoring complete!" -ForegroundColor Green
Write-Host "Data saved to: $OutputPath" -ForegroundColor Cyan
Write-Host "`nAnalyzing results..." -ForegroundColor Yellow

# Simple analysis
$data = Import-Csv $OutputPath
$avgCPU = ($data | Measure-Object -Property CPU_Percent -Average).Average
$maxCPU = ($data | Measure-Object -Property CPU_Percent -Maximum).Maximum
$avgRAM = ($data | Measure-Object -Property RAM_GB -Average).Average
$maxRAM = ($data | Measure-Object -Property RAM_GB -Maximum).Maximum
$avgPageFaults = ($data | Measure-Object -Property PageFaults_Sec -Average).Average
$maxPageFaults = ($data | Measure-Object -Property PageFaults_Sec -Maximum).Maximum

Write-Host "`nSummary:" -ForegroundColor Cyan
Write-Host "CPU: Avg=$([math]::Round($avgCPU,2))% Max=$([math]::Round($maxCPU,2))%" -ForegroundColor $(if($avgCPU -gt 80){"Red"}else{"Green"})
Write-Host "RAM: Avg=$([math]::Round($avgRAM,2))GB Max=$([math]::Round($maxRAM,2))GB" -ForegroundColor $(if($maxRAM -gt 16){"Yellow"}else{"Green"})
Write-Host "Page Faults: Avg=$([math]::Round($avgPageFaults,2))/s Max=$([math]::Round($maxPageFaults,2))/s" -ForegroundColor $(if($avgPageFaults -gt 100){"Red"}elseif($avgPageFaults -gt 50){"Yellow"}else{"Green"})

# Recommendations
Write-Host "`nRecommendations:" -ForegroundColor Cyan
if ($avgCPU -gt 85) {
    Write-Host "- CPU bottleneck detected. Consider CPU upgrade or reducing workload." -ForegroundColor Red
}
if ($maxRAM -gt 20 -or $avgPageFaults -gt 100) {
    Write-Host "- RAM shortage detected. Increase RAM allocation or add physical RAM." -ForegroundColor Red
}
if ($avgPageFaults -lt 50 -and $avgCPU -lt 70) {
    Write-Host "- System resources look healthy!" -ForegroundColor Green
}
```

**Usage:**
```powershell
# Monitor for 5 minutes (default)
.\PS_Monitor.ps1

# Monitor for 10 minutes
.\PS_Monitor.ps1 -Duration 600

# Custom output path
.\PS_Monitor.ps1 -Duration 300 -OutputPath "C:\Logs\test_session.csv"
```

**Analysis in Excel:**
```
1. Open CSV in Excel
2. Create line charts:
   - CPU_Percent over time
   - RAM_GB over time
   - PageFaults_Sec over time
3. Identify spikes correlating with user actions
4. Look for sustained high values (bottlenecks)
5. Compare multiple sessions to track improvements
```

---

## 7. Automation & Scripting

## 7.1 Automated Diagnostic Suite

**PowerShell Script: `C:\PerformanceScripts\PS_Diagnostics.ps1`**
```powershell
# Automated Photoshop Diagnostics Suite
# Runs multiple checks and generates report

$reportPath = "C:\PS_Diagnostics\Report_$(Get-Date -Format 'yyyyMMdd_HHmmss').txt"

function Write-Report {
    param([string]$Text, [string]$Color = "White")
    Write-Host $Text -ForegroundColor $Color
    $Text | Out-File -FilePath $reportPath -Append -Encoding UTF8
}

Write-Report "======================================" "Cyan"
Write-Report "PHOTOSHOP PERFORMANCE DIAGNOSTICS" "Cyan"
Write-Report "======================================" "Cyan"
Write-Report "Date: $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')" "Gray"
Write-Report ""

# System Info
Write-Report "[1] SYSTEM INFORMATION" "Yellow"
$os = Get-CimInstance Win32_OperatingSystem
$cpu = Get-CimInstance Win32_Processor | Select-Object -First 1
$ram = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)
$freeRAM = [math]::Round($os.FreePhysicalMemory / 1MB, 2)

Write-Report "OS: $($os.Caption) $($os.Version)"
Write-Report "CPU: $($cpu.Name)"
Write-Report "CPU Cores: $($cpu.NumberOfCores) cores, $($cpu.NumberOfLogicalProcessors) threads"
Write-Report "RAM: $ram GB total, $freeRAM GB free"
Write-Report ""

# GPU Detection
Write-Report "[2] GPU DETECTION" "Yellow"
$gpu = Get-CimInstance Win32_VideoController | Where-Object { $_.Name -notlike "*Microsoft*" }
if ($gpu) {
    Write-Report "GPU: $($gpu.Name)" "Green"
    $vram = [math]::Round($gpu.AdapterRAM / 1GB, 2)
    if ($vram -gt 0) {
        Write-Report "VRAM: $vram GB"
    }
    Write-Report "Driver: $($gpu.DriverVersion)"
} else {
    Write-Report "GPU: Not detected or integrated only" "Red"
}
Write-Report ""

# Storage Info
Write-Report "[3] STORAGE ANALYSIS" "Yellow"
$drives = Get-CimInstance Win32_LogicalDisk | Where-Object { $_.DriveType -eq 3 }
foreach ($drive in $drives) {
    $freeGB = [math]::Round($drive.FreeSpace / 1GB, 2)
    $totalGB = [math]::Round($drive.Size / 1GB, 2)
    $percentFree = [math]::Round(($drive.FreeSpace / $drive.Size) * 100, 1)
    
    $color = "Green"
    if ($percentFree -lt 20) { $color = "Red" }
    elseif ($percentFree -lt 30) { $color = "Yellow" }
    
    Write-Report "$($drive.DeviceID) $freeGB GB free / $totalGB GB total ($percentFree% free)" $color
}
Write-Report ""

# Photoshop Status
Write-Report "[4] PHOTOSHOP STATUS" "Yellow"
$ps = Get-Process Photoshop -ErrorAction SilentlyContinue
if ($ps) {
    Write-Report "Status: Running (PID: $($ps.Id))" "Green"
    Write-Report "RAM Usage: $([math]::Round($ps.WorkingSet64/1GB, 2)) GB"
    Write-Report "CPU Time: $([math]::Round($ps.CPU, 2)) seconds"
    Write-Report "Threads: $($ps.Threads.Count)"
    
    # Real-time CPU
    Start-Sleep -Seconds 1
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    if ($cpu) {
        Write-Report "Current CPU: $([math]::Round($cpu, 2))%"
    }
    
    # Page Faults
    $pf = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    if ($pf) {
        $pfColor = if ($pf -gt 100) { "Red" } elseif ($pf -gt 50) { "Yellow" } else { "Green" }
        Write-Report "Page Faults: $([math]::Round($pf, 2))/sec" $pfColor
    }
} else {
    Write-Report "Status: Not running" "Red"
}
Write-Report ""

# Performance Counters
Write-Report "[5] SYSTEM PERFORMANCE" "Yellow"
$totalCPU = (Get-Counter "\Processor(_Total)\% Processor Time").CounterSamples.CookedValue
Write-Report "System CPU: $([math]::Round($totalCPU, 2))%"

$memoryPressure = [math]::Round((1 - ($freeRAM / $ram)) * 100, 1)
$memColor = if ($memoryPressure -gt 90) { "Red" } elseif ($memoryPressure -gt 80) { "Yellow" } else { "Green" }
Write-Report "Memory Pressure: $memoryPressure%" $memColor

$diskQueue = (Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length").CounterSamples.CookedValue
$diskColor = if ($diskQueue -gt 5) { "Red" } elseif ($diskQueue -gt 2) { "Yellow" } else { "Green" }
Write-Report "Disk Queue: $([math]::Round($diskQueue, 2))" $diskColor
Write-Report ""

# Recommendations
Write-Report "[6] RECOMMENDATIONS" "Yellow"
$issues = @()

if ($ram -lt 16) {
    $issues += "- RAM is below 16GB. Recommended: 32GB+ for Photoshop."
}
if ($freeRAM -lt 4) {
    $issues += "- Free RAM is low (<4GB). Close unnecessary applications."
}
if (-not $gpu -or $gpu.Name -like "*Intel*") {
    $issues += "- No dedicated GPU detected. Consider adding NVIDIA/AMD GPU."
}
$scratchDrive = $drives | Where-Object { $_.DeviceID -eq "C:" }
if ($scratchDrive -and $scratchDrive.FreeSpace / 1GB -lt 50) {
    $issues += "- C: drive has <50GB free. Free up space for scratch disk."
}
if ($ps -and $pf -gt 100) {
    $issues += "- High page faults detected. Increase RAM allocation in Photoshop."
}

if ($issues.Count -eq 0) {
    Write-Report "No critical issues detected! System is well-configured for Photoshop." "Green"
} else {
    foreach ($issue in $issues) {
        Write-Report $issue "Red"
    }
}

Write-Report ""
Write-Report "======================================" "Cyan"
Write-Report "Report saved to: $reportPath" "Cyan"
Write-Report "======================================" "Cyan"
```

**Usage:**
```powershell
# Run diagnostics
.\PS_Diagnostics.ps1

# Output saved to timestamped report file
# Review recommendations at bottom
```

---

## 7.2 Batch Performance Testing

**Script: Test filter performance across settings**
```powershell
# Photoshop Filter Performance Test
# Tests Gaussian Blur at different settings

$testResults = @()

Write-Host "Photoshop Performance Test Suite" -ForegroundColor Cyan
Write-Host "Testing Gaussian Blur performance...`n"

# Test configurations
$configs = @(
    @{ Name = "GPU Advanced"; GPU = $true; Mode = "Advanced" },
    @{ Name = "GPU Normal"; GPU = $true; Mode = "Normal" },
    @{ Name = "GPU Basic"; GPU = $true; Mode = "Basic" },
    @{ Name = "CPU Only"; GPU = $false; Mode = "Basic" }
)

foreach ($config in $configs) {
    Write-Host "Testing: $($config.Name)" -ForegroundColor Yellow
    
    # User must manually change Photoshop settings
    Write-Host "Please configure Photoshop:" -ForegroundColor Gray
    Write-Host "  Edit → Preferences → Performance" -ForegroundColor Gray
    if ($config.GPU) {
        Write-Host "  ☑ Use Graphics Processor" -ForegroundColor Gray
        Write-Host "  Advanced Settings → $($config.Mode)" -ForegroundColor Gray
    } else {
        Write-Host "  ☐ Use Graphics Processor (disable)" -ForegroundColor Gray
    }
    Write-Host "  Press Enter when ready..." -ForegroundColor Cyan
    Read-Host
    
    # Monitor during test
    Write-Host "Apply Filter → Blur → Gaussian Blur (500px radius)" -ForegroundColor Yellow
    Write-Host "Press Enter when filter completes..." -ForegroundColor Cyan
    
    $startTime = Get-Date
    Read-Host
    $endTime = Get-Date
    
    $duration = ($endTime - $startTime).TotalSeconds
    
    # Collect resource usage
    $ps = Get-Process Photoshop -ErrorAction SilentlyContinue
    $peakRAM = if ($ps) { [math]::Round($ps.PeakWorkingSet64 / 1GB, 2) } else { 0 }
    
    $result = [PSCustomObject]@{
        Configuration = $config.Name
        Duration_Seconds = [math]::Round($duration, 2)
        Peak_RAM_GB = $peakRAM
    }
    
    $testResults += $result
    Write-Host "Duration: $($result.Duration_Seconds)s | Peak RAM: $($result.Peak_RAM_GB)GB`n" -ForegroundColor Green
}

# Display results
Write-Host "`nTest Results Summary:" -ForegroundColor Cyan
$testResults | Format-Table -AutoSize

# Find fastest
$fastest = $testResults | Sort-Object Duration_Seconds | Select-Object -First 1
Write-Host "Fastest configuration: $($fastest.Configuration) ($($fastest.Duration_Seconds)s)" -ForegroundColor Green

# Export results
$csvPath = "C:\PS_Diagnostics\Performance_Test_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
$testResults | Export-Csv -Path $csvPath -NoTypeInformation
Write-Host "`nResults exported to: $csvPath" -ForegroundColor Cyan
```

---

## 7.3 Watchdog Script

**Background monitoring with alerts**
```powershell
# Photoshop Performance Watchdog
# Monitors and alerts on performance issues

param(
    [int]$CPUThreshold = 95,
    [int]$PageFaultThreshold = 100,
    [int]$CheckIntervalSeconds = 5
)

Write-Host "Photoshop Watchdog Started" -ForegroundColor Cyan
Write-Host "CPU Threshold: $CPUThreshold%" -ForegroundColor Yellow
Write-Host "Page Fault Threshold: $PageFaultThreshold/sec" -ForegroundColor Yellow
Write-Host "Monitoring every $CheckIntervalSeconds seconds`n" -ForegroundColor Yellow

$alertCount = 0

while ($true) {
    $ps = Get-Process Photoshop -ErrorAction SilentlyContinue
    
    if ($ps) {
        $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
        $pageFaults = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
        
        $timestamp = Get-Date -Format "HH:mm:ss"
        $alert = $false
        $message = "[$timestamp] "
        
        if ($cpu -gt $CPUThreshold) {
            $message += "⚠ CPU HIGH ($([math]::Round($cpu,1))%) "
            $alert = $true
        }
        
        if ($pageFaults -gt $PageFaultThreshold) {
            $message += "⚠ PAGE FAULTS HIGH ($([math]::Round($pageFaults,1))/s) "
            $alert = $true
        }
        
        if ($alert) {
            Write-Host $message -ForegroundColor Red
            $alertCount++
            
            # Play system beep
            [console]::beep(1000, 200)
            
            # Detailed info
            $ram = [math]::Round($ps.WorkingSet64 / 1GB, 2)
            Write-Host "  RAM: $ram GB | Threads: $($ps.Threads.Count)" -ForegroundColor Yellow
        } else {
            Write-Host "[$timestamp] OK - CPU: $([math]::Round($cpu,1))% | PageFaults: $([math]::Round($pageFaults,1))/s" -ForegroundColor Green
        }
    } else {
        Write-Host "[$((Get-Date).ToString('HH:mm:ss'))] Photoshop not running" -ForegroundColor Gray
    }
    
    Start-Sleep -Seconds $CheckIntervalSeconds
}
```

**Usage:**
```powershell
# Default thresholds
.\PS_Watchdog.ps1

# Custom thresholds
.\PS_Watchdog.ps1 -CPUThreshold 90 -PageFaultThreshold 50 -CheckIntervalSeconds 3

# Run in background (separate PowerShell window)
Start-Process powershell -ArgumentList "-NoExit", "-File", "C:\PerformanceScripts\PS_Watchdog.ps1"
```

---

## 8. Quick Reference

### Essential Commands Cheat Sheet
```powershell
# QUICK CHECKS

# Photoshop status
Get-Process Photoshop | Select-Object CPU, WorkingSet64, Threads

# Current CPU usage
Get-Counter "\Process(Photoshop)\% Processor Time"

# Page faults
Get-Counter "\Process(Photoshop)\Page Faults/sec"

# Disk queue
Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length"

# GPU usage (if available)
Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage"

# Free RAM
[math]::Round((Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory/1MB,2)

# TRACES

# Start WPR trace
wpr -start CPU -start GPU -start DiskIO -filemode

# Stop WPR trace
wpr -stop output.etl

# Open in WPA
wpa output.etl

# PROCESS MONITOR

# Launch and filter
procmon /AcceptEula /Minimized /LoadConfig filter.pms /BackingFile capture.pml

# CLEANUP

# Photoshop temp files
Remove-Item "$env:LOCALAPPDATA\Temp\Photoshop*" -Recurse -Force

# Clear font cache
Remove-Item "$env:WinDir\ServiceProfiles\LocalService\AppData\Local\FontCache*" -Force
```

### Troubleshooting Flow Diagram (Text)
```
SLOW PERFORMANCE
       |
       v
Check Efficiency Indicator
       |
       |---> <95%? -----> [RAM ISSUE]
       |                      |
       |                      |---> Increase RAM allocation
       |                      |---> Close other apps
       |                      `---> Check scratch disk space
       |
       `---> >95%
               |
               v
        Check GPU Usage
               |
               |---> <30%? -----> [GPU NOT USED]
               |                       |
               |                       |---> Enable in Preferences
               |                       `---> Update GPU drivers
               |
               `---> >30%
                       |
                       v
                Check CPU Usage
                       |
                       |---> >85%? -----> [CPU BOTTLENECK]
                       |                       |
                       |                       |---> Reduce workload
                       |                       `---> Upgrade CPU
                       |
                       `---> <85%
                               |
                               v
                        Check Disk Activity
                               |
                               |---> 100%? -----> [DISK BOTTLENECK]
                               |                       |
                               |                       `---> Upgrade to SSD
                               |
                               `---> Normal
                                       |
                                       v
                                [SOFTWARE ISSUE]
                                       |
                                       |---> Update Photoshop
                                       |---> Reset preferences
                                       `---> Check for plugin conflicts
```

### Common Issues Quick Fix Table

| Symptom | Quick Fix | Time to Implement |
|---------|-----------|-------------------|
| Brush lag | GPU preferences → Advanced | 30 seconds |
| Low efficiency | Increase RAM allocation to 75% | 30 seconds |
| Startup slow | Reset preferences (Ctrl+Alt+Shift) | 1 minute |
| Filter slow | Enable GPU acceleration | 30 seconds |
| "Out of RAM" error | Purge (Edit → Purge → All) | 10 seconds |
| Scratch disk full | Free up 50GB+ on scratch drive | 5 minutes |
| GPU not detected | Update GPU drivers | 10 minutes |
| High page faults | Close other applications | 1 minute |

---

## Appendix: Tool Download Links

- **Windows Performance Toolkit**: https://learn.microsoft.com/windows-hardware/get-started/adk-install
- **Sysinternals Suite**: https://learn.microsoft.com/sysinternals/downloads/
- **Process Explorer**: https://learn.microsoft.com/sysinternals/downloads/process-explorer
- **Process Monitor**: https://learn.microsoft.com/sysinternals/downloads/procmon
- **RAMMap**: https://learn.microsoft.com/sysinternals/downloads/rammap
- **MSI Afterburner**: https://www.msi.com/Landing/afterburner
- **GPU-Z**: https://www.techpowerup.com/gpuz/
- **HWiNFO64**: https://www.hwinfo.com/download/
- **LatencyMon**: https://www.resplendence.com/latencymon
- **CrystalDiskMark**: https://crystalmark.info/en/software/crystaldiskmark/
- **DDU (Display Driver Uninstaller)**: https://www.guru3d.com/files-details/display-driver-uninstaller-download.html

---

## END OF GUIDE

This comprehensive guide covers Windows performance troubleshooting for Photoshop from installation through advanced analysis. Use the table of contents to navigate to specific sections as needed.

For automated diagnostics, use the PowerShell scripts in Section 7.

For quick troubleshooting, refer to Section 8 Quick Reference.

---

**Document Version**: 1.0 Complete
**Last Updated**: 2025
**Total Sections**: 8
**Total Word Count**: ~28,000 words
