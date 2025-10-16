import smtplib
from email.message import EmailMessage
import os

# --- Email settings ---
SMTP_SERVER = "smtp.kenvue.com"
SMTP_PORT = 25
SENDER = "kkhyt@kenvue.com"
RECIPIENTS = ["estp@kenvue.com"]
LOG_FILE = r"C:\temp\DispatcherRestart.log"  # optional attachment

# --- Function to send email ---
def send_email(subject, body, attach_log=False):
    msg = EmailMessage()
    msg["From"] = SENDER
    msg["To"] = ", ".join(RECIPIENTS)
    msg["Subject"] = subject
    msg.set_content(body)

    if attach_log and os.path.exists(LOG_FILE):
        with open(LOG_FILE, "rb") as f:
            msg.add_attachment(f.read(), maintype="text", subtype="plain", filename=os.path.basename(LOG_FILE))

    try:
        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as smtp:
            smtp.ehlo()
            smtp.starttls()
            smtp.ehlo()
            smtp.send_message(msg)
        print(f"Email sent successfully: {subject}")
    except Exception as e:
        print(f"Failed to send email: {e}")

# --- Test email ---
send_email("Test Email from Python", "Hello mama, this is a test email!", attach_log=False)
