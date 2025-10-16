import smtplib
from email.message import EmailMessage

def send_email(subject, body, attach_log=False, log_file=None):
    msg = EmailMessage()
    msg["From"] = "kkhyt@kenvue.com"
    msg["To"] = "estp@kenvue.com"
    msg["Subject"] = subject
    msg.set_content(body)

    if attach_log and log_file and os.path.exists(log_file):
        with open(log_file, "rb") as f:
            msg.add_attachment(f.read(), maintype="text", subtype="plain", filename=os.path.basename(log_file))

    try:
        with smtplib.SMTP("smtp.kenvue.com", 25) as smtp:
            smtp.ehlo()           # Identify to server
            smtp.starttls()       # Upgrade connection to secure
            smtp.ehlo()           # Re-identify after TLS
            smtp.send_message(msg)
        print(f"Email sent successfully: {subject}")
    except Exception as e:
        print(f"Failed to send email: {e}")
