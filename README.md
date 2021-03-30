# Windows-PowerShell-Script-to-disable-common-anti-privacy-features
Windows PowerShell Script to disable common anti-privacy features

*First and foremost: like any code or script snippet on the internet, it's generally not a good idea to run code on your computer if you don't know what it does. I can give you my word that this script does what I say it does, but if you don't believe me, don't run it.*

The script disables the following:

* General Telemetry
* Advertising ID
* App Launch tracking
* Windows Start Search History
* Tailored Experiences
* Improve Narrator
* Windows Suggestions

It also checks the length of your host file. If it has been edited without you knowing, some domains could resolve to a spoofed or fake server. This is a form of DNS cache poisoning

Paste the following code into a PowerShell Window, it will prompt you in red text if any of the checks failed. It will also resolve them by setting the appropriate registry settings. When you run it a second time, the output should be white and green only (no red), unless your host file was edited, that won't be fixed automatically

    function registryCheck($key,$value,$desc) {
     
     $avalue = get-itemProperty $key
    
     $found = 0
     $avalue.PSObject.Properties | ForEach-Object {
       if ($_.Name -like $value){
      $found = 1
       }
     }
     if ($found -eq 0) {
      $score--
      write-host "-1" -Foregroundcolor Red
      write-host "Missing value" -foregroundcolor Red
      New-ItemProperty $key -Name "$value" -Value 0 -PropertyType DWORD
     } else {
      if ($avalue."$value" -eq 1) {
       $score--
       write-host "-1" -Foregroundcolor Red
       write-host "$value has bad value" -foregroundcolor Red
       Set-ItemProperty $key -Name "$value" -Value 0
      } else {
       write-host "Passed $desc" -foregroundcolor green
      }
     }
     write-output "Finished $desc"
     
    }
    # 1. Check Registry for policies and common settings
    
    ########################################################
    # General Telemetry
    registryCheck "HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection" "Allow Telemetry" "Telemetry Test"
    
    # Advertising ID
    registryCheck "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\AdvertisingInfo" "Enabled" "Advertising ID Test"
    
    # App launch tracking
    registryCheck "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Advanced" "Start_TrackProgs" "App Launch Tracking Test"
    
    # Search History
    registryCheck "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\SearchSettings" "IsDeviceSearchHistoryEnabled" "Search History Test"
    
    # Tailored Experiences
    registryCheck "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\Privacy" "TailoredExperiencesWithDiagnosticDataEnabled" "Tailored Experiences Test"
    
    # Improve Narrator
    registryCheck "HKCU:\SOFTWARE\Microsoft\Narrator\NoRoam" "DetailedFeedback" "Send Narrator Data Test"
    
    # Windows Suggestions
    registryCheck "HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\UserProfileEngagement" "ScoobeSystemSettingEnabled" "Windows Suggestions Test"
    
    # Check hosts file length
    
    if ((Get-ItemProperty -Path C:\Windows\System32\drivers\etc\hosts).Length -eq 824) {
     write-host "Host File Length OK" -ForegroundColor green
    } else {
     $score--
     write-host "Host file has been edited" -ForegroundColor red
    }

Feel free to add to this script and post it elsewhere. Lastly, I tested this script on a number of Windows 10 systems and it worked fine. If you get any errors though let me know
