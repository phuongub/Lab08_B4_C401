# Báo Cáo Nhóm — Lab Day 09: Multi-Agent Orchestration

**Tên nhóm:** B4_C401

**Thành viên:**
| Tên | Vai trò | Email |
|-----|---------|-------|
| Nguyễn Thị Thùy Trang | Trace & Docs Owner | nuyenthuytrang372004@gmail.com |
| Trần Việt Phương | Worker Owner | phuong251202@gmail.com |
| Lê Đức Anh | MCP Owner | ducanh198369@gmail.com |
| Phạm Hoàng Tiến Mạnh | Trace & Docs Owner |phamnguyentienmanh2004@gmail.com |
| Nguyễn Ngọc Tân | Trace & Docs Owner | |
| Nguyễn Hoàng Nghĩa | Worker Owner | nhnghia210@gmail.com |
| Trịnh Uyên Chi | Supervisor Owner | trinhuyenchi2003@gmail.com |

**Ngày nộp:** 14/4/2026
**Repo:** https://github.com/phuongub/Lab08_B4_C401
**Độ dài khuyến nghị:** 600–1000 từ

---

---

## 1. Kiến trúc nhóm đã xây dựng (150–200 từ)

> Mô tả ngắn gọn hệ thống nhóm: bao nhiêu workers, routing logic hoạt động thế nào,
> MCP tools nào được tích hợp. Dùng kết quả từ `docs/system_architecture.md`.

**Hệ thống tổng quan:**

Nhóm xây dựng hệ thống theo pattern Supervisor-Worker, trong đó mỗi thành phần đảm nhận một vai trò cụ thể trong pipeline. Supervisor chịu trách nhiệm phân tích câu hỏi và quyết định route sang worker phù hợp. Retrieval worker thực hiện truy xuất thông tin từ ChromaDB, policy_tool worker xử lý các câu hỏi liên quan đến policy và gọi MCP tools khi cần, còn synthesis worker tổng hợp câu trả lời cuối cùng dựa trên context đã thu thập.
Pipeline giúp tách rõ các bước xử lý từ phân loại, truy xuất, kiểm tra policy đến sinh câu trả lời, thay vì xử lý tất cả trong một lần gọi LLM như ở Day 08. Điều này giúp hệ thống dễ debug hơn, dễ kiểm soát và có khả năng mở rộng khi cần thêm worker hoặc tool mới.

---

---

**Routing logic cốt lõi:**

Supervisor sử dụng rule-based routing dựa trên keyword. Nếu câu hỏi chứa các từ khóa liên quan đến IT support (SLA, P1, VPN, password) thì route sang retrieval_worker; nếu liên quan đến policy (refund, access, level 1–4) thì route sang policy_tool_worker; nếu có dấu hiệu rủi ro hoặc khẩn cấp thì route sang human_review. Các trường hợp còn lại mặc định sử dụng retrieval_worker.

---

**MCP tools đã tích hợp:**

- `search_kb`: dùng để tìm thêm tài liệu khi retrieval chưa đủ context
- `get_ticket_info`: lấy thông tin ticket từ hệ thống nội bộ
- `check_access_permission`: kiểm tra quyền truy cập và danh sách approver

---

## 2. Quyết định kỹ thuật quan trọng nhất (200–250 từ)

> Chọn **1 quyết định thiết kế** mà nhóm thảo luận và đánh đổi nhiều nhất.
> Phải có: (a) vấn đề gặp phải, (b) các phương án cân nhắc, (c) lý do chọn phương án đã chọn.

**Quyết định:** **\*\*\*\***\_\_\_**\*\*\*\***

**Bối cảnh vấn đề:**

---

**Các phương án đã cân nhắc:**

| Phương án | Ưu điểm | Nhược điểm |
| --------- | ------- | ---------- |
| \_\_\_    | \_\_\_  | \_\_\_     |
| \_\_\_    | \_\_\_  | \_\_\_     |

**Phương án đã chọn và lý do:**

---

**Bằng chứng từ trace/code:**

> Dẫn chứng cụ thể (VD: route_reason trong trace, đoạn code, v.v.)

```
[NHÓM ĐIỀN VÀO ĐÂY — ví dụ trace hoặc code snippet]
```

---

## 3. Kết quả grading questions (150–200 từ)

> Sau khi chạy pipeline với grading_questions.json (public lúc 17:00):
>
> - Nhóm đạt bao nhiêu điểm raw?
> - Câu nào pipeline xử lý tốt nhất?
> - Câu nào pipeline fail hoặc gặp khó khăn?

**Tổng điểm raw ước tính:** \_\_\_ / 96

**Câu pipeline xử lý tốt nhất:**

- ID: **\_ — Lý do tốt: **\*\*\***\*\_\*\***\*\*\*\*\*\*

**Câu pipeline fail hoặc partial:**

- ID: **\_ — Fail ở đâu: **\*\*\***\*\_\*\***\*\*\***\*  
  Root cause: **\*\*\*\***\_\_\_**\*\*\*\*\*\*

**Câu gq07 (abstain):** Nhóm xử lý thế nào?

---

**Câu gq09 (multi-hop khó nhất):** Trace ghi được 2 workers không? Kết quả thế nào?

---

---

## 4. So sánh Day 08 vs Day 09 — Điều nhóm quan sát được (150–200 từ)

> Dựa vào `docs/single_vs_multi_comparison.md` — trích kết quả thực tế.

**Metric thay đổi rõ nhất (có số liệu):**
Multi-hop accuracy tăng từ khoảng 40% lên 70%,

---

**Điều nhóm bất ngờ nhất khi chuyển từ single sang multi-agent:**
Khả năng debug cải thiện rõ rệt. Với trace và route_reason, nhóm có thể xác định chính xác lỗi nằm ở bước nào thay vì phải đọc toàn bộ pipeline.

---

**Trường hợp multi-agent KHÔNG giúp ích hoặc làm chậm hệ thống:**
Với các câu hỏi đơn giản, multi-agent không cải thiện nhiều về accuracy nhưng lại làm tăng latency do thêm bước routing và worker.

---

---

## 5. Phân công và đánh giá nhóm (100–150 từ)

> Đánh giá trung thực về quá trình làm việc nhóm.

**Phân công thực tế:**

| Thành viên            | Phần đã làm                                                | Sprint |
| --------------------- | ---------------------------------------------------------- | ------ |
| Nguyễn Thị Thùy Trang | Làm báo cáo file routing_decisions, system_architecture    | 4      |
| Trần Việt Phương      | Làm 2 worker: policy_tool với synthesis                    | 2      |
| Lê Đức Anh            | Hoàn thiện file mcp_server.py                              | 3      |
| Phạm Hoàng Tiến Mạnh  | Làm báo cáo file single_vs_multi_comparison và report nhóm | 4      |
| Nguyễn Ngọc Tân       | Hoàn thiện và chạy code file eval_trace                    | 4      |
| Nguyễn Hoàng Nghĩa    | Làm worker retrieval                                       | 2      |
| Trịnh Uyên Chi        | Hoàn thiện file graph.py                                   | 1      |

**Điều nhóm làm tốt:**
Phân chia rõ ràng theo từng module (worker, MCP, docs), giúp làm song song hiệu quả.

---

**Điều nhóm làm chưa tốt hoặc gặp vấn đề về phối hợp:**
Routing logic còn đơn giản, dễ sai với câu hỏi phức tạp. Chưa tối ưu latency.

---

**Nếu làm lại, nhóm sẽ thay đổi gì trong cách tổ chức?**
Sử dụng LLM classifier cho routing thay vì keyword-based để tăng độ chính xác.

---

---

## 6. Nếu có thêm 1 ngày, nhóm sẽ làm gì? (50–100 từ)

> Nhóm sẽ cải thiện routing bằng cách dùng LLM classifier thay vì rule-based để giảm lỗi route sai. Ngoài ra, sẽ thêm caching cho retrieval và tối ưu pipeline để giảm latency, đồng thời bổ sung re-ranking để cải thiện chất lượng context.

---

---

_File này lưu tại: `reports/group_report.md`_  
_Commit sau 18:00 được phép theo SCORING.md_
