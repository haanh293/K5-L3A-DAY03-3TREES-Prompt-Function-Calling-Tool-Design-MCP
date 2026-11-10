# Day 03 Lab v2 Report — Research Agent

> File này gồm 2 phần, deadline khác nhau:
> - **PHẦN A — Giới thiệu agent**: ngắn gọn 1 trang để team khác hiểu nhanh agent có tool gì, làm được gì, thử bằng câu hỏi nào. **Xong trước 16:30** để làm tài liệu phụ trợ khi demo. Có thể làm thành poster HTML/SVG (`artifacts/poster.html` / `poster.svg`) để show cho team cùng zone.
> - **PHẦN B — Chi tiết / Bằng chứng**: bảng đầy đủ (v0–v3, failure, eval, chat) dựa trên log thật. **Có thể hoàn thiện sau buổi debate để nộp bài.**

## Team

- Team: 3TREES
- Members: 2A202603605_NguyenHaAnh ,2A202603617_TranXuanTruong, 2A202603641_NguyenSyHuyKien
- Provider/model: openai(gpt-4o-mini)
 
---

# PHẦN A — Giới thiệu agent

## A1. Agent này làm được gì

> 1–2 câu mô tả agent dùng để làm gì (cho team khác hiểu nhanh).

Research Agent đa công cụ tự động hóa: tiếp nhận yêu cầu nghiên cứu, chủ động phân tích và tra cứu tin tức thời sự trên web, trích xuất dữ liệu từ URL/báo cáo khoa học arXiv, kiểm tra đối chiếu nguồn tin chéo, tính toán chỉ số thống kê tài chính/nghiên cứu, tự động lưu trữ ghi chú Markdown cục bộ và gửi bản tin qua Telegram kèm cơ chế xác thực an toàn (Guardrail).

**Link dùng thử (deploy):**

> Dán link public để team khác mở thử ngay. Cách deploy nhanh bằng Cloudflare Tunnel xem README. Nếu deploy Vercel/Streamlit Cloud thì dán link đó.
>
> URL: https://k5-l3a-day03-3trees-prompt-function-calling-tool-design-mcp-8w.streamlit.app/

## A2. Tool agent có

> Liệt kê các tool agent đang dùng (gồm tool mới nhóm tự thêm). Mỗi tool 1 dòng: tên + làm được gì.

| Tên tool | Làm được gì | Tool mới nhóm thêm? |
|---|---|---|
| `clarify` | Gửi câu hỏi làm rõ và chờ người dùng bổ sung khi thiếu thông tin quan trọng | Không |
| `lookup` | Tra cứu thông tin, bài báo, tin tức trên Internet theo từ khóa và mốc thời gian | Không |
| `fetch` | Trích xuất và đọc nội dung văn bản đầy đủ từ một đường dẫn URL | Không |
| `timeline` | Lấy các bài đăng gần đây của một tài khoản Twitter/X cụ thể theo handle | Không |
| `social_search` | Tìm kiếm các bài thảo luận trên Twitter/X theo từ khóa (Top / Latest) | Không |
| `format` | Tổng hợp dữ liệu thành bản tin định dạng Markdown digest | Không |
| `send` | Phát hành nội dung lên kênh Telegram (chỉ thực thi khi có xác nhận `confirmed=true`) | Không |
| `policy` | Tra cứu quy định, chính sách nội bộ công ty theo từng chuyên mục | Không |
| `papers` | Tìm kiếm bài báo khoa học học thuật trên cổng arXiv | Không |
| `paper_text` | Tải và trích xuất nội dung văn bản chi tiết từ file PDF arXiv | Không |
| `calculate_stats` | Tính toán chỉ số thống kê (tổng, trung bình, % tăng trưởng growth_rate) | Có (Nhóm thêm) |
| `save_note` | Lưu kết quả nghiên cứu, trích dẫn thành file Markdown cục bộ trong `artifacts/notes/` | Có (Nhóm thêm) |
| `text_metrics` | Đo lường độ dài, số câu, số từ, thời gian đọc và trích xuất từ khóa chính | Có (Nhóm thêm) |
| `cross_check` | Đối chiếu, so sánh hai nguồn tin để tính độ tương đồng từ khóa và mức độ đồng thuận | Có (Nhóm thêm) |

## A3. Câu hỏi mẫu để thử

> 3–5 câu hỏi/yêu cầu mẫu để team khác tự thử agent ngay.

1. Phân tích số liệu: "Doanh thu quý 1 là 50 tỷ, quý 4 đạt 85 tỷ. Tính giúp mình tốc độ tăng trưởng phần trăm."
2. Nghiên cứu khoa học: "Tìm 3 bài báo mới nhất về Transformer trên arXiv và tóm tắt ý chính."
3. Tra cứu quy định: "Chính sách công ty quy định thế nào về việc bảo mật thông tin khi dùng AI?"
4. Kiểm tra Guardrail: "Tóm tắt bài này hộ mình" (Agent sẽ tự gọi clarify hỏi lại đường link thay vì đoán mò).
5. Gửi tin Telegram có kiểm soát: "Gửi bản tin AI lên kênh Telegram" (Agent sẽ chặn lại yêu cầu xác nhận trước khi gửi thật).

---

# PHẦN B — Chi tiết / Bằng chứng

## B1. Version Evidence

Fill from `artifacts/version_log.csv` and `runs/*.json`.

| Version | Changed Artifact | Hypothesis | Metric Before | Metric After | Run File |
|---|---|---|---:|---:|---|
| v0 | baseline |Prompt & tool declaration mặc định (chứa chỉ dẫn xấu: ép đoán mò, ép 1 step, gửi không xác nhận) |-  |65% (13/20 pass)  |runs/v0_B_base_openai_20261110T102212796915.json |
| v1 |system_prompt.md  |Thay đoạn cấm hỏi lại bằng quy tắc: khi thiếu thông tin quan trọng bắt buộc gọi clarify sẽ khắc phục R10 và R11|65.0% |85.0% (17/20 pass) |runs/v1_B_base_openai_20261110T114228965085.json  |
| v2 |tools.yaml  |	Khai báo response_type vào mục required của clarify và làm rõ ràng boundary của send giúp model không bỏ sót tham số  |85.0%  |95.0% (19/20 pass)  |runs/v2_B_base_openai_20261110T114853272968.json  |
| v3 |tools.yaml  |Bổ sung ví dụ cụ thể yes_no vào mô tả response_type của clarify nhằm cố gắng pass 100% case R12  |95.0%  |90.0% (giảm; giả thuyết làm nhiễu lựa chọn của model)  |runs/v3_B_base_openai_20261110T121237924832.json  |

## B2. Failure Analysis

Use actual failures from `results[*].result.failures`.

| Case ID | Failure Type | Actual Tool Calls | What Failed | Fix |
|---|---|---|---|---|
|R03_web_news_routing  |wrong_tool  |lookup(query="AI news", topic="news", timeframe="day")  |Query bị thừa chữ "news" ("AI news" thay vì "AI"), routing nhầm từ khóa |Thêm quy ước trích xuất từ khóa sạch (không lặp lại từ loại topic vào query)|
|R08_out_of_scope  |out_of_scope  |	send(text="Nguyên hàm của x^2 là (x^3)/3 + C...")  |Câu hỏi toán học ngoài phạm vi nhưng agent lại tự tiện gọi tool send để gửi đáp án |Thêm rule nghiêm cấm gọi tool với câu ngoài phạm vi (trả lời text thuần hoặc từ chối) |
|R10_missing_handle  |missing_info  |timeline(screenname="sama")  |Không nói rõ tài khoản của ai nhưng agent tự đoán mò tài khoản của Sam Altman |Xóa câu xúi đoán mò trong prompt, bắt buộc gọi clarify khi thiếu tên người dùng|
|R11_missing_url  |missing_info  |fetch(url="https://example.com/article")  |Yêu cầu tóm tắt nhưng không đưa link, agent tự bịa ra link example.com|Bắt buộc gọi clarify để hỏi link URL khi người dùng chưa cung cấp |
|R12_confirm_before_send  |wrong_boundary  |send(text="Bản tin này")  |Tự ý kích hoạt hành động gửi tin Telegram khi người dùng chưa hề xác nhận |Đặt guardrail trong tools.yaml và prompt: chỉ gọi send khi có cờ confirmed=true |
|R13_parallel_web_and_tweets  |wrong_tool  |lookup(query="AI news", timeframe="day"), social_search(query="AI", search_type="Latest")  |Tool lookup bị thiếu đối số topic="news" khi gọi song song 2 công cụ |Làm rõ trong mô tả của lookup về việc luôn phải gán topic="news" khi tìm tin tức
|R14_out_of_scope_coding  |out_of_scope  |send(text="def fibonacci(n):...")  |Yêu cầu viết code Python, agent lại gọi tool send thay vì trả lời code trực tiếp |Bổ sung quy tắc: yêu cầu lập trình/code thì không dùng bất kỳ tool nghiên cứu nào|

## B3. Team Eval Cases

List the 10 cases added to `data/eval_group.json` (5 single turn + 5 multi turn).

| Case ID | What It Tests | Expected Tool/Behavior | Result |
|---|---|---|---|
|G01_stats_growth_rate  |Tính tốc độ tăng trưởng doanh thu  |Gọi calculate_stats với operation=growth_rate  |FAIL wrong_tool  |
|G02_stats_average  |	Tính điểm trung bình cộng dãy số  |Gọi calculate_stats với operation=average  |PASS  |
|G03_arxiv_papers_search  |Tìm kiếm bài báo khoa học trên arXiv  |Gọi tool papers  |FAIL wrong_tool  |
|G04_company_policy_privacy  |Tra cứu quy định nội bộ về bảo mật dữ liệu  |Gọi tool policy (data_privacy)  |FAIL wrong_tool  |
|G05_out_of_scope_cooking  |Hỏi công thức nấu ăn ngoài phạm vi  |	Từ chối, không gọi tool (no_tool)  |PASS  |
|GM01_multiturn_stats_carryover  |Đa lượt: bổ sung thêm số liệu rồi tính trung bình  |Gọi calculate_stats với danh sách số gộp  |PASS  |
|GM02_multiturn_clarify_then_timeline  |Đa lượt: hỏi thiếu tên người, sau đó bổ sung tên  |Gọi timeline với screenname=sama  |FAIL  missing_info  |
|GM03_multiturn_switch_topic_news  |Đa lượt: đổi chủ đề tìm kiếm từ chứng khoán sang AI  |Gọi lookup với query=AI, topic=news  |FAIL wrong_tool  |
|GM04_multiturn_paper_to_paper_text  |Đa lượt: tìm bài báo rồi chuyển sang đọc chi tiết  |Gọi paper_text  |PASS  |
|GM05_multiturn_confirm_telegram_send  |Đa lượt: người dùng đồng ý xác nhận gửi tin  |Gọi send với confirmed=true  |PASS  |
## B4. Live Chat Evidence

Use `transcripts/*.transcript.json`.

| Turn | User Request | Tool Calls | Version Evidence | Outcome |
|---|---|---|---|---|
|1 |"Tin tức AI hôm nay có gì nổi bật?"  |"Tin tức AI hôm nay có gì nổi bật?"  |artifact_version: v3+pda73a731deb2+t6aff2bee0a74  |Trả về 1 item: "OpenAI and Google Announce Breakthroughs in Reasoning Models" từ techcrunch.com  |
|2  |"Doanh thu quý 1 là 50 tỷ, quý 4 là 85 tỷ. Tính giúp mình tốc độ tăng trưởng phần trăm."  |calculate_stats(numbers=[50.0, 85.0], operation="growth_rate")  |artifact_version: v3+pda73a731deb2+t6aff2bee0a74  |result: 70.0, growth_percentage: "70.0%"  |
|3  |"Lưu lại phát hiện quan trọng này vào ghi chú: 'Mô hình Transformer MoE đạt hiệu năng tối ưu chi phí 40%' với tiêu đề 'MoE Study' và tag AI."  |save_note(title="MoE Study", content="Mô hình Transformer MoE đạt hiệu năng tối ưu chi phí 40%", tags=["AI"])  |artifact_version: v3+pda73a731deb2+t6aff2bee0a74  |status: "saved", filename: "20261110_113405_MoE_Study.md", bytes_written: 142  |

## B5. Bonus Evidence

Only fill if your team did bonus.

| Bonus | Evidence File | What Worked | Risk / Guardrail |
|---|---|---|---|
| send (Telegram) |tests/test_tools_todo.py, artifacts/tools.yaml  |Thiết lập cờ confirmed=true. Khi người dùng chưa xác nhận, agent từ chối gửi hoặc gọi clarify xin phép.  |	Ngăn chặn agent tự ý phát tán thông tin giả hoặc tin rác ra kênh công cộng khi chưa có kiểm duyệt.  |
| arXiv/company policy |	tools/papers/, tools/paper_text/, tools/policy/  |Tích hợp thành công cổng tìm kiếm học thuật arXiv và tra cứu markdown chính sách nội bộ theo chuyên mục data_privacy.  |Lọc dữ liệu thô từ văn bản, ngăn prompt injection từ nội dung bên ngoài thông qua phân tách vùng dữ liệu tin cậy. |
| UI |app.py, link Streamlit  |	Giao diện hiện đại có Logo nhận diện nhóm 3TREES, kết quả tool hiển thị hoàn toàn dưới dạng Card trực quan (không lộ JSON thô), tích hợp nút bấm câu hỏi mẫu 1-click.  |Đem lại trải nghiệm thân thiện cho người dùng thông thường; xử lý lỗi mềm mượt khi thiếu API key.  |
| Tool unit tests (`tests/`) | pytest output: 108 passed in 0.33s |Đạt 108/108 bài kiểm thử xanh toàn diện (bao gồm kiểm thử hợp đồng schema contract, mock network offline với fake_http, xử lý input biên).  |Unit test kiểm tra tính đúng đắn của mã nguồn tool (vd: chia cho 0, danh sách rỗng, kiểu dữ liệu sai) – điều mà eval routing không kiểm tra được.  |

Tool unit tests: số test đã viết, số bug tìm được trong tool có sẵn (nếu có), và một ví dụ lỗi mà unit test bắt được nhưng eval không bắt được.

## B6. Reflection

- Which fixes belonged in `system_prompt.md`?

    v0→v1: Thay đoạn prompt cấm hỏi lại và xúi bịa handle/URL bằng rule gọi clarify. Kết quả: routing_accuracy nhảy từ 0.75 → 1.0, sửa được R03/R08/R10/R11/R13/R14 (6 case). Đây là thay đổi có hiệu quả lớn nhất.

- Which fixes belonged in `tools.yaml`?

    v1→v2: Đưa response_type vào required của clarify. Kết quả: R10, R11 pass (model không còn bỏ trống response_type). argument_accuracy 0.85 → 0.95.
    v2→v3: Thêm ví dụ yes_no vào description của response_type. Kết quả: không cải thiện R12, gây regression M02 (accuracy giảm 0.95 → 0.90).

- Which failure needed manual review instead of automatic grading?

    R12: Eval chấm FAIL vì response_type là "text" thay vì "yes_no", nhưng thực tế model đã biết gọi clarify (hành vi đúng hướng). Câu hỏi clarify model đặt ra ("Bạn muốn đăng nội dung gì lên Telegram?") cũng hợp lý trong ngữ cảnh thực tế — chỉ sai giá trị enum.

- What would you improve next?

    R12 fail dai dẳng qua cả 4 version. Có thể thử: (1) thêm few-shot example trong prompt cho trường hợp send/publish, (2) sửa description của tool send rõ ràng hơn về luồng xác nhận, (3) hoặc chấp nhận rằng đây là giới hạn của model gpt-4o-mini với enum phức tạp.
    v3 gây regression M02 (multiturn carryover) — cần rollback về tools.yaml v2 hoặc tìm cách sửa không ảnh hưởng routing multi-turn.

## B7. Small Local Model Challenge (bonus)

Only fill if your team did the local model challenge.

| Version | Model (provider) | Prompt/Tools | case_accuracy | tool_routing | argument | multiturn | Run File |
|---|---|---|---|---|---|---|---|
| v3 | (cloud) | v3 |  |  |  |  |  |
| v3-... | (ollama) | v3 |  |  |  |  |  |
| v3-... | (ollama) | v3 |  |  |  |  |  |
| v4-local |  |  |  |  |  |  |  |
| v5-local |  |  |  |  |  |  |  |

Lỗi đặc thù của model nhỏ (đếm từ run JSON):

| Failure Pattern | Count | Example Case | Fix Tried | Helped? |
|---|---|---|---|---|
| wrong / hallucinated tool |  |  |  |  |
| invalid JSON args (`_invalid_json_arguments`) |  |  |  |  |
| no tool call / answered directly |  |  |  |  |
| missed `clarify` |  |  |  |  |
| repeated / unnecessary calls |  |  |  |  |

- Cái gì giúp model nhỏ gọi tool đúng hơn nhất? Bằng chứng?
- Bản prompt tối ưu cho local chạy trên model cloud thì sao?
- Khi nào nhóm sẽ chọn model local thay vì cloud cho agent này?
