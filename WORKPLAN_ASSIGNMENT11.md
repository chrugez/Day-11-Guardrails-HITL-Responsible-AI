# Assignment 11 Workplan

Tài liệu này tổng hợp các việc cần làm từ:
- `README.md`
- `assignment11_defense_pipeline.md`

Mục tiêu là biến phần lab thành một pipeline phòng thủ hoàn chỉnh để nộp bài.

## 1. Hoàn thành nền tảng từ lab

Bạn cần hoàn thành các phần chính trong lab trước, vì đây là các khối xây dựng cho assignment:

1. Viết 5 adversarial prompts để tấn công agent không có guardrails.
2. Dùng AI sinh thêm attack prompts để red teaming tự động.
3. Implement phát hiện prompt injection bằng regex.
4. Implement topic filter để chặn câu hỏi off-topic hoặc nguy hiểm.
5. Tạo Input Guardrail Plugin để chặn input trước khi vào model.
6. Implement content filter cho output để phát hiện PII, secret, dữ liệu nhạy cảm.
7. Implement LLM-as-Judge để đánh giá response.
8. Tạo Output Guardrail Plugin để chặn hoặc chỉnh response trước khi trả về user.
9. Cấu hình NeMo Guardrails bằng Colang.
10. Chạy lại các attack trên protected agent để so sánh trước/sau.
11. Tạo pipeline test bảo mật tự động.
12. Implement confidence router cho HITL.
13. Thiết kế 3 điểm quyết định cần human-in-the-loop.

## 2. Xây pipeline defense-in-depth hoàn chỉnh

Assignment yêu cầu bạn xây một pipeline nhiều lớp phòng thủ, không chỉ từng guardrail rời rạc.

Kiến trúc tối thiểu nên có:

`User Input -> Rate Limiter -> Input Guardrails -> LLM -> Output Guardrails -> Audit Log / Monitoring -> Response`

Bạn phải có ít nhất 4 lớp an toàn độc lập, cộng thêm audit/monitoring.

## 3. Các thành phần bắt buộc phải có

Bạn cần implement các thành phần sau:

1. `Rate Limiter`
   Chặn user gửi quá nhiều request trong một khoảng thời gian.

2. `Input Guardrails`
   Phát hiện prompt injection, chặn off-topic, chặn yêu cầu nguy hiểm.
   Có thể kết hợp regex, topic filter, và NeMo Guardrails.

3. `Output Guardrails`
   Lọc PII, secrets, internal info trong phản hồi của model.
   Có cơ chế redact hoặc block.

4. `LLM-as-Judge`
   Dùng một model riêng để chấm response theo nhiều tiêu chí:
   - safety
   - relevance
   - accuracy
   - tone

5. `Audit Log`
   Log toàn bộ interaction:
   - input
   - output
   - layer nào block
   - latency
   - metadata liên quan

6. `Monitoring & Alerts`
   Theo dõi các chỉ số:
   - block rate
   - số lần rate limit hit
   - judge fail rate
   - bất thường theo ngưỡng

## 4. Yêu cầu test bắt buộc

Bạn phải chạy pipeline với các bộ test sau và hiển thị output trong notebook:

1. `Safe queries`
   Tất cả phải pass.

2. `Attack queries`
   Tất cả phải bị block.

3. `Rate limiting test`
   Gửi 15 request nhanh từ cùng một user.
   Kỳ vọng: 10 request đầu pass, 5 request sau bị block.

4. `Edge cases`
   Bao gồm:
   - input rỗng
   - input rất dài
   - emoji-only input
   - SQL injection
   - câu hỏi off-topic đơn giản

## 5. Kết quả bạn cần thể hiện trong notebook

Notebook nộp bài nên thể hiện rõ:

1. Pipeline chạy end-to-end.
2. Rate limiter hoạt động đúng.
3. Input guardrails chặn được attack và cho biết chặn bởi pattern/layer nào.
4. Output guardrails redact hoặc block dữ liệu nhạy cảm.
5. LLM-as-Judge in ra điểm theo nhiều tiêu chí.
6. Có comment rõ cho từng class và function:
   - nó làm gì
   - vì sao cần nó
   - nó bắt loại tấn công nào mà layer khác có thể bỏ sót

## 6. Báo cáo cá nhân cần làm

Bạn cần nộp thêm báo cáo 1-2 trang, trả lời các câu hỏi sau:

1. Với 7 attack prompts trong test, layer nào bắt attack đầu tiên.
   Nếu có nhiều layer cùng bắt được, liệt kê tất cả.

2. Có false positive với safe queries không.
   Nếu chưa có, hãy siết guardrails hơn để xem false positive xuất hiện ở đâu.
   Phân tích trade-off giữa security và usability.

3. Thiết kế 3 attack prompts mà pipeline hiện tại chưa bắt được.
   Giải thích vì sao bypass được.
   Đề xuất layer mới để chặn.

4. Nếu triển khai cho ngân hàng thật với 10,000 users, bạn sẽ thay đổi gì.
   Nên bàn về:
   - latency
   - cost
   - monitoring ở quy mô lớn
   - cập nhật rule mà không redeploy

5. Trả lời câu hỏi đạo đức:
   Có thể xây AI “perfectly safe” không?
   Khi nào nên từ chối trả lời, khi nào nên trả lời kèm disclaimer?

## 7. Deliverables cuối cùng

Bạn cần chuẩn bị:

1. Notebook hoặc bộ file `.py` chạy được.
2. Security report so sánh trước/sau guardrails.
3. HITL flowchart với 3 decision points.
4. Audit log hoặc JSON export của interactions.
5. Báo cáo cá nhân 1-2 trang.

## 8. Thứ tự làm khuyến nghị

Để làm nhanh và ít vỡ nhất, nên theo thứ tự này:

1. Hoàn thành TODO 3-9 trong lab.
2. Hoàn thành TODO 10-11 để có so sánh và test pipeline.
3. Thêm `Rate Limiter`.
4. Thêm `Audit Log`.
5. Thêm `Monitoring & Alerts`.
6. Chạy toàn bộ test suite.
7. Ghi lại kết quả vào notebook.
8. Viết báo cáo cá nhân.

## 9. Nếu muốn lấy điểm tốt hơn

Bạn có thể làm thêm một lớp an toàn thứ 6 để lấy bonus, ví dụ:

1. Toxicity classifier
2. Language detection
3. Session anomaly detector
4. Embedding similarity filter
5. Hallucination detector
6. Cost guard

## 10. Checklist ngắn

- [x] Hoàn thành 13 TODO của lab  ← TODO 3-11 done in lab; TODO 12 (ConfidenceRouter) + TODO 13 (HITL) fixed in notebook
- [x] Có rate limiter              ← RateLimiterPlugin (sliding window) — Part 5 notebook
- [x] Có input guardrails          ← InputGuardrailPlugin (detect_injection + topic_filter) — Part 2A
- [x] Có output guardrails         ← OutputGuardrailPlugin (content_filter + LLM-as-Judge) — Part 2B (bug fixed)
- [x] Có LLM-as-Judge              ← llm_safety_check() với safety_judge_agent — Part 2B
- [x] Có audit log                 ← AuditLogPlugin + export_json() — Part 5 notebook
- [x] Có monitoring và alert       ← MonitoringAlert (block rate / rate-limit / judge fail) — Part 5
- [x] Chạy đủ 4 test suites        ← Test Suite 1–4 trong Part 5 notebook
- [x] Xuất được kết quả trong notebook ← audit_log.json export ở cuối Part 5
- [x] Viết báo cáo cá nhân         ← INDIVIDUAL_REPORT_Assignment11.md
- [x] Chuẩn bị security report     ← SECURITY_REPORT.md (before/after 5+7 attacks, NeMo coverage, residual risks)
- [x] Chuẩn bị HITL flowchart      ← HITL_FLOWCHART.md (ASCII diagram + 3 decision points + HITL model table)
