### Security Management

##### This script is used for vulnerability monitoring and safeguarding the system.

##### Steps involved in the process!

1. Monitor system logs for keywords
2. Audit login history
3. Check for failed logins
4. Automatically block suspicious IPs
5. Manage firewall rules
6. Run antivirus or malware scans
7. Encrypt / decrypt files

```
#!/usr/bin/env bash
LOG_KEYWORDS=("error" "failed" "unauthorized" "ssh")
FAILED_THRESHOLD=5
BLOCK_DURATION=3600
BLOCK_DB="/var/tmp/macos_blocked_ips.db"
CLAMSCAN_CMD="$(command -v clamscan || true)"

log(){ echo "[$(date -Is)] $*"; }

ensure_block_db(){ mkdir -p "$(dirname "$BLOCK_DB")"; touch "$BLOCK_DB"; }

cleanup_expired(){
  [[ -f "$BLOCK_DB" ]] || return
  tmp=$(mktemp)
  while read -r ip expires; do
    [[ -z "$ip" ]] && continue
    if (( $(date +%s) > expires )); then
      log "Unblocking $ip (expired)"
      sudo pfctl -t blocklist -T delete "$ip" 2>/dev/null || true
    else
      echo "$ip $expires" >> "$tmp"
    fi
  done < "$BLOCK_DB"
  mv "$tmp" "$BLOCK_DB"
}

# Step 1. Monitor system logs for keywords
echo "Commencing system log monitoring process!"
monitor_logs_once(){
  log "Searching macOS logs for keywords: ${LOG_KEYWORDS[*]}"
  log show --last 1h --style syslog 2>/dev/null | grep -iE "$(IFS="|"; echo "${LOG_KEYWORDS[*]}")" || true
}

monitor_logs_follow(){
  log "Following console output for keywords (this uses 'log stream')"
  log stream --style syslog --predicate 'true' | grep --line-buffered -iE "$(IFS="|"; echo "${LOG_KEYWORDS[*]}")"
}
echo "Ending system log monitoring process!"

# Step 2. Audit login history
echo "Commencing audting login history process!"
audit_login_history(){
  log "=== last -20 ==="
  last -20
}
echo "Ending audting login history process!"

# Step 3. Check for failed logins
echo "Commencing failed login checking process!"
check_failed_logins(){
  log "Parsing /var/log/system.log and unified log for failed auth entries"
  sudo grep -i "failed" /var/log/system.log || true
  log show --last 1d --style syslog | grep -i "failed" || true
  # extract IPv4 addresses from system.log
  sudo awk '/failed/ {for(i=1;i<=NF;i++) if ($i ~ /^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$/) print $i }' /var/log/system.log \
    | sort | uniq -c | sort -nr | awk '$1>0 {print $2, $1}' || true
}
echo "Ending failed login checking process!"

# Step 4. Automatically block suspicious IPs by adding to pf table <blocklist>
echo "Commencing suspicious IP blocking process!"
auto_block(){
  ensure_block_db; cleanup_expired
  # find IPs from last 1000 lines of system.log
  sudo awk '/failed/ {for(i=1;i<=NF;i++) if ($i ~ /^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$/) print $i }' /var/log/system.log \
    | tail -n 1000 | sort | uniq -c | while read -r cnt ip; do
      if (( cnt >= FAILED_THRESHOLD )); then
        if grep -q "^$ip " "$BLOCK_DB" 2>/dev/null; then
          log "$ip already recorded"
        else
          log "Adding $ip to pf blocklist"
          sudo pfctl -t blocklist -T add "$ip"
          echo "$ip $(( $(date +%s) + BLOCK_DURATION ))" >> "$BLOCK_DB"
        fi
      fi
    done
}
echo "Ending suspicious IP blocking process!"

# Step 5. Manage firewall rules
echo "Commencing firewall rules management process!"
# For simplicity we show how to add/remove PF rule to allow port (non-persistent example)
allow_port_pf(){
  local port="$1"
  sudo pfctl -a com.apple -f - <<EOF
pass in proto tcp from any to any port $port
EOF
  log "Temporary allowed TCP port $port via pf anchor load (non-persistent)"
}
echo "Ending firewall rules management process!"

# Step 6. Run antivirus or malware scans
echo "Commencing antivirus scan running process!"
run_av(){
  if [[ -n "$CLAMSCAN_CMD" ]]; then
    sudo $CLAMSCAN_CMD -r -i /
  else
    log "ClamAV not found. Consider installing via brew install clamav or use your AV solution."
  fi
}
echo "Ending antivirus scan running process!"

# Step 7. Encrypt / decrypt files
echo "Commencing encrypting / decrypting files process!"
encrypt_file(){ local in="$1"; local out="${2:-$in.enc}"; read -s -p "Password: " pw; echo; openssl enc -aes-256-cbc -salt -pbkdf2 -in "$in" -out "$out" -pass pass:"$pw"; log "Encrypted $in -> $out"; }
decrypt_file(){ local in="$1"; local out="${2:-${in%.enc}.dec}"; read -s -p "Password: " pw; echo; openssl enc -aes-256-cbc -d -pbkdf2 -in "$in" -out "$out" -pass pass:"$pw"; log "Decrypted $in -> $out"; }|
echo "Ending encrypting / decrypting files process!"

case "$1" in
  monitor_once) monitor_logs_once ;;
  monitor_follow) monitor_logs_follow ;;
  audit_logins) audit_login_history ;;
  check_failed) check_failed_logins ;;
  auto_block) auto_block ;;
  allow_port) allow_port_pf "$2" ;;
  run_av) run_av ;;
  encrypt) encrypt_file "$2" "$3" ;;
  decrypt) decrypt_file "$2" "$3" ;;
  cleanup_blocks) cleanup_expired ;;
  *) echo "Usage: $0 {monitor_once|monitor_follow|audit_logins|check_failed|auto_block|allow_port <port>|run_av|encrypt|decrypt|cleanup_blocks}" ;;
esac
```


