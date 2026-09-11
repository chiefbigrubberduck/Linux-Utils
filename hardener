#!/bin/bash

TEST=$(echo "Testing..."
echo "If this works, its not good, the hardening script failed")

# 1. Get the path safely
echo -n "Enter the absolute path to the folder to isolate (e.g., /home/USER/Downloads): "
read TARGET_PATH

# 2. Validate the path exists
if [ ! -d "$TARGET_PATH" ]; then
    echo "Error: That path does not exist or is not a directory."
    exit 1
fi

# 3. Safety Check: Don't lock root or home entirely
if [ "$TARGET_PATH" = "/" ] || [ "$TARGET_PATH" = "/home" ]; then
    echo "Error: Refusing to lock root (/) or /home directly. Can break system"
    exit 1
fi

echo "Folder found. Preparing to lock..."

# 4. Backup fstab
sudo cp /etc/fstab /etc/fstab.backup.$(date +%F)
echo "Backed up fstab to /etc/fstab.backup.$(date +%F)"

LINE="$TARGET_PATH $TARGET_PATH none bind,noexec 0 0"
echo "$LINE" | sudo tee -a /etc/fstab > /dev/null

echo "Updated /etc/fstab."

# 6. Reload systemd
sudo systemctl daemon-reload
echo "Systemd reloaded."

# 7. Mount the folder
sudo mount "$TARGET_PATH"

# 8. Verify
echo "Verifying mount..."
echo "$TEST" | sudo tee -a "$TARGET_PATH"test.sh > /dev/null
sudo chmod +x "$TARGET_PATH"test.sh

cd "$TARGET_PATH"

if "$TARGET_PATH/test.sh" >/dev/null 2>&1; then
    echo "FAILED! Test script executed despite noexec. Mount may not have applied."
    echo "Done..."
    exit 1
else
    echo "SUCCESS! Folder is locked with noexec."
    echo "Run 'mount | grep "$TARGET_PATH"' to verify mount options."
    echo "Nothing should show up"
    echo "Done..."
    exit 0
fi
rm test.sh
