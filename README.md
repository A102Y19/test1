import subprocess
import logging
import os
import sys
import smtplib
from email.message import EmailMessage
from datetime import datetime

# --- CONFIG ---
STATUS_FILE = r"\\conusea1fs01-1.jx2.com\oradbworkjx2\SRAMAC01\MARD_PQAstatus_check.log\dispatcher_status"
TASK_NAME = r"OMP Dispatcher\OMP_KNV_CO_MADRID_PQA"
LOG_FILE = r"C:\temp\DispatcherRestart.log"  # Make sure C:\temp exists

# --- EMAIL SETTINGS ---
SMTP_SERVER = "smtp.kenvue.com"
SMTP_PORT = 25  # STARTTLS will be used
SENDER = "kkhyt@kenvue.com"
RECIPIENTS = ["estp@kenvue.com"]

# --- Logging setup ---
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
    handlers=[
        logging.StreamHandler(),
        logging.FileHandler(LOG_FILE, mode="a", encoding="utf-8")
    ]
)

# --- Helper: send email ---
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
        logging.info(f"Email sent successfully: {subject}")
    except Exception as e:
        logging.error(f"Failed to send email: {e}")

# --- Check status file ---
def is_file_empty(path):
    if not os.path.exists(path):
        logging.error(f"Status file missing: {path}")
        send_email(
            subject="Dispatcher Check Failed - File Missing",
            body=(
                f"Dispatcher status file not found:\n{path}\n\n"
                f"Timestamp: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}"
            )
        )
        sys.exit(1)
    return os.path.getsize(path) == 0

# --- Run local task ---
def run_local_task(task):
    cmd = ["schtasks.exe", "/run", "/TN", task]
    result = subprocess.run(cmd, capture_output=True, text=True)
    return result

# --- Main Execution ---
logging.info(f"Checking dispatcher status file: {STATUS_FILE}")

if not is_file_empty(STATUS_FILE):
    logging.info("Status file not empty → Dispatcher needs restart.")
    logging.info(f"Running task: {TASK_NAME}")
    result = run_local_task(TASK_NAME)

    if result.returncode != 0:
        logging.error(f"Failed to run task: {result.stderr.strip()}")
        send_email(
            subject="Dispatcher Restart Failed",
            body=(
                f"Dispatcher restart failed with error:\n\n"
                f"{result.stderr.strip()}\n\nCheck logs at: {LOG_FILE}"
            ),
            attach_log=True
        )
    else:
        logging.info("Dispatcher restarted successfully.")
        send_email(
            subject="Dispatcher Restart Successful",
            body="Dispatcher restart completed successfully.\n\nSee attached log for details.",
            attach_log=True
        )
else:
    logging.info("Dispatcher already up and running — no action taken.")
