# Báo Cáo Phân Tích Kỹ Thuật Chi Tiết: Garena.exe

* **Mục tiêu phân tích**: `C:\Program Files (x86)\Garena\Garena\Garena.exe`
* **Ngày phân tích**: 20/10/2026
* **Trạng thái**: Hoàn thành phân tích tĩnh (Static Analysis & PE Forensics)

---

## 1. Thông Tin Định Danh Tệp (File Identification & Hashes)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Tên tệp gốc (Original Filename)** | `Garena.exe` |
| **Đường dẫn đầy đủ** | `C:\Program Files (x86)\Garena\Garena\Garena.exe` |
| **Dung lượng tệp** | 457,600 bytes (khoảng 446.88 KB) |
| **Định dạng** | PE32 executable for MS Windows (GUI) Intel 80386 (32-bit) |
| **Mô tả (File Description)** | `Garena Platform` |
| **Tên sản phẩm (Product Name)**| `Garena Platform` |
| **Công ty phát hành** | `Garena Online` |
| **Bản quyền pháp lý** | `Copyright (C) 2010-2019 Garena Online. All Rights Reserved.` |
| **Phiên bản tệp (File Version)**| `2019.09.26.1823` |
| **Phiên bản sản phẩm (Product Version)** | `2.0.1909.2618` |
| **Mã băm MD5** | `2D190AACF6356488DA7F34B4C2747B11` |
| **Mã băm SHA-256** | `B9C49B85E8851F0EB5B00F94CA71600FE4A059DAA0AC49CBB0738BC2C9D4A226` |

---

## 2. Xác Thực Chữ Ký Số (Authenticode Digital Signature)

Tệp thực thi được ký số chính chủ bởi Garena và chữ ký vẫn ở trạng thái toàn vẹn (Valid Signature):

* **Trạng thái chữ ký**: `Valid` (`Signature verified`)
* **Chủ thể ký (Signer Certificate Subject)**:
  * `CN = Garena Online Pte Ltd`
  * `O = Garena Online Pte Ltd`
  * `L = Singapore`
  * `S = Singapore`
  * `C = SG`
* **Nhà cung cấp chứng chỉ (Issuer)**:
  * `CN = VeriSign Class 3 Code Signing 2010 CA`
  * `OU = VeriSign Trust Network`
  * `O = VeriSign, Inc.`
* **Số serial chứng chỉ**: `46A0800C34C5FEE85DB1758D556C2D0E`
* **Thumbprint (SHA1)**: `2BB95F164C168BFE64340802000B54E3F3460DD7`
* **Thời gian hiệu lực chứng chỉ**: `31/01/2019` đến `02/02/2020`
* **Máy chủ đóng dấu thời gian (Timestamp Authority)**:
  * `CN = Symantec Time Stamping Services Signer - G4`
  * `Issuer: Symantec Time Stamping Services CA - G2`
  * `Thumbprint: 65439929B67973EB192D6FF243E6767ADF0834E4`

---

## 3. Môi Trường Biên Dịch & CI/CD (Build & Toolchain Environment)

Dữ liệu debug và chuỗi runtime cho thấy chi tiết quy trình phát triển và build nội bộ của Garena:

* **Trình biên dịch**: Microsoft Visual C++ 2013 (MSVC v120).
* **Môi trường tự động hóa (CI/CD)**: Jenkins CI Server.
* **Đường dẫn PDB nội bộ (Debug Information)**:
  ```text
  D:\Gas\Jenkins\workspace\gas_client_master\output\Release\Garena.pdb
  ```
  *(Dự án mang tên mã **GAS** - Garena Application Services / Gas Client, nhánh `gas_client_master`).*
* **Thời điểm đóng gói biên dịch (TimeDateStamp)**: `26/09/2019 17:23:54 UTC` (Unix timestamp: `1569493434`).
* **Các thư viện mã nguồn mở/bên thứ ba tích hợp tĩnh**:
  * **Boost C++ Libraries**: Đặc biệt là `boost::filesystem` (xử lý đường dẫn) và `boost::unique_lock` (đồng bộ luồng / mutex).
  * **Dinkumware Standard C++ Library**: `Copyright (c) 1992-2004 by P.J. Plauger, licensed by Dinkumware, Ltd.`

---

## 4. Cấu Trúc PE Header & Cơ Chế Giảm Thiểu Khai Thác (Mitigations)

### 4.1. Thông số PE Header
* **Machine Type**: `0x014C` (i386 / 32-bit x86 architecture).
* **Image Base**: `0x00400000`
* **Address of Entry Point (RVA)**: `0x0002842D`
* **Subsystem**: `0x0002` (Windows GUI Subsystem).
* **Số lượng Section**: 6 sections.

### 4.2. Bảng Phân Vùng (PE Sections & Entropy)
| Tên Phân Vùng | Virtual Address | Virtual Size | Raw Size | Entropy | Đặc tính (Characteristics) | Ý nghĩa kỹ thuật |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `.text` | `0x00001000` | `0x00047BA5` | `0x00047C00` | **6.63** | `0x60000020` (Code, Execute, Read) | Mã máy thực thi, không bị nén/mã hóa (unpacked). |
| `.rdata` | `0x00049000` | `0x0000F58A` | `0x0000F600` | **4.55** | `0x40000040` (Initialized Data, Read) | Bảng IAT, chuỗi hằng, cấu trúc RTTI C++. |
| `.data` | `0x00059000` | `0x00005448` | `0x00003000` | **4.65** | `0xC0000040` (Initialized Data, Read, Write) | Dữ liệu biến toàn cục có thể thay đổi. |
| `.tls` | `0x0005F000` | `0x00000002` | `0x00000200` | **0.00** | `0xC0000040` (Initialized Data, Read, Write) | Thread Local Storage callbacks/dữ liệu luồng. |
| `.rsrc` | `0x00060000` | `0x0000AD18` | `0x0000AE00` | **5.66** | `0x40000040` (Initialized Data, Read) | Chứa Icon, Manifest XML, Version Information. |
| `.reloc` | `0x0006B000` | `0x000041C4` | `0x00004200` | **6.59** | `0x42000040` (Discardable, Read) | Bảng dịch chuyển địa chỉ (Relocation Table). |

*Đánh giá đóng gói (Packer check)*: Độ ngẫu nhiên (Entropy) tối đa của `.text` là `6.63` (ngưỡng file bị pack/mã hóa thường > `7.2`). Tệp không sử dụng bất kỳ packer/protector thương mại nào (như Themida, VMProtect, UPX) mà là binary biên dịch native tiêu chuẩn.

### 4.3. Các cờ bảo mật bộ nhớ (Exploit Mitigations)
* **ASLR (Address Space Layout Randomization)**: **BẬT** (`IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE` = `0x0040`). Địa chỉ nạp vào bộ nhớ sẽ được ngẫu nhiên hóa bởi Windows kernel.
* **DEP / NX (Data Execution Prevention)**: **BẬT** (`IMAGE_DLLCHARACTERISTICS_NX_COMPAT` = `0x0100`). Ngăn chặn việc thực thi shellcode trên Stack hoặc Heap.
* **Terminal Server Aware**: **BẬT** (`IMAGE_DLLCHARACTERISTICS_TERMINAL_SERVER_AWARE` = `0x8000`).

---

## 5. Bảng Nhập Hàm (Import Address Table - IAT)

`Garena.exe` nhập hàm rất chọn lọc từ đúng **4 DLL hệ thống**, chứng minh cấu trúc siêu tối giản của một launcher:

1. **`KERNEL32.dll` (92 hàm)**:
   * **Quản lý tiến trình & luồng**: `CreateProcessW`, `CreateThread`, `ResumeThread`, `TerminateProcess`, `ExitProcess`, `ExitThread`.
   * **Quản lý thư viện động**: `LoadLibraryW`, `LoadLibraryExW`, `GetProcAddress`.
   * **Đồng bộ hóa (Synchronization)**: `CreateEventA`, `OpenEventA`, `SetEvent`, `ResetEvent`, `ReleaseSemaphore`, `WaitForSingleObjectEx`.
   * **Bộ nhớ & Heap**: `HeapAlloc`, `HeapFree`, `HeapReAlloc`, `LocalAlloc`, `LocalFree`, `TlsAlloc`, `TlsGetValue`, `TlsSetValue`.
   * **Tập tin & Thư mục**: `FindFirstFileW`, `FindNextFileW`, `FindClose`, `CreateFileW`, `WriteFile`, `SetCurrentDirectoryW`, `GetModuleFileNameW`.
2. **`ADVAPI32.dll` (4 hàm)**:
   * `CreateWellKnownSid`
   * `GetNamedSecurityInfoW`
   * `SetEntriesInAclW`
   * `SetNamedSecurityInfoW`
   * *(Mục đích: Thiết lập và sửa đổi quyền bảo mật ACL trên file/folder để tránh lỗi phân quyền trên Windows).*
3. **`ole32.dll` (1 hàm)**:
   * `CoInitializeEx` (Khởi tạo môi trường COM cho luồng giao tiếp IPC).
4. **`SHLWAPI.dll` (2 hàm)**:
   * `SHGetValueW`
   * `SHSetValueW`
   * *(Mục đích: Đọc và ghi trực tiếp giá trị vào Windows Registry).*

---

## 6. Manifest Nhúng (Embedded Application Manifest)

Manifest quy định quyền tài khoản người dùng và tương thích hệ điều hành:

```xml
<assembly xmlns="urn:schemas-microsoft-com:asm.v1" manifestVersion="1.0">
  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
    <security>
      <requestedPrivileges>
        <requestedExecutionLevel level="asInvoker" uiAccess="false"></requestedExecutionLevel>
      </requestedPrivileges>
    </security>
  </trustInfo>
  <compatibility xmlns="urn:schemas-microsoft-com:compatibility.v1">
    <application>
      <!-- Windows Vista -->
      <supportedOS Id="{e2011457-1546-43c5-a5fe-008deee3d3f0}"></supportedOS>
      <!-- Windows 7 -->
      <supportedOS Id="{35138b9a-5d96-4fbd-8e2d-a2440225f93a}"></supportedOS>
      <!-- Windows 8 -->
      <supportedOS Id="{4a2f28e3-53b9-4441-ba9c-d69d4a4a6e38}"></supportedOS>
      <!-- Windows 8.1 -->
      <supportedOS Id="{1f676c76-80e1-4239-95bb-83d0f6d0da78}"></supportedOS>
      <!-- Windows 10 -->
      <supportedOS Id="{8e0f7a12-bfb3-4fe8-b9a5-48fd50a15a9a}"></supportedOS>
    </application>
  </compatibility>
  <dependency>
    <dependentAssembly>
      <assemblyIdentity type="win32" name="2.0.1909.2618" version="2.0.1909.2618" language="*">
      </assemblyIdentity>
    </dependentAssembly>
  </dependency>
</assembly>
```

* **Mức đặc quyền (Privilege Level)**: `asInvoker`. Ứng dụng khởi động với quyền user thông thường, không làm phiền người dùng bằng hộp thoại UAC nhắc cấp quyền Admin khi vừa mở.
* **Phụ thuộc lắp ráp (Assembly Dependency)**: Khai báo phụ thuộc trực tiếp vào phiên bản `2.0.1909.2618`.

---

## 7. Kiến Trúc Vận Hành & Khám Phá Nội Tại (Deep Behavioral Findings)

### 7.1. Mô hình Bootstrap Launcher & Dispatcher
Khác với kiến trúc cũ gộp toàn bộ vào một file thực thi duy nhất, `Garena.exe` hoạt động theo mô hình **Dispatcher (Bộ điều hướng)**:

```
[Người dùng click Garena.exe]
           │
           ▼
Kiểm tra Mutex Single Instance
("Local\{C15730E2-145C-4c5e-B005-3BC753F42475}-once-flag")
           │
           ▼
Đọc Registry: HKCU/HKLM "Software\Garena\gxx\"
           │
           ▼
Tìm thư mục phiên bản hợp lệ gần nhất (VD: "2.0.1909.2618")
           │
           ▼
Kiểm tra & chuẩn hóa quyền truy cập tệp (SetEntriesInAclW)
           │
           ▼
LoadLibraryW(".../2.0.1909.2618/gxx.dll")
           │
           ▼
GetProcAddress("gxxmain" hoặc "gxxlauncher")
           │
           ▼
Khởi chạy UI Qt5 + Chromium CEF (gxxcef.exe) + AppShield Engine
```

### 7.2. Các cơ chế nội tại then chốt phát hiện qua chuỗi mã:
1. **Mutex chống mở nhiều tiến trình (Single Instance Guard)**:
   * Đối tượng Mutex được đặt tên: `Local\{C15730E2-145C-4c5e-B005-3BC753F42475}-once-flag`.
   * Khi khởi động, nếu Mutex này đã tồn tại, tiến trình mới sẽ kích hoạt instance cũ nổi lên giao diện rồi tự kết thúc.
2. **Quản lý phiên bản động (Version Directory Scaffolding)**:
   * Chuỗi log nội bộ:
     * `"success reading from registry, %s %s"`
     * `"fail read reg, %d %d"`
     * `"directory found with version installed %s %s"`
     * `"no valid directory found in %s"`
   * Quá trình cập nhật do `gxxupdate.exe` tải về phiên bản mới vào thư mục con (ví dụ `2.0.1909.2618`). Khi mở `Garena.exe`, nó quét thư mục phiên bản mới nhất trên ổ đĩa và cập nhật giá trị vào khóa Registry `Software\Garena\gxx\`.
3. **Cơ chế thiết lập quyền bảo mật tự động (ACL Auto-Repair)**:
   * Chuỗi lỗi tương ứng: `"ensure file permissions failed for %s"`, `"SetEntriesInAcl error %d"`, `"setNamedSecurityInfo error %d"`.
   * Garena chủ động kiểm tra quyền ghi của người dùng hiện tại lên thư mục cài đặt, nếu phát hiện thiếu quyền sẽ tự gán thêm SID qua Win32 Security API để chống lỗi phát sinh khi ghi cache/log.
4. **Tham số dòng lệnh hỗ trợ (Command-Line Arguments)**:
   * `-regop`: Chạy tiến trình phụ để thực hiện các thao tác cấu hình Registry.
   * `-localgxx`: Bỏ qua việc tìm kiếm đường dẫn từ Registry, bắt buộc nạp module `gxx.dll` nằm cục bộ.
5. **Thành phần lõi được nạp động**:
   * Tệp `gxx.dll` là thư viện chứa điểm vào chính (`gxxmain` hoặc `gxxlauncher`).
   * Thư mục con bổ sung: `\cef` (Chromium framework), `\libs`, `\plugins` (các plugin mở rộng).

---

## 8. Danh Sách Các Thành Phần Liên Quan Trong Thư Mục Phiên Bản (`2.0.1909.2618`)

| Tên Module | Kích thước | Vai trò kiến trúc |
| :--- | :--- | :--- |
| **`Qt5Core.dll` / `Qt5Widgets.dll` / `Qt5Gui.dll`** | ~12.4 MB | Bộ khung giao diện C++ đa nền tảng Qt 5. |
| **`gacode.dll`** | 2.37 MB | Thư viện mã hóa/giải mã & logic kết nối giao thức mạng của Garena. |
| **`log4cxx.dll`** | 1.94 MB | Hệ thống ghi log chuyên dụng (Apache Log4cxx port). |
| **`CrashReporter.exe`** | 1.57 MB | Bộ phận bắt ngoại lệ và gửi báo cáo sự cố về máy chủ Garena. |
| **`dbghelp.dll`** | 1.23 MB | Thư viện hỗ trợ trích xuất minidump bộ nhớ khi xảy ra crash. |
| **`gxx.dll`** | 352 KB | Module điều khiển trung tâm được `Garena.exe` nạp trực tiếp qua `gxxmain`. |
| **`appshield.dll`** | 329 KB | Module bảo vệ chống can thiệp và bảo mật ứng dụng của Garena. |
| **`gxxcef.exe`** | 248 KB | Tiến trình con nhúng trình duyệt web Chromium Embedded Framework (CEF) hiển thị tab tin tức/sự kiện. |
| **`gxxupdate.exe`** | 229 KB | Trình tải xuống và cài đặt các bản vá tự động. |
| **`ipc.dll`** | 268 KB | Giao tiếp liên tiến trình (Inter-Process Communication) giữa client và các dịch vụ nền (`gxxsvc.exe`). |
| **`libtcmalloc.dll`** | 188 KB | Bộ cấp phát bộ nhớ hiệu năng cao của Google (Thread-Caching Malloc). |
| **`uninst.exe`** | 1.55 MB | Trình gỡ cài đặt ứng dụng. |

---

## 9. Kết Luận

Tệp `Garena.exe` (kích thước 446 KB) là một **Bootstrap Dispatcher** được thiết kế bài bản:
1. **Tính độc lập cao**: Không phụ thuộc vào thư viện bên ngoài ở cấp root (chỉ dùng 4 DLL Windows chuẩn).
2. **Cơ chế cập nhật không Downtime**: Cho phép cập nhật toàn bộ client sang các thư mục phiên bản mới (`2.0.xxxx.xxxx`) mà không cần thay đổi file chạy chính `Garena.exe` hay làm hỏng các shortcut ngoài Desktop.
3. **Độ an toàn cao**: Đầy đủ chữ ký số hợp lệ từ `Garena Online Pte Ltd`, hỗ trợ đầy đủ các cơ chế bảo vệ khai thác bộ nhớ hiện đại (ASLR, DEP/NX), không bị pack, và chỉ yêu cầu quyền hạn tiêu chuẩn `asInvoker`.
