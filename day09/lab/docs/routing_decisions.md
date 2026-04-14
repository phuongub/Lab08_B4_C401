# Routing Decisions Log — Lab Day 09

**Nhóm:** C401-B4  
**Ngày:** 14/04/2026

> **Hướng dẫn:** Ghi lại ít nhất **3 quyết định routing** thực tế từ trace của nhóm.
> Không ghi giả định — phải từ trace thật (`artifacts/traces/`).
> 
> Mỗi entry phải có: task đầu vào → worker được chọn → route_reason → kết quả thực tế.

---

## Routing Decision #1

**Task đầu vào:**
> "Nhân viên vừa vào thử việc (trong probation period) muốn làm remote vì lý do cá nhân. Điều kiện là gì?"

**Worker được chọn:** `policy_tool_worker`  
**Route reason (từ trace):** `Task liên quan đến chính sách/điều khoản → policy_tool_worker (MCP) | có thể cần MCP search_kb/get_ticket_info nếu context nội bộ chưa đủ`  
**MCP tools được gọi:** `search_kb`  
**Workers called sequence:** policy_tool_worker → synthesis_worker

**Kết quả thực tế:**
- final_answer (ngắn): Khách hàng Flash Sale **không được hoàn tiền** ngay cả khi sản phẩm lỗi [POLICY EXCEPTIONS]
- confidence: 0.48
- Correct routing? Yes 

**Nhận xét:** _(Routing này đúng hay sai? Nếu sai, nguyên nhân là gì?)_

Routing đúng vì đây là câu hỏi policy/HR. Tuy nhiên policy analysis bị lệch sang refund exception, cho thấy worker policy hiện còn dùng rule quá rộng hoặc context retrieval chưa đủ sạch.

---

## Routing Decision #2

**Task đầu vào:**
> "SLA xử lý ticket P1 là bao lâu?"

**Worker được chọn:** `retrieval_worker`  
**Route reason (từ trace):** `Task hỏi về hệ thống IT/SLA → retrieval_worker (no MCP)`  
**MCP tools được gọi:** None
**Workers called sequence:** retrieval_worker → synthesis_worker

**Kết quả thực tế:**
- final_answer (ngắn): SLA P1: phản hồi 15 phút, xử lý 4 giờ
- confidence: 0.46
- Correct routing? Yes 

**Nhận xét:**

Đây là case retrieval tốt nhất trong các trace bạn gửi. Routing đúng và retrieval lấy được chunk SLA thật, nên confidence cao hơn các case còn lại.

---

## Routing Decision #3

**Task đầu vào:**
> "Khách hàng Flash Sale yêu cầu hoàn tiền vì sản phẩm lỗi — được không?"

**Worker được chọn:** `policy_tool_worker`  
**Route reason (từ trace):** `Task liên quan đến chính sách/điều khoản → điều hướng policy_tool_worker | có thể cần MCP search_kb/get_ticket_info nếu context nội bộ chưa đủ`  
**MCP tools được gọi:** search_kb  
**Workers called sequence:** policy_tool_worker → synthesis_worker

**Kết quả thực tế:**
- final_answer (ngắn): Không được hoàn tiền do Flash Sale là ngoại lệ của policy
- confidence: 0.3
- Correct routing? Yes

**Nhận xét:**

Routing đúng vì câu hỏi thuộc refund policy và có ngoại lệ rõ. Confidence không cao vì chunks lấy về không thật sự bám đúng refund docs, nhưng policy worker vẫn phát hiện đúng exception `flash_sale_exception`. 

---

## Routing Decision #4 (bonus)

**Task đầu vào:**
> "Cần cấp quyền Level 3 để khắc phục P1 khẩn cấp. Quy trình là gì?"

**Worker được chọn:** `policy_tool_worker`  
**Route reason:** `Task liên quan đến chính sách/điều khoản → điều hướng policy_tool_worker | có yếu tố emergency → cần HITL sau policy check và có thể gọi MCP check_access_permission/get_ticket_info | human approved → retrieval`  

**Nhận xét: Đây là trường hợp routing khó nhất trong lab. Tại sao?**

Đây là case khó nhất vì vừa có policy, vừa có yếu tố emergency, vừa phải gọi nhiều MCP tools (`search_kb`, `check_access_permission`, `get_ticket_info`), lại còn kích hoạt HITL. Routing đúng nhưng confidence chỉ 0.3 vì answer vẫn thiếu dữ liệu đặc thù cho đường ưu tiên khẩn cấp.
_________________

---

## Tổng kết

### Routing Distribution

| Worker | Số câu được route | % tổng |
|--------|------------------|--------|
| retrieval_worker | 1 | 25% |
| policy_tool_worker | 3 | 75% |
| human_review | 1 | 25% |

### Routing Accuracy

> Trong số 4 câu nhóm đã chạy, bao nhiêu câu supervisor route đúng?

- Câu route đúng: 4 / 4  
- Câu route sai (đã sửa bằng cách nào?): 0  
- Câu trigger HITL: 2 

### Lesson Learned về Routing

> Quyết định kỹ thuật quan trọng nhất nhóm đưa ra về routing logic là gì?  
> (VD: dùng keyword matching vs LLM classifier, threshold confidence cho HITL, v.v.)

1. Keyword-based routing hoạt động tốt với domain rõ ràng (IT, policy), nhưng cần kết hợp thêm signal như risk/emergency để xử lý các case phức tạp (ví dụ P1 + cấp quyền).  
2. Routing đúng chưa đảm bảo answer đúng — chất lượng retrieval (chunks, indexing) ảnh hưởng lớn đến final_answer và confidence.

### Route Reason Quality

> Nhìn lại các `route_reason` trong trace — chúng có đủ thông tin để debug không?  
> Nếu chưa, nhóm sẽ cải tiến format route_reason thế nào?

Route_reason hiện tại khá rõ ràng và đủ để debug, vì đã mô tả loại task (IT/policy) và hướng xử lý (retrieval hay policy_tool). Tuy nhiên, để cải thiện hơn, nhóm có thể bổ sung thêm thông tin như keyword match cụ thể hoặc confidence/score để giải thích chi tiết hơn vì sao Supervisor chọn route đó.
_________________
