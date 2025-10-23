# คู่มือติดตั้ง Zen MCP Server ด้วย Docker

เอกสารนี้ครอบคลุมการดีพลอย Zen MCP Server บนสภาพแวดล้อมโปรดักชันด้วย Docker และ Docker Compose

## เริ่มแบบรวดเร็ว

1. โคลนรีโพ:
   ```bash
   git clone https://github.com/BeehiveInnovations/zen-mcp-server.git
   cd zen-mcp-server
   ```
2. ตั้งค่าไฟล์ `.env`:
   ```bash
   cp .env.example .env
   # เติม API key ของคุณ
   ```
3. ดีพลอยด้วย Docker Compose:
   ```bash
   # Linux/macOS
   ./docker/scripts/deploy.sh

   # Windows PowerShell
   .\docker\scripts\deploy.ps1
   ```

## การตั้งค่าตัวแปรสภาพแวดล้อม

### API key ที่จำเป็น (ต้องมีอย่างน้อยหนึ่งตัว)
```env
GEMINI_API_KEY=your_gemini_api_key_here
OPENAI_API_KEY=your_openai_api_key_here
XAI_API_KEY=your_xai_api_key_here
OPENROUTER_API_KEY=your_openrouter_api_key_here
DIAL_API_KEY=your_dial_api_key_here
DIAL_API_HOST=your_dial_host
```

### ตัวเลือกเพิ่มเติม
```env
DEFAULT_MODEL=auto
LOG_LEVEL=INFO
LOG_MAX_SIZE=10MB
LOG_BACKUP_COUNT=5
DEFAULT_THINKING_MODE_THINKDEEP=high
DISABLED_TOOLS=
MAX_MCP_OUTPUT_TOKENS=
TZ=UTC
```

## สคริปต์ดีพลอย

### Linux/macOS
```bash
./docker/scripts/deploy.sh
```
คุณสมบัติ: ตรวจสอบสภาพแวดล้อม, เช็กสุขภาพแบบ backoff, จัดการล็อกอัตโนมัติ, แสดงสถานะบริการ

### Windows PowerShell
```powershell
.\docker\scripts\deploy.ps1
.\docker\scripts\deploy.ps1 -SkipHealthCheck
.\docker\scripts\deploy.ps1 -HealthCheckTimeout 120
```

## สถาปัตยกรรม Docker

- Dockerfile แบบ multi-stage: build dependencies ในขั้นแรก แล้วคัดไฟล์จำเป็นไปยัง runtime image ที่เล็กกว่า
- ความปลอดภัย: รันด้วยผู้ใช้ `zenuser` (ไม่ใช่ root), filesystem แบบ read-only, ปิดการยกระดับสิทธิ์, tmpfs ปลอดภัย
- การจัดการทรัพยากร: จำกัดหน่วยความจำเริ่มต้นที่ 512M (แก้ไขได้ใน docker-compose)

## โครงสร้างไฟล์สำคัญ

```
docker/
 ├─ Dockerfile
 ├─ docker-compose.yml
 ├─ scripts/
 │   ├─ deploy.sh
 │   └─ deploy.ps1
 └─ configs/
     └─ ...
```

## โวลุมและการแมปไดเรกทอรี

- `zen-mcp-config`: เก็บคอนฟิกและพลักอิน (แมปไปยัง `/app/config`)
- `logs/`: แมปออกมาเพื่อดูและเก็บรักษา
- `conf/custom_models.json`: คัดลอกไฟล์โมเดลคัสตอมก่อน build เพื่อให้ Docker เห็น

ตัวอย่างจาก `docker-compose.yml`:
```yaml
services:
  zen-mcp:
    volumes:
      - zen-mcp-config:/app/config
      - ./logs:/app/logs
      - ./conf/custom_models.json:/app/conf/custom_models.json:ro
```

## การตั้งค่าตัวแปรกับ Docker Compose

กุญแจสำคัญใน `docker-compose.yml`:
```yaml
environment:
  DEFAULT_MODEL: ${DEFAULT_MODEL:-auto}
  GEMINI_API_KEY: ${GEMINI_API_KEY}
  OPENAI_API_KEY: ${OPENAI_API_KEY}
  ...
```

ไฟล์ `.env` อยู่ระดับเดียวกับ compose ช่วยให้จัดการง่าย ค่าที่ไม่ตั้งจะ fallback ตามที่กำหนดใน compose

## การรวมกับลูกค้าตระกูล MCP

### Option 1: Docker run (โปรไฟล์พื้นฐาน)
```json
{
  "mcpServers": {
    "zen": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-v", "/absolute/path/to/zen-mcp-server:/app",
        "--env-file", "/absolute/path/to/zen-mcp-server/.env",
        "zen-mcp-server:latest"
      ]
    }
  }
}
```

### Option 2: Docker Compose run
```json
{
  "mcpServers": {
    "zen-mcp": {
      "command": "docker-compose",
      "args": [
        "-f", "/absolute/path/to/zen-mcp-server/docker-compose.yml",
        "run", "--rm", "zen-mcp"
      ]
    }
  }
}
```

### Option 3: ระบุ environment inline
```json
{
  "mcpServers": {
    "zen-mcp": {
      "command": "docker",
      "args": [
        "run", "--rm", "-i",
        "-e", "GEMINI_API_KEY=your_key_here",
        "-e", "LOG_LEVEL=INFO",
        "-e", "DEFAULT_MODEL=auto",
        "-v", "/path/to/logs:/app/logs",
        "zen-mcp-server:latest"
      ]
    }
  }
}
```

**หมายเหตุ:**
- เปลี่ยน `/absolute/path/...` เป็น path จริง
- ใช้เครื่องหมาย `/` กับ volume เสมอ แม้บน Windows
- ตรวจว่ามี `.env` และ API key ครบ
- Option 2 ใช้ volume `zen-mcp-config` สำหรับเก็บคอนฟิกถาวร

## การปรับแต่งขั้นสูง

### เครือข่ายแบบกำหนดเอง
```yaml
networks:
  zen-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
          gateway: 172.20.0.1
```

### รันหลายอินสแตนซ์
```bash
cp docker-compose.yml docker-compose.dev.yml
# แก้ชื่อ service และ port
docker-compose -f docker-compose.dev.yml up -d
```

## การอัปเดตและการย้ายข้อมูล

```bash
git pull origin main
docker-compose down
docker-compose build --no-cache
./docker/scripts/deploy.sh
```

คอนฟิกจะถูกเก็บในโวลุม `zen-mcp-config` ต่อให้รีบิลด์ใหม่

ตรวจ [CHANGELOG](../CHANGELOG.md) หากมี breaking change ก่อนอัปเกรดใหญ่

## การแก้ปัญหา

- `docker ps` ดูสถานะ container
- `docker logs zen-mcp` หรือ `tail -f logs/mcp_server.log` เพื่อตรวจข้อผิดพลาด
- ตรวจ permission ของโฟลเดอร์ `logs/`
- หาก health check ล้มเหลว ใช้ `./docker/scripts/deploy.sh --skip-health-check`

## ขั้นตอนถัดไป

- อ่าน [Configuration Guide](configuration.md) สำหรับตัวแปรเพิ่ม
- ดู [Advanced Usage](advanced-usage.md) เพื่อปรับโมเดล
- ศึกษา [Troubleshooting](troubleshooting.md) เมื่อมีปัญหา