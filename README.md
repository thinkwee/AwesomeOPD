<div align="center">

# AwesomeOPD

</div>

<div align="center">

![Surveys](https://img.shields.io/badge/Surveys_&_Position-7-4E6813?style=for-the-badge)
![White-Box](https://img.shields.io/badge/White--Box_OPD-17-BFA2DB?style=for-the-badge)
![Black-Box](https://img.shields.io/badge/Black--Box_OPD-3-845C40?style=for-the-badge)
![OPSD](https://img.shields.io/badge/OPSD_Q1=B-9-A259FF?style=for-the-badge)
<br>
![Iterative](https://img.shields.io/badge/Iterative_Q1=C-2-50C878?style=for-the-badge)
![OPD-RL](https://img.shields.io/badge/OPD--RL_Hybrids-17-9B59B6?style=for-the-badge)
![Reasoning](https://img.shields.io/badge/Reasoning_OPD-3-FF69B4?style=for-the-badge)
![Multimodal](https://img.shields.io/badge/Multimodal_OPD-5-2ECC71?style=for-the-badge)
<br>
![Agent](https://img.shields.io/badge/Agent_&_Embodied-4-1F4CAD?style=for-the-badge)
![SpecDec](https://img.shields.io/badge/Speculative_Decoding-19-D89F7B?style=for-the-badge)
![Frameworks](https://img.shields.io/badge/Frameworks-14-FA5A4C?style=for-the-badge)
![Industrial](https://img.shields.io/badge/Production_Reports-11-ffc884?style=for-the-badge)
<br>
![Off-Policy KD](https://img.shields.io/badge/Off--Policy_KD_Refs-58-007a88?style=for-the-badge)
![Verified](https://img.shields.io/badge/Verified_Apr_2026-✓-success?style=for-the-badge)

</div>

# When LLMs Distill On-Policy

**AwesomeOPD** is a curated awesome-list of **open-source repositories and papers** for training LLMs (and VLMs / agents / draft models) with **On-Policy Distillation (OPD)** and **On-Policy Self-Distillation (OPSD)**.

 - 🎯 **OPD definition (strict, two criteria)**:
   - **C1 (on-policy data)** — during training, the student samples its own trajectories `y ~ π_student(·|x)`.
   - **C2 (teacher supervision on student samples)** — the teacher provides per-token or per-sequence supervision **evaluated on those student-generated samples** (not on a fixed corpus or teacher-generated data).
   
   Both must hold for an entry to be *strict* OPD. Methods that satisfy C1 but not C2 are usually RL (only outcome reward, no teacher distillation). Methods that satisfy C2 but not C1 are off-policy KD. Each section below has a **📝 Strictness notes** block flagging entries that only partially satisfy the criteria — they're kept in the list because the community searches for them under "OPD", but readers should know what they are.
 - 🪞 **OPSD**: a special case of OPD where the teacher *is the same model* — typically the same weights but conditioned on **privileged information** (a verified reasoning trace, ground-truth answer, "be concise" prefix, longer context, document, etc.), or an earlier checkpoint of itself.
 - 🚧 The line between OPD and on-policy RL is intentionally fuzzy: both train on student rollouts. OPD provides **dense per-token supervision** from a teacher; on-policy RL provides a **scalar reward**. Many recent works (G-OPD, RLAD, KDRL, SDPO, HDPO, REOPOLD …) explicitly unify the two — those are listed under **OPD-RL Hybrids**.
 - 🧭 **Editorial framing (this list's contribution)**: we organise the field around [**four design questions**](#taxonomy) (who is the teacher? what signal? how are rollouts consumed? where in the pipeline?), a [**timeline reading**](#a-timeline-lens) of how the wave evolved, a [**practical decision guide**](#practical-decision-guide--which-opd-do-i-reach-for), [**curator's picks**](#-curators-picks--where-to-start), and a list of [**open problems**](#-open-problems--whats-missing). Existing surveys (e.g. [Tencent's 2604.00626](https://arxiv.org/abs/2604.00626)) are used as a *paper pool*; their internal taxonomies are not adopted.
 - ⚠️ This list is built by carefully reading paper PDFs, project pages, and source code. Star counts and pushed-at dates were captured via `gh api` near 2026-04-27. Some 2026 papers cited in surveys do not yet have public code; we mark those `📄 paper-only`. If you spot mistakes please open an issue/PR.
 - 📅 **Last updated**: 2026-04-28
 - 🤗 PRs welcome — especially for OPD work in robotics / speech / pretraining and for non-English production reports.

---

## Why OPD now

Three coincident events made OPD the dominant LLM post-training paradigm of 2025–2026:

1. **Qwen3 Tech Report** (May 2025, [arXiv 2505.09388](https://arxiv.org/abs/2505.09388)) showed that a *strong-to-weak on-policy logit-distillation* leg of post-training matched RL performance on AIME'24 at **~1/10 the GPU hours** of RL. This is the canonical industrial OPD recipe.
2. **Thinking Machines Lab blog** ([Lu et al., Oct 2025](https://thinkingmachines.ai/blog/on-policy-distillation/)) re-popularised OPD with a one-line recipe: "swap the KL regulariser model in your RL trainer for a stronger teacher model" — and released the [Tinker cookbook](https://github.com/thinking-machines-lab/tinker-cookbook/tree/main/tinker_cookbook/recipes/distillation) with reproducible scripts.
3. The **DeepSeek-R1 wave** (Jan 2025) trained the field to think of post-training as token-by-token imitation of a stronger reasoner. Most R1-distill releases were *off-policy*; the OPD wave that followed in late-2025/2026 fixed the resulting exposure-bias issues (OPSD, OPCD, GAD, Lightning OPD, Rethinking OPD …).

By Q1 2026 OPD had been adopted across **Qwen3, GLM-5, MiMo-V2-Flash, Gemma 2/3, Nemotron-Cascade-2, Llama 4 Maverick (codistillation)**, and the literature around OPD was large enough to need [a survey of its own](https://arxiv.org/abs/2604.00626).

---

## Taxonomy

The whole zoo of OPD methods reduces to **four design questions**. Once you answer them, every paper in this list slots into one cell of a 4D grid. The categories you see below are just the most common combinations.

### Q1 — Who plays the role of the teacher?

The single most informative axis. Every other choice cascades from this one.

| Teacher source | Mechanism | Examples |
|---|---|---|
| **A. Larger external model** | Classical compression: a 32B/235B teacher transfers signal to a 1B–14B student. | GKD, MiniLLM, DistiLLM/2, BOND, Qwen3 strong-to-weak, Gemma 2/3, Phi-4-Mini-Reasoning |
| **B. Same model + privileged context** | Teacher = student weights, but conditioned on a verified trace / answer / "be concise" prefix / longer context / document. The teacher is *yourself with a hint*. | OPSD, OPCD, OEL, π-Distill, GATES, CRISP/OPSDC, OPSDL, GAD-context |
| **C. Same model, earlier checkpoint** | Iterative self-bootstrapping. The teacher snapshot is frozen for one round, the student trains, then the snapshot rolls forward. | SPIN, BOND iterations, rStar / rStar2-Agent (with PPM filter), iterative RAFT |
| **D. A panel of specialist teachers** | Multiple domain-specialised teachers; the student aggregates per-domain signals. Production-favoured because it lets you hire the best teacher for each capability. | MiMo-V2-Flash (MOPD), Nemotron-Cascade-2 (multi-domain OPD), GLM-5 cross-stage, Llama 4 codistillation |
| **E. A discriminator** | Adversarial: a co-trained classifier learns to tell student outputs from teacher outputs and serves as on-policy reward. Indispensable when the teacher is a closed API (no logits). | GAD, Lion |
| **F. No teacher — only a verifier** | Strictly speaking RL, not distillation. Listed here because *inside-RL* OPD (Q4 row 4) glues verifier-RL onto teacher-KL, and many community releases blur the line. | (Tongyi DeepResearch, Magistral, MiniMax-M2 — flagged in Caveats; not OPD) |

Two notes: (i) the same paper can pick different teachers in different stages — Qwen3 starts with **A** (larger Qwen3) for cold-start and ends with **B** (self with `/think` mode) on the on-policy leg; (ii) **B+C+D are the 2026 frontier** — most genuinely-novel OPD work in the last 12 months pushed one of these three.

### Q2 — What signal does the teacher emit?

This determines what your loss can look like.

| Signal | Per-token? | Requires logit access? | Examples |
|---|---|---|---|
| Per-token logits / log-probs | yes | yes (white-box) | GKD, MiniLLM, DistiLLM, OPSD, OPCD, the Qwen3 recipe |
| Top-k truncated logits | yes | partial | NeMo-Aligner KD, verl async OPD, FastDraft |
| Sequence-level reward / preference | no | no | BOND, Lion, RLHF-style hybrids |
| Verbal score (e.g. 0–9) | no | no | OVD |
| Discriminator real/fake | no | no | GAD |
| Verifier accept/reject | no | no | RLVR, SCoRe, KDRL outcome leg |
| Hidden-state / attention map | yes (feature) | yes | DSKD, Tunix attention-strategy, Minitron |

Mixing signals is normal. KDRL pairs token-logit + verifier-reward; SCoRe combines verifier-reward + privileged-trace correction.

### Q3 — How are student rollouts consumed?

| Consumption mode | Mechanism | Examples |
|---|---|---|
| All tokens, equally | Standard recipe. | GKD baseline, Qwen3 |
| Selected tokens (entropy-, error-, or value-gated) | Trains only on the tokens that carry signal; cuts compute and noise. | TIP (top-50% entropy), SelecTKD, AdaKD, Speculative KD (only accepted tokens) |
| Truncated prefix only | Only the first *k* tokens of each rollout. | Fast OPD, prefix-only OPSD |
| Replaced / corrected tokens | Teacher overwrites bad student tokens; student trains on the *fixed* sequence. | Speculative Knowledge Distillation, SCoRe (earliest-error correction), HASS (multi-step harmonisation) |
| As policy gradient samples | Rollout becomes the on-policy data for an RL step; teacher signal becomes the reward / advantage. | SDPO, G-OPD, KDRL, RLAD, REOPOLD |

### Q4 — Where does OPD sit in the post-training pipeline?

| Pipeline slot | What OPD replaces / adds | Examples |
|---|---|---|
| Cold-start (replaces SFT) | Skip teacher-trace SFT; cold-start directly with on-policy KD. | Qwen3 strong-to-weak first phase, Phi-4-Mini-Reasoning |
| Mid-training | Between SFT and RL, refine on student samples. | THUNLP OPD recipe (off-policy SFT → OPD → eval) |
| RL-replacement | Use OPD instead of RL for the final stage. The Qwen3 / Thinking Machines blog claim: ~1/10 RL cost, equal AIME score. | Qwen3 final stage, Thinking Machines tinker-cookbook recipe |
| Inside-RL (dense reward) | KL-to-teacher becomes a per-token dense reward shaping inside GRPO/PPO. | KDRL, G-OPD, RLAD, HDPO, SD-Zero |
| Inter-stage glue | Between RL stages, prevent forgetting / consolidate gains. | GLM-5 cross-stage OPD, Nemotron-Cascade-2 between RL cascades |
| Post-RL compression | Compress long-CoT into short-CoT without entropy collapse. | CRISP/OPSDC |
| Continual-learning anchor | Anchor against an earlier checkpoint to prevent drift during fine-tuning. | SDFT (Shenfeld), Apple Memory-Retaining FT |

### A timeline lens

Sorting by Q1 (teacher source), the field's evolution becomes legible. Larger-external-teacher OPD (A) dominated 2023–2024; the field hit a 17-month gap before production adoption (Qwen3, May 2025) because the bottleneck was infrastructure (vLLM/SGLang/veRL) not algorithms. Privileged-context OPSD (B) and multi-teacher OPD (D) exploded between October 2025 and March 2026 — researchers independently discovered "teacher = self with hint" from many angles within a single quarter. RL-native OPD (Q4 row 4) is the live frontier as of April 2026 and is where the next 12 months of research will concentrate.

```
2023.06   GKD + MiniLLM                       [Q1=A, larger external]   name "OPD"
2023.10   DistillSpec                         [Q1=A]                    OPD applied to spec decoding
2024.02   DistiLLM (ICML 2024)                [Q1=A]                    skew-KL, adaptive on/off
2024.07   Gemma 2 tech report                 [Q1=A]                    first to NAME OPD in a release
2024.07   BOND                                [Q1=C, iterative]         Best-of-N as target distribution
2024.10   Speculative KD                      [Q1=A]                    interleaved propose-and-correct
2025.05   Qwen3 tech report                   [Q1=A → A+B]              industrial proof: OPD ≈ RL @ 1/10 cost
2025.10   Thinking Machines blog (Lu)         [Q1=A]                    one-line recipe → wave goes mainstream
2025.11   GAD (Microsoft)                     [Q1=E, discriminator]     OPD without teacher logits
2026.01   OPSD (Zhao)                         [Q1=B, privileged]        canonical privileged-context OPSD
2026.01   SDPO (ETH/MIT)                      [Q1=B + RL]               OPD-as-RL-objective
2026.02   OPCD / π-Distill / GATES            [Q1=B]                    Microsoft B-family explodes
2026.02   G-OPD / RLAD                        [Q1=A + RL]               OPD = KL-constrained RL formalised
2026.03   MiMo-V2-Flash                       [Q1=D, multi-teacher]     MOPD in production
2026.03   Nemotron-Cascade-2 (NVIDIA)         [Q1=D]                    OPD as inter-stage glue
2026.04   THUNLP Rethinking OPD               [Q1=A diagnostic]         two success conditions
2026.04   Lightning OPD                       [Q1=A engineering]        offline OPD via teacher consistency
```

### Practical decision guide — *"which OPD do I reach for?"*

```
                         Do you have an external model stronger than yours?
                                       │
                  ┌────────────── Yes ──┴── No ─────────────┐
                  │                                          │
        Can you see its logits?                 Do you have verifiable rewards?
                  │                                          │
        ┌── Yes ──┴── No ──┐                       ┌── Yes ──┴── No ──┐
        │                   │                       │                   │
   Q1=A white-box      Q1=A black-box           Q1=A/B + RL          Q1=B privileged
   GKD / MiniLLM /     GAD / Lion /             SDPO / G-OPD /        self-distill
   Qwen3 recipe /      SuperCorrect /           KDRL / RLAD /         OPSD / OPCD /
   verl on-policy /    OVD                      HDPO / SD-Zero        SDFT / CRISP
   slime / SkyRL /
   tinker-cookbook
```

Domain refinements layered on top:

 - **Reasoning (math/code)** — Q1=A with reverse KL is the safe default (Qwen3 / Thinking Machines recipe). For long-CoT compression specifically, switch to Q1=B (CRISP/OPSDC).
 - **Agent / multi-turn** — Q1=B with a "verified-trajectory" privilege (SCoRe earliest-error correction) or Q4=inside-RL (OpenClaw-RL).
 - **VLM** — VOLD or π-Flow are the only flagship Q1=A VLM-OPD recipes; VL-Rethinker / Vision-R1 / Video-R1 are RL-only "OPD-adjacent".
 - **Production at scale** — Q1=D multi-teacher recipes (MiMo-V2-Flash MOPD, Nemotron-Cascade-2). The "one student, many specialist teachers" pattern is replacing single-teacher recipes in flagship releases.
 - **Inference speedup** — different problem entirely; see the dedicated [Speculative-Decoding Distillation](#-speculative-decoding-distillation) section.

### Awesome-list categories (what each section below contains)

The sections below are sorted by Q1 first, then by application:

 - **Surveys, Foundations & Position Papers** — meta-references and the canonical algorithm seeds.
 - **OPD with Larger External Teachers — White-Box** — Q1=A, white-box logits.
 - **OPD with Larger External Teachers — Black-Box / Outcome-Based** — Q1=A or E, black-box.
 - **Self-Distillation with Privileged Context (OPSD)** — Q1=B.
 - **Multi-Teacher / Iterative Self-Distillation** — Q1=C and D.
 - **OPD-RL Hybrids** — Q4=inside-RL, regardless of Q1.
 - **By application — Reasoning / Multimodal / Agent** — cuts across all of Q1.
 - **Speculative-Decoding Distillation** — a parallel universe where the "student" is a draft model and the goal is inference speedup.
 - **Frameworks & Toolkits** — what to actually run.
 - **Industrial / Production Model Reports** — what the labs ship.
 - **Off-Policy KD References** — context only; *not* OPD.

Shorthand used inside technical details:
 - **FKL** = forward KL (teacher‖student, mode-covering); **RKL** = reverse KL (student‖teacher, mode-seeking); **JSD** / **GJSD** = (generalised) Jensen–Shannon; **Skew-KL** / **AKL** = adaptive / skewed KL.
 - **Q1**: A = larger external · B = self+privilege · C = self+earlier-checkpoint · D = multi-teacher · E = discriminator.
 - **Pipeline slot**: cold-start · mid · RL-replacement · inside-RL · inter-stage · post-RL compression · continual-anchor.

---

## Updates
 - 📢 **2026-04-28 — Strict-OPD verification pass**: Every entry was rechecked against the strict OPD definition (student samples its own trajectories + teacher provides supervision on those samples). ~50 entries that turned out to be off-policy SFT, pure RL, pretraining-side KD, or train-free were moved to a structured [Off-Policy KD References](#-off-policy-kd-references-for-context) section with sub-buckets, or removed entirely. See [Caveats](#%EF%B8%8F-caveats--works-that-are-not-opd-despite-naming) for the per-entry rationale.
 - 📢 **2026-04-27 — Initial release**: Coverage 2024-04 → 2026-04. Includes the Tencent OPD survey ([arXiv 2604.00626](https://arxiv.org/abs/2604.00626)) and the Tsinghua THUNLP "Rethinking OPD" recipe ([arXiv 2604.13016](https://arxiv.org/abs/2604.13016)). The Thinking Machines Lab blog is the spiritual centre of the wave; Qwen3's tech report is the production reference recipe.

---

## 🌟 Curator's Picks — where to start

Opinionated must-reads, not the most-cited or most-starred. If you are starting an OPD project today, read these in order.

| # | Why it's the pick | Resource |
|---|---|---|
| 1 | The clearest one-page explanation of why OPD beats both SFT and RL on token efficiency. Skip if you've read it; read first if you haven't. | [Thinking Machines Lab blog (Oct 2025)](https://thinkingmachines.ai/blog/on-policy-distillation/) |
| 2 | The production recipe everyone is now copying. Read §3.4 of the tech report. | [Qwen3 Technical Report](https://arxiv.org/abs/2505.09388) |
| 3 | Reproducible OPD in <200 lines on a real training stack. | [tinker-cookbook recipes/distillation](https://github.com/thinking-machines-lab/tinker-cookbook/tree/main/tinker_cookbook/recipes/distillation) |
| 4 | The "theory of OPD" — when it works, when it fails. Two success conditions named. | [THUNLP Rethinking OPD (arXiv 2604.13016)](https://arxiv.org/abs/2604.13016) |
| 5 | The paper that gave OPSD its name and established the privileged-context pattern (Q1=B). | [Self-Distilled Reasoner / OPSD (arXiv 2601.18734)](https://arxiv.org/abs/2601.18734) |
| 6 | Crystallises OPD as a special case of KL-constrained RL with reward extrapolation (the cleanest "inside-RL" formulation). | [G-OPD (arXiv 2602.12125)](https://arxiv.org/abs/2602.12125) |
| 7 | A useful catalogue of 50+ methods to mine for related work — read as an index, not as a taxonomy. | [Tencent OPD Survey (arXiv 2604.00626)](https://arxiv.org/abs/2604.00626) |
| 8 | The single most diverse open-source OPD trainer collection — `gkd`, `gold`, `minillm`, `sdft`, `self_distillation`, `sdpo` — read the source. | [TRL `experimental/`](https://github.com/huggingface/trl/tree/main/trl/experimental) |
| 9 | The black-box-OPD seed paper — adversarial discriminator as on-policy reward model. Important if you're distilling from a closed API. | [GAD / Black-Box OPD (arXiv 2511.10643)](https://arxiv.org/abs/2511.10643) |
| 10 | The empirical failure-modes paper. Saves a week of debugging. | [Revisiting OPD (arXiv 2603.25562)](https://arxiv.org/abs/2603.25562) |

---

## 🔭 Open Problems & What's Missing

What this list also tells you, by the gaps it has:

1. **OPD has no large-scale empirical scaling-law study.** Qwen3 reports 1/10 RL cost on AIME at 8B; nobody has plotted OPD compute-vs-quality curves across model scales. The Thinking Machines blog touches this but doesn't sweep it.
2. **Cross-tokenizer OPD is half-solved.** [DSKD](https://github.com/songmzhang/DSKD), [DSKDv2](https://github.com/songmzhang/DSKDv2), and ULD exist, but none are production-deployed and the dual-space projector approach doesn't cleanly extend to MoE teachers. This will matter the moment someone wants to OPD-distill a Qwen3-235B-A22B teacher into a Llama-class student.
3. **No public Q1=B production report exists.** Every privileged-context-OPSD paper is research-scale. We're missing the "Qwen3 of OPSD" — a flagship that uses privileged-context self-distillation as its main post-training recipe and reports head-to-head numbers vs. RL.
4. **Pretraining-time OPD is wide open.** Llama 4's "codistillation" of Maverick from Behemoth is the only public production example. The pretraining-time KD literature ([MiniPLM](https://arxiv.org/abs/2410.17215), [Distilled Pretraining](https://arxiv.org/abs/2509.01649)) is mostly off-policy (data-side) — true student-rollout pretraining-OPD has barely started.
5. **OPD for non-text modalities is sparse.** [VOLD](https://arxiv.org/abs/2510.23497) (LLM→VLM), [π-Flow](https://github.com/Lakonik/piFlow) (image), [Step-Audio-R1](https://github.com/stepfun-ai/Step-Audio-R1) (audio), [RPD](https://arxiv.org/abs/2503.05833) (VLA), [X-OPD](https://arxiv.org/abs/2603.24596) (speech) are individually strong but there's no unified multimodal OPD framework.
6. **Evaluation is fragmented.** Half the OPD papers benchmark on AIME'24, half on GSM8K/MATH, a few on HMMT25. No standard "OPD-Bench" exists. The community would benefit from one.
7. **Theoretical guarantees lag.** OPD inherits the DAGGER-style O(T) regret bound from interactive imitation learning, but the actual divergence choice (FKL vs RKL vs Skew) lacks a clean theoretical preference under realistic teacher–student gaps. [Constrained OPD (CMDP)](https://arxiv.org/abs/2509.22921) and [G-OPD](https://arxiv.org/abs/2602.12125) are first steps; there's a thesis in here.
8. **Inside-RL OPD unification is in flux.** SDPO, KDRL, RLAD, G-OPD, REOPOLD, SD-Zero, RLSD, HDPO all propose slightly different ways of fusing OPD with RLVR/GRPO/PPO. As of Apr 2026 there's no consensus on the right form. This is where the next 6–12 months of research will concentrate.
9. **OPD safety/alignment**: black-box OPD can extract aligned behaviour from a closed API, but there's no work on whether OPD-distilled students *inherit* the teacher's alignment properties or just its surface tokens.
10. **Tooling**: the awesome-list itself reveals that OPD support is *retro-fitted* to most RL frameworks (verl `recipe/`, slime `examples/`, SkyRL PR-as-feature). A purpose-built "OPD-first" framework probably doesn't need to exist, but if someone built one with first-class privileged-context teachers, multi-teacher MOPD scheduling, and inside-RL KL-as-reward fusion, the field would adopt it overnight.

---

## 📚 Surveys, Foundations & Position Papers

| Resource | 🌟 Stars | Date | Org | Paper / Link |
| :----: | :----: | :----: |  :----: | :----: |
| **Thinking Machines Lab — On-Policy Distillation (blog)** | ![](https://img.shields.io/badge/blog_post-3.2k_cookbook-blue?style=for-the-badge) | 2025.10 | Thinking Machines Lab (Kevin Lu et al.) | [Blog](https://thinkingmachines.ai/blog/on-policy-distillation/) · [tinker-cookbook](https://github.com/thinking-machines-lab/tinker-cookbook) |
| [tinker-cookbook](https://github.com/thinking-machines-lab/tinker-cookbook) | <img src="https://img.shields.io/github/stars/thinking-machines-lab/tinker-cookbook?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.10 | Thinking Machines Lab | Reference impl. of the OPD recipe on the Tinker SDK |
| **A Survey of On-Policy Distillation for LLMs** | ![](https://img.shields.io/badge/Survey-paper-845C40?style=for-the-badge) | 2026.04 | Tencent (Mingyang Song & Mao Zheng) | [arXiv 2604.00626](https://arxiv.org/abs/2604.00626) |
| **Rethinking On-Policy Distillation: Phenomenology, Mechanism & Recipe** | <img src="https://img.shields.io/github/stars/thunlp/OPD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.04 | Tsinghua THUNLP | [arXiv 2604.13016](https://arxiv.org/abs/2604.13016) · [thunlp/OPD](https://github.com/thunlp/OPD) |
| **Revisiting On-Policy Distillation: Failure Modes & Simple Fixes** | ![](https://img.shields.io/badge/Diagnosis-paper-845C40?style=for-the-badge) | 2026.03 | CASIA (Fu et al.) | [arXiv 2603.25562](https://arxiv.org/abs/2603.25562) |
| **Lightning OPD: Efficient Post-Training with Offline OPD** | ![](https://img.shields.io/badge/Recipe-paper-845C40?style=for-the-badge) | 2026.04 | Wu, Han, Cai | [arXiv 2604.13010](https://arxiv.org/abs/2604.13010) |
| **GKD: On-Policy Distillation of Language Models — Learning from Self-Generated Mistakes** | ![](https://img.shields.io/badge/Seminal-ICLR_2024-4E6813?style=for-the-badge) | 2023.06 | Google DeepMind (Agarwal et al.) | [arXiv 2306.13649](https://arxiv.org/abs/2306.13649) — implemented in [TRL `GKDTrainer`](https://github.com/huggingface/trl/blob/main/trl/experimental/gkd/gkd_trainer.py) |

<details>
<summary>📋 Click to view technical details</summary>

| Resource | Loss / Divergence | Data | Teacher Access | Granularity | Notes |
| :----: | :----: | :----: | :----: | :----: | :---- |
| Thinking Machines blog | Reverse KL (student‖teacher) | Student rollouts | White-box | Token | "Swap KL ref model for stronger teacher" recipe; one-line addition to RL trainer. Replicates Qwen3 result at ~1/10 RL cost. |
| Tencent OPD Survey | (survey) | (survey) | (survey) | (survey) | Catalogues 50+ methods; useful as a reference index. |
| THUNLP Rethinking OPD | Reverse KL with progressive top-K alignment | Student | White-box | Token | Identifies two success conditions: compatible thinking patterns + genuinely new teacher capability. Recipe = **off-policy cold-start + teacher-aligned prompt selection**. |
| Revisiting OPD | Truncated reverse KL + top-p sampling + special-token masking | Student | White-box | Token (filtered) | Diagnoses 3 failure modes: imbalanced one-token signal, unreliable prefix guidance, tokenizer mismatch. |
| Lightning OPD | Cached teacher log-probs over SFT rollouts (offline OPD) | Student (cached) | White-box | Token | Introduces "teacher consistency" — same teacher must be used for SFT and OPD or else gradient bias. Eliminates the live teacher server. |
| GKD (Agarwal) | Generalised JSD (FKL/RKL configurable) | Mixed (`λ` interpolates teacher↔student) | White-box | Token | The seminal paper that named OPD; introduced student-self-rollout supervision. |

</details>

> 📝 **Strictness notes** (against the strict OPD definition `C1: student samples its own trajectories during training` + `C2: teacher provides supervision on those samples`)
> - **Lightning OPD** — ⚠️ partially satisfies C1: teacher log-probs are pre-computed *once* over SFT rollouts and reused during training; student doesn't actively sample during the OPD step. Authors call this "offline OPD" explicitly. Listed in OPD because the data is past-student-generated rollouts, not teacher-generated.

---

## 🔬 OPD with Larger External Teachers — White-Box (Q1=A)

White-box methods use **teacher logits / log-probabilities** to supervise the student on **student-generated rollouts**. Each entry below has been verified to (a) train on student rollouts and (b) operate at the token level.

Methods that turned out to be off-policy / pure-loss-function / pretraining-side / RL-style on verification have been moved to [Off-Policy KD References](#-off-policy-kd-references-for-context) or [OPD-RL Hybrids](#-opd-rl-hybrids); see those sections.

| Github Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [microsoft/LMOps `/minillm`](https://github.com/microsoft/LMOps/tree/main/minillm) (MiniLLM) | <img src="https://img.shields.io/github/stars/microsoft/LMOps?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2023.06 | Microsoft / Tsinghua | [arXiv 2306.08543](https://arxiv.org/abs/2306.08543) (ICLR 2024) |
| [jongwooko/distillm](https://github.com/jongwooko/distillm) | <img src="https://img.shields.io/github/stars/jongwooko/distillm?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.02 | KAIST / Microsoft | [arXiv 2402.03898](https://arxiv.org/abs/2402.03898) (ICML 2024) |
| [jongwooko/distillm-2](https://github.com/jongwooko/distillm-2) | <img src="https://img.shields.io/github/stars/jongwooko/distillm-2?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | KAIST / Microsoft | [arXiv 2503.07067](https://arxiv.org/abs/2503.07067) (ICML 2025 Oral) |
| [songmzhang/DSKDv2](https://github.com/songmzhang/DSKDv2) (cross-tokenizer; supports on-policy mode) | <img src="https://img.shields.io/github/stars/songmzhang/DSKDv2?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | BJTU | [arXiv 2504.11426](https://arxiv.org/abs/2504.11426) |
| [RUCBM/G-OPD](https://github.com/RUCBM/G-OPD) | <img src="https://img.shields.io/github/stars/RUCBM/G-OPD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.02 | RUC / Tencent | [arXiv 2602.12125](https://arxiv.org/abs/2602.12125) |
| Speculative KD (interleaved propose-and-correct) | 📄 paper-only | 2024.10 | UCSB / Google | [arXiv 2410.11325](https://arxiv.org/abs/2410.11325) (ICLR 2025) |
| AdaSwitch (on-/off-policy switching) | 📄 paper-only | 2025.10 | RUC / Baidu | [arXiv 2510.07842](https://arxiv.org/abs/2510.07842) |
| Constrained OPD (CMDP) | 📄 paper-only | 2025.09 | Huawei Noah's Ark | [arXiv 2509.22921](https://arxiv.org/abs/2509.22921) |
| REOPOLD (Relaxed OPD) | 📄 paper-only | 2026.03 | KAIST / Microsoft | [arXiv 2603.11137](https://arxiv.org/abs/2603.11137) |
| PACED (frontier curriculum self-distill) | 📄 paper-only | 2026.03 | LinkedIn | [arXiv 2603.11178](https://arxiv.org/abs/2603.11178) |
| Fast OPD (prefix-truncated) | 📄 paper-only | 2026.02 | Industrial | [arXiv 2602.15260](https://arxiv.org/abs/2602.15260) |
| Entropy-Aware OPD | 📄 paper-only | 2026.03 | KAIST / IBM | [arXiv 2603.07079](https://arxiv.org/abs/2603.07079) |
| Veto (Stable OPD) | 📄 paper-only | 2026.01 | SNU | [arXiv 2601.07155](https://arxiv.org/abs/2601.07155) (ACL 2026 Findings) |
| TIP (Token Importance) | 📄 paper-only | 2026.04 | Meta / LinkedIn | [arXiv 2604.14084](https://arxiv.org/abs/2604.14084) |
| SCOPE (signal-calibrated dual-path) | 📄 paper-only | 2026.04 | USTC / Meituan / Fudan | [arXiv 2604.10688](https://arxiv.org/abs/2604.10688) |
| TSD-KD (token-selective dual KD) | 📄 paper-only | 2026.03 | Korea Univ. | [arXiv 2603.13260](https://arxiv.org/abs/2603.13260) (ICLR 2026) |
| SelecTKD | 📄 paper-only | 2025.10 | XJTU | [arXiv 2510.24021](https://arxiv.org/abs/2510.24021) |

<details>
<summary>📋 Click to view technical details</summary>

| Method | Loss / Divergence | Data | Granularity | Domain | Notes |
| :----: | :----: | :----: | :----: | :----: | :---- |
| MiniLLM | Reverse KL via policy gradient | Student | Sequence (PG) | General | The seminal "OPD" recipe by Yuxian Gu et al.; predates GKD by days. Mode-seeking. |
| DistiLLM | Skewed-KL (mix of FKL/RKL) | Mixed (adaptive off→on, with student samples) | Token | General | Skew parameter `α` interpolates between FKL and RKL; importance-reweighted student samples. |
| DistiLLM-2 | Contrastive: Skew-FKL on teacher data + Skew-RKL on student data | Mixed | Token | General | Asymmetric losses on each data source; ICML 2025 oral. |
| DSKDv2 | KL in dual aligned space; explicit on-policy mode | Student | Token | Cross-tokenizer | Cross-vocabulary distillation; supports both on/off-policy. |
| G-OPD / ExOPD | Reverse KL + scaled reward extrapolation | Student | Token | General | Generalises OPD as KL-constrained RL; allows reward scale > 1 to "exceed" the teacher. |
| Speculative KD (Xu) | Interleaved propose-and-correct (gated KL) | Student-proposed, teacher-corrected | Token | General | Bridges teacher-student gap via interleaved sampling. |
| AdaSwitch | Adaptive on/off-policy switching | Mixed | Token | General | Switches between teacher-data and student-rollout based on divergence threshold. |
| Constrained OPD | KL-constrained CMDP | Student | Token | General | Hard KL constraint instead of soft penalty. Borderline OPD-RL. |
| REOPOLD | Mixture-based reward clipping + entropy-based dynamic sampling | Student | Token | Reasoning | "Relaxed OPD"; views OPD as policy optimisation with teacher-student log-ratio reward. |
| PACED | Frontier curriculum at student competence boundary | Student | Token | General | Self-distill style (Q1=B/C); difficulty weighting `w(p)=p(1−p)`. |
| Fast OPD | Prefix-truncated distillation reducing FLOPs | Student | Token (truncated) | Reasoning | 2× to 47× speedup via reasoning-prefix truncation. |
| Entropy-Aware OPD | Switch between FKL and RKL based on teacher entropy | Student | Token | Reasoning | When teacher entropy high → FKL; low → RKL. |
| Veto | Logit-space geometric bridge with adaptive gradient veto | Student | Token | General | Adaptive Target Reformulation. |
| TIP | Top-50% high-entropy student tokens carry the OPD signal | Student (selected) | Token (filtered) | Reasoning | ~47% memory savings; only entropy-high student tokens trained. |
| SCOPE | Teacher-PPL-weighted KL on incorrect rollouts; student-PPL-weighted MLE on correct | Student | Token | Reasoning | Signal-Calibrated OPD with Dual-Path Adaptive Weighting; verifier-routing. |
| TSD-KD | Indirect (student-propose / teacher re-rank) + direct selective logit KD | Mixed | Token (selected) | General | Hybrid; partial OPD + partial preference. |
| SelecTKD | Selective token-weighted KD; supports on-policy data | Student (optional) | Token (selected) | General | Selective Token-Weighted KD; on-policy mode supported. |

</details>

---

## 🎭 OPD with Black-Box / Outcome-Based Teachers (Q1=A black-box · Q1=E discriminator)

When the teacher is **API-only** (no logits), OPD uses scalar rewards, verbal scores, preferences, or adversarial discriminators — all evaluated on **student rollouts**. We removed entries on verification that turned out to use static teacher data only (Lion, SuperCorrect, DAIL, SODA — see [Off-Policy KD References](#-off-policy-kd-references-for-context)).

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [microsoft/LMOps `/gad`](https://github.com/microsoft/LMOps) (GAD — Black-Box OPD) | <img src="https://img.shields.io/github/stars/microsoft/LMOps?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.11 | Microsoft Research | [arXiv 2511.10643](https://arxiv.org/abs/2511.10643) · [project](https://ytianzhu.github.io/Generative-Adversarial-Distillation/) |
| OVD (On-policy Verbal Distillation) | 📄 paper-only | 2026.01 | HKU / Huawei | [arXiv 2601.21968](https://arxiv.org/abs/2601.21968) |
| ORPO-Distill | 📄 paper-only | 2025.09 | Industrial | [arXiv 2509.25100](https://arxiv.org/abs/2509.25100) |

<details>
<summary>📋 Click to view technical details</summary>

| Method | Feedback Signal | Data | Granularity | Domain | Notes |
| :----: | :----: | :----: | :----: | :----: | :---- |
| GAD (Generative Adversarial Distillation) | Discriminator (on-policy reward model) | Student | Sequence | General | A trained discriminator distinguishes student outputs from teacher (e.g. GPT-5) responses; minimax game makes the discriminator co-evolve into an on-policy reward model. Qwen2.5-14B student becomes comparable to GPT-5-Chat on LMSYS. |
| OVD | Verbal scores (0–9) on student trajectories | Student | Sequence | General | Replaces token-level logit matching with verbal scoring; +25.7% over baselines. |
| ORPO-Distill | Student-Generated Outputs (SGO) + ORPO contrastive | Mixed (student-generated negatives, teacher positives) | Sequence | Cross-architecture | "Mixed-policy strategy utilizing student-generated outputs"; NeurIPS 2025 WS. |

</details>

---

## ♻️ Self-Distillation with Privileged Context — OPSD (Q1=B)

**Same model = teacher = student**, but the teacher is conditioned on something the student doesn't see (verified trace, ground-truth answer, "be concise" prefix, longer context, document, …). The gap exists *because of the conditioning*, not weights.

Several entries previously listed here turned out on verification to use static teacher data or a fixed self-rewritten dataset rather than student rollouts; they have been moved to [Off-Policy KD References](#-off-policy-kd-references-for-context). SPIN was reclassified to [Iterative Self-Bootstrapping (Q1=C)](#-iterative-self-bootstrapping-q1c).

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [siyan-zhao/OPSD](https://github.com/siyan-zhao/OPSD) (Self-Distilled Reasoner) | <img src="https://img.shields.io/github/stars/siyan-zhao/OPSD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | UCLA / Meta FAIR | [arXiv 2601.18734](https://arxiv.org/abs/2601.18734) · [blog](https://siyan-zhao.github.io/blog/2026/opsd/) |
| [HJSang/CRISP_Reasoning_Compression](https://github.com/HJSang/CRISP_Reasoning_Compression) (OPSDC / CRISP) | <img src="https://img.shields.io/github/stars/HJSang/CRISP_Reasoning_Compression?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.03 | LinkedIn | [arXiv 2603.05433](https://arxiv.org/abs/2603.05433) |
| [idanshen/Self-Distillation](https://github.com/idanshen/Self-Distillation) (SDFT-Continual) | <img src="https://img.shields.io/github/stars/idanshen/Self-Distillation?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | MIT / ETH | [arXiv 2601.19897](https://arxiv.org/abs/2601.19897) |
| [microsoft/LMOps `/opcd`](https://github.com/microsoft/LMOps) (OPCD — On-Policy Context Distillation) | <img src="https://img.shields.io/github/stars/microsoft/LMOps?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.02 | Microsoft Research | [arXiv 2602.12275](https://arxiv.org/abs/2602.12275) |
| [microsoft/LMOps `/oel`](https://github.com/microsoft/LMOps) (OEL — Online Experiential Learning) | <img src="https://img.shields.io/github/stars/microsoft/LMOps?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.03 | Microsoft Research | [arXiv 2603.16856](https://arxiv.org/abs/2603.16856) |
| [jwkirchenbauer/mtp-lm](https://github.com/jwkirchenbauer/mtp-lm) (MTP Self-Distill) | <img src="https://img.shields.io/github/stars/jwkirchenbauer/mtp-lm?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.02 | UMD / LLNL | [arXiv 2602.06019](https://arxiv.org/abs/2602.06019) |
| [apple/ml-ssd](https://github.com/apple/ml-ssd) (Apple — Embarrassingly Simple Self-Distillation) | ![](https://img.shields.io/badge/Apple_MLR-2026.04-grey?style=for-the-badge&logo=apple) | 2026.04 | Apple MLR | [arXiv 2604.01193](https://arxiv.org/abs/2604.01193) |
| GATES (Self-Distillation under Privileged Context) | 📄 paper-only | 2026.02 | UMD | [arXiv 2602.20574](https://arxiv.org/abs/2602.20574) |
| OPSDL (Long-Context Self-Distillation) | 📄 paper-only | 2026.04 | Baidu | [arXiv 2604.17535](https://arxiv.org/abs/2604.17535) |

<details>
<summary>📋 Click to view technical details</summary>

| Method | Privileged Context (Teacher) | Loss / Divergence | Granularity | Domain | Notes |
| :----: | :----: | :----: | :----: | :----: | :---- |
| OPSD (Self-Distilled Reasoner) | Verified reasoning trace | Per-token RKL with point-wise clipping | Token | Math reasoning | Same-model OPSD; matches GRPO with 1×8 rollouts and 1024 length vs. GRPO's 8×16 / 16k. **The canonical OPSD paper.** Built on TRL's GOLD trainer. |
| CRISP / OPSDC | "Be concise" instruction prefix | Per-token RKL on student rollouts | Token | Reasoning compression | Compresses long-CoT without entropy collapse (unlike RL-with-length-penalty). |
| SDFT-Continual (idanshen) | Demo-conditioned same model | RKL on student rollouts vs. demo-conditioned teacher | Token | Continual learning | Self-distillation enables continual learning. |
| OPCD | In-context-knowledge-augmented same model | RKL on student rollouts | Token | Knowledge internalisation | Internalise context to be faithful even after context is removed. |
| OEL (Online Experiential Learning) | Same model with interactive game environment | RKL on student rollouts | Token | Game / planning | Self-distillation on interactive trajectories. |
| MTP Self-Distill | Multi-token prediction same model | RKL on student rollouts | Token | General | Multi-Token Prediction via Self-Distillation. Author-stated on-policy. |
| Apple SSD | Same model w/ temperature/truncation sampling | Cross-entropy on its own samples | Sequence | Code generation | "Embarrassingly simple" — sample, then SFT on those samples. Degenerate OPSD; "decoding-config" privilege. |
| GATES | Document-conditioned tutor (same model) | RKL gated by tutor consensus | Token (gated) | Document QA | Both tutor and student sample rollouts; on-policy student-rollout updates contribute "modest additional improvement" on top of off-policy distillation. Mixed. |
| OPSDL | Short-context same model | Point-wise RKL | Token | Long-context | On-Policy Self-Distillation for Long-Context LMs. |

</details>

> 📝 **Strictness notes**
> - **Apple SSD** — ⚠️ C2 is degenerate: no teacher KL signal; pure self-generated SFT (sample with temperature/truncation, then SFT on those samples). Closer to STaR-style self-bootstrapping than to OPSD. Kept because the "teacher" is the same model with a different decoding config (Q1=B by privilege).
> - **GATES** — ⚠️ Authors' own ablation says off-policy trajectory-level distillation drives the *primary gains*; on-policy student-rollout updates contribute only "modest additional improvement". Mixed; the OPSD leg is genuine but secondary.

### 🔁 Iterative Self-Bootstrapping (Q1=C)

Same model is the teacher, but as a *frozen earlier checkpoint*, not a privileged-context view. The teacher snapshot is frozen for one round, the student trains, then the snapshot rolls forward. Listed separately because the supervision is typically sequence-level / preference, not per-token logit-distillation.

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [uclaml/SPIN](https://github.com/uclaml/SPIN) (Self-Play Fine-Tuning) | <img src="https://img.shields.io/github/stars/uclaml/SPIN?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.01 | UCLA | [arXiv 2401.01335](https://arxiv.org/abs/2401.01335) (ICML 2024) |
| [microsoft/rStar](https://github.com/microsoft/rStar) (rStar / rStar-Math / rStar2-Agent) | <img src="https://img.shields.io/github/stars/microsoft/rStar?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | Microsoft Research | [rStar-Math 2501.04519](https://arxiv.org/abs/2501.04519) · [rStar2-Agent 2508.20722](https://arxiv.org/abs/2508.20722) |

> 📝 **Strictness notes (Q1=C)**
> - **SPIN** — ⚠️ C1 ✓ (student samples), but C2 fails strict per-token logit form: supervision is *sequence-level DPO preference* against the previous frozen checkpoint. More accurately "iterative on-policy DPO" than per-token OPD. Kept because the "teacher = previous self" pattern is what people search for in OPD lists.
> - **rStar / rStar-Math / rStar2-Agent** — ⚠️ MCTS-filtered student samples + SFT; the "teacher signal" is a step-level PPM / discriminator score, not per-token logit KL. Iterative self-improvement, not classical OPD.

---

## 🤝 OPD-RL Hybrids — Inside-RL OPD

Methods that fuse OPD with **RLVR / GRPO / PPO / DPO**. Teacher logits become a dense reward shaping or trust-region anchor inside an RL objective; or BoN / preference signals are used as the imitation target.

Newly added on verification: **AlignDistil** (RLHF-equivalent distillation), **BOND / Faster WIND** (sequence-level Best-of-N as target), **KETCHUP** (k-step RL-based KD), **𝒳-KD / DDT** (IRL-style), **LUFFY** (mixed-policy GRPO with off-policy traces). Removed on verification: **RLKD** (only sequence-level structural reward), **ExGRPO** (pure RL, no teacher), **REDI** (offline R1 traces, no student rollouts).

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [lasgroup/SDPO](https://github.com/lasgroup/SDPO) (RL via Self-Distillation) | <img src="https://img.shields.io/github/stars/lasgroup/SDPO?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | ETH / MIT | [arXiv 2601.20802](https://arxiv.org/abs/2601.20802) · [project](https://self-distillation.github.io/SDPO) |
| [Gen-Verse/OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL) | <img src="https://img.shields.io/github/stars/Gen-Verse/OpenClaw-RL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.03 | Gen-Verse | [arXiv 2603.10165](https://arxiv.org/abs/2603.10165) — combines GRPO + OPD |
| [Gen-Verse/Open-AgentRL](https://github.com/Gen-Verse/Open-AgentRL) | <img src="https://img.shields.io/github/stars/Gen-Verse/Open-AgentRL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.02 | Gen-Verse | RLAnything / DemyAgent multi-domain |
| [songmzhang/AlignDistil](https://github.com/songmzhang/AlignDistil) (RLHF-equivalent KD) | <img src="https://img.shields.io/github/stars/songmzhang/AlignDistil?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | BJTU / Tencent | [arXiv 2503.02832](https://arxiv.org/abs/2503.02832) (ACL 2025) |
| [ElliottYan/LUFFY](https://github.com/ElliottYan/LUFFY) (mixed-policy GRPO) | <img src="https://img.shields.io/github/stars/ElliottYan/LUFFY?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | Westlake U. | [arXiv 2504.14945](https://arxiv.org/abs/2504.14945) |
| [Corleno/KEPO](https://github.com/Corleno/KEPO) | <img src="https://img.shields.io/github/stars/Corleno/KEPO?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | Industrial | [arXiv 2602.00400](https://arxiv.org/abs/2602.00400) |
| BOND (Best-of-N Distillation) | 📄 paper-only | 2024.07 | Google DeepMind | [arXiv 2407.14622](https://arxiv.org/abs/2407.14622) |
| Faster WIND (iterative BoN) | 📄 paper-only | 2024.10 | CMU / Google | [arXiv 2410.20727](https://arxiv.org/abs/2410.20727) (AISTATS 2025) |
| KETCHUP (k-step RL-KD) | 📄 paper-only | 2025.04 | U. Alberta | [arXiv 2504.19024](https://arxiv.org/abs/2504.19024) |
| 𝒳-KD (IRL-style) | 📄 paper-only | 2026.02 | BUPT | [arXiv 2602.12674](https://arxiv.org/abs/2602.12674) |
| DDT (on-policy SFT theory) | 📄 paper-only | 2026.02 | MSRA / Shopee | [arXiv 2602.12222](https://arxiv.org/abs/2602.12222) |
| RLAD (Reinforcement-aware KD) | 📄 paper-only | 2026.02 | AWS | [arXiv 2602.22495](https://arxiv.org/abs/2602.22495) |
| KDRL (Joint KD + RL) | 📄 paper-only | 2025.06 | HIT / Huawei | [arXiv 2506.02208](https://arxiv.org/abs/2506.02208) |
| Self-Distilled RLVR (RLSD) | 📄 paper-only | 2026.04 | Multi-org | [arXiv 2604.03128](https://arxiv.org/abs/2604.03128) |
| HDPO (Hybrid Distillation PO) | 📄 paper-only | 2026.03 | NVIDIA | [arXiv 2603.23871](https://arxiv.org/abs/2603.23871) |
| Self-Distillation Zero (SD-Zero) | 📄 paper-only | 2026.04 | Princeton / Toronto / CMU | [arXiv 2604.12002](https://arxiv.org/abs/2604.12002) |
| Probing-to-Refine (EI / EXGRPO) | 📄 paper-only | 2026.03 | UNC / ASU | [arXiv 2603.19266](https://arxiv.org/abs/2603.19266) |

<details>
<summary>📋 Click to view technical details</summary>

| Method | Inner RL | Teacher Role | Data | Granularity | Domain | Notes |
| :----: | :----: | :----: | :----: | :----: | :----: | :---- |
| SDPO | Custom self-distillation policy gradient | Feedback-conditioned same model = self-teacher | Student | Token | Code, tool-use, science | Sample student rollout, get tokenised feedback, re-evaluate under feedback-conditioned self-teacher, distill the corrected next-token distribution back into policy. |
| OpenClaw-RL | GRPO + OPD | Judge model extracts hindsight hints, teacher token-logprob gap = directional advantage | Mixed | Token | Terminal / GUI / SWE / Tool-call | Unifies binary RL and OPD in one trainer. |
| Open-AgentRL | GRPO-TCR | Multi-domain teachers | Student | Token | Reasoning / GUI / Coding | Includes process-reward modelling via SandboxFusion. |
| AlignDistil | RLHF-equivalent KD | DPO-derived combination of DPO model + ref-model logits | Student | Token | Alignment | Re-frames DPO as policy distillation. |
| LUFFY | Mixed-Policy GRPO + policy shaping | Off-policy R1 traces inserted into student rollouts | Mixed | Token + sequence | Reasoning | "Learn to reason under off-policy guidance". On-policy student-roll + off-policy teacher-trace mix. |
| KEPO | Knowledge-enhanced PO | Knowledge-base teacher | Mixed | Sequence | Reasoning | Adds KB grounding to preference RL. |
| BOND | Best-of-N distillation | Same model's BoN target | Student (iterative) | Sequence | Alignment | Treats Best-of-N as the target distribution; iterative anchor; Jeffreys divergence. |
| Faster WIND | Win-rate dominance | Same model BoN | Student (iterative) | Sequence | Alignment | Game-theoretic acceleration of BOND. |
| KETCHUP | k-step return REINFORCE on KD | External teacher | Student | Sequence | General | RL-based KD with k-step Bellman returns. |
| 𝒳-KD | AVRIL inverse-RL | Joint reward + policy distillation | Student | Token + sequence | General | IRL-flavoured experiential KD. |
| DDT | On-policy SFT theory | Theoretical | Student | Token | General | Distribution Discriminant Theory; foundations for on-policy SFT. |
| RLAD | PPO/GRPO ratio anchored to teacher–old-policy mixture | External teacher (Qwen3-32B) | Student | Token | Reasoning | Trust-region likelihood-ratio. |
| KDRL | Joint reverse-KL + GRPO rule-based reward | External teacher (Skywork-OR1) | Student | Token + outcome | Reasoning | Unified KD + RL objective. |
| Self-Distilled RLVR (RLSD) | RLVR direction + teacher evidence-ratio modulates magnitude | Same model + privileged answer | Student | Token + outcome | Reasoning | Combines self-distillation magnitudes with RLVR directions. |
| HDPO | RL on most prompts; on "cliff" prompts generate privileged rollouts and self-distill | Same model w/ privilege | Student | Token | Reasoning | Privileged self-distillation as RL fallback. |
| SD-Zero | Generator + Reviser share weights; reviser supplies dense supervision | Same model = reviser | Student | Token | Reasoning | Turns binary rewards into dense supervision. |
| Probing-to-Refine | "Explanatory probes" force logical articulation; GRPO + dialogue-structure reward | Self-probe | Student | Sequence | Reasoning | Reinforcement Distillation via Explanatory Inversion. |

</details>

> 📝 **Strictness notes**
> - **LUFFY** — ⚠️ Mixed-policy: half on-policy student rollouts (C1+C2 ✓) + half *off-policy R1 traces* inserted into GRPO (C1 ✗ on the off-policy half). Net is OPD-flavor with off-policy import.
> - **BOND, Faster WIND** — ⚠️ Q1=C iterative; teacher = same model's BoN distribution. Loss is Jeffreys / win-rate-dominance at the **sequence level** — *no per-token logit supervision* (C2 partially fails strict form). More accurately "on-policy iterative alignment" than OPD.
> - **KETCHUP** — ⚠️ Sequence-level RL-based KD with k-step Bellman returns; the paper itself self-describes as "RL-based KD". Closer to RL with KD-anchor reward than per-token OPD.
> - **𝒳-KD** — ⚠️ Built on AVRIL inverse-RL framework with joint reward modeling; closer to IRL+OPD hybrid than pure OPD.
> - **DDT** — ⚠️ Theoretical foundations paper for "on-policy SFT" (Distribution Discriminant Theory); not a specific deployable algorithm. Kept for completeness.
> - **KEPO, Open-AgentRL, Probing-to-Refine** — ⚠️ C1 ✓ (on-policy student rollouts), but the per-token KL component vs. sequence-level reward shaping vs. preference optimization is not fully resolved from abstracts. Listed because the papers self-describe as OPD/on-policy distillation but exact form of C2 needs full-paper reading.

---

## 🧠 Reasoning OPD (by application)

Genuine OPD work on math / code / long-CoT reasoning. Off-policy SFT-distill from R1, pure RL methods (Skywork-OR1, SimpleRL-Zoo, Time-R1), and analysis-only papers were moved to [Off-Policy KD References](#-off-policy-kd-references-for-context) or [Surveys](#-surveys-foundations--position-papers); each had no student-rollout-with-teacher-supervision component.

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [thunlp/OPD](https://github.com/thunlp/OPD) (Rethinking OPD recipe) | <img src="https://img.shields.io/github/stars/thunlp/OPD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.04 | Tsinghua THUNLP | [arXiv 2604.13016](https://arxiv.org/abs/2604.13016) |
| [RUCBM/G-OPD](https://github.com/RUCBM/G-OPD) (cross-list) | <img src="https://img.shields.io/github/stars/RUCBM/G-OPD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.02 | RUC / Tencent | [arXiv 2602.12125](https://arxiv.org/abs/2602.12125) |
| OPD for Autonomous Vehicle Motion Planning | 📄 paper-only | 2026.04 | Academic | [arXiv 2604.07944](https://arxiv.org/abs/2604.07944) |

The reasoning-OPD canon already lives across **OPSD** (siyan-zhao/OPSD, CRISP), **Self-Distillation Q1=C** (rStar / rStar-Math), **OPD-RL Hybrids** (LUFFY, RLAD, KDRL, RLSD, HDPO, SD-Zero), and **White-Box** (REOPOLD, Fast OPD, Entropy-Aware OPD, TIP, SCOPE, PACED). This section only lists items not already covered above.

<details>
<summary>📋 Click to view technical details</summary>

| Method | Loss / Objective | Data | Teacher | Granularity | Base / Benchmark | Notes |
| :----: | :----: | :----: | :----: | :----: | :----: | :---- |
| Rethinking OPD (THUNLP) | RKL with progressive top-K alignment + off-policy cold-start | Mixed | White-box (Qwen3-4B/1.7B teacher pairs) | Token | Math reasoning | Identifies *teacher-novelty* and *thinking-pattern compatibility* as success conditions. |
| OPD for AV Motion Planning | GPT-Driver framework + GKD on student-generated trajectories | Student | White-box (LLM teacher) | Token | Driving | 5× model-size reduction. |

</details>

---

## 🖼️ Multimodal OPD (VLM, Video, Audio, Image)

Strict OPD work in non-text modalities. Many "R1"/"GRPO" multimodal models that bear the brand are pure RL (no teacher-distillation loss) and were moved to [Caveats — Related RL but NOT OPD](#-caveats--works-that-are-not-opd-despite-naming).

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [Lakonik/piFlow](https://github.com/Lakonik/piFlow) (π-Flow — image / flow OPD) | <img src="https://img.shields.io/github/stars/Lakonik/piFlow?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.10 | Multi-org | [arXiv 2510.14974](https://arxiv.org/abs/2510.14974) (ICLR 2026) |
| [stepfun-ai/Step-Audio-R1](https://github.com/stepfun-ai/Step-Audio-R1) | <img src="https://img.shields.io/github/stars/stepfun-ai/Step-Audio-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.11 | StepFun | [arXiv 2511.15848](https://arxiv.org/abs/2511.15848) |
| VOLD (LLM→VLM OPD) | 📄 paper-only | 2025.10 | INRIA / Goethe Univ. | [arXiv 2510.23497](https://arxiv.org/abs/2510.23497) (ICLR 2026) |
| Video-OPD | 📄 paper-only | 2026.02 | Industrial | [arXiv 2602.02994](https://arxiv.org/abs/2602.02994) |
| X-OPD (Speech LLM) | 📄 paper-only | 2026.03 | Tencent Hunyuan / ZJU | [arXiv 2603.24596](https://arxiv.org/abs/2603.24596) |

<details>
<summary>📋 Click to view technical details</summary>

| Method | Modality | Teacher | Loss | Data | Notes |
| :----: | :----: | :----: | :----: | :----: | :---- |
| π-Flow | Image generation (flow models) | Teacher velocity field | L2 imitation distillation | Student | Strict OPD for diffusion: student predicts policy at each timestep along its own trajectory. |
| Step-Audio-R1 | Audio reasoning | Self (modality-grounded) | Iterative self-distillation + SFT + PPO/RLVR | Student | Iterative on-policy cycles; only audio-relevant questions used in self-distill. |
| VOLD | LLM → VLM | Text-only LLM | GRPO + on-policy KL distillation | Student | Cold-start SFT alignment + unified RL+KD; ICLR 2026. The flagship VLM OPD recipe. |
| Video-OPD | MLLM | LLM teacher | Token-level KL on student rollouts | Student | Temporal video grounding via OPD. |
| X-OPD | Speech LLM | Text LLM | Cross-modal token-level KL | Student | Capability alignment in speech LLMs. |

</details>

---

## 🤖 Agent & Embodied OPD (by application)

Genuine OPD where the **student is an agent** rolling out actions; teacher (or self) supervises those trajectories. Pure-RL agent works (WebRL, WebAgent-R1, InfiGUI-G1, GUI-R1) and off-policy SFT-on-teacher-trajectories (Nardien, AgentRefine, Chain-of-Agents, MapCoder-Lite, SAD, Structured-Web) were moved to Caveats / Off-Policy KD.

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [Gen-Verse/OpenClaw-RL](https://github.com/Gen-Verse/OpenClaw-RL) (cross-list with OPD-RL) | <img src="https://img.shields.io/github/stars/Gen-Verse/OpenClaw-RL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.03 | Gen-Verse | [arXiv 2603.10165](https://arxiv.org/abs/2603.10165) |
| [modelscope/easydistill](https://github.com/modelscope/easydistill) (`/projects/SCoRe`) | <img src="https://img.shields.io/github/stars/modelscope/easydistill?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.09 | Alibaba ModelScope | [SCoRe arXiv 2509.14257](https://arxiv.org/abs/2509.14257) |
| RPD (Refined Policy Distillation, VLA) | 📄 paper-only ([project](https://refined-policy-distillation.github.io/)) | 2025.03 | TUM / Freiburg | [arXiv 2503.05833](https://arxiv.org/abs/2503.05833) (IROS 2026) |
| LLM4Teach (small-RL agent guided by LLM) | 📄 paper-only | 2023.11 (updated 2025) | Academic | [arXiv 2311.13373](https://arxiv.org/abs/2311.13373) |

<details>
<summary>📋 Click to view technical details</summary>

| Method | Domain | Teacher Role | Loss | Notes |
| :----: | :----: | :----: | :----: | :---- |
| OpenClaw-RL | Terminal / GUI / SWE / Tool-call | Judge model + token-logprob gap | GRPO + OPD | Hindsight-hint extraction; combines binary RL and per-token OPD. |
| SCoRe | 12 agent benchmarks | Larger teacher (72B) corrects earliest error in student rollout | SFT-on-corrections + short-horizon RL | 7B student matches 72B teacher. |
| RPD | VLA / robot manipulation | Teacher VLA actions | PPO + behavioural cloning on student rollouts | Cleanest VLA-OPD recipe. |
| LLM4Teach | Small RL agent | LLM teacher (action-level) | Distillation + RL annealed | Strict OPD for embodied; predates the wave. |

</details>

---

## ⚡ Speculative-Decoding Distillation

Distillation **of the draft model** so it better mimics the verifier/target. The on-policy element here is over the *drafter*'s own continuations as judged by the *target*. Listed separately because the goal is *inference speedup*, not student capability.

Removed on verification: **Ouroboros** (training-free phrase recycling), **Sequoia** (system-only DP tree), **TriForce** (system-only KV hierarchy), **SwiftKV** (KV-cache transform, not a draft KD), **SuffixDecoding** (model-free suffix tree). Each does *not* train the drafter and therefore doesn't fit "speculative-decoding distillation".

| Method / Repo | 🌟 Stars | Date | Org | Paper Link |
| :----: | :----: | :----: |  :----: | :----: |
| [SafeAILab/EAGLE](https://github.com/SafeAILab/EAGLE) (EAGLE-1/2/3) | <img src="https://img.shields.io/github/stars/SafeAILab/EAGLE?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.01 | PKU / Microsoft | [EAGLE-1](https://arxiv.org/abs/2401.15077) · [EAGLE-2](https://arxiv.org/abs/2406.16858) · [EAGLE-3](https://arxiv.org/abs/2503.01840) |
| [FasterDecoding/Medusa](https://github.com/FasterDecoding/Medusa) | <img src="https://img.shields.io/github/stars/FasterDecoding/Medusa?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.01 | Princeton / Together | [arXiv 2401.10774](https://arxiv.org/abs/2401.10774) (ICML 2024) |
| [zankner/Hydra](https://github.com/zankner/Hydra) | <img src="https://img.shields.io/github/stars/zankner/Hydra?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.02 | MIT | [arXiv 2402.05109](https://arxiv.org/abs/2402.05109) |
| [Equationliu/Kangaroo](https://github.com/Equationliu/Kangaroo) | <img src="https://img.shields.io/github/stars/Equationliu/Kangaroo?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.04 | Multi-org | [arXiv 2404.18911](https://arxiv.org/abs/2404.18911) |
| [apple/ml-recurrent-drafter](https://github.com/apple/ml-recurrent-drafter) (ReDrafter) | <img src="https://img.shields.io/github/stars/apple/ml-recurrent-drafter?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.03 | Apple | [arXiv 2403.09919](https://arxiv.org/abs/2403.09919) |
| [linfeng93/BiTA](https://github.com/linfeng93/BiTA) | <img src="https://img.shields.io/github/stars/linfeng93/BiTA?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.01 | Multi-org | [arXiv 2401.12522](https://arxiv.org/abs/2401.12522) |
| [HArmonizedSS/HASS](https://github.com/HArmonizedSS/HASS) | <img src="https://img.shields.io/github/stars/HArmonizedSS/HASS?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.08 | Academic | [arXiv 2408.15766](https://arxiv.org/abs/2408.15766) |
| [Kaffaljidhmah2/SpecDec_pp](https://github.com/Kaffaljidhmah2/SpecDec_pp) (SpecDec++) | <img src="https://img.shields.io/github/stars/Kaffaljidhmah2/SpecDec_pp?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.05 | Multi-org | [arXiv 2405.19715](https://arxiv.org/abs/2405.19715) |
| [LiuXiaoxuanPKU/OSD](https://github.com/LiuXiaoxuanPKU/OSD) (Online Speculative Decoding) | <img src="https://img.shields.io/github/stars/LiuXiaoxuanPKU/OSD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2023.10 | UCB / NVIDIA | [arXiv 2310.07177](https://arxiv.org/abs/2310.07177) |
| [facebookresearch/LayerSkip](https://github.com/facebookresearch/LayerSkip) | <img src="https://img.shields.io/github/stars/facebookresearch/LayerSkip?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.04 | Meta | [arXiv 2404.16710](https://arxiv.org/abs/2404.16710) |
| [raymin0223/fast_robust_early_exit](https://github.com/raymin0223/fast_robust_early_exit) (FREE) | <img src="https://img.shields.io/github/stars/raymin0223/fast_robust_early_exit?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2023.10 | KAIST | [arXiv 2310.05424](https://arxiv.org/abs/2310.05424) |
| [Bestpay-inc/Falcon](https://github.com/Bestpay-inc/Falcon) | <img src="https://img.shields.io/github/stars/Bestpay-inc/Falcon?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.12 | Bestpay | [arXiv 2412.12639](https://arxiv.org/abs/2412.12639) |
| [yuezhouhu/adaspec](https://github.com/yuezhouhu/adaspec) (AdaSPEC) | <img src="https://img.shields.io/github/stars/yuezhouhu/adaspec?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.10 | Academic | [arXiv 2510.19779](https://arxiv.org/abs/2510.19779) |
| [shrango/poss](https://github.com/shrango/poss) (POSS) | <img src="https://img.shields.io/github/stars/shrango/poss?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.06 | WashU / CMU | [arXiv 2506.03566](https://arxiv.org/abs/2506.03566) |
| [sgl-project/SpecForge](https://github.com/sgl-project/SpecForge) (open EAGLE-3 training framework) | <img src="https://img.shields.io/github/stars/sgl-project/SpecForge?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.03 | SGLang | [LMSYS blog](https://www.lmsys.org/blog/2025-07-25-spec-forge/) |
| DistillSpec | 📄 paper-only | 2023.10 | Google DeepMind | [arXiv 2310.08461](https://arxiv.org/abs/2310.08461) (ICLR 2024) |
| SpecKD (verification-gated KD) | 📄 paper-only | 2025.10 | Academic | [arXiv 2510.24021](https://arxiv.org/abs/2510.24021) |
| ReSpec (RL drafter evolution) | 📄 paper-only | 2025.10 | Academic | [arXiv 2510.26475](https://arxiv.org/abs/2510.26475) |
| DVI (Draft-Verify-Improve, online RL) | 📄 paper-only | 2025.10 | Academic | [arXiv 2510.05421](https://arxiv.org/abs/2510.05421) |
| CORAL (Cross-Step Representation Alignment) | 📄 paper-only | 2025.02 | Academic | [arXiv 2502.16880](https://arxiv.org/abs/2502.16880) (ACL 2025) |
| MASSV (multimodal SD draft) | 📄 paper-only | 2025.05 | Cerebras | [arXiv 2505.10526](https://arxiv.org/abs/2505.10526) |

> 📝 **Strictness notes** — by strict OPD (C1+C2), this section splits into two camps:
>
> **✓ Strict on-policy SD training** (drafter samples its own continuations during training):
> EAGLE-3 (TTT mode), HASS (multi-step), Falcon (CSGD partial on-policy), OSD (canonical online), DistillSpec (explicit on-policy), SpecForge (TTT supported), SpecKD (verification-gated), ReSpec (on-policy online RL), DVI (on-policy KL→RL schedule), CORAL (on-policy multi-step), MASSV (on-policy drafter samples).
>
> **⚠️ Off-policy drafter training** — kept under SpecDec because they fit the section theme of "draft-model distillation methods", but they fail strict OPD's C1 (training data is teacher-forced / a fixed corpus, not drafter rollouts):
> - **EAGLE-1, EAGLE-2** — regression on fixed-data features (off-policy). EAGLE-3 changes this.
> - **Medusa** — Medusa-1 is teacher-forced on a fixed dataset (off-policy); Medusa-2 self-distill is borderline on-policy.
> - **Hydra** — per-head CE on fixed teacher probs.
> - **Kangaroo** — adapter trained offline with CE on ShareGPT.
> - **ReDrafter** — teacher forcing; no draft sampling during training.
> - **BiTA** — soft prompts trained on target-greedy generated SFT data (off-policy).
> - **SpecDec++** — BCE on acceptance-prediction head, off-policy.
> - **LayerSkip** — layer-dropout + early-exit loss; no draft sampling during training.
> - **FREE** — shallow-deep heads trained jointly, off-policy.
> - **AdaSPEC** — two-stage *offline* selective-token distillation.
> - **POSS** — position-specialised layers trained offline on ShareGPT.

<details>
<summary>📋 Click to view technical details</summary>

| Method | Drafter type | On-/Off-policy | Loss | Notes |
| :----: | :----: | :----: | :----: | :---- |
| EAGLE-1/2/3 | Self-speculative (uses target features) | Off-policy (EAGLE-1/2); **on-policy multi-step (EAGLE-3 TTT)** | Smooth-L1 (feature) + CE (token) | EAGLE-3 introduces "Training-Time Test" simulating draft rollouts during training. |
| Medusa | Self-speculative (multiple decoding heads) | Off-policy | CE per head; Medusa-2 adds joint loss | "Self-distillation recipe" for adding Medusa to any FT'd LLM. |
| Hydra | Sequentially-dependent draft heads | Off-policy | CE per head | +0.46 token avg. acceptance over Medusa. |
| Kangaroo | Self-speculative (frozen sub-network + adapter) | Off-policy | CE | Double early-exit. |
| ReDrafter (Apple) | RNN drafter conditioned on target hidden state | Off-policy | KD CE | Recurrent. |
| BiTA | Self-speculative (frozen target + prompt mask tuning) | Off-policy | CE | Bi-directional tuning. |
| HASS | Self-speculative | **Partial on-policy** (multi-step draft trajectory in training) | Multi-step KD CE + feature alignment | Harmonized objective + harmonized context alignment. |
| SpecDec++ | Draft-model | Off-policy | BCE on acceptance prediction | Adaptive K. |
| Online Speculative Decoding (OSD) | Draft-model | **On-policy / online** | Online KD on rejected tokens | The canonical online/on-policy SD paper. |
| Ouroboros | Draft-model | Off-policy (recycles verified phrases) | CE | Phrase candidate pool. |
| Sequoia / TriForce | Draft-model (system) | n/a (system) | n/a | DP-optimal tree (Sequoia); hierarchical (TriForce). |
| LayerSkip | Self-speculative | Off-policy | Per-layer CE with early-exit | Layer-dropout + early-exit loss. |
| FREE | Self-speculative (early-exit) | Off-policy | Per-layer CE + adaptive threshold | Synchronized parallel decoding. |
| Falcon | Draft-model (semi-AR) | **Partial on-policy** (glancing uses draft samples) | Glancing CE + KD | Coupled Sequential Glancing Distillation. |
| AdaSPEC | Draft-model | Off-policy (selective) | Filtered KL | Reference-model based filtering of "easy" tokens; +15% acceptance over DistillSpec. |
| POSS | Draft-model | Off-policy | Position-specialized layers | +5.7% over HASS. |
| SwiftKV / SuffixDecoding | Self-speculative / Train-free | n/a (system) | n/a | Snowflake's inference engine. |
| SpecForge | Self-speculative (EAGLE-3 framework) | **On-policy TTT supported** | EAGLE-3 losses | Open-source EAGLE-3 training framework. |
| DistillSpec | Draft-model | **On-policy** (draft samples) | Choice of FKL/RKL/JSD/TVD | The seminal "OPD for SD" paper. |
| SpecKD | Distillation framework | **On-policy with verification gating** | Gated KL (accepted tokens only) | Inverts SD: uses accept/reject as KD-loss gate. |
| ReSpec | Draft-model | **On-policy online (RL rollouts)** | KD weighted by rollout reward | Drafter evolved during RL training. |
| DVI | Self-speculative | **On-policy online (RL on verifier signal)** | KL → reward-masked CE + PG | Continual online training. |
| CORAL | Self-speculative | **On-policy multi-step** | Cross-step alignment + CE | Fixes draft training/inference mismatch. |

</details>

---

## 🛠️ Frameworks & Toolkits

Open-source frameworks / libraries that support OPD (with student-generated rollouts during distillation training).

| Github Repo | 🌟 Stars | Date | Org | OPD Code Path |
| :----: | :----: | :----: |  :----: | :---- |
| [volcengine/verl](https://github.com/volcengine/verl) | <img src="https://img.shields.io/github/stars/volcengine/verl?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.10 | ByteDance Seed | `recipe/on_policy_distill/`; [Async OPD doc](https://verl.readthedocs.io/en/latest/advance/async-on-policy-distill.html) |
| [huggingface/trl](https://github.com/huggingface/trl) | <img src="https://img.shields.io/github/stars/huggingface/trl?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2019.11 | Hugging Face | `trl/experimental/{gkd,gold,minillm,sdft,self_distillation,sdpo,nash_md,xpo,online_dpo}/` — **the most diverse OPD trainer collection** |
| [hiyouga/LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) | <img src="https://img.shields.io/github/stars/hiyouga/LLaMA-Factory?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2023.05 | hiyouga | OPD only via TRL integration; not native |
| [modelscope/ms-swift](https://github.com/modelscope/ms-swift) | <img src="https://img.shields.io/github/stars/modelscope/ms-swift?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024 | Alibaba ModelScope | `examples/train/rlhf/gkd/`, multimodal/megatron variants — wraps TRL `GKDTrainer` |
| [rllm-org/rllm](https://github.com/rllm-org/rllm) | <img src="https://img.shields.io/github/stars/rllm-org/rllm?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | UC Berkeley Sky | `examples/math_distill/` (incl. `opsd/` self-distill); `rllm/trainer/distill/` |
| [alibaba/ROLL](https://github.com/alibaba/ROLL) | <img src="https://img.shields.io/github/stars/alibaba/ROLL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.06 | Alibaba | `roll/pipeline/distill/` with VLM support and various-divergence library |
| [inclusionAI/AReaL](https://github.com/inclusionAI/AReaL) | <img src="https://img.shields.io/github/stars/inclusionAI/AReaL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.06 | AntGroup / Tsinghua | `examples/distillation/gsm8k_grpo_distill.yaml` |
| [NVIDIA-NeMo/RL](https://github.com/NVIDIA-NeMo/RL) | <img src="https://img.shields.io/github/stars/NVIDIA-NeMo/RL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | NVIDIA | `nemo_rl/algorithms/distillation.py` — native OPD with student rollouts |
| [NVIDIA/TensorRT-Model-Optimizer](https://github.com/NVIDIA/TensorRT-Model-Optimizer) | <img src="https://img.shields.io/github/stars/NVIDIA/TensorRT-Model-Optimizer?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024 | NVIDIA | `mtd.LogitsDistillationLoss()`; mostly off-policy KD — listed for completeness |
| [NovaSky-AI/SkyRL](https://github.com/NovaSky-AI/SkyRL) | <img src="https://img.shields.io/github/stars/NovaSky-AI/SkyRL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | UC Berkeley NovaSky | `skyrl-train/examples/on_policy_distillation/`; [blog](https://novasky-ai.notion.site/on-policy-distillation) |
| [THUDM/slime](https://github.com/THUDM/slime) | <img src="https://img.shields.io/github/stars/THUDM/slime?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.06 | Tsinghua THUDM | `examples/on_policy_distillation/` — RL framework behind GLM-4.5/4.6/4.7 |
| [google/tunix](https://github.com/google/tunix) | <img src="https://img.shields.io/github/stars/google/tunix?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | Google | `tunix/distillation/` — JAX-native; **only `logit_distillation.ipynb` (offline) as of v0.1.6**, no on-policy strategy file |
| [arcee-ai/DistillKit](https://github.com/arcee-ai/DistillKit) | <img src="https://img.shields.io/github/stars/arcee-ai/DistillKit?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.08 | Arcee AI | Online (live teacher) + offline KD; "online" mode is teacher-runs-during-student-training but data is typically a fixed corpus |
| [modelscope/easydistill](https://github.com/modelscope/easydistill) | <img src="https://img.shields.io/github/stars/modelscope/easydistill?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025 | Alibaba ModelScope | Black-box + white-box KD; AgentKD subdir (incl. SCoRe project) |

<details>
<summary>📋 Click to view technical details</summary>

| Framework | KL Direction(s) | OPD Primary? | Backbone | Multi-GPU | Notes |
| :----: | :----: | :----: | :----: | :----: | :---- |
| verl | Forward KL with sparse top-k teacher logits | One of many | PyTorch | Yes (FSDP, Megatron, Ray) | `recipe/on_policy_distill/` — the most production-ready OPD recipe; integrates with vLLM. |
| TRL | FKL, RKL, GJSD (β); GOLD trainer; SDFT trainer; MiniLLM trainer | One of many; **most diverse OPD collection** | PyTorch | Yes (Accelerate, DeepSpeed) | `trl/experimental/` contains **gkd, gold, minillm, sdft, self_distillation, sdpo, nash_md, xpo, online_dpo, papo, prm**. The single broadest OPD trainer set. |
| LLaMA-Factory | Via TRL integration | No native | PyTorch | Yes | Most-starred fine-tuning framework. |
| ms-swift | Same as TRL GKD | One of many | PyTorch | Yes (DeepSpeed, Megatron) | Wraps TRL GKDTrainer; multimodal variants. |
| rllm (Berkeley) | Reverse KL (advantage = log P_teacher − log P_student) | **Primary in math_distill example** | PyTorch | Single (tinker) + Multi-GPU (verl) | Self-distill subdir `opsd/`. |
| ROLL | Multiple divergences (`various_divergence.py`) | First-class `DistillPipeline` | PyTorch | Yes (Megatron) | VLM support. |
| AReaL | KL-controlled (off-policy default; integrates into GRPO) | One of many | PyTorch | Yes (async distributed) | `distill_loss_weight`. |
| NeMo-RL | FKL / RKL / mixed (configurable `kl_type`) | OPD documented | PyTorch | Yes (Ray + Megatron + vLLM) | Replaces archived NeMo-Aligner. |
| Modelopt | Standard KL on logits | One of many; mostly off-policy | PyTorch | Yes | Listed for completeness; not strict OPD. |
| SkyRL | Reverse KL + importance sampling | OPD added Nov 2025 (PR #585) | PyTorch | Yes (Ray + vLLM/SGLang) | Notion blog "On-Policy Distillation in SkyRL". |
| slime | Reverse KL token-level | OPD as additive penalty on any advantage estimator | PyTorch + Megatron | Yes (SGLang teacher mode) | Behind GLM-4.5/4.6/4.7. |
| Tunix | RKL (logit strategy) | **Distillation is offline only** in v0.1.6 | JAX/Flax NNX | Yes (TPU/GPU multi-host) | No on-policy strategy file as of Apr 2026. |
| DistillKit | Standard KL | KD primary | PyTorch | Yes | Online + offline modes; data typically a fixed corpus. |
| easydistill | Black-box + white-box | KD primary | PyTorch | Yes | Has agent-distill recipes. |

**Removed on verification** (no native OPD support in their READMEs as of Apr 2026): axolotl, OpenRLHF, allenai/open-instruct, prime-rl, TextBrewer (pre-LLM era), open-r1 (off-policy SFT + GRPO). gpt-oss is a model release not a framework — moved to [Industrial Reports](#-industrial--production-model-reports).

</details>

> 📝 **Strictness notes** — frameworks judged by whether they ship a recipe that satisfies C1+C2:
> - **LLaMA-Factory** — ⚠️ OPD only available *via* TRL integration; no native OPD trainer. Listed for users who already use LLaMA-Factory and want to know it can host OPD.
> - **Modelopt** — ⚠️ `mtd.LogitsDistillationLoss()` operates on a fixed dataset; no student-rollout loop. Listed for completeness; **not strict OPD** (kept because it's the canonical NVIDIA distillation entry-point).
> - **Tunix v0.1.6** — ⚠️ `tunix/distillation/` ships only an offline `logit_distillation.ipynb`; no on-policy strategy file as of v0.1.6 (Mar 2026). Roadmap-only for OPD.
> - **DistillKit** — ⚠️ "Online" mode keeps the teacher live during training, but data is typically a fixed corpus rather than fresh student rollouts. Borderline.
> - **easydistill** — ⚠️ Black-box + white-box KD; AgentKD subdir hosts the SCoRe project, but the typical recipe is data-synthesis + SFT + RL, not strict student-rollout OPD.

---

## 🏭 Industrial / Production Model Reports

Flagship model technical reports that publicly describe **on-policy** distillation in their post-training pipeline. We removed reports whose tech papers don't actually describe student-rollout distillation (Qwen2.5, Qwen2.5-Math, MiMo predecessor, DeepSeek-V3 / V3.2-Exp / R1, Phi-4, Hunyuan-Large / A13B, Kimi-K2 / K2.5, Yi-Lightning, DistilQwen) — those were moved to [Off-Policy KD References](#-off-policy-kd-references-for-context). Keep with caveat: Gemma 3 (mostly pretraining off-policy), Llama 4 (codistillation in pretraining), Phi-4-Mini-Reasoning (Rollout-DPO is borderline).

| Model | Repo | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: | :----: |  :----: | :----: |
| **Qwen3** (canonical OPD recipe) | [QwenLM/Qwen3](https://github.com/QwenLM/Qwen3) | <img src="https://img.shields.io/github/stars/QwenLM/Qwen3?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | Alibaba Qwen | [arXiv 2505.09388](https://arxiv.org/abs/2505.09388) |
| Qwen3-Coder | [QwenLM/Qwen3-Coder](https://github.com/QwenLM/Qwen3-Coder) | <img src="https://img.shields.io/github/stars/QwenLM/Qwen3-Coder?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.03 | Alibaba Qwen | Tech report |
| **Gemma 2** (explicit OPD) | [google-deepmind/gemma](https://github.com/google-deepmind/gemma) | <img src="https://img.shields.io/github/stars/google-deepmind/gemma?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.07 | Google DeepMind | [arXiv 2408.00118](https://arxiv.org/abs/2408.00118) |
| Gemma 3 (mostly pretraining off-policy; some on-policy in post) | [google-deepmind/gemma](https://github.com/google-deepmind/gemma) | (same repo) | 2025.03 | Google DeepMind | [arXiv 2503.19786](https://arxiv.org/abs/2503.19786) |
| **GLM-4.5 / 4.6** | [zai-org/GLM-4.5](https://github.com/zai-org/GLM-4.5) | <img src="https://img.shields.io/github/stars/zai-org/GLM-4.5?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.08 | Zhipu / Z.ai | [arXiv 2508.06471](https://arxiv.org/abs/2508.06471) |
| **GLM-5** (cross-stage OPD) | (HF release) | — | 2026.02 | Zhipu / Z.ai | [arXiv 2602.15763](https://arxiv.org/abs/2602.15763) |
| **MiMo-V2-Flash** (MOPD) | [XiaomiMiMo/MiMo-V2-Flash](https://github.com/XiaomiMiMo/MiMo-V2-Flash) | <img src="https://img.shields.io/github/stars/XiaomiMiMo/MiMo-V2-Flash?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | Xiaomi | [arXiv 2601.02780](https://arxiv.org/abs/2601.02780) |
| **Llama 4** (Maverick codistilled from Behemoth in pretraining; borderline OPD) | [meta-llama/llama-models](https://github.com/meta-llama/llama-models) | <img src="https://img.shields.io/github/stars/meta-llama/llama-models?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | Meta AI | [Llama 4 blog](https://ai.meta.com/blog/llama-4-multimodal-intelligence/) |
| Phi-4-Mini-Reasoning (Rollout-DPO; borderline) | (HF only) | — | 2025.04 | Microsoft Research | [arXiv 2504.21233](https://arxiv.org/abs/2504.21233) |
| **Nemotron Cascade 2** (multi-domain OPD; "we sample y∼π_inf(·|x)") | (HF release) | — | 2026.03 | NVIDIA | [arXiv 2603.19220](https://arxiv.org/abs/2603.19220) |
| **gpt-oss-120b/20b** ("trained with large-scale distillation and RL") | [openai/gpt-oss](https://github.com/openai/gpt-oss) | <img src="https://img.shields.io/github/stars/openai/gpt-oss?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.08 | OpenAI | [Model card 2508.10925](https://arxiv.org/abs/2508.10925) |

<details>
<summary>📋 Click to view technical details</summary>

| Model | Stage(s) using OPD | Mechanism | Notes |
| :----: | :----: | :----: | :---- |
| **Qwen3** | Strong-to-Weak Distillation | Two-phase: (1) off-policy SFT cold-start with `/think` and `/no_think` teacher samples; (2) **on-policy phase** — student generates, teacher provides logit-KL targets | Reports ~10× cheaper than RL for equal performance. The canonical industrial OPD recipe. Inspired the Thinking Machines blog. |
| Qwen3-Coder-Next | Distillation of multi-experts into 80A3 student | Combined SFT + on-policy logit alignment | Production scaling of Qwen3 recipe. |
| **Gemma 2** | Post-training | "We also use **on-policy distillation**, where the student generates completions from the SFT prompts" — KL on student samples | Among the first production models to *name* OPD. |
| Gemma 3 | Pretraining KD + post-training | Sample 256 logits from larger teacher; some on-policy in post | Mostly off-policy KD in pretraining; partial OPD in post. |
| **GLM-5** | Throughout post-training | "On-Policy Cross-Stage Distillation" — a final anti-forgetting refinement applied between stages | Generalises Qwen3 recipe to "OPD as a stage glue". |
| GLM-4.5 / 4.6 | Multi-stage post-training | Expert iteration; SFT distillation merges experts into hybrid generalist | Predecessors of GLM-5. |
| **MiMo-V2-Flash** | Post-training | **Multi-Teacher On-Policy Distillation (MOPD)** — "the student model samples from its own evolving distribution and receives token-level supervision from domain-specific teachers" | Multi-teacher OPD; per-token MOPD advantage formula. |
| **Nemotron Cascade 2** | Between Cascade RL stages | Multi-Domain On-Policy Distillation (MOPD) — "we sample y∼π_inf(·|x)"; teacher provides token-level distillation advantage | Sample-efficient: matches RL in 30–160 steps vs 1000+. |
| **Llama 4 Maverick** | Pretraining | **Codistillation** of Maverick from Behemoth during pretraining with novel dynamic soft/hard target loss | Pretraining-time co-distillation; borderline OPD. |
| Phi-4-Mini-Reasoning | Post-training | 4-step recipe: long-CoT mid-training → SFT → Rollout DPO → RLVR | Rollout DPO is on-policy student-rollout-based; DPO loss itself is technically off-policy on those collected rollouts. Borderline. |
| openai/gpt-oss-120b/20b | Post-training | "Trained with large-scale distillation and RL" from internal o3 / frontier models | Open-weight; explicit distillation claim. |

</details>

> 📝 **Strictness notes**
> - **Gemma 3** — ⚠️ Primary recipe is *pretraining* off-policy KD ("sample 256 logits per token from teacher"). Some on-policy distillation in post-training but not the focus of the report. Kept on the strength of the post-training leg.
> - **GLM-4.5 / 4.6** — ⚠️ Tech report describes "expert iteration + RL" without explicit OPD wording. Kept as predecessor of GLM-5 which does have explicit cross-stage OPD.
> - **Llama 4** — ⚠️ "Codistillation" of Maverick from Behemoth happens *during pretraining* with student and teacher trained jointly under a shared loss — teacher is **not frozen** when student is rolling out, so this fails the classical OPD frozen-teacher property. Kept as the only public flagship example of pretraining-time co-distillation.
> - **Phi-4-Mini-Reasoning** — ⚠️ Rollout-DPO step *collects* student rollouts, but the DPO loss applied is technically off-policy on those collected samples (preference pairs over collected rollouts, not per-token KL on currently-sampled tokens). Borderline.
> - **gpt-oss-120b/20b** — ⚠️ Model card says "trained with large-scale distillation and RL" without specifying whether the distillation phase was on-policy (student rollouts) or off-policy (teacher data). Listed on the strength of the explicit "distillation" wording but **mechanism unverified**.

---

## 📕 Off-Policy KD References (for context)

Widely-cited **off-policy** KD baselines and "almost-OPD" works that the OPD wave compares against. **Not OPD** by the strict definition (no student-rollouts-with-teacher-supervision). Organised by sub-type.

### Off-policy logit / loss-function KD (no student rollouts)

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [songmzhang/DSKD](https://github.com/songmzhang/DSKD) | <img src="https://img.shields.io/github/stars/songmzhang/DSKD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.06 | BJTU | [arXiv 2406.17328](https://arxiv.org/abs/2406.17328) |
| [SakanaAI/TAID](https://github.com/SakanaAI/TAID) | <img src="https://img.shields.io/github/stars/SakanaAI/TAID?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | Sakana AI | [arXiv 2501.16937](https://arxiv.org/abs/2501.16937) (ICLR 2025) |
| [D2I-ai/dasd-thinking](https://github.com/D2I-ai/dasd-thinking) (DASD) | <img src="https://img.shields.io/github/stars/D2I-ai/dasd-thinking?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2026.01 | Alibaba | [arXiv 2601.09088](https://arxiv.org/abs/2601.09088) |
| [wutaiqiang/LLM_KD_AKL](https://github.com/wutaiqiang/LLM_KD_AKL) (AKL) | <img src="https://img.shields.io/github/stars/wutaiqiang/LLM_KD_AKL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.04 | HKU | [arXiv 2404.02657](https://arxiv.org/abs/2404.02657) (COLING 2025) |
| [jungseongryong/ToDi](https://github.com/jungseongryong/ToDi) | <img src="https://img.shields.io/github/stars/jungseongryong/ToDi?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | Yonsei | [arXiv 2505.16297](https://arxiv.org/abs/2505.16297) (EMNLP 2025) |
| [SassyRong/AdaKD](https://github.com/SassyRong/AdaKD) | <img src="https://img.shields.io/github/stars/SassyRong/AdaKD?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.10 | Industrial | [arXiv 2510.11615](https://arxiv.org/abs/2510.11615) (AAAI 2026) |
| Cross-Tokenizer KD (ULD) | [Diabolocom-Research/ULD-Loss](https://github.com/Diabolocom-Research/ULD-Loss) | 2024.02 | Diabolocom / CentraleSupélec | [arXiv 2402.12030](https://arxiv.org/abs/2402.12030) (TMLR 2025) |
| AMiD (α-mixture) | 📄 paper-only | 2025.10 | Academic | [arXiv 2510.15982](https://arxiv.org/abs/2510.15982) |
| DRKL | 📄 paper-only | 2026.04 | Wright State | [arXiv 2604.00223](https://arxiv.org/abs/2604.00223) |
| CSD (Concrete Score) | 📄 paper-only | 2025.09 | KAIST | [arXiv 2509.25837](https://arxiv.org/abs/2509.25837) (ICLR 2026) |
| Delta KD | 📄 paper-only | 2025.09 | LinkedIn | [arXiv 2509.14526](https://arxiv.org/abs/2509.14526) |
| HPD (mostly off-policy) | 📄 paper-only | 2026.04 | Academic | [arXiv 2604.20244](https://arxiv.org/abs/2604.20244) |
| SODA (semi-on-policy black-box) | 📄 paper-only | 2026.04 | Microsoft Research | [arXiv 2604.03873](https://arxiv.org/abs/2604.03873) |

### Pretraining-time / pruning-time KD (on a static corpus)

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [NVlabs/Minitron](https://github.com/NVlabs/Minitron) | <img src="https://img.shields.io/github/stars/NVlabs/Minitron?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.07 | NVIDIA | [arXiv 2407.14679](https://arxiv.org/abs/2407.14679) |
| [thu-coai/MiniPLM](https://github.com/thu-coai/MiniPLM) | <img src="https://img.shields.io/github/stars/thu-coai/MiniPLM?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.10 | Tsinghua | [arXiv 2410.17215](https://arxiv.org/abs/2410.17215) (ICLR 2025) |
| [sail-sg/sdft](https://github.com/sail-sg/sdft) (SDFT-2024 distribution-bridge) | <img src="https://img.shields.io/github/stars/sail-sg/sdft?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.02 | Sea AI Lab | [arXiv 2402.13669](https://arxiv.org/abs/2402.13669) (ACL 2024) |
| Distilled Pretraining (token routing) | 📄 paper-only | 2025.09 | Meta FAIR | [arXiv 2509.01649](https://arxiv.org/abs/2509.01649) |
| Pre-training Distillation Design Space | 📄 paper-only | 2024.10 | Tsinghua | [arXiv 2410.16215](https://arxiv.org/abs/2410.16215) (ACL 2025) |
| Self-Data Distillation for Pruned LLMs | 📄 paper-only | 2024.10 | Cerebras | [arXiv 2410.09982](https://arxiv.org/abs/2410.09982) |

### Off-policy R1-distill / reasoning SFT

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [deepseek-ai/DeepSeek-R1](https://github.com/deepseek-ai/DeepSeek-R1) (R1-Distill family) | <img src="https://img.shields.io/github/stars/deepseek-ai/DeepSeek-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | DeepSeek | [arXiv 2501.12948](https://arxiv.org/abs/2501.12948) |
| [deepseek-ai/DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) (R1→V3 specialist SFT) | <img src="https://img.shields.io/github/stars/deepseek-ai/DeepSeek-V3?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.12 | DeepSeek | [arXiv 2412.19437](https://arxiv.org/abs/2412.19437) |
| [deepseek-ai/DeepSeek-V3.2-Exp](https://github.com/deepseek-ai/DeepSeek-V3.2-Exp) | <img src="https://img.shields.io/github/stars/deepseek-ai/DeepSeek-V3.2-Exp?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.11 | DeepSeek | Tech PDF |
| [simplescaling/s1](https://github.com/simplescaling/s1) | <img src="https://img.shields.io/github/stars/simplescaling/s1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | Stanford / UW | [arXiv 2501.19393](https://arxiv.org/abs/2501.19393) |
| [GAIR-NLP/LIMO](https://github.com/GAIR-NLP/LIMO) | <img src="https://img.shields.io/github/stars/GAIR-NLP/LIMO?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.02 | SJTU GAIR | [arXiv 2502.03387](https://arxiv.org/abs/2502.03387) |
| [open-thoughts/open-thoughts](https://github.com/open-thoughts/open-thoughts) | <img src="https://img.shields.io/github/stars/open-thoughts/open-thoughts?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.06 | OpenThoughts | [arXiv 2506.04178](https://arxiv.org/abs/2506.04178) |
| [NovaSky-AI/SkyThought](https://github.com/NovaSky-AI/SkyThought) (Sky-T1, Bespoke-Stratos) | <img src="https://img.shields.io/github/stars/NovaSky-AI/SkyThought?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | UCB NovaSky | [Sky-T1 blog](https://novasky-ai.github.io/posts/sky-t1/) |
| [bespokelabsai/curator](https://github.com/bespokelabsai/curator) | <img src="https://img.shields.io/github/stars/bespokelabsai/curator?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | Bespoke Labs | [Bespoke-Stratos blog](https://www.bespokelabsai.com/blog/bespoke-stratos-the-unreasonable-effectiveness-of-reasoning-distillation) |
| [Qihoo360/Light-R1](https://github.com/Qihoo360/Light-R1) | <img src="https://img.shields.io/github/stars/Qihoo360/Light-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | Qihoo 360 | [arXiv 2503.10460](https://arxiv.org/abs/2503.10460) |
| [a-m-team/a-m-models](https://github.com/a-m-team/a-m-models) | <img src="https://img.shields.io/github/stars/a-m-team/a-m-models?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | a-m-team | [arXiv 2503.19633](https://arxiv.org/abs/2503.19633) · [arXiv 2505.14464](https://arxiv.org/abs/2505.14464) |
| [huggingface/open-r1](https://github.com/huggingface/open-r1) | <img src="https://img.shields.io/github/stars/huggingface/open-r1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | Hugging Face | (open R1 reproduction) |
| Beyond Scaling Law (DED, NTele-32B-V1) | 📄 paper-only | 2025.08 | China Telecom | [arXiv 2508.09883](https://arxiv.org/abs/2508.09883) |
| Merge-of-Thought Distillation | 📄 paper-only | 2025.09 | Academic | [arXiv 2509.08814](https://arxiv.org/abs/2509.08814) |
| Reasoning Scaffolding | 📄 paper-only | 2025.09 | Multi-org | [arXiv 2509.23619](https://arxiv.org/abs/2509.23619) |
| DLCoT (Deconstructing Long CoT Distillation) | 📄 paper-only | 2025.03 | Alibaba | [arXiv 2503.16385](https://arxiv.org/abs/2503.16385) |
| **DistilQwen** series | [modelscope/easydistill](https://github.com/modelscope/easydistill) | 2025.04 | Alibaba PAI | [DistilQwen2.5 2504.15027](https://arxiv.org/abs/2504.15027) · [Thinking with DistilQwen 2511.01354](https://arxiv.org/abs/2511.01354) |

### Off-policy black-box / preference distillation

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [YJiangcm/Lion](https://github.com/YJiangcm/Lion) | <img src="https://img.shields.io/github/stars/YJiangcm/Lion?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2023.05 | HKUST | [arXiv 2305.12870](https://arxiv.org/abs/2305.12870) (EMNLP 2023) |
| [YangLing0818/SuperCorrect-llm](https://github.com/YangLing0818/SuperCorrect-llm) | <img src="https://img.shields.io/github/stars/YangLing0818/SuperCorrect-llm?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.10 | PKU / Skywork | [arXiv 2410.09008](https://arxiv.org/abs/2410.09008) (ICLR 2025) |
| DAIL | 📄 paper-only | 2026.02 | Georgia Tech | [arXiv 2602.02405](https://arxiv.org/abs/2602.02405) |
| REDI (negative-signal RL distillation, offline) | 📄 paper-only | 2025.05 | Academic | [arXiv 2505.24850](https://arxiv.org/abs/2505.24850) |

### Off-policy speech / audio / multimodal KD

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [huggingface/distil-whisper](https://github.com/huggingface/distil-whisper) | <img src="https://img.shields.io/github/stars/huggingface/distil-whisper?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2023.11 | Hugging Face | [arXiv 2311.00430](https://arxiv.org/abs/2311.00430) |
| [kyutai-labs/moshi](https://github.com/kyutai-labs/moshi) (Mimi codec) | <img src="https://img.shields.io/github/stars/kyutai-labs/moshi?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.09 | Kyutai | [arXiv 2410.00037](https://arxiv.org/abs/2410.00037) |
| [FunAudioLLM/CosyVoice](https://github.com/FunAudioLLM/CosyVoice) | <img src="https://img.shields.io/github/stars/FunAudioLLM/CosyVoice?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024 | Alibaba | [CosyVoice 2 arXiv 2412.10117](https://arxiv.org/abs/2412.10117) |
| SightSound-R1 (vision→audio SFT+GRPO) | 📄 paper-only | 2025.09 | Multi-org | [arXiv 2509.15661](https://arxiv.org/abs/2509.15661) |
| Cross-Modal KD for Speech LLMs | 📄 paper-only | 2025.09 | Tencent | [arXiv 2509.14930](https://arxiv.org/abs/2509.14930) |

### Off-policy agent SFT-distillation

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [Nardien/agent-distillation](https://github.com/Nardien/agent-distillation) | <img src="https://img.shields.io/github/stars/Nardien/agent-distillation?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | Multi-org | [arXiv 2505.17612](https://arxiv.org/abs/2505.17612) (NeurIPS 2025) |
| [Fu-Dayuan/AgentRefine](https://github.com/Fu-Dayuan/AgentRefine) | <img src="https://img.shields.io/github/stars/Fu-Dayuan/AgentRefine?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.01 | Multi-org | [arXiv 2501.01702](https://arxiv.org/abs/2501.01702) (ICLR 2025) |
| [OPPO-PersonalAI/Agent_Foundation_Models](https://github.com/OPPO-PersonalAI/Agent_Foundation_Models) | <img src="https://img.shields.io/github/stars/OPPO-PersonalAI/Agent_Foundation_Models?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.08 | OPPO | [arXiv 2508.13167](https://arxiv.org/abs/2508.13167) |
| Structured Agent Distillation (SAD) | 📄 paper-only | 2025.05 | Academic | [arXiv 2505.13820](https://arxiv.org/abs/2505.13820) |
| MapCoder-Lite | 📄 paper-only | 2025.09 | Academic | [arXiv 2509.17489](https://arxiv.org/abs/2509.17489) |
| Structured Distillation of Web Agent Capabilities | 📄 paper-only | 2026.04 | Multi-org | [arXiv 2604.07776](https://arxiv.org/abs/2604.07776) |
| [InfiXAI/InfiGUI-R1](https://github.com/InfiXAI/InfiGUI-R1) (off-policy distill + RL) | <img src="https://img.shields.io/github/stars/InfiXAI/InfiGUI-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | InfiX AI | [arXiv 2504.14239](https://arxiv.org/abs/2504.14239) |

### OPSD-style works that turned out to be off-policy on verification

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| Privileged Information Distillation (π-Distill) | 📄 paper-only | 2026.02 | ServiceNow / Mila | [arXiv 2602.04942](https://arxiv.org/abs/2602.04942) |
| TMS (Trajectory-Mixed Supervision) | 📄 paper-only | 2026.02 | UNC | [arXiv 2602.03073](https://arxiv.org/abs/2602.03073) |
| Apple Memory-Retaining FT | 📄 page | 2025 | Apple MLR | [Apple ML page](https://machinelearning.apple.com/research/memory-retaining) |

### Production reports without strict OPD evidence

| Model | Repo | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: | :----: |  :----: | :----: |
| Qwen2.5 | [QwenLM/Qwen2.5](https://github.com/QwenLM/Qwen2.5) | <img src="https://img.shields.io/github/stars/QwenLM/Qwen2.5?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.12 | Alibaba Qwen | [arXiv 2412.15115](https://arxiv.org/abs/2412.15115) |
| Qwen2.5-Math | [QwenLM/Qwen2.5-Math](https://github.com/QwenLM/Qwen2.5-Math) | <img src="https://img.shields.io/github/stars/QwenLM/Qwen2.5-Math?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.09 | Alibaba Qwen | [arXiv 2409.12122](https://arxiv.org/abs/2409.12122) |
| MiMo (predecessor of MiMo-V2-Flash) | [XiaomiMiMo/MiMo](https://github.com/XiaomiMiMo/MiMo) | <img src="https://img.shields.io/github/stars/XiaomiMiMo/MiMo?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | Xiaomi | [arXiv 2505.07608](https://arxiv.org/abs/2505.07608) |
| Phi-4 (synthetic-data SFT) | (HF only) | — | 2024.12 | Microsoft Research | [arXiv 2412.08905](https://arxiv.org/abs/2412.08905) |
| [Tencent-Hunyuan/Hunyuan-A13B](https://github.com/Tencent-Hunyuan/Hunyuan-A13B) | <img src="https://img.shields.io/github/stars/Tencent-Hunyuan/Hunyuan-A13B?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.07 | Tencent | Tech report |
| [Tencent/Hunyuan-Large](https://github.com/Tencent/Hunyuan-Large) | <img src="https://img.shields.io/github/stars/Tencent/Hunyuan-Large?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.11 | Tencent | [arXiv 2411.02265](https://arxiv.org/abs/2411.02265) |
| [MoonshotAI/Kimi-K2](https://github.com/MoonshotAI/Kimi-K2) | <img src="https://img.shields.io/github/stars/MoonshotAI/Kimi-K2?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.07 | Moonshot AI | [arXiv 2507.20534](https://arxiv.org/abs/2507.20534) |
| [01-ai/Yi](https://github.com/01-ai/Yi) (Yi-Lightning) | <img src="https://img.shields.io/github/stars/01-ai/Yi?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.12 | 01.AI | [arXiv 2412.01253](https://arxiv.org/abs/2412.01253) |

### Pure-RL or pure-self-evolution works (NOT distillation)

| Resource | 🌟 Stars | Date | Org | Paper |
| :----: | :----: | :----: |  :----: | :----: |
| [facebookresearch/swe-rl](https://github.com/facebookresearch/swe-rl) | <img src="https://img.shields.io/github/stars/facebookresearch/swe-rl?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.02 | Meta / UIUC / CMU | [arXiv 2502.18449](https://arxiv.org/abs/2502.18449) |
| [SkyworkAI/Skywork-OR1](https://github.com/SkyworkAI/Skywork-OR1) | <img src="https://img.shields.io/github/stars/SkyworkAI/Skywork-OR1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | Skywork AI | [arXiv 2505.22312](https://arxiv.org/abs/2505.22312) |
| [hkust-nlp/simpleRL-reason](https://github.com/hkust-nlp/simpleRL-reason) (SimpleRL-Zoo) | <img src="https://img.shields.io/github/stars/hkust-nlp/simpleRL-reason?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | HKUST | [arXiv 2503.18892](https://arxiv.org/abs/2503.18892) |
| [ulab-uiuc/Time-R1](https://github.com/ulab-uiuc/Time-R1) | <img src="https://img.shields.io/github/stars/ulab-uiuc/Time-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | UIUC | [arXiv 2505.13508](https://arxiv.org/abs/2505.13508) |
| [facebookresearch/coconut](https://github.com/facebookresearch/coconut) (Coconut latent CoT) | <img src="https://img.shields.io/github/stars/facebookresearch/coconut?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.12 | Meta FAIR | [arXiv 2412.06769](https://arxiv.org/abs/2412.06769) |
| [Open-Reasoner-Zero/Open-Vision-Reasoner](https://github.com/Open-Reasoner-Zero/Open-Vision-Reasoner) (RL) | <img src="https://img.shields.io/github/stars/Open-Reasoner-Zero/Open-Vision-Reasoner?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.07 | OR-Zero | [arXiv 2507.05255](https://arxiv.org/abs/2507.05255) |
| [yihedeng9/OpenVLThinker](https://github.com/yihedeng9/OpenVLThinker) | <img src="https://img.shields.io/github/stars/yihedeng9/OpenVLThinker?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | UC | [arXiv 2503.17352](https://arxiv.org/abs/2503.17352) |
| [Fancy-MLLM/R1-onevision](https://github.com/Fancy-MLLM/R1-onevision) | <img src="https://img.shields.io/github/stars/Fancy-MLLM/R1-onevision?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | Fancy-MLLM | [arXiv 2503.10615](https://arxiv.org/abs/2503.10615) |
| [TIGER-AI-Lab/VL-Rethinker](https://github.com/TIGER-AI-Lab/VL-Rethinker) | <img src="https://img.shields.io/github/stars/TIGER-AI-Lab/VL-Rethinker?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | TIGER-AI-Lab | [arXiv 2504.08837](https://arxiv.org/abs/2504.08837) |
| [Osilly/Vision-R1](https://github.com/Osilly/Vision-R1) | <img src="https://img.shields.io/github/stars/Osilly/Vision-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | Osilly | [arXiv 2503.06749](https://arxiv.org/abs/2503.06749) |
| [tulerfeng/Video-R1](https://github.com/tulerfeng/Video-R1) | <img src="https://img.shields.io/github/stars/tulerfeng/Video-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | Multi-org | [arXiv 2503.21776](https://arxiv.org/abs/2503.21776) |
| [CSfufu/Revisual-R1](https://github.com/CSfufu/Revisual-R1) | <img src="https://img.shields.io/github/stars/CSfufu/Revisual-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.06 | Multi-org | [arXiv 2506.04207](https://arxiv.org/abs/2506.04207) |
| [HumanMLLM/R1-Omni](https://github.com/HumanMLLM/R1-Omni) | <img src="https://img.shields.io/github/stars/HumanMLLM/R1-Omni?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.03 | HumanMLLM | [arXiv 2503.05379](https://arxiv.org/abs/2503.05379) |
| [aim-uofa/Omni-R1](https://github.com/aim-uofa/Omni-R1) | <img src="https://img.shields.io/github/stars/aim-uofa/Omni-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | aim-uofa | [arXiv 2505.20256](https://arxiv.org/abs/2505.20256) |
| Online In-Context Distillation (inference-time) | 📄 paper-only | 2025.10 | Toyota / Inria | [arXiv 2510.18117](https://arxiv.org/abs/2510.18117) |
| [THUDM/WebRL](https://github.com/THUDM/WebRL) | <img src="https://img.shields.io/github/stars/THUDM/WebRL?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2024.11 | Tsinghua | [arXiv 2411.02337](https://arxiv.org/abs/2411.02337) |
| [weizhepei/WebAgent-R1](https://github.com/weizhepei/WebAgent-R1) | <img src="https://img.shields.io/github/stars/weizhepei/WebAgent-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.05 | Amazon / UVA | [arXiv 2505.16421](https://arxiv.org/abs/2505.16421) |
| [InfiXAI/InfiGUI-G1](https://github.com/InfiXAI/InfiGUI-G1) | <img src="https://img.shields.io/github/stars/InfiXAI/InfiGUI-G1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.08 | InfiX AI | [arXiv 2508.05731](https://arxiv.org/abs/2508.05731) |
| [ritzz-ai/GUI-R1](https://github.com/ritzz-ai/GUI-R1) | <img src="https://img.shields.io/github/stars/ritzz-ai/GUI-R1?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.04 | CAS / NUS | [arXiv 2504.10458](https://arxiv.org/abs/2504.10458) |
| Magistral (Mistral) | (HF release) | — | 2025.06 | Mistral AI | [arXiv 2506.10910](https://arxiv.org/abs/2506.10910) |
| [Alibaba-NLP/DeepResearch](https://github.com/Alibaba-NLP/DeepResearch) (Tongyi DeepResearch — RL not OPD) | <img src="https://img.shields.io/github/stars/Alibaba-NLP/DeepResearch?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.10 | Alibaba Tongyi | [arXiv 2510.24701](https://arxiv.org/abs/2510.24701) |
| [MiniMax-AI/MiniMax-M2](https://github.com/MiniMax-AI/MiniMax-M2) (CISPO RL) | <img src="https://img.shields.io/github/stars/MiniMax-AI/MiniMax-M2?style=for-the-badge&logo=github&logoColor=white&labelColor=181717&color=ffd700" alt="Stars"> | 2025.11 | MiniMax | [Blog](https://www.minimax.io/news/post-training-experience-and-insights-for-agent-models) |

### Analysis-only / position papers (no algorithm)

| Title | arXiv | Date | Org |
| :----: | :----: | :----: | :----: |
| Why Distillation > Zero-RL? | [2505.21067](https://arxiv.org/abs/2505.21067) | 2025.05 | Academic |
| RL vs. Distillation | [2505.14216](https://arxiv.org/abs/2505.14216) | 2025.05 | NYU / Stanford |
| Why Self-Distillation (Sometimes) Degrades Reasoning? | [2603.24472](https://arxiv.org/abs/2603.24472) | 2026.03 | MSR / KAIST / SNU |

---

## ⚠️ Caveats — works that are NOT OPD (despite naming)

The April 2026 verification pass moved ~50 entries out of OPD sections after reading abstracts / methodology / repos. Highlights of the corrections:

**Production reports that look like OPD but aren't**:
- **DeepSeek-R1 / V3 / V3.2-Exp** — R1's `Distill` family is pure off-policy SFT on 800K R1-generated traces; no student rollouts. V3 / V3.2 inherit this off-policy specialist-distill pipeline.
- **Tongyi DeepResearch** — strict on-policy *GRPO RL*, not OPD. The paper has no teacher-distillation loss.
- **Magistral** — Medium is pure RL; Small is off-policy SFT on Medium's traces. "Self-distillation" here = off-policy SFT on a same-family teacher.
- **MiniMax-M2** — CISPO (clipped importance-sampled RL); the system-prompt distillation trick is heuristic, not classical OPD.
- **Qwen2.5 / Qwen2.5-Math** — tech reports describe SFT + RL; no on-policy distillation phase.
- **Phi-4** — synthetic-data SFT; the abstract literally says it goes "beyond distillation".
- **Hunyuan-Large / A13B** — reports describe SFT + DPO, no on-policy distillation.
- **Kimi K2 / K2.5, Yi-Lightning** — RLHF, no distillation in the report.
- **DistilQwen** — black-box (instruction rewrite) + white-box top-K logit matching on fixed teacher outputs; explicitly off-policy.

**Algorithmic papers that turned out off-policy**:
- **Minitron** — pruning + retraining on a *static fraction* (<3%) of the original training corpus. No student rollouts.
- **MiniPLM** — explicitly "operates solely on the training corpus" with offline teacher inference. Pretraining-time, not OPD.
- **ULD (Cross-Tokenizer KD)** — OT loss on text *generated by the teacher*; teacher-driven offline KD.
- **TAID / DSKD / DASD** — distribution-interpolation / projection methods; abstracts give no on-policy commitment, primarily framed as offline white-box KD.
- **Lion** — gradients flow only from teacher targets. "Adversarial" stage only selects hard prompts.
- **SuperCorrect** — cross-model DPO uses *static* preference pairs collected before training begins.
- **DAIL / SODA** — teacher-rewritten / one-time static student-output snapshots.
- **AKL, ToDi, AMiD, DRKL, AdaKD, Delta KD, CSD, HPD** — pure loss-function / divergence-design papers; no on-policy commitment in abstracts. Treated as off-policy KD references.
- **AlignDistil** — RLHF-equivalent KD via DPO-derived teacher; an RL-distill hybrid, not pure white-box OPD. **Moved to OPD-RL Hybrids.**
- **BOND, Faster WIND, KETCHUP, 𝒳-KD, DDT** — sequence-level RL / IRL formulations of distillation. **Moved to OPD-RL Hybrids.**

**OPSD-named works that don't sample student rollouts**:
- **SDFT-2024 (sail-sg)** — offline data rewriting + standard SFT. README itself flagged "Off (data-side)".
- **π-Distill** — student trains on *teacher's* trajectories with importance weighting, not its own rollouts.
- **TMS** — student trains on harvested, pre-computed rollouts from intermediate checkpoints.
- **Self-Data Distillation for Pruned LLMs** — original model generates a *fixed* dataset before fine-tuning.
- **Apple Memory-Retaining FT** — KL anchor on FT corpus to original pretrained checkpoint; standard regularisation.
- **SPIN** — teacher is the *previous iteration's checkpoint* (Q1=C, not Q1=B); supervision is DPO-style sequence-level. Moved to **Iterative Self-Bootstrapping**.

**Pure-RL works mistakenly called OPD-adjacent**:
- **Skywork-OR1, SimpleRL-Zoo, Time-R1** — pure rule-based RL, no teacher distillation (Skywork-OR1 is *used as* a teacher by other OPD papers like KDRL).
- **Coconut** — staged-curriculum SFT on synthesized continuous-thought traces; not student-rollout KL OPD.
- **Open-Vision-Reasoner, OpenVLThinker, R1-Onevision, VL-Rethinker, Vision-R1, Video-R1, Revisual-R1, R1-Omni, Omni-R1** — all are SFT cold-start + GRPO; no teacher-KL on student rollouts.
- **WebRL, WebAgent-R1, InfiGUI-G1, GUI-R1** — pure RL agent training.
- **rStar / rStar-Math / rStar2-Agent** — iterative MCTS self-improvement; the model filters its own samples through PPM. Listed under [Iterative Self-Bootstrapping (Q1=C)](#-iterative-self-bootstrapping-q1c).
- **RLKD, ExGRPO, REDI** — pure RL or pure off-policy distillation, no per-token teacher signal.

**Speculative-decoding entries removed (don't train the drafter)**:
- **Ouroboros** (training-free phrase recycling), **Sequoia** (system DP tree), **TriForce** (system KV hierarchy), **SwiftKV** (KV transform), **SuffixDecoding** (model-free).

**Framework section cleanup**:
- **axolotl, OpenRLHF, prime-rl, allenai/open-instruct, TextBrewer, open-r1** — no native OPD support in their READMEs as of Apr 2026. Removed or moved to Off-Policy KD references.
- **openai/gpt-oss** — moved from frameworks to industrial reports (it's a model release).
- **Tunix v0.1.6** — has a `DistillationTrainer` but only `logit_distillation.ipynb` (offline). Kept with caveat.

These corrections are documented per-entry in their target sections.

---

## 📜 Citation guidance & Acknowledgements

If this list helped your research, please cite the works directly. The two most-recommended canonical references for "OPD as a 2026-era research area":

```bibtex
@article{lu2025opd,
  title  = {On-Policy Distillation},
  author = {Lu, Kevin and others},
  journal = {Thinking Machines Lab Connectionism},
  year   = {2025},
  doi    = {10.64434/tml.20251026},
  url    = {https://thinkingmachines.ai/blog/on-policy-distillation/}
}

@article{song2026opdsurvey,
  title  = {A Survey of On-Policy Distillation for Large Language Models},
  author = {Song, Mingyang and Zheng, Mao},
  journal = {arXiv preprint arXiv:2604.00626},
  year   = {2026}
}
```

Inspired by [AgentsMeetRL](https://github.com/thinkwee/AgentsMeetRL). Thanks to all contributors who have written explicit-OPD code or technical reports — particularly the Microsoft Research / THUNLP / KAIST groups whose 2026-Q1 paper output drove most of this list.

---

## 🤝 Contributing

PRs are very welcome. When adding an entry, please attempt to fill the technical-details columns (loss / divergence, data source, teacher access, granularity). If you cannot determine these by reading the paper or repo, leave a `?` — that's still useful.

Particularly looking for:
 - Pretraining-time distillation (DeepSeek-V3, Llama 4 Behemoth, Gemma 3 Pro, etc.)
 - Robotics / VLA OPD
 - Speech / TTS OPD beyond X-OPD
 - Non-English production reports (DingTalk Math, Doubao, ERNIE 4.5, …)
 - Industrial OPD failure-mode analyses
