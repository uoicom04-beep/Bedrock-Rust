# Build APK for Android (มือถือ)

มี 2 ทาง: **GitHub Actions (แนะนำ)** และ **build บนเครื่อง**

---

## ทางที่ 1 — GitHub Actions (ไม่ต้องติดตั้ง NDK เอง)

1. สร้าง repo บน GitHub แล้ว push โค้ดทั้งโฟลเดอร์ `bedrock-rs`
2. ไปที่แท็บ **Actions** → workflow **Build Android APK**
3. กด **Run workflow** (หรือ push ขึ้น `main`)
4. รอ job เสร็จ → เปิด run นั้น → **Artifacts** → ดาวน์โหลด `bedrock-rs-android`
5. ใน artifact จะมี:
   - `.apk` (ถ้า cargo-apk / xbuild สำเร็จ)
   - หรือ `libbedrock_android.so` สำหรับเอาไปใส่ใน Android Studio เอง

ไฟล์ workflow: `.github/workflows/android.yml`

---

## ทางที่ 2 — Build บนเครื่อง

### ความต้องการ
- Rust stable
- Android SDK + NDK 26+
- `cargo install cargo-ndk`
- (ถ้าจะได้ APK สำเร็จรูป) `cargo install cargo-apk` หรือ `xbuild`

```bash
rustup target add aarch64-linux-android armv7-linux-androideabi

# สร้าง .so
cargo ndk -t arm64-v8a -o ./jniLibs build -p bedrock-client --lib --release

# หรือลอง cargo-apk
cd crates/client && cargo apk build --lib
```

APK อยู่ที่ประมาณ:
`target/release/apk/` หรือตามที่ cargo-apk รายงาน

---

## ติดตั้งลงมือถือ

1. เปิด **Developer options** + **USB debugging** (หรือส่งไฟล์ APK เข้าเครื่อง)
2. อนุญาตติดตั้งจากแหล่งที่ไม่รู้จัก (Install unknown apps)
3. ติดตั้ง APK แล้วเปิด **Bedrock-RS**

---

## หมายเหตุสำคัญ

- ตอนนี้ native lib เป็น **entry + log** ยังไม่เต็ม renderer บน Android
- พอ CI ผ่านและได้ `.so` / `.apk` แล้ว ขั้นถัดไปคือเปิด `android-activity` + `winit` android + wgpu ใน `crates/android`
- ปุ่มสัมผัส / virtual joystick ยังไม่ได้ใส่ — จะเพิ่มหลัง APK รันได้บนเครื่องจริง

---

## Package id
`dev.bedrockrs.client`  
ชื่อบนหน้าจอ: **Bedrock-RS**
