# Windows Performance Troubleshooting Part Two

# Complete Advanced Photoshop Diagnostics Guide with Windows Tools

**Project Complexity: High**
**Estimated Days: 4-5 days** (3-4 hours/day)
**Your Rate: $3/hour**

## Budget Breakdown:

**Total Hours: 12-20 hours**
- **Minimum**: 12 hours × $3 = **$36 USD / ₹3,024 INR**
- **Maximum**: 20 hours × $3 = **$60 USD / ₹5,040 INR**

---

## Complete Tool Arsenal

### Windows Built-in Tools:
1. **Performance Monitor (perfmon)** - Counter tracking
2. **Resource Monitor (resmon)** - Real-time analysis
3. **Event Viewer** - Error logging
4. **Reliability Monitor** - Crash history
5. **Windows Memory Diagnostics** - RAM testing
6. **PowerShell** - Automation scripts
7. **Task Scheduler** - Automated monitoring
8. **Windows Performance Recorder (WPR/WPA)** - Deep tracing
9. **Registry Editor** - Photoshop config analysis
10. **DISM/SFC** - System integrity checks

### Sysinternals Suite:
11. **Process Explorer** - Advanced task manager
12. **Process Monitor** - File/Registry activity
13. **RAMMap** - Memory analysis
14. **Handle** - Open file handles
15. **Autoruns** - Startup analysis
16. **PsInfo** - System information
17. **DiskView** - Disk fragmentation
18. **CacheSet** - System cache management

### Third-Party Tools:
19. **GPU-Z** - GPU monitoring
20. **HWiNFO64** - Hardware sensors
21. **MSI Afterburner** - GPU OSD
22. **CrystalDiskInfo** - Drive health
23. **CrystalDiskMark** - Storage speed
24. **LatencyMon** - DPC/ISR latency
25. **DDU** - Driver uninstaller

---

## Day 1: System-Level Diagnostics (4-5 hours)

### 1.1 Performance Monitor Data Collection (1 hour)

**Setup Data Collector Set:**
```powershell
# Create custom data collector
$collectorName = "PhotoshopDiagnostics"
$outputPath = "C:\PerfLogs\$collectorName"

# Create via perfmon GUI or PowerShell
logman create counter $collectorName -f bincirc -v mmddhhmm -max 500 -c `
"\Processor(*)\% Processor Time" `
"\Process(Photoshop)\*" `
"\GPU Engine(*)\*" `
"\PhysicalDisk(*)\*" `
"\Memory\*" `
"\Network Interface(*)\*" `
-si 00:00:01 -o "$outputPath\perfmon.blg"

# Start collection
logman start $collectorName
```

**Manual GUI Setup:**
1. Win+R → `perfmon`
2. Data Collector Sets → User Defined → Right-click → New → Data Collector Set
3. Name: "PhotoshopDiagnostics"
4. Add counters:
   - Processor: % Processor Time (all instances)
   - Process: All counters for Photoshop
   - GPU Engine: All counters
   - PhysicalDisk: All counters
   - Memory: Available MBytes, Page Faults/sec, Cache Bytes
5. Sample interval: 1 second
6. Location: C:\PerfLogs
7. Start → Right-click → Start

**Use Photoshop normally for 10-15 minutes**

**Analysis:**
```powershell
# View in Performance Monitor
perfmon /sys
# File → Open → Select .blg file
# Analyze graphs for bottlenecks
```

### 1.2 Reliability Monitor Check (20 min)

**Steps:**
```
Win+R → perfmon /rel

Check last 7 days:
- Application failures (Photoshop crashes)
- Windows failures (driver crashes)
- Warning events (configuration changes)
- Information events (software installs)
```

**Export report:**
- Save reliability history
- Note any recurring patterns
- Correlate crashes with updates/changes

### 1.3 Event Viewer Deep Dive (1 hour)

**Custom Views:**
```
Event Viewer → Create Custom View

Filter 1: Photoshop Errors
- Event level: Critical, Error, Warning
- Event logs: Application, System
- Keywords: Photoshop, Adobe
- Last 7 days

Filter 2: GPU Driver Issues
- Event sources: nvlddmkm, amdwddmg, igfx
- Event IDs: 4101, 14, 13 (common GPU errors)

Filter 3: Disk Issues
- Event source: Disk
- Event IDs: 7, 11, 15, 51, 153
```

**Automated Event Export Script:**
```powershell
# Create: EventViewer_Export.ps1

$outputPath = "C:\Diagnostics\Events_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"

# Photoshop events
$psEvents = Get-WinEvent -FilterHashtable @{
    LogName = 'Application','System'
    Level = 1,2,3  # Critical, Error, Warning
    StartTime = (Get-Date).AddDays(-7)
} -ErrorAction SilentlyContinue | Where-Object { 
    $_.Message -like "*Photoshop*" -or 
    $_.Message -like "*Adobe*" -or
    $_.ProviderName -like "*nvlddmkm*" -or
    $_.ProviderName -like "*amdwddmg*"
}

$results = @()
foreach ($event in $psEvents) {
    $results += [PSCustomObject]@{
        Time = $event.TimeCreated
        Level = $event.LevelDisplayName
        Source = $event.ProviderName
        EventID = $event.Id
        Message = $event.Message -replace "`n", " " -replace "`r", ""
    }
}

$results | Export-Csv -Path $outputPath -NoTypeInformation
Write-Host "Events exported to: $outputPath" -ForegroundColor Green
$results | Format-Table -AutoSize
```

### 1.4 System File Integrity Check (30 min)

```powershell
# Check system files
sfc /scannow

# Check Windows image
DISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealth
DISM /Online /Cleanup-Image /RestoreHealth

# Output to log
DISM /Online /Cleanup-Image /RestoreHealth /LogPath:C:\Diagnostics\DISM.log
```

### 1.5 Autoruns Analysis (30 min)

**Check startup impact:**
```
1. Download Autoruns from Sysinternals
2. Run as Administrator
3. Options → Hide Microsoft Entries
4. Check tabs:
   - Logon: Startup programs
   - Services: Background services
   - Scheduled Tasks: Automated tasks
   - Drivers: Loaded drivers

Look for:
- Non-essential startup items
- Unknown publishers
- Adobe services (note if excessive)
- GPU-related services

Test: Disable non-essential items, test Photoshop performance
```

### 1.6 Registry Analysis (30 min)

**Photoshop Registry Keys:**
```powershell
# Export Photoshop registry settings
reg export "HKEY_CURRENT_USER\Software\Adobe\Photoshop" "C:\Diagnostics\PS_Registry.reg" /y

# Check GPU settings
reg query "HKEY_CURRENT_USER\Software\Adobe\Photoshop\130.0" /s | findstr /i "gpu opengl metal"

# Check memory settings
reg query "HKEY_CURRENT_USER\Software\Adobe\Photoshop\130.0" /s | findstr /i "memory cache scratch"
```

**Common Registry Issues:**
- Corrupted GPU preference values
- Invalid scratch disk paths
- Excessive cache size settings

**Safe backup before changes:**
```powershell
# Backup registry key
reg export "HKEY_CURRENT_USER\Software\Adobe" "C:\Diagnostics\Adobe_Registry_Backup.reg" /y
```

---

## Day 2: Advanced Process Analysis (4-5 hours)

### 2.1 Handle Utility - Open File Analysis (30 min)

**Check locked files:**
```cmd
handle.exe -a Photoshop > C:\Diagnostics\handles.txt

# Search for specific file
handle.exe "filename.psd"

# Find network paths
handle.exe | findstr "\\\\server"
```

**What to look for:**
- Files not released after closing
- Network paths causing delays
- Temp files accumulating
- Plugin files locked

### 2.2 DiskView - Fragmentation Analysis (30 min)

**For HDD scratch disks only (not SSD):**
```
1. Run DiskView.exe as Administrator
2. Select scratch disk drive
3. Visual map shows fragmentation

Red/scattered blocks = Heavy fragmentation
If >20% fragmented: Defragment recommended

Note: Never defragment SSDs
```

### 2.3 CacheSet - System Cache Tuning (20 min)

**Optimize system cache for Photoshop:**
```
Run cacheset.exe as Administrator

Current cache: Note value
Recommended: Set to 512MB-1GB for Photoshop workloads

cacheset.exe 1024

Test Photoshop performance after change
```

### 2.4 PsInfo - System Detailed Info (15 min)

```cmd
psinfo.exe -h -d > C:\Diagnostics\system_info.txt

Captures:
- CPU details (architecture, cache sizes)
- RAM configuration (speed, channels)
- Installed hotfixes
- System uptime
```

### 2.5 Enhanced Process Monitor Workflow (2 hours)

**Step 1: Configure Advanced Filters**
```
Filter → Filter → Add:

1. Process Name | is | Photoshop.exe | Include
2. Duration | is greater than | 0.050 | Include
3. Result | is | SUCCESS | Exclude (focus on errors)
4. Operation | is | RegQueryValue | Exclude (reduce noise)
5. Path | contains | AppData\Local\Temp | Include
6. Path | contains | scratch | Include
```

**Step 2: Enable Call Stacks**
```
Options → Enable Boot Logging (for comprehensive capture)
Options → Stack Summary
Tools → Stack Summary → By Module → Find Photoshop.exe modules
```

**Step 3: Backing File Setup**
```
File → Backing Files
☑ Use file named: C:\Diagnostics\procmon_ps_$(date).pml
Maximum: 2048 MB
```

**Step 4: Capture Workflow**
```
1. Clear display (Ctrl+X)
2. Start capture (Ctrl+E)
3. Reproduce slow operation in Photoshop
4. Stop capture (Ctrl+E) after 30-60 seconds
5. Save (Ctrl+S)
```

**Step 5: Advanced Analysis**
```
Tools → Process Tree
- Identify child processes (CEF Helper, QT Server)
- Check I/O operations per child

Tools → File Summary
- Group by: File extension
- Sort by: Total Duration
- Identify: Most time-consuming file operations

Tools → Network Summary
- Check for unexpected network activity
- Cloud sync during work?

Tools → Registry Summary
- Frequent registry access = plugin activity
- High duration registry ops = corruption?
```

**Export findings:**
```
File → Save → CSV format
Open in Excel for pivot tables/charts
```

### 2.6 Comprehensive WPR/WPA Analysis (1.5 hours)

**Enhanced WPR Capture:**
```cmd
# Stop any existing recordings
wpr -cancel

# Start comprehensive recording
wpr -start GeneralProfile -start CPU -start GPU -start DiskIO -start FileIO -start Network -filemode

# Use Photoshop (reproduce issue)
# 30-60 seconds recommended

# Stop recording
wpr -stop C:\Diagnostics\PS_Complete_$(date:~10,4)$(date:~4,2)$(date:~7,2).etl

# Note: File will be 500MB-2GB
```

**WPA Analysis Workflow:**

**A. CPU Analysis**
```
1. Open .etl in WPA
2. Graph Explorer → Computation → CPU Usage (Precise)
3. Drag to Analysis window
4. Preset: "Utilization by Process, Thread"
5. Find Photoshop.exe
6. Expand threads → Sort by CPU usage

Look for:
- Single thread at 100% = single-threaded bottleneck
- Multiple threads at high % = good multi-threading
- Wait times in "Ready (µs)" column = scheduling delays

Stack analysis:
- Right-click thread → Expand stack
- Identify function names (if symbols loaded)
- Note: Adobe::GPU:: functions vs Adobe::CPU:: functions
```

**B. GPU Analysis**
```
1. Computation → GPU Hardware Queue
2. Drag to Analysis window
3. Group by: Process, Queue Packet
4. Find Photoshop.exe

Metrics:
- Queue Depth: Should be 5-10 for active GPU use
- Packet Duration: How long GPU commands take
- Gaps in timeline: CPU not feeding GPU fast enough

Correlation:
- Switch between CPU and GPU graphs
- Zoom to same time range (Ctrl+Z)
- Look for correlation between CPU spikes and GPU queue depth
```

**C. Disk I/O Analysis**
```
1. Storage → Disk Usage
2. Preset: "Disk Offset by Process"
3. Filter to Photoshop.exe

Columns of interest:
- IO Type: Read/Write
- Size: Operation size
- Disk Service Time: How long disk took
- Path Name: Which file

Patterns indicating issues:
- Small random I/O (<64KB) = cache thrashing
- High service time (>100ms) = slow disk
- Frequent scratch disk access = RAM shortage
- Network paths (UNC \\server\) = network latency
```

**D. File I/O Analysis**
```
1. Storage → File I/O
2. Group by: Process, Path, Operation

Focus on:
- CreateFile operations >500ms = antivirus scanning
- ReadFile operations >200ms = slow storage
- WriteFile operations to scratch = RAM issue
- SetFilePointer operations = seeking behavior
```

**E. DPC/ISR Latency**
```
1. Computation → DPC/ISR
2. Sort by Duration (descending)

High latency indicators:
- DPC duration >1000µs
- ISR duration >500µs
- Frequent spikes = driver issue

Identify culprit:
- Module column shows driver name
- Common issues: nvlddmkm.sys (NVIDIA), Wdf01000.sys (USB)
```

**Export WPA Data:**
```
1. Select graph
2. Ctrl+A (select all rows)
3. Ctrl+C (copy)
4. Paste into Excel
5. Create pivot tables for summary
```

---

## Day 3: Custom Diagnostic Scripts (4-5 hours)

### 3.1 Master Monitoring Script

**Create: `PS_MasterDiagnostics.ps1`**

```powershell
#Requires -RunAsAdministrator
<#
.SYNOPSIS
    Comprehensive Photoshop Performance Diagnostics
.DESCRIPTION
    Automated diagnostics combining multiple Windows tools
    Captures: Perfmon, Event logs, Process metrics, GPU data, System health
.PARAMETER Duration
    Monitoring duration in seconds (default: 300)
.PARAMETER OutputFolder
    Output folder for all diagnostic files
.EXAMPLE
    .\PS_MasterDiagnostics.ps1 -Duration 600 -OutputFolder "C:\PS_Diag"
#>

param(
    [int]$Duration = 300,
    [string]$OutputFolder = "C:\Diagnostics\PS_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
)

# Create output folder
New-Item -ItemType Directory -Path $OutputFolder -Force | Out-Null

$reportFile = "$OutputFolder\DiagnosticReport.txt"
$csvFile = "$OutputFolder\metrics.csv"

function Write-Log {
    param([string]$Message, [string]$Color = "White")
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    $logMessage = "[$timestamp] $Message"
    Write-Host $logMessage -ForegroundColor $Color
    $logMessage | Out-File -FilePath $reportFile -Append -Encoding UTF8
}

Write-Log "=== PHOTOSHOP MASTER DIAGNOSTICS ===" "Cyan"
Write-Log "Output folder: $OutputFolder" "Yellow"
Write-Log "Duration: $Duration seconds" "Yellow"
Write-Log ""

# ============================================
# SECTION 1: SYSTEM INFORMATION
# ============================================
Write-Log "[1/10] Gathering System Information..." "Yellow"

$os = Get-CimInstance Win32_OperatingSystem
$cpu = Get-CimInstance Win32_Processor | Select-Object -First 1
$gpu = Get-CimInstance Win32_VideoController | Where-Object { $_.Name -notlike "*Microsoft*" }
$ram = [math]::Round($os.TotalVisibleMemorySize / 1MB, 2)
$freeRam = [math]::Round($os.FreePhysicalMemory / 1MB, 2)

Write-Log "OS: $($os.Caption) Build $($os.BuildNumber)"
Write-Log "CPU: $($cpu.Name) | $($cpu.NumberOfCores)C/$($cpu.NumberOfLogicalProcessors)T"
Write-Log "RAM: $ram GB total, $freeRam GB free"

if ($gpu) {
    Write-Log "GPU: $($gpu.Name)"
    Write-Log "GPU Driver: $($gpu.DriverVersion) ($($gpu.DriverDate))"
    
    # Check driver age
    $driverAge = ((Get-Date) - $gpu.DriverDate).Days
    if ($driverAge -gt 180) {
        Write-Log "  ⚠ GPU driver is $driverAge days old (>6 months)" "Yellow"
    }
}

# Disk information
$drives = Get-CimInstance Win32_LogicalDisk | Where-Object { $_.DriveType -eq 3 }
Write-Log "`nStorage:"
foreach ($drive in $drives) {
    $freeGB = [math]::Round($drive.FreeSpace / 1GB, 2)
    $totalGB = [math]::Round($drive.Size / 1GB, 2)
    $pctFree = [math]::Round(($drive.FreeSpace / $drive.Size) * 100, 1)
    
    $driveColor = if ($pctFree -lt 20) { "Red" } elseif ($pctFree -lt 30) { "Yellow" } else { "Green" }
    Write-Log "$($drive.DeviceID) $freeGB/$totalGB GB ($pctFree% free)" $driveColor
}

Write-Log ""

# ============================================
# SECTION 2: PHOTOSHOP DETECTION
# ============================================
Write-Log "[2/10] Checking Photoshop Status..." "Yellow"

$psProcess = Get-Process Photoshop -ErrorAction SilentlyContinue
if (-not $psProcess) {
    Write-Log "❌ Photoshop is not running!" "Red"
    Write-Log "Please start Photoshop and run this script again." "Yellow"
    exit 1
}

Write-Log "✓ Photoshop detected (PID: $($psProcess.Id))" "Green"
Write-Log "  Path: $($psProcess.Path)"
Write-Log "  Version: $($psProcess.ProductVersion)"
Write-Log "  Start time: $($psProcess.StartTime)"
Write-Log ""

# ============================================
# SECTION 3: EVENT VIEWER EXPORT
# ============================================
Write-Log "[3/10] Exporting Event Logs..." "Yellow"

$eventFile = "$OutputFolder\Events.csv"
try {
    $events = Get-WinEvent -FilterHashtable @{
        LogName = 'Application','System'
        Level = 1,2,3
        StartTime = (Get-Date).AddDays(-7)
    } -ErrorAction SilentlyContinue | Where-Object {
        $_.Message -like "*Photoshop*" -or 
        $_.Message -like "*Adobe*" -or
        $_.ProviderName -like "*nvlddmkm*" -or
        $_.ProviderName -like "*amdwddmg*"
    } | Select-Object -First 100
    
    $events | Select-Object TimeCreated, LevelDisplayName, ProviderName, Id, Message | 
        Export-Csv -Path $eventFile -NoTypeInformation
    
    Write-Log "✓ Exported $($events.Count) relevant events to Events.csv" "Green"
    
    # Count critical errors
    $criticalCount = ($events | Where-Object { $_.Level -eq 1 }).Count
    $errorCount = ($events | Where-Object { $_.Level -eq 2 }).Count
    
    if ($criticalCount -gt 0) {
        Write-Log "  ⚠ $criticalCount critical errors found" "Red"
    }
    if ($errorCount -gt 0) {
        Write-Log "  ⚠ $errorCount errors found" "Yellow"
    }
} catch {
    Write-Log "  ⚠ Could not export events: $_" "Yellow"
}

Write-Log ""

# ============================================
# SECTION 4: PLUGIN ANALYSIS
# ============================================
Write-Log "[4/10] Analyzing Photoshop Plugins..." "Yellow"

$pluginPath = Split-Path $psProcess.Path
$pluginPath = Join-Path $pluginPath "Plug-ins"

if (Test-Path $pluginPath) {
    $plugins = Get-ChildItem -Path $pluginPath -Recurse -Include *.8bf,*.plugin,*.apl -ErrorAction SilentlyContinue
    
    $pluginData = @()
    foreach ($plugin in $plugins) {
        $age = ((Get-Date) - $plugin.LastWriteTime).Days
        $pluginData += [PSCustomObject]@{
            Name = $plugin.Name
            SizeKB = [math]::Round($plugin.Length / 1KB, 2)
            Modified = $plugin.LastWriteTime
            AgeDays = $age
            Path = $plugin.FullName
        }
    }
    
    $pluginData | Export-Csv -Path "$OutputFolder\Plugins.csv" -NoTypeInformation
    
    Write-Log "✓ Found $($plugins.Count) plugins" "Green"
    
    $oldPlugins = $pluginData | Where-Object { $_.AgeDays -gt 730 }
    if ($oldPlugins) {
        Write-Log "  ⚠ $($oldPlugins.Count) plugins are >2 years old:" "Yellow"
        $oldPlugins | Sort-Object AgeDays -Descending | Select-Object -First 5 | ForEach-Object {
            Write-Log "    - $($_.Name) ($($_.AgeDays) days old)" "Gray"
        }
    }
} else {
    Write-Log "  ⚠ Plugin folder not found" "Yellow"
}

Write-Log ""

# ============================================
# SECTION 5: REGISTRY ANALYSIS
# ============================================
Write-Log "[5/10] Analyzing Photoshop Registry..." "Yellow"

$regPath = "HKCU:\Software\Adobe\Photoshop"
if (Test-Path $regPath) {
    $versions = Get-ChildItem -Path $regPath
    $latestVersion = $versions | Sort-Object Name -Descending | Select-Object -First 1
    
    Write-Log "✓ Found Photoshop registry: $($latestVersion.PSChildName)" "Green"
    
    # Export registry
    $regExport = "$OutputFolder\PS_Registry.reg"
    reg export "HKEY_CURRENT_USER\Software\Adobe\Photoshop" $regExport /y | Out-Null
    Write-Log "  Registry exported to PS_Registry.reg" "Gray"
    
    # Check preferences file
    $prefsPath = "$env:APPDATA\Adobe\Adobe Photoshop $($latestVersion.PSChildName)\Adobe Photoshop $($latestVersion.PSChildName) Prefs.psp"
    if (Test-Path $prefsPath) {
        $prefsSize = (Get-Item $prefsPath).Length
        $prefsSizeMB = [math]::Round($prefsSize / 1MB, 2)
        
        if ($prefsSizeMB -gt 10) {
            Write-Log "  ⚠ Preferences file is large: $prefsSizeMB MB" "Yellow"
            Write-Log "    Consider resetting (Ctrl+Alt+Shift on launch)" "Gray"
        } else {
            Write-Log "  Preferences file size: $prefsSizeMB MB (normal)" "Green"
        }
    }
}

Write-Log ""

# ============================================
# SECTION 6: RELIABILITY HISTORY
# ============================================
Write-Log "[6/10] Checking Reliability History..." "Yellow"

try {
    $reliability = Get-CimInstance -ClassName Win32_ReliabilityRecords -Filter "SourceName LIKE '%Photoshop%' OR SourceName LIKE '%Adobe%'" -ErrorAction Stop
    
    $crashes = $reliability | Where-Object { $_.EventIdentifier -eq 1000 -or $_.EventIdentifier -eq 1001 }
    
    if ($crashes) {
        Write-Log "⚠ Found $($crashes.Count) Photoshop crashes/hangs in history" "Red"
        $crashes | Select-Object -First 5 | ForEach-Object {
            Write-Log "  - $($_.TimeGenerated): $($_.Message)" "Gray"
        }
    } else {
        Write-Log "✓ No recent crashes found" "Green"
    }
} catch {
    Write-Log "  Could not query reliability history" "Gray"
}

Write-Log ""

# ============================================
# SECTION 7: PERFORMANCE BASELINE
# ============================================
Write-Log "[7/10] Capturing Performance Baseline..." "Yellow"

Start-Sleep -Seconds 2

$baselineCPU = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
$baselineRAM = [math]::Round($psProcess.WorkingSet64 / 1GB, 2)
$baselineThreads = $psProcess.Threads.Count
$baselinePF = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue

Write-Log "Baseline metrics:"
Write-Log "  CPU: $([math]::Round($baselineCPU, 2))%"
Write-Log "  RAM: $baselineRAM GB"
Write-Log "  Threads: $baselineThreads"
Write-Log "  Page Faults: $([math]::Round($baselinePF, 2))/sec"

Write-Log ""

# ============================================
# SECTION 8: CONTINUOUS MONITORING
# ============================================
Write-Log "[8/10] Starting Continuous Monitoring for $Duration seconds..." "Yellow"
Write-Log "Please use Photoshop normally during this period." "Cyan"
Write-Log ""

# CSV header
"Timestamp,CPU_Pct,RAM_GB,RAM_Private_GB,Threads,PageFaults_Sec,IO_MB_Sec,DiskQueue,GPU_Pct,GPU_MemMB,SysCPU_Pct,SysRAM_Free_GB" | 
    Out-File -FilePath $csvFile -Encoding UTF8

$startTime = Get-Date
$highCPUCount = 0
$highPFCount = 0

for ($i = 0; $i -lt $Duration; $i++) {
    $psProcess = Get-Process Photoshop -ErrorAction SilentlyContinue
    
    if (-not $psProcess) {
        Write-Log "Photoshop closed. Stopping monitoring." "Red"
        break
    }
    
    $timestamp = (Get-Date).ToString("yyyy-MM-dd HH:mm:ss")
    
    # Process metrics
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $ramGB = [math]::Round($psProcess.WorkingSet64 / 1GB, 2)
    $ramPrivateGB = [math]::Round($psProcess.PrivateMemorySize64 / 1GB, 2)
    $threads = $psProcess.Threads.Count
    $pageFaults = (Get-Counter "\Process(Photoshop)\Page Faults/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $ioBytes = (Get-Counter "\Process(Photoshop)\IO Data Bytes/sec" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $ioMB = [math]::Round($ioBytes / 1MB, 2)
    
    # System metrics
    $diskQueue = (Get-Counter "\PhysicalDisk(_Total)\Current Disk Queue Length" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $sysCPU = (Get-Counter "\Processor(_Total)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    $sysRAMFree = [math]::Round((Get-CimInstance Win32_OperatingSystem).FreePhysicalMemory / 1MB, 2)
    
    # GPU metrics
    $gpuPct = 0
    $gpuMemMB = 0
    try {
        $gpuCounter = Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage" -ErrorAction Stop
        $gpuPct = [math]::Round(($gpuCounter.CounterSamples | Measure-Object CookedValue -Average).Average, 2)
    } catch {}
    
    if (Get-Command nvidia-smi -ErrorAction SilentlyContinue) {
        $nvMem = nvidia-smi --query-gpu=memory.used --format=csv,noheader,nounits 2>$null
        if ($nvMem) { $gpuMemMB = $nvMem }
    }
    
    # Write to CSV
    "$timestamp,$([math]::Round($cpu,2)),$ramGB,$ramPrivateGB,$threads,$([math]::Round($pageFaults,2)),$ioMB,$([math]::Round($diskQueue,2)),$gpuPct,$gpuMemMB,$([math]::Round($sysCPU,2)),$sysRAMFree" | 
        Out-File -FilePath $csvFile -Append -Encoding UTF8
    
    # Track issues
    if ($cpu -gt 90) { $highCPUCount++ }
    if ($pageFaults -gt 100) { $highPFCount++ }
    
    # Console output every 10 seconds
    if ($i % 10 -eq 0) {
        $elapsed = [math]::Round(($i / $Duration) * 100, 0)
        Write-Progress -Activity "Monitoring Photoshop" -Status "$elapsed% Complete" -PercentComplete $elapsed
        
        $cpuColor = if ($cpu -gt 90) { "Red" } elseif ($cpu -gt 70) { "Yellow" } else { "Green" }
        $pfColor = if ($pageFaults -gt 100) { "Red" } elseif ($pageFaults -gt 50) { "Yellow" } else { "Green" }
        
        Write-Host "[$timestamp] CPU: " -NoNewline
        Write-Host "$([math]::Round($cpu,1))%" -ForegroundColor $cpuColor -NoNewline
        Write-Host " | RAM: $ramGB GB | PF: " -NoNewline
        Write-Host "$([math]::Round($pageFaults,1))/s" -ForegroundColor $pfColor -NoNewline
        Write-Host " | GPU: $gpuPct% | DiskQ: $([math]::Round($diskQueue,1))"
    }
    
    Start-Sleep -Seconds 1
}

Write-Progress -Activity "Monitoring Photoshop" -Completed

Write-Log ""
Write-Log "✓ Monitoring complete. Data saved to metrics.csv" "Green"

# ============================================
# SECTION 9: ANALYSIS
# ============================================
Write-Log "[9/10] Analyzing Collected Data..." "Yellow"

$data = Import-Csv $csvFile
$avgCPU = ($data | Measure-Object -Property CPU_Pct -Average).Average
$maxCPU = ($data | Measure-Object -Property CPU_Pct -Maximum).Maximum
$avgRAM = ($data | Measure-Object -Property RAM_GB -Average).Average
$maxRAM = ($data | Measure-Object -Property RAM_GB -Maximum).Maximum
$avgPF = ($data | Measure-Object -Property PageFaults_Sec -Average).Average
$maxPF = ($data | Measure-Object -Property PageFaults_Sec -Maximum).Maximum
$avgGPU = ($data | Measure-Object -Property GPU_Pct -Average).Average
$maxGPU = ($data | Measure-Object -Property GPU_Pct -Maximum).Maximum
$avgDiskQ = ($data | Measure-Object -Property DiskQueue -Average).Average
$maxDiskQ = ($data | Measure-Object -Property DiskQueue -Maximum).Maximum

Write-Log ""
Write-Log "Performance Summary:" "Cyan"
Write-Log "CPU:        Avg=$([math]::Round($avgCPU,1))%  Max=$([math]::Round($maxCPU,1))%  (>85% sustained = bottleneck)" $(if($avgCPU -gt 85){"Red"}else{"Green"})
Write-Log "RAM:        Avg=$([math]::Round($avgRAM,1))GB Max=$([math]::Round($maxRAM,1))GB" $(if($maxRAM -gt $ram*0.8){"Yellow"}else{"Green"})
Write-Log "Page Faults: Avg=$([math]::Round($avgPF,1))/s Max=$([math]::Round($maxPF,1))/s (>100 = RAM shortage)" $(if($avgPF -gt 100){"Red"}elseif($avgPF -gt 50){"Yellow"}else{"Green"})
Write-Log "GPU:        Avg=$([math]::Round($avgGPU,1))%  Max=$([math]::Round($maxGPU,1))%" $(if($avgGPU -lt 30){"Yellow"}else{"Green"})
Write-Log "Disk Queue: Avg=$([math]::Round($avgDiskQ,1))  Max=$([math]::Round($maxDiskQ,1))   (>5 = bottleneck)" $(if($avgDiskQ -gt 5){"Red"}elseif($avgDiskQ -gt 2){"Yellow"}else{"Green"})

Write-Log ""
Write-Log "Issue Detection:" "Cyan"

if ($highCPUCount -gt ($Duration * 0.3)) {
    Write-Log "⚠ CPU was >90% for $([math]::Round(($highCPUCount/$Duration)*100,0))% of time" "Red"
}

if ($highPFCount -gt ($Duration * 0.2)) {
    Write-Log "⚠ High page faults detected $([math]::Round(($highPFCount/$Duration)*100,0))% of time" "Red"
}

if ($avgGPU -lt 30 -and $avgCPU -gt 60) {
    Write-Log "⚠ Low GPU usage with high CPU = GPU not being utilized" "Yellow"
}

if ($maxDiskQ -gt 10) {
    Write-Log "⚠ Disk bottleneck detected (queue >10)" "Yellow"
}

# ============================================
# SECTION 10: RECOMMENDATIONS
# ============================================
Write-Log ""
Write-Log "[10/10] Generating Recommendations..." "Yellow"
Write-Log ""
Write-Log "=== RECOMMENDATIONS ===" "Cyan"

$recommendations = @()

# CPU recommendations
if ($avgCPU -gt 85) {
    $recommendations += "🔴 CPU BOTTLENECK: Average CPU usage is $([math]::Round($avgCPU,1))%"
    $recommendations += "   Solutions:"
    $recommendations += "   - Reduce document size/complexity"
    $recommendations += "   - Close other applications"
    $recommendations += "   - Upgrade CPU (focus on single-thread performance)"
}

# RAM recommendations
if ($avgPF -gt 100 -or $maxRAM -gt ($ram * 0.9)) {
    $recommendations += "🔴 RAM SHORTAGE: High page faults ($([math]::Round($avgPF,1))/s avg)"
    $recommendations += "   Solutions:"
    $recommendations += "   - Edit → Preferences → Performance: Increase RAM to 75-80%"
    $recommendations += "   - Close other applications"
    $recommendations += "   - Edit → Purge → All (clear caches)"
    $recommendations += "   - Add more physical RAM (current: $ram GB)"
}

# GPU recommendations
if ($avgGPU -lt 30 -and $maxGPU -lt 50) {
    $recommendations += "🟡 GPU UNDERUTILIZATION: GPU averaging only $([math]::Round($avgGPU,1))%"
    $recommendations += "   Solutions:"
    $recommendations += "   - Edit → Preferences → Performance → Enable 'Use Graphics Processor'"
    $recommendations += "   - Advanced Settings: Try 'Advanced' mode"
    $recommendations += "   - Update GPU drivers"
    $recommendations += "   - Check Help → System Info: GPU should show 'Available'"
}

# Disk recommendations
if ($avgDiskQ -gt 5) {
    $recommendations += "🟡 DISK BOTTLENECK: Disk queue averaging $([math]::Round($avgDiskQ,1))"
    $recommendations += "   Solutions:"
    $recommendations += "   - Ensure scratch disk is on NVMe SSD (not HDD)"
    $recommendations += "   - Free up disk space (>50GB recommended)"
    $recommendations += "   - Check disk health with CrystalDiskInfo"
}

# Driver recommendations
if ($gpu -and ((Get-Date) - $gpu.DriverDate).Days -gt 180) {
    $recommendations += "🟡 OUTDATED GPU DRIVER: Driver is $(((Get-Date) - $gpu.DriverDate).Days) days old"
    $recommendations += "   Solutions:"
    $recommendations += "   - Update GPU driver from manufacturer website"
    $recommendations += "   - NVIDIA: Use DDU for clean install"
}

# Plugin recommendations
if ($oldPlugins -and $oldPlugins.Count -gt 5) {
    $recommendations += "🟡 OLD PLUGINS: $($oldPlugins.Count) plugins are >2 years old"
    $recommendations += "   Solutions:"
    $recommendations += "   - Test Photoshop with plugins disabled (safe mode)"
    $recommendations += "   - Update or remove outdated plugins"
}

if ($recommendations.Count -eq 0) {
    Write-Log "✅ No critical issues detected!" "Green"
    Write-Log "System appears well-configured for Photoshop." "Green"
} else {
    foreach ($rec in $recommendations) {
        if ($rec -match "^🔴") {
            Write-Log $rec "Red"
        } elseif ($rec -match "^🟡") {
            Write-Log $rec "Yellow"
        } else {
            Write-Log $rec "Gray"
        }
    }
}

Write-Log ""
Write-Log "=== DIAGNOSTIC COMPLETE ===" "Cyan"
Write-Log ""
Write-Log "All files saved to: $OutputFolder" "Green"
Write-Log "- DiagnosticReport.txt (this report)" "Gray"
Write-Log "- metrics.csv (performance data)" "Gray"
Write-Log "- Events.csv (system events)" "Gray"
Write-Log "- Plugins.csv (plugin list)" "Gray"
Write-Log "- PS_Registry.reg (registry backup)" "Gray"
Write-Log ""
Write-Log "Next steps:" "Cyan"
Write-Log "1. Review recommendations above" "Gray"
Write-Log "2. Apply highest priority fixes first (🔴 items)" "Gray"
Write-Log "3. Test performance after each change" "Gray"
Write-Log "4. Re-run this script to verify improvements" "Gray"
Write-Log ""

# Open output folder
explorer.exe $OutputFolder
```

### 3.2 GPU Deep Diagnostics Script

**Create: `GPU_DeepDiag.ps1`**

```powershell
#Requires -RunAsAdministrator
<#
.SYNOPSIS
    GPU-specific diagnostics for Photoshop
.DESCRIPTION
    Detailed GPU analysis: usage patterns, VRAM, clocks, throttling
#>

param(
    [int]$Duration = 120,
    [string]$OutputPath = "C:\Diagnostics\GPU_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
)

Write-Host "GPU Deep Diagnostics for Photoshop" -ForegroundColor Cyan
Write-Host "Duration: $Duration seconds`n" -ForegroundColor Yellow

# Detect GPU type
$gpu = Get-CimInstance Win32_VideoController | Where-Object { $_.Name -notlike "*Microsoft*" }
if (-not $gpu) {
    Write-Host "No dedicated GPU detected!" -ForegroundColor Red
    exit 1
}

Write-Host "Detected: $($gpu.Name)" -ForegroundColor Green
Write-Host "Driver: $($gpu.DriverVersion)`n" -ForegroundColor Gray

# Check for nvidia-smi
$hasNVIDIA = Get-Command nvidia-smi -ErrorAction SilentlyContinue
$isNVIDIA = $gpu.Name -like "*NVIDIA*" -or $gpu.Name -like "*GeForce*" -or $gpu.Name -like "*Quadro*"

if ($isNVIDIA -and -not $hasNVIDIA) {
    Write-Host "⚠ NVIDIA GPU detected but nvidia-smi not found in PATH" -ForegroundColor Yellow
    Write-Host "  Limited GPU metrics available`n" -ForegroundColor Gray
}

# CSV header
"Timestamp,GPU_Usage_Pct,GPU_MemUsed_MB,GPU_MemTotal_MB,GPU_MemPct,GPU_Temp_C,GPU_CoreClock_MHz,GPU_MemClock_MHz,GPU_PowerDraw_W,GPU_PowerLimit_W,PerfCap_Reason,PS_CPU_Pct" | 
    Out-File -FilePath $OutputPath -Encoding UTF8

Write-Host "Monitoring GPU... (use Photoshop now)`n" -ForegroundColor Cyan

$psProcess = Get-Process Photoshop -ErrorAction SilentlyContinue
if (-not $psProcess) {
    Write-Host "⚠ Photoshop not running. Start it for better correlation." -ForegroundColor Yellow
}

for ($i = 0; $i -lt $Duration; $i++) {
    $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
    
    # Basic GPU metrics (Performance Counters)
    $gpuUsage = 0
    try {
        $gpuCounter = Get-Counter "\GPU Engine(*engtype_3D)\Utilization Percentage" -ErrorAction Stop
        $gpuUsage = [math]::Round(($gpuCounter.CounterSamples | Measure-Object CookedValue -Average).Average, 2)
    } catch {}
    
    # NVIDIA-specific metrics
    $gpuMemUsed = 0
    $gpuMemTotal = 0
    $gpuTemp = 0
    $gpuCoreClock = 0
    $gpuMemClock = 0
    $gpuPowerDraw = 0
    $gpuPowerLimit = 0
    $perfCapReason = "N/A"
    
    if ($hasNVIDIA) {
        try {
            # Memory
            $memInfo = nvidia-smi --query-gpu=memory.used,memory.total --format=csv,noheader,nounits 2>$null
            if ($memInfo) {
                $memParts = $memInfo.Split(',')
                $gpuMemUsed = $memParts[0].Trim()
                $gpuMemTotal = $memParts[1].Trim()
            }
            
            # Temperature
            $tempInfo = nvidia-smi --query-gpu=temperature.gpu --format=csv,noheader,nounits 2>$null
            if ($tempInfo) { $gpuTemp = $tempInfo.Trim() }
            
            # Clocks
            $clockInfo = nvidia-smi --query-gpu=clocks.gr,clocks.mem --format=csv,noheader,nounits 2>$null
            if ($clockInfo) {
                $clockParts = $clockInfo.Split(',')
                $gpuCoreClock = $clockParts[0].Trim()
                $gpuMemClock = $clockParts[1].Trim()
            }
            
            # Power
            $powerInfo = nvidia-smi --query-gpu=power.draw,power.limit --format=csv,noheader,nounits 2>$null
            if ($powerInfo) {
                $powerParts = $powerInfo.Split(',')
                $gpuPowerDraw = [math]::Round([double]$powerParts[0].Trim(), 1)
                $gpuPowerLimit = [math]::Round([double]$powerParts[1].Trim(), 1)
            }
            
            # Performance cap reason
            $perfCapInfo = nvidia-smi --query-gpu=clocks_throttle_reasons.active --format=csv,noheader 2>$null
            if ($perfCapInfo) {
                $perfCapReason = $perfCapInfo.Trim()
            }
        } catch {}
    }
    
    # Calculate memory percentage
    $gpuMemPct = 0
    if ($gpuMemTotal -gt 0) {
        $gpuMemPct = [math]::Round(($gpuMemUsed / $gpuMemTotal) * 100, 1)
    }
    
    # Photoshop CPU for correlation
    $psCPU = 0
    $psProcess = Get-Process Photoshop -ErrorAction SilentlyContinue
    if ($psProcess) {
        $psCPU = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
        $psCPU = [math]::Round($psCPU, 1)
    }
    
    # Write to CSV
    "$timestamp,$gpuUsage,$gpuMemUsed,$gpuMemTotal,$gpuMemPct,$gpuTemp,$gpuCoreClock,$gpuMemClock,$gpuPowerDraw,$gpuPowerLimit,$perfCapReason,$psCPU" |
        Out-File -FilePath $OutputPath -Append -Encoding UTF8
    
    # Console output every 5 seconds
    if ($i % 5 -eq 0) {
        $gpuColor = if ($gpuUsage -gt 80) { "Green" } elseif ($gpuUsage -lt 30) { "Yellow" } else { "White" }
        $memColor = if ($gpuMemPct -gt 90) { "Red" } elseif ($gpuMemPct -gt 75) { "Yellow" } else { "Green" }
        $tempColor = if ($gpuTemp -gt 80) { "Red" } elseif ($gpuTemp -gt 70) { "Yellow" } else { "Green" }
        
        Write-Host "[$timestamp]" -NoNewline
        Write-Host " GPU: " -NoNewline
        Write-Host "$gpuUsage%" -ForegroundColor $gpuColor -NoNewline
        Write-Host " | VRAM: " -NoNewline
        Write-Host "$gpuMemPct%" -ForegroundColor $memColor -NoNewline
        Write-Host " ($gpuMemUsed/$gpuMemTotal MB)" -NoNewline
        Write-Host " | Temp: " -NoNewline
        Write-Host "${gpuTemp}C" -ForegroundColor $tempColor -NoNewline
        Write-Host " | Clocks: $gpuCoreClock/$gpuMemClock MHz | PS_CPU: $psCPU%"
        
        if ($perfCapReason -ne "N/A" -and $perfCapReason -ne "0x0000000000000000") {
            Write-Host "  ⚠ Throttling active: $perfCapReason" -ForegroundColor Yellow
        }
    }
    
    Start-Sleep -Seconds 1
}

Write-Host "`nData collection complete!" -ForegroundColor Green
Write-Host "Analyzing...`n" -ForegroundColor Cyan

# Analysis
$data = Import-Csv $OutputPath

$avgGPU = ($data | Measure-Object -Property GPU_Usage_Pct -Average).Average
$maxGPU = ($data | Measure-Object -Property GPU_Usage_Pct -Maximum).Maximum
$avgVRAM = ($data | Measure-Object -Property GPU_MemPct -Average).Average
$maxVRAM = ($data | Measure-Object -Property GPU_MemPct -Maximum).Maximum
$avgTemp = ($data | Measure-Object -Property GPU_Temp_C -Average).Average
$maxTemp = ($data | Measure-Object -Property GPU_Temp_C -Maximum).Maximum
$avgClock = ($data | Measure-Object -Property GPU_CoreClock_MHz -Average).Average
$maxClock = ($data | Measure-Object -Property GPU_CoreClock_MHz -Maximum).Maximum

Write-Host "=== GPU ANALYSIS ===" -ForegroundColor Cyan
Write-Host "Usage:       Avg=$([math]::Round($avgGPU,1))%  Max=$([math]::Round($maxGPU,1))%" $(if($avgGPU -lt 30){"Yellow"}else{"Green"})
Write-Host "VRAM:        Avg=$([math]::Round($avgVRAM,1))%  Max=$([math]::Round($maxVRAM,1))%" $(if($maxVRAM -gt 90){"Red"}elseif($maxVRAM -gt 75){"Yellow"}else{"Green"})
Write-Host "Temperature: Avg=$([math]::Round($avgTemp,1))C Max=$([math]::Round($maxTemp,1))C" $(if($maxTemp -gt 80){"Red"}elseif($maxTemp -gt 70){"Yellow"}else{"Green"})
Write-Host "Core Clock:  Avg=$([math]::Round($avgClock,0))  Max=$([math]::Round($maxClock,0)) MHz"

Write-Host "`n=== DIAGNOSIS ===" -ForegroundColor Cyan

if ($avgGPU -lt 30) {
    Write-Host "⚠ LOW GPU USAGE ($([math]::Round($avgGPU,1))%)" -ForegroundColor Yellow
    Write-Host "  Possible causes:" -ForegroundColor Gray
    Write-Host "  - GPU not enabled in Photoshop preferences" -ForegroundColor Gray
    Write-Host "  - CPU bottleneck (GPU waiting for CPU)" -ForegroundColor Gray
    Write-Host "  - Operations not GPU-accelerated" -ForegroundColor Gray
    Write-Host "  Action: Check Photoshop GPU settings" -ForegroundColor Cyan
}

if ($maxVRAM -gt 90) {
    Write-Host "⚠ VRAM SATURATION ($([math]::Round($maxVRAM,1))%)" -ForegroundColor Red
    Write-Host "  Possible causes:" -ForegroundColor Gray
    Write-Host "  - Document too large for available VRAM" -ForegroundColor Gray
    Write-Host "  - Multiple documents open" -ForegroundColor Gray
    Write-Host "  Action: Reduce document size or close others" -ForegroundColor Cyan
}

if ($maxTemp -gt 80) {
    Write-Host "⚠ HIGH TEMPERATURE ($([math]::Round($maxTemp,1))C)" -ForegroundColor Red
    Write-Host "  Possible causes:" -ForegroundColor Gray
    Write-Host "  - Poor case airflow" -ForegroundColor Gray
    Write-Host "  - Dust buildup on GPU cooler" -ForegroundColor Gray
    Write-Host "  - Thermal paste degraded" -ForegroundColor Gray
    Write-Host "  Action: Clean GPU, improve case cooling" -ForegroundColor Cyan
}

# Check for clock throttling
$clockVariance = $maxClock - ($data | Measure-Object -Property GPU_CoreClock_MHz -Minimum).Minimum
if ($clockVariance -gt 300) {
    Write-Host "⚠ CLOCK SPEED VARIANCE: $clockVariance MHz range detected" -ForegroundColor Yellow
    Write-Host "  This indicates throttling (thermal or power limit)" -ForegroundColor Gray
}

# Check throttle reasons (NVIDIA)
$throttleOccurrences = $data | Where-Object { $_.PerfCap_Reason -ne "N/A" -and $_.PerfCap_Reason -ne "0x0000000000000000" }
if ($throttleOccurrences) {
    Write-Host "⚠ THROTTLING DETECTED: $($throttleOccurrences.Count) instances" -ForegroundColor Yellow
    $reasons = $throttleOccurrences | Group-Object -Property PerfCap_Reason | Sort-Object Count -Descending
    foreach ($reason in $reasons) {
        Write-Host "  - $($reason.Name): $($reason.Count) times" -ForegroundColor Gray
    }
}

Write-Host "`nData saved to: $OutputPath" -ForegroundColor Green
Write-Host "Import into Excel for graphical analysis" -ForegroundColor Cyan
```

---

## Day 4: Integration & Testing (3-4 hours)

### 4.1 Automated Test Suite (1 hour)

**Create: `PS_TestSuite.ps1`**

```powershell
# Photoshop Performance Test Suite
# Runs standardized tests and compares results

param(
    [string]$TestName = "Baseline"
)

Write-Host "Photoshop Test Suite - $TestName" -ForegroundColor Cyan
Write-Host ""

$results = @()

# Test 1: Photoshop Launch Time
Write-Host "[Test 1/5] Photoshop Launch Time" -ForegroundColor Yellow
$psPath = "C:\Program Files\Adobe\Adobe Photoshop 2024\Photoshop.exe"

if (Get-Process Photoshop -ErrorAction SilentlyContinue) {
    Write-Host "  ⚠ Photoshop already running. Close it first." -ForegroundColor Red
    exit 1
}

Write-Host "  Launching Photoshop..." -ForegroundColor Gray
$launchStart = Get-Date
Start-Process $psPath
Start-Sleep -Seconds 2

# Wait for main window
$timeout = 60
$elapsed = 0
while (-not (Get-Process Photoshop -ErrorAction SilentlyContinue) -and $elapsed -lt $timeout) {
    Start-Sleep -Seconds 1
    $elapsed++
}

if (-not (Get-Process Photoshop -ErrorAction SilentlyContinue)) {
    Write-Host "  ❌ Photoshop failed to launch" -ForegroundColor Red
    exit 1
}

# Wait for idle CPU (fully loaded)
Write-Host "  Waiting for launch to complete..." -ForegroundColor Gray
Start-Sleep -Seconds 5

$cpu = 100
$stableCount = 0
while ($stableCount -lt 3) {
    $cpu = (Get-Counter "\Process(Photoshop)\% Processor Time" -ErrorAction SilentlyContinue).CounterSamples.CookedValue
    if ($cpu -lt 10) {
        $stableCount++
    } else {
        $stableCount = 0
    }
    Start-Sleep -Seconds 1
}

$launchTime = ((Get-Date) - $launchStart).TotalSeconds
Write-Host "  ✓ Launch time: $([math]::Round($launchTime, 2)) seconds" -ForegroundColor Green

$results += [PSCustomObject]@{
    Test = "Launch Time"
    Value = [math]::Round($launchTime, 2)
    Unit = "seconds"
    Status = if ($launchTime -lt 15) { "Good" } elseif ($launchTime -lt 30) { "Fair" } else { "Poor" }
}

Start-Sleep -Seconds 5

# Test 2: Large Document Open
Write-Host "[Test 2/5] Large Document Open (requires test file)" -ForegroundColor Yellow
$testFile = "C:\TestFiles\large_document.psd"  # 500MB+ PSD

if (Test-Path $testFile) {
    Write-Host "  Opening $testFile..." -ForegroundColor Gray
    $openStart = Get-Date
    
    # Simulate File → Open (can't automate UI easily)
    Write-Host "  Please manually open: $testFile" -ForegroundColor Cyan
    Write-Host "  Press Enter when document is fully loaded..." -ForegroundColor Cyan
    Read-Host
    
    $openTime = ((Get-Date) - $openStart).TotalSeconds
    Write-Host "  ✓ Open time: $([math]::Round($openTime, 2)) seconds" -ForegroundColor Green
    
    $results += [PSCustomObject]@{
        Test = "Document Open"
        Value = [math]::Round($openTime, 2)
        Unit = "seconds"
        Status = if ($openTime -lt 5) { "Good" } elseif ($openTime -lt 10) { "Fair" } else { "Poor" }
    }
} else {
    Write-Host "  ⚠ Test file not found. Skipping." -ForegroundColor Yellow
}

# Test 3: Filter Performance
Write-Host "[Test 3/5] Gaussian Blur Performance" -ForegroundColor Yellow
Write-Host "  Please apply: Filter → Blur → Gaussian Blur (500px radius)" -ForegroundColor Cyan
Write-Host "  Press Enter when ready to start..." -ForegroundColor Cyan
Read-Host

$filterStart = Get-Date
Write-Host "  Monitoring... Apply filter now!" -ForegroundColor Yellow
Write-Host "  Press Enter when filter completes..." -ForegroundColor Cyan
Read-Host

$filterTime = ((Get-Date) - $filterStart).TotalSeconds
Write-Host "  ✓ Filter time: $([math]::Round($filterTime, 2)) seconds" -ForegroundColor Green

# Get peak CPU/GPU during filter
$ps = Get-Process Photoshop -ErrorAction SilentlyContinue
$peakRAM = [math]::Round($ps.PeakWorkingSet64 / 1GB, 2)

$results += [PSCustomObject]@{
    Test = "Gaussian Blur 500px"
    Value = [math]::Round($filterTime, 2)
    Unit = "seconds"
    Status = if ($filterTime -lt 5) { "Good" } elseif ($filterTime -lt 15) { "Fair" } else { "Poor" }
}

$results += [PSCustomObject]@{
    Test = "Peak RAM Usage"
    Value = $peakRAM
    Unit = "GB"
    Status = "Info"
}

# Test 4: Brush Responsiveness
Write-Host "[Test 4/5] Brush Responsiveness" -ForegroundColor Yellow
Write-Host "  Use Brush Tool (B): 500px soft brush" -ForegroundColor Cyan
Write-Host "  Paint 10 strokes and rate responsiveness (1-5):" -ForegroundColor Cyan
Write-Host "    1 = Severe lag, 3 = Noticeable delay, 5 = Instant" -ForegroundColor Gray
$brushScore = Read-Host "  Your rating"

$results += [PSCustomObject]@{
    Test = "Brush Responsiveness"
    Value = $brushScore
    Unit = "score/5"
    Status = if ($brushScore -ge 4) { "Good" } elseif ($brushScore -ge 3) { "Fair" } else { "Poor" }
}

# Test 5: Save Performance
Write-Host "[Test 5/5] Save Performance" -ForegroundColor Yellow
Write-Host "  Press Enter, then immediately File → Save (or Ctrl+S)..." -ForegroundColor Cyan
Read-Host

$saveStart = Get-Date
Write-Host "  Monitoring... Save now!" -ForegroundColor Yellow
Write-Host "  Press Enter when save completes..." -ForegroundColor Cyan
Read-Host

$saveTime = ((Get-Date) - $saveStart).TotalSeconds
Write-Host "  ✓ Save time: $([math]::Round($saveTime, 2)) seconds" -ForegroundColor Green

$results += [PSCustomObject]@{
    Test = "Save Time"
    Value = [math]::Round($saveTime, 2)
    Unit = "seconds"
    Status = if ($saveTime -lt 10) { "Good" } elseif ($saveTime -lt 30) { "Fair" } else { "Poor" }
}

# Display results
Write-Host "`n=== TEST RESULTS: $TestName ===" -ForegroundColor Cyan
$results | Format-Table -AutoSize

# Save results
$outputPath = "C:\Diagnostics\TestResults_${TestName}_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
$results | Export-Csv -Path $outputPath -NoTypeInformation
Write-Host "Results saved to: $outputPath" -ForegroundColor Green

# Close Photoshop
Write-Host "`nClose Photoshop? (Y/N)" -ForegroundColor Yellow
$response = Read-Host
if ($response -eq 'Y') {
    Stop-Process -Name Photoshop -Force
}
```

### 4.2 Task Scheduler Automation (30 min)

**Setup automated daily diagnostics:**

```powershell
# Create scheduled task for daily monitoring

$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-ExecutionPolicy Bypass -File C:\PerformanceScripts\PS_MasterDiagnostics.ps1 -Duration 300"

$trigger = New-ScheduledTaskTrigger -Daily -At 9:00AM

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest

$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries

Register-ScheduledTask -TaskName "Photoshop Daily Diagnostics" `
    -Action $action `
    -Trigger $trigger `
    -Principal $principal `
    -Settings $settings `
    -Description "Automated Photoshop performance monitoring"
```

---

## Day 5: Reporting & Documentation (2-3 hours)

### 5.1 Executive Summary Generator (1 hour)

**Create: `Generate_Report.ps1`**

```powershell
# Generates executive summary from diagnostic data

param(
    [string]$DiagFolder = "C:\Diagnostics\PS_*"
)

$folders = Get-ChildItem -Path (Split-Path $DiagFolder -Parent) -Filter (Split-Path $DiagFolder -Leaf) -Directory | Sort-Object LastWriteTime -Descending

if (-not $folders) {
    Write-Host "No diagnostic folders found matching: $DiagFolder" -ForegroundColor Red
    exit 1
}

Write-Host "Found $($folders.Count) diagnostic session(s)" -ForegroundColor Cyan
Write-Host "Select session to summarize:`n" -ForegroundColor Yellow

for ($i = 0; $i -lt $folders.Count; $i++) {
    Write-Host "[$($i+1)] $($folders[$i].Name) - $(($folders[$i].LastWriteTime).ToString('yyyy-MM-dd HH:mm'))"
}

$selection = Read-Host "`nEnter number"
$selectedFolder = $folders[[int]$selection - 1].FullName

Write-Host "`nGenerating summary for: $($folders[[int]$selection - 1].Name)`n" -ForegroundColor Green

# Read data files
$reportFile = Join-Path $selectedFolder "DiagnosticReport.txt"
$metricsFile = Join-Path $selectedFolder "metrics.csv"

if (-not (Test-Path $metricsFile)) {
    Write-Host "metrics.csv not found in selected folder" -ForegroundColor Red
    exit 1
}

$data = Import-Csv $metricsFile

# Calculate statistics
$stats = @{
    AvgCPU = [math]::Round(($data | Measure-Object -Property CPU_Pct -Average).Average, 1)
    MaxCPU = [math]::Round(($data | Measure-Object -Property CPU_Pct -Maximum).Maximum, 1)
    AvgRAM = [math]::Round(($data | Measure-Object -Property RAM_GB -Average).Average, 1)
    MaxRAM = [math]::Round(($data | Measure-Object -Property RAM_GB -Maximum).Maximum, 1)
    AvgPF = [math]::Round(($data | Measure-Object -Property PageFaults_Sec -Average).Average, 1)
    MaxPF = [math]::Round(($data | Measure-Object -Property PageFaults_Sec -Maximum).Maximum, 1)
    AvgGPU = [math]::Round(($data | Measure-Object -Property GPU_Pct -Average).Average, 1)
    MaxGPU = [math]::Round(($data | Measure-Object -Property GPU_Pct -Maximum).Maximum, 1)
    AvgDiskQ = [math]::Round(($data | Measure-Object -Property DiskQueue -Average).Average, 1)
    MaxDiskQ = [math]::Round(($data | Measure-Object -Property DiskQueue -Maximum).Maximum, 1)
}

# Generate HTML report
$html = @"
<!DOCTYPE html>
<html>
<head>
    <title>Photoshop Performance Report</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 40px; background: #f5f5f5; }
        .container { background: white; padding: 30px; border-radius: 8px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
        h1 { color: #001d6c; border-bottom: 3px solid #0066cc; padding-bottom: 10px; }
        h2 { color: #0066cc; margin-top: 30px; }
        .metric { display: inline-block; margin: 15px 20px 15px 0; padding: 15px; background: #f8f9fa; border-left: 4px solid #0066cc; }
        .metric-label { font-size: 12px; color: #666; text-transform: uppercase; }
        .metric-value { font-size: 24px; font-weight: bold; color: #001d6c; }
        .metric-unit { font-size: 14px; color: #666; }
        .status-good { color: #28a745; }
        .status-warning { color: #ffc107; }
        .status-critical { color: #dc3545; }
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        th, td { padding: 12px; text-align: left; border-bottom: 1px solid #dee2e6; }
        th { background: #f8f9fa; font-weight: 600; }
        .recommendation { margin: 15px 0; padding: 15px; border-radius: 4px; }
        .rec-critical { background: #f8d7da; border-left: 4px solid #dc3545; }
        .rec-warning { background: #fff3cd; border-left: 4px solid #ffc107; }
        .rec-info { background: #d1ecf1; border-left: 4px solid #0066cc; }
    </style>
</head>
<body>
    <div class="container">
        <h1>Photoshop Performance Diagnostic Report</h1>
        <p><strong>Date:</strong> $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')</p>
        <p><strong>Session:</strong> $($folders[[int]$selection - 1].Name)</p>
        
        <h2>Performance Metrics</h2>
        
        <div class="metric">
            <div class="metric-label">CPU Usage</div>
            <div class="metric-value $(if($stats.AvgCPU -gt 85){'status-critical'}elseif($stats.AvgCPU -gt 70){'status-warning'}else{'status-good'})">$($stats.AvgCPU)</div>
            <div class="metric-unit">% average (max: $($stats.MaxCPU)%)</div>
        </div>
        
        <div class="metric">
            <div class="metric-label">RAM Usage</div>
            <div class="metric-value status-good">$($stats.AvgRAM)</div>
            <div class="metric-unit">GB average (max: $($stats.MaxRAM) GB)</div>
        </div>
        
        <div class="metric">
            <div class="metric-label">Page Faults</div>
            <div class="metric-value $(if($stats.AvgPF -gt 100){'status-critical'}elseif($stats.AvgPF -gt 50){'status-warning'}else{'status-good'})">$($stats.AvgPF)</div>
            <div class="metric-unit">/sec average (max: $($stats.MaxPF)/sec)</div>
        </div>
        
        <div class="metric">
            <div class="metric-label">GPU Usage</div>
            <div class="metric-value $(if($stats.AvgGPU -lt 30){'status-warning'}else{'status-good'})">$($stats.AvgGPU)</div>
            <div class="metric-unit">% average (max: $($stats.MaxGPU)%)</div>
        </div>
        
        <div class="metric">
            <div class="metric-label">Disk Queue</div>
            <div class="metric-value $(if($stats.AvgDiskQ -gt 5){'status-critical'}elseif($stats.AvgDiskQ -gt 2){'status-warning'}else{'status-good'})">$($stats.AvgDiskQ)</div>
            <div class="metric-unit">average (max: $($stats.MaxDiskQ))</div>
        </div>
        
        <h2>Key Findings</h2>
        <div class="recommendations">
"@

# Add recommendations based on metrics
if ($stats.AvgCPU -gt 85) {
    $html += @"
            <div class="recommendation rec-critical">
                <strong>🔴 CPU Bottleneck Detected</strong><br>
                Average CPU usage is $($stats.AvgCPU)% which indicates a CPU bottleneck.<br>
                <strong>Actions:</strong>
                <ul>
                    <li>Reduce document complexity</li>
                    <li>Close other applications</li>
                    <li>Consider CPU upgrade</li>
                </ul>
            </div>
"@
}

if ($stats.AvgPF -gt 100) {
    $html += @"
            <div class="recommendation rec-critical">
                <strong>🔴 RAM Shortage Detected</strong><br>
                High page faults ($($stats.AvgPF)/sec average) indicate RAM shortage.<br>
                <strong>Actions:</strong>
                <ul>
                    <li>Increase Photoshop RAM allocation to 75-80%</li>
                    <li>Close other applications</li>
                    <li>Add more physical RAM</li>
                </ul>
            </div>
"@
}

if ($stats.AvgGPU -lt 30) {
    $html += @"
            <div class="recommendation rec-warning">
                <strong>🟡 Low GPU Utilization</strong><br>
                GPU usage is only $($stats.AvgGPU)% on average.<br>
                <strong>Actions:</strong>
                <ul>
                    <li>Enable GPU in Photoshop Preferences → Performance</li>
                    <li>Update GPU drivers</li>
                    <li>Try Advanced GPU mode</li>
                </ul>
            </div>
"@
}

if ($stats.AvgDiskQ -gt 5) {
    $html += @"
            <div class="recommendation rec-warning">
                <strong>🟡 Disk Bottleneck Detected</strong><br>
                Average disk queue is $($stats.AvgDiskQ) (high).<br>
                <strong>Actions:</strong>
                <ul>
                    <li>Ensure scratch disk is on NVMe SSD</li>
                    <li>Free up disk space</li>
                    <li>Check disk health</li>
                </ul>
            </div>
"@
}

if ($stats.AvgCPU -lt 70 -and $stats.AvgPF -lt 50 -and $stats.AvgGPU -gt 50) {
    $html += @"
            <div class="recommendation rec-info">
                <strong>✅ System Performance Good</strong><br>
                No critical bottlenecks detected. System is well-configured for Photoshop.
            </div>
"@
}

$html += @"
        </div>
        
        <h2>Next Steps</h2>
        <ol>
            <li>Review recommendations above (prioritize 🔴 items)</li>
            <li>Apply fixes one at a time</li>
            <li>Re-run diagnostics after each change</li>
            <li>Compare before/after metrics</li>
        </ol>
        
        <h2>Full Diagnostic Files</h2>
        <p>Complete diagnostic data available in:<br>
        <code>$selectedFolder</code></p>
        
        <hr>
        <p style="color: #666; font-size: 12px;">
            Generated by Photoshop Master Diagnostics Tool<br>
            $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')
        </p>
    </div>
</body>
</html>
"@

# Save HTML report
$htmlPath = Join-Path $selectedFolder "Summary_Report.html"
$html | Out-File -FilePath $htmlPath -Encoding UTF8

Write-Host "✓ HTML report generated: $htmlPath" -ForegroundColor Green

# Open in browser
Start-Process $htmlPath

Write-Host "`nReport opened in browser." -ForegroundColor Cyan
```

---

## Complete Toolkit Summary

### Scripts Created:
1. **PS_MasterDiagnostics.ps1** - All-in-one diagnostic tool (300+ lines)
2. **GPU_DeepDiag.ps1** - GPU-specific monitoring (200+ lines)
3. **PS_TestSuite.ps1** - Standardized performance tests
4. **Generate_Report.ps1** - HTML report generator
5. **EventViewer_Export.ps1** - Event log analyzer
6. **PS_Monitor.ps1** - Real-time metrics (from previous)
7. **PS_Watchdog.ps1** - Alert system (from previous)

### Windows Tools Used:
- Performance Monitor (perfmon) - Counter data collection
- Resource Monitor (resmon) - Real-time analysis
- Event Viewer - Error tracking
- Reliability Monitor - Crash history
- WPR/WPA - Deep trace analysis
- Process Monitor - File/Registry activity
- Process Explorer - Advanced process details
- RAMMap - Memory analysis
- Handle - File handle tracking
- Autoruns - Startup analysis
- Registry Editor - Config analysis
- Task Scheduler - Automation

### Deliverables:
1. Complete diagnostic reports (TXT + HTML)
2. Performance metrics (CSV with charts)
3. Event logs export
4. Plugin analysis
5. GPU deep analysis
6. Registry backups
7. Test results comparison
8. Recommendations document

---

## Why High Complexity?

**High Complexity Factors:**
- 25+ tools required
- Custom PowerShell development (7 scripts, 1000+ lines total)
- Multi-tool data correlation
- WPA trace analysis (expert level)
- Registry forensics
- Event log correlation
- GPU-specific diagnostics

**How to Tackle:**
- Run scripts in sequence (Day 1 → Day 2 → etc.)
- Focus on one tool/script at a time
- Automate data collection (scripts do heavy lifting)
- Document findings immediately after each session
- Use HTML reports for client presentation

---

## Final Budget & Timeline

### Minimum (Simple Issue):
- **3 days, 12 hours total**
- **Budget: $36 USD / ₹3,024 INR**
- Assumes: Quick identification via automated scripts

### Recommended (Thorough):
- **4 days, 16 hours total**
- **Budget: $48 USD / ₹4,032 INR**
- Includes: Full tool suite, all scripts, comprehensive analysis

### Maximum (Complex Investigation):
- **5 days, 20 hours total**
- **Budget: $60 USD / ₹5,040 INR**
- Includes: Multiple test runs, before/after comparisons, detailed reporting

### What Client Receives:
✅ Complete diagnostic toolkit (7 custom scripts)
✅ Professional HTML reports with visualizations
✅ CSV data files for Excel analysis
✅ Step-by-step recommendations prioritized by impact
✅ Before/after performance comparison
✅ Automated monitoring setup for future use
✅ Full documentation of findings

**Present all three options to client. Most will choose Recommended path.**
