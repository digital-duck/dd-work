# SPL Solver Research Plan
## New TMLR Submission: SPL for Operations Research

**Builds on:** arXiv:2607.07727 (Gong, 2026) — SPL foundational paper, currently under TMLR review  
**Reference guide:** `Solver-Reference-Guide.pdf` (recipes 75–94)  
**Empirical data:** `cookbook/78_constraint_opt/` — 8 logged ablation runs (2026-08-30)  
**Date:** 2026-08-30

---

## 1. The Gap This Paper Fills

The foundational SPL paper (arXiv:2607.07727) establishes the **declarative deterministic-probabilistic composition** paradigm and validates it on symbolic mathematics: a 1,320-run evaluation across 11 models × 10 SymPy problems × 2 arms × 6 repetitions. The verifier ladder covers SymPy (algebraic), SageMath (number-theoretic), and Lean 4 (formal proof).

A TMLR reviewer asking *"how useful is SPL for real-world problems?"* gets an honest but narrow answer from the current submission: symbolic math is rigorous, reproducible, and machine-verifiable — but it is not where most practitioners need help. The problems that fill ops meetings, supply chain dashboards, and project portfolio reviews are **operations research problems**: LP, ILP, routing, scheduling, knapsack. These are just as solver-amenable as SymPy differentiation, but they are where hallucinated answers cause business harm.

This new submission makes the practical case:

> *SPL's GENERATE → CALL solver → ASSERT → WHILE repair pattern applies directly and empirically to real-world OR problems — production planning, logistics, workforce scheduling, portfolio selection — using PuLP/CBC, Google OR-Tools, and a growing ecosystem of domain solvers. The same four-token ASSERT gate that certifies a SymPy derivative certifies a CBC optimal solution. The language generality is not aspirational; it is demonstrated.*

**Self-citation:** This paper builds on arXiv:2607.07727 for the SPL language design, primitives (`GENERATE`, `EVALUATE`, `WHILE`, `EXCEPTION`, `SOLVE`, `ASSERT`), DODA principle, and verifier-ladder concept. The new contribution is the empirical OR domain expansion, the verification-pattern taxonomy, and two engineering principles (§6).

---

## 2. Reviewer Concern: What "Usefulness" Requires

From the current TMLR paper (v3.0), the reviewer challenge on usefulness stems from:

1. **Narrow benchmark scope.** 20 math problems (T0–T5) is too small a sample to support general conclusions (as the paper itself acknowledges in C3). The benchmark is illustrative, not representative of real business workflows.

2. **Symbolic math is not a practitioner pain point.** SymPy/SageMath correctness matters for researchers; LP/ILP correctness matters for ops teams, finance analysts, and supply-chain engineers — a far larger audience.

3. **The 8.3% round-trip failure gap is underexplored.** The current paper shows 87.1% of solver-arm passes are also semantically correct (round-trip verified). The 8.3% false-positive rate — solver ran cleanly but answer is wrong — is exactly the gap that domain-specific ASSERT gates close. OR solvers return `Infeasible` / `Optimal` / `Unbounded`; this is a categorical oracle, not a numerical tolerance check.

4. **LLM-only baseline is not a correctness baseline.** In the current paper, the LLM-only arm measures output production (non-empty response), not mathematical correctness. An OR-domain ablation provides a *correctness baseline*: known optimal solutions for hand-verifiable problems let us measure whether solver=OFF gets the right answer, not just any answer.

**Response strategy:** Replace the narrow math benchmark as the primary empirical claim with a multi-domain OR study. Keep the math evaluation as Appendix evidence of the verifier ladder. Lead with OR because it is the domain where practitioners feel the pain and where the ASSERT guarantee is most tangible.

---

## 3. The OR Solver Ecosystem

The OR ecosystem is orthogonal: it covers a different problem class with a different verification structure.

- [Optimisation Process in OR](https://coin-or.github.io/pulp/main/the_optimisation_process.html)
- [Optimisation Concepts](https://coin-or.github.io/pulp/main/optimisation_concepts.html)

The foundational paper's verifier ladder (R1 SymPy → R2 SageMath → R3 Lean) is a basic *mathematical* hierarchy. 

### 3.1 Core OR solvers (implemented in recipes 75–94)

| Solver | Problem class | SPL recipe | ASSERT predicate | Real-world domains | References |
|---|---|---|---|---|---|
| **PuLP + CBC** | LP / MIP — linear objectives, linear constraints | r78 | `status == "Optimal"` | Production planning, staffing, blending, budget allocation | [PuLP docs](https://coin-or.github.io/pulp/) · [CBC](https://github.com/coin-or/Cbc) · [COIN-OR](https://www.coin-or.org/) |
| **Google OR-Tools** (CP routing) | NP-hard TSP / VRP — vehicle routing | r87 | feasible route found | Delivery routing, technician dispatch, last-mile logistics | [OR-Tools](https://developers.google.com/optimization) · [GitHub](https://github.com/google/or-tools) |
| **python-constraint** | efficient solvers for Constraint Satisfaction Problems [(CSP)](https://www.wikiwand.com/en/Constraint_satisfaction_problem) over finite domains | r82 | `status == "Unique"` | Shift exclusions, SKU incompatibility, eligibility rules | [docs](https://python-constraint.github.io/python-constraint/) · [GitHub](https://github.com/python-constraint/python-constraint) |
| **networkx** | Graph algorithms — shortest path, bipartite matching | r81 | ground-truth match + round-trip | Org-chart reachability, dependency graphs, supply-chain topology | [networkx.org](https://networkx.org/) · [docs](https://networkx.org/documentation/stable/) |
| **pint** (UnitRegistry) | Dimensional analysis — unit safety | r83 | no `DimensionalityError` | Engineering calculators, BOM calculations, multi-unit pipelines | [pint.readthedocs.io](https://pint.readthedocs.io/) · [GitHub](https://github.com/hgrecco/pint) |
| **sqlite3** (stdlib) | Text-to-SQL — NL → query → verified answer | r84, r94 | row-diff vs ground truth | Chat-with-database, NL reporting, data governance | [Python docs](https://docs.python.org/3/library/sqlite3.html) · [SQLite](https://www.sqlite.org/) |
| **decimal** (stdlib) | Exact financial arithmetic | r86 | cent-level tolerance | Invoicing, loan amortization, customer-facing dollar figures | [Python docs](https://docs.python.org/3/library/decimal.html) · [IBM spec](https://speleotrove.com/decimal/) |
| **pytest** | Code-from-spec correctness | r79 | all tests pass | LLM-written scripts / data transforms | [pytest.org](https://docs.pytest.org/) · [GitHub](https://github.com/pytest-dev/pytest) |
| **NumPy + recomputation** | Physics conservation laws | r90 | 1% tolerance on invariant | Engineering simulations, scientific computing | [numpy.org](https://numpy.org/) · [docs](https://numpy.org/doc/stable/) |
| **SymPy** | Symbolic mathematics — algebra, calculus, ODEs, Taylor series | r67, r75, r77 | `simplify(result − expected) == 0` | STEM education, scientific computing, equation solvers, CAS | [sympy.org](https://www.sympy.org/) · [docs](https://docs.sympy.org/) · [GitHub](https://github.com/sympy/sympy) |
| **SageMath** | Advanced math — Laplace transforms, ODEs, infinite sums, Galois groups | r77 | symbolic identity or round-trip re-transform | Research computing, number theory, algebraic topology | [sagemath.org](https://www.sagemath.org/) · [docs](https://doc.sagemath.org/) · [GitHub](https://github.com/sagemath/sage) |

### 3.2 Suggested expansion (high-priority)

| Solver | Problem class | Why now | References |
|---|---|---|---|
| **Google OR-Tools CP-SAT** | Job-shop / shift scheduling — distinct from routing | Machine scheduling, shift optimization; very high business demand | [CP-SAT guide](https://developers.google.com/optimization/cp) · [OR-Tools](https://developers.google.com/optimization) |
| **cvxpy** | Convex optimization — portfolio risk, sector-cap allocation | Natural sibling to Recipe 78; finance-facing, widely used in quant | [cvxpy.org](https://www.cvxpy.org/) · [GitHub](https://github.com/cvxpy/cvxpy) · [paper](https://www.jmlr.org/papers/v17/15-408.html) |
| **Z3** (Microsoft SMT) | theorem prover: Eligibility rules, pricing tiers, compliance logic | More expressive than python-constraint for non-toy rule engines | [Z3 guide](https://microsoft.github.io/z3guide/) · [GitHub](https://github.com/Z3Prover/z3) · [Python API](https://z3prover.github.io/api/html/namespacez3py.html) |
| **GX** | Data quality framework | Productionizes Recipe 94's hand-rolled quality gates | [Great Expectations](https://greatexpectations.io/) |
| **pandera** | Dataset Validation | make data processing pipelines more readable and robust with statistically typed dataframes | [pandera](https://pandera.readthedocs.io/) |
| **Prolog** (free) | Contract / compliance satisfiability | "Can clauses X and Y ever both hold?" — insurance, legal | [swi-prolog.org](https://www.swi-prolog.org/) · [docs](https://www.swi-prolog.org/pldoc/index.html) |
| **Shapely / PostGIS** | Geospatial — polygon intersection, service-area coverage | Zoning constraints, service territory verification | [Shapely](https://shapely.readthedocs.io/) · [PostGIS](https://postgis.net/) · [GitHub](https://github.com/shapely/shapely) |

### 3.3 The verification-pattern taxonomy (7 classes)

Reading recipes 75–94 in order, the verification structure gets progressively stronger:

| ID | Class | Description | OR examples |
|---|---|---|---|
| **C1** | **Categorical oracle** | Solver status is binary: success or typed failure. No tolerance. | PuLP `Optimal` vs `Infeasible`; python-constraint `Unique`; OR-Tools feasible-route |
| **C2** | **Numeric round-trip** | Solver's answer compared to LLM narration within explicit tolerance | `classify_roundtrip()` in graph reasoning (r81), financial calc (r86), physics (r90) |
| **C3** | **Independent recomputation** | Separate deterministic function re-derives invariant from primitive inputs | Physics momentum recomputation (r90); crystal density from lattice parameters (r92) |
| **C4** | **Structural ground-truth diff** | Structured output compared row-by-row against independently authored reference | SQL row-diff vs human-written query (r84) |
| **C5** | **Two-independent-ways agreement** | Same quantity via two structurally different derivations — domain property, LLM-independent | `AVG` vs `SUM/COUNT` gate (r94) — strongest for compliance/audit |
| **C6** | **Falsification-first** | Genuine counter-example is terminal, not a repair trigger | Hypothesis properties (r85) — repair loop must be able to fail honestly |
| **C7** | **Kernel-checked proof + faithfulness gap** | Solver proves formalized statement; LLM separately certifies faithfulness | Lean 4 (r76) — formal verification has a boundary; SPL surfaces it |

**Design principle:** Class 1 covers most "does this satisfy hard constraints" OR problems at minimal cost. Classes 4–5 are required for data-governance and finance audiences. Class 7 is for policy-compliance claims.

---

## 4. Empirical Evidence: Recipe 78 Ablation Study

**Design:** 4 OR problem types × 2 conditions (solver=ON / solver=OFF) × same model (`claude-sonnet-4-6`, `claude_cli`) × same problem text, same environment.  
**Date:** 2026-08-30  
**Logs:** `cookbook/78_constraint_opt/logs/` (8 full execution traces)  
**Note:** The §4.1 table below is the original `claude-sonnet-4-6` (m001) ablation. Experiment H1 (gemma3/m002) results are being collected separately — see `experiment_results.db`. The Production planning known-optimal was corrected from $168 to $200 (furniture problem: 24h labor, 30kg wood, optimal at chairs=10, tables=0).

### 4.1 Results — m001 (claude-sonnet-4-6), n05 baseline

| Recipe | PuLP type | ON tokens | OFF tokens | ON output tok | OFF output tok | Output Δ | ON latency | OFF latency | ON correct | OFF correct |
|---|---|---|---|---|---|---|---|---|---|---|
| Production planning | LP | 982 | 1,035 | 441 | 760 | **+72%** | 22.6s | 45.2s | ✅ $200 | ✅ $200 |
| Transportation | LP | 1,468 | 1,180 | 734 | 856 | **+17%** | 38.5s | 36.8s | ✅ $370 | ✅ $370 |
| Staff scheduling | ILP | 1,502 | 961 | 636 | 566 | **−11%** | 27.0s | 24.9s | ✅ $1,080 | ✅ $1,080 |
| Portfolio selection | Binary ILP | 1,837 | 1,629 | 832 | 1,158 | **+39%** | 50.1s | 76.9s | ✅ 28 | ✅ 28 |
| **Mean** | | **1,447** | **1,201** | **661** | **835** | **+29%** | **34.6s** | **46.0s** | | |

### 4.1b Results — Experiment H1: m002 (gemma3), n05 model-downgrade study

**Design:** Same 4 problems × 2 solver modes × gemma3 (local 8B) vs claude-sonnet-4-6 above.  
**Purpose:** Test F5 (model portability) and F6 (solver=ON as correctness equalizer).

| Recipe | PuLP type | ON obj | ON correct | ON LLM calls | OFF obj | OFF verify | OFF correct |
|---|---|---|---|---|---|---|---|
| r78a — Production planning | LP | **200.0** | ✅ | 2 | 180.0 | PASS\* | ❌ suboptimal |
| r78b — Transportation | LP | **370.0** | ✅ | 2 | 260.0 | FAIL | ❌ violated |
| r78c — Staff scheduling | ILP | **1,080.0** | ✅ | 2 | 1,080.0 | PASS | ✅ |
| r78d — Portfolio selection | Binary ILP | **28.0** | ✅ | 4 | 17.0 | PASS\* | ❌ suboptimal |
| **Summary** | | **4/4 correct** | | 2.5 avg | | | **1/4 correct** |

\* PASS = back-substitution confirmed feasibility only; verify cannot detect suboptimality (F8).

**H1 key observations:**
- solver=ON: 4/4 correct across all problem types despite gemma3 being a fraction of claude-sonnet-4-6's capability. Confirms F6.
- solver=OFF: 1/4 correct. r78b violated a demand constraint (infeasible solution); r78a and r78d found feasible but non-optimal corners.
- r78d needed 3 repair iterations (4 LLM calls total) — the binary ILP formulation is the hardest for gemma3 to write correctly on the first attempt.
- r78c solver=OFF correct: the small scheduling problem (5 nurses, 3 shifts) is solvable by greedy; a larger instance would likely fail (see H2).

### 4.2 Experiment H2: Scale Sensitivity (2026-08-30)

**Design:** 4 recipes × 2 models × 3 sizes (n05/n10/n20) × 2 solver modes = 48 cells. n05 rows reuse H1 results (m002) and baseline data (m001). m001 cells hit claude_cli session timeouts in the batch run; re-run pending — r78a and r78b solver=ON results are available for m001.

**Problem scaling:**
- **n05** — minimal: 2–3 warehouses, 5 nurses, 6 projects
- **n10** — medium: 4–5 warehouses, 10 nurses, 10 projects
- **n20** — large: 8 warehouses, 5 shifts, 20 projects

#### 4.2a m002 (gemma3) — solver=ON accuracy by scale

| Recipe | Type | n05 (H1) | n10 | n20 |
|--------|------|-----------|-----|-----|
| r78a — LP (production) | LP | ✅ 200 | ❌ infeasible (3 calls) | ❌ infeasible (3 calls) |
| r78b — LP (transport) | LP | ✅ 370 | ❌ infeasible (3 calls) | ✓ 1660 (no ref) |
| r78c — ILP (scheduling) | ILP | ✅ 1,080 | ✅ **2,640** | ✅ **5,030** |
| r78d — Binary ILP (portfolio) | Binary ILP | ✅ 28 | ✅ **45** | ❌ infeasible (3 calls) |
| **Correct** | | **4/4** | **2/4** | **1/4 (+1 unverified)** |

`infeasible` = CBC returned Infeasible on gemma3's formulation (wrong constraints, not a truly infeasible problem). Bold = matches known_optimal. r78b/n20=1660 ran to completion but has no reference solution to verify against.

#### 4.2b m002 (gemma3) — solver=OFF accuracy by scale

| Recipe | n05 (H1) | n10 | n20 |
|--------|----------|-----|-----|
| r78a | ❌ 180 (PASS\*, suboptimal) | ❌ 178 FAIL | ❌ UNPARSEABLE |
| r78b | ❌ 260 FAIL | ❌ 710 FAIL | ❌ UNPARSEABLE |
| r78c | ✅ 1,080 PASS | ❌ 7,000 FAIL | ❌ 9,270 FAIL |
| r78d | ❌ 17 (PASS\*, suboptimal) | ❌ 65 FAIL | ❌ 54 FAIL |
| **Correct** | **1/4** | **0/4** | **0/4** |

`UNPARSEABLE` = gemma3's reasoning output at n20 scale could not be parsed for an objective value — a qualitatively worse failure mode than wrong-but-structured answers at n05/n10.

#### 4.2c m001 (claude-sonnet-4-6) — partial results (re-run pending)

| Recipe | Type | n05 solver=ON | n10 solver=ON | n20 solver=ON | n05 solver=OFF | n10 solver=OFF | n20 solver=OFF |
|--------|------|--------------|--------------|--------------|---------------|---------------|---------------|
| r78a — LP (production) | LP | ✅ 200 | ✓ 1,470 (unverified) | ✓ 2,355 (unverified) | ✅ 200 PASS | *pending* | *pending* |
| r78b — LP (transport) | LP | ✅ 370 | ✓ 840 (unverified) | *pending* | ✅ 370 PASS | *pending* | *pending* |
| r78c — ILP (scheduling) | ILP | *pending* | *pending* | *pending* | *pending* | *pending* | *pending* |
| r78d — Binary ILP | Binary ILP | *pending* | *pending* | *pending* | *pending* | *pending* | *pending* |

"Unverified" = solver returned `Optimal` (ASSERT passed) but no known_optimal reference exists. r78c and r78d rows are all pending — those cells timed out in the batch run.

### 4.3 Key findings

| ID | Finding | Description |
|---|---|---|
| F1 | solver=ON is 1.3× faster end-to-end | Stage 1 latency is stable 7–9s across all four problem types (LLM writes a short template; CBC searches feasible space in milliseconds). solver=OFF Stage 1 grows with complexity: 44.7s (LP) → 76.5s (Binary ILP, 2^6=64 subsets). |
| F2 | Token cost is O(formulate + repair + interpret), not O(reasoning) | solver=OFF generates 29% more output tokens on average. For Binary ILP, Stage 1 alone (76.5s, 1,158 tokens) exceeded solver=ON's total latency (50.1s). Combinatorial blowup happens inside CBC, not the context window. |
| F3 | ASSERT is a correctness oracle, not an execution monitor | `{"status": "Infeasible"}` is a successful Python execution but a wrong answer; ASSERT catches it. solver=OFF has no equivalent gate — a confident, wrong answer has no failure signal. |
| F4 | LLM correct on all 4 solver=OFF runs (default n=5 problems) | The LLM used named algorithms (corner enumeration, MODI, greedy, exhaustive C(6,4)=15 enumeration). Solver value: (a) efficiency at small n, (b) guaranteed correctness where LLM enumeration fails at large n, (c) ASSERT certificate transforms probabilistic claim into verifiable fact. |
| F5 | Model portability holds | Same .spl workflow for solver=ON and solver=OFF — only `use_solver` parameter differs. Identical spec runs on `gemma3` (local, cheap) or `claude_cli` (cloud, capable). DODA applied to OR: orchestration is invariant, only model selection changes. |
| F6 | solver=ON is a correctness equalizer across model tiers | gemma3 fails solver=OFF on Binary ILP (selects all projects ignoring both constraints; claimed obj=41 vs optimal 28; verified FAIL). With solver=ON it reaches the same provably-correct answer as claude-sonnet-4-6. Correctness is a property of the architecture, not the model: the solver is the correctness source; the LLM is the interface layer; model capability determines formulation quality, not solution quality. |
| F8 | Back-substitution (solver=OFF verify) detects infeasibility but not suboptimality | H1 gemma3 solver=OFF: r78a verify=PASS at obj=180 (optimal=200); r78d verify=PASS at obj=17 (optimal=28). In both cases the LLM found a valid corner of the feasible region but not the optimal corner. The verifier re-checks constraints and objective arithmetic — it cannot certify optimality without an exhaustive search. This is the structural gap solver=OFF cannot close: PASS means "no constraint is violated and the arithmetic is consistent," not "this is the best feasible solution." The ASSERT gate in solver=ON closes this gap categorically — `Optimal` is the solver's claim, verified by a proof-complete search. |
| F9 | Scale breaks LP formulation for gemma3 but not ILP scheduling | solver=ON accuracy: n05=4/4 → n10=2/4 → n20=1/4 for m002. LP recipes (r78a, r78b) fail with "infeasible" at n10/n20 — gemma3 hallucinates extra constraints as problem description grows, making CBC correctly declare the *wrong* problem infeasible. r78c (scheduling ILP) stays perfect at all three sizes: the concrete 2D-dict code skeleton in `formulate_scheduling_ilp` is scale-invariant — the LLM fills in data; the structure is provided. Engineering principle: for problem classes where gemma3 formulation degrades, a template skeleton in the prompt is more reliable than repair-loop correction. |
| F10 | Solver=OFF collapses at scale with a new failure mode (UNPARSEABLE) | solver=OFF correctness: n05=1/4 → n10=0/4 → n20=0/4 for m002. Two n20 cells return UNPARSEABLE output — direct LLM reasoning fails not just in accuracy but in producing interpretable structure. At n05 gemma3 sometimes finds a feasible (if suboptimal) corner; at n20 it cannot organize its reasoning into any parseable form. This is a qualitative phase transition: wrong-but-structured → wrong-and-unstructured. |
| F7 | Systematic LLM code-generation hallucinations require TOOL_API preprocessing, not repair prompting | gemma3 consistently generates `from pulp import *` despite explicit instructions to the contrary. In PuLP 3.x, this statement exports the internal `pulp.pulp` submodule under the name `pulp`, silently rebinding the injected top-level module reference so that `pulp.LpStatus` fails with `AttributeError`. Three repair-loop iterations fed the resulting error back to the LLM; all three attempts reproduced the same pattern — the model has no self-correction path for its own systematic hallucination. Resolution required deterministic preprocessing in the `run_pulp` TOOL_API wrapper: strip `from pulp import *` before exec, pre-inject PuLP exports with the submodule name excluded. Engineering principle: the TOOL_API wrapper is the correct layer for absorbing model-specific code generation quirks — the repair prompt is not. Models cannot reliably correct patterns they cannot observe as wrong. |

---

## 5. Two Engineering Principles for the Paper

These generalize beyond the Recipe 78 data to the full OR solver ecosystem.

### Principle A — For problems with solvers, do not train LLMs to solve them

Training signal on LP solutions, scheduling assignments, and knapsack enumerations teaches the model to emulate a deterministic algorithm at probabilistic cost. The correct division of labor:

| Subtask | Mode | Why |
|---|---|---|
| Parse NL problem → formal intent | Probabilistic (LLM) | Semantics, ambiguity, domain knowledge |
| Translate intent → solver input code | Probabilistic (LLM) | Code synthesis; bounded repair loop corrects errors |
| Search feasible space for optimum | **Deterministic (solver)** | Provably optimal; scales with n; zero token cost |
| Interpret verified result | Probabilistic (LLM) | Stakeholder communication, strategy, context |

Model portability is a corollary: since the LLM only formulates and narrates (not computes), a smaller local model can often formulate correct PuLP code even when it would get the raw arithmetic wrong. This decouples model cost from problem complexity.

> *"Making the model bigger to enumerate binary subsets faster is the wrong investment. The right investment is the ASSERT boundary: teach the model to hand off, not to enumerate."*

### Principle B — Once LLM synthesis maps to a deterministic solver, encode it permanently

The solver=OFF pattern re-derives from scratch on every invocation: 76s, 1,158 output tokens, zero reuse. The solver=ON pattern does something qualitatively different: **the LLM's first synthesis becomes a permanent deterministic artifact.** The `run_pulp()` TOOL_API body, once generated, is called for any instance of the same problem class at zero LLM token cost for the solve step.

Token savings over k invocations:
```
savings(k) = k × cost(solver=OFF) - cost(formulate) - k × cost(interpret)
           ≈ k × 1,158 - 238 - k × 594    [Binary ILP numbers]
           = k × 564 - 238
```
This becomes positive at k = 1 (second invocation). Every subsequent run saves ~564 output tokens and ~50 seconds.

The ASSERT gate is the structural mechanism: once a deterministic solution exists, execution cannot re-enter the probabilistic loop. The one-time synthesis becomes a certified rung.

> *"There is no point generating tokens from scratch for a problem you already solved deterministically. Encode it once; run it forever."*

---

## 6. Paper Outline for New TMLR Submission

**Proposed title:** *"SPL for Operations Research: Verified Hybrid Workflows Across LP, ILP, and Combinatorial Optimization"*

**Self-citation:** arXiv:2607.07727 (Gong, 2026) — for SPL language design, DODA principle, GENERATE/EVALUATE/WHILE/EXCEPTION/SOLVE/ASSERT primitives, and verifier ladder concept.

### Abstract (draft)
Operations research problems — production planning, vehicle routing, workforce scheduling, portfolio selection — require exact, verifiable answers, not probabilistic approximations. We present an empirical study of hybrid declarative workflows using SPL (arXiv:2607.07727) applied to four OR problem classes: continuous LP, transportation LP, ILP, and Binary ILP. Each workflow uses the same five-stage pattern: LLM formulates solver code → deterministic solver (PuLP/CBC) executes → ASSERT gates on `Optimal` status → bounded LLM repair loop on failure → LLM interprets verified result. We conduct a controlled ablation (solver=ON vs solver=OFF) across 8 runs with known optimal solutions, finding: (1) solver=ON is 1.3× faster end-to-end despite making an extra LLM call; (2) solver=OFF generates 29% more output tokens on average, with the Binary ILP showing 8.6× slower Stage 1 than solver=ON; (3) ASSERT provides a categorical optimality certificate unavailable in the LLM-only path; (4) the same .spl specification runs unmodified from gemma3 (local) to claude-sonnet-4-6 (cloud). We further present a solver taxonomy covering 17 problem classes (LP, CSP, graph, units, SQL, code, financial, physical, planning) and a 7-class verification-pattern hierarchy. The results operationalize DODA — Design Once, Deploy Anywhere — in the operations research domain: one workflow specification, multiple solver backends, each step assigned to the computation mode where it is provably most efficient.

### Section outline

| Section | Content |
|---|---|
| **1. Introduction** | The OR practitioner's problem: "I need an answer I can trust, not a plausible one." Gap in current LLM tooling. Self-cite arXiv:2607.07727 for SPL language. |
| **2. Background** | SPL primitives: GENERATE, EVALUATE, WHILE, EXCEPTION, SOLVE, ASSERT, TOOL_API. The DODA principle. The canonical 5-stage spine. One paragraph per primitive; forward reference to arXiv:2607.07727 for full grammar. |
| **3. Related work** | LLM+P, MCP-Solver, DUPLEX — all tool-call patterns without language-level mode boundary. Operations research literature (LP/ILP/VRP). Neurosymbolic AI. |
| **4. The OR solver ecosystem** | §3 of this document: 17 current recipes mapped to domain/solver/verification class. 7-class verification taxonomy. Business quick-reference table. |
| **5. Recipe 78: Controlled ablation** | The 4-recipe × 2-condition study. Design, methodology, results table (§4 of this document). Key findings F1–F10. |
| **6. Engineering principles** | Principle A (don't train LLMs on solver problems). Principle B (one-time synthesis, infinite reuse). Token cost model. Model portability. |
| **7. Discussion** | The 8.3% false-positive gap from arXiv:2607.07727: ASSERT closes it categorically for OR (no tolerance needed — `Optimal` is binary). Honest failure vs hallucinated number. Scalability: solver cost stays flat; LLM formulation cost stays flat; only solver runtime grows with n. |
| **8. Future work** | OR-Tools CP-SAT job-shop (Recipe 95), cvxpy portfolio (96), Z3 compliance (97). Experiment H1: model downgrade study. Experiment H2: problem-scale sensitivity. |
| **9. Conclusion** | One paragraph. |

---

## 7. Connections to arXiv:2607.07727

| Current paper claim | New paper strengthens via |
|---|---|
| "Any Python-callable verifier plugs in" (§4.6 Pluggability) | 17 recipe domains demonstrated empirically, not just asserted |
| "ASSERT checks solver's verdict, not execution success" (§3.3.2) | Recipe 78: `Infeasible` is the case — categorical, no tolerance |
| "Token cost is O(formulate + repair + interpret)" (§1 intro framing) | Quantified: 29% mean output token savings, 1.3× latency improvement |
| "DODA: same .spl runs from gemma3 to claude_cli" (C2) | Confirmed on OR recipes; model swap requires no specification change |
| Verifier ladder: R1 SymPy → R2 SageMath → R3 Lean | Extended: horizontal solver ecosystem (OR, CSP, graph, SQL, financial) orthogonal to the ladder |
| 8.3% round-trip failure gap (§6.2) | OR ASSERT closes this categorically — binary verdict leaves no gap |

---

## 8. Tasks

| ID | Task | Purpose |
|---|---|---|
| T1 | ~~Run Experiment H2~~ **m002 COMPLETE; m001 re-run pending** (`bash cookbook/78_constraint_opt/run_experiment.sh -m m001 -n n05,n10,n20`) | m002: solver=ON 4/4→2/4→1/4; solver=OFF 1/4→0/4→0/4. m001 r78a/r78b solver=ON partial (available); r78c/r78d timed out |
| T2 | ~~Run Experiment H1 (model downgrade): Recipe 78 with gemma3~~ **COMPLETE** | 4/4 solver=ON correct; 1/4 solver=OFF correct; see §4.1b and F6–F8 |
| T3 | Draft Recipe 98 (OR-Tools CP-SAT job-shop) | First new recipe for new paper; different solver class from PuLP |
| T4 | Draft Recipe 99 (cvxpy portfolio optimization) | Finance domain; convex vs combinatorial — links to C1 oracle; personal investment demo |
| T5 | Add arXiv self-citation to §2 background | Formal connection to language design paper |
| T6 | Write §7 discussion on the 8.3% gap | Most load-bearing section for TMLR reviewers — references C1 |
