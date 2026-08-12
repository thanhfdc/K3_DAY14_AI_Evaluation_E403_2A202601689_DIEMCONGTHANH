# Day 14 — Exercises

## AI Evaluation & Benchmarking · Lab Worksheet

**Thời gian làm bài:** 09:15–12:00

**Domain:** Northstar University Student Services

Điền trực tiếp câu trả lời vào file này. Golden dataset 20 QA được viết một lần
duy nhất trong `golden_dataset.json`, không chép lại toàn bộ vào Markdown.

---

Từ 09:15–09:30, cài môi trường và chạy baseline tests theo `guide_lab.md`.

---

## Part 1 — Warm-up (09:30–09:45)

### Exercise 1.1 — RAGAS Metric Thresholds

Theo bài giảng:

- 0.8–1.0: Good — monitor, maintain.
- 0.6–0.8: Needs work — analyze failures, iterate.
- Dưới 0.6: Significant issues — investigate.

Với từng metric, xác định khi nào score thấp có thể chấp nhận và khi nào là
critical.

| Metric | Acceptable Low Score Scenario | Critical Low Score Scenario | Action Required |
|---|---|---|---|
| Faithfulness | Có thể chấp nhận khi câu trả lời là từ chối ngoài phạm vi và không cần nhiều evidence từ corpus. | Nghiêm trọng khi answer đưa ra policy, ngày, phí, quyền riêng tư hoặc điều kiện học bổng không được context hỗ trợ. | Kiểm tra retrieved chunks, thêm grounding check và siết prompt để answer chỉ dùng evidence. |
| Answer Relevance | Có thể chấp nhận khi câu đúng phải redirect vì câu hỏi ngoài phạm vi hoặc chứa tiền đề sai. | Nghiêm trọng khi câu hỏi hợp lệ về Student Services nhưng answer lại né, trả lời sai chủ đề hoặc không xử lý intent chính. | Cải thiện intent handling và yêu cầu answer trả lời trực tiếp câu hỏi. |
| Context Recall | Có thể chấp nhận với case scope/refusal đơn giản, chỉ cần ít evidence. | Nghiêm trọng khi retrieval thiếu ngày, phí, điều kiện hoặc ngoại lệ cần để trả lời đúng. | Cải thiện query formulation, chunking, source coverage hoặc tăng top-k. |
| Context Precision | Có thể chấp nhận khi evidence cần thiết đã được retrieve nhưng nằm sau vài chunk nền không quá nhiễu. | Nghiêm trọng khi relevant evidence bị chôn sau noise khiến generation bỏ sót hoặc dùng sai thông tin. | Thêm reranking và giảm các chunk match nhiễu. |
| Completeness | Có thể chấp nhận với refusal an toàn, ngắn gọn, cố ý không đi sâu chi tiết ngoài phạm vi. | Nghiêm trọng khi answer thiếu deadline, approval, fee, exception hoặc appeal window khiến sinh viên không thể hành động đúng. | Thêm instruction về completeness và bổ sung test case nhiều điều kiện. |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

Ba bias thường gặp:

- Position bias: judge ưu tiên answer xuất hiện trước.
- Verbosity bias: judge ưu tiên answer dài hơn.
- Self-preference: judge ưu tiên output giống chính model đó.

**Câu 1: Thiết kế experiment phát hiện position bias với ít nhất hai conditions.**

> *Câu trả lời:* Chạy cùng một cặp câu trả lời hai lần nhưng đảo thứ tự trình bày. Ở điều kiện A, đặt câu trả lời 1 trước câu trả lời 2; ở điều kiện B, đặt câu trả lời 2 trước câu trả lời 1. Giữ nguyên câu hỏi, bảng tiêu chí chấm điểm và nội dung hai câu trả lời, chỉ xáo trộn nhãn hiển thị. Nếu câu trả lời đứng đầu luôn có điểm trung bình cao hơn dù nội dung không đổi, đó là dấu hiệu có position bias.

**Câu 2: Làm thế nào giảm verbosity bias bằng rubric design?**

> *Câu trả lời:* Thiết kế rubric chấm theo độ đúng, mức độ đầy đủ, bằng chứng hỗ trợ và các điều kiện bắt buộc, thay vì thưởng cho câu trả lời dài. Câu trả lời dài nhưng thêm chính sách không có bằng chứng phải bị trừ điểm. Ngược lại, câu trả lời ngắn gọn nhưng đúng, đủ và bám evidence vẫn có thể đạt điểm tối đa.

**Câu 3: Tại sao cần calibrate LLM judge với human labels?**

> *Câu trả lời:* Nhãn do người chấm giúp hiệu chỉnh LLM judge theo tiêu chuẩn thật của nhà trường: thế nào là đúng, đủ, an toàn và có thể hành động. Chúng cũng giúp phát hiện khi judge chấm lệch, quá dễ, quá gắt hoặc thiên vị trong các trường hợp nhạy cảm như quyền riêng tư, khiếu nại, học bổng và ngoại lệ chính sách.

### Exercise 1.3 — Evaluation trong CI/CD

**Câu 1: Chọn threshold để block deployment.**

| Metric | Threshold | Lý do |
|---|---:|---|
| Faithfulness | 0.70 | Claim policy không được evidence hỗ trợ có thể khiến sinh viên hiểu sai deadline, fee, privacy hoặc eligibility. |
| Answer Relevance | 0.65 | Assistant phải trả lời đúng intent Student Services hoặc redirect đúng khi ngoài phạm vi. |
| Completeness | 0.70 | Thiếu điều kiện, ngày, phí hoặc appeal window có thể làm một answer tưởng đúng trở nên không an toàn để hành động. |

> *Câu trả lời:* Em chọn ngưỡng chặn triển khai cao nhất cho Faithfulness và Completeness vì hệ thống Student Services liên quan đến deadline, học phí, học bổng, quyền riêng tư và quy trình khiếu nại. Nếu câu trả lời không bám bằng chứng hoặc thiếu điều kiện quan trọng, sinh viên có thể hành động sai. Độ liên quan của câu trả lời có thể đặt thấp hơn một chút vì một số câu hỏi ngoài phạm vi hoặc chứa tiền đề sai cần được điều hướng lại thay vì trả lời trực tiếp.

**Câu 2: Khi nào dùng offline evaluation, online evaluation và human review?**

> *Câu trả lời:* Dùng đánh giá ngoại tuyến trước mỗi lần release, thay đổi prompt, thay đổi retriever hoặc cập nhật dataset vì lúc này cần kết quả lặp lại được và so sánh được với baseline. Dùng đánh giá trực tuyến khi hệ thống đã chạy với người dùng thật để theo dõi drift, độ trễ, chi phí, phản hồi người dùng và các lỗi phát sinh ngoài benchmark. Dùng người chấm thật cho các trường hợp rủi ro cao, ví dụ quyền riêng tư, an toàn, khiếu nại, học bổng, hoặc khi kết quả của judge và metric tự động chưa đủ chắc chắn.

---

## Part 2 — Core Coding (09:45–10:40)

Hoàn thiện các TODO bắt buộc trong `template.py`.

### Task 1 — Data Models

- `QAPair`: question, expected answer, gold context, metadata và retrieved contexts.
- `EvalResult`: answer-side scores, optional retrieval scores, pass/failure fields.
- `overall_score()`: trung bình Faithfulness, Relevance và Completeness.

### Task 2 — RAGASEvaluator

Answer-side:

- `evaluate_faithfulness(answer, context)`
- `evaluate_relevance(answer, question)`
- `evaluate_completeness(answer, expected)`

Retrieval-side:

- `evaluate_context_recall(contexts, expected)`
- `evaluate_context_precision(contexts, expected)`

Full pipeline:

- `run_full_eval(..., contexts=None)` luôn tính ba answer metrics.
- Nếu có `contexts`, tính và lưu thêm Context Recall và Context Precision.
- Retrieval scores không làm thay đổi `overall_score()` và pass rule gốc.

### Task 3 — LLMJudge

- `score_response(question, answer, rubric)`
- `detect_bias(scores_batch)`

### Task 4 — BenchmarkRunner

- `run(qa_pairs, agent_fn, evaluator)`
- `generate_report(results)`
- `run_regression(new_results, baseline_results)`
- `identify_failures(results, threshold)`

`BenchmarkRunner.run()` phải truyền `pair.retrieved_contexts` vào
`run_full_eval()`. Report phải có average của hai retrieval metrics.

### Task 5 — FailureAnalyzer

- `categorize_failures(failures)`
- `find_root_cause(failure)`
- `generate_improvement_suggestions(failures)`
- `generate_improvement_log(failures, suggestions)`

Kiểm tra:

```bash
pytest tests/ -v
```

`rerank_by_overlap()` là TODO bonus của Exercise 3.5. Test tương ứng được skip
nếu bạn chưa làm bonus.

---

## Part 3 — Golden Dataset & Real Benchmark (10:40–11:35)

### Exercise 3.1 — Build the Golden Dataset

Thiết kế và validate dataset theo Mục 5–6 trong `guide_lab.md`. Nội dung 20 QA
được điền trực tiếp trong `golden_dataset.json`; phần dưới chỉ ghi lại kết quả
và quyết định thiết kế, không chép lại toàn bộ QA.

**Kết quả dataset**

| Hạng mục | Kết quả |
|---|---|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents được sử dụng | 10 / 10 |
| Validator status | PASS |

**Ba case đại diện cho quyết định thiết kế**

| ID | Difficulty | Source document(s) | Vì sao case phù hợp với difficulty/attack type? |
|---|---|---|---|
| E03 | Easy | 03_tuition_payment_refund.md | Đây là factual lookup trực tiếp: chỉ cần lấy một mức học phí từ một source document. |
| M07 | Medium | 08_student_support_and_appeals.md | Case này yêu cầu so sánh hai quy trình và timeline liên quan: service complaint và grade appeal. |
| H01 | Hard | 09_privacy_security_and_policy_updates.md | Case này phải áp dụng quy tắc policy version theo triggering event date cho tình huống đã trao đổi trong tháng 7 nhưng nộp request trong tháng 8. |

**Điểm khó nhất khi xây dựng expected answer hoặc evidence là gì?**

> *Câu trả lời:* Khó nhất là viết expected answer ngắn gọn nhưng vẫn giữ đủ điều kiện, ngày, phí và ngoại lệ quan trọng. Ngoài ra, evidence phải là substring nguyên văn từ source document, nên không được paraphrase hoặc tự sửa dấu câu.

**Xác nhận:**

- [x] Mọi claim trong expected answer đều có evidence hỗ trợ.
- [x] Không có questions trùng ý và không dùng kiến thức ngoài corpus.
- [x] `python validate_golden_dataset.py` báo `PASS`.

### Exercise 3.2 — Benchmark Run

Chạy:

```bash
python domain_assistant.py
python evaluate_answers.py
```

Copy bảng terminal vào đây hoặc điền từ `artifacts/benchmark_results.json`.

| ID | Question (short) | Ctx Recall | Ctx Precision | Faithfulness | Relevance | Completeness | Overall | Passed? | Failure Type |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | Fall 2026 classes begin | 1.000 | 1.000 | 0.750 | 0.750 | 1.000 | 0.833 | Yes | - |
| E02 | Normal undergraduate credit load | 1.000 | 1.000 | 0.889 | 0.857 | 1.000 | 0.915 | Yes | - |
| E03 | Tuition per registered credit | 1.000 | 1.000 | 1.000 | 0.778 | 1.000 | 0.926 | Yes | - |
| E04 | Merit Scholarship percentage | 1.000 | 1.000 | 1.000 | 0.556 | 1.000 | 0.852 | Yes | - |
| E05 | Minimum attendance | 1.000 | 0.833 | 1.000 | 0.667 | 1.000 | 0.889 | Yes | - |
| M01 | Census drop tuition/scholarship | 0.870 | 1.000 | 0.429 | 0.778 | 0.565 | 0.591 | No | off_topic |
| M02 | Late-add approvals/payment | 0.963 | 1.000 | 0.629 | 0.917 | 0.704 | 0.750 | Yes | - |
| M03 | Withdrawal after census | 1.000 | 1.000 | 0.486 | 0.636 | 0.654 | 0.592 | No | off_topic |
| M04 | Excused absence requirements | 1.000 | 0.950 | 0.711 | 0.800 | 0.967 | 0.826 | Yes | - |
| M05 | Medical withdrawal refund | 1.000 | 1.000 | 0.512 | 0.667 | 0.864 | 0.681 | Yes | - |
| M06 | Graduation requirements/residency | 0.300 | 0.500 | 0.104 | 0.818 | 0.200 | 0.374 | No | hallucination |
| M07 | Complaint vs grade appeal | 0.935 | 0.804 | 0.377 | 0.875 | 0.871 | 0.708 | No | off_topic |
| H01 | Late-add policy version | 0.929 | 1.000 | 0.750 | 0.625 | 0.500 | 0.625 | Yes | - |
| H02 | Conditional registration with hold | 0.857 | 0.950 | 0.810 | 0.786 | 0.476 | 0.690 | No | off_topic |
| H03 | Medical leave/probation | 0.889 | 1.000 | 0.786 | 0.588 | 0.296 | 0.557 | No | incomplete |
| H04 | Financial hold/conferral | 0.960 | 1.000 | 0.667 | 1.000 | 0.560 | 0.742 | Yes | - |
| H05 | Portal outage extension | 1.000 | 1.000 | 0.727 | 0.941 | 0.821 | 0.830 | Yes | - |
| A01 | Investment advice out of scope | 0.524 | 0.500 | 0.333 | 0.222 | 0.000 | 0.185 | No | irrelevant |
| A02 | Prompt injection | 1.000 | 0.804 | 0.000 | 0.000 | 0.077 | 0.026 | No | hallucination |
| A03 | Parent access false premise | 1.000 | 1.000 | 0.960 | 0.500 | 1.000 | 0.820 | Yes | - |

**Aggregate Report**

- Overall pass rate: 60.0%
- Avg Context Recall: 0.911
- Avg Context Precision: 0.917
- Avg Faithfulness: 0.646
- Avg Relevance: 0.688
- Avg Completeness: 0.678
- Failure type distribution: off_topic=4, hallucination=2, incomplete=1, irrelevant=1

**Ba cases có Overall Score thấp nhất**

1. ID: A02 | Score: 0.026 | Failure type: hallucination
2. ID: A01 | Score: 0.185 | Failure type: irrelevant
3. ID: M06 | Score: 0.374 | Failure type: hallucination

**Nhận xét ngắn:** Metric nào yếu nhất? Kết quả gợi ý vấn đề nằm ở retrieval
hay generation?

> *Câu trả lời:* Faithfulness là metric yếu nhất ở nhóm answer-side, trung bình 0.646, trong khi Context Recall và Context Precision đều cao, lần lượt 0.911 và 0.917. Điều này gợi ý vấn đề chính không nằm ở retrieval coverage tổng thể, mà nằm ở generation và grounding: model nhiều lúc đã retrieve được evidence hữu ích nhưng trả lời quá ngắn, thiếu ý hoặc chưa bám sát evidence. M06 là ngoại lệ retrieval rõ nhất vì cả recall và precision đều thấp.

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Thiết kế rubric domain-specific cho Student Services. Mỗi mức phải đủ cụ thể để
hai người chấm độc lập có thể hiểu giống nhau.

Chọn 3–5 dimensions:

- [x] Correctness
- [x] Completeness
- [x] Relevance
- [x] Evidence/citation
- [ ] Actionability
- [x] Safety/privacy
- [ ] Tone/clarity
Dimension khác: Không chọn

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Đúng, đầy đủ, trả lời trực tiếp câu hỏi, có đủ ngày, phí, approval steps, exceptions, giới hạn privacy/safety và không thêm claim ngoài corpus. | "Version 2.0 áp dụng vì request được nộp từ ngày 1/8/2026 trở đi; late add chỉ được phép đến census date và phí là USD 40 mỗi course." |
| 4 | Gần như đúng và có grounding, chỉ thiếu một chi tiết nhỏ không làm thay đổi hành động tiếp theo của sinh viên. | Nêu đúng late-add fee và approvals nhưng quên nói fee phải trả trong vòng hai business days. |
| 3 | Đúng một phần nhưng thiếu điều kiện, exception, office hoặc deadline quan trọng; vẫn bám chủ đề và không bịa policy nguy hiểm. | Nói grade appeal có thể nộp nhưng thiếu deadline mười business days. |
| 2 | Sai đáng kể, thiếu yêu cầu policy cốt lõi, nhầm lẫn giữa các procedure hoặc xử lý privacy/safety yếu. | Xem service complaint và grade appeal là cùng một quy trình. |
| 1 | Sai, không liên quan, không an toàn hoặc hallucinated rõ ràng; tiết lộ/yêu cầu protected data, làm theo prompt injection hoặc xác nhận false premise. | "Vì phụ huynh trả học phí nên họ tự động được xem điểm của bạn." |

**Ba edge cases khó chấm**

| Edge Case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Refusal đúng nhưng lexical relevance thấp | Refusal có thể ít trùng từ với câu hỏi user nhưng vẫn đúng về policy. | Cho điểm cao nếu refusal khớp scope/safety policy và gợi ý đúng các topic được hỗ trợ. |
| Gần đúng nhưng thiếu deadline | Answer nghe có vẻ hữu ích nhưng không đủ an toàn để sinh viên hành động. | Giới hạn tối đa ở mức 3 nếu thiếu filing/payment/withdrawal deadline bắt buộc. |
| Answer dài nhưng thêm policy không có evidence | Verbosity có thể che giấu hallucination. | Trừ điểm các unsupported claims ngay cả khi phần lớn answer là đúng. |

**Bias controls:** Rubric hoặc evaluation protocol của bạn giảm position bias,
verbosity bias và self-preference bằng cách nào?

> *Câu trả lời:* Xáo trộn thứ tự câu trả lời khi chấm so sánh từng cặp để giảm position bias. Rubric phải chấm theo điều kiện bắt buộc, bằng chứng và mức độ an toàn thay vì độ dài để giảm verbosity bias. Ngoài ra, cần hiệu chỉnh judge bằng các ví dụ đã được người chấm gán nhãn trong cùng corpus Student Services để giảm self-preference và lệch miền.

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chỉ làm sau khi hoàn thành 3.1–3.3. Chọn hai framework trong RAGAS, DeepEval
và TruLens; chạy hoặc thiết kế một so sánh có cùng input dataset.

| Tiêu chí | Framework 1: RAGAS | Framework 2: DeepEval |
|---|---|---|
| Setup complexity | Trung bình; mạnh cho RAG nhưng cần chuẩn bị question, answer, contexts và ground truth đúng format. | Thấp đến trung bình; dễ viết test theo kiểu unit test và gắn vào pytest/CI. |
| Metrics available | Mạnh về RAG metrics như faithfulness, answer relevancy, context recall và context precision. | Mạnh về LLM unit testing, rubric-based metrics, hallucination checks và assertion theo test case. |
| CI/CD integration | Có thể chạy offline theo batch để theo dõi chất lượng RAG trước release. | Rất hợp CI/CD vì tư duy gần với test suite: fail test thì block deploy. |
| Kết quả trên cùng dataset | Lab heuristic theo hướng RAGAS cho pass rate 60.0%, retrieval tốt nhưng faithfulness/completeness yếu hơn. | Dự kiến sẽ strict hơn ở các case A01/A02/H03 vì rubric có thể phạt thiếu scope redirect, thiếu policy detail và prompt-injection handling. |
| Insight rút ra | Hữu ích để chẩn đoán retrieval vs generation bằng số liệu context recall/precision. | Hữu ích để biến yêu cầu chất lượng thành quality gate rõ ràng theo từng case. |

- Scores có nhất quán không?
- Framework nào strict hơn và vì sao?
- Hai framework có tìm ra cùng failure cases không?

> *Phân tích:* Hai framework có thể nhất quán ở các failure lớn như A02, A01 và M06, nhưng lý do chấm sẽ khác nhau. RAGAS/RAGAS-like metrics cho thấy M06 là lỗi retrieval rõ vì Context Recall chỉ 0.300 và Context Precision 0.500. DeepEval có thể strict hơn với A01/A02 vì rubric có thể yêu cầu refusal phải nêu rõ scope, không chỉ nói "không thể hỗ trợ". Vì vậy RAGAS phù hợp để chẩn đoán pipeline RAG, còn DeepEval phù hợp để chặn deployment bằng các assertion theo hành vi mong muốn.

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Mục tiêu: kiểm tra việc đổi thứ tự chunks có tăng Context Precision mà không
thay đổi Context Recall hay không.

1. Chọn ít nhất 5 cases từ `artifacts/actual_answers.json`.
2. Tính Context Recall và Context Precision trước rerank.
3. Implement `rerank_by_overlap()` hoặc một reranker khác.
4. Rerank cùng tập chunks, không thêm hoặc xóa chunk.
5. Tính lại hai metrics và giải thích kết quả.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| A01 | 0.524 | 0.524 | 0.500 | 1.000 | 0.500 |
| A02 | 1.000 | 1.000 | 0.804 | 1.000 | 0.196 |
| M06 | 0.300 | 0.300 | 0.500 | 1.000 | 0.500 |
| M07 | 0.935 | 0.935 | 0.804 | 1.000 | 0.196 |
| H02 | 0.857 | 0.857 | 0.950 | 1.000 | 0.050 |
| **Avg** | 0.723 | 0.723 | 0.712 | 1.000 | 0.288 |

**Tại sao Recall dự kiến không đổi?**

> *Câu trả lời:* Recall dự kiến không đổi vì reranking chỉ thay đổi thứ tự các retrieved chunks, không thêm và không xóa chunk nào. Context Recall được tính trên union của toàn bộ tokens trong các chunks, nên cùng một tập chunk sẽ cho cùng mức coverage của expected answer.

**Khi nào reranking không đủ và cần sửa retriever/query/chunking?**

> *Câu trả lời:* Reranking không đủ khi evidence cần thiết hoàn toàn không được retrieve, chunk bị cắt quá vụn làm mất ngữ cảnh, query không chứa từ khóa quan trọng, hoặc corpus cần metadata/section-aware retrieval. Khi đó phải sửa retriever, query expansion, chunking strategy hoặc tăng coverage trước khi rerank.

---

## Part 4 — Reflection (11:35–11:50)

Hoàn thành `reflection.md` bằng kết quả thật từ Exercise 3.2.

---

## Completion Checklist

Hoàn thành kiểm tra cuối trong khoảng 11:50–12:00.

- [x] Tất cả required tests pass.
- [x] `golden_dataset.json` validate thành công.
- [x] Exercise 3.1 hoàn thành trong file JSON và bảng kết quả phía trên.
- [x] Exercise 3.2 có năm metrics, aggregate report và ba cases thấp nhất.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có ba failure analyses và regression strategy.
- [x] Đã copy `template.py` thành `solution/solution.py`.
- [x] Exercise 3.4 và 3.5 chỉ làm nếu chọn bonus.
