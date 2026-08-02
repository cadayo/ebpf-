# ebpf-
# singbox备份
cat << 'EOF' > sb
#!/data/data/com.termux/files/usr/bin/bash

# ==========================================
# 请修改为你实际存放 sing-box 的绝对路径！
SING_BOX_DIR="/data/data/com.termux/files/home/ebpf" 
# ==========================================

# 1. 核心：如果当前不是 root (id != 0)，自动调用 su 重新执行当前脚本
if [ "$(id -u)" -ne 0 ]; then
  exec su -c "$0 $@"
fi

cd "$SING_BOX_DIR" || { echo "[-] 错误：找不到目录 $SING_BOX_DIR"; exit 1; }
SING_BOX_PATH="./sing-box"
CONFIG_PATH="config.json"

case "$1" in
  start)
    if pgrep -x "sing-box" > /dev/null; then
      echo "[!] sing-box 已经在运行中！"
      exit 1
    fi
    echo "[*] 正在验证配置..."
    "$SING_BOX_PATH" check -c "$CONFIG_PATH" || { echo "[-] 配置有误，启动中止！"; exit 1; }
    
    echo "[*] 正在后台启动 sing-box..."
    nohup "$SING_BOX_PATH" run -c "$CONFIG_PATH" > /dev/null 2>&1 &
    echo "[+] 启动成功！(PID: $!)"
    ;;
    
  stop)
    if ! pgrep -x "sing-box" > /dev/null; then
      echo "[!] sing-box 当前未运行。"
      exit 0
    fi
    echo "[*] 正在停止 sing-box 并清理内核资源..."
    pkill -15 -x "sing-box"
    sleep 1
    if pgrep -x "sing-box" > /dev/null; then
      pkill -9 -x "sing-box"
    fi
    echo "[+] 已完全停止。"
    ;;
    
  restart)
    sb stop
    sleep 1
    sb start
    ;;
    
  status)
    if pgrep -x "sing-box" > /dev/null; then
      echo "[+] sing-box 正在运行中 (PID: $(pgrep -x "sing-box"))"
    else
      echo "[-] sing-box 未运行。"
    fi
    ;;
    
  *)
    echo "用法: sb {start|stop|restart|status}"
    exit 1
    ;;
esac
EOF



chmod +x sb
mv sb /data/data/com.termux/files/usr/bin/sb
