# คู่มือติดตั้ง WSL (Windows Subsystem for Linux)

เอกสารนี้อธิบายขั้นตอนการตั้งค่า Zen MCP Server บน Windows โดยใช้ WSL

## ข้อกำหนดเบื้องต้นสำหรับ WSL

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-venv python3-pip curl git
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt install -y nodejs
npm install -g @anthropic-ai/claude-code
```

## ขั้นตอนติดตั้งเฉพาะ WSL

1. โคลนรีโพภายใน WSL (ไม่ควรอยู่ใต้ `/mnt/c`):
   ```bash
   cd ~
   git clone https://github.com/BeehiveInnovations/zen-mcp-server.git
   cd zen-mcp-server
   ```

2. รันสคริปต์ตั้งค่า:
   ```bash
   chmod +x run-server.sh
   ./run-server.sh
   ```

3. ตรวจว่า Claude Code เห็น MCP server:
   ```bash
   claude mcp list   # ควรเห็น 'zen'
   ```

## การแก้ปัญหาใน WSL

### ปัญหา virtualenv/Python
```bash
sudo apt install -y python3.12-venv python3.12-dev
python3 -m pip install --upgrade pip
```

### ปัญหาเรื่อง path
- ใช้ path ของ WSL (`/home/<ชื่อคุณ>/zen-mcp-server/`)
- สคริปต์ตั้งค่าจะตรวจจับ WSL และใช้ path ที่ถูกต้องให้อัตโนมัติ

### Claude Code ต่อไม่ติด
```bash
cat ~/.claude.json | grep -A 10 "zen"
# ต้องชี้ไปที่ /home/.../.zen_venv/bin/python
```

### เคล็ดลับประสิทธิภาพ

เก็บ repo ไว้ใน filesystem ของ WSL (เช่น `~/zen-mcp-server`) ไม่ควรอยู่ในไดรฟ์ Windows (`/mnt/c/...`)