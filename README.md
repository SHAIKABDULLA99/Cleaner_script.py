# Cleaner_script.py
https://github.com/SHAIKABDULLA99/Cleaner_script.py.git
import os
import shutil
import logging
from datetime import datetime, timedelta

# Initialize system logging
logging.basicConfig(
    filename="cleanup_operations.log",
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)

def run_directory_cleanup(folder_path, days_old=30):
    """Finds, logs, and cleans up old or temporary files in a directory."""
    folder_path = os.path.abspath(folder_path)
    
    # Validation checks using OS module
    if not os.path.exists(folder_path):
        logging.error(f"Target path does not exist: {folder_path}")
        print(f"Error: Directory target '{folder_path}' was not found.")
        return

    print(f"\nScanning directory: {folder_path}")
    logging.info(f"Directory scan initialized for: {folder_path}")

    # Set up a safe archival folder for items to be removed
    archive_dir = os.path.join(folder_path, "Cleaned_Archive")
    time_threshold = datetime.now() - timedelta(days=days_old)
    
    cleaned_count = 0

    try:
        # Loop through files in the target directory
        for item in os.listdir(folder_path):
            item_path = os.path.join(folder_path, item)

            # Skip directories (including our own archive folder)
            if os.path.isdir(item_path):
                continue

            try:
                # Retrieve file metadata (last modification time)
                mod_time = datetime.fromtimestamp(os.path.getmtime(item_path))
                
                # Check if file matches cleanup criteria (older than threshold or temporary file)
                is_old = mod_time < time_threshold
                is_temp = item.endswith(('.tmp', '.log_old', '.bak'))

                if is_old or is_temp:
                    if not os.path.exists(archive_dir):
                        os.makedirs(archive_dir)
                    
                    # Safely relocate the file
                    shutil.move(item_path, os.path.join(archive_dir, item))
                    
                    log_entry = f"Archived file: {item} (Reason: {'Old' if is_old else 'Temp Extension'})"
                    logging.info(log_entry)
                    print(log_entry)
                    cleaned_count += 1

            except PermissionError:
                logging.warning(f"Permission denied while reading file: {item}")
                print(f"Skipped (No Access): {item}")
            except Exception as e:
                logging.error(f"Error handling file {item}: {e}")
                print(f"Skipped (Error encountered): {item}")

    except Exception as e:
        logging.critical(f"Failed to complete directory scan: {e}")
        print(f"Critical System Error: {e}")
        return

    print(f"\nCleanup complete. Processed and moved {cleaned_count} files.")
    logging.info(f"Scan finished. Total items archived: {cleaned_count}")

if __name__ == "__main__":
    print("=== Automated System File Cleaner ===")
    
    # Handle user interaction
    target_dir = input("Enter path of the directory to clean: ").strip()
    age_input = input("Archive items older than how many days? (Default 30): ").strip()
    
    # Parse age input with fallback error handling
    try:
        days = int(age_input) if age_input else 30
    except ValueError:
        print("Invalid number entered. Reverting to default of 30 days.")
        days = 30

    run_directory_cleanup(target_dir, days)
    
