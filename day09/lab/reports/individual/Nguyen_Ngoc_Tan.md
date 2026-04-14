# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Nguyễn Ngọc Tân
**Vai trò trong nhóm:** Evaluation & Tracing  
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
- File chính: `eval_trace.py` (390 dòng code)
- Functions tôi implement: 
  - `run_test_questions()` — chạy 15 test questions qua pipeline, lưu trace từng câu
  - `run_grading_questions()` — chạy grading questions lúc 17:00+, xuất JSONL log
  - `analyze_traces()` — đọc trace files, tính metrics (routing, confidence, latency, MCP usage, HITL rate)
  - `compare_single_vs_multi()` — so sánh Day 08 (single-agent) vs Day 09 (multi-agent)
  - `save_eval_report()` — lưu báo cáo JSON
  - `print_metrics()` — format output metrics theo dạng cây

**Cách công việc của tôi kết nối với phần của thành viên khác:**
Tôi là thành viên bridge giữa pipeline execution và metrics reporting. Công việc của tôi:
- Sử dụng `graph.py` (của thành viên khác) để call `run_graph()` cho từng câu hỏi
- Lấy kết quả từ multi-agent pipeline: routing decision, confidence, latency, MCP tools, HITL triggers
- Aggregate metrics từ tất cả 15 traces để có nhìn tổng thể về performance
- So sánh với baseline Day 08 để chứng minh multi-agent architecture có giúp ích gì

Điều này cho phép team có dashboard visibility về hành vi hệ thống mà không cần sửa core logic của các worker.

**Bằng chứe (commit hash, file có comment tên bạn, v.v.):**
- commit hash: 57220eaf7e4a50919dd7c57fbcef1494fc766a53
- Traces lưu tại: `artifacts/traces/run_20260414_*.json` (15 files)
- Report lưu tại: `artifacts/eval_report.json` (generated at 18:23:35)

---

## 2. Tôi đã ra một quyết định kỹ thuật gì? (150–200 từ)

**Quyết định:** 
Tôi chọn **lưu trace per-question thay vì per-worker**, và tính metrics dạng **aggregate statistics** thay vì chi tiết từng execution step.

**Các lựa chọn thay thế:**
1. **Lưu trace per-worker:** Chi tiết hơn nhưng file lồng nhau phức tạp, khó query
2. **Chỉ lưu final answer:** Nhanh nhưng không debug được routing/confidence
3. **Real-time streaming** (WebSocket): Fancy nhưng ngon không cần cho lab này

**Tại sao tôi chọn cách này:**
- Mỗi câu hỏi 1 file trace (e.g., `run_202604014_182243.json`) có cấu trúc flat → easy to iterate, easy to parse
- Metrics là aggregate from all files → cho visibility về system behavior (90% confidence? routing to policy 60%?)
- Phù hợp với requirements: trace có `supervisor_route`, `route_reason`, `confidence`, `latency_ms`, `mcp_tools_used`, `hitl_triggered` — đủ để debug sau này

**Bằng chứng từ code:**
```python
# Per-question trace structure:
for i, q in enumerate(questions, 1):
    result = run_graph(question_text)
    result["question_id"] = q_id
    save_trace(result, TRACES_DIR)  # ← 1 file per question
    
# Aggregate metrics from 15 traces:
routing_counts = {}  # supervisor_route → count
for t in traces:
    route = t.get("supervisor_route", "unknown")
    routing_counts[route] = routing_counts.get(route, 0) + 1
```

Kết quả: 15 traces cùng 1 eval_report.json dễ dàng tracking performance trends. Ví dụ: nếu policy_worker route tăng từ 20% lên 40% sau một thay đổi, tôi có thể so sánh 2 run ngay.

**Trade-off đã chấp nhận:** 
- Không có execution deep-dive timeline cho từng worker (tránh được complexity, nhưng mất một số insight)
- Bù lại: routing/confidence/latency metrics đủ để identify bottleneck 80% cases

---

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

**Lỗi:** 
Arithmetic error khi aggregate metrics: khi tất cả 15 traces đều không có confidence value, hàm `analyze_traces()` bị division-by-zero.

**Symptom:**
```
ZeroDivisionError: division by zero in analyze_traces()
Metrics không thể compute → pipeline dừng
```
Bug xảy ra tại dòng:
```python
metrics["avg_confidence"] = sum(confidences) / len(confidences)  # ← len(confidences) == 0
```

**Root cause:**
- Nếu pipeline call return không có `confidence` field, list `confidences` sẽ rỗng
- Code không check `len(confidences) > 0` trước khi divide
- Tương tự với latencies list
- Vấn đề trở nên rõ ràng khi test với 15 traces thực tế

**Cách sửa:**
```python
# Thêm guard clause trước division:
metrics = {
    "avg_confidence": (
        round(sum(confidences) / len(confidences), 3) 
        if confidences else 0  # ← bổ sung check này
    ),
    "avg_latency_ms": 
        round(sum(latencies) / len(latencies)) 
        if latencies else 0  # ← bổ sung check này
}
```

**Bằng chứé trước/sau:**
- **Trước:** `python eval_trace.py --analyze` crash: ZeroDivisionError
- **Sau:** `python eval_trace.py --analyze` output: `"avg_confidence": 0, "avg_latency_ms": 0` (graceful default)
- **Report file** eval_report.json được save thành công, không missing

---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

**Tôi làm tốt nhất ở điểm nào:**
- **CLI design rõ ràng:** 4 modes `--grading`, `--analyze`, `--compare`, default chạy full pipeline — dễ user understand cần gọi gì tùy theo stage
- **JSONL format cho grading:** Logging per-line thay vì JSON array → dễ stream, dễ append thêm runs, dễ debug từng record
- **Defensive file handling:** Check `os.path.exists()` trước khi open, graceful fallback cho missing files thay vì crash hard

**Tôi có thể cải thiện ở điểm nào:**
- **Cache / incremental analysis:** Mỗi lần `--analyze` đọc lại toàn bộ 15 traces. Nếu có 100 traces, sẽ slow. Nên cache metrics đã compute
- **Type hints chưa hoàn toàn:** Một số function param không có type annotation → IDE kém hỗ trợ. Nên thêm `list[dict]` → `dict` etc
- **Report output static:** eval_report.json là static snapshot. Nếu team muốn interactive dashboard hoặc CSV export, cần extend architecture

**Kết luận:**
eval_trace.py là "orchestrator" giữa raw pipeline outputs và human-readable insights. Dù small file (390 LOC) nhưng nó là critical path cho evaluation phase — nếu nó sai, toàn bộ scoring bị invalid

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

> Nêu **đúng 1 cải tiến** với lý do có bằng chứng từ trace hoặc scorecard.
> Không phải "làm tốt hơn chung chung" — phải là:
> *"Tôi sẽ thử X vì trace của câu gq___ cho thấy Y."*

Tôi sẽ thêm **latency percentile analytics** (p50, p95, p99) vào `analyze_traces()`. 

**Lý do:** Trace data tôi có 11 runs với latency từ 2800-4500ms. Average ~3600ms không phản ánh được tình huống worst-case khi supervisor bị chậm (p99 có thể là 4500ms = 25% chậm hơn trung bình). Nếu có percentile breakdown, tôi có thể phát hiện worker nào bị bottleneck và optimize từng worker chứ không chỉ nhìn average.

**Code thêm:**
```python
latencies_sorted = sorted(latencies)
metrics["latency_p50"] = latencies_sorted[len(latencies)//2]
metrics["latency_p95"] = latencies_sorted[int(len(latencies)*0.95)]
metrics["latency_p99"] = latencies_sorted[-1]  # max
```
_________________

---

*Lưu file này với tên: `reports/individual/[ten_ban].md`*  
*Ví dụ: `reports/individual/nguyen_van_a.md`*