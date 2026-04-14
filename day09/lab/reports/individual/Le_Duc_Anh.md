# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Lê Đức Anh
**Vai trò trong nhóm:**  MCP Owner
**Ngày nộp:** 14/-4/2026  
**Độ dài yêu cầu:** 500–800 từ

---

---

## 1. Tôi phụ trách phần nào? (100–150 từ)

> Mô tả cụ thể module, worker, contract, hoặc phần trace bạn trực tiếp làm.
> Không chỉ nói "tôi làm Sprint X" — nói rõ file nào, function nào, quyết định nào.

Trong Day 09, tôi phụ trách phần MCP theo hướng Standard (mock server chạy trong-process), tập trung vào việc biến policy worker từ gọi cứng sang gọi tool có contract rõ ràng. Cụ thể, tôi làm chính ở file `mcp_server.py` và phần tích hợp ở `workers/policy_tool.py`. Ở `mcp_server.py`, tôi định nghĩa tool schemas, registry, dispatcher, và implement 4 tools gồm `search_kb`, `get_ticket_info`, `check_access_permission`, `create_ticket` (trong đó 2 tool bắt buộc của đề là `search_kb` và `get_ticket_info`). Ở policy worker, tôi thêm lớp gọi `_call_mcp_tool(...)`, ghi trace vào `mcp_tools_used`, `mcp_tool_called`, `mcp_result` và mở rộng theo ngữ cảnh câu hỏi để gọi đúng tool.

**Module/file tôi chịu trách nhiệm:**
- File chính: `day09/lab/mcp_server.py`, `day09/lab/workers/policy_tool.py`
- Functions tôi implement/chỉnh: `tool_search_kb`, `tool_get_ticket_info`, `dispatch_tool`, `list_tools`, `_call_mcp_tool`, phần MCP branch trong `run(state)`

**Cách công việc của tôi kết nối với phần của thành viên khác:**

- **Tôi phụ thuộc vào:** Retrieval Owner cung cấp `chunks` từ ChromaDB; nếu chunks không đủ (vd: confidence thấp), policy worker gọi MCP `search_kb` để mở rộng evidence. Supervisor Owner định tuyến bao nhiêu query đến policy_tool_worker dựa vào keyword analysis. Ticket system / Access control Owner cung cấp dữ liệu mock để `get_ticket_info()` và `check_access_permission()` trả lời.

- **Thành viên khác phụ thuộc vào tôi:** Supervisor/Graph Owner dựa vào output của `_call_mcp_tool()` (mcp_tools_used, mcp_result, mcp_tool_called) để ghi trace và debug routing logic. Synthesis Owner dùng evidence từ MCP call để tổng hợp câu trả lời cân bằng hơn (hỏi access → synthesis biết đã check permission; hỏi ticket → synthesis biết ticket status).

- **Connection point:** Policy worker (`workers/policy_tool.py`) là gateway: nhận `(task, chunks, state)` từ supervisor → nếu cần MCP thì gọi `_call_mcp_tool(tool_name, input)` từ mcp_server.py → append vào trace state → trả về synthesis. Điểm then chốt là `state["mcp_tools_used"]` và `state["mcp_result"]` phải được chuẩn hóa để Sprint 4 phân tích routing accuracy.

**Bằng chứng (commit hash, file có comment tên bạn, v.v.):**

`graph.py`
```
#Duc Anh

from workers.retrieval import run as retrieval_run
from workers.policy_tool import run as policy_tool_run
from workers.synthesis import run as synthesis_run

```

`policy_tool.py`
```
# Step 1: Nếu chưa có chunks, gọi MCP search_kb để mở rộng evidence
        if not chunks and needs_tool:
            mcp_result = _call_mcp_tool("search_kb", {"query": task, "top_k": 3})
            state["mcp_tools_used"].append(mcp_result)
            state["mcp_result"].append(mcp_result)
            state["mcp_tool_called"] = True
            state["history"].append(f"[{WORKER_NAME}] called MCP search_kb")

            if mcp_result.get("output") and mcp_result["output"].get("chunks"):
                chunks = mcp_result["output"]["chunks"]
                state["retrieved_chunks"] = chunks
                ...
```

`mcp_server.py`
```
def tool_search_kb(query: str, top_k: int = 3) -> dict:
    """
    Tìm kiếm Knowledge Base bằng semantic search.

    TODO Sprint 3: Kết nối với ChromaDB thực.
    Hiện tại: Delegate sang retrieval worker.
    """
    if not query or not str(query).strip():
        return {
            "chunks": [],
            "sources": [],
            "total_found": 0,
        }

    try:
        # Tái dùng retrieval logic từ workers/retrieval.py
        import sys
        sys.path.insert(0, os.path.dirname(__file__))
        from workers.retrieval import retrieve_dense
        chunks = retrieve_dense(query, top_k=top_k)
        sources = list({c["source"] for c in chunks})
        return {
            "chunks": chunks,
            "sources": sources,
            "total_found": len(chunks),
        }
    except Exception as e:
        # Fallback: return mock data nếu ChromaDB chưa setup
        return {
            "chunks": [
                {
                    "text": f"[MOCK] Không thể query ChromaDB: {e}. Kết quả giả lập.",
                    "source": "mock_data",
                    "score": 0.5,
                }
            ],
            "sources": ["mock_data"],
            "total_found": 1,
        }
    ...
```

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)


**Quyết định:** Tôi chọn kiến trúc “tool registry + dispatcher” cho MCP mock thay vì để policy worker import trực tiếp từng hàm tool.

**Lý do:**

Tôi cân nhắc 2 hướng: (1) policy worker gọi trực tiếp từng function như `tool_search_kb(...)`, `tool_get_ticket_info(...)`; (2) policy worker chỉ gọi `dispatch_tool(tool_name, tool_input)`. Tôi chọn hướng (2) vì phù hợp tinh thần MCP: worker không cần biết implementation backend, chỉ biết tên tool + schema input. Cách này giúp mở rộng nhanh: thêm tool mới chỉ cần đăng ký vào `TOOL_REGISTRY`, không phải sửa logic orchestration nhiều nơi. Ngoài ra, nó giúp trace thống nhất vì output `_call_mcp_tool` luôn cùng format `tool/input/output/error/timestamp`, thuận tiện phân tích Sprint 4.

**Trade-off đã chấp nhận:**

Đổi lại, mock MCP chưa phải server HTTP thật nên vẫn còn coupling trong cùng process. Đồng thời, với lớp dispatcher, việc debug sâu logic từng tool cần xem thêm 1 tầng trung gian.

**Bằng chứng từ trace/code:**

```
# mcp_server.py
TOOL_REGISTRY = {
	"search_kb": tool_search_kb,
	"get_ticket_info": tool_get_ticket_info,
	"check_access_permission": tool_check_access_permission,
	"create_ticket": tool_create_ticket,
}

def dispatch_tool(tool_name: str, tool_input: dict) -> dict:
	tool_fn = TOOL_REGISTRY[tool_name]
	result = tool_fn(**tool_input)
	return result

# trace run_20260414_182332.json
"mcp_tools_used": ["search_kb", "check_access_permission", "get_ticket_info"]
"mcp_tool_called": true
```

---

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

**Lỗi:** Pipeline chạy được nhưng mất bằng chứng MCP trong trace hoặc chỉ thấy một phần tool call, gây khó audit.

**Symptom (pipeline làm gì sai?):**

Trước khi tôi chuẩn hóa phần logging, nhóm gặp tình trạng khó đối chiếu vì không chắc policy worker đã gọi những tool nào theo thứ tự nào. Điều này làm việc debug các câu multi-hop (vừa access vừa SLA) tốn thời gian vì phải đọc log console thay vì đọc trace JSON.

**Root cause (lỗi nằm ở đâu — indexing, routing, contract, worker logic?):**

Root cause nằm ở worker logic của `policy_tool.py`: trace field cho MCP chưa đồng bộ (không luôn append đầy đủ cho mọi nhánh), và chưa có cờ tổng quát để biết run đó có gọi MCP hay không.

**Cách sửa:**

Tôi thêm chuẩn trace ở `run(state)`: `state.setdefault("mcp_tools_used", [])`, `state.setdefault("mcp_result", [])`, `state.setdefault("mcp_tool_called", False)`. Mỗi lần gọi `_call_mcp_tool` đều append cả vào `mcp_tools_used` lẫn `mcp_result`, đồng thời set `mcp_tool_called=True`. Tôi cũng bổ sung nhánh gọi `check_access_permission` cho câu hỏi access/emergency để đúng nhu cầu nghiệp vụ.

**Bằng chứng trước/sau:**

Trước: trace thiếu hoặc không nhất quán các mốc MCP.

Sau (trace `run_20260414_182332.json`):
- `mcp_tool_called: true`
- `mcp_tools_used`: có đủ `search_kb`, `check_access_permission`, `get_ticket_info`
- `history` có chuỗi `called MCP ...` theo đúng thứ tự thực thi.

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

> Trả lời trung thực — không phải để khen ngợi bản thân.

**Tôi làm tốt nhất ở điểm nào?**

Tôi làm tốt ở phần biến yêu cầu Sprint 3 thành một lớp MCP có thể mở rộng, có contract và trace rõ ràng. Đặc biệt, tôi ưu tiên tính quan sát được (observability) nên ngay từ tool call đã chuẩn hóa format để phục vụ Sprint 4.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**

Tôi chưa đẩy phần MCP lên mức Advanced (HTTP server thật). Ngoài ra, tôi còn phụ thuộc vào trạng thái ChromaDB và API quota nên một số run confidence vẫn thấp. Thêm vào đó, cần cải thiện khả năng tư duy, tự xử lý vấn đề để debug các lỗi tốt hơn.

**Nhóm phụ thuộc vào tôi ở đâu?**

Nếu tôi chưa xong MCP layer, policy worker sẽ khó xử lý câu hỏi multi-hop (policy + ticket + access), đồng thời trace không đủ dữ liệu để nhóm làm phân tích routing và báo cáo.

**Phần tôi phụ thuộc vào thành viên khác:** 

Tôi phụ thuộc vào teammate phụ trách retrieval/index để đảm bảo collection có dữ liệu đúng, và phụ thuộc teammate synthesis để tận dụng tốt evidence do MCP trả về.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

Tôi sẽ nâng MCP từ mock in-process lên HTTP server thật (FastAPI + endpoint chuẩn `tools/list` và `tools/call`) rồi benchmark lại latency và độ ổn định. Lý do là trace `run_20260414_182332.json` cho thấy luồng MCP đã có giá trị thực tế (3 tool calls trong 1 câu hỏi), nên bước tiếp theo hợp lý nhất là tách deployment boundary để giảm coupling và tiến gần kiến trúc production.

---