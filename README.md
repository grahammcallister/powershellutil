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

