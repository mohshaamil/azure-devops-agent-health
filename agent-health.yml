trigger:
- none

pool:
  vmImage: 'windows-latest'

steps:
- task: PowerShell@2
  displayName: "Monitor Agent Health"
  inputs:
    targetType: 'inline'
    script: |
      # Get CPU usage
      $cpuUsage = Get-WmiObject Win32_Processor | Measure-Object -Property LoadPercentage -Average | Select-Object -ExpandProperty Average

      # Get Disk utilization
      $diskUsage = Get-WmiObject Win32_LogicalDisk | Where-Object { $_.DriveType -eq 3 } |
        ForEach-Object {
          [PSCustomObject]@{
            Drive = $_.DeviceID
            FreeSpace = ($_.FreeSpace / 1GB).ToString("0.00") + " GB"
            TotalSpace = ($_.Size / 1GB).ToString("0.00") + " GB"
            UsedPercentage = [math]::Round((($_.Size - $_.FreeSpace) / $_.Size) * 100, 2)
          }
        }

      # Output CPU usage
      Write-Output "CPU Usage: $cpuUsage%"

      # Output Disk usage
      $diskUsage | ForEach-Object {
        Write-Output "Drive: $($_.Drive)"
        Write-Output "  Free Space: $($_.FreeSpace)"
        Write-Output "  Total Space: $($_.TotalSpace)"
        Write-Output "  Used Percentage: $($_.UsedPercentage)%"
      }

      # Optionally, fail the pipeline if thresholds are exceeded
      if ($cpuUsage -gt 80) {
        Write-Error "CPU usage exceeds 80%"
      }

      foreach ($disk in $diskUsage) {
        if ($disk.UsedPercentage -gt 90) {
          Write-Error "Disk usage on $($disk.Drive) exceeds 90%"
        }
      }
