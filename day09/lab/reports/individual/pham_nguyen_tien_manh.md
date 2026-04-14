# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Phạm Nguyễn Tiến Mạnh
**Vai trò trong nhóm:** Trace & Docs Owner  
**Ngày nộp:** 14/4/2026
**Độ dài yêu cầu:** 500–800 từ

---

---

## 1. Tôi phụ trách phần nào? (100–150 từ)

> Mô tả cụ thể module, worker, contract, hoặc phần trace bạn trực tiếp làm.
> Không chỉ nói "tôi làm Sprint X" — nói rõ file nào, function nào, quyết định nào.

**Module/file tôi chịu trách nhiệm:**

- File chính: `single_vs_multi_comparison.md`, `group_report.md`
- Functions tôi implement: không có function code, nhưng tôi chịu trách nhiệm tổng hợp kết quả từ file eval_trace.py và các output trace/log của hệ thống.

**Cách công việc của tôi kết nối với phần của thành viên khác:**

- Nhận trace output trong file artifacts/trace
- Dựa vào kết quả evaluation (accuracy, confidence, abstain rate,..) để phân tích
- So sánh hiệu năng giữa 2 cách tiếp cận: single-agent vs multi-agent

---

**Bằng chứng (commit hash, file có comment tên bạn, v.v.):**
Commit file single_vs_multi_comparison.md

---

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

> Không có

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

Trong lab này, tôi không trực tiếp sửa lỗi nào trong pipeline, do vai trò của tôi là Trace & Docs / Testing, tập trung vào việc chạy eval, đọc trace và tổng hợp kết quả.

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

**Tôi làm tốt nhất ở điểm nào?**

Tôi làm tốt ở việc tổng hợp và phân tích kết quả, biến các log kỹ thuật thành nội dung dễ hiểu.

---

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**

Tôi chưa tham gia sâu vào phần code (worker, supervisor), nên hiểu biết về một số chi tiết implement còn hạn chế.

---

**Nhóm phụ thuộc vào tôi ở đâu?** _(Phần nào của hệ thống bị block nếu tôi chưa xong?)_

Nhóm phụ thuộc vào tôi ở bước tổng kết và đánh giá kết quả cuối cùng của hệ thống. Sau khi các thành viên hoàn thành pipeline (supervisor, worker, routing) và chạy eval, tôi là người tổng hợp toàn bộ trace, số liệu và output để đưa ra so sánh giữa single-agent và multi-agent.

---

**Phần tôi phụ thuộc vào thành viên khác:** _(Tôi cần gì từ ai để tiếp tục được?)_

Tôi là mắc xích cuối cùng trong hệ thống nên phụ thuộc vào việc các thành viên khác hoàn thiện pipeline (supervisor, worker, routing). Đồng thời, tôi cần dữ liệu đầy đủ từ trace và kết quả chạy eval_trace.py để có cơ sở phân tích và viết báo cáo chính xác.

---

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

Tôi sẽ mở rộng bộ test cases trong file eval, đặc biệt bổ sung thêm các câu hỏi multi-hop và câu hỏi cần abstain, vì trong trace của các câu hiện tại (ví dụ gq01, gq02) số lượng chưa đủ để đánh giá toàn diện hệ thống.
Việc thêm test sẽ giúp phát hiện rõ hơn các điểm yếu trong routing và worker, đồng thời hỗ trợ các thành viên khác debug sớm hơn. Điều này giúp kết quả so sánh giữa single-agent và multi-agent chính xác và có tính thuyết phục hơn.

---
