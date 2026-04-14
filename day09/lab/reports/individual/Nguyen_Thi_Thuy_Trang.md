# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Nguyễn Thị Thùy Trang
**Vai trò trong nhóm:** Documentation Owner  
**Ngày nộp:** 14/04/2026  
**Độ dài yêu cầu:** 500–800 từ

---

> **Lưu ý quan trọng:**
> - Viết ở ngôi **"tôi"**, gắn với chi tiết thật của phần bạn làm
> - Phải có **bằng chứng cụ thể**: tên file, đoạn code, kết quả trace, hoặc commit
> - Nội dung phân tích phải khác hoàn toàn với các thành viên trong nhóm
> - Deadline: Được commit **sau 18:00** (xem SCORING.md)
> - Lưu file với tên: `reports/individual/[ten_ban].md` (VD: `nguyen_van_a.md`)

---

## 1. Tôi phụ trách phần nào? (100–150 từ)

> Mô tả cụ thể module, worker, contract, hoặc phần trace bạn trực tiếp làm.
> Không chỉ nói "tôi làm Sprint X" — nói rõ file nào, function nào, quyết định nào.

**Module/file tôi chịu trách nhiệm:**
- File chính: `system_architecture.md`, `routing_decisions.md`
- Functions tôi implement: Tôi không trực tiếp implement code; phụ trách phân tích và mô tả `supervisor_node`, pipeline và routing logic dựa trên code nhóm

**Cách công việc của tôi kết nối với phần của thành viên khác:**

Tôi chịu trách nhiệm tổng hợp và chuẩn hóa kiến trúc hệ thống từ code của nhóm thành tài liệu rõ ràng, bao gồm pipeline, vai trò từng worker và schema của AgentState. Ngoài ra, tôi phân tích trace từ hệ thống để ghi lại các quyết định routing thực tế, giúp nhóm hiểu rõ cách Supervisor hoạt động và phát hiện lỗi routing. Công việc của tôi giúp các thành viên khác debug dễ hơn và hiểu rõ luồng xử lý toàn hệ thống.

**Bằng chứng (commit hash, file có comment tên bạn, v.v.):**

File `system_architecture.md` và `routing_decisions.md` được tôi hoàn thiện dựa trên code trong `graph.py` và các worker, cùng với trace được lưu trong `artifacts/traces/`.

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

> Chọn **1 quyết định** bạn trực tiếp đề xuất hoặc implement trong phần mình phụ trách.
> Giải thích:
> - Quyết định là gì?
> - Các lựa chọn thay thế là gì?
> - Tại sao bạn chọn cách này?
> - Bằng chứng từ code/trace cho thấy quyết định này có effect gì?

**Quyết định:** Chọn 4 task đại diện từ bộ 10 test questions để đưa vào routing_decisions.md thay vì liệt kê toàn bộ

**Lý do:**

Trong test set có tổng cộng 10 task với nhiều mức độ khó và category khác nhau (SLA, refund, access control, HR policy). Tuy nhiên khi chạy thực tế, một số task có hành vi tương tự nhau hoặc cho kết quả trùng lặp. Vì vậy, tôi quyết định chọn ra 4 task đại diện cho các tình huống quan trọng nhất của hệ thống, bao gồm: (1) retrieval thành công (SLA), (2) policy với exception (refund), (3) case phức tạp có HITL và MCP tools (access control), và (4) case routing đúng nhưng answer chưa tốt (HR policy).

So với việc liệt kê toàn bộ 10 task, cách này giúp report ngắn gọn hơn nhưng vẫn thể hiện đầy đủ hành vi của hệ thống. Đồng thời, việc chọn các case có insight rõ ràng giúp phân tích dễ hiểu và có giá trị hơn.

**Trade-off đã chấp nhận:**

Việc không đưa toàn bộ 10 task vào report có thể bỏ sót một số edge case, nhưng đổi lại giúp tránh trùng lặp và tập trung vào các tình huống tiêu biểu.

**Bằng chứng từ trace/code:**
```
Remote probation:
route=policy_tool_worker
reason=Task liên quan policy → policy_tool_worker
confidence=0.48

SLA P1:
route=retrieval_worker
reason=Task hỏi về hệ thống IT/SLA → retrieval_worker
confidence=0.46

Flash Sale refund:
route=policy_tool_worker
reason=Task liên quan đến chính sách/điều khoản → policy_tool_worker
confidence=0.3

Level 3 access (P1):
route=policy_tool_worker
reason=Task liên quan đến chính sách/điều khoản + emergency → cần HITL
hitl_triggered=True
mcp_tools_used=[search_kb, check_access_permission, get_ticket_info]
```

---

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

> Mô tả 1 bug thực tế bạn gặp và sửa được trong lab hôm nay.
> Phải có: mô tả lỗi, symptom, root cause, cách sửa, và bằng chứng trước/sau.

**Lỗi:** Đánh giá sai chất lượng hệ thống khi không phân biệt giữa routing correctness và answer correctness

**Symptom (pipeline làm gì sai?):**

Ban đầu khi phân tích trace, tôi thấy một số câu trả lời có confidence thấp hoặc nội dung chưa đầy đủ (ví dụ Flash Sale refund hoặc Level 3 access), nên có xu hướng đánh giá hệ thống routing chưa tốt.

**Root cause (lỗi nằm ở đâu — indexing, routing, contract, worker logic?):**

Vấn đề không nằm ở routing mà ở cách phân tích. Tôi chỉ nhìn vào final_answer mà chưa kiểm tra đầy đủ route_reason, workers_called và confidence trong trace. Điều này dẫn đến việc nhầm lẫn giữa lỗi routing và lỗi retrieval hoặc policy analysis.

**Cách sửa:**

Tôi điều chỉnh cách đọc trace bằng cách luôn tách riêng hai yếu tố:
- routing đúng/sai (dựa vào supervisor_route và expected behavior)
- answer đúng/sai (dựa vào final_answer và confidence)

Nhờ đó, tôi xác định được rằng trong các case đã chọn, routing đều đúng nhưng chất lượng answer phụ thuộc vào retrieval và policy logic.

**Bằng chứng trước/sau:**
> Dán trace/log/output trước khi sửa và sau khi sửa.

Trước:

- confidence thấp → kết luận hệ thống routing kém

Sau:
- route=retrieval_worker / policy_tool_worker (đúng)
- confidence thấp → do retrieval hoặc context chưa tốt, không phải lỗi routing
_________________

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

> Trả lời trung thực — không phải để khen ngợi bản thân.

**Tôi làm tốt nhất ở điểm nào?**

Tôi làm tốt ở việc phân tích hệ thống và chuyển đổi logic code thành tài liệu rõ ràng, giúp cả nhóm hiểu pipeline và cách các thành phần tương tác với nhau.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**

Tôi chưa tham gia trực tiếp vào phần code nên mức độ hiểu sâu về implementation chi tiết của từng worker còn hạn chế.

**Nhóm phụ thuộc vào tôi ở đâu?** _(Phần nào của hệ thống bị block nếu tôi chưa xong?)_

Nhóm phụ thuộc vào tôi trong việc tổng hợp kiến trúc hệ thống và phân tích routing, đặc biệt khi cần debug hoặc trình bày hệ thống.

**Phần tôi phụ thuộc vào thành viên khác:** _(Tôi cần gì từ ai để tiếp tục được?)_

Tôi phụ thuộc vào code từ các thành viên khác (graph, workers) và trace output để có dữ liệu chính xác phục vụ việc phân tích và viết tài liệu.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

> Nêu **đúng 1 cải tiến** với lý do có bằng chứng từ trace hoặc scorecard.
> Không phải "làm tốt hơn chung chung" — phải là:
> *"Tôi sẽ thử X vì trace của câu gq___ cho thấy Y."*

_________________

---
