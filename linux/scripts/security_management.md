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
LOG_KEYWORDS=("error" "failed" "unauthorized" "ssh" "panic")
AUTH_LOG="/var/log/auth.log"   # Debian/Ubuntu; on RHEL/CentOS use /var/log/secure or use journalctl
USE_JOURNALCTL=true            # set false to use logfile tailing
FAILED_THRESHOLD=5             # number of failed attempts from same IP to trigger block
BLOCK_DURATION_SECONDS=3600    # how long to block (script stores expiry)
BLOCKED_DB="/var/tmp/blocked_ips.db"
CLAMSCAN_CMD="$(command -v clamscan || true)"
IPTABLES_CMD="$(command -v iptables || command -v iptables-legacy || true)"
ENCRYPT_ALGO="aes-256-cbc"

log() { echo "[$(date -Is)] $*"; }

ensure_block_db(){
  mkdir -p "$(dirname "$BLOCKED_DB")"
  touch "$BLOCKED_DB"
}

record_block(){ # ip expiry
  echo "$1 $2" >> "$BLOCKED_DB"
}

cleanup_expired_blocks(){
  if [[ ! -f "$BLOCKED_DB" ]]; then return; fi
  tmp=$(mktemp)
  while read -r ip expires; do
    [[ -z "$ip" ]] && continue
    if (( $(date +%s) > expires )); then
      log "Unblocking expired IP $ip"
      $IPTABLES_CMD -D INPUT -s "$ip" -j DROP 2>/dev/null || true
    else
      echo "$ip $expires" >> "$tmp"
    fi
  done < "$BLOCKED_DB"
  mv "$tmp" "$BLOCKED_DB"
}

block_ip(){
  local ip="$1"
  local expiry=$(( $(date +%s) + BLOCK_DURATION_SECONDS ))
  log "Blocking IP $ip until $(date -d "@$expiry" -Is 2>/dev/null || date -Iseconds -d "@$expiry")"
  $IPTABLES_CMD -I INPUT -s "$ip" -j DROP
  record_block "$ip" "$expiry"
}

# Step 1. Monitor logs for keywords
echo "Commencing monitoring logs process!"
monitor_logs_once(){
  log "Searching logs for keywords: ${LOG_KEYWORDS[*]}"
  if $USE_JOURNALCTL && command -v journalctl >/dev/null 2>&1; then
    journalctl -n 1000 --no-pager | grep -iE "$(IFS="|"; echo "${LOG_KEYWORDS[*]}")" || true
  else
    if [[ -f "$AUTH_LOG" ]]; then
      tail -n 1000 "$AUTH_LOG" | grep -iE "$(IFS="|"; echo "${LOG_KEYWORDS[*]}")" || true
    else
      log "No auth log at $AUTH_LOG and journald not used."
    fi
  fi
}

monitor_logs_follow(){
  log "Starting follow-mode log monitor (CTRL-C to stop)"
  if $USE_JOURNALCTL && command -v journalctl >/dev/null 2>&1; then
    journalctl -f | grep --line-buffered -iE "$(IFS="|"; echo "${LOG_KEYWORDS[*]}")"
  else
    tail -F "$AUTH_LOG" | grep --line-buffered -iE "$(IFS="|"; echo "${LOG_KEYWORDS[*]}")"
  fi
}
echo "Ending monitoring logs process!"

# Step 2. Audit login history
echo "Commencing auditing login history!"
audit_login_history(){
  log "=== Last logins (last 20) ==="
  if command -v last >/dev/null 2>&1; then
    last -n 20
  else
    log "last command not available"
  fi
}
echo "Ending auditing login history!"

# Step 3. Check for failed logins
echo "Commencing failed login checking process!"
check_failed_logins(){
  log "Checking failed login events and aggregating by IP..."
  if command -v journalctl >/dev/null 2>&1; then
    journalctl -u ssh -o short -n 1000 --no-pager | grep -i "failed" || true
  elif [[ -f "$AUTH_LOG" ]]; then
    grep -i "failed" "$AUTH_LOG" || true
  else
    log "Cannot find auth logs to parse failed logins."
  fi

  # simple extraction of IPv4 from auth log
  if [[ -f "$AUTH_LOG" ]]; then
    awk '/failed/ {for(i=1;i<=NF;i++) if ($i ~ /^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$/) print $i }' "$AUTH_LOG" \
      | sort | uniq -c | sort -nr | awk '$1>0 {print $2, $1}' || true
  fi
}
echo "Ending failed login checking process!"

# Step 4. Automatically block suspicious IPs
echo "Commencing suspicious IP blocking process!"
auto_block_from_authlog(){
  ensure_block_db
  cleanup_expired_blocks

  if [[ ! -f "$AUTH_LOG" ]]; then
    log "No auth log found at $AUTH_LOG; skipping auto-block."
    return
  fi

  # find IPs with failed attempts in last N lines
  awk '/failed/ {for(i=1;i<=NF;i++) if ($i ~ /^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$/) print $i }' "$AUTH_LOG" \
    | tail -n 1000 | sort | uniq -c | while read -r cnt ip; do
      if (( cnt >= FAILED_THRESHOLD )); then
        # check if already blocked
        if grep -q "^$ip " "$BLOCKED_DB" 2>/dev/null; then
          log "$ip already in blocked DB"
        else
          block_ip "$ip"
        fi
      fi
    done
}
echo "Ending suspicious IP blocking process!"

# Step 5. Manage firewall rules
echo "Commencing firewall rule management process!"
add_firewall_rule_allow_port(){
  local port="$1"
  $IPTABLES_CMD -I INPUT -p tcp --dport "$port" -m conntrack --ctstate NEW -j ACCEPT
  log "Allowed TCP port $port"
}

remove_firewall_rule_allow_port(){
  local port="$1"
  $IPTABLES_CMD -D INPUT -p tcp --dport "$port" -m conntrack --ctstate NEW -j ACCEPT 2>/dev/null || true
  log "Removed allow rule for TCP port $port (if existed)"
}
echo "Ending firewall rule management process!"

# Step 6. Run antivirus or malware scans
echo "Commencing antivirus scan running process!"
run_antivirus_scan(){
  if [[ -n "$CLAMSCAN_CMD" ]]; then
    log "Running ClamAV scan on / (this may take time). Consider scanning specific paths."
    $CLAMSCAN_CMD -r --bell -i /
  else
    log "No clamscan found. Install ClamAV (apt install clamav) or adapt scan to your AV."
  fi
}
echo "Ending antivirus scan running process!"

# Step 7. Encrypt / decrypt files
echo "Commencing encrypting and decrypting files process!"
encrypt_file(){
  local infile="$1"; local outfile="${2:-$infile.enc}"
  if [[ -z "$infile" || ! -f "$infile" ]]; then
    echo "Usage: encrypt_file <infile> [outfile]"
    return 1
  fi
  read -s -p "Enter password for encryption: " pw; echo
  openssl enc -"${ENCRYPT_ALGO}" -salt -pbkdf2 -in "$infile" -out "$outfile" -pass pass:"$pw"
  log "Encrypted $infile -> $outfile"
}

decrypt_file(){
  local infile="$1"; local outfile="${2:-${infile%.enc}.dec}"
  if [[ -z "$infile" || ! -f "$infile" ]]; then
    echo "Usage: decrypt_file <infile> [outfile]"
    return 1
  fi
  read -s -p "Enter password for decryption: " pw; echo
  openssl enc -"${ENCRYPT_ALGO}" -d -pbkdf2 -in "$infile" -out "$outfile" -pass pass:"$pw"
  log "Decrypted $infile -> $outfile"
}
echo "Ending encrypting and decrypting files process!"

#######################
# Main CLI
#######################
show_help(){
cat <<EOF
Usage: $0 <command>
Commands:
  monitor_once           - search recent logs for keywords
  monitor_follow         - follow logs and print matching lines
  audit_logins           - show 'last' output
  check_failed           - parse failed login events
  auto_block             - auto-block IPs found above threshold
  add_allow_port <port>  - add iptables allow for a TCP port
  remove_allow_port <p>  - remove allow
  run_av                 - run antivirus scan (ClamAV if installed)
  encrypt <in> [out]     - encrypt file with openssl
  decrypt <in> [out]     - decrypt file with openssl
  cleanup_blocks         - cleanup expired blocks
EOF
}

case "$1" in
  monitor_once) monitor_logs_once ;;
  monitor_follow) monitor_logs_follow ;;
  audit_logins) audit_login_history ;;
  check_failed) check_failed_logins ;;
  auto_block) auto_block_from_authlog ;;
  add_allow_port) add_firewall_rule_allow_port "$2" ;;
  remove_allow_port) remove_firewall_rule_allow_port "$2" ;;
  run_av) run_antivirus_scan ;;
  encrypt) encrypt_file "$2" "$3" ;;
  decrypt) decrypt_file "$2" "$3" ;;
  cleanup_blocks) cleanup_expired_blocks ;;
  *) show_help ;;
esac
```


