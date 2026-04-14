# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Trịnh Uyên Chi  
**Vai trò trong nhóm:** Supervisor Owner
**Ngày nộp:** 14/04/2026
**Độ dài yêu cầu:** 500–800 từ

## 1. Tôi phụ trách phần nào? (100–150 từ)

**Module/file tôi chịu trách nhiệm:**
- File chính: `graph.py`
- Functions tôi implement: `supervisor_node()`, `build_graph()`, `run_graph()`

**Cách công việc của tôi kết nối với phần của thành viên khác:**

Phần `graph.py` đóng vai trò là điều phối trung tâm của toàn bộ hệ thống Multi-Agent. Khi nhận câu hỏi từ user, node của tôi sẽ khởi tạo và cập nhật AgentState, sau đó dùng conditional_edges để điều hướng task đến đúng Worker của các thành viên khác. Nếu tôi phân luồng sai, các Worker sẽ nhận dữ liệu đầu vào rác. Cuối cùng, `graph.py` đảm bảo gom toàn bộ output từ các Worker để đẩy về synthesis_worker tổng hợp thành câu trả lời hoàn chỉnh.

**Bằng chứng (commit hash, file có comment tên bạn, v.v.):**
* Commit Hash: 
    * c52405f5b8a3b33bfb1266b1111a76cd5e12b851
    * 388d67ca6413224face22754d0e79adfb62cdbc6

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

**Quyết định:** Chuyển đổi kiến trúc Orchestrator từ hàm Python thuần (dùng lệnh if/elif lồng nhau) sang sử dụng thư viện LangGraph (StateGraph).

**Lý do:** Việc phân luồng với 5 nhánh (P1, lỗi lạ, hoàn tiền, cấp quyền, mặc định) khiến hàm điều phối thủ công trở nên dài dòng và khó kiểm soát trạng thái. LangGraph giúp quản lý luồng dữ liệu (AgentState) xuyên suốt một cách tường minh, giúp dễ dàng mở rộng, vẽ sơ đồ trực quan (visualize), và thiết yếu nhất là hỗ trợ luồng ngắt quãng (interrupt) khi tích hợp cơ chế Human-in-the-loop (chờ con người duyệt mã lỗi khẩn cấp).

**Trade-off đã chấp nhận:** Tăng độ phức tạp của code khởi tạo. Thay vì chạy code từ trên xuống dưới, phải tư duy theo dạng đồ thị có hướng, quản lý chính xác các conditional_edges và chấp nhận việc thay đổi cách gọi hàm (phải qua phương thức .invoke()).

**Bằng chứng từ trace/code:**

```
def build_graph():
    """
    Xây dựng graph bằng LangGraph.
    """
    # 1. Khởi tạo đồ thị với cấu trúc State của chúng ta
    workflow = StateGraph(AgentState)

    # 2. Khai báo các Nodes (Các hàm xử lý)
    workflow.add_node("supervisor", supervisor_node)
    workflow.add_node("retrieval_worker", retrieval_worker_node)
    workflow.add_node("policy_tool_worker", policy_tool_worker_node)
    workflow.add_node("human_review", human_review_node)
    workflow.add_node("synthesis_worker", synthesis_worker_node)

    # 3. Định nghĩa Edges (Luồng chảy của data)
    
    # Điểm bắt đầu luôn đi vào Supervisor
    workflow.add_edge(START, "supervisor")

    # Supervisor chia nhánh dựa trên hàm route_decision
    workflow.add_conditional_edges(
        "supervisor",
        route_decision, 
        {
            "retrieval_worker": "retrieval_worker",
            "policy_tool_worker": "policy_tool_worker",
            "human_review": "human_review"
        }
    )

    # Nếu vào Human Review, sau khi duyệt xong thì đi tới Retrieval
    workflow.add_edge("human_review", "retrieval_worker")

    # Nếu vào Policy Tool, dùng hàm phụ để xét xem có cần sang Retrieval không
    workflow.add_conditional_edges(
        "policy_tool_worker",
        post_policy_route,
        {
            "retrieval_worker": "retrieval_worker",
            "synthesis_worker": "synthesis_worker"
        }
    )

    # Retrieval xong thì luôn gom lại về Synthesis
    workflow.add_edge("retrieval_worker", "synthesis_worker")

    # Synthesis xong là Kết thúc (END)
    workflow.add_edge("synthesis_worker", END)

    # 4. Compile đồ thị thành một ứng dụng chạy được
    app = workflow.compile()
    return app
```


## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

**Lỗi:** `TypeError: 'CompiledStateGraph' object is not callable` khi chạy thử hệ thống.

**Symptom (pipeline làm gì sai?):**

Khi chạy các câu hỏi test trong terminal, chương trình bị crash ngay lập tức ở hàm run_graph, pipeline không thể khởi động để đưa task vào đồ thị phân luồng.

**Root cause (lỗi nằm ở đâu — indexing, routing, contract, worker logic?):**

Lỗi nằm ở Orchestrator execution. Trước đây hàm build_graph() trả về một function nên có thể gọi trực tiếp bằng ngoặc kép _graph(state). Tuy nhiên, khi tích hợp LangGraph, hàm workflow.compile() trả về một đối tượng đồ thị (CompiledStateGraph). Việc cố gắng gọi đối tượng này như một hàm đã gây ra ngoại lệ.

**Cách sửa:**
Thay đổi cách khởi chạy đồ thị trong API run_graph bằng cách gọi phương thức .invoke() của đối tượng LangGraph, truyền vào state ban đầu.

**Bằng chứng trước/sau:**

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

**Tôi làm tốt nhất ở điểm nào?**

Xây dựng logic phân luồng (routing) của Supervisor gọn gàng, chia chính xác các nhóm từ khóa. Tích hợp thành công cấu trúc LangGraph giúp hệ thống có nền tảng vững chắc để nhóm mở rộng thêm các Worker mới sau này.

**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**

Đôi khi còn nhầm lẫn giữa cú pháp của Python thuần và cấu trúc đặc thù của framework mới, dẫn đến các lỗi runtime nhỏ (như quên cách gọi `.invoke()`)

**Nhóm phụ thuộc vào tôi ở đâu?**

Graph là xương sống của hệ thống đa tác vụ. Nếu luồng Supervisor của tôi phân loại sai task, hoặc đồ thị đứt gãy, các Node Worker sẽ không nhận được dữ liệu đầu vào. Pipeline bị nghẽn hoàn toàn.

**Phần tôi phụ thuộc vào thành viên khác:**

Tôi cần các bạn phụ trách Workers đảm bảo các hàm run(state) trả về đúng format đã thỏa thuận trong AgentState. Nếu Worker thiếu các key quan trọng như retrieved_chunks hay policy_result, node Synthesis cuối cùng của tôi sẽ không có gì để tổng hợp câu trả lời.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

Tôi sẽ nâng cấp hàm supervisor_node bằng cách kết hợp Semantic Router (dùng LLM hoặc Nhúng - Embedding) thay vì chỉ dùng keyword matching cứng cứng nhắc hiện tại. Tôi nhận thấy nếu người dùng nhập sai chính tả hoặc dùng từ đồng nghĩa (ví dụ: gõ "xin nghỉ đẻ" thay vì "thai sản"), từ khóa sẽ không khớp, dẫn đến route_reason trả về "mặc định retrieval_worker" thay vì đi đúng vào policy_tool_worker. Semantic routing sẽ giải quyết triệt để điểm yếu này.

---
