# Bedrock-RS

**Minecraft Bedrock–style engine skeleton written in pure Rust.**

เป้าหมาย: client ที่ลื่น, โหลด resource pack แบบ Bedrock จริง, รองรับ native addon ภาษา Rust,  
dedicated server, และโครง APK สำหรับ Android — ให้ใกล้เคียง Bedrock มากที่สุดในขอบเขตที่ทำได้แบบเปิด

> ไม่ใช่ Minecraft ทางการ และไม่ได้มี protocol / Xbox Live ครบในเวอร์ชันนี้  
> ใช้สำหรับเรียนรู้, วิจัย, และต่อยอดเป็น client/server ของตัวเอง

---

## สิ่งที่มีใน repo นี้

| ส่วน | สถานะ |
|------|--------|
| Cargo workspace | ครบ |
| Core (block, chunk, world, NBT stub) | ครบ |
| Resource Pack loader (manifest.json + PNG) | ครบ |
| Texture extractor จากเกมจริง | สคริปต์พร้อม |
| Protocol skeleton (packet + RakNet stub) | ครบ |
| Client (winit window) | ครบ |
| Dedicated server (UDP 19132) | ครบ |
| Native Rust Addon API (cdylib) | ครบ |
| Android crate skeleton | ครบ |
| GitHub Actions CI | ครบ |

---

## ความต้องการ

- Rust stable (1.75+)
- Python 3.10+ (สำหรับสคริปต์ดึง texture)
- (ถ้าจะทำ APK) Android NDK + `cargo-apk` หรือ `xbuild`

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
rustup default stable
```

---

## เริ่มต้นเร็ว

```bash
# clone / แตก zip
cd bedrock-rs

# build ทั้งหมด
cargo build --workspace

# รัน client (เปิดหน้าต่าง)
cargo run -p bedrock-client

# รัน server (listen UDP 19132)
cargo run -p bedrock-server
```

---

## ดึง Texture จากเกมจริง

1. หาโฟลเดอร์ Minecraft Bedrock ที่ติดตั้ง (Windows ตัวอย่าง):
   ```
   %LOCALAPPDATA%\Packages\Microsoft.MinecraftUWP_8wekyb3d8bbwe\LocalState\games\com.mojang
   ```
   หรือ Android backup / extracted APK assets

2. รันสคริปต์:
   ```bash
   python tools/extract_textures.py --src "/path/to/com.mojang" --out ./extracted_packs
   ```

3. รัน client พร้อม texture เหล่านั้น:
   ```bash
   export BEDROCK_TEXTURES=./extracted_packs
   cargo run -p bedrock-client
   ```

Resource pack ต้องมี `manifest.json` แบบ Bedrock (format_version 2) ถึงจะโหลดได้

---

## โครงสร้างโปรเจกต์

```
bedrock-rs/
├── Cargo.toml                 # workspace
├── crates/
│   ├── core/                  # block, world, resource pack, nbt
│   ├── protocol/              # packets + RakNet stub
│   ├── client/                # window + future renderer
│   ├── server/                # dedicated server
│   ├── addon/                 # native Rust addon (cdylib)
│   └── android/               # APK entry skeleton
├── assets/resource_packs/     # ตัวอย่าง pack
├── tools/
│   ├── extract_textures.py
│   └── pack_addon.py
└── .github/workflows/ci.yml
```

---

## Native Addon (Rust)

```bash
cargo build -p bedrock-addon --release
# ได้ target/release/libbedrock_addon.so (หรือ .dll / .dylib)

python tools/pack_addon.py --lib target/release/libbedrock_addon.so --out my_addon.mcaddon
```

API ที่ expose ตอนนี้:
- `bedrock_addon_init()`
- `bedrock_addon_tick(dt)`
- `bedrock_addon_on_block_break(x,y,z,id) -> i32`
- `bedrock_addon_name() -> *const u8`

ขยายได้อิสระตามต้องการ

---

## Android APK (โครง)

1. ติดตั้ง target:
   ```bash
   rustup target add aarch64-linux-android
   cargo install cargo-apk
   ```
2. เติม `android-activity` + winit android ใน `crates/android`
3. สร้าง `Cargo.toml` ที่ root สำหรับ apk ตามเอกสาร cargo-apk
4. `cd crates/client && cargo apk build --lib`

(รายละเอียด NDK / keystore อยู่ในเอกสาร Android มาตรฐาน)

---

## เล่นเซิร์ฟ

Server ฟังที่ **UDP 19132** (พอร์ตมาตรฐาน Bedrock)

```bash
cargo run -p bedrock-server
```

ตอนนี้ยังเป็น stub (รับ packet แล้ว log)  
ขั้นถัดไป: ใส่ RakNet offline message → connection request → Login → StartGame

Client ภายนอก (เช่น official Bedrock) ยังต่อไม่ได้จนกว่า protocol จะครบ

---

## แผนต่อ (แนะนำลำดับ)

1. **Renderer** — wgpu: chunk mesh, texture atlas, camera
2. **Input + movement** — keyboard/mouse หรือ touch
3. **RakNet จริง** — offline pings, open connection, reliability
4. **Login / StartGame / LevelChunk** packets
5. **Addon loader** — โหลด .so จาก behavior pack folder
6. **APK** — android-activity + touch controls
7. **Xbox / Microsoft auth** — เฉพาะถ้าจำเป็น (ซับซ้อนและเปลี่ยนบ่อย)

---

## ใบอนุญาต / กฎหมาย

- โค้ดใน repo นี้เป็น MIT
- Texture และ asset จากเกมทางการเป็นของ Microsoft / Mojang  
  → ใช้ได้เฉพาะจากเครื่องผู้เล่นที่ซื้อเกมแล้ว (เหมือนแนวทางของ Horizon กับ Java)
- ห้ามแจก texture ที่ดึงมาพร้อม zip

---

## สร้างโดย

Bedrock-RS skeleton — สำหรับ OPERATOR ที่อยากได้ foundation เต็มรูปแบบใน Rust  
พร้อมต่อยอดเป็น client / server / APK ของตัวเอง

```bash
cargo run -p bedrock-client
```

---

## UI เหมือนเกมจริง (ดึงจากเครื่องมึง)

Bedrock-RS **ไม่แจก** ไฟล์ UI/texture ของ Microsoft  
แต่มีระบบโหลด JSON UI + texture แบบเดียวกับเกมจริง

### ขั้นตอนให้ UI เหมือนเป๊ะ

1. ติดตั้ง Minecraft Bedrock บนเครื่อง (Windows / มือถือ backup)

2. รันสคริปต์ดึงของจริง:
   ```bash
   python tools/extract_ui_and_textures.py --src "C:/Users/YOU/AppData/Local/Packages/Microsoft.MinecraftUWP_8wekyb3d8bbwe/LocalState/games/com.mojang" --out ./extracted_bedrock
   ```

3. ตั้ง env แล้วรัน client:
   ```bash
   export BEDROCK_UI=./extracted_bedrock/ui
   export BEDROCK_TEXTURES=./extracted_bedrock/resource_packs
   cargo run -p bedrock-client
   ```

4. ใน client กด `1` = title screen, `2` = hotbar (debug)

พอมึงดึงไฟล์จากเกมตัวเองสำเร็จ ระบบจะอ่าน JSON UI + texture ของจริง  
→ หน้าตาจะเหมือนเกม เพราะใช้ไฟล์ชุดเดียวกัน

### สิ่งที่ยังต้องทำต่อในโค้ด (renderer)

- วาด control จาก JSON ด้วย wgpu
- รองรับ anchor / size แบบ `%` และ px
- ปุ่ม, hover, click, touch
- font แบบ Bedrock

ระบบโหลดพร้อมแล้ว เหลือชั้นวาดภาพ


---

## UI Renderer (wgpu) — รอบนี้

Client วาดหน้าจอจริงแล้ว:

| ปุ่ม | หน้าจอ |
|------|--------|
| `1` | Title screen (พื้นหลัง + โลโก้ + ปุ่ม) |
| `2` | Hotbar + XP bar (สไตล์ HUD) |
| `3` | Inventory panel |

ตอนนี้เป็น layout สีที่เลียนแบบสัดส่วน Bedrock  
พอมึง extract UI/texture จากเกมจริง ขั้นถัดไปคือผูก texture เข้า quad (ระบบโหลดพร้อมแล้ว)

```bash
cargo run -p bedrock-client
# กด 1 / 2 / 3 สลับหน้าจอ
```


---

## Android / APK (มือถือ)

ดูคู่มือเต็ม: [ANDROID.md](ANDROID.md)

**ทางเร็ว (GitHub):**
1. Push ขึ้น GitHub
2. Actions → **Build Android APK** → Run workflow
3. ดาวน์โหลด artifact `bedrock-rs-android`

Package: `dev.bedrockrs.client`

---

## ควบคุมครบ (อัปเดต)

### PC
| อย่าง | วิธี |
|------|------|
| Title Play / Settings | คลิกปุ่มบนจอ |
| เดิน | WASD หรือลากเมาส์ซีกซ้าย (stick) |
| มอง | เมาส์ (จับเคอร์เซอร์) หรือลากซีกขวา |
| Hotbar | คลิกช่อง / 1-9 / scroll |
| กระโดด | Space หรือปุ่มเขียว |
| ใช้ของ | คลิกขวา หรือปุ่มน้ำเงิน |
| กระเป๋า | E หรือปุ่มมุมขวาบน |
| กลับ | Esc |

### มือถือ / Touch
- ซีกซ้ายจอ = virtual joystick เดิน
- ซีกขวา = ลากมอง
- ปุ่มเขียว = กระโดด
- ปุ่มแดง = โจมตี
- ปุ่มน้ำเงิน = ใช้ของ
- ปุ่มมุมบน = เปิดกระเป๋า
- แตะช่อง hotbar = เลือกของ


---

## รอบสุดท้าย — สิ่งที่มีครบ

### Title (หน้าแรก)
- ปุ่ม **Play / Realms / Marketplace / Settings / Quit / Profile** ตำแหน่งแบบ Bedrock
- คลิก Play เข้าโลก

### สกิน
```bash
mkdir -p skins
# ใส่ไฟล์ PNG 64x64 (หรือ 64x32) ชื่ออะไรก็ได้
cp my_skin.png skins/
export BEDROCK_SKINS=./skins
cargo run -p bedrock-client
```
- มี Steve placeholder ในตัว
- Profile ปุ่มบน title = เตรียมสลับสกิน

### มือถือ controls
- D-pad ซ้าย, Attack/Jump/Interact/Shield/Inv ขวา ตามภาพ Bedrock touch

### ดึง texture จริง
```bash
python tools/extract_ui_and_textures.py --src "PATH/com.mojang" --out ./extracted_bedrock
export BEDROCK_UI=./extracted_bedrock/ui
export BEDROCK_TEXTURES=./extracted_bedrock/resource_packs
```

### GitHub APK
Actions → **Build Android APK** → artifact

### ขอบเขตที่ซื่อสัตย์
- ไม่แจก texture/UI ของ Mojang — ดึงจากเครื่องผู้เล่นเอง
- โลก 3D mesh อยู่ใน memory ยังไม่ upload ครบทุกเฟรม (ขั้นถัดไปของ renderer)
- Realms / Marketplace / Xbox = stub (ต้อง service ของ Microsoft)

