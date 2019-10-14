# Search_DHCP.ps1
#DHCP_Search_and_Reservation_Tool in Shell
Function Create-Form {

	Add-Type -AssemblyName System.Windows.Forms
    Add-Type -AssemblyName System.Drawing
	
	$Form = New-Object System.Windows.Forms.Form
	$Form.Size = New-Object System.Drawing.Size(840,425)
	$Form.Text = "Search DHCP"
	$Form.StartPosition = "CenterScreen"
	
	#Text fields
	
	$SearchMACTextbox = New-Object System.Windows.Forms.TextBox
	$SearchMACTextbox.Location = New-Object System.Drawing.Size(160,13)
	$SearchMACTextbox.Size = New-Object System.Drawing.Size(110,15)
	$Form.Controls.Add($SearchMACTextbox)
	
	#Source path label
	$SearchMACLabel = New-Object System.Windows.Forms.Label
	$SearchMACLabel.Text="Search by MAC Address: "
	$SearchMACLabel.Location = New-Object System.Drawing.Size(15,15) 
	$SearchMACLabel.Size = New-Object System.Drawing.Size(150,15) 
	$Form.Controls.Add($SearchMACLabel)
	
	$SearchNameTextbox = New-Object System.Windows.Forms.TextBox
	$SearchNameTextbox.Location = New-Object System.Drawing.Size(160,40)
	$SearchNameTextbox.Size = New-Object System.Drawing.Size(110,15)
	$Form.Controls.Add($SearchNameTextbox)
	
	#Target path label
	$SearchNameLabel = New-Object System.Windows.Forms.Label
	$SearchNameLabel.Text="Search by machine name: "
	$SearchNameLabel.Location = New-Object System.Drawing.Size(15,43) 
	$SearchNameLabel.Size = New-Object System.Drawing.Size(595,15) 
	$Form.Controls.Add($SearchNameLabel)
	
	#Output box
	$outputBox = New-Object System.Windows.Forms.RichTextBox 
	$outputBox.Location = New-Object System.Drawing.Size(15,75) 
	$outputBox.Size = New-Object System.Drawing.Size(610,290)
	$outputBox.MultiLine = $True
	$outputBox.WordWrap = $False
	$outputBox.ScrollBars = "Both"
	$outputBox.Font = "Courier New"
	$Form.Controls.Add($outputBox)
	
	#end text fields
	
	#Start buttons
	
	#Button Start
	$ButtonStart = New-Object System.Windows.Forms.Button 
	$ButtonStart.Location = New-Object System.Drawing.Size(640,75) 
	$ButtonStart.Size = New-Object System.Drawing.Size(175,80) 
	$ButtonStart.Text = "START SEARCH" 
	$ButtonStart.BackColor = "Green" 
	$ButtonStart.Add_Click({Search-DHCP})
	$Form.Controls.Add($ButtonStart) 
	
	#Button Clear
	$ButtonClear = New-Object System.Windows.Forms.Button 
	$ButtonClear.Location = New-Object System.Drawing.Size(640,165) 
	$ButtonClear.Size = New-Object System.Drawing.Size(175,80) 
	$ButtonClear.Text = "CLEAR RESULTS" 
	$ButtonClear.BackColor = "Red" 
	$ButtonClear.Add_Click({Clear-Textbox})
	$Form.Controls.Add($ButtonClear)

    #Button Create-Reservation
	$ButtonCreateReservation = New-Object System.Windows.Forms.Button 
	$ButtonCreateReservation.Location = New-Object System.Drawing.Size(640,255) 
	$ButtonCreateReservation.Size = New-Object System.Drawing.Size(175,45) 
	$ButtonCreateReservation.Text = "CREATE RESERVATION" 
	$ButtonCreateReservation.BackColor = "Yellow" 
	$ButtonCreateReservation.Add_Click({Create-Reservation})
	$Form.Controls.Add($ButtonCreateReservation)
	
	#end buttons
	
	
	$Form.Add_Shown({$Form.Activate()})
	[void]$Form.ShowDialog()

}

Function Search-DHCP {

    $result = @{}
	
    if ( $SearchMACTextbox.Text -notlike $null ) {
        $outputBox.Text = " Searching DHCP for MAC address " + $SearchMACTextbox.Text + "`r`n"
        $result = ( Get-DhcpServerv4Scope -ComputerName "IP-ADDRESS-HERE"  | Get-DhcpServerv4Lease -ComputerName "IP-ADDRESS-HERE" -ErrorAction SilentlyContinue | ? ClientId -match $SearchMACTextbox.Text | select ipaddress,clientid,hostname | Format-List | Out-String )           
    }
    elseif ( $SearchNameTextbox.Text -notlike $null ) {
        $outputBox.Text = " Searching DHCP for machine name " + $SearchNameTextbox.Text + "`r`n"
        $result = ( Get-DhcpServerv4Scope -ComputerName "IP-ADDRESS-HERE"  | Get-DhcpServerv4Lease -ComputerName "IP-ADDRESS-HERE" -ErrorAction SilentlyContinue | ? Hostname -match $SearchNameTextbox.Text | select ipaddress,clientid,hostname | Format-List | Out-String )
    } else { $outputBox.Text = " Please enter a search query " }

    $outputBox.AppendText($result)
	
}

Function Clear-Textbox {

    $SearchMACTextbox.Text = ''
    $SearchNameTextbox.Text = ''
    $outputBox.Text = ''
	$result = ''

}

Function Create-Reservation {

    $reservation = @{}

    [void][Reflection.Assembly]::LoadWithPartialName('Microsoft.VisualBasic')

    $title = 'Create DHCP Reservation'
    $msg   = 'Enter IP address to reserve:'
    $ip = [Microsoft.VisualBasic.Interaction]::InputBox($msg, $title)

    $outputBox.AppendText(" Converting " + $ip + " to lease... `r`n")

    $reservation = Get-DhcpServerv4Lease -ComputerName "IP-ADDRESS-HERE" -ErrorAction SilentlyContinue -IPAddress $ip
    Add-DhcpServerv4Reservation -ComputerName "IP-ADDRESS-HERE" -ScopeId $reservation.ScopeId -IPAddress $reservation.IPAddress -ClientId $reservation.ClientId
    $checkReservation = (Get-DhcpServerv4Lease -ComputerName "IP-ADDRESS-HERE" -ErrorAction SilentlyContinue -IPAddress $ip).AddressState
    if ($checkReservation -match "Reservation") { 
        $outputBox.AppendText(' Lease has been successfully converted to reservation. ') 
    } else { $outputBox.AppendText(' Lease has been successfully converted to reservation. ') }

}

Create-Form
