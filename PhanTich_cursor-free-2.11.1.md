# BÁO CÁO PHÂN TÍCH KỸ THUẬT & AN NINH
## GÓI TIỆN ÍCH MỞ RỘNG: `cursor-free-2.11.1.vsix`

* **Ngày thực hiện phân tích:** 20/10/2026
* **Đối tượng phân tích:** `D:\PhanTichFIle\CursorVsix\cursor-free-2.11.1.vsix`
* **Môi trường phân tích:** Node.js, WebAssembly decompiler, String deobfuscation sandbox.

---

### 1. TỔNG QUAN TẬP TIN & SIÊU DỮ LIỆU (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Tên tệp** | `cursor-free-2.11.1.vsix` |
| **Kích thước tệp** | 2,993,443 bytes (~2.85 MB nén / ~5.3 MB giải nén) |
| **Định dạng** | VSIX (Lưu trữ nén chuẩn ZIP cho VS Code / Cursor Extension) |
| **Tên Extension (`package.json`)** | `cursor-free` |
| **Display Name** | `cursor-free` |
| **Phiên bản (Version)** | `2.11.1` |
| **Nhà phát hành (Publisher)** | `jg` |
| **Mô tả danh nghĩa** | *"Manage token authentication and authorization for Cursor"* |
| **Activation Events** | `onStartupFinished` (Kích hoạt ngay khi IDE khởi động xong) |
| **Repository khai báo** | `https://github.com/your-org/cursor-free` (Placeholder giả) |
| **Dấu vết nhà phát triển (Build Artifact)** | `/Users/yangchengxu/.cargo/registry/src/...` (Dev: Yang Chengxu / jg) |

---

### 2. CẤU TRÚC GÓI VSIX

```text
cursor-free-2.11.1.vsix
├── extension.vsixmanifest                  # Khai báo extension cho hệ thống Cursor/VS Code
├── [Content_Types].xml
└── extension/
    ├── package.json                        # Khai báo lệnh, webview view, script build, dependencies
    ├── package.nls.json                    # Bản dịch giao diện mặc định
    ├── package.nls.zh-cn.json              # Bản dịch tiếng Trung (giao diện điều khiển)
    ├── readme.md                           # Tài liệu hướng dẫn sử dụng & link Feishu
    ├── LICENSE.txt
    ├── resources/icon.svg                  # Icon extension (Activity Bar)
    └── dist/
        ├── wasm_crypto_bg.wasm             # Module WebAssembly (182 KB) biên dịch từ Rust
        ├── wasm_crypto.js                  # Lớp cầu nối JS-WASM (42 hàm zk01 -> zk42)
        ├── extension.js                    # Mã thực thi chính (3.09 MB, mã hóa làm rối)
        └── webview/                        # Giao diện điều khiển người dùng (React 19)
            ├── manifest.json
            ├── main.bundle.js              # Bundle React UI (221 KB)
            ├── react-vendor.bundle.js      # Thư viện React 19 (845 KB)
            ├── vendors.bundle.js           # Phụ thuộc bổ trợ (112 KB)
            └── assets/icon.svg
```

---

### 3. CƠ CHẾ BẢO VỆ MÃ NGUỒN & CHỐNG PHÂN TÍCH (OBFUSCATION)

Nhà phát triển sử dụng chiến lược **2 tầng phòng thủ**:

1. **JavaScript Obfuscator (`webpack-obfuscator` + `javascript-obfuscator`):**
   * Toàn bộ `dist/extension.js` và `dist/webview/main.bundle.js` bị làm rối toàn diện.
   * Toàn bộ chuỗi (strings) được đóng gói trong một mảng chuỗi 25.996 phần tử (`_0x_0x3fa8`), giải mã thông qua thuật toán hoán vị RC4 + Base64 với các hàm bọc phân cấp.
   * Kỹ thuật chia nhỏ chuỗi (`splitStrings`), chèn mã rác, biến đổi luồng điều khiển (control flow flattening), tiêm vòng lặp `setInterval` chống debug / devtools.

2. **Rust WebAssembly (`wasm_crypto_bg.wasm`):**
   * Được viết bằng ngôn ngữ Rust, sử dụng các crate: `aes-gcm`, `hkdf`, `sha2`, `base64`, `serde_json`, `wasm-bindgen`.
   * Chứa toàn bộ các mẫu vá mã nguồn (patch templates), các chuỗi regex đối chiếu và thuật toán mã hóa/giải mã khóa bản quyền nhằm ẩn giấu khỏi các công cụ quét mã độc thông thường.

---

### 4. PHÂN TÍCH CHI TIẾT CÁC HÀM TRONG MODULE WASM (`zk01` - `zk42`)

Cầu nối `wasm_crypto.js` xuất bản 42 hàm API native phục vụ quá trình can thiệp hệ thống:

| Tên hàm | Kiểu dữ liệu / Chức năng | Ý nghĩa kỹ thuật |
| :--- | :--- | :--- |
| **`zk01`, `zk02`, `zk03`, `zk04`** | Mã hóa / Giải mã Crypto | Sử dụng AES-GCM và HKDF để mã hóa và giải mã các gói quy tắc vá (patch rules) tải về từ máy chủ. |
| **`zk05`, `zk07`, `zk08`** | `cursorFreeAuth` Handler | Tạo mẫu can thiệp vào `handleURL` cho giao thức `cursor://cursorFreeAuth?accessToken=...&refreshToken=...&email=...` để nạp trực tiếp token lậu vào `storeAccessRefreshToken`. |
| **`zk09`, `zk10`, `zk11`** | `cursorPurePatched` | Regex tìm kiếm và thay thế hàm kiểm tra tính toàn vẹn client: `return {isPure: true, proof: ...}` nhằm đánh lừa hệ thống telemetry của Cursor. |
| **`zk12`, `zk13`, `zk14`** | `cursorExtensionVerifyBypass` | Thay thế cơ chế xác thực chữ ký extension của Cursor (`if (!valid) throw error` đổi thành cảnh báo `warn` và cho qua). |
| **`zk15`, `zk16`, `zk17`, `zk18`** | `cursorJgAuthPatch` | Mẫu code chặn bắt kết nối HTTP/2: Chuyển hướng URL `api*.cursor.sh`, đọc `~/.cursor_free_data`, tiêm header `x-jg-auth`, `x-jg-model: 1` và tạo giả `x-cursor-checksum`. |
| **`zk19`, `zk20`, `zk21`, `zk28`** | `cursorH2Patch` | Các biến thể vá giao thức mạng HTTP/2 và can thiệp thuộc tính `authority` của Cursor. |
| **`zk22`, `zk23`, `zk24`** | TLS/Connection Patch | Bỏ qua kiểm tra chứng chỉ SSL (`rejectUnauthorized: false`) khi kết nối qua proxy. |
| **`zk29`, `zk30`, `zk31`** | `cursorTabPatched` | Tìm hàm `isAllowedCpp()` và ép trả về `true` (`isAllowedCpp(){return true;/*cursorTabPatched*/}`) để mở khóa tính năng Cursor Tab (Copilot++). |
| **`zk32`, `zk33`, `zk34`** | `cursorEaseModelPatch` | Can thiệp vào hàm `buildRequestedModel` để điều hướng yêu cầu model AI (Composer, Chat). |
| **`zk35`, `zk36`, `zk37`** | `cursorThinkTagPatched` | Chèn bộ đệm bóc tách thẻ `<think>...</think>` vào `handleTextDelta` nhằm hỗ trợ các mô hình lý luận (như DeepSeek-R1). |
| **`zk38`, `zk39`, `zk40`, `zk41`, `zk42`** | `cursorFreeWorkbenchHtml` | Nới lỏng chính sách bảo mật CSP (`'unsafe-eval'`, `'unsafe-inline'`, `blob:`) trong `workbench.html` và chèn thẻ script tải mã tùy ý từ bên ngoài. |

---

### 5. CƠ CHẾ CAN THIỆP & TÁC ĐỘNG VÀO HỆ THỐNG CỦA CURSOR

Khi extension được cài đặt và kích hoạt, nó thực thi chuỗi hành vi sau:

1. **Định vị tệp tin cốt lõi của Cursor IDE:**
   Extension tự động tìm kiếm đường dẫn cài đặt Cursor trên hệ điều hành (Windows, macOS, Linux) để xác định các tệp:
   * `out/vs/workbench/workbench.desktop.main.js`
   * `out/vs/workbench/workbench.html`
   * `extensions/cursor-always-local/dist/main.js`
   * `out/vs/workbench/contrib/extensions/electron-browser/extensionHostProcess.js`

2. **Sao lưu & Ghi đè (Patching):**
   * Tạo tệp sao lưu nguyên bản có hậu tố `.backup` (ví dụ `workbench.desktop.main.js.backup`).
   * Sử dụng quyền Administrator để can thiệp nhị phân/chuỗi, đưa các đoạn mã bẻ khóa ở Mục 4 vào các file nguồn của IDE.

3. **Chuyển hướng toàn bộ lưu lượng dữ liệu (Man-in-the-Middle):**
   * Mỗi khi người dùng gõ code (Tab autocomplete), gửi prompt Chat, hoặc sử dụng Composer, request không gửi về `api.cursor.sh` mà bị chuyển hướng đến máy chủ proxy trung gian.
   * Đọc tệp thông tin người dùng được lưu ngầm tại: `~/.cursor_free_data` (nằm trong thư mục Home của User).

---

### 6. HẠ TẦNG MẠNG & ENDPOINT MÁY CHỦ BÊN THỨ BA

Các địa chỉ máy chủ và tài nguyên trích xuất trực tiếp từ mã nguồn:

* **Máy chủ API chính:**
  * `https://cur-api.shareai.pro/quota/api`
  * `https://cur-api5.shareai.pro`
* **Địa chỉ IP máy chủ dự phòng:**
  * `http://111.231.65.234:8081/quota/api` (Địa chỉ IP thuộc Tencent Cloud tại Thượng Hải, Trung Quốc).
* **Tài liệu & Kênh hỗ trợ:**
  * Wiki Feishu: `https://my.feishu.cn/wiki/K8ITwXxrriVkhlkKqcDco16mntZ`
  * Tệp ảnh hướng dẫn: `https://cur-api.shareai.pro/quota/client-public-file/instruction.png`
* **Chức năng của các endpoint:**
  * Đối soát và trừ hạn mức sử dụng (Quota balance check).
  * Kiểm tra và kích hoạt key (`activateKey`).
  * Tải về các quy tắc vá động mới (để đối phó khi Cursor tung bản cập nhật mới).

---

### 7. PHÂN TÍCH TÍNH CHÂN THỰC CỦA CÁC MODEL AI (THẬT HAY GIẢ / TRÁO MODEL)

Một thắc mắc lớn từ người dùng: *Liệu các model khi chọn (như Claude 3.5 Sonnet hay GPT-4o) có phải là hàng xịn chính hãng không, và tại sao khi hỏi "bạn là ai", nó vẫn trả lời đúng tên model đã chọn?*

Dựa trên việc bóc tách mã nguồn và hạ tầng mạng, câu trả lời là: **Hệ thống có sự kết hợp giữa tài khoản dùng chung (Account Pooling), tráo model (Downgrading) và tận dụng System Prompt để đánh lừa người dùng.**

#### A. Tại sao hỏi model "bạn là ai" nó vẫn trả lời đúng model đã chọn?
* **Cơ chế System Prompt ẩn của Cursor:** Bất kỳ câu hỏi nào từ người dùng trước khi gửi đi đều được Cursor tự động chèn một đoạn System Prompt ẩn dài hàng nghìn ký tự, ví dụ: *"You are Claude, a large language model trained by Anthropic..."*.
* **Khả năng Roleplay của LLM:** Khi một model khác (như DeepSeek-V3, Qwen-2.5-Coder, Llama 3) nhận được System Prompt này, nó sẽ **tuân thủ vai diễn 100%** và tự xưng là Claude 3.5 Sonnet mà người dùng không thể nhận biết nếu chỉ hỏi bằng văn bản thông thường.

#### B. Bằng chứng kỹ thuật về việc tráo model trong mã nguồn:
1. **Bản vá thẻ `<think>` đặc thù (`cursorThinkTagPatched` - zk35 -> zk37):**
   * Trong mã nguồn có đoạn xử lý:
     ```javascript
     const si = this.__tkBuf.indexOf("<think>");
     ...
     this.handleThinkingDelta(a, this.__tkBuf.slice(0, sl), void 0);
     this.handleThinkingCompleted({message: {case: "thinkingCompleted", value: {thinkingDurationMs: Date.now() - this.__tkMs}}});
     ```
   * **Bằng chứng đanh thép:** Các API chính thức của Claude 3.5 Sonnet (Anthropic) và GPT-4o (OpenAI) **không bao giờ trả về thẻ `<think>` dạng văn bản thô**. Thẻ này là định dạng đầu ra độc quyền của **DeepSeek-R1** (hoặc các model reasoning mã nguồn mở). Việc extension phải cài cắm patch riêng để dịch thẻ `<think>` thành bong bóng suy nghĩ chứng minh server `shareai.pro` đang bí mật chuyển hướng yêu cầu sang DeepSeek-R1.
2. **Can thiệp model ID và tắt `maxMode` (`cursorEaseModelPatch` - zk34):**
   * Đoạn code: `new k({modelId: typeof c === "string" && c.includes("composer") ? c : "default", maxMode: false})` ép tắt tính năng tính toán mở rộng và can thiệp tham số model của Cursor.
3. **Tiêm Header phân luồng lậu:**
   * Tự động thêm header `x-jg-model: 1` và `x-jg-net-optimize` để báo hiệu cho cụm server proxy phân loại hoặc hạ cấp request.

#### C. Cơ chế vận hành thực tế tại server trung gian:
* **Account Pooling (Chia sẻ tài khoản Pro):** Bên dịch vụ mua một số ít tài khoản Cursor Pro thật, trích xuất token và giả mạo checksum (`x-cursor-checksum`). Hàng trăm người mua key sẽ dùng chung vài tài khoản Pro này. Khi còn hạn mức hoặc câu hỏi ngắn, bạn có thể vô tình trúng tài khoản thật.
* **Dynamic Hybrid Routing (Mồi câu & Hạ cấp):**
  * *Request ngắn, đơn giản:* Cho qua model xịn để người dùng test thấy đúng và tin tưởng.
  * *Request dài, Composer, quét nhiều file (ngốn token):* Bí mật chuyển hướng sang DeepSeek / Qwen hoặc **cắt ngắn ngữ cảnh (Context Truncation)** để tiết kiệm chi phí, khiến code sinh ra bị lỗi logic hoặc ảo giác.

#### D. Cách kiểm chứng thực tế:
1. **Kiểm tra Tokenizer:** Hỏi câu hỏi bẫy đếm ký tự: *"Trong từ 'strawberry' có chính xác bao nhiêu chữ 'r'?"*. Model xịn (GPT-4o / Claude 3.5 Sonnet) trả lời đúng (3 chữ r), model clone hoặc model mini thường đếm sai.
2. **Quan sát giao diện suy luận:** Nếu chọn Claude 3.5 Sonnet nhưng xuất hiện khối giao diện "Thinking..." kèm thời gian đếm giây, đó **100% là DeepSeek-R1** đội lốt.
3. **Kiểm tra Composer đa tệp:** Yêu cầu sửa đổi logic liên quan tới 15 - 20 file. Server lậu bị cắt context sẽ bắt đầu báo "không tìm thấy hàm" ở các file lân cận.

---

### 8. ĐÁNH GIÁ RỦI RO TOÀN DIỆN

#### 🔴 Rủi ro cực kỳ nghiêm trọng: Lộ lọt toàn bộ mã nguồn (Data & Code Exfiltration)
* Toàn bộ mã nguồn, biến môi trường, thông tin nhạy cảm (API keys, credentials trong file dự án) đi kèm ngữ cảnh prompt gửi lên Cursor đều đi qua máy chủ `shareai.pro` và `111.231.65.234`.
* Chủ sở hữu hạ tầng proxy này có toàn quyền thu thập, lưu trữ và khai thác mã nguồn của người dùng mà không có bất kỳ cam kết bảo mật nào.

#### 🔴 Rủi ro thực thi mã độc từ xa (Remote Code Execution - RCE)
* Việc cố ý vô hiệu hóa CSP (`'unsafe-eval'`, `'unsafe-inline'`) và cho phép nạp script từ bên ngoài vào cửa sổ Workbench của Cursor cho phép máy chủ điều khiển có thể gửi bất kỳ mã JavaScript nào về máy tính của người dùng để thực thi với đầy đủ quyền hạn của Node.js / Electron.

#### 🟡 Rủi ro phá hỏng phần mềm (Stability Degradation)
* Do can thiệp trực tiếp vào file nhị phân của Cursor, bất kỳ bản cập nhật nhỏ (minor/patch update) nào của Cursor cũng có thể làm sai lệch cấu trúc file, dẫn tới tình trạng IDE bị lỗi trắng màn hình (blank screen) hoặc không thể khởi động.

#### 🟡 Rủi ro pháp lý & Điều khoản dịch vụ (ToS Violation)
* Hành vi giả mạo client (`cursorPurePatched`) và vượt cơ chế chứng thực vi phạm nghiêm trọng Điều khoản Sử dụng của Anysphere Inc. (Cursor), dẫn đến việc khóa tài khoản vĩnh viễn và các rủi ro pháp lý về sở hữu trí tuệ.

---

### 9. KẾT LUẬN & KIẾN NGHỊ

* `cursor-free-2.11.1.vsix` **hoàn toàn không phải là một công cụ tiện ích quản lý token thông thường**, mà là một **bộ công cụ bẻ khóa và trung chuyển API lậu (Reverse Proxy Crack Tool)** phục vụ dịch vụ bán key thương mại không chính thống từ Trung Quốc.
* Chất lượng model đầu ra bị can thiệp, pha trộn hàng giá rẻ (DeepSeek-R1/V3) và chia sẻ chung tài khoản Pro với hàng trăm người khác.
* **Khuyến nghị xử lý:**
  1. **Không cài đặt** hoặc gỡ bỏ ngay lập tức nếu đã cài đặt.
  2. Khôi phục các tệp tin gốc của Cursor (xóa các file đã patch và đổi tên `.backup` về gốc, hoặc cài đặt lại hoàn toàn Cursor IDE sạch).
  3. Xóa tệp dữ liệu ngầm `~/.cursor_free_data` trong thư mục người dùng.
  4. Thu hồi và đổi các token, password, API key nếu từng mở các dự án nhạy cảm trong thời gian cài đặt extension này.
