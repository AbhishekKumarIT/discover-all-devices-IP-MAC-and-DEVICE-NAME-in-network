# discover-all-devices-IP-MAC-and-DEVICE-NAME-in-network
# change ip range as per your Network Range
(paste this script in powershell as administrate)

$Subnet = "192.168.1"
$OutputFile = "$env:USERPROFILE\Desktop\Network_Discovery.csv"

$Results = @()

Write-Host "Scanning Network..."

1..254 | ForEach-Object {

    $IP = "$Subnet.$_"

    if (Test-Connection -ComputerName $IP -Count 1 -Quiet -ErrorAction SilentlyContinue) {

        # Ping once to populate ARP
        ping -n 1 $IP | Out-Null

        # Get MAC Address
        $MAC = (arp -a $IP | Select-String $IP).ToString() -replace '.*?(([0-9a-f]{2}-){5}[0-9a-f]{2}).*','$1'

        # Get Host Name
        try {
            $HostName = ([System.Net.Dns]::GetHostEntry($IP)).HostName
        }
        catch {
            $HostName = "Unknown"
        }

        $Results += [PSCustomObject]@{
            IP_Address = $IP
            Host_Name  = $HostName
            MAC_Address = $MAC
        }

        Write-Host "$IP  $HostName  $MAC"
    }
}

$Results | Export-Csv -NoTypeInformation $OutputFile

Write-Host ""
Write-Host "Scan Complete."
Write-Host "Saved to $OutputFile"
