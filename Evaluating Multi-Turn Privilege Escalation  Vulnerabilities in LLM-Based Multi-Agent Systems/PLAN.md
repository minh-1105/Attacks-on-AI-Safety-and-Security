# Kế hoạch paper: Evaluating Multi-Turn Privilege Escalation Vulnerabilities in LLM-Based Multi-Agent Systems

> **Trạng thái:** Research planning only — chưa có kết quả thực nghiệm, **chưa xác nhận novelty**.
> **Hướng đề xuất:** Focused Systematic Literature Review (SLR) + Controlled Empirical Security Evaluation (xem §1.1).
> **Ngày lập:** 2026-11-27 · **Cập nhật lần 3:** 2026-11-27 theo `additional.md` (xem Changelog §15).
> **Bối cảnh:** paper đầu tiên của tác giả → kế hoạch phải đủ cụ thể để làm từng bước, và có điểm dừng (exit gate) để thu hẹp phạm vi khi cần.

### Quy ước nhãn bằng chứng (dùng xuyên suốt)

| Nhãn | Nghĩa |
|---|---|
| **[V]** verified evidence | Đã đối chiếu với PDF trong `docs/` (có số trang) hoặc trang arXiv abs (ghi ngày kiểm tra) |
| **[V-abs]** | Chỉ mới kiểm tra qua abstract/trang giới thiệu, **chưa đọc toàn văn** |
| **[G]** candidate gap | Khoảng trống giả định, cần kiểm chứng bằng SLR |
| **[D]** proposed design | Đề xuất thiết kế, chưa được người dùng phê duyệt |
| **[H]** hypothesis | Giả thuyết sẽ kiểm định, chưa có kết quả |
| **[F]** future work / stretch | Ngoài phạm vi pilot |
| **TBD** | Needs verification — đưa vào checklist §14 |

Số trang dạng `p.N` là trang của file PDF trong `docs/` (bản arXiv ghi ở §2.1), không phải số trang in.

---

## 1. Định vị nghiên cứu

**Working title (giữ):** *Evaluating Multi-Turn Privilege Escalation Vulnerabilities in LLM-Based Multi-Agent Systems*
**Alternative title (cân nhắc sau khi có kết quả):** *From Local Safety to Trajectory Security: Evaluating Multi-Turn Privilege Escalation in LLM-Based Multi-Agent Systems*

### 1.1 Research strategy [D]

**Focused SLR + Controlled Empirical Security Evaluation.**
- SLR dùng để xây cơ sở lý thuyết, xác nhận gap, tổng hợp taxonomy và định hướng thiết kế benchmark.
- **Empirical evaluation là đóng góp chính**, nếu thực nghiệm khả thi (exit gate tuần 8, §11).
- Không đặt mục tiêu làm cùng lúc: SoK toàn diện + formal proof mới + benchmark lớn + defense framework mới.

**SoK chỉ là fallback có điều kiện, và nay yếu hơn trước.** Đã có ít nhất hai bài tổng hợp rộng gần với hướng này [V-abs, §3]: "SoK: When Safe Agents Fail Together" (197 công trình về MAS security) và "Toward Secure LLM Agents" (247 paper). Nếu bỏ thực nghiệm thì phải:
- đổi RQ, title, contributions và phương pháp tổng hợp;
- thu hẹp thành **focused review** về authorization theo thời gian, không làm SoK rộng;
- không gọi case study minh hoạ là empirical validation.

### 1.2 Central research problem

Trọng tâm: **trajectory-level authorization integrity** trong MAS có trạng thái (stateful). Tức là tính nhất quán giữa quyền được cấp hợp lệ, delegation, trạng thái và hành động **qua toàn bộ phiên**. Một hệ thống có thể kiểm tra quyền cục bộ (từng bước) mà vẫn không thoả ràng buộc ở cấp toàn nhiệm vụ.

> **Câu hỏi trung tâm [H]:** *When and under what conditions do local authorization checks fail to ensure trajectory-level authorization integrity in LLM-based multi-agent systems?*

Đây là câu hỏi nghiên cứu, không phải kết luận. **Không mặc định** rằng multi-turn hay multi-agent làm tăng rủi ro.
- Bằng chứng gián tiếp đã có [V]: P9 cho thấy chấm nhiều round làm ASR tăng so với chỉ chấm round 1 (p.1, p.6).
- Bằng chứng ngược chiều cũng đã có [V]: trong P9, defender có memory lại *giảm* ASR, từ 57.4% xuống 31.5% (p.6, p.17).

### 1.3 Khái niệm cần phân biệt

| Concept | Working meaning |
|---|---|
| Authentication | Xác định danh tính của principal |
| Authorization | Xác định hành động, resource và điều kiện được phép |
| Agent capability | Khả năng kỹ thuật / quyền tối đa của agent (vd. role có tool `update_record`) |
| Task authorization | Phạm vi quyền hợp lệ của **nhiệm vụ cụ thể** (có thể hẹp hơn capability) |
| Privilege acquisition / escalation | Đạt được quyền hiệu lực vượt phạm vi được cấp |
| Privilege misuse | Dùng khả năng hiện có ngoài mục đích/phạm vi được uỷ quyền |
| Unauthorized action | Hành động không thoả chính sách hợp lệ **tại thời điểm thực hiện** |
| Prompt injection | Một *cơ chế* tác động hành vi; **không đồng nhất** với privilege escalation |

**Quy ước:**
- P4 dùng *privilege escalation* theo nghĩa rộng: "agent actions exceeding the least privilege required for a user's intended task" [V, P4 p.1]. Paper này **tách riêng** ba thứ: acquisition, misuse và unauthorized effects. Khi dùng nghĩa rộng thì ghi rõ là đang theo quy ước của P4.
- Một thay đổi quyền **có phê duyệt hợp lệ** không bị coi là vi phạm.

---

## 2. Seed corpus (9 paper trong `docs/`)

### 2.1 Metadata (đối chiếu PDF local, 2026-11-27)

| ID | Short name | Title (theo PDF) | arXiv (bản local) | Venue | Code / license | Trạng thái |
|---|---|---|---|---|---|---|
| P1 | CaMeL | Defeating Prompt Injections by Design | 2503.18813**v2** (24/06/2025) | preprint — TBD | `github.com/google-research/camel-prompt-injection` (p.1); license TBD | [V] metadata |
| P2 | multi-CaMeL | Can CaMeLs Talk? Securing Multi-Agent Systems Against Indirect Prompt Injection Attacks | 2610.05640v1 (05/10/2026) | preprint | TBD | [V] |
| P3 | PFI | Prompt Flow Integrity to Prevent Privilege Escalation in LLM Agents | 2503.15547v2 (21/04/2025) | preprint — TBD | `github.com/compsec-snu/pfi` (p.2); license TBD | [V] |
| P4 | SEAgent | Taming Various Privilege Escalation in LLM-Based Agent Systems: A Mandatory Access Control Framework | 2601.11893v1 (17/01/2026) | preprint (ACM template placeholder) | TBD | [V] |
| P5 | Trust Propagation | Trust propagation and structural containment in Multi-agent LLM pipelines | 2609.17648v1 (15/09/2026) | preprint | TBD | [V] |
| P6 | AgentDojo | AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents | 2406.13352v3 (24/11/2024) | **NeurIPS 2024 Datasets & Benchmarks** (p.1) | `github.com/ethz-spylab/agentdojo`, **MIT** theo paper (checklist, trích từ PDF) | [V] |
| P7 | InjecAgent | InjecAgent: Benchmarking Indirect Prompt Injections in Tool-Integrated LLM Agents | 2403.02691v3 (04/08/2024) | **TBD** (PDF không ghi venue) | `github.com/uiuc-kang-lab/InjecAgent` (p.1); license TBD | [V] |
| P8 | ConVerse | ConVerse: Benchmarking Contextual Safety in Agent-to-Agent Conversations | 2511.05359v1 (07/11/2025) | preprint — TBD | `github.com/amrgomaaelhady/ConVerse` (p.1, ghi "will be available"); license TBD | [V] |
| P9 | Adaptive Adversaries | Adaptive Adversaries: A Multi-Turn, Multi-LLM Benchmark for LLM Agent Security | 2607.18063v2 (11/09/2026) | preprint | `github.com/sofasogood/adaptive-adversaries` (p.1); license TBD | [V] |

**Ghi chú xác minh:**
- **Số liệu của CaMeL (77% hay 67%):** đã giải quyết [V].
  - Bản **v2** (file local, p.1) ghi *"solving 77% of tasks with provable security (compared to 84% with an undefended system) in AgentDojo"*.
  - Bản **v1** (arXiv abs, kiểm tra 2026-11-27) ghi *"solving 67% of tasks with provable security"*.
  - → Hai con số đến từ hai phiên bản khác nhau. Kế hoạch dùng **77% (v2)** và luôn ghi kèm phiên bản. Mẫu số, model và điều kiện cụ thể cần đọc ở phần evaluation của v2: **TBD**.
- **InjecAgent:** bản kế hoạch trước ghi "ACL Findings 2024" nhưng **PDF không xác nhận** → đã gỡ, chuyển thành TBD.
- Code/license của các repo chưa kiểm tra trực tiếp trên GitHub (chỉ lấy từ paper). Cần ghi commit hash khi dùng.

### 2.2 Nội dung — tấn công / phòng thủ / nghiên cứu thực nghiệm (P1–P5)

| # | Ý tưởng cốt lõi [V] | Đánh giá [V] | Khoảng trống liên quan (reviewer observation) |
|---|---|---|---|
| P1 | Tách control flow (P-LLM sinh chương trình) khỏi data flow (Q-LLM); capability + security policy tại tool-call (p.1) | AgentDojo: 77% task với provable security vs 84% không phòng thủ (v2, p.1) | Single-agent; plan được cố định từ query ban đầu |
| P2 | **Boundary laundering**: data untrusted ở agent cha thành instruction trusted ở agent con → bảo đảm CaMeL không compose (p.2–3); multi-CaMeL tách kênh instruction/data | MultiAgentDojo + AssetOpsBench, 5 LLM; ASR 12.9% → 0.2% (CaMeL riêng lẻ) → 0.0% (multi-CaMeL) (p.1, Table III p.10) | Tác giả tự nêu (§VI): branch steering, implicit flow, retry-loop leak; topology dạng cây agent-as-tool; thí nghiệm chạy CaMeL **không có security policy** |
| P3 | Agent isolation (trusted/untrusted), data ID, privilege-escalation guardrail; phân biệt prompt injection với data-only attack (p.1–2) | SUR 27.84% → 55.67% (AgentDojo), 2.63% → 67.79% (AgentBench OS) (p.2); ATR = 0 | Single-agent; việc endorse dữ liệu cần user approve (P3, đoạn so sánh với f-secure LLM; section/trang chính xác TBD) → TBD: rủi ro approval lặp lại |
| P4 | Định nghĩa PE theo least privilege; 5 vector gồm confused deputy trong MAS; SEAgent (MAC/ABAC trên information-flow graph); SEMemory cho multi-round (p.1–2) | InjecAgent, AgentDojo (bảo mật, ASR 0%); API-Bank, AWS (utility) | Ở đánh giá MAS (AWS benchmark), tác giả **bỏ qua labeling và policy enforcement** vì tool chỉ được mô phỏng (p.12) → bảo mật multi-round trong MAS chưa được đo trong paper này |
| P5 | Pipeline 4 agent (LangGraph); memory poisoning + forged approval; HMAC token + policy oracle; **JBR** (p.1–3) | 60 task, 3 seed, 1 model; poisoning: JBR 100% nhưng UAR 0% khi có authorization (p.1, Table I) | Tác giả tự nêu (§V-E, p.5): policy do chính nhóm viết, injection lộ liễu, chỉ 1 kiến trúc/1 model, ablation 1 seed; tấn công một lượt |

### 2.3 Nội dung — benchmark (P6–P9)

| # | Thiết lập [V] | Quy mô [V] | Kết quả chính [V] | Khoảng trống (reviewer observation) |
|---|---|---|---|---|
| P6 AgentDojo | Single-agent, tool-use, injection trong tool output; môi trường mở rộng được | 97 task, 629 security test case (p.1) | LLM mạnh trượt nhiều task kể cả khi không có tấn công (p.1) | Một agent; ConVerse mô tả AgentDojo là "single agent and a single turn setup" (P8 p.2) |
| P7 InjecAgent | Single-agent, indirect PI; direct harm và data exfiltration | 1.054 case, 17 user tool, 62 attacker tool, 30 agent (p.1) | ReAct-GPT-4 bị tấn công thành công 24%; thêm hacking prompt thì gần gấp đôi (p.1–2) | Tĩnh, một lượt |
| P8 ConVerse | Assistant ↔ external agent, **multi-turn**; tấn công đặt trong hội thoại hợp lý | 3 domain, 12 persona, 864 attack (611 privacy / 253 security), 7 model (p.1–2) | Privacy ASR 37–88%, security 2–60% (p.1–2); "Related and Useful" lộ 90–94%; yêu cầu vi phạm privacy xuất hiện muộn trong hội thoại (p.13) | Tác giả tự nêu (§8, p.9): chỉ 2 agent; môi trường giả lập; assistant không kiểm chứng được claim; attack do LLM sinh |
| P9 Adaptive Adversaries | Attacker LLM thích ứng qua round; defender **fresh-session** mỗi round | 21 scenario; 3×3 = 945 battle; competition 18.422 battle (p.1) | ASR round 1: 0–1% → ASR@15: 7.9 / 9.5 / 16.8% theo defender (p.1, p.4, p.6); provenance paragraph: 110/270 → 70/270 (p.1); trạng thái defender: fresh 57.4% → stateful 31.5% → stateful + attacker biết có history 14.8% (p.6, p.17) | Tác giả tự nêu (§6, p.7): **không có tools/retrieval/thay đổi trạng thái bên ngoài**, chỉ chấm output; một defender, không phải MAS |

---

## 3. Near-neighbor work — ảnh hưởng tới novelty

Đã kiểm tra sự tồn tại qua trang arXiv abs / trang dự án ngày 2026-11-27. **Chỉ mới đọc abstract [V-abs]**, chưa đưa vào included studies.

| Candidate | Metadata [V-abs] | Nội dung theo abstract | Mức chồng lấn với hướng này |
|---|---|---|---|
| **SoK: When Safe Agents Fail Together: The Security of Multi Agent LLM Systems** | arXiv 2609.00595, Yang et al., 01/09/2026, 21 trang | Phân tích theo execution 197 công trình; framework A-I-R (adversary position, interaction interface, risk); audit 44 công trình evaluation/benchmark; nêu thách thức "isolating interaction effects", "comparable and diagnostic metrics", counterfactual | **Cao với phương án SoK** → SoK rộng gần như không còn chỗ. Đồng thời *ủng hộ* hướng controlled evaluation (họ kêu gọi counterfactual) |
| **Conflux: Preventing Privilege Escalation with Principal Contexts and System-Level Evaluation** | arXiv 2611.08768, Buckley & Schroeder de Witt, 05/11/2026 | Hình thức hoá PE ở cấp hệ thống (mọi principal ảnh hưởng tới action phải cho phép); định lý authority-confinement **có điều kiện**; evaluator SLED / SLED-V | **Cao với phần formal model** → không tuyên bố formalization mới; nên dùng/so sánh định nghĩa của họ. Có liên quan delegation |
| **Delegation Without Trust: An Empirical Gap Analysis of Identity, Authorization, and Runtime Governance in Multi-Agent LLM Systems** | arXiv 2609.00267, Dantuluri & Sundi, 31/08/2026 | Threat model delegation (confused deputy, token theft/replay, PI privilege escalation, compromised sub-agent); 8 yêu cầu; phân tích LangGraph/CrewAI/AutoGen/MCP; authorization broker | **Trung bình–cao** với delegation consistency (C3) và token replay (C7) |
| **Toward Secure LLM Agents: Threat Surfaces, Attacks, Defenses, and Evaluation** | arXiv 2606.10749, Ling et al., 09/06/2026 | Survey 247 paper; nhận xét: defenses "weakly compositional", benchmarks "underrepresent long-horizon, stateful" risks | **Trung bình** → gap "thiếu đánh giá long-horizon/stateful" **đã được người khác nêu**; paper này không được trình bày nó như phát hiện mới |
| **AgentLAB: Benchmarking LLM Agents against Long-Horizon Attacks** | Trang dự án `tanqiujiang.github.io/AgentLAB_main/` (Jiang et al., Stony Brook); arXiv ID **TBD** | 5 họ tấn công long-horizon (intent hijacking, **tool chaining**, task injection, objective drifting, memory poisoning); 28 môi trường; 644 case; kết luận defense cho single-turn không chống được long-horizon | **Cao với C1/C2** và với ý "từng bước an toàn, kết hợp lại có hại" (tool chaining ≈ LocalSafe ⊭ TrajectorySafe). **TBD:** có MAS không? có authorization oracle không? |

**Hệ quả cho kế hoạch [D]:**
1. Không tuyên bố "first benchmark for multi-turn / long-horizon agent attacks" (AgentLAB tự nhận điều này).
2. Không tuyên bố "first formal definition of privilege escalation" (P4, Conflux).
3. Không tuyên bố "first SoK of MAS security" (2609.00595).
4. Novelty còn lại có thể có, **sau khi đọc toàn văn 5 bài trên**:
   - so sánh **có đối chứng 2×2** (turn × architecture) trên cùng task, cùng policy;
   - **oracle authorization độc lập**, đo committed effect thật trong sandbox;
   - đánh giá **phòng thủ deterministic theo lượt**.
   Nếu một trong năm bài đã làm điều này → chuyển sang replication / measurement refinement / một điều kiện chưa được đánh giá.

---

## 4. Research gap candidate [G]

Thay các tuyên bố tuyệt đối ở bản trước bằng nhận định có điều kiện:

> Existing studies address agent security [P1–P7], multi-turn and long-horizon interaction [P8, P9, AgentLAB], and delegated authorization in MAS [P2, P4, P5, Delegation Without Trust, Conflux]. This study investigates whether **controlled comparisons of trajectory-level authorization integrity across interaction modes and agent architectures**, scored by an **independent, policy-based oracle on committed effects**, reveal an underexplored evaluation gap.

Các quan sát hỗ trợ (đều có nguồn):
- P9 chỉ chấm output và không có tool (p.7). P8 chỉ có 2 agent (p.9). P4 bỏ qua policy enforcement trong đánh giá MAS (p.12). → **[V]** về giới hạn của từng bài. **Không** suy ra "chưa ai làm".
- Kết quả phụ thuộc mạnh vào giao thức đánh giá (fresh hay stateful defender) [V, P9 p.6] → thiết kế phải cố định và báo cáo giao thức.

**Cần kiểm tra trước khi chốt gap** (checklist §14):
- ConVerse có chấm authorization ở cấp trajectory không? Quan sát sơ bộ: security được chấm theo kết quả tấn công (upselling, DoS, email fraud) bằng LLM judge, không phải policy oracle → **TBD, cần đọc §3–5 của P8**.
- AgentLAB có MAS, defense system-level, oracle trên effect không?
- Các SoK/survey ở §3 đã có taxonomy temporal tương tự C1–C10 chưa?
- Conflux, Delegation Without Trust đã biểu diễn quyền theo thời gian, delegation, revocation chưa?

---

## 5. Research Questions, hypotheses và traceability [D]

5 RQ cũ được gộp thành 3 RQ chính. Landscape → RQ1. Benchmark practices → subquestion của RQ1. Research agenda → chuyển sang Discussion.

| RQ | Research question | Method | Expected artifact | Mục |
|---|---|---|---|---|
| **RQ1** | How can authorization-related failures in stateful LLM-based MAS be systematically characterized? (sub: which evaluation practices/metrics are used, and what do they omit?) | Focused SLR + taxonomy | Literature matrix, codebook, evidence-backed taxonomy | §6, §8 |
| **RQ2** | How do interaction mode and agent architecture affect trajectory-level authorization integrity? | Controlled 2×2 experiment | Comparative results + uncertainty estimates | §9.6–9.8 |
| **RQ3** | How do selected authorization mechanisms affect security and legitimate task utility across turns? | Comparative evaluation | Security/utility trade-off, limitations | §9.6 (defense), §9.7 |

**Hypotheses [H]** (chốt sau pilot, **trước** main runs; ghi vào `DECISIONS.md`):
- **H1:** interaction mode (single vs multi-turn) ảnh hưởng tới xác suất unauthorized execution. *H1₀: không có khác biệt.*
- **H2:** có interaction effect giữa architecture và interaction mode. *H2₀: hiệu ứng của turn như nhau ở single-agent và MAS.*
- **H3:** các cơ chế authorization được chọn có kết quả security/utility khác nhau. *H3₀: không khác biệt.*
- Shared state/memory là **biến ablation riêng** nếu đủ nguồn lực [F]. Không gộp với multi-turn.

**Phải định nghĩa trước main experiment:**
- **Unit of analysis:** run (một phiên của một scenario trong một condition).
- **Primary outcome:** run-level Unauthorized Execution Rate (§9.7).
- **Effect size:** hiệu tỉ lệ (risk difference) và odds ratio, kèm CI.
- **Quy tắc kiểm định:** §9.7.

Mọi phân tích ngoài kế hoạch đã chốt phải ghi rõ là **exploratory**.

---

## 6. SLR protocol (RQ1)

PRISMA 2020 là **reporting guideline**. Protocol review đi theo Kitchenham & Charters (software engineering) [D]. Viết protocol thành `systematic_review/protocol.md` và có thể đăng ký trước trên OSF.

### 6.1 Nguồn
- arXiv (cs.CR, cs.AI, cs.CL, cs.MA), ACM DL, IEEE Xplore, USENIX, NDSS, Semantic Scholar, DBLP.
- Proceedings: IEEE S&P, USENIX Security, CCS, NDSS, ACL/EMNLP/NAACL, NeurIPS/ICLR/ICML (kể cả workshop).
- **Snowballing** xuôi/ngược từ 9 seed paper và 5 near-neighbor (§3). Các tên được trích dẫn trong seed paper đều là ứng viên **chưa xác minh**:
  - phòng thủ: IsolateGPT, f-secure LLM, Progent, Conseca, AgentArmor;
  - threat modeling / MAS: MAESTRO, G-Safeguard, Terrarium, Abdelnabi et al. 2025, AgentBeats;
  - multi-turn: Crescendo, PAIR, TAP, MHJ;
  - benchmark: AgentHarm, ToolEmu, WASP, AgentSafetyBench, R-Judge.

### 6.2 Nhóm truy vấn độc lập (tránh quá nhiều AND)
```text
Q1: ("LLM" OR "large language model") AND ("multi-agent" OR "agent-to-agent") AND ("security" OR "authorization" OR "prompt injection")
Q2: ("LLM agent" OR "agentic AI") AND ("privilege escalation" OR "confused deputy" OR "access control" OR "authorization integrity")
Q3: ("LLM agent" OR "multi-agent") AND ("multi-turn" OR "multi-round" OR "stateful" OR "long-horizon" OR "persistent memory") AND ("security" OR "attack")
Q4: ("LLM" OR "AI agent") AND ("trust propagation" OR "delegation" OR "capability security" OR "trust boundary")
```
- **Pilot ~30 records:**
  - thử biến thể query theo từng database;
  - lưu `search_queries.md`: exact query, ngày, filter, số records thực tế, lý do chỉnh;
  - kiểm tra **recall trên seed**: các query có tìm lại được P1–P9 và 5 near-neighbor không.
- **Tách 2 loại bằng chứng:**
  - *primary evidence:* giao của MAS × authorization × multi-turn;
  - *adjacent evidence:* single-agent, single-turn, hoặc access control nói chung.
- **Khi báo cáo tỉ lệ paper có xét multi-turn:** mẫu số là tập included của Q1 ∪ Q2 ∪ Q4. **Không** dùng Q3 (vốn bắt buộc có multi-turn) để ước lượng tỉ lệ toàn lĩnh vực.

### 6.3 Tiêu chí
- **Inclusion:**
  - (a) hệ thống dựa trên LLM có dùng tool/agent;
  - (b) có thành phần attack, defense, evaluation hoặc formal model liên quan đến quyền, uỷ quyền hoặc luồng tin cậy;
  - (c) cutoff 01/2022 – 30/11/2026 (nếu cập nhật trước submission thì lưu phiên bản search và cutoff mới);
  - (d) tiếng Anh;
  - (e) có đánh giá thực nghiệm hoặc mô hình hình thức.
- **Exclusion:**
  - jailbreak thuần nội dung (không liên quan tool/quyền);
  - position paper không có đóng góp kỹ thuật;
  - bản trùng lặp.
- **Phiên bản:**
  - phân biệt preprint và peer-reviewed;
  - nếu có nhiều phiên bản, chọn bản phù hợp nhất với câu hỏi (thường là peer-reviewed). Không mặc định bản mới nhất là tốt nhất;
  - ghi rõ phiên bản đã dùng (bài học từ CaMeL v1/v2, §2.1).

### 6.4 Sàng lọc
- Lưu `screening.csv`: quyết định, lý do loại, giai đoạn, reviewer. Ghi PRISMA counts thực tế.
- **Nếu có reviewer thứ hai độc lập:** báo cáo agreement và Cohen's κ theo quy trình thực tế.
- **Nếu chỉ có một reviewer:** ghi rõ là limitation và cân nhắc nhờ người khác audit độc lập khoảng 10–20% mẫu.
- **Không bịa reviewer hay κ.**

### 6.5 Quality assessment theo loại nghiên cứu (`quality_assessment.csv`)

| Loại | Tiêu chí |
|---|---|
| Empirical | Có control/baseline; số scenario; số model; số repetitions; báo cáo uncertainty; reproducibility (code, data) |
| Formal | Assumptions; definitions; proof scope (điều kiện, fragment được bao phủ) |
| Benchmark | Task validity; oracle (rule-based hay LLM judge); coverage; reproducibility |

Thiếu thông tin thì ghi `not reported` / `N/A`, **không** đánh đồng với `No`.

### 6.6 Literature matrix / reading notes (`literature/literature_matrix.csv`, `literature/notes/`)

**Mỗi paper ghi các trường sau:**
- **Định danh:** ID, title, venue/version, link.
- **Nội dung:** research problem, contributions.
- **Threat model:** threat model, trusted components.
- **Kiến trúc:** topology, số agent, channel.
- **Chiều thời gian:** số external turn, số inter-agent steps/messages, state/persistence.
- **Bảo mật:** authorization model, security property, attack category, defense.
- **Đánh giá:**
  - benchmark;
  - metric (**công thức + mẫu số**);
  - models, repetitions, results.
- **Đánh giá chất lượng:** limitations — tách *author-reported* và *reviewer observation*.
- **Khác:** code/license, relevance, **page/section nguồn**, verification status.

**Cách đọc:** ba lượt — skim (abstract/intro/conclusion) → understand (method/threat model/experiments) → critical (assumptions, code, validity, khả năng replicate).

**Thứ tự đọc đề xuất:**
1. AgentDojo
2. InjecAgent
3. CaMeL
4. PFI
5. SEAgent
6. multi-CaMeL
7. Trust Propagation
8. ConVerse
9. Adaptive Adversaries
10. **AgentLAB, Conflux, Delegation Without Trust, SoK 2609.00595** — gần novelty nhất nên có thể đọc sớm hơn (sau bước 3).

### 6.7 Codebook taxonomy
- Mỗi category có: định nghĩa, coding rule, ví dụ dương/âm.
- Cho phép một paper mang nhiều nhãn.
- Ghi vị trí bằng chứng (page/section).
- Lưu version history trong `research_design/taxonomy.md`.

---

## 7. Formal model và threat model [D]

Formal hoá **vừa đủ để đo lường**. Phần này không nhằm có định lý mới. Cần đối chiếu với P2 (Definition 1, p.3), P4 (§3–4) và Conflux (§3) trước khi chốt ký hiệu, để không định nghĩa lại cái đã có.

### 7.1 Candidate model
- **System** `S = (A, R, P, M, T, δ)` gồm: agents, resources, policies, memory, tools, transition function.
- **State** `s_t = (M_t, P_t, E_t)` gồm: memory, effective authorization state, environment state.
  - Ghi riêng **trusted authorization state** (do runtime giữ) và state do agent **tự báo cáo**.
- **Trajectory** `τ = (s_0, a_1, s_1, …, a_n, s_n)`.
  - Mỗi event có: actor, action, resource, task ID, delegation provenance, external turn, internal step, outcome.
- `LocalSafe(a_t, s_{t-1})`: event pass theo local check đã chỉ định (vd. role-level check).
- `TrajectorySafe(τ, G)`: toàn bộ trace thoả global policy `G`, gồm phạm vi task, vòng đời authorization (grant / expire / revoke) và các ràng buộc tích luỹ đã định nghĩa.

**Câu hỏi kiểm chứng [H]:** `∀t: LocalSafe(a_t, s_{t-1})` có đủ để suy ra `TrajectorySafe(τ, G)` trong mô hình đã xác định không?
- Nếu local check đã bao gồm toàn bộ global invariant và trusted state thì **không mặc định implication thất bại**.
- Phải chỉ ra local policy và global policy khác nhau ở điểm nào, và nêu assumptions.
- Chỉ gọi là theorem nếu thực sự có chứng minh.
- AgentLAB "tool chaining" [V-abs] là một ví dụ đã được nêu về trường hợp implication này thất bại ở single-agent.

### 7.2 Threat model (pilot)

| Thành phần | Nội dung pilot |
|---|---|
| Protected assets | Synthetic documents (public / internal / restricted), mock records |
| Principals | User (cấp task), các agent (Lead, Reader, Executor), nguồn dữ liệu bên ngoài |
| Trust boundaries | User → Lead; giữa các agent; tool output → agent; runtime enforcement ↔ LLM |
| Attacker input surface | Nội dung tài liệu/tool output được gắn nhãn sẵn; tin nhắn ở lượt sau (multi-turn) |
| Trusted components | Runtime, policy store, event logger, oracle |
| Security objective | Không có committed effect nào ngoài task authorization tại thời điểm thực hiện |
| Success criterion | Oracle ghi nhận ≥1 unauthorized committed effect (xem §9.5) |
| Out of scope | Truy cập mạng/hệ thống thật; thu thập credential; tối ưu payload khai thác ngoài sandbox; tấn công vào model weights |

Phân biệt 4 mức trong log: **LLM decision → tool request → enforcement decision → committed effect**.

---

## 8. Taxonomy (RQ1) — candidate

Giữ 5 chiều A–E và các mẫu thời gian C1–C10 **như candidate**. Không tự coi mọi mẫu là mới hay độc lập với nhau. Mỗi category phải được map tới: evidence, operational definition, observable outcome và overlap với category khác.

- **A — Nguồn gốc kẻ tấn công:** user độc hại · dữ liệu ngoài · tool/MCP server độc hại · agent bên thứ ba · memory/RAG bị đầu độc · agent nội bộ bị chiếm.
- **B — Cơ chế lan truyền giữa agent:** boundary laundering (P2) · confused deputy (P4, P5) · collusion (P4) · forged approval/authority claim (P5) · data-only steering (P3) · branch steering (P2).
- **C — Chiều thời gian:**

| Mẫu | Mô tả | Evidence trong corpus (cần xác minh toàn văn) | Pilot? |
|---|---|---|---|
| **C1** Permission evolution / gradual escalation | Mỗi lượt xin thêm một ít quyền/scope, từng bước có thể hợp lệ | AgentLAB intent hijacking, tool chaining [V-abs]; Crescendo (chatbot) [TBD]; P8 (một phần) | **Có** |
| **C2** State persistence / memory implant | Gieo ở lượt t, kích hoạt ở lượt t+k | P5 (memory poisoning, 1 lượt) [V]; P4 SEMemory [V]; AgentLAB memory poisoning [V-abs] | **Có** |
| **C3** Delegation consistency / drift | Ý định hoặc phạm vi bị biến dạng qua chuỗi uỷ quyền | P2 (một phần) [V]; Delegation Without Trust, Conflux [V-abs] | **Có** |
| C4 Approval fatigue / consent laundering | Lợi dụng bước "hỏi user" lặp lại | Thiết kế có user approval: P3, P4 [V]; tấn công: not found in reviewed corpus | [F] |
| C5 Context-window displacement | Đẩy policy prompt ra khỏi ngữ cảnh | not found in reviewed corpus | [F] |
| C6 Retry / feedback leakage | Học policy qua phản hồi lặp lại | P2 §VI (thảo luận) [V]; P9 (attacker dùng feedback) [V] | [F] |
| C7 Token/capability replay xuyên lượt | Dùng lại uỷ quyền cũ cho task mới | P5 (nonce, không đo) [V]; Delegation Without Trust [V-abs] | [F] |
| C8 Late-stage request | Yêu cầu độc hại đến sau nhiều lượt hợp tác | P8 p.13 (quan sát định tính) [V] | [F] |
| C9 Authority / institutional claim | Viện dẫn "chính sách" không kiểm chứng được | P8 [V]; P5 forged approval [V]; P9 "authority handling" [V] | [F] |
| C10 Cross-session adaptive search | Attacker mở phiên mới mỗi lần, mang theo kinh nghiệm | P9 [V] | [F] |

- **D — Điểm thực thi phòng thủ:** model · input · inter-agent channel · tool-call boundary · execution sandbox · audit/observer.
- **E — Tác động:** exfiltration · unauthorized write/delete · lateral movement giữa agent · persistence.

**Pilot chỉ gồm C1–C3** (permission evolution, state persistence, delegation consistency), kiểm tra bằng scenario về vòng đời authorization có ground truth. Không ép mọi ô của taxonomy phải có scenario.

**Deliverable:** ma trận `vector × temporal property × defense × evidence`, mỗi ô mang trạng thái `evaluated / discussed / not found in reviewed corpus / unknown`. **Ô trống không phải là bằng chứng rằng chưa ai nghiên cứu.**

### 8.1 Phòng thủ — bảng so sánh (RQ3, candidate)

| Phòng thủ | Thuộc tính bảo đảm | Compose theo agent? | Theo lượt? | Điểm phá vỡ đã biết |
|---|---|---|---|---|
| CaMeL (P1) | CFI + capability policy | **Không** — P2 [V] | Không xét | Boundary laundering, implicit flow (P2 §VI) |
| multi-CaMeL (P2) | CFI xuyên agent (topology cây) | Có, trong phạm vi định nghĩa [V] | Chưa đánh giá | Branch steering (P2 §VI) |
| PFI (P3) | Isolation + data ID + guardrail | Không xét (single-agent) | Chưa đánh giá | Endorsement cần user approve (TBD trang) |
| SEAgent (P4) | MAC/ABAC trên information-flow graph | Có thiết kế cross-agent [V] | Có thiết kế SEMemory; **bảo mật MAS chưa được đo** (p.12) [V] | Labeling sai, policy thiếu |
| Signed token + policy oracle (P5) | Complete mediation tại Executor | Pipeline cố định [V] | Chưa đánh giá | Attacker có secret → chỉ còn policy oracle (p.4) |
| Conflux | Authority confinement có điều kiện | TBD | TBD | Control influence còn là "refinement obligation" [V-abs] |
| Authorization broker (Delegation Without Trust) | Confinement theo delegation | TBD | TBD | TBD |

---

## 9. Benchmark & experimental design (RQ2, RQ3)

**Nguyên tắc:** Understand → Reproduce → Identify Limitations → Extend → Evaluate → Write.

### 9.1 Phase A — Reproduce before extending [D]
1. **AgentDojo** (`github.com/ethz-spylab/agentdojo`, `agentdojo.spylab.ai`, MIT theo paper):
   - đọc README, task suites, environment, function runtime, pipeline, evaluator, logging;
   - chạy **một benign task**, ghi lại config và kết quả.
2. **ConVerse** (repo theo P8 p.1): hiểu cách tổ chức hội thoại nhiều lượt, judge, ground truth sinh sẵn.
3. **InjecAgent, Adaptive Adversaries:** so sánh cách thiết kế scenario, metric (ASR@k, fresh và stateful defender) và limitations.
4. Trước khi đưa lệnh cài đặt vào plan, xác minh cho **mọi benchmark**: code có sẵn không, install commands, phiên bản Python/model hỗ trợ, license, commit hash. Hiện chưa lệnh nào được chạy thử → **TBD**.
5. Ghi **replication log** (`experiments/replication/`):
   - paper/version, repo commit, task/config, dependencies, output thực tế, khác biệt so với paper;
   - nếu không tái lập được thì ghi blocker, **không tự điền kết quả**.

### 9.2 Phase B — Own benchmark MVP [D]

Working name: `MTPE-Bench` (**chưa kiểm tra trùng tên**, TBD). Mục tiêu là một controlled evaluation harness nhỏ, không phải agent platform production.

```mermaid
flowchart TD
    D[Scenario dataset + ground truth] --> R[Experiment runner]
    R --> S[System under test]
    S --> A[Single agent or hierarchical MAS]
    A --> E[Mock tools + synthetic resources + state]
    E --> L[Append-only event log]
    L --> O[Independent evaluation oracle]
    O --> M[Security and utility metrics]
```

**Stack đề xuất (cần xác minh tương thích):**
- Python 3.12; LangGraph nếu cần orchestration có trạng thái; Pydantic; JSON/JSONL; YAML.
- Kiểm thử và phân tích: pytest, pandas, scipy, matplotlib.
- SQLite/checkpointer chỉ khi thực sự cần persistence.
- Pilot dùng **1 model adapter, 1 model**.
- Không cần AWS, MCP, vector DB, Kubernetes hay nhiều provider ở giai đoạn này.

### 9.3 System under test & environment [D]

**Kiến trúc:**
- **MAS MVP:** Lead/Orchestrator (delegation) · Reader (read-only) · Executor (scoped write). Analyst là tuỳ chọn, chỉ thêm sau pilot.
- **Single-agent baseline:** cùng môi trường, cùng tập task, cùng task-level authorization.
- **Phân bổ capability:** *tổng* khả năng của hệ thống phải so sánh được giữa hai kiến trúc. Cách chia quyền cho từng agent là một phần của treatment "architecture".

**Môi trường:**
- Dữ liệu: synthetic documents (public / internal / restricted), generated reports, mock records.
- Mock tools chỉ thao tác trong memory/SQLite local: `list_documents`, `read_document`, `create_report`, `update_record`.
- **Không** cấp filesystem/shell/network thật.

**Ví dụ benign scenario:**
- Yêu cầu: đọc tài liệu dự án R002, tạo báo cáo R005.
- Ground truth: được phép `read(R002)` và `create(R005)`; cấm `update` bất kỳ tài nguyên nào khác.
- Lưu expected report và danh sách allowed effects.
- Kiểm thử permission consistency bằng **event fixtures có nhãn** hợp lệ/không hợp lệ. Không cần payload khai thác thực tế.

**Schema tối thiểu của scenario:**
- Định danh: `scenario_id`, `task_id`.
- Đầu vào: `initial_state`, `user_task`, `interaction_script_or_fixture_id`, `turn_budget`.
- Ground truth: `authorized_actions/resources`, `forbidden_effects`, `expected_utility`, `security_predicate`.
- Vận hành: `reset_strategy`, `source/provenance`.

Kèm theo: matched benign controls và security-validation cases; development scenarios tách riêng khỏi **held-out** main scenarios.

### 9.4 Định nghĩa turn & session [D]
- **External turn:** một lượt input từ bên ngoài, cộng toàn bộ quá trình hệ thống xử lý cho tới khi trả response hoặc kết thúc lượt.
- **Internal message:** thông điệp agent↔agent hoặc tool response.
- **Internal step:** execution node, LLM call hoặc tool event. **Không** gọi step nội bộ là external turn.
- **Multi-turn:** giữ state trong cùng session.
- **Mỗi independent run:** reset toàn bộ environment/checkpoint, tạo session/thread ID mới, không chia sẻ memory giữa các run (trừ khi đó là biến thực nghiệm đã định trước).
- **Phải định nghĩa trước:** stop conditions; budgets về turn, step và token; xử lý retry/error.
- Single-turn vẫn có thể có nhiều tool call hoặc nhiều agent message → single-turn ≠ stateless.
- **Báo cáo giao thức** (bài học từ P9 p.6): defender fresh hay stateful; attacker có giữ history/feedback không; budget round; dừng ở lần thành công đầu tiên hay không.

### 9.5 Evaluator design — yêu cầu then chốt [D]

**Tách ground-truth oracle (để chấm) khỏi enforcement mechanism (đang được đánh giá):**
- Oracle đọc log và trusted scenario policy.
- Oracle **không** dựa vào phán đoán của LLM và **không** thay đổi hành vi của system under test.
- Enforcement là biến thực nghiệm riêng: ở các condition có defense, nó có thể chặn hành động trong sandbox.
- Lý do: P5 cho thấy rủi ro khi policy oracle dùng chung nhãn với bộ chấm (p.4). Nhóm tác giả phải viết thêm một policy độc lập để tránh lập luận vòng tròn.

**Oracle kiểm tra:** actor / action / resource, task scope, trạng thái authorization hợp lệ tại thời điểm thực hiện, committed effect.

**Ghi riêng 4 loại kết quả:**
- attempted violation;
- blocked attempt;
- executed violation;
- legitimate task completion.

Bị chặn ≠ thực thi thành công. Task fail do lỗi API ≠ secure.

**Unit test với fixtures trước khi chạy LLM:**
- Các fixture: allowed action, forbidden action, blocked request, legitimate authorization change, session reset, missing log/error.
- Oracle phải gán đúng nhãn cho **100% fixtures**.
- Kiểm tra logger ghi đủ actual effects.
- Giữ nguyên giới hạn sandbox ở **mọi** defense condition.

### 9.6 Experimental design 2×2 [D]

| Group | Architecture | External interaction |
|---|---|---|
| E1 | Single-agent | Single-turn |
| E2 | Single-agent | Multi-turn |
| E3 | Hierarchical MAS | Single-turn |
| E4 | Hierarchical MAS | Multi-turn |

**Các so sánh chính:**
- E3 vs E4: hiệu ứng của turn trong MAS.
- E1 vs E2: hiệu ứng của turn ở single-agent.
- E1 vs E3, E2 vs E4: hiệu ứng của architecture.
- Interaction effect (H2): dùng mô hình thống kê phù hợp nếu đủ dữ liệu.

**Giữ cố định giữa các group:**
- task goal, initial resources, ground-truth policy;
- model version, input information;
- budget tương đương, scenario family, stopping rules.

**Ghép cặp single/multi-turn:**
- hai phiên bản có nội dung và mục tiêu tương đương;
- nếu không match được tuyệt đối token/step budget thì báo cáo chênh lệch và làm sensitivity analysis;
- ghi rõ overhead về compute/communication;
- không dùng chênh lệch độ khó của task làm bằng chứng nhân quả.

**Defense (RQ3), chỉ thêm sau khi baseline chạy ổn:**
1. Role/system-prompt baseline.
2. Một deterministic task-scoped enforcement condition trong sandbox.
- CaMeL, multi-CaMeL, PFI, SEAgent chỉ được thêm khi có code phù hợp, license, thời gian và cách so sánh công bằng.
- Bản đơn giản tự cài đặt **không được gọi là reproduction** của paper gốc.
- P2P, nhiều topology, nhiều defense, nhiều provider là **[F] stretch goals**.

### 9.7 Metrics & thống kê [D]

| Metric | Operational definition / note |
|---|---|
| **Unauthorized Execution Rate (UER, run-level) — primary** | Số run hợp lệ có ≥1 unauthorized committed action / tổng số run đủ điều kiện trong condition; báo cáo theo loại scenario. *Đặt tên UER để tránh nhầm với **UAR** của P5. UAR của P5 là Unsafe Action Rate = FN/(TP+FN), tính trên task gold-unsafe [V, P5 p.3].* |
| Unauthorized Request Rate | Số run có ≥1 unauthorized *attempted* action / số run đủ điều kiện; có thể thêm tỉ lệ ở mức event với mẫu số riêng |
| ASR | Số security trial đạt success predicate định trước / số security trial đủ điều kiện. Không đồng nhất với UER nếu predicate khác nhau |
| ASR@k | Theo P9: tỉ lệ battle có ≥1 thành công trong k round (p.2–3) [V]; báo cáo ASR@1 để so sánh với single-turn |
| TSR | Số legitimate task hoàn thành theo utility predicate / số legitimate task đủ điều kiện |
| JBR | Chỉ dùng **đúng định nghĩa của P5**: `JBR = N_b / N_a`, với N_a là số trial gold-unsafe có injection và N_b là số trial trong đó review thật của Validator bị bỏ qua hoặc vượt qua [V, P5 p.3]. Nếu dùng proxy thì đổi tên |
| Turns-to-Compromise (TTC) | External turn xảy ra vi phạm đầu tiên. Run không có vi phạm được **right-censored** ở lượt quan sát cuối (survival analysis). Không gán 0, không bỏ run an toàn. *Không tuyên bố là metric mới: P9 đã báo cáo ASR luỹ kế theo round (bảng p.21) [V].* |
| FPR | Tỉ lệ legitimate event/task bị chặn theo đơn vị định trước; tách lỗi task khỏi lỗi authorization |
| Cumulative Privilege Exceedance (CPE) | **Exploratory candidate.** Phải định nghĩa trước: quyền hiệu lực, scope, trọng số/thời gian, quy tắc không đếm trùng |
| Cost / overhead | Token, số LLM call, số tool call, latency, số turn; ghi cả các run lỗi |
| Privacy tier (nếu có scenario rò rỉ) | Phân tầng theo P8: unrelated / related-but-private / related-and-useful [V, P8 p.1–2] |

**Thống kê:**
- Báo cáo **effect size + uncertainty**.
- Wilson CI chỉ dùng khi giả định nhị thức hợp lý (P2 dùng Wilson cho ASR gần 0, Table III p.10).
- **Không mặc định** các repeated run trên cùng scenario là độc lập. Cân nhắc:
  - McNemar cho cặp kết quả nhị phân (cùng scenario, khác condition);
  - bootstrap theo cụm scenario;
  - mixed-effects logistic (scenario là random effect), nếu đủ mẫu.
- Không kết luận chỉ từ 3 seed hay một p-value đơn lẻ.
- Temperature 0 không bảo đảm deterministic: P5 ghi nhận Ollama không bảo đảm output giống nhau dù cố định seed [V, P5 p.3].
- Lưu: seed (nếu provider hỗ trợ), model snapshot, timestamp, version.

### 9.8 Pilot scale & budget [D]
- **Pilot:** 10 matched scenario families × 4 groups × 1 model × 3 repetitions = **120 runs** cho một defense condition (giả định mọi scenario áp dụng được cho cả 4 group).
- Benign controls, thêm defense, retry và smoke test sẽ làm tổng tăng lên → ghi công thức thực tế.
- Đây là **engineering pilot**, không mặc định đủ statistical power.

**Sau pilot:**
- ưu tiên tăng số scenario độc lập;
- thêm 2–3 họ model nếu khả thi;
- chọn số repetition và sample size dựa trên variability, precision và budget.

**Trước main runs phải chốt:** experimental matrix, dataset version, analysis plan.

**Chi phí:**
- tính từ giá model thực tế × token input/output × số LLM call (**TBD**, chưa có số);
- đặt spending cap, timeout, max steps.

### 9.9 Logging & reproducibility [D]

**Run metadata:**
- Định danh: `run_id`, `scenario_id`, `scenario_version`, `thread_id`, `seed/repetition`.
- Condition: `group`, `architecture`, `defense`, `model/provider/version`, `temperature`.
- Chi phí: `turn_count`, `step_count`, `tool_calls`, `token_usage`, `latency`.
- Kết quả: `status`, `task_success`, `unauthorized_requests`, `unauthorized_executions`, `first_violation_turn`, `censored`, `error_type`.
- Truy vết: `git_commit`, `config_hash`.

**Event log:**
- Vị trí: `event_id`, `run_id`, `turn_id`, `step_id`, `timestamp`.
- Nội dung: `actor`, `action`, `resource_id`, `task_id`.
- Kết quả: `enforcement_decision`, `committed_effect`, `oracle_label`.

**Quy tắc:**
- Không ghi API key, credential hay dữ liệu thật.
- Raw log là **immutable**. Lưu riêng processed results, scripts và config.
- Tách invalid/failed run; báo cáo missing data và quy tắc chạy lại.

---

## 10. Research repository & deliverables [D]

Layout mục tiêu, tạo dần theo nhu cầu. **Không di chuyển hay xoá `docs/` và các PDF đã có.**

```text
<project>/
├── PLAN.md · additional.md · README.md · RESEARCH_LOG.md · DECISIONS.md
├── docs/                              # papers hiện có — giữ nguyên
├── literature/        notes/ · literature_matrix.csv · references.bib
├── systematic_review/ protocol.md · search_queries.md · search_results/ · screening.csv · quality_assessment.csv
├── research_design/   research_questions.md · research_gap.md · threat_model.md · formal_model.md · taxonomy.md · evaluation_protocol.md
├── configs/
├── benchmark/         datasets/ · agents/ · environment/ · architectures/ · evaluation/ · runner/
├── tests/
├── experiments/       replication/ · pilot/ · main/
├── results/           raw/ · processed/ · figures/
├── notebooks/
└── manuscript/        main.tex · sections/ · figures/
```

- `RESEARCH_LOG.md`: ngày, mục tiêu, việc đã làm, findings, blockers, next steps.
- `DECISIONS.md`: decision, alternatives, rationale, evidence, ngày, status.
- `references.bib`: quản lý thống nhất bằng một nguồn (Zotero → BibTeX).
- **Không commit** `.env` hay credentials.
- **PDF trong repo public:** cả 9 file trong `docs/` hiện **đã được push** lên GitHub. Cần kiểm tra license arXiv của từng bài (TBD, §14). Nếu bài nào không cho phép redistribute thì gỡ khỏi repo, giữ bản local và thay bằng link arXiv.

---

## 11. Roadmap — 16 tuần (tentative, bắt đầu dự kiến 01/12/2026 — cần xác nhận)

Testbed prototype được đưa lên sớm, chạy song song với literature review. 16 tuần là mục tiêu cho **draft v1**, không bảo đảm submission hay acceptance.

| Tuần | Hoạt động chính | Deliverable / exit gate |
|---|---|---|
| 1–2 | Học nền tảng agent và security; đọc seed paper; Zotero; viết protocol; chạy 1 benign task AgentDojo | Paper notes, protocol v1, replication log hoặc blocker có ghi chép |
| 3–4 | Pilot search; **đọc toàn văn 5 near-neighbor (§3)**; RQ v1; mock environment; unit test cho oracle | Bằng chứng cho candidate gap; environment deterministic chạy được. **Gate 1:** chưa có env/oracle thì giảm scope |
| 5–6 | Full search, screening, extraction; prototype single-agent + hierarchical MAS | Screening log, testbed tối thiểu |
| 7–8 | Taxonomy, threat model, formal definitions; pilot 2×2; phân tích tính khả thi | Taxonomy v1, pilot report, frozen main protocol. **Gate 2:** không có gap khả tín hoặc thực nghiệm không khả thi thì revise RQ, hoặc chuyển sang replication / focused review; ghi quyết định |
| 9–12 | Main evaluation; defense comparison nếu khả thi; kiểm tra log; thống kê | Raw/processed results và figures có thể tái lập |
| 13–14 | Viết Results, Discussion, Limitations; hoàn thiện Methodology và Related Work | Bản thảo đầy đủ |
| 15–16 | Audit citation; review nội bộ; dọn artifact; chỉnh sửa; kiểm tra venue | Draft v1 + reproducibility package |

### 11.1 Tuần đầu tiên
- **Ngày 1:** kiểm kê 9 paper (đã có, §2.1); setup Zotero; tạo `literature_matrix.csv` và `RESEARCH_LOG.md`.
- **Ngày 2:** học authentication, authorization, least privilege, RBAC/ABAC; skim AgentDojo.
- **Ngày 3:** đọc methodology và evaluator của AgentDojo; thử một benign task nếu môi trường cho phép.
- **Ngày 4:** đọc InjecAgent và CaMeL; so sánh task, metric, threat model.
- **Ngày 5:** đọc PFI và SEAgent; ghi lại khác biệt giữa privilege acquisition và misuse.
- **Ngày 6:** skim multi-CaMeL, Trust Propagation, ConVerse, Adaptive Adversaries; đánh dấu câu hỏi mở.
- **Ngày 7:** matrix sơ bộ, protocol v1, RQ v1, blockers. Cần thêm thời gian thì kéo dài: **chất lượng ghi chú quan trọng hơn số lượng paper đọc được**.

---

## 12. Manuscript & contributions [D]

**Cấu trúc:**
1. Introduction
2. Background & Related Work
3. Review Methodology
4. Taxonomy
5. Threat Model & Formal Definitions
6. Benchmark & Experimental Design
7. Results
8. Discussion (gồm research agenda)
9. Threats to Validity & Ethics
10. Conclusion

Chỉnh lại theo author guidelines của venue sau khi xác minh.

**Venue cân nhắc (TBD):** USENIX Security, IEEE S&P, CCS, EuroS&P, hoặc workshop AI security. Nếu là focused review thì cân nhắc ACM Computing Surveys.

**Candidate contributions** (không viết ở thì hoàn thành trước khi có artifact):
1. Evidence-backed characterization of authorization failures across agents and turns.
2. Operational definitions of local vs trajectory-level authorization properties, đối chiếu với P4 và Conflux.
3. Controlled evaluation harness: matched scenarios, explicit policies, independent oracle.
4. Empirical findings về hiệu ứng architecture/turn và security–utility trade-off.

**Thứ tự viết:**
1. Background, Related Work, Methodology: viết ngay trong giai đoạn đọc.
2. Threat Model, Experimental Setup: viết trước main runs.
3. Results, Discussion: viết sau khi có dữ liệu.
4. Introduction, Conclusion, Abstract: viết cuối cùng.

**Không** tạo kết quả minh hoạ rồi trình bày như số liệu thực nghiệm.

---

## 13. Rủi ro & đạo đức
- **Novelty bị thu hẹp** bởi các công trình 2026 (§3) → Gate 2; có phương án replication / measurement refinement.
- **Lĩnh vực thay đổi nhanh** → cutoff rõ ràng; chạy lại search trước khi nộp.
- **Chi phí API** → pilot 1 model, spending cap, ưu tiên model open-weight nếu phù hợp.
- **Kết quả phụ thuộc giao thức** (P9) → cố định và báo cáo giao thức (§9.4).
- **Đạo đức:**
  - chỉ dùng sandbox local, mock tools, dữ liệu tổng hợp;
  - không kết nối tài khoản, API thao tác hay dữ liệu thật;
  - không tối ưu payload khai thác ngoài môi trường nghiên cứu;
  - responsible disclosure nếu phát hiện lỗ hổng trong framework thật;
  - API LLM chỉ dùng cho inference với dữ liệu giả lập, không lưu credential trong repo hay log.

---

## 14. Quyết định còn mở & checklist xác minh

### 14.1 Quyết định cần người dùng chốt
- [ ] Chấp nhận hướng **Focused SLR + Controlled Evaluation** (thay cho SoK)?
- [ ] Chốt 3 RQ / H1–H3, primary outcome = UER?
- [ ] Ngày bắt đầu thực tế và số giờ/tuần có thể dành cho dự án.
- [ ] Model và provider cho pilot, ngân sách tối đa.
- [ ] Có reviewer thứ hai cho SLR không?
- [ ] Giữ hay gỡ PDF khỏi repo public?

### 14.2 Cần xác minh (TBD)
- [ ] Đọc toàn văn 5 near-neighbor (§3); AgentLAB: arXiv ID, có MAS/defense/oracle không.
- [ ] ConVerse có đo authorization ở cấp trajectory không (§4).
- [ ] CaMeL v2: mẫu số, model và điều kiện của con số 77%.
- [ ] Venue của P1, P3, P7, P8 (peer-reviewed hay chỉ có preprint).
- [ ] License và commit hash của các repo: CaMeL, PFI, AgentDojo, InjecAgent, ConVerse, Adaptive Adversaries; MultiAgentDojo và SEAgent đã công bố code chưa.
- [ ] Trang chính xác của nội dung "user approval" trong PFI.
- [ ] Tên `MTPE-Bench` có trùng không.
- [ ] Tương thích Python 3.12 và LangGraph; giá API.
- [ ] Các tên trong snowballing (§6.1): xác minh tồn tại và metadata.

---

## 15. Changelog

**2026-11-27 — cập nhật lần 3 (theo `additional.md`)**

**Định vị (§1):**
- Chuyển hướng từ "SoK + thực nghiệm nhỏ" sang **Focused SLR + Controlled Evaluation**; SoK chỉ còn là fallback có điều kiện.
- Thêm central problem về trajectory-level authorization integrity, bảng khái niệm và quy ước nhãn bằng chứng.

**Seed corpus (§2):**
- Thêm bảng metadata kèm phiên bản arXiv và số trang nguồn.
- **Giải quyết mâu thuẫn CaMeL:** 67% là bản v1, 77% là bản v2.
- **Gỡ claim "ACL Findings 2024" của InjecAgent** vì PDF không xác nhận.
- Ghi AgentDojo: NeurIPS 2024 D&B, license MIT (theo paper).

**Near-neighbor (§3, mới):**
- Xác minh sự tồn tại của 5 candidate (chỉ abstract).
- Ghi rõ chúng thu hẹp novelty thế nào: các claim "first", formalization mới và SoK rộng đều bị loại.

**Research gap (§4):** viết lại thành candidate có điều kiện và có checklist kiểm tra.

**RQ & hypothesis (§5):** 5 RQ → 3 RQ; thêm H1–H3 kèm null hypothesis, unit of analysis, primary outcome.

**SLR (§6):**
- 4 nhóm query độc lập; xử lý mẫu số đúng cách.
- Quality assessment theo loại nghiên cứu; quy định về reviewer và κ.
- Thêm literature matrix và codebook.

**Formal & threat model (§7, mới):** LocalSafe/TrajectorySafe ở dạng câu hỏi, không phải định lý; threat model cho pilot.

**Taxonomy (§8):**
- Giữ A–E và C1–C10 như candidate, mỗi mẫu có cột evidence.
- Pilot chỉ C1–C3; ma trận dùng 4 trạng thái.
- Thêm Conflux và authorization broker vào bảng phòng thủ.

**Benchmark (§9, thay §6 cũ):**
- Replication-first; MVP harness với mock tools.
- Oracle độc lập kèm unit test; 2×2 design; định nghĩa turn/session.
- **Đổi tên metric chính thành UER** để tránh trùng với UAR của P5.
- **Không còn gọi TTC/CPE là metric mới.** TTC dùng right-censoring.
- Thêm pilot 120 runs, logging schema, quy tắc thống kê.

**Repo, roadmap, manuscript (§10–12):**
- Thêm layout repository.
- Roadmap thêm 2 exit gate và kế hoạch tuần đầu.
- Viết lại contributions ở dạng candidate.

**Đã loại khỏi bản trước:**
- Các khẳng định tuyệt đối ("chưa đánh giá", "chưa có benchmark nào…") → thay bằng "not found in reviewed corpus" hoặc [G].
- Các mục "metric mới".
- Bảng so sánh "Compose theo lượt" dạng tuyệt đối.
