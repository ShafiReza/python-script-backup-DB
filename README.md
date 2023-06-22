# python-script-backup-DB

import os
import getpass
import datetime
import subprocess

# Read user input for source database credentials
SRC_DB_HOST = input("Enter source database host: ")
SRC_DB_USER = input("Enter source database username: ")
SRC_DB_PASS = getpass.getpass("Enter source database password: ")
SRC_DB_NAME = input("Enter source database name: ")

# Read user input for destination database credentials
DEST_DB_HOST = input("Enter destination database host: ")
DEST_DB_USER = input("Enter destination database username: ")
DEST_DB_PASS = getpass.getpass("Enter destination database password: ")
DEST_DB_NAME = input("Enter destination database name: ")

# Backup destination directory
BACKUP_DIR = "/path/to/backup/directory"

# Create backup directory if it doesn't exist
os.makedirs(BACKUP_DIR, exist_ok=True)

# Generate a timestamp for the backup file
timestamp = datetime.datetime.now().strftime("%Y%m%d%H%M%S")
backup_file = f"{SRC_DB_NAME}_backup_{timestamp}.sql"

# Run mysqldump command to backup the source database
backup_command = [
    "mysqldump",
    "--single-transaction",
    "-h", SRC_DB_HOST,
    "-u", SRC_DB_USER,
    f"-p{SRC_DB_PASS}",
    SRC_DB_NAME,
]
backup_path = os.path.join(BACKUP_DIR, backup_file)
with open(backup_path, "w") as backup_file:
    subprocess.run(backup_command, stdout=backup_file)

# Check if the backup was successful
if os.path.exists(backup_path):
    print(f"Backup created successfully: {backup_path}")
else:
    print("Backup failed!")
    exit(1)

# Restore the backup to the destination database
restore_command = [
    "mysql",
    "-h", DEST_DB_HOST,
    "-u", DEST_DB_USER,
    f"-p{DEST_DB_PASS}",
    DEST_DB_NAME,
    "<",
    backup_path,
]
restore_process = subprocess.Popen(" ".join(restore_command), shell=True)
restore_process.communicate()

# Check if the restore was successful
if restore_process.returncode == 0:
    print("Database restore completed successfully.")
else:
    print("Database restore failed!")
