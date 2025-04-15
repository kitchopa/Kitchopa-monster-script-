 
# ------------------------------
# Monster KitchopOS - Ultimate Performance Script
# ------------------------------

# Disable unnecessary services to free up resources
Stop-Service -Name "wuauserv" # Disable Windows Update
Stop-Service -Name "msiserver" # Disable Windows Installer
Stop-Service -Name "bits" # Disable Background Intelligent Transfer Service
Stop-Service -Name "Spooler" # Disable Print Spooler
Stop-Service -Name "SysMain" # Disable Superfetch
Stop-Service -Name "WinDefend" # Disable Windows Defender
Stop-Service -Name "DiagnosticPolicyService" # Disable Diagnostic Service
Stop-Service -Name "RemoteRegistry" # Disable Remote Registry

# Disable power-saving settings and set maximum performance
$powerplan = powercfg -list | Select-String -Pattern "High performance" | ForEach { ($_ -split " ")[3] }
powercfg -setactive $powerplan

# Disable Windows visual effects
$regPath = "HKCU:\Control Panel\Desktop"
Set-ItemProperty -Path $regPath -Name "VisualFXSetting" -Value 2
Set-ItemProperty -Path $regPath -Name "UserPreferencesMask" -Value "90 12 03 80"

# Set priority for CPU performance (High)
$regPath = "HKLM:\SYSTEM\CurrentControlSet\Control\PriorityControl"
Set-ItemProperty -Path $regPath -Name "Win32PrioritySeparation" -Value 26

# Disable unnecessary scheduled tasks (helps reduce processes)
Unregister-ScheduledTask -TaskName "OneDrive Standalone Update Task"
Unregister-ScheduledTask -TaskName "MicrosoftEdgeUpdate"

# Disable auto-updates for apps and Windows Store
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" -Name "NoAutoUpdate" -Value 1

# Optimize RAM (adjusts virtual memory settings)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "DisablePagingExecutive" -Value 1
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Memory Management" -Name "LargeSystemCache" -Value 1

# Install HIDUSBF (USB Overclocking Tool) and set polling rate to 1000Hz or 2000Hz
$HIDUSBFPath = "$env:temp\HIDUSBF.zip"
Invoke-WebRequest "https://github.com/LordOfMice/hidusbf/releases/download/v3.0/hidusbf.zip" -OutFile $HIDUSBFPath
Expand-Archive -Path $HIDUSBFPath -DestinationPath "$env:temp\HIDUSBF"
Start-Process "$env:temp\HIDUSBF\setup.exe" -ArgumentList "/silent" -Wait

# Force apply 1000Hz or 2000Hz polling rate to USB devices
# Automatically applies HIDUSBF settings to connected devices
Start-Process "$env:temp\HIDUSBF\HIDUSBF.exe" -ArgumentList "/setpollingrate 1000" -Wait

# Ensure device settings persist after reboot
Start-Process "$env:temp\HIDUSBF\HIDUSBF.exe" -ArgumentList "/install" -Wait

# Optimize Mouse and Keyboard Input Delay (Disable Filtering)
$regPath = "HKCU:\Control Panel\Mouse"
Set-ItemProperty -Path $regPath -Name "MouseSpeed" -Value 0
Set-ItemProperty -Path $regPath -Name "MouseThreshold1" -Value 0
Set-ItemProperty -Path $regPath -Name "MouseThreshold2" -Value 0

# Disable Mouse Acceleration (ensure max performance)
$regPath = "HKCU:\Control Panel\Desktop"
Set-ItemProperty -Path $regPath -Name "MouseSpeed" -Value 0

# Optimize Audio Device Settings (for minimal latency)
$audioRegPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Multimedia\Audio"
Set-ItemProperty -Path $audioRegPath -Name "EnableEnhancements" -Value 0
Set-ItemProperty -Path $audioRegPath -Name "PreferredSampleRate" -Value 48000
Set-ItemProperty -Path $audioRegPath -Name "PreferredBitDepth" -Value 24

# Disable power-saving on USB Audio Ports
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Enum\USB\VID_XXXX&PID_XXXX" -Name "DevicePowerPolicy" -Value 0

# ------------------------------
# Reboot to Apply Settings
# ------------------------------
Restart-Computer -
