# Báo Cáo Cá Nhân — Lab Day 09: Multi-Agent Orchestration

**Họ và tên:** Nguyễn Hoàng Nghĩa  
**Vai trò trong nhóm:** Worker Owner  
**Ngày nộp:** 14/04/2026  

---

## 1. Tôi phụ trách phần nào?

Trong lab này, tôi chịu trách nhiệm chính cho worker retrieval, cụ thể là file `workers/retrieval.py`. Nhiệm vụ của tôi là implement cơ chế truy vấn dữ liệu từ ChromaDB để phục vụ cho các worker downstream như policy_tool.  

Các function tôi trực tiếp implement gồm:
- `_get_embedding_fn()` để tạo embedding từ query  
- `_get_collection()` để kết nối tới ChromaDB  
- `retrieve_dense()` để thực hiện dense retrieval  
- `run()` để tích hợp vào AgentState pipeline  

Worker của tôi là bước đầu tiên trong pipeline, nên output của nó (`retrieved_chunks`, `retrieved_sources`) sẽ được truyền sang policy_tool_worker để phân tích policy. Nếu retrieval sai hoặc trả về rỗng, toàn bộ pipeline phía sau sẽ bị ảnh hưởng.  

Bằng chứng: commit 'Update retrieval.py' trên github

---

## 2. Tôi đã ra một quyết định kỹ thuật gì?

**Quyết định:** Tôi chọn sử dụng Sentence Transformers (`all-MiniLM-L6-v2`) cho embedding thay vì gọi API bên ngoài.

**Lý do:**  
Có hai lựa chọn:
1. Dùng OpenAI embedding API  
2. Dùng Sentence Transformers local  

Tôi chọn cách 2 vì:
- Không cần API key → dễ setup cho toàn nhóm  
- Latency thấp hơn (local inference)  
- Phù hợp với lab environment (không yêu cầu production scale)  


**Trade-off đã chấp nhận:**
- Model local có thể kém chính xác hơn OpenAI embedding  
- Cần tải model lần đầu  

**Bằng chứng từ code:**

```python
from sentence_transformers import SentenceTransformer
        model = SentenceTransformer("all-MiniLM-L6-v2")
        def embed(text: str) -> list:
            return model.encode([text])[0].tolist()
        return embed
```

## 3. Tôi đã sửa một lỗi gì? (150–200 từ)

> Mô tả 1 bug thực tế bạn gặp và sửa được trong lab hôm nay.
> Phải có: mô tả lỗi, symptom, root cause, cách sửa, và bằng chứng trước/sau.

**Lỗi:** Retrieval luôn trả về 0 chunks dù đã chạy pipeline đúng.

**Symptom (pipeline làm gì sai?):**

Khi chạy test:
Retrieved: 0 chunks
Sources: []
Pipeline downstream không có context để xử lý.

**Root cause (lỗi nằm ở đâu — indexing, routing, contract, worker logic?):**

Có 2 lỗi:
- Sai biến môi trường CHROMA_DB_PATH (có dấu cách → path sai)
- Mismatch collection name:
    + Query: "rag_lab"
    + Create: "day09_docs"
→ Dẫn đến query vào collection không tồn tại hoặc DB rỗng.


**Cách sửa:**

- Sửa .env: CHROMA_DB_PATH=./Day_08/day08/lab/chroma_db
- Thống nhất collection name: COLLECTION_NAME = "rag_lab"
- Thêm debug log để kiểm tra:
```
print("DB_PATH:", DB_PATH)
print("Collections:", client.list_collections())
print("Count:", col.count())
```

**Bằng chứng trước/sau:**
> Dán trace/log/output trước khi sửa và sau khi sửa.

Trước: 
⚠️ Collection chưa tồn tại hoặc chưa index data
Retrieved: 0 chunks

Sau:
DB_PATH: /Users/nghia/.../chroma_db
Collections: ['rag_lab']
Documents in collection: 120
Retrieved: 3 chunks
---

## 4. Tôi tự đánh giá đóng góp của mình (100–150 từ)

> Trả lời trung thực — không phải để khen ngợi bản thân.

**Tôi làm tốt nhất ở điểm nào?**

Tôi debug khá nhanh các lỗi liên quan đến ChromaDB (path, collection, data), và cả file policy_tool. Đây là phần quan trọng vì retrieval là bước đầu pipeline, nếu sai thì toàn bộ hệ thống không hoạt động.


**Tôi làm chưa tốt hoặc còn yếu ở điểm nào?**

Ban đầu tôi chưa chú ý đến việc load .env và xử lý path (relative vs absolute), dẫn đến mất thời gian debug lỗi không cần thiết.

**Nhóm phụ thuộc vào tôi ở đâu?** _(Phần nào của hệ thống bị block nếu tôi chưa xong?)_

Các worker khác (đặc biệt policy_tool) phụ thuộc hoàn toàn vào output retrieval. Nếu retrieval sai, các worker sau không thể hoạt động đúng. Hơn nữa tôi còn fix bug duplicate ở file synthesis.


**Phần tôi phụ thuộc vào thành viên khác:** _(Tôi cần gì từ ai để tiếp tục được?)_

Tôi phụ thuộc vào phần indexing data (MCP hoặc ingestion script). Nếu dữ liệu chưa được index vào ChromaDB thì retrieval không thể trả kết quả.

---

## 5. Nếu có thêm 2 giờ, tôi sẽ làm gì? (50–100 từ)

Tôi sẽ thêm reranker (cross-encoder) sau bước retrieval.
Lý do: một số query trả về chunk chưa đúng thứ tự relevance. Trace cho thấy top-1 đôi khi không phải chunk tốt nhất.
Tôi sẽ thử dùng model reranker để cải thiện ranking, giúp tăng độ chính xác cho policy analysis phía sau.

_________________

---

