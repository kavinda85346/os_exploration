### Network Management 

##### This script is used for network management.

##### Steps involved in the process!

1. Monitor network connectivity.
2. Ping Test.
3. Port connectivity test.
4. Restart network adapters.
5. Connect/Disconnect VPN.
6. Map/Unmap network drive.
7. Download resources from URL

```
#!/bin/zsh

# Define directories and variables
LOGFILE="$HOME/network_automation.log"
VPN_NAME="MyVPN"
DOWNLOAD_URL="https://example.com/sample.txt"
DEST_FILE="$HOME/sample.txt"
TARGET="google.com"
PORT=443

# Step 1. Monitor network connectivity.
echo "Commencing network connectivity monitoring process!"
ping -c 1 "$TARGET" >/dev/null 2>&1 && echo "Online" || echo "Offline"
echo "Ending network connectivity monitoring process!"

# Step 2. Ping Test
echo "Commencing ping test"
for host in 8.8.8.8 1.1.1.1 "$TARGET"; do
  ping -c 2 "$host" >/dev/null 2>&1 && echo "Ping OK: $host" || echo "Ping FAIL: $host"
done | tee -a "$LOGFILE"
echo "Ending ping test"

# Step 3. Port connectivity test
echo "Commencing port connectivity test"
nc -zv "$TARGET" "$PORT" &>>"$LOGFILE"
echo "Ending port connectivity test"

# Step 4. Restart network adapters
echo "Commencing network adapter restarting process!"
sudo ifconfig en0 down && sudo ifconfig en0 up
echo "Ending network adapter restarting process!"

# Step 5. Connect/Disconnect VPN.
echo "Commencing VPN management process!"
scutil --nc start "$VPN_NAME" || scutil --nc stop "$VPN_NAME"
echo "Ending VPN management process!"

# Step 6. Map/Unmap network drive
echo "Commencing Network Drive management process!"
mkdir -p /Volumes/Share
mount_smbfs //user@server/share /Volumes/Share
echo "Ending Network Drive management process!"

# Step 7. Download resources from URL
echo "Commencing resource download from URL process!"
curl -o "$DEST_FILE" "$DOWNLOAD_URL"
echo "Ending resource download from URL process!"

echo "[$(date)] Tasks completed." | tee -a "$LOGFILE"
```


