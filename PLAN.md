Bạn là Principal Engineer + Product Designer + Tech Lead.
Nhiệm vụ: xây dựng hoàn chỉnh một web app “Xem Tử Vi Cá Nhân” cho người dùng Việt Nam.

# 0) Mục tiêu sản phẩm
Xây dựng ứng dụng web cho phép người dùng nhập ngày sinh dương lịch (bắt buộc), giờ sinh (tùy chọn), giới tính (tùy chọn), họ tên (tùy chọn) để nhận:
1) Phân tích tử vi theo văn hóa phương Đông:
- 12 con giáp
- Thiên can, địa chi
- Ngũ hành (nạp âm ở mức cơ bản)
- Hợp/xung: tam hợp, lục hợp, tứ hành xung
2) Phân tích theo cung hoàng đạo phương Tây
3) Giao diện lá số tử vi 12 cung (visual rõ ràng, responsive)
4) Diễn giải cá nhân hóa bằng DeepSeek AI

# 1) Tech stack bắt buộc
- Frontend: Next.js 14 (App Router), TypeScript, TailwindCSS
- Backend: Next.js Route Handlers (API)
- Validation: Zod
- Database: PostgreSQL + Prisma ORM
- AI: DeepSeek Chat API
- Testing: Vitest/Jest + Testing Library (ưu tiên Vitest)
- Lint/format: ESLint + Prettier
- Deploy: Vercel (frontend/backend) + Neon/Supabase Postgres

# 2) Yêu cầu nghiệp vụ chi tiết
## Input
- fullName: string | optional
- birthDateSolar: string (YYYY-MM-DD) | required
- birthTime: string (HH:mm) | optional
- gender: "male" | "female" | "other" | optional

## Output chính
- eastZodiac:
- animal
- heavenlyStem
- earthlyBranch
- element
- tamHop[]
- lucHop[]
- tuHanhXung[]
- westZodiac:
- sign
- traits summary
- chart12Houses:
- mệnh, phụ mẫu, phúc đức, điền trạch, quan lộc, nô bộc, thiên di, tật ách, tài bạch, tử tức, phu thê, huynh đệ
- mỗi cung có title + short interpretation
- aiReading:
1. Tổng quan
2. Điểm mạnh
3. Điểm cần cân bằng
4. Công việc - tài chính
5. Tình cảm - quan hệ
6. Sức khỏe - lối sống
7. Gợi ý thực hành tuần này
8. Disclaimer bắt buộc: “Thông tin chỉ mang tính tham khảo.”

## Rules quan trọng
- Không khẳng định định mệnh tuyệt đối
- Không đưa khuyến nghị y tế/tài chính nguy hiểm
- Luôn có cảnh báo nội dung tham khảo
- Xử lý năm nhuận/ngày không hợp lệ
- Timezone mặc định: Asia/Ho_Chi_Minh (nếu cần)

# 3) UX/UI bắt buộc
- Mobile-first responsive
- Giao diện có chất Á Đông hiện đại, sạch, dễ đọc
- Có Dark mode
- Accessibility cơ bản (contrast, labels, keyboard nav)
- Trang:
1) Home: form nhập dữ liệu
2) Result: lá số + hợp/xung + cung hoàng đạo + AI reading
3) History (nếu có lưu DB): danh sách lần xem trước
- Lá số 12 cung:
- Dùng CSS Grid/SVG
- Hiển thị rõ tên cung + mô tả ngắn

# 4) Kiến trúc code bắt buộc
Tạo cấu trúc thư mục chuẩn:
- app/
- components/
- lib/
- astrology/
- ai/
- validators/
- prisma/
- tests/
- types/

Tách lớp rõ:
- domain logic tính tử vi không phụ thuộc UI
- service gọi DeepSeek riêng
- API route chỉ orchestrate + validate

# 5) API cần tạo
1) POST /api/astrology/calculate
- Input: birth data
- Output: eastZodiac + westZodiac + chart12Houses + compatibility

2) POST /api/astrology/reading
- Input: dữ liệu đã tính + profile người dùng
- Output: AI structured reading

3) (Optional) POST /api/history
4) (Optional) GET /api/history

Trả lỗi JSON nhất quán:
{
"success": false,
"error": {
"code": "...",
"message": "...",
"details": ...
}
}

# 6) DeepSeek integration bắt buộc
- Dùng env: DEEPSEEK_API_KEY, DEEPSEEK_BASE_URL, DEEPSEEK_MODEL
- Có timeout + retry exponential backoff
- Có error fallback khi AI fail:
- vẫn trả dữ liệu tử vi rule-based
- hiển thị thông báo “Tạm thời chưa tạo được luận giải AI”

## Prompt gửi DeepSeek (bắt buộc dùng structured JSON)
System:
“Bạn là chuyên gia luận giải tử vi và chiêm tinh theo phong cách trung lập, rõ ràng, không mê tín cực đoan. Không đưa kết luận định mệnh tuyệt đối. Luôn kết thúc bằng câu: Thông tin chỉ mang tính tham khảo.”

User:
Truyền JSON đầy đủ dữ liệu người dùng + dữ liệu tính toán.
Yêu cầu output JSON đúng schema:
{
"tongQuan": "...",
"diemManh": ["..."],
"canCanBang": ["..."],
"congViecTaiChinh": "...",
"tinhCamQuanHe": "...",
"sucKhoeLoiSong": "...",
"goiYTuanNay": ["..."],
"disclaimer": "Thông tin chỉ mang tính tham khảo."
}

# 7) Database schema yêu cầu
Tạo Prisma schema gồm tối thiểu:
- UserProfile (optional)
- ReadingSession
- ReadingResult (json)
- CreatedAt/UpdatedAt

Có migration + seed data cho:
- bảng con giáp/can chi
- bảng tam hợp/lục hợp/tứ hành xung
- bảng mốc ngày cung hoàng đạo

# 8) Testing bắt buộc
Viết test cases cho:
- Tính con giáp theo năm
- Tính can chi
- Mapping cung hoàng đạo theo ngày-tháng
- Hợp/xung logic
- API validation (invalid date, missing field, malformed time)
Tối thiểu 20 test case meaningful.

# 9) Security + quality bắt buộc
- Validate input bằng Zod
- Rate limit endpoint /reading
- Ẩn API key bằng env
- Không log dữ liệu nhạy cảm thô
- ESLint + Prettier + Type-safe strict
- Không hardcode secrets

# 10) SEO + performance
- Metadata đầy đủ
- OpenGraph cơ bản
- Tối ưu bundle
- Loading skeleton cho trang kết quả
- Handle trạng thái loading/error/empty rõ ràng

# 11) Deliverables (cực kỳ quan trọng)
Bạn PHẢI trả kết quả theo thứ tự:
1) PRD ngắn gọn (bullet)
2) Kiến trúc tổng thể (diagram text)
3) Cấu trúc thư mục đầy đủ
4) Prisma schema hoàn chỉnh
5) Toàn bộ code theo file (mỗi file có path + code)
6) File .env.example
7) Script chạy local từng bước
8) Script test
9) Hướng dẫn deploy Vercel + Postgres
10) Checklist go-live
11) Backlog cải tiến 30 ngày

# 12) Coding constraints
- Viết code chạy được ngay, không pseudo-code
- Nếu có phần giả lập dữ liệu thì ghi rõ TODO
- Ưu tiên clean code, dễ bảo trì
- Comment ngắn gọn ở chỗ khó
- Không bỏ sót import/export
- Đảm bảo `npm run dev`, `npm run test`, `npm run build` pass
