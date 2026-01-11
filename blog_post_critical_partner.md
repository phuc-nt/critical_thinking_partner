# Đừng để AI "nghĩ hộ". Hãy để AI dạy bạn cách nghĩ.

Là một kỹ sư phần mềm, tôi tin rằng kỹ năng quan trọng nhất không phải là coding, mà là **Self-Reflection (Tự phản biện)**. 
Khi bạn là PM, Team Lead hay Architect, bạn phải đưa ra hàng tá quyết định mỗi ngày. Một giả định sai lầm ở khâu Planning có thể đánh đổi bằng hàng tháng trời OT của cả team.

Nhưng "tự phản biện" là việc cực khó. Bộ não của chúng ta được lập trình để bảo vệ cái tôi (Ego) và lờ đi các điểm mù (Blindspots). Chúng ta thường quá yêu giải pháp của mình và vô thức bác bỏ các rủi ro.

Thông thường, ta hay dùng AI như một "Thư ký cao cấp": *Viết cho tao PRD. Gợi ý cho tao Architecture.*
Cách này nhanh, nhưng nguy hiểm. Nó biến ta thành những kẻ lười tư duy (Cognitive Misers), dần dần mất đi khả năng đặt câu hỏi sắc bén.

Dự án **Critical Thinking Partner** này ra đời với một sứ mệnh khác hoàn toàn:
**Biến AI thành một Coach khắc nghiệt, người sẽ ép bạn phải tư duy thay vì đưa ra câu trả lời.**

---

## "Probe Only Mode": Trái tim của giải pháp

Trong setup này, tôi xây dựng tính năng quan trọng nhất gọi là **Probe Only Mode** (Chế độ Thử thách).

Ở chế độ này, AI bị cấm tuyệt đối việc đưa ra lời khuyên (Advice) hay giải pháp (Solution). Nhiệm vụ duy nhất của nó là **đặt câu hỏi**.
*   Tại sao bạn lại chọn Tech Stack này?
*   Dựa vào dữ liệu nào bạn tin user cần tính năng đó?
*   Nếu giả định của bạn sai, hậu quả tồi tệ nhất là gì?

Mục tiêu không phải để AI giải quyết vấn đề cho bạn, mà để bạn **học cách đặt câu hỏi**. Qua quá trình bị AI "vặn vẹo", chính bạn sẽ tự hình thành tư duy phản biện sắc bén hơn.

*Tất nhiên, tôi cũng thiết lập một chế độ phụ là **Consult Mode** (Đối tác chuyên gia), nơi AI vừa hỏi vừa gợi ý giải pháp. Chế độ này dành cho lúc bạn bí quá hoặc cần tham khảo kiến thức chuyên ngành (Best Practices), nhưng tôi khuyến khích bạn hạn chế dùng nó nếu muốn thực sự luyện não.*

## Context Engineering: "Cấy" tư duy cho AI

Để AI làm được điều này, tôi không chat chay. Tôi "cấy" cho nó các Framework tư duy bài bản thông qua cấu trúc file trong dự án:

### 1. The Router (`MASTER_INSTRUCTION.md`)
Thay vì trả lời ngay, AI sẽ dừng lại phân tích nhu cầu của bạn để chọn chiến lược:
> "Anh muốn dùng **Socratic** để em vặn vẹo logic, hay dùng **6 Chiếc Mũ** để nhìn tổng quan? Anh muốn em chỉ hỏi (Probe) hay vừa hỏi vừa khuyên (Consult)?"

### 2. Các Framework chuyên sâu (`.ai_prompts/frameworks/`)
Tôi tích hợp sẵn 3 công cụ tư duy kinh điển:
*   **Socratic Method:** Để tìm lỗ hổng trong Requirement.
*   **Six Thinking Hats:** Để cân nhắc lợi/hại trong Planning.
*   **5 Whys:** Để tìm nguyên nhân gốc rễ (Root Cause) trong Incident.

---

## Demo: Những màn "đấu trí" thực tế

Để minh họa, tôi sẽ show cho các bạn 2 kịch bản mà tôi đã test thử.

### Case 1: Sự ngây thơ của Product Owner (Socratic Method)
Tôi đóng vai một PO đang muốn làm tính năng "Mua trước trả sau" (BNPL), với yêu cầu duyệt vay tự động trong 200ms để tăng số.
Tôi chọn **Probe Mode**.

> **Tôi:** "Duyệt vay dưới $500 thì cứ auto duyệt đi. Hacker ai thèm lấy $500 bõ công làm gì."
>
> **AI (Socratic):** "Bạn giả định rằng kẻ tấn công hành động đơn lẻ và thủ công. Nhưng nếu kẻ tấn công dùng script để tạo 1.000 tài khoản ảo và vay đồng loạt trong 1 giây (`Threaded Attack`), thì thiệt hại là $500 hay là $500,000? Hệ thống của bạn có cơ chế nào để phát hiện ra 'Hacker lười' đó không?"

**Bài học:** Tôi nhận ra mình đã tư duy theo kiểu "Happy Path" (User người tốt) mà quên mất tư duy "Adversarial" (Kẻ địch).

### Case 2: Sự cố sập hệ thống (5 Whys)
Hệ thống sập. Nguyên nhân bề mặt là "Dev cấu hình sai timeout".
Tôi lại nhờ AI giúp (vẫn Probe Mode).

AI hỏi tôi chọn Mode. Tôi chọn **Mode A: Probe Only** (Chế độ Coach).

> **Tôi:** "Lỗi do dev frontend không biết impact lên backend nên chỉnh timeout xuống thấp quá."
>
> **AI (Coach):** "Bạn đang đổ lỗi cho cá nhân. Hãy nhìn vào quy trình. Tại sao định nghĩa rủi ro của bạn lại dựa trên **'loại file'** (config vs code) thay vì **'phạm vi ảnh hưởng'**? Dựa vào cơ sở nào mà quy trình release lại cho phép một file config 5 dòng có thể kill 100k users đi qua mà không cần Load Test?"

Câu hỏi này như một cú tát tỉnh ngủ. 
AI không bảo tôi "Hãy sửa quy trình release". Nó buộc tôi phải tự nhận ra sự ngây thơ trong quy trình hiện tại của mình. 
Chính khoảnh khắc tôi phải vắt óc trả lời câu hỏi đó, tôi đã học được bài học về "Blast Radius" (Phạm vi ảnh hưởng) sâu sắc hơn bất kỳ cuốn sách giáo khoa nào.

Đó chính là giá trị cốt lõi: **Học thông qua việc bị thách thức.**

## Tải về và Luyện tập

Tôi đã đóng gói toàn bộ bộ prompt và kịch bản test này trên GitHub.
Bạn chỉ cần clone về, mở bằng Cursor/VS Code (với Copilot), và bắt đầu phiên "đối luyện" của mình.

👉 **Repo:** [`phuc-nt/critical_thinking_partner`](https://github.com/phuc-nt/critical_thinking_partner)

Đừng dùng AI để làm bài hộ nữa. Hãy dùng nó để trở thành một người giải quyết vấn đề (Problem Solver) sắc bén hơn.
