# Kế hoạch paper: Evaluating Multi-Turn Privilege Escalation Vulnerabilities in LLM-Based Multi-Agent Systems

> Hướng: **Systematic** — Systematization of Knowledge (SoK) kết hợp Systematic Literature Review (SLR, theo PRISMA 2020), có một phần **đánh giá có hệ thống (systematic evaluation)** quy mô nhỏ để kiểm chứng taxonomy.
> Ngày lập: 2026-11-27 · Cập nhật: 2026-11-27 (thêm 4 paper benchmark P6–P9) · Nguồn khởi điểm: 9 paper trong `docs/`

---

## 1. Tóm tắt 9 paper nền (seed papers)

### 1a. Tấn công / phòng thủ / nghiên cứu thực nghiệm (P1–P5)

| # | Paper | Loại | Ý tưởng cốt lõi | Đánh giá | Khoảng trống liên quan "multi-turn + MAS" |
|---|---|---|---|---|---|
| P1 | **Defeating Prompt Injections by Design** (CaMeL, Debenedetti et al., arXiv 2503.18813) | Defense, single-agent | Tách control flow (P-LLM sinh chương trình Python) khỏi data flow (Q-LLM xử lý dữ liệu không tin cậy); capability + security policy tại tool-call | AgentDojo: 77% task với bảo đảm an ninh (vs 84% không phòng thủ) | Chỉ 1 agent; plan cố định từ query đầu → không mô hình hoá hội thoại nhiều lượt |
| P2 | **Can CaMeLs Talk?** (multi-CaMeL, Peters-Gill et al., arXiv 2610.05640) | Attack + Defense, MAS phân cấp | **Boundary laundering**: dữ liệu untrusted ở agent cha bị agent con đọc như instruction trusted → bảo đảm CaMeL **không compose**. Đề xuất kênh instruction/kênh data tách biệt | MultiAgentDojo (mở rộng AgentDojo), AssetOpsBench, 5 LLM. ASR: 12.9% → 0.2% (CaMeL riêng lẻ) → 0.0% (multi-CaMeL) | Thừa nhận **branch steering**, implicit flow, retry-loop leak; chỉ topology cây agent-as-tool; tấn công 1 lượt |
| P3 | **Prompt Flow Integrity (PFI)** (Kim et al., arXiv 2503.15547) | Defense, single-agent | Agent isolation (trusted/untrusted agent), thay dữ liệu bằng data ID, guardrail chống privilege escalation; phân biệt *prompt injection* vs *data-only attack* | AgentDojo, AgentBench OS; metric **SUR**, **ATR** | Single-agent; endorsement cần user approve → bề mặt tấn công qua nhiều lượt (approval fatigue) chưa xét |
| P4 | **Taming Various Privilege Escalation** (SEAgent, Ji et al., arXiv 2601.11893) | Formal model + Defense (MAC/ABAC) | Định nghĩa privilege escalation = hành động vượt least privilege của ý định user; 5 vector: direct PI, indirect PI, RAG poisoning, untrusted agent, **confused deputy trong MAS**; SEMemory theo dõi ngữ cảnh **multi-round** | InjecAgent, AgentDojo (bảo mật); API-Bank, AWS multi-agent (tiện dụng) | **Multi-round chỉ được đánh giá về utility**, phần MAS bỏ qua labeling/policy → **chưa có đánh giá bảo mật multi-turn trong MAS** |
| P5 | **Trust Propagation and Structural Containment** (Safin et al., arXiv 2609.17648) | Empirical study, MAS 4 agent (LangGraph) | Memory poisoning & indirect PI qua forged approval; HMAC task-bound token + policy oracle; metric **JBR** (Judgment Bypass Rate) tách "agent bị lừa" khỏi "hành động xảy ra" | 60 task, 3 seed, 1 model (gemma), 13 action type | Một kiến trúc, một model, tấn công "lộ liễu", không adaptive, không multi-turn |

### 1b. Benchmark (P6–P9)

| # | Benchmark | Thiết lập | Quy mô | Kết quả chính | Khoảng trống liên quan "multi-turn + MAS" |
|---|---|---|---|---|---|
| P6 | **AgentDojo** (Debenedetti et al., NeurIPS 2024 D&B, arXiv 2406.13352) | Single-agent, tool-use, injection trong tool output; môi trường mở rộng được (task, defense, adaptive attack) | 97 user task, 629 security test case; 4 suite: Workspace, Slack, Banking, Travel | LLM mạnh vẫn trượt nhiều task kể cả khi không bị tấn công; attack phá được một số nhưng không phải mọi security property | Một agent, injection một lượt; là nền của P1, P3, P4 và MultiAgentDojo (P2) |
| P7 | **InjecAgent** (Zhan et al., ACL Findings 2024, arXiv 2403.02691) | Single-agent, indirect prompt injection; 2 loại ý đồ: *direct harm* và *data exfiltration* | 1.054 test case, 17 user tool, 62 attacker tool, 30 agent | ReAct-GPT-4 bị tấn công thành công 24%; thêm "hacking prompt" thì gần gấp đôi | Tĩnh, một lượt, không có agent khác; được P4 dùng (mở rộng thêm RAG poisoning) |
| P8 | **ConVerse** (Gomaa, Salem, Abdelnabi, arXiv 2511.05359) | **Agent-to-agent, multi-turn**: assistant của user ↔ external service agent (có thể độc hại); tấn công cài trong hội thoại hợp lý | 3 domain (travel, real estate, insurance), 12 persona, 864 attack (611 privacy, 253 security: upselling, DoS, email fraud); 7 model, judge GPT-5 | Privacy ASR 37–88%, security ASR 2–60%; **model mạnh hơn lộ nhiều hơn**; dữ liệu "related & useful" lộ 90–94%; yêu cầu độc hại thường xuất hiện **muộn** trong hội thoại; assistant không kiểm chứng "authority claim" của agent ngoài | Chỉ 2 agent (1 assistant + 1 external); không có phòng thủ system-level; attack do LLM sinh, không adaptive |
| P9 | **Adaptive Adversaries / Security Arena** (Jain, Hartmann, Li, arXiv 2607.18063) | **Attacker LLM thích ứng qua nhiều round**, defender fresh-session mỗi round (cross-session search); chạy trên AgentBeats | 21 scenario; ma trận 3×3 attacker×defender = 945 battle; thêm 18.422 battle trong competition (gpt-oss-20b) | **ASR round 1: 0–1% → ASR@15: 7.9–16.8%**; gộp 3 attacker tìm ra 1.7–2.2× input thành công; một đoạn provenance giảm ASR 110/270 → 70/270; defender có memory: ASR 57.4% → 31.5%, nhưng attacker biết defender có memory: 14.8% | **Không có tool/retrieval/thay đổi trạng thái thật** (chỉ chấm output); một defender, không phải MAS; giao thức (history, state) làm thay đổi mạnh kết quả |

**Kết luận gap chung** (luận điểm trung tâm của paper):
1. Hầu hết đánh giá là **single-turn / single-task injection** (AgentDojo, InjecAgent). P9 cho thấy bằng số liệu rằng **chấm lượt đầu đánh giá thấp rủi ro khoảng 10 lần** (0–1% → 7.9–16.8%).
2. Bảo đảm cho từng agent **không tự compose** lên hệ thống (P2), và **không tự kéo dài theo thời gian** (chưa ai chứng minh cho multi-turn).
3. Hai benchmark multi-turn hiện có **nằm ở hai phía khác nhau và không giao nhau**: P8 có hội thoại agent↔agent nhưng chỉ 2 agent và không có phòng thủ system-level; P9 có attacker thích ứng nhưng không có tool, không có MAS. **Chưa benchmark nào kết hợp: multi-turn + MAS ≥3 agent + tool có tác động thật + phòng thủ system-level (CaMeL/PFI/SEAgent/policy oracle).**
4. Metric phân mảnh: ASR, ASR@k, ATR, SUR, PAR, UAR, JBR, GSR, FPR… không thống nhất → khó so sánh. Giao thức đánh giá (fresh vs stateful defender, attacker có history hay không) thay đổi kết quả mạnh (P9) nhưng hiếm khi được báo cáo.
5. Chưa có taxonomy hệ thống cho **privilege escalation tích luỹ qua nhiều lượt** (gradual escalation, memory persistence, approval fatigue, delegation chain drift, late-stage request).

---

## 2. Định vị & câu hỏi nghiên cứu

**Tiêu đề làm việc:** *SoK: Multi-Turn Privilege Escalation in LLM-Based Multi-Agent Systems — Taxonomy, Defenses, and Evaluation Gaps*
(giữ tiêu đề gốc "Evaluating …" nếu bổ sung phần đánh giá thực nghiệm đủ mạnh — xem §6.)

**Định nghĩa làm việc** (mở rộng từ P4):
> *Multi-turn privilege escalation* là chuỗi hành động của MAS qua ≥2 lượt tương tác (user↔agent, agent↔agent, hoặc qua memory/state bền vững) mà **tổng hợp** vượt quá least privilege cần cho ý định của user, trong khi **từng lượt riêng lẻ** có thể không vi phạm policy.

**Research Questions:**
- **RQ1 (Landscape):** Những vector tấn công privilege escalation nào trong MAS đã được nghiên cứu, và bao nhiêu trong số đó xét đến chiều thời gian (multi-turn / stateful)?
- **RQ2 (Taxonomy):** Có thể phân loại có hệ thống các cơ chế multi-turn privilege escalation theo những chiều nào?
- **RQ3 (Defenses):** Các phòng thủ hiện có (model-level, input-level, system-level) bảo đảm được thuộc tính gì, và bảo đảm đó có **compose theo agent** và **theo lượt** không?
- **RQ4 (Evaluation):** Benchmark và metric hiện tại đo được gì / bỏ sót gì đối với kịch bản multi-turn MAS?
- **RQ5 (Agenda):** Cần gì cho một phương pháp đánh giá chuẩn hoá?

---

## 3. Phương pháp Systematic Literature Review (PRISMA 2020)

### 3.1 Nguồn dữ liệu
- arXiv (cs.CR, cs.AI, cs.CL, cs.MA), ACM DL, IEEE Xplore, USENIX, NDSS, Semantic Scholar, DBLP
- Proceedings: IEEE S&P, USENIX Security, CCS, NDSS, ACL/EMNLP/NAACL, NeurIPS/ICLR/ICML (kể cả workshop an toàn AI)
- **Snowballing** xuôi/ngược từ 9 seed paper (danh mục tham khảo của P2, P4, P5, P8, P9 rất giàu: IsolateGPT, f-secure LLM, Progent, Conseca, AgentArmor, MAESTRO, G-Safeguard; về multi-turn: Crescendo, PAIR, TAP, Multi-turn Human Jailbreaks (MHJ); về MAS: Terrarium, Abdelnabi et al. 2025 (phòng thủ agent-to-agent), AgentBeats; về benchmark: AgentHarm, ToolEmu, WASP, AgentSafetyBench, R-Judge, …). Tất cả cần xác minh khi làm SLR.

### 3.2 Chuỗi truy vấn (khởi điểm, tinh chỉnh sau pilot)
```
("LLM" OR "large language model") AND ("agent" OR "multi-agent" OR "agentic")
AND ("privilege escalation" OR "confused deputy" OR "prompt injection"
     OR "memory poisoning" OR "RAG poisoning" OR "agent hijacking"
     OR "trust propagation" OR "excessive agency")
AND ("multi-turn" OR "multi-round" OR "conversation" OR "stateful"
     OR "long-horizon" OR "persistent memory" OR "delegation")
```
Chạy thêm biến thể không có khối multi-turn để đo tỉ lệ paper **có / không** xét đa lượt (dữ liệu cho RQ1).

### 3.3 Tiêu chí
- **Inclusion:** (a) hệ thống dựa trên LLM có dùng tool/agent; (b) có thành phần tấn công, phòng thủ, hoặc đánh giá liên quan đến quyền/ủy quyền/luồng tin cậy; (c) 2022 – 11/2026; (d) tiếng Anh; (e) có đánh giá thực nghiệm hoặc mô hình hình thức.
- **Exclusion:** jailbreak thuần nội dung (không liên quan tool/quyền); position paper không có đóng góp kỹ thuật; bản trùng lặp (giữ bản mới nhất).
- **Quality assessment** (thang 0–2 mỗi mục): threat model rõ ràng; có baseline; số model/seed; có adaptive attacker; reproducibility (code/data); có báo cáo CI/thống kê.

### 3.4 Quy trình
1. Pilot 30 paper → tinh chỉnh query & form trích xuất.
2. Hai người sàng lọc độc lập title/abstract → full text; đo **Cohen's κ**, giải quyết bất đồng bằng thảo luận.
3. Ghi lại PRISMA flow diagram (identified → screened → eligible → included).
4. Lưu tất cả trong spreadsheet/Zotero có tag.

### 3.5 Form trích xuất dữ liệu (mỗi paper)
| Trường | Ví dụ giá trị |
|---|---|
| Loại đóng góp | attack / defense / benchmark / formal model / empirical study |
| Kiến trúc MAS | single / hierarchical (agent-as-tool) / pipeline / P2P / broadcast |
| Kênh truyền | message, shared memory/state, RAG store, tool output, MCP |
| Số lượt tấn công | 1 / nhiều (cố định) / adaptive nhiều lượt |
| Persistence | không / trong phiên / xuyên phiên (memory) |
| Vector (theo P4) | direct PI, indirect PI, RAG poisoning, untrusted agent, confused deputy, collusion |
| Loại escalation | vertical / horizontal / confused deputy / colluding |
| Cơ chế phòng thủ | detection / model / input / system (isolation, IFC, capability, MAC/ABAC, signed token, policy oracle) |
| Bảo đảm | heuristic / deterministic / formal (CFI, noninterference, …) |
| Compose? | theo agent / theo lượt / không xét |
| Benchmark & metric | AgentDojo, MultiAgentDojo, InjecAgent, AgentBench, AWS MAS, … ; ASR/UAR/JBR/SUR/… |
| Model, seed, CI | … |

---

## 4. Khung phân loại (Taxonomy) — bản nháp để kiểm chứng

**Chiều A — Nguồn gốc kẻ tấn công:** user độc hại · dữ liệu ngoài (web, email, tài liệu) · tool/MCP server độc hại · agent bên thứ ba không tin cậy · memory/RAG bị đầu độc · agent nội bộ bị chiếm.

**Chiều B — Cơ chế lan truyền giữa agent:**
- *Boundary laundering* (P2): untrusted data → instruction trusted ở agent con
- *Confused deputy* (P4, P5): agent quyền thấp khiến agent quyền cao dùng quyền
- *Collusion*: hai agent ghép quyền
- *Forged approval / authority claim* (P5)
- *Data-only steering* (P3) / *branch steering* (P2)

**Chiều C — Chiều thời gian (đóng góp mới chính):**
| Mẫu | Mô tả | Ai đã chạm tới |
|---|---|---|
| C1 Gradual escalation | Mỗi lượt xin thêm một ít quyền/scope, từng bước hợp lệ | Crescendo (chatbot, không phải MAS); P8 (một phần, 2 agent) |
| C2 Persistent memory implant | Gieo ở lượt t, kích hoạt ở lượt t+k | P5 (memory poisoning, 1 lượt), P4 (SEMemory) |
| C3 Delegation drift | Ý định gốc bị biến dạng dần qua chuỗi ủy quyền nhiều tầng | P2 (một phần) |
| C4 Approval fatigue / consent laundering | Lợi dụng bước "hỏi user" lặp lại (PFI, SEAgent "prompt user") | chưa |
| C5 Context-window displacement | Đẩy system/policy prompt ra khỏi ngữ cảnh qua hội thoại dài | chưa trong MAS |
| C6 Retry / feedback leakage | Học policy qua lỗi lặp lại (P2 dẫn Garby et al.) | P2 (thảo luận), **P9 (attacker học từ phản hồi của defender qua 15 round)** |
| C7 Token/capability replay xuyên lượt | Dùng lại uỷ quyền cũ cho task mới | P5 (nonce, chưa đo) |
| C8 Late-stage request / sunk-cost pressure | Xây dựng hợp tác nhiều lượt rồi mới đưa yêu cầu độc hại, lợi dụng áp lực "hoàn thành task" | **P8** (quan sát định tính) |
| C9 Authority/institutional claim qua hội thoại | Agent ngoài viện dẫn "chính sách", "yêu cầu bắt buộc" không kiểm chứng được | **P8**, P5 (forged approval, 1 lượt), P9 (authority handling) |
| C10 Cross-session adaptive search | Attacker mở phiên mới mỗi lần, mang theo kinh nghiệm từ các phiên trước | **P9** |

**Chiều D — Điểm thực thi phòng thủ:** model · input · inter-agent channel · tool-call boundary · execution sandbox · audit/observer.

**Chiều E — Tác động:** exfiltration · unauthorized write/delete · lateral movement giữa agent · persistence.

→ Sản phẩm: bảng ma trận **Vector × Temporal pattern × Defense**, đánh dấu ô "chưa được nghiên cứu".

---

## 5. Phân tích phòng thủ (RQ3)

Bảng so sánh có hệ thống các phòng thủ (bắt đầu với CaMeL, multi-CaMeL, PFI, SEAgent, structural authorization của P5, + IsolateGPT, f-secure LLM, Progent, Conseca, AgentArmor, Llamafirewall… sau khi SLR xác nhận):

| Phòng thủ | Thuộc tính bảo đảm | Compose theo agent? | Compose theo lượt? | Điều kiện phá vỡ đã biết |
|---|---|---|---|---|
| CaMeL | CFI + capability policy | **Không** (P2) | Không xét (plan cố định) | boundary laundering, implicit flow |
| multi-CaMeL | CFI xuyên agent | Có (cây) | Chưa | branch steering, topology không phải cây |
| PFI | isolation + data ID + guardrail | Chưa xét | Chưa | endorsement do user approve |
| SEAgent | MAC/ABAC trên information-flow graph | Có (cross-agent tracing) | Có thiết kế (SEMemory), **chưa đánh giá bảo mật** | labeling sai, policy thiếu |
| Signed token + policy oracle (P5) | complete mediation tại executor | Có (pipeline) | Chưa | attacker có secret → chỉ còn oracle; policy do tác giả viết |

Câu hỏi phân tích: một thuộc tính "per-step safe" có kéo theo "trajectory safe" không? (Gợi ý hình thức hoá: safety như tính chất trên **trace**, không phải trên từng action — liên hệ temporal logic / hyperproperty / noninterference.)

---

## 6. Phần đánh giá có hệ thống (để paper xứng với chữ "Evaluating")

Quy mô vừa phải, mục tiêu là **kiểm chứng taxonomy**, không phải đề xuất phòng thủ mới.

- **Testbed:** tái sử dụng MultiAgentDojo (P2) và/hoặc pipeline 4-agent kiểu P5 (LangGraph); thêm topology P2P (theo AWS benchmark mở rộng của P4).
  - Mượn **hội thoại agent↔agent** và kịch bản tấn công theo ngữ cảnh của ConVerse (P8), cắm vào MAS ≥3 agent có tool thật.
  - Mượn **vòng lặp attacker thích ứng** của Security Arena (P9, code công khai) để sinh tấn công đa lượt thay vì dùng payload cố định.
  - Đóng góp đánh giá: lấp đúng giao điểm còn trống ở gap #3 (multi-turn + MAS + tool thật + phòng thủ system-level).
- **Kịch bản:** với mỗi mẫu C1–C10, xây 1–2 kịch bản đa lượt, đặt cạnh phiên bản **single-turn tương đương** (cặp đối chứng).
- **Giao thức phải báo cáo rõ** (bài học từ P9): defender fresh-session hay stateful; attacker có giữ history/feedback không; ngân sách số round; dừng ở lần thành công đầu tiên hay không.
- **Điều kiện phòng thủ:** none · model-level (system prompt/spotlighting) · CaMeL riêng lẻ · multi-CaMeL · policy oracle/MAC (nếu chạy được mã nguồn công khai của CaMeL, PFI).
- **Model:** ≥3 họ model, nhiều mức năng lực; ≥3 seed; temperature 0 + ghi log đầy đủ.
- **Metric thống nhất** (đề xuất trong paper):
  - **ASR / UAR** — hành động trái phép thực sự xảy ra (đo bằng sandbox như P5)
  - **ASR@k** — tỉ lệ thành công trong k round (P9); báo cáo cả ASR@1 để so với single-turn
  - **JBR** — agent bị lừa dù hành động bị chặn (P5)
  - **Turns-to-Compromise (TTC)** — số lượt tới khi vi phạm *(mới)*
  - **Cumulative Privilege Exceedance** — tổng scope vượt least privilege trên cả trajectory *(mới)*
  - **SUR / utility, FPR/LTC** — chi phí tiện dụng
  - Thống kê: Wilson CI cho tỉ lệ gần 0 (theo P2), t-interval theo seed (theo P5)
  - Với rò rỉ dữ liệu: phân tầng privacy theo P8 (unrelated / related-but-private / related-and-useful)
- **Câu hỏi kiểm chứng chính:** Phòng thủ đạt ASR≈0 ở single-turn có giữ được khi chuyển sang multi-turn không? P9 đã cho thấy điều này **không** đúng với defender chỉ dựa vào LLM (0–1% → 7.9–16.8%); câu hỏi mở là phòng thủ **deterministic/system-level** (CaMeL, multi-CaMeL, PFI, SEAgent, policy oracle) có giữ được không.

> Nếu thời gian/tài nguyên hạn chế: bỏ phần thực nghiệm, đổi tiêu đề thành "SoK: …", và thay bằng **case study phân tích** (walk-through) cho từng mẫu C1–C7 dựa trên mô hình hình thức của P2/P4.

---

## 7. Dàn ý paper

1. **Introduction** — động cơ, gap (single-turn bias, non-composition), đóng góp
2. **Background** — LLM agent, MAS topology, privilege escalation cổ điển (vertical/horizontal/confused deputy/collusion), Dual-LLM/CaMeL, PoLP
3. **Formal model** — mở rộng mô hình của P4 và Định nghĩa 1 của P2 với chiều thời gian (trace của hệ thống); định nghĩa multi-turn privilege escalation
4. **Methodology (SLR)** — PRISMA, query, tiêu chí, κ, quality assessment
5. **Landscape (RQ1)** — thống kê: theo năm, venue, kiến trúc, tỉ lệ có xét multi-turn
6. **Taxonomy (RQ2)** — các chiều A–E, ví dụ minh hoạ
7. **Defenses (RQ3)** — bảng so sánh, phân tích composition theo agent và theo lượt
8. **Evaluation practices (RQ4)** — benchmark, metric, threats-to-validity phổ biến (1 model, conspicuous injection, policy do tác giả tự viết…)
9. **Systematic evaluation** (§6) — kết quả cặp single vs multi-turn
10. **Research agenda (RQ5)** — benchmark multi-turn MAS, metric thống nhất, bảo đảm trên trajectory, adaptive attacker
11. **Threats to validity** của chính SoK
12. **Conclusion**

Venue mục tiêu (cân nhắc): IEEE S&P (SoK track), USENIX Security, EuroS&P (SoK), ACM Computing Surveys (nếu thuần review), hoặc workshop AI safety/security.

---

## 8. Lộ trình (giả định ~16 tuần, bắt đầu 01/12/2026)

| Tuần | Công việc | Sản phẩm |
|---|---|---|
| 1–2 | Đọc sâu 5 seed paper, chốt định nghĩa & RQ, viết protocol SLR (có thể đăng ký trước trên OSF) | Protocol v1 |
| 3–4 | Pilot search, tinh chỉnh query, form trích xuất | Query cuối, form |
| 5–7 | Search đầy đủ + snowballing, sàng lọc 2 vòng, κ | PRISMA diagram, danh sách included |
| 8–9 | Trích xuất dữ liệu, quality assessment | Bảng dữ liệu |
| 10–11 | Xây & kiểm chứng taxonomy, bảng phòng thủ | §6, §7 bản nháp |
| 11–14 | Thực nghiệm (§6) song song | Kết quả, log, code |
| 14–15 | Viết toàn bộ bản thảo | Draft v1 |
| 16 | Review nội bộ, chỉnh sửa, chuẩn bị artifact (code + dữ liệu SLR) | Bản nộp |

---

## 9. Rủi ro & giảm thiểu
- **Lĩnh vực thay đổi rất nhanh** (nhiều paper 2026) → đặt ngày cắt (cut-off) rõ ràng, chạy lại search trước khi nộp.
- **Ít paper đúng "multi-turn + MAS"** → đó chính là phát hiện; mở rộng phạm vi sang single-agent multi-turn và MAS single-turn rồi phân tích giao.
- **Thực nghiệm tốn chi phí API** → giới hạn số kịch bản, ưu tiên model mã nguồn mở + 1–2 model frontier.
- **Đạo đức:** chỉ chạy trong sandbox, dữ liệu công khai (vd. Enron như P5); không công bố payload tấn công có thể dùng ngay ngoài thực tế ở dạng tối ưu hoá; responsible disclosure nếu phát hiện lỗ hổng trong framework thực.

---

## 10. Việc cần làm ngay
- [ ] Chốt: SoK thuần hay SoK + thực nghiệm (§6)?
- [ ] Tạo thư viện Zotero, nhập 9 seed paper + toàn bộ reference list của P2, P4, P5, P8, P9
- [ ] Viết protocol SLR (§3) thành file riêng
- [ ] Kiểm tra mã nguồn công khai: CaMeL (`google-research/camel-prompt-injection`), PFI (`compsec-snu/pfi`), AgentDojo (`agentdojo.spylab.ai`), InjecAgent (`uiuc-kang-lab/InjecAgent`), ConVerse (`amrgomaaelhady/ConVerse`), Security Arena (`sofasogood/adaptive-adversaries`), MultiAgentDojo/SEAgent (xác minh có công bố không)
- [ ] Đọc Crescendo, MHJ, Terrarium và Abdelnabi et al. 2025 (đều được P8/P9 trích dẫn) để bổ sung phần multi-turn và phòng thủ agent-to-agent
- [ ] Tìm người sàng lọc thứ hai (để tính κ)
