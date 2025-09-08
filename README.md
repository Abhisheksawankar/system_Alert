# system_Alert
#pip install psutil

import psutil
import logging
from logging.handlers import RotatingFileHandler
import smtplib
from email.mime.text import MIMEText
import time

LOG_FILE = "system_monitor.log"
LOG_SIZE = 5 * 1024 * 1024
BACKUP_COUNT = 3
INTERVAL = 20

# alerts (%)
CPU_THRESHOLD = 60
MEM_THRESHOLD = 75
DISK_THRESHOLD = 60


SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
EMAIL_FROM = "your_email@gmail.com"
EMAIL_TO = "receiver_email@gmail.com"
EMAIL_PASSWORD = "none"


# === Logging Setup ===
logger = logging.getLogger("SystemMonitor")
logger.setLevel(logging.INFO)
handler = RotatingFileHandler(LOG_FILE, maxBytes=LOG_SIZE, backupCount=BACKUP_COUNT)
formatter = logging.Formatter("%(asctime)s - %(levelname)s - %(message)s")
handler.setFormatter(formatter)
logger.addHandler(handler)


def send_email(subject, body):

    try:
        msg = MIMEText(body)
        msg["Subject"] = subject
        msg["From"] = EMAIL_FROM
        msg["To"] = EMAIL_TO

        with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server:
            server.starttls()
            server.login(EMAIL_FROM, EMAIL_PASSWORD)
            server.sendmail(EMAIL_FROM, EMAIL_TO, msg.as_string())
        print(f"📧 Email sent: {subject}")
    except Exception as e:
        print(f"❌ Failed to send email: {e}")


def log_and_alert(metric, value, threshold):
    """Log system usage and send email if threshold exceeded."""
    message = f"{metric} usage: {value:.2f}%"
    if value > threshold:
        logger.warning(f"ALERT! {message} (Threshold: {threshold}%)")
        print(f"⚠️ ALERT: {message} (Threshold: {threshold}%)")
        send_email(f"⚠️ System Alert: {metric} High Usage", message)
    else:
        logger.info(message)


def monitor_system():

    while True:
        cpu_usage = psutil.cpu_percent(interval=1)
        mem_usage = psutil.virtual_memory().percent
        disk_usage = psutil.disk_usage("/").percent

        log_and_alert("CPU", cpu_usage, CPU_THRESHOLD)
        log_and_alert("Memory", mem_usage, MEM_THRESHOLD)
        log_and_alert("Disk", disk_usage, DISK_THRESHOLD)

        time.sleep(INTERVAL)


if __name__ == "__main__":
    print("🚀 Starting System Monitor... Press Ctrl+C to stop.")
    try:
        monitor_system()
    except KeyboardInterrupt:
        print("\n🛑 System Monitor stopped.")

















