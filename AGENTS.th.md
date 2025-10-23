# แนวทางสำหรับรีโพนี้

โปรดดูไฟล์ `requirements.txt` และ `requirements-dev.txt`

หากมีไฟล์ CLAUDE.md และ CLAUDE.local.md ให้ศึกษาเพิ่มเติมด้วย

## โครงสร้างโปรเจ็กต์และการจัดระเบียบโมดูล
Zen MCP Server มีแกนหลักที่ `server.py` ซึ่งเปิด entrypoint ของ MCP และประสานเวิร์กโฟลว์หลายโมเดล
- เครื่องมือสำหรับฟีเจอร์เฉพาะอยู่ใน `tools/`
- การเชื่อมต่อผู้ให้บริการโมเดลอยู่ใน `providers/`
- ฟังก์ชันช่วยเหลือที่ใช้ร่วมกันอยู่ใน `utils/`
- แฟ้ม prompt และ system context อยู่ใน `systemprompts/`
- แม่แบบคอนฟิกและสคริปต์อัตโนมัติอยู่ใน `conf/`, `scripts/`, และ `docker/`
- ชุด unit test อยู่ใน `tests/`
- สถานการณ์จำลองและยูทิลิตีสำหรับล็อกอยู่ใน `simulator_tests/` ซึ่งใช้ฮาร์เนส `communication_simulator_test.py`
- เอกสารอ้างอิงและตัวอย่างอยู่ใน `docs/`
- ข้อมูลการวินิจฉัยขณะรันงานจะบันทึกหมุนเวียนใน `logs/`

## คำสั่งสำหรับ build, test และการพัฒนา
- `source .zen_venv/bin/activate` – เปิดใช้สภาพแวดล้อม Python ที่จัดการไว้
- `./run-server.sh` – ติดตั้ง dependencies รีเฟรช `.env` และเปิด MCP server ในเครื่อง
- `./code_quality_checks.sh` – รัน Ruff (autofix), Black, isort และ pytest ชุดปกติ
- `python communication_simulator_test.py --quick` – ทดสอบการประสานงานข้ามเครื่องมือและผู้ให้บริการแบบรวดเร็ว
- `./run_integration_tests.sh [--with-simulator]` – ทดสอบเวิร์กโฟลว์ที่พึ่งพาผู้ให้บริการและโมเดลระยะไกลหรือ Ollama

รันเช็กคุณภาพโค้ด:
```bash
.zen_venv/bin/activate && ./code_quality_checks.sh
```

ตัวอย่างการรันเทสต์ทีละไฟล์หรือทั้งหมด:
```bash
.zen_venv/bin/activate && pytest tests/test_auto_mode_model_listing.py -q
.zen_venv/bin/activate && pytest -q
```

## สไตล์โค้ดและการตั้งชื่อ
- ใช้ Python 3.9+ พร้อม Black และ isort (จำกัด 120 อักขระต่อบรรทัด)
- Ruff บังคับใช้ pycodestyle, pyflakes, bugbear, comprehension และ pyupgrade
- ให้ใช้ type hint ชัดเจน, ตั้งชื่อโมดูลแบบ snake_case และเขียน docstring แบบกริยาเชิงคำสั่งเมื่อคอมมิต
- หากต้องขยายเวิร์กโฟลว์ให้สร้าง hook หรือเมท็อด abstract แทนการเช็ก `hasattr()`/`getattr()` เพื่อให้พฤติกรรมค้นพบและทดสอบได้ผ่านการสืบทอด

## แนวทางการทดสอบ
- สร้างโครงทดสอบให้สะท้อนโมดูลโปรดักชันภายใต้ `tests/`
- ตั้งชื่อเทสต์เป็น `test_<behavior>` หรือคลาส `Test<Feature>`
- รัน `python -m pytest tests/ -v -m "not integration"` ก่อนคอมมิตทุกครั้ง และเพิ่ม `--cov=. --cov-report=html` หากต้องการดู coverage
- ใช้ `python communication_simulator_test.py --verbose` หรือ `--individual <case>` เพื่อตรวจเวิร์กโฟลว์ข้ามเอเจนต์
- สำรอง `./run_integration_tests.sh` สำหรับการแก้ไขที่กระทบผู้ให้บริการหรือการขนส่งข้อมูล
- หากมีเทสต์ล้มเหลวให้เก็บส่วนที่เกี่ยวข้องจาก `logs/mcp_server.log` หรือ `logs/mcp_activity.log` ประกอบเอกสาร

## แนวทางคอมมิตและ Pull Request
- ใช้มาตรฐาน Conventional Commits: `type(scope): summary` โดย `type` อยู่ในชุด `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, หรือ `chore`
- ทำคอมมิตให้โฟกัสและอ้างอิง issue หรือเคสจำลองเมื่อจำเป็น
- PR ควรอธิบายเจตนา ระบุคำสั่งที่ใช้ยืนยันผลลัพธ์ แจ้งการปรับคอนฟิกหรือเครื่องมือ และแนบสกรีนช็อตหรือส่วนล็อกเมื่อมีผลต่อผู้ใช้

## คำสั่ง GitHub CLI ที่ใช้บ่อย
GitHub CLI (`gh`) ช่วยจัดการ issue และ PR จากเทอร์มินัลได้สะดวก

### ดูรายละเอียด issue
```bash
gh issue view <issue-number>
gh issue view <issue-number> --repo owner/repo-name
gh issue view <issue-number> --comments
gh issue view <issue-number> --json title,body,author,state,labels,comments
gh issue view <issue-number> --web
```

### จัดการ issue
```bash
gh issue list
gh issue list --label bug --state open
gh issue create --title "Issue title" --body "Description"
gh issue close <issue-number>
gh issue reopen <issue-number>
```

### จัดการ Pull Request
```bash
gh pr view <pr-number>
gh pr list
gh pr create --title "PR title" --body "Description"
gh pr checkout <pr-number>
gh pr merge <pr-number>
```

ติดตั้ง GitHub CLI ได้ด้วย `brew install gh` (macOS) หรือดู https://cli.github.com สำหรับแพลตฟอร์มอื่น

## เคล็ดลับด้านความปลอดภัยและคอนฟิก
- เก็บ API key และ URL ของผู้ให้บริการไว้ใน `.env` หรือคอนฟิกของ MCP client ห้ามคอมมิตความลับหรือไฟล์ล็อกที่สร้างขึ้น
- ใช้ `run-server.sh` เพื่อสร้างสภาพแวดล้อมใหม่และตรวจสอบการเชื่อมต่อหลังเปลี่ยน dependency
- เมื่อต้องเพิ่มผู้ให้บริการหรือเครื่องมือ ให้ทำความสะอาด prompt และคำตอบ ระบุ environment variable ที่จำเป็นในเอกสาร `docs/` และอัปเดต `claude_config_example.json` หากต้องเปิดความสามารถใหม่เป็นค่าเริ่มต้น
