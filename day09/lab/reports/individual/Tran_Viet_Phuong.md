# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Trần Việt Phương
**Vai trò trong nhóm:** Worker Owner
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

**Module/file tôi chịu trách nhiệm:**
- File chính: `workers/policy_tool.py`, `workers/synthesis.py`
- Functions tôi implement: policy_tool.py: _call_mcp_tool, analyze_policy; systhesis.py: synthesize, estimate_confidence.

**Cách công việc của tôi kết nối với phần của thành viên khác:**
Trong quá trình làm việc, tôi đã kết nối với các thành viên khác thông qua việc sử dụng các file mà họ đã tạo ra. Cụ thể, tôi đã sử dụng file `graph.py` để kết nối các worker với nhau và sử dụng file `contracts/worker_contracts.yaml` để định nghĩa các worker. Các worker hoạt động tuần tự, mỗi worker sẽ thực hiện một nhiệm vụ cụ thể, kết hợp cùng supervisor và MCP tools để tạo thành 1 hệ thống multi-agent hoàn chỉnh

_________________

**Bằng chứng (commit hash, file có comment tên bạn, v.v.):**
commit hash: 57220eaf7e4a50919dd7c57fbcef1494fc766a53
_________________

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

> Chọn **1 quyết định** bạn trực tiếp đề xuất hoặc implement trong phần mình phụ trách.
> Giải thích:
> - Quyết định là gì?
> - Các lựa chọn thay thế là gì?
> - Tại sao bạn chọn cách này?
> - Bằng chứng từ code/trace cho thấy quyết định này có effect gì?

**Quyết định:** Quyết định sử dụng LLM-based explaination trong policy_tool.py, thay vì chỉ sử dụng rule-based check. Đơn giản là thay vì nhìn vào kết quả và đối chiếu nhiều lần với policy và exceptions, tôi đưa cho gemini làm hộ và lấy kết quả từ nó. 

**Trade-off đã chấp nhận:** Tốn thêm 1 call LLM, bù lại làm việc nhanh hơn

_________________

**Bằng chứng từ trace/code:**

```
explanation = "Analyzed via rule-based policy check."
    gemini_key = os.getenv("GOOGLE_API_KEY") or os.getenv("GEMINI_API_KEY")
    if gemini_key:
        try:
            from google import genai

            client = genai.Client(api_key=gemini_key)
            prompt = (
                "Bạn là policy analyst. Dựa vào context, xác định policy áp dụng và các exceptions.\n\n"
                f"Task: {task}\n\nContext:\n"
                + "\n".join([c.get("text", "") for c in chunks])
            )
            response = client.models.generate_content(
                model="gemini-2.5-flash",
                contents=prompt,
            )
            analysis = response.text
            if analysis:
                explanation = analysis
        except Exception as e:
            explanation += f" (LLM analysis skipped: {e})"

```

---

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

> Mô tả 1 bug thực tế bạn gặp và sửa được trong lab hôm nay.
> Phải có: mô tả lỗi, symptom, root cause, cách sửa, và bằng chứng trước/sau.

**Lỗi:** LLM based explaination gặp lỗi

**Symptom (pipeline làm gì sai?):** Sử dụng code mock cho sẵn, code được cấu hình cho phiên bản gemini cũ hơn, chạy không được.

_________________

**Root cause (lỗi nằm ở đâu — indexing, routing, contract, worker logic?):** nằm ở phần llm cho sẵn, không tương thích với phiên bản gemini hiện tại.

_________________

**Cách sửa:** Vibe-code lại phần llm cho sẵn, sử dụng phiên bản gemini mới nhất.

_________________

**Bằng chứng trước/sau:**
> Dán trace/log/output trước khi sửa và sau khi sửa.

_________________

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

> Trả lời trung thực — không phải để khen ngợi bản thân.

**Tôi làm tốt nhất ở điểm nào?**

Tôi thấy bản thân làm việc ổn, các TODO hoàn thành nhanh. Gặp lỗi không bị hoảng, không cố gắng fix bừa mà đọc log và tìm hiểu nguyên nhân.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**

Giao tiếp với các thành viên khác cần cải thiện, đặc biệt học thêm cách dùng github

**Nhóm phụ thuộc vào tôi ở đâu?** Sprint 3+4

_________________

**Phần tôi phụ thuộc vào thành viên khác:** Sprint 1

_________________

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

> Nêu **đúng 1 cải tiến** với lý do có bằng chứng từ trace hoặc scorecard.
> Không phải "làm tốt hơn chung chung" — phải là:
> *"Tôi sẽ thử X vì trace của câu gq___ cho thấy Y."*

Tôi sẽ thử sử dụng đề xuất llm embedded thay vì dùng local model. Dù local model đã đủ tốt nhưng với bài toán to hơn, local model sẽ không đủ mạnh.
_________________

---

*Lưu file này với tên: `reports/individual/[ten_ban].md`*  
*Ví dụ: `reports/individual/nguyen_van_a.md`*
