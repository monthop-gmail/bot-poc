# ประวัติและวิวัฒนาการของตระกูล xxx-bot-poc

> เริ่มเขียนเมื่อ 2026-10-06 · ปรับสถานะตามหลักฐานถึง 2026-10-16 · เล่าจากมุมมองของผู้ร่วมสร้าง (owner + ทีม agent ใน session ต่างๆ)
> เอกสารนี้ตอบคำถาม "ทำไมตระกูลนี้ถึงมีรูปแบบอย่างที่เห็นวันนี้" — รายละเอียดปฏิบัติการอยู่ที่
> README/CHECKLIST/handoff.md ของแต่ละโครงเสมอ · อัปเดตไฟล์นี้เมื่อเกิดเหตุการณ์สำคัญ

วันที่ของแต่ละระลอกใช้เวลาไทย (+07) จากกระทู้ FAST POC REVIEW;
เรื่อง SPH ที่กล่าวถึงภายหลังอิงกระทู้ Spare-Parts Hub ในพื้นที่งานภายใน

**อ่านชื่อให้ตรงกัน:** `Codex` เมื่อใช้เรียกผู้ลงมือช่วงแรกหมายถึง session/client ที่คนใช้สั่งงาน;
`Codex Bot` คือ node อัตโนมัติของ `codex-bot-poc` ที่รับ handoff และโพสต์ผลในชื่อตัวเอง
ส่วน `codex-ui-bot-poc` เป็นเว็บอีกโครง · ชื่อ `Codex` ที่เคยปรากฏเพราะ token ผูกชื่อนั้น
อย่างเดียวไม่พอจะบอกว่า session หรือ bot ตัวใดเป็นผู้ลงมือ

## 1. จุดกำเนิด — ทำไมตระกูลนี้ถึงเกิด

ทีมเราทำงานผ่าน **โต๊ะกลาง (ai-collaboration MCP)** ที่มี agent หลายตระกูล
(ChatGPT, Claude, Codex, Cursor, Grok, Gemini, ...) รับ-ส่งงานกันเป็น task/handoff มี review board
และกติกาการทำงานร่วมกัน

ปัญหาที่เกิด: agent ฝั่ง coding แต่ละตัว (ZCode, OpenCode, Codex, Claude Code, ...) ใช้งานผ่าน
terminal/IDE ซึ่ง **ผูกกับเครื่องของคนใช้** — ไม่มี "หน้าต่างกลาง" ที่ (1) อยู่บนเซิร์ฟเวอร์ของทีมตลอด 24
ชม. (2) เข้าได้จากที่ไหนก็ได้อย่างปลอดภัย (3) ต่อกับโต๊ะกลางและ GitHub ได้พร้อมกัน

owner จึงสั่งชุดทดลองชื่อ **xxx-bot-poc**: เริ่มจากรัน coding agent แต่ละตัวเป็น **เว็บบน Docker ของ VPS**
เข้าจากเบราว์เซอร์ผ่าน **Cloudflare Tunnel + Access** โดยใน container เชื่อม
ai-collab MCP + gh ให้พร้อมใช้ — และเป็น Fast POC: ทำน้อย → เก็บ evidence จริง → ให้ทีมรีวิว
(กระทู้ [FAST POC REVIEW] เป็นกระดาน evidence กลางของตระกูล)
ภายหลัง Claude Code และ UltraCode ทั้งเว็บและ node ทำงานบน VPS อีกเครื่อง; node อัตโนมัติ
เป็นอีกชั้นหนึ่งของตระกูล

## 2. รูปแบบร่วม (family pattern) ที่ค้นพบแล้วกลายเป็นมาตรฐาน

โครงแรก (zcode, herdr) ลองผิดลองถูก จนจับรูปแบบร่วมของ **เว็บ surface** ได้ว่า:

```
เบราว์เซอร์ → ชั้นยืนยันตัวตน → tunnel → เว็บของ agent บน VPS
                                        └─ เครื่องมือร่วมงานและ repository ที่จำเป็น
```

- **โครงพื้นฐานใช้ระบบอัตโนมัติ** และบทเรียนจากงานเดิมของทีม เพื่อลดขั้นตอนตั้งค่าซ้ำ
- **ข้อจำกัดในระยะแรก**: หลาย surface ยืม token ที่ผูกชื่อ "Codex" จึงโผล่เป็น Codex แม้ส่ง
  `X-Client-Name` ต่างกัน · ต่อมาทีมย้ายไป credential มาตรฐานและประกาศชื่อผ่าน header
  ได้แล้ว (ระลอก 8) แต่ชื่อที่ประกาศยังไม่ใช่การพิสูจน์ตัวตนแยกด้วย credential ราย bot
- **เอกสารประจำโครง**: README (สถานะ+จุดเรียนรู้) · CHECKLIST (evidence ทีละข้อ — ตัวหนา=บังคับ)
  · **handoff.md ใน folder ของงานเอง** (สืบทอดให้ session ถัดไป)

## 3. กติกาที่ owner กำหนด (และเหตุผล)

| กติกา | เหตุผล |
|---|---|
| ไม่ทำช่องทาง LINE เด็ดขาด | LINE ไม่เหมาะกับการ dev หลายจุด (botforge v1 เคยใช้ — ตระกูลนี้ไม่ทางนั้น) |
| build บน VPS ไม่ใช่ WSL ของ owner | build บน WSL ทำให้เครื่อง owner ช้า |
| ห้ามเก็บไฟล์งานใน /tmp | WSL shutdown แล้วหายหมด |
| repo GitHub private ชื่อ `<agent>-bot-poc` (main) | ตระกูลเดียว ระบบเดียว ค้นง่าย |
| token ย้ายผ่าน SSH stdin / ห้ามพิมพ์ค่าลง log | ความลับหลุดครั้งเดียวคือจบ |
| evidence ลงกระทู้ [FAST POC REVIEW] | ข้อความบนโต๊ะลบไม่ได้ — ห้ามโพสต์ความลับ (standing rule) |

## 4. วิวัฒนาการตามระลอก

### ระลอก 1 — ตั้งต้น (3–4 ต.ค.): zcode + herdr
- **zcode-bot-poc**: โครงแรก — พิสูจน์ทั้งสาย (build → tunnel/Access → WebSocket ผ่าน Access
  → login โมเดล GLM) ได้เว็บแรกของตระกูล
- **herdr-bot-poc**: อีกสาย — **fleet node**: โต๊ะกลางสั่ง → node แจกงานให้ fleet (boss=codex
  คุม oc1–oc3=opencode) → done/blocked + result_ref · รีวิวไล่จาก "not demonstrated" ขึ้น
  "PROMISING" ด้วย evidence แบบแยกชิ้น: build ผ่าน ≠ รันได้ ≠ วงจรครบ ≠ recovery ได้

### ระลอก 2 — มาตรฐานสมบูรณ์ (4 ต.ค.): opencode
- ตระกูลได้รูปแบบที่ "ถูกต้องที่สุด" ตั้งแต่ยังไม่ build: **opencode มีเว็บในตัว** (`opencode web`)
  จึงไม่ต้องเขียน bridge เลย · ได้ gotcha สำคัญ: `--port` default สุ่ม, ต้อง pin;
  `OPENCODE_DISABLE_AUTOUPDATE`; workspace ต้องเป็น git repo; ยิง MCP ต้องมี header
  `Accept: application/json, text/event-stream` (ไม่งั้น 406)
- **ไฮไลต์**: ใช้โมเดลฟรี (big-pickle) พิสูจน์ "โมเดลเรียก tool โต๊ะกลางได้จริง" เป็นตัวแรกของตระกูล —
  ปิด gate ที่ zcode ยังติด 429 · Cloudflare สร้างผ่าน API ทั้งหมดด้วย script ที่ยืมมา

### ระลอก 3 — ปัญหา "ไม่มีเว็บในตัว" และสองคำตอบ (4–5 ต.ค. เวลาไทย, session คู่ขนาน)
- codex และ claude-code **ไม่มีโหมดเว็บในตัว** — แก้ด้วยสองเทคนิค:
  - **codex-ui-bot-poc**: ยก **codex-gateway** (เว็บสำเร็จรูป MIT ของชุมชน) มาแปะ — สรุปว่า
    "อย่าเขียนเองถ้ามีของดีรออยู่"
  - **claudecode-bot-poc**: เขียน bridge เองด้วย **Claude Agent SDK** + web chat UI — deploy จริง
    โดยมีขั้นตอนเตรียมสิทธิ์ใช้งานโมเดลแยกต่างหาก
- งานวิจัยร่วม: **botforge v1** (9 engines) คือบทเรียนรุ่นเก่า — spawn CLI / app-server — บางส่วน
  ล้าสมัยแล้ว · **botforge v2 (botforge-multi-agent)** คือทิศ fleet: orchestrator/worker + A2A

### ระลอก 4 — meta-layer (5 ต.ค. เวลาไทย): openhands
- **openhands-bot-poc**: deploy **Agent Canvas** — "control center" ที่โฮสต์ได้หลาย agent ผ่าน
  **ACP** + automations (webhook/schedule) + public mode (API key เป็นชั้นสองหลัง Access)
  · image 5.57GB ใหญ่สุดตระกูล · MCP transport ผ่าน (17 tools)

### ระลอก 5 — สืบสายพันธุ์และวางแผนรุ่นถัดไป (6 ต.ค.)
- **บทเรียนใหญ่: ของ upstream เปลี่ยนเร็วกว่าบทเรียนเรา** — qwen-code (fork ของ gemini-cli)
  ปล่อย **`qwen serve` + Web Shell UI ในตัว** ทำให้ botforge-technique ล้าสมัยทันทีสำหรับตัวมัน
  (ขณะที่ codex/claude ยังไม่มีเว็บในตัวเลย) → กติกาใหม่: **ทุก handoff ต้องมีวันที่ตรวจ upstream**
- ค้น "สมุดทะเบียน" จากสาย fleet: **ACP catalog (~40 agents)** + botforge engines + A2A —
  ได้ candidate ชุดใหม่: Cursor CLI, Junie, Kiro, Factory Droid, Mistral Vibe, Docker cagent,
  Kimi CLI, Goose, ...
- **วางกลยุทธ์สองชั้น**: agent ที่พูด ACP เข้าบ้าน Canvas ได้เลย (ไม่ต้องทำเว็บใหม่) —
  ส่วน native-web agent ทำ standalone แบบเดิม
- โครง/เอกสารใหม่ที่วางไว้: **qwencode-bot-poc** (native serve+Web Shell — ง่ายสุด) ·
  **canvas-acp-agents-poc** (โฮสต์หลาย agent บน Canvas — คุ้มสุด) · **gemini-bot-poc**
  (ปิดสาย Google) · **canvasnode-bot-poc** (skeleton จริง: ยก Canvas เป็น node ระดับ 1
  ของสาย fleet — state machine จาก herdr + Canvas API)
- **qwencode-bot-poc deploy จริง (6 ต.ค. — ตัวแรกที่ใช้ `qwen serve`)**: ยืนยันว่า "ง่ายสุด
  รองจาก opencode" จริง แต่บทเรียนข้อ 2 (ตรวจ upstream) พิสูจน์ซ้ำอีกครั้ง — **Qwen OAuth
  ฟรีถูกยุบ 2026-04-15** (handoff ยังเชื่อว่าฟรี ~2000 req/วัน — ตรวจ issue ต้นทางก่อนทำ
  เลยรอด: เปลี่ยนแผนเป็น "deploy ก่อน รอ key ทีหลัง") · gotcha ใหม่ของตระกูล: reverse proxy
  เปลี่ยนข้อมูล origin ที่บริการปลายทางมองเห็น จึงต้องตรวจการตั้งค่าเว็บและ healthcheck ให้ตรงกับสภาพจริง
- **สืบ hermes บน VPS อีกเครื่อง (6 ต.ค.)**: ทีมเคยทดลอง hermes (NousResearch — 251k stars, MIT,
  self-improving agent) ไว้ลึกมากแล้ว — test-hermes (OKMD provider 23 models, tool calling
  ผ่าน 20/23, บทเรียน "โควต้าใช้ร่วมทั้งตระกูล provider") · nst-hermes (จาก
  server-botforge-v2) คือ **สายคุยผ่าน LINE โมเดลเบา — ไม่ใช่สาย dev** (owner ยืนยัน;
  ยืมได้เฉพาะฝั่งกายภาพ dashboard/PUID/layout) — ในช่วงที่บันทึก container หยุดอยู่
- **ค้นพบเปลี่ยนกลยุทธ์ fleet**: herdr integration รองรับ kind ถึง 18 ตัว (pi, omp, claude,
  codex, copilot, devin, droid, kimi, opencode, kilo, **hermes**, qodercli, qwen, cursor,
  mastracode, antigravity-cli, grok, letta — ตรวจจาก image จริง) — fleet "codex boss +
  worker หลายสายพันธุ์" ไม่ต้องเขียน adapter เอง · **hermes-bot-poc** จึงถูกวางเป็นสององก์:
  พิสูจน์ hermes สาย dev ก่อน (dashboard เว็บ + probe tool calling) แล้ว §8 ยกระดับเป็น
  worker ใต้ boss ของ herdr (ขึ้นกับกรอบ quota ที่ owner กำหนด เพราะ herdr ถูกพักอยู่ในช่วงนั้น)

### ระลอก 6 — big-4 ครบ (6 ต.ค.): gemini-bot-poc
- **gemini-bot-poc deploy จริง (6 ต.ค.) — ปิดช่องว่างสาย Google**: ตอนนี้ big-4 ผู้ให้บริการ
  มีตัวในตระกูลครบ (OpenAI=codex, Anthropic=claudecode, Z.AI=zcode, **Google=gemini**)
- โครง: bridge ยกจาก botforge template `bot-service-gemini-cli` (spawn `gemini -p …
  --output-format json --approval-mode yolo` ต่อ prompt) + หน้าแชทเว็บแบบ claudecode —
  Gemini CLI ไม่มีเว็บในตัว (a2a-server ไม่ใช่เรื่องเว็บ — เก็บไว้ทาง fleet/BOTFORGE V2;
  ACP อยู่ที่ canvas-acp-agents-poc)
- **ยืนยันกับ bundle จริง 0.62.0** (เปิด dist อ่านเอง): JSON output มี `session_id` →
  resume ข้าม turn ได้ · headless trust ผ่าน env `GEMINI_CLI_TRUST_WORKSPACE=true` ·
  ปิด auto-update ด้วย settings `general.enableAutoUpdate` · MCP ผูกที่
  `~/.gemini/settings.json` ด้วยคีย์ **`httpUrl`** (ไม่ใช่ `url` แบบ Claude) + `trust: true`
- **บทเรียนใหม่ 2 ข้อ (ทดสอบ spawn กับ CLI จริง)**: (1) โหมด yolo พิมพ์ banner คร่อม
  output และกรณี error ทั้งก้อนไปลง **stderr** (stdout ว่าง) — parser ต้องหา JSON object
  จาก stdout แล้ว fallback stderr แบบนับ brace; (2) **stdin ของ child ต้องเป็น open pipe
  — ปิด (/dev/null) แล้ว CLI ค้างเงียบ** (spawn ของ botforge driver ถูกอยู่แล้ว ห้าม
  `stdin.end()`)
- deploy จริง: เว็บผ่าน tunnel และ Access · image 870MB · MCP transport 200 · gh ผ่าน · error path ผ่านโมเดลจริง ·
  ในช่วงนั้นยังต้องเตรียมสิทธิ์ใช้งานโมเดล — แนว "deploy ก่อน รอ key" แบบเดียวกับ qwencode/claudecode
- บันทึกกระบวนการ: ถาม owner เรื่อง auth + ยืนยันชื่อ DNS/tunnel ผ่านช่องทาง session
  แล้วไม่มีคำตอบ (สั่ง "ดำเนินการต่อ") → ใช้ชื่อตามที่ handoff เสนอเอง (ย้อนกลับได้ทาง API)
  และทำ stack รองรับวิธีเข้าใช้งานโมเดลที่กำลังพิจารณา ให้เลือกทีหลังได้
- **รอบรีวิวข้ามตัวครั้งแรก (Grok review bot, seq 30 — PASS_WITH_GATES)**: รีวิวพี่น้อง #6–#8 พร้อมกัน
  แล้วออก gates C1–C5 — ปิดแล้ว: C1 (MCP **read-only allowlist** บนเว็บ surface
  Hermes/Gemini/Qwen ในรีวิวรอบนี้ — Hermes นำมาใช้ก่อน แล้ว Gemini+Qwen ตาม) · C4 ของ qwencode
  (settings generate จาก env ทุก start + non-root uid 1000 + pin cloudflared) · บทเรียน
  ใหม่: บริการต้องรันด้วยสิทธิ์เท่าที่จำเป็น และตั้งค่า auth ให้ครบทุก endpoint;
  การโหลดค่า config ต้องไม่เขียนความลับลงไฟล์คงค้าง — ครบทุก finding
  ของรอบนี้; เหลือ B3/B4 ระดับตระกูล (รอ handoffs ครบ) · บั๊กซ่อนระดับตระกูลที่ถูกจับ:
  `RUN git config --global` ก่อน `USER node` เขียนลง /root/.gitconfig — claudecode/codex
  น่าจะมีเหมือนกัน (รอทีม surface นั้นตรวจ)

### ระลอก 7 — รีวิวข้ามตัวปิด security sweep + เอกสารเล่าเรื่อง (6 ต.ค.)
- **C1–C4 ปิดครบด้วยการส่งไม้ต่อข้าม session** (seq 31–34): Gemini/Qwen/Hermes เว็บ
  surface ที่อยู่ในรีวิวรอบนั้นจำกัดเครื่องมือ MCP เป็น **read-only allowlist** · ข้อนี้ไม่ครอบ
  node อัตโนมัติที่ต้องเขียน task/handoff ตามหน้าที่ · ชื่อ "Codex" กลางยังเป็นข้อจำกัดของ
  surface ที่ใช้ token ผูกชื่อนั้นในเวลานั้น (แก้ทิศทางในระลอก 8)
- **เอกสารเล่าเรื่องชุดใหม่**: `talk-then-sleep.md` (ตอบคำโซเชียล "คุยกับ AI แล้วไปนอน"
  ด้วยของจริง — subagent คือกล้ามเนื้อ กองกำลังคือทีม) · `grok-bot-story.md` (นักตรวจ
  อิสระแห่งโต๊ะกลางใน session review bot; mentor bot ที่ขับงาน SPH เป็นอีก session)
  · แผนตัวตนราย bot ฉบับเก่าถูกเก็บไว้เป็นบันทึกภายใน

### ระลอก 8 — identity พลิกทิศ: credential ร่วม + ชื่อประกาศเอง, per-bot tokens กลายเป็นข้อยกเว้น (9–12 ต.ค. เวลาไทย)
- **owner เปิดเผยบริบท**: ในช่วงทดลองมี credential ร่วมหลายชุดจากเหตุผลทางประวัติศาสตร์
  ทิศทางใหม่คือใช้ชุดมาตรฐานร่วมกับ header ประกาศชื่อ;
  token เฉพาะตัวใช้เมื่อมี security boundary ที่ต้องแยกจริง
- **สัญญา 3 ชั้นเคาะแล้ว** (seq 51–52): token = access credential · X-Client-Name =
  declared client attribution (ประกาศเอง ไม่ใช่ crypto identity) · acting_context =
  บริบทการทำงานแทนใคร · fallback = `unidentified-static-client`
- **hub ทำงานตาม**: `acting_context` ขึ้น production (c0726f0 — 3 tools: update_task/
  accept_handoff/resolve_decision) · `unresolved_attribution` จับผู้เรียกไม่ประกาศชื่อได้
  จริงตั้งแต่ call แรก (7ebe35f) · `STATIC_CLIENT_NAME` เปลี่ยนเป็น
  `unidentified-static-client` แล้ว (seq 55) · `capability` บน handoff ขึ้นแล้ว (9652922,
  migration 0007) + ตัวกรองแบบ bounded-delimiter (b02d44c)
- **canary ผ่านครบสาม client**: Codex (seq 57 — ชื่อ client กลับมาเป็น Codex ด้วย header บน credential มาตรฐาน; ไม่ใช่ Codex Bot) ·
  ZCode (seq 63 — บทเรียน case-sensitivity: `zcode`≠`ZCode` ทำ delegated flag ผิด →
  hub แก้เป็น `same_party_different_case`) · OpenCode (seq 66)
- **ผลลัพธ์สำคัญ**: จับได้ว่าผู้เรียกที่ไม่ประกาศชื่อเคยปิด decision ระดับ project
  — ร่องรอยที่เครื่องมือเดิมมองไม่เห็น

### ระลอก 9 — nodes อัตโนมัติมีชื่อ live ครบ + ultracode (12–15 ต.ค. เวลาไทย)
- **node อัตโนมัติ 3 ตัว live ในช่วงที่บันทึก** ต่างสายพันธุ์: **OpenCode Bot** (branch
  autonomy-pilot, routines daily-summary โพสต์กระทู้ของตัวเองหลังแก้ P1 safe_body) ·
  **Codex Bot** (node แยก auth ใหม่ — owner login device-auth ครั้งแรก ไม่ copy ของเดิม) ·
  **Gemini Bot** (Antigravity CLI!) · ทั้งสามมี preflight `you_are` ตรงชื่อและรับ handoff เฉพาะ
  ชื่อตัวเอง · OpenCode Bot/Codex Bot มีผล forced-fail และ restart-mid-run สดในกระทู้รีวิว #72/#76/#85;
  Gemini มี live handoff canary และ unit tests สำหรับ recovery (#99/#101) แต่โพสต์เหล่านั้น
  ยังไม่ใช่หลักฐานว่า Gemini ผ่าน restart-mid-run แบบสด
- **gemini ย้ายไป Antigravity CLI 1.3.5** + OAuth (Gemini 3.8 Flash) — คำทำนาย "จับตา
  Antigravity" ใน story นี้มาเป็นจริงใน 4 วัน · รูปแบบการยืนยันตัวตนหน้าเว็บปรับตามการตัดสินใจของ owner
- **ultracode-bot-poc** (15 ต.ค.): Claude Agent SDK bridge + Ultracode plugin (pipeline
  explore→spec→plan→implement→review) — แยกไปอยู่ VPS อีกเครื่องคู่ claudecode · บทเรียนจาก botforge: ultracode เป็น plugin ไม่ใช่
  harness ที่สาม
- claudecode: PR #1 merged (worker task status preserve ตอน error/restart) · zcode:
  ปรับการเลือกบัญชีโมเดลและทดสอบกับบัญชีที่จัดไว้สำหรับช่วงทดลอง

## 5. แผนที่จากกระทู้ถึง 15 ต.ค. (ภาพประวัติ)

ตระกูลนี้มีทั้งเว็บให้มนุษย์คุยกับ agent, node รับงานอัตโนมัติ และงานทดลองสาย fleet
หลักฐานในกระทู้ ณ 15 ต.ค. แสดงการ deploy และ canary หลายแบบ แต่การรัน container
เพียงอย่างเดียวไม่พิสูจน์ว่าโมเดลหรือ handoff ผ่านทุกเส้นทาง

| สาย | ตัวอย่างที่มีในเรื่อง |
|---|---|
| เว็บให้มนุษย์ใช้ | ZCode, OpenCode, Codex UI, Claude Code, OpenHands, Qwen Code, Gemini, UltraCode, Hermes |
| node รับงานอัตโนมัติ | OpenCode Bot, Codex Bot, Gemini Bot, Claude Code Bot, UltraCode Bot |
| งานทดลอง fleet | Herdr, Canvas ACP agents, Canvas node |

Herdr ซึ่งเป็นต้นแบบ fleet ถูกพักตามคำสั่ง owner ในช่วงที่บันทึก
ส่วนแนวคิด state machine ถูกนำไปใช้ต่อในงาน Canvas node

(โครงสาย fleet ที่เกี่ยวแต่ไม่ได้ชื่อ *-bot-poc: botforge-multi-agent — orchestrator/worker + A2A)

## 6. บทเรียนที่ต้องไม่ลืม (สำหรับคนที่มาต่อ)

1. **พิสูจน์เป็นชิ้นๆ**: build ผ่าน ≠ รันได้ ≠ ต่อ hub ครบวงจร ≠ มี UI — อย่าโปรโมท claim โดยไม่มี
   evidence ที่รันได้จริง (done และ forced-fail ต้องมีทั้งคู่; สถานะต้อง enforce ในโค้ด
   และตรวจจาก hub ด้วย เพราะเคยเกิด wrapper ทับ worker `blocked` เป็น `done` ในงาน SPH #62–66)
2. **ตรวจ upstream ก่อนเชื่อบทเรียนเก่า** — อุตสาหกรรมเดินเร็ว: qwen เพิ่งมี serve+Web Shell,
   botforge v1 ล้าสมัยบางส่วนในหนึ่งเดือน; ทุก handoff จึงมี "วันที่ตรวจ" กำกับ
3. **อย่าเขียน bridge/UI ถ้ามีของสำเร็จรูป** (codex-gateway, opencode web, qwen Web Shell,
   Canvas ACP) — แรงงานหลักคือ credential/config/verification ไม่ใช่โค้ด
4. **ความลับมีทางเดินตายตัว**: เก็บในระบบจัดการความลับที่ทีมกำหนด · ห้ามลง git · ห้ามลงโต๊ะกลาง
5. **ชื่อบนโต๊ะกลางเป็น attribution**: ระยะแรก token ที่ผูกชื่อ "Codex" ทับชื่อจาก header;
   ระลอก 8 เปลี่ยนค่าเริ่มต้นเป็น credential มาตรฐาน + `X-Client-Name` · การแยก credential จริงราย bot
   ยังเป็นทางเลือกเมื่อมีขอบเขตสิทธิ์หรือความเสี่ยงที่ต้องแยก
6. **ประสานก่อนแตะ stack คนอื่น** — หลายโครงถูกสร้างพร้อมกันจากหลาย session; commit แยก,
   rollback ได้, คุยกันก่อนเสมอ

## 7. คำถามเปิดของตระกูล (รอ owner ตัดสินเมื่อ evidence ครบ)

- **ตัวตนจริงบนโต๊ะกลาง — อัปเดตสถานะ (11 ต.ค.)**: ทิศเดิม "per-bot tokens สำหรับทุกตัว"
  **ถูกแทนด้วยทิศใหม่ของ owner**: ใช้ credential มาตรฐานร่วมกับชื่อที่แต่ละ bot ประกาศ
  เป็นค่าเริ่มต้น; token เฉพาะตัวใช้เมื่อจำเป็นต้องแยกขอบเขตสิทธิ์
  การแยกชื่อที่ประกาศไม่เท่ากับการพิสูจน์ตัวตนด้วย credential ราย bot
- **การลู่เข้าหากัน**: herdr (hub-driven fleet) vs Agent Canvas (control center มนุษย์) vs
  botforge v2 (A2A fleet) — สามสถาปัตยกรรมซ้อนทับกันบางส่วน จะรวมเป็นอะไร?
  (canvasnode-bot-poc คือการทดลองแรกที่ต่อสองโลกเข้าด้วยกัน)
- **local-model variant**: Ollama/vLLM บน VPS สำหรับสาย private/no-quota — ทำเมื่อไร?
- **Antigravity CLI** (Google) — Gemini bot ย้ายมาใช้และพิสูจน์ OAuth/model แล้วในระลอก 9;
  การใช้กับ bot ตัวอื่นต้องประเมินตามหลักฐานของตัวนั้น

---
*ตระกูลนี้ยังเดินต่อ — ถ้าคุณอ่านถึงบรรทัดนี้ในฐานะคนใหม่: เริ่มจาก handoff.md ของโครงที่สนใจ,
อ่าน CHECKLIST เพื่อเข้าใจ evidence ที่มี, และอย่าลืมว่ากติกาข้อ 3 คือข้อตกลงของ owner ไม่ใช่ข้อเสนอ*
