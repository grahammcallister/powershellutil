# powershellutil
Powershell snippets

## Send email
    function sendMail($sendTo, $subject, $body) {

        Write-Host "Sending Email"

        #SMTP server name
        $smtpServer = "smtp.demo.com"

        #Creating a Mail object
        $msg = new-object Net.Mail.MailMessage

        #Creating SMTP server object
        $smtp = new-object Net.Mail.SmtpClient($smtpServer)

        #Email structure 
        $msg.From = "donotreply@demo.com"
        $msg.ReplyTo = "donotreply@demo.com"
        $msg.To.Add($sendTo)
        $msg.subject = $subject + " " + [System.DateTime]::Now
        $msg.body = $body

        #Sending email 
        $smtp.Send($msg)
  
        Write-Host "Email sent: " + $users
    }

## IIS Administration

		# Utilities for working with IIS in PowerShell
		#
		# Useful IIS PoSH cheat sheet:
		# http://blogs.iis.net/jeonghwan/archive/2008/07/30/iis-powershell-user-guide-comparing-representative-iis-ui-tasks.aspx

		# Load-WebAdministration
		#
		# Tries to ensure Powershell IIS administration module or snapin is loaded
		# Script courtesy of http://forums.iis.net/t/1166784.aspx
		# See also http://forums.iis.net/t/1189482.aspx
		function Load-WebAdministration
		{
			Write-Host "Loading WebAdministration"
			
			$ModuleName = "WebAdministration"
			$ModuleLoaded = $false
			$LoadAsSnapin = $false

			if ($PSVersionTable.PSVersion.Major -ge 2)
			{
				if ((Get-Module -ListAvailable | ForEach-Object {$_.Name}) -contains $ModuleName)
				{
					Write-Host "Loading as Module"
					Import-Module $ModuleName
					if ((Get-Module | ForEach-Object {$_.Name}) -contains $ModuleName)
					{
						$ModuleLoaded = $true
						Write-Host "Loaded WebAdministration"
					}
					else
					{
						$LoadAsSnapin = $true
					}
				}
				elseif ((Get-Module | ForEach-Object {$_.Name}) -contains $ModuleName)
				{
					Write-Host "Already loaded WebAdministration as module"
					$ModuleLoaded = $true
				}
				else
				{
					$LoadAsSnapin = $true
				}
			}
			else
			{
				$LoadAsSnapin = $true
			}

			if ($LoadAsSnapin)
			{
				if ((Get-PSSnapin -Registered | ForEach-Object {$_.Name}) -contains $ModuleName)
				{
					Write-Host "Loading WebAdministration as Snapin"
					Add-PSSnapin $ModuleName
					if ((Get-PSSnapin | ForEach-Object {$_.Name}) -contains $ModuleName)
					{
						$ModuleLoaded = $true
					}
				}
				elseif ((Get-PSSnapin | ForEach-Object {$_.Name}) -contains $ModuleName)
				{
					$ModuleLoaded = $true
				}
			}
			
			Write-Host "Done Load-WebAdministration"
		}


		# Toggle-Authentication($websiteName, $authenticationType, $enabled)
		#
		# Sets authentication enabled/disabled for a given site and authentication type
		# to true or false.
		# If website is not found, does nothing
		# If authentication type is not present on the site, does nothing
		#
		function Toggle-Authentication ($websiteName = "Default Web Site", $authenticationType = "anonymousAuthentication", $enabled = $true) {
			
			# Get the web site - see http://forums.iis.net/p/1167298/1943273.aspx
			# Can't just use Get-Website :-(
			# http://stackoverflow.com/questions/4170530/powershell-get-website-name-parameter-is-ignored

			Write-Host "Attempting to toggle $websiteName $authenticationType $enabled"

			$web = Get-Website | where { $_.Name -eq $websiteName }
			
			if($web) {

				$authentications = Get-WebConfiguration `
								   -filter "system.webServer/security/authentication/*" `
								   -PSPath "IIS:\Sites\$websiteName"

				foreach ($auth in $authentications)
				{
					# Nice Regex in Powershell article
					# http://www.johndcook.com/regex.html
					if($auth.SectionPath -match $authenticationType)
					{
						# Had trouble with "Locked" configuration - needed to read this - Dealing with Locked Sections at
						# http://learn.iis.net/page.aspx/436/powershell-snap-in-changing-simple-settings-in-configuration-sections/
							Write-Host "Setting $authenticationType to $enabled"
							Set-WebConfigurationProperty `
							 -filter $auth.SectionPath `
							 -name enabled -value $enabled -PSPath "IIS:\" -location $websiteName
					}
				}
				
			} # end if (get web)
			
			Write-Host "Done Toggle-Authentication"

		} # end Toggle-Authentication

		# Add-Binding($websiteName, $protocol, $host, $port)
		#
		# Adds a binding to a web site using protocol, host and port
		# If website is not found, does nothing
		# If binding already exists, does nothing
		#
		function Add-Binding ($websiteName = "Default Web Site", $protocol = "http", $ipAddress = "*", $hostHeader = "", $port = "80") {
			
			# Get the web site - see http://forums.iis.net/p/1167298/1943273.aspx
			# Can't just use Get-Website :-(
			# http://stackoverflow.com/questions/4170530/powershell-get-website-name-parameter-is-ignored

			Write-Host "Attempting to add binding to web: $websiteName protocol: $protocol ipAddress: $ipAddress hostHeader: $hostHeader port: $port"

			$web = Get-Website | where { $_.Name -eq $websiteName }
			
			if($web) {

				Write-Host "Found site"
				Write-Host
							
				New-WebBinding -Name $websiteName -Protocol $protocol -Port $port -IpAddress $ipAddress -HostHeader $hostHeader -Force
					   
			} # end if (get web)
			
			Write-Host "Done Add-Binding"

		} # end Add-Binding

		Write-Host

		Load-WebAdministration

		Write-Host

		$ANONYMOUS = "anonymousAuthentication"
		$NTLM = "windowsAuthentication"

		$websiteName = "Default Web Site"

		Toggle-Authentication $websiteName $ANONYMOUS $false

		Write-Host

		Toggle-Authentication $websiteName $NTLM $true

		Add-Binding $websiteName "http" "*" "localhost" "80"
