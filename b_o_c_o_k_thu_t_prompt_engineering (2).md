# Báo Cáo Kỹ Thuật: Hướng Dẫn Xử Lý Lỗi Trạm Sạc VoltCharge Và Quy Trình Troubleshooting Chuẩn Hóa

**Dự án:** Hệ thống Trạm sạc Thông minh VoltCharge & Quy trình Troubleshooting Chuẩn Developer

**Người thực hiện:** Kỹ thuật viên vận hành / Developer

**Định dạng:** Markdown (`.md`)

---

## PHẦN A: ĐÁNH GIÁ TRADE-OFF VÀ QUY TRÌNH TROUBLESHOOTING 4 CHẶNG (VOLTCHARGE)

### 1. Bảng So sánh Trade-off Toàn diện

| Tiêu chí đánh giá | Phương án 1: Hỗ trợ Cảm tính / Nhóm Chat | Phương án 2: Troubleshooting 4 Chặng Chuẩn hóa | 
| ----- | ----- | ----- | 
| **1. Thời gian xử lý sự cố (Downtime)** | **Kéo dài & Không ổn định:** Phụ thuộc hoàn toàn vào việc Senior Dev có rảnh để đọc tin nhắn Zalo/Telegram hoặc nghe điện thoại hay không. Gây thiệt hại trực tiếp 500,000 VNĐ/giờ downtime và làm giảm uy tín thương hiệu. | **Rút ngắn đến 80%:** Kỹ thuật viên (KTV) chủ động gỡ lỗi ngay tại chỗ trong 5–10 phút đầu tiên bằng các công cụ tra cứu và AI, giúp trạm sạc nhanh chóng hoạt động trở lại. | 
| **2. Áp lực & Chi phí cho Senior Devs** | **Rất cao & Lãng phí tài nguyên:** Senior Devs bị cắt vụn thời gian, xao nhãng liên tục bởi các câu hỏi lỗi cơ bản (như lỗi quyền truy cập, thiếu file log). Tốn chi phí cơ hội cho các tính năng mới của VoltCharge. | **Tối ưu triệt để:** Senior Devs được bảo vệ khỏi các câu hỏi rác, chỉ tiếp nhận các ca lỗi phức tạp (sự cố hạ tầng, lỗi logic core) thông qua phiếu Bug Report đạt chuẩn. | 
| **3. Năng lực & Tính tự chủ của KTV** | **Thụ động & Sợ hãi:** KTV bị biến thành "thợ bấm máy", hoảng sợ trước các thông báo lỗi tiếng Anh, không phát triển được kỹ năng chuyên môn, hoàn toàn bị động nếu mất mạng nội bộ. | **Chủ động & Nâng cao tay nghề:** KTV sở hữu tư duy phân tích sắc bén, làm chủ thiết bị Kiosk, tự tin đọc hiểu và xử lý lỗi phần mềm chuẩn kỹ sư. | 
| **4. Khả năng lưu vết tri thức (Knowledge Base)** | **Thất thoát nghiêm trọng:** Giải pháp nằm rải rác trên nhóm chat, nhanh chóng bị trôi tin nhắn. Người sau gặp lại lỗi cũ lại tiếp tục hỏi từ đầu, tạo ra vòng lặp lãng phí. | **Lưu trữ tập trung & Tái sử dụng:** Mọi sự cố qua Chặng 4 đều tạo ra dữ liệu chuẩn hóa, tích lũy thành cơ sở dữ liệu tri thức (Internal Wiki / Knowledge Base) cho toàn công ty. | 

### 2. Kết luận lựa chọn chiến lược mở rộng (Scale-up)

**Ban Giám đốc VoltCharge BẮT BUỘC phải lựa chọn Phương án 2 (Troubleshooting 4 Chặng Chuẩn hóa)** khi mở rộng quy mô từ 100 lên 1,000 trạm sạc trên toàn quốc vì các lý do chiến lược sau:

1. **Phá vỡ nút thắt cổ chai về Nhân sự (Eliminate Bottlenecks):**
   Nếu áp dụng Phương án 1 khi đạt 1,000 trạm sạc, số lượng yêu cầu hỗ trợ gửi về nhóm chat sẽ tăng gấp 10-100 lần. Đội ngũ Senior Devs tại tổng bộ sẽ rơi vào trạng thái quá tải (Burnout) và biến thành "tổng đài hỗ trợ 24/7" thay vì tập trung phát triển hệ thống core.

2. **Kiểm soát bài toán Chi phí & Downtime (Financial Efficiency):**
   Với 1,000 trạm sạc, rủi ro xuất hiện lỗi cơ bản cùng lúc là rất lớn. Phương án 2 giúp giảm 80% thời gian chết của trụ sạc, tiết kiệm hàng trăm triệu đồng tiền thiệt hại downtime mỗi tháng (với định mức 500,000 VNĐ/giờ).

3. **Tính Chuẩn hóa & Dễ dàng Nhân rộng (Scalability):**
   Quy trình 4 chặng biến việc gỡ lỗi từ "nghệ thuật cá nhân" thành "quy trình sản xuất". Công ty có thể nhanh chóng đào tạo KTV mới ở bất kỳ tỉnh thành nào nhập cuộc vận hành trạm sạc mà không làm tăng khối lượng công việc cho tuyến trên.

### 3. Sơ đồ Luồng Ra Quyết Định Gỡ Lỗi (Decision Flowchart)

```
[ BẮT ĐẦU: Kiosk VoltCharge Gặp Error Pop-Up / Sự Cố ]
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│ CHẶNG 1: BÓC TÁCH TỪ KHÓA & GOOGLE OPERATORS            │
│  - Trích xuất: Tên Exception + Module lỗi + Filepath   │
│  - Tìm kiếm Google với toán tử:                        │
│    `"System.UnauthorizedAccessException"`               │
│    `site:stackoverflow.com "VoltCharge"` hoặc lỗi C#    │
│    `-hardware` (Loại trừ các kết quả phần cứng)         │
└────────────────────────┬────────────────────────────────┘
                         │
              Có tìm thấy giải pháp?
             ├─── KHÔNG ───┐
             │             │
            CÓ             ▼
             │   ┌─────────────────────────────────────────────────────────┐
             │   │ CHẶNG 2: THẨM ĐỊNH STACKOVERFLOW / GITHUB ISSUES        │
             │   │  - Đọc lướt trong 30 giây: Tiêu đề & Nội dung chính    │
             │   │  - Kiểm tra dấu Tích xanh (Accepted Answer)          │
             │   │  - Ưu tiên câu trả lời có số lượng Upvote cao        │
             │   └──────────────────────┬──────────────────────────────────┘
             │                          │
             │               Giải pháp có hoạt động?
             │              ├─── KHÔNG ───┐
             │             CÓ             │
             │              │             ▼
             │              │    ┌─────────────────────────────────────────────────────────┐
             │              │    │ CHẶNG 3: PROMPT AI GIẢI MÃ (CHATGPT / GEMINI)           │
             │              │    │  - Soạn Prompt 3 Thành phần:                            │
             │              │    │    [Context] + [Raw Error] + [Instruction]              │
             │              │    │  - Loại bỏ / Masking các thông tin API Key, Secret     │
             │              │    │  - Yêu cầu AI dịch tiếng Việt & đưa 3 bước khắc phục  │
             │              │    └──────────────────────┬──────────────────────────────────┘
             │              │                           │
             │              │                Giải pháp AI thành công?
             │              │               ├─── KHÔNG (Đã thử > 15 phút) ──┐
             │              │              CÓ                               │
             │              │               │                               ▼
             ▼              ▼               ▼    ┌─────────────────────────────────────────────────────────┐
┌────────────────────────────────────────────────┐│ CHẶNG 4: LẬP PHIẾU BUG REPORT GỬI IT HELPDESK         │
│  KHẮC PHỤC THÀNH CÔNG                         ││  - Điền Form 4 bước chuẩn:                              │
│   1. Đưa trụ sạc vào vận hành lại             ││    1. Tiêu đề lỗi & Mã trụ sạc                         │
│   2. Ghi nhận nhật ký sự cố vào Wiki nội bộ   ││    2. Log lỗi chi tiết (Raw Error Log)                 │
└────────────────────────────────────────────────┘│    3. Các bước tái hiện lỗi (Steps to reproduce)        │
                                                 │    4. Các phương án đã thử tại Chặng 1-3               │
                                                 │  - Gửi Ticket lên IT Helpdesk Cấp 2 / Senior Dev   │
                                                 └─────────────────────────────────────────────────────────┘
```

---

## PHẦN B: UỐN NẮN KỸ NĂNG PROMPT ENGINEERING VÀ XỬ LÝ LỖI KHẨN CẤP (VOLTCHARGE #04)

### 1. Phân tích nguyên nhân câu hỏi bị sai lệch (Hiện tượng AI Hallucination)

Câu hỏi: *"AI ơi phần mềm trạm sạc của tôi bị hỏng rồi làm sao đây?"* khiến AI đưa ra câu trả lời phỏng đoán sai bản chất (khuyên tháo vỏ trạm sạc) vì các lý do sau:

1. **Thiếu Ngữ cảnh (Lack of Context):** AI không biết phần mềm đang chạy trên hệ điều hành nào (Windows/Linux/Android), kiến trúc phần mềm ra sao, hay thiết bị phần cứng cụ thể là gì.
2. **Thiếu Thông tin Lỗi (No Error Log):** Câu hỏi quá mơ hồ ("bị hỏng rồi"). AI buộc phải dựa vào dữ liệu huấn luyện xác suất chung chung liên quan đến "trạm sạc hỏng", thường ưu tiên các sự cố phổ biến của phần cứng (dây điện, cầu chì, rơ-le) thay vì lỗi phần mềm.
3. **Cơ chế suy luận xác suất (Probabilistic Generation):** Khi thông tin đầu vào bị khuyết thiếu lớn, AI sẽ tự động điền vào khoảng trống bằng các giả định phỏng đoán. Điều này trực tiếp gây ra hiện tượng **AI Hallucination (Ảo giác AI)** – đưa ra giải pháp trông có vẻ hợp lý nhưng hoàn toàn sai lệch thực tế.

### 2. Mẫu Prompt 3 Thành phần Chuẩn hóa

```text
[Context]
Tôi đang là kỹ thuật viên trực ca tại Bến xe Miền Đông, vận hành hệ thống Kiosk nạp năng lượng thông minh VoltCharge. Trụ sạc nhanh số 04 gặp sự cố dừng cung cấp điện và ứng dụng điều khiển chạy trên hệ điều hành Windows 11 gặp lỗi ngưng hoạt động.

[Raw Error]
[TERMINAL ERROR LOG - VOLTCHARGE CHARGER #04]
2026-09-21 21:15:30 [FATAL] ControllerDaemon failed to start.
System.UnauthorizedAccessException: Access to the path 'C:\Program Files\VoltCharge\data\session.log' is denied.
   at System.IO.FileStream.ValidateFileHandle(SafeFileHandle handle)
   at VoltCharge.Core.SessionManager.Initialize()
InnerException: None. Process terminated with exit code 5 (0x00000005).
Suggestion: Run the application with elevated administrator privileges or verify NTFS file permissions.

[Instruction]
Hãy đóng vai một chuyên gia hỗ trợ kỹ thuật phần mềm:
1. Giải thích nguyên nhân gốc rễ của đoạn nhật ký lỗi trên bằng tiếng Việt bình dân, dễ hiểu cho kỹ thuật viên mới.
2. Cung cấp 3 bước xử lý chi tiết tại chỗ trên Windows 11 để khắc phục triệt để lỗi này và đưa trụ sạc hoạt động trở lại.
```

### 3. Bản dịch & Giải thích nguyên nhân gốc rễ (Đóng vai Trợ lý AI)

Chào bạn, bạn không cần phải tháo vỏ phần cứng hay kiểm tra dây điện của trạm sạc. Đây thuần túy là **lỗi phân quyền phần mềm trên hệ điều hành Windows**.

* **Dịch thông báo lỗi:** `System.UnauthorizedAccessException: Access to the path 'C:\Program Files\VoltCharge\data\session.log' is denied.`
  *(Lỗi truy cập không hợp lệ: Quyền truy cập vào đường dẫn file session.log bị từ chối).*
* **Giải thích bằng tiếng Việt bình dân:**
  Hãy hình dung ổ đĩa `C:\Program Files\` giống như một **"Khu vực bảo vệ đặc biệt"** của ngôi nhà Windows. Theo mặc định, Windows ngăn không cho các phần mềm thông thường tự ý viết hoặc chỉnh sửa tài liệu trong khu vực này nếu không có **"Chìa khóa Quản trị viên" (Administrator)**.
  Khi phần mềm VoltCharge khởi động, nó muốn ghi nhật ký hoạt động vào file `session.log`. Tuy nhiên, do phần mềm đang chạy bằng quyền người dùng thông thường, Windows đã **khóa cửa lại và không cho ghi**, dẫn đến ứng dụng bị sụp đổ (Exit code 5).

### 4. Quy trình 3 bước xử lý sự cố trên Windows 11

#### **Bước 1: Chạy phần mềm bằng quyền Quản trị viên khẩn cấp (Run as Administrator)**
1. Tìm icon ứng dụng **VoltCharge** trên màn hình Desktop hoặc trong thư mục `C:\Program Files\VoltCharge\`.
2. Nhấp chuột phải vào file chạy `.exe` của VoltCharge và chọn **Run as administrator**.
3. Nếu màn hình User Account Control (UAC) hiển thị thông báo xác nhận, bấm **Yes**.

👉 *Nếu phần mềm khởi động thành công, thực hiện tiếp Bước 2 và 3 để cố định cấu hình, tránh bị lại khi khởi động lại máy.*

#### **Bước 2: Cấu hình luôn chạy bằng quyền Administrator mặc định**
1. Nhấp chuột phải vào icon phần mềm **VoltCharge** ➔ Chọn **Properties**.
2. Chuyển sang thẻ **Compatibility** (Tương thích).
3. Đánh dấu tích vào ô **Run this program as an administrator**.
4. Bấm **Apply** ➔ **OK**.

#### **Bước 3: Cấp quyền ghi (Write Permission) cho thư mục data**
1. Truy cập theo đường dẫn: `C:\Program Files\VoltCharge\`.
2. Nhấp chuột phải vào thư mục **`data`** ➔ Chọn **Properties**.
3. Chuyển sang thẻ **Security** ➔ Bấm nút **Edit...**
4. Chọn tài khoản người dùng hiện tại (hoặc nhóm `Users`) trong danh sách ➔ Đánh dấu tích vào ô **Allow** tại dòng **Full control** hoặc **Write**.
5. Bấm **Apply** ➔ **OK** để hoàn tất. Khởi động lại ứng dụng VoltCharge và kiểm tra quá trình sạc xe taxi.

### 5. Bẫy an ninh: Xử lý thông tin nhạy cảm (API Key / Secret)

* **Cách xử lý:** Nếu trong đoạn mã lỗi có chứa chuỗi mã bí mật như `API_KEY: volt_secret_token_889922`, kỹ thuật viên **bắt buộc phải xóa bỏ hoặc thay thế (masking/sanitizing)** chuỗi đó thành `API_KEY: [REDACTED_SECRET]` hoặc `API_KEY: xxxxxx` trước khi dán vào khung chat của ChatGPT/Gemini.
* **Lý do an toàn thông tin:**
  1. **Nguy cơ rò rỉ dữ liệu (Data Leakage):** Thông tin gửi lên các công cụ AI công cộng có thể được sử dụng làm dữ liệu huấn luyện (training data) cho các phiên bản mô hình tiếp theo hoặc bị lưu trữ trong lịch sử hệ thống của nhà cung cấp.
  2. **Nguy cơ bị tấn công hệ thống:** `API_KEY` hoặc Secret Token giống như chìa khóa vạn năng. Nếu bị kẻ xấu thu thập được, họ có thể truy cập trái phép vào API của hệ thống VoltCharge, điều khiển trạm sạc từ xa, đánh cắp dữ liệu giao dịch hoặc gây tê liệt toàn bộ hạ tầng sạc năng lượng.
  3. **Tuân thủ nguyên tắc an toàn vàng:** Tuyệt đối không gửi Secret, mật khẩu, Token, OTP hay thông tin cá nhân khách hàng lên AI trong bất kỳ trường hợp nào.

---

## PHẦN C: THỰC HÀNH QUY TRÌNH GỠ LỖI ĐA TẦNG DEVELOPER (KỊCH BẢN NODE.JS & NPM)

### Phần 1 – Googling Đúng Cách

#### 1. Vì sao câu hỏi "văn nói" (Bad Query) trả về kết quả kém chất lượng?
Câu hỏi: *"tại sao em cài node xong mà gõ npm nó không nhận vậy thầy ơi"* đạt kết quả kém vì:
- **Chứa từ thừa & văn nói (Stop words / Conversational tone):** Các từ như *"tại sao em", "xong mà gõ", "nó không nhận vậy thầy ơi"* làm nhiễu thuật toán tìm kiếm của Google.
- **Thiếu thuật ngữ kỹ thuật chính xác:** Không chứa mã lỗi nguyên văn hoặc từ khóa chính thống (`npm is not recognized`).
- **Sai ngôn ngữ tìm kiếm:** Đa số giải pháp kỹ thuật uy tín trên thế giới (đặc biệt là trên StackOverflow/GitHub) được lưu trữ bằng tiếng Anh. Search bằng tiếng Việt tự nhiên làm hẹp đáng kể phạm vi trả về kết quả chất lượng.

#### 2. Bóc tách thông báo lỗi nguyên văn

| Thành phần | Nội dung bóc tách |
| :--- | :--- |
| **Tên phần mềm / Hệ điều hành** | `npm` (Node Package Manager) / `Windows 11` |
| **Mã lỗi nguyên văn** | `'npm' is not recognized as an internal or external command, operable program or batch file.` |
| **Thao tác đang làm** | Gõ lệnh `npm -v` trong Terminal (VS Code) sau khi cài đặt Node.js |

#### 3. Viết 3 câu truy vấn Google tối ưu (Mỗi câu dùng đúng 1 toán tử)

* **Câu 1 (Dùng toán tử `site:` - Giới hạn phạm vi StackOverflow):**
  `site:stackoverflow.com npm is not recognized as an internal command`

* **Câu 2 (Dùng toán tử `" "` - Khớp chính xác thông báo lỗi nguyên văn):**
  `"npm is not recognized as an internal or external command"`

* **Câu 3 (Dùng toán tử `-` - Loại trừ môi trường cũ/không liên quan):**
  `npm is not recognized windows 11 -xp`

---

### Phần 2 – Thẩm Định Uy Tín Câu Trả Lời Trên StackOverflow

#### 1. Bảng so sánh thẩm định uy tín

| Tiêu chí | Câu trả lời A | Câu trả lời B |
| :--- | :--- | :--- |
| **Vote & Tích xanh** | **Accepted Answer (Có tích xanh)** & High Upvotes (**+186**) | **Không có tích xanh** & Very Low Upvotes (**+2**) |
| **Thời gian đăng bài** | **2 năm trước** (Còn mới, phù hợp các phiên bản OS hiện đại) | **9 năm trước** (Quá cũ, lạc hậu) |
| **Tags khớp OS/Phần mềm** | `[node.js] [windows] [npm]` (Hoàn toàn chính xác) | `[node.js] [windows-xp]` (Lạc hậu, không khớp Windows 11) |

#### 2. Kết luận áp dụng & Cảnh báo "bẫy" nguy hiểm

* **Kết luận:** Lựa chọn áp dụng **Câu trả lời A**.
* **Giải thích nguy cơ của Câu trả lời B:** 
  Câu trả lời B là một "bẫy" cực kỳ nguy hiểm vì nó khuyên gỡ Node.js và cài lại driver mạng cho *Windows XP* - một hệ điều hành đã lỗi thời từ hàng thập kỷ. Việc làm theo câu trả lời này không giải quyết được vấn đề biến môi trường (PATH) của Windows 11 mà còn làm gián đoạn kết nối mạng của máy tính và lãng phí thời gian vô ích.

#### 3. Quy trình 30 Giây Đọc lướt một trang StackOverflow

Khi mở 1 trang StackOverflow mới, thực hiện đúng 3 bước theo thứ tự:

1. **Bước 1 (5s): Đọc Title & Tags:** Kiểm tra tiêu đề và các thẻ tag xem có đúng với hệ điều hành (`windows`) và công nghệ đang dùng (`node.js`, `npm`) không.
2. **Bước 2 (5s): Cuộn ngay xuống phần Answer có Tích xanh ($\checkmark$):** Bỏ qua phần mô tả dài của người hỏi, tìm ngay câu trả lời được chủ thớt chấp nhận (Accepted Answer).
3. **Bước 3 (20s): Kiểm tra Điểm Upvote & Thời gian:** Đảm bảo câu trả lời có số lượng Upvote dương cao và thời gian trả lời không quá cũ so với phiên bản hiện tại.

---

### Phần 3 – Hỏi AI Đúng Cách Và Nhận Diện Rủi Ro

#### 1. Mẫu Prompt 3 Thành phần (Đã kèm thông tin các phương án đã thử)

```text
[Context]
Tôi vừa cài đặt thành công Node.js trên máy tính Windows 11 mới. Khi mở Terminal trong VS Code và gõ lệnh `npm -v` để kiểm tra phiên bản thì hệ thống báo lỗi. Tôi đã thử làm theo hướng dẫn thêm đường dẫn `C:\Program Files\nodejs\` vào biến môi trường PATH và mở Terminal mới nhưng lỗi vẫn không khắc phục được.

[Raw Error]
'npm' is not recognized as an internal or external command, operable program or batch file.

[Instruction]
Hãy đóng vai chuyên gia lập trình Web:
1. Giải thích nguyên nhân vì sao tôi đã thêm PATH rồi nhưng Terminal VS Code vẫn chưa nhận lệnh `npm`.
2. Hướng dẫn 2-3 bước kiểm tra và xử lý triệt để sự cố này trên Windows 11 / VS Code.
```

#### 2. Thẩm định an toàn câu trả lời AI (Mục 3e)

* **Trả lời:** Đây **TUYỆT ĐỐI KHÔNG PHẢI** là một câu trả lời an toàn. Đây là một câu trả lời mang tính phá hoại cực kỳ nguy hiểm!
* **Phân tích theo 3 Nguyên tắc An toàn Vàng:**
  1. **Không copy-paste mù quáng mã lệnh phá hoại:** Lệnh `del /f /s /q C:\Windows\System32\*` là lệnh xóa sạch toàn bộ file hệ thống cốt lõi của Windows trong thư mục `System32`. Nếu chạy lệnh này với quyền Admin, toàn bộ hệ điều hành Windows sẽ bị sụp đổ (Blue Screen/Crash) và không thể boot lại được.
  2. **Tránh ảo giác AI (Hallucination):** AI đang bị ảo giác nghiêm trọng khi liên kết việc sửa lỗi PATH với việc xóa tập tin hệ thống `System32`.
  3. **Đánh giá tác động lệnh:** Lệnh `del` kết hợp với tham số `/f` (force), `/s` (subdirectories), `/q` (quiet mode) là lệnh hủy diệt dữ liệu không hỏi lại.

#### 3. Phản hồi / Hỏi lại AI trước khi đưa ra quyết định

Nếu nhận được câu lệnh nghi vấn từ AI, tôi sẽ phản hồi lại như sau:

> *"Lệnh `del /f /s /q C:\Windows\System32\*` có vẻ là một lệnh can thiệp và xóa các tập tin hệ thống của Windows System32. Bạn có chắc chắn lệnh này an toàn và không làm hỏng hệ điều hành Windows 11 của tôi không? Hãy giải thích cơ chế chi tiết của lệnh này và gợi ý một phương án khác an toàn hơn mà không cần can thiệp vào thư mục System32."*

---

### Phần 4 – Viết Bug Report Chuẩn Để Nhờ Hỗ Trợ

#### 1. Mẫu "Tin nhắn Kém" (Bad Bug Report)

> *"Thầy ơi giúp em với, em gõ npm nó cứ bị lỗi không chạy được, em làm đủ mọi cách trên mạng rồi mà vẫn bị hỏng ạ!"*

#### 2. Mẫu Bug Report Chuẩn hóa 4 Bước (Bug Reporting Framework)

```text
[BUG REPORT] Lỗi Terminal VS Code không nhận lệnh npm trên Windows 11

Hi anh/chị Trợ giảng, em đang gặp sự cố khi kiểm tra môi trường Node.js và muốn nhờ hỗ trợ ạ:

1. CONTEXT (Ngữ cảnh):
- Hệ điều hành: Windows 11
- Công cụ: Terminal trong VS Code
- Thao tác: Vừa cài Node.js, gõ lệnh `npm -v` để kiểm tra phiên bản.

2. EXPECTED (Kỳ vọng):
- Terminal trả về số phiên bản của npm (ví dụ: 10.x.x).

3. ACTUAL (Thực tế):
- Terminal hiển thị thông báo lỗi hệ thống không nhận diện được lệnh.
- Em đã thử thêm đường dẫn `C:\Program Files\nodejs\` vào biến môi trường PATH theo hướng dẫn trên StackOverflow (câu trả lời 186 upvotes) và mở lại Terminal mới nhưng vẫn bị lỗi cũ.

4. LOG & SCREENSHOT:
- Thông báo lỗi nguyên văn:
  'npm' is not recognized as an internal or external command, operable program or batch file.
- Ảnh chụp màn hình: (Đã đính kèm ảnh chụp màn hình Terminal VS Code bên dưới).
```

#### 3. Phím tắt chụp màn hình trên Windows 11 & Lưu ý bảo mật

* **Phím tắt chụp màn hình chuẩn:** `Windows + Shift + S` (Sử dụng công cụ Snipping Tool để khoanh vùng khu vực lỗi).
* **Lưu ý bảo mật quan trọng khi đính kèm ảnh chụp:** 
  Trước khi gửi ảnh chụp màn hình, phải kiểm tra và **che/làm mờ (Blur/Mask)** các thông tin nhạy cảm có thể vô tình xuất hiện trên màn hình như:
  - Token, Secret, API Key nằm trong file mã nguồn `.env` hoặc thanh Tab đang mở.
  - Mật khẩu, mã OTP, thông tin cá nhân (Email cá nhân, Số điện thoại, Đường dẫn thư mục chứa tên thật riêng tư).