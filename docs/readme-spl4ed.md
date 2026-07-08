**Blocking (reviewers will find these):**
- [x] **§6.4 pass-rate table (M2)** — the promissory "will populate in the final version" sentence was
      deleted (2026-07-04); §6.4 is now qualitative-only. Still to decide: back the remaining qualitative
      claims with a small table from run logs (`SPL.py/cookbook/74_concept_book/logs-spl/`) — even partial
      ("N domains with books; first-pass rate, mean refinement count, failure category") — or trim §6.4
      further. Note §6's preamble still says "run-log data is still being collected"; keep it consistent.

[WEN] create a small table from run logs for internal review, we may not use it for paper, Let us keep claim qualitative-only.

DONE — internal-only report at `internal-run-stats-2026-07-08.md` (not referenced from the paper).
Mined `logs-spl/` (277 logged section-generation events) and `SPL.py/.spl/content_meta.db` (161 rows).
**Headline finding: the pass-rate/refinement data doesn't actually exist.** `content_meta.verdict` is
empty on all 161 rows, and `badges` (the closest proxy) is populated on only 4 legacy rows that predate
per-domain tracking — every row for the five domains we do track (Calculus, Chinese Characters,
Classical Mechanics, Cs Algorithms, English Morphology) has `badges = []`. `token_cost` is 0 everywhere
too. So there is no honest first-pass-rate or failure-category table to build — confirms keeping §6.4
qualitative-only was correct, not just cautious. What the logs *do* support: domain/language/LLM coverage
per row, a 69%/31% cache-hit/real-generation split across 277 events, real-generation timing (mean 19.3s,
median 19.2s, range 3.2–82.7s), and one concrete full-book timing (English Morphology → Chinese, 33
concepts + capstone, local ollama:gemma4: 34 LLM calls, ~11.1 min wall-clock, $0) — the last one doubles
as a data point for the separate "Cost/time numbers" cheap-win item below, since it fell out of today's
multilingual-run task for free. Also noticed in passing: Chinese Characters already had a full `zh`-pass
in the cache from a prior `claude_cli` run, predating today's English-Morphology multilingual work.


- [ ] **§6.3 stability numbers (M2)** — report Kendall's τ (or pairwise inversions) between orderings
      at α ∈ {0.5, 1.0, 1.5, 2.0} across the twenty domains; minutes of compute with existing code.

[WEN] this is for future work, I have no resource and bandwidth 

- [x] **Node granularity + scale (M4)** — one §3.1 paragraph defining what makes one concept node;
      one honest sentence that current graphs are unit-scale (14–28 concepts ≈ one chapter), course
      scale untested. Also state once in §6 whether the full 422-primitive Chinese graph was ever
      loaded/validated by this system or only the 12-primitive evaluation subgraph.

[WEN] let us do

DONE (v1.1.tex): added a "Node granularity" paragraph to §3.1 (Schema of 4 Elements) — defines a
concept node as one `defines` sentence + one `verifier` tag + one generated section, notes node
boundaries are a modeling choice (e.g. velocity could split into average/instantaneous), and states
the real range from the corrected Table B2: primitives 3–12, concepts 11–25 (superseding the review's
own "14–28" estimate, which was based on the pre-fix approximate counts) — all unit-scale, course-scale
untested, cross-referenced to the Limitations subsection (§7.4). Added a second paragraph to §6.2 (The
Reducibility Theorem Across Domains) stating plainly that the `chinese_characters` row evaluated in this
paper is the 12-primitive pilot subgraph, not the full 422-primitive ZiNets graph — the latter was
validated in the separate prior paper and has never been loaded into this system's `graph_lib`.
Also caught and fixed three section-number errors of my own (§3.6→§4.6 twice, a stray §3.1 self-reference
→§4.1, §7.3→§7.4) introduced while cross-referencing — worth double-checking any new cross-references
against the actual `\section`/`\subsection` count rather than assuming the numbering from an older draft.

- [x] **Table B2 exact counts** — replace "~20 / ~18 / ~16" with exact counts from the YAML files;
      reconcile B1 vs B2 (Chinese: 11 pictograms + 1 phonetic lender = 12 — say so; English: 5 roots
      vs 13 primitives).

[WEN] let us do

DONE (v1.1.tex, §2.3 excerpt + Appendix B full Table B2): replaced every approximate count with an
exact one parsed directly from each domain's `graph.yaml` (cross-checked against `concept-book/public/
domains/catalog.json`, which agreed everywhere except the two OpenStax chapters — catalog.json is stale
there, see below). Also corrected primitive counts that were off by one in six domains (quantum_physics,
cs_algorithms, cs_data_structures already right, sql, biology, medicine, molecular_biology all 5→4).
Added a footnote reconciling English Morphology's "13 primitives" (= 5 stem + 5 prefix + 3 suffix brick
*instances*) against Table B1's "5 roots" (the stem-class subset) and against the 3 schema-level
primitive categories — three different, all-correct numbers depending on what's being counted, now
disambiguated in one footnote instead of silently inconsistent.

Physics Ch1/Ch2 (Path B, §3.6) also had primitive/concept counts off by much more than rounding (ch1:
table said 4/~12, actual graph.yaml has 3/19; ch2: table said 4/~14, actual has 5/15) — fixed to exact
counts. Neither chapter's original "Capstone" name (Measurement & Error Analysis / Projectile Motion)
corresponded to any node in the ingested graph. [WEN] confirmed no capstone is expected for a single
ingested chapter — a chapter ends in a recap, not a payoff application — so the Capstone column for both
rows now reads "(chapter recap - no capstone)" rather than inventing or repurposing one. Also softened
ch2's payoff text from "1D/2D motion problems" to "one-dimensional motion problems" since only the 1D
kinematics chapter (Ch.2) is ingested — 2D motion is Ch.3, extracted per the new §3.6 bifurcation
paragraph but not yet published.

- [ ] **Figure 3** — decide: regenerate to show the `normal_modes` capstone (matching Table B2), or
      keep current orbital-mechanics render (caption now matches it).

[WEN] keep `orbital-mechanics`

- [x] **Figure 1** — fix overlapping "Lean Proving"/"Python for Science" labels; state the selection
      rule for which 10 of 20 domains are shown (or plot all 20).

[WEN] let us do

DONE (Figures/figure1-lego-payoff.py + v1.1.tex caption): the overlap was actually 3-way, not 2 —
Classical Mechanics, Python for Science, and Lean Proving all sit at the identical (4 primitives,
19 concepts) coordinate. [WEN] confirmed: merge all three into one annotation box rather than just the
two the review named. Rewrote the annotation logic to group domains by exact coordinate and join names
with " / " (generalizes to any future overlap, not just this one). Also updated the script's hardcoded
data to the corrected Table B2 counts (Calculus 20→14, Quantum Mechanics 5/18→4/16), regenerated
PNG/PDF/SVG, and rewrote the caption to state the actual selection rule: all 3 Morphological/Structural
domains (the smallest category) plus a representative sample of the other two categories, chosen for
legibility, not a numeric formula. Corrected the caption's stated concept-count range from the old
"14–28" to the actual data range "14–25" (25 = Linear Algebra, the max in this subset).

- [x] **References** — key survey entries by author not venue ("ACM Computing Surveys (2025)",
      "Khan Academy (2023)"); add publisher/URL to IMS LD; consider Novak (concept mapping),
      Madaan et al. Self-Refine (GENERATE→EVALUATE→REFINE lineage), an autoformalization cite for
      the Lean row; verify arXiv IDs resolve.

[WEN] let us do

DONE (v1.1.tex, §2.1 + §2.3(Related Work GENERATE/EVALUATE/REFINE) + §4.3 Verifier Dispatch + References):
- Re-keyed "ACM Computing Surveys (2025)" → real authors, found via web search: Bai, Y., Liu, Z., Guo, T.,
  Hou, M., & Xiao, K. (2025), ACM Computing Surveys 57(11) Art. 279 — updated both the reference entry and
  its in-text citation marker.
- Left "Khan Academy (2023)" as an organizational citation — Khanmigo has no single named academic author
  (Sal Khan is the public spokesperson, not a paper author), so this is the same convention as citing
  "OpenAI (2023)" for a product; flagging in case you'd rather handle it differently.
- Added publisher (IMS Global Learning Consortium, now 1EdTech) and URL (imsglobal.org/learningdesign) to
  the IMS LD entry.
- Added Novak & Gowin (1984, concept mapping) with a new paragraph in §2.1 positioning the concept-graph
  as a formalization of concept maps (checked, machine-executable, drives generation vs. a study aid).
- Added Madaan et al. Self-Refine (2023), cited at the existing GENERATE→EVALUATE→REFINE mention in
  Related Work as the lineage source for that pattern.
- Added Weng et al. autoformalization survey (2025), cited in §4.3 next to the Lean verifier row, tying
  it to the existing "Lean is the most refinement-hungry backend" observation in §7.
- Verified all 9 arXiv IDs in the paper resolve and match their cited titles (fetched each abstract page).
- Cross-checked every reference entry has a matching in-text citation (30/30) — no orphaned references.

**Cheap wins (convert limitations into results):**
- [x] **Path B bifurcation data point (M5)** — report whether the OpenStax-extracted graphs bifurcated
      into two radical classes *before* the human-editing pass; closest available independent evidence
      for the two-radical motif, already sitting in pipeline artifacts.

[WEN] let us do

DONE (v1.1.tex, §3.6 + forward-ref in §2.2): confirmed via diff that college_physics_ch1/ch2's
published graph.yaml is byte-identical to concept-book-press's raw extractor output — no human-editing
pass actually occurred for either. Extended the same zero-touch pipeline to Ch.3 (2D Kinematics) and
Ch.4 (Dynamics), not yet published: primitive sets are cleanly disjoint —
{reference_frame, position, time, vector, scalar, displacement, velocity, acceleration} (Ch.2-3) vs.
{force, mass, dynamics, gravitational_acceleration} (Ch.4) — mirroring the authored mechanics domain's
kinematics/dynamics split, but arising from OpenStax's own chapter boundaries and an LLM with no
exposure to the two-radical framework. Reported as a single data point, not a corpus-wide claim.

- [x] **Multilingual run** — one French or Chinese end-to-end book; converts a §7.5 bullet into a result.

[WEN] let us do for a small concept-book domain, choose English/Spanish or English/Chinese

DONE: generated the full English Morphology book (33 concepts + capstone) in Chinese via local
`ollama:gemma4`, zero cost — `SPL.py/cookbook/74_concept_book/generate_english_morphology_zh.sh`.
All 33 concept pages + book index verified as real Chinese prose (not just headers). Along the way,
found and fixed a real bug this task surfaced: `write_concept_html`/`build_book_index` in `tools.py`
built filenames from the concept/target name only, with no language suffix, so a same-domain run in a
different language silently overwrote the other language's HTML files (and, more seriously, the
underlying `content_cache.db` entry too, since the cache key doesn't include `@language` either — the
"regenerate English from cache" attempt returned Chinese text). Fixed the filename collision by adding
a `_{language}` suffix for every non-English language (English stays unsuffixed, so existing links don't
break) in both functions, and wired `@language` through from `build_concept_book.spl`. Restored the
original English HTML files from git (they're tracked, so recovery was clean) rather than trusting the
cache. **The content-cache-key bug itself (not scoped to language) is still open** — flagged to Wen,
not fixed, since it's a deeper change to the caching layer than this task called for. Also found in
passing: Chinese Characters already had an independent `zh` pass in the cache from an earlier run, predating
this task — a second, pre-existing multilingual data point alongside today's new one.

- [x] **Cost/time numbers** — approximate $ and minutes to generate one book; makes §1.3's
      "a concept-graph and a single command" concrete.

[WEN] let us do

DONE — see `internal-run-stats-2026-07-08.md`: one full book (English Morphology, 33 concepts + capstone,
local ollama:gemma4) took 34 LLM calls / ~11.1 min wall-clock / $0. Folded into §1.3 (v1.1.tex) right
after "A concept-graph and a single command should be enough."

**Follow-up fix (Wen's call, 2026-07-08):** the cache-key bug behind the filename collision was deeper
than filenames — `cache_get`/`cache_put` in `build_concept_book.spl` never passed `params_json`, so the
Layer-2 content cache key was built from `concept` name alone; regenerating in a different language
silently overwrote the same cache slot's *text*, not just the HTML file. Fixed: added
`CALL json_set("{}", "language", @language) INTO @_params_json` once per run, threaded into both
`cache_get(@concept, params_json=@_params_json)` and `cache_put(@concept, @section, params_json=@_params_json)`.
Verified live on `music_theory_graph.yaml`: same-language re-run → cache HIT (0 LLM calls); same concept,
different language → cache MISS with genuinely distinct generated content (English vs. French prose
confirmed by inspection). `domain`/`style`/`lvl` are not included in the params key yet — same
theoretical collision risk, out of scope for what was asked, noted for later.

- [ ] **Expert audit (venue-dependent)** — 3 raters × 5 sections × rubric on one or two domains;
      the single cheapest study that converts §7.4's biggest limitation into a result.

[WEN] let us do
I can review most STEM domains, and can cover English and Chinese

**Editorial decisions (Wen's call):**
- [x] Promote the primitive-budget constraint to the contributions list.

[WEN] mention it in discussion section, not a contribution item

DONE (v1.1.tex, §7.1): it was never actually listed in the 5-item Contributions list (§1.5), so there
was nothing to remove — added a new paragraph at the end of §7.1 (Curriculum Knowledge is Graph-Structured)
framing the primitive-budget constraint as a pacing/cognitive-load mechanism the graph enables, not a
verifier or a domain claim, and explicitly stating it's a design detail of the generation loop rather
than a standalone contribution — cross-referenced to §4.1 (where it's defined) and §6.4 (the pass-rate
evidence that it's doing its job, catching sections that front-load too much new material).

- [ ] Consider promoting `answer_on_demand.spl` (personal-tutor mode) from §7.3/Appendix A.2 into §4–5
      with one worked example.

[WEN] no, leave it to reader

- [ ] Length: 28 pp total vs AIED/LNCS ~12–15 — §1.3 (Origins) can halve; §5 (web app) can compress
      toward Appendix C.

[WEN] we have to fix this when close to submission deadline, but not now

- [x] Neutral radical labels: make state-class vs operation-class primary outside the Chinese domain,
      keep FORM/SOUND (形声) as the motivating instance.

[WEN] let us do

DONE (v1.1.tex): swapped state-class/operation-class to primary in §3.1 (schema definition of
first_radical_primitives), §3.2 (the general bifurcation motif sentence, plus an explicit note that
FORM/SOUND is the Chinese instance, not the general vocabulary), Table B1's column header (both the
§3.2 excerpt and the Appendix B full table), and the Calculus sample in Appendix D.4. Left FORM/SOUND
as-is in the three genuinely Chinese-specific spots: §1.3 (ZiNets origin story), the Chinese-characters
paragraph in §3.2, and the Appendix D.2 Chinese sample.

**What FORM/SOUND actually means, for context:** it comes from 形声字 (xíngshēngzì,
"form-sound characters"), one of the six traditional categories of Chinese character
formation (六书) and by far the most productive — it accounts for over 80% of all Chinese
characters. Every 形声字 has two components: 形 (xíng, "form/shape") is the *semantic*
radical — a pictogram that signals what category of meaning the character belongs to —
and 声 (shēng, "sound") is the *phonetic* radical — a component borrowed purely for its
pronunciation, contributing no meaning of its own. Classic example: 妈 (mā, "mother") =
女 (nǚ, "woman" — the FORM radical, tells you the character is about a female person) +
马 (mǎ, "horse" — the SOUND radical, borrowed only because it's pronounced "mǎ"; it says
nothing about horses). The ZiNets paper's reducibility theorem is that 马-as-phonetic-lender
is *not derivable* from the 11 semantic pictograms alone — FORM and SOUND are mutually
irreducible, and that irreducibility is what the paper's `graph_lib.reducible()` check
formalizes.

**Why we didn't keep FORM/SOUND as the general term:** the metaphor is exact for Chinese
writing (form vs. sound is literally what the two radicals encode) but doesn't read
naturally once you leave that domain. Calling `force` a "SOUND-class primitive" in the
Classical Mechanics section would be opaque — force isn't a sound, it's an operation
(something that acts on a state). So the paper now uses **state-class** (what things
*are* — position, mass, vectors, scalars, semantic pictograms) and **operation-class**
(what *acts on* them — force, derivatives, inner products, phonetic lenders) as the
domain-neutral vocabulary for the same structural motif, and keeps FORM/SOUND only where
we're literally talking about the Chinese writing system — the instance that inspired the
naming in the first place, per §3.2's footnote on why the term "radical" is borrowed from
Chinese character analysis (部首) to begin with.

**Follow-up (2026-07-08, Wen's call): §3.2 rewritten completely, not just relabeled.** Wen's
observation: physics' matter/interaction split is FORM-and-structure vs. SOUND-or-action, and the
same duality is normally called *form and function* in biology — this pattern recurs across many
domains, and since we're dealing with graph structure, the paper should explain *why* the
two-radical structure exists rather than just present it. Rewrote §3.2 (v1.1.tex) around that: the
opening now argues the split is rooted in the duality of description itself (any sufficiently rich
account of a system must say both what its parts *are* and what happens *between* them), not a
convenience invented for this paper. Leads with biology's *form and function* (the most familiar,
oldest-named instance) rather than a Chinese-writing-first framing, then gives a bulleted
cross-domain vocabulary correspondence, refined over a few rounds of Wen's feedback to the final set:

- Biology and medicine: form and function (anatomy and physiology)
- Physics: matter states and interaction — "matter" explicitly qualified as *matter states*
  (configuration), not substance in general, per Wen's correction
- Chemistry: element identity and bonding behavior
- Computer science: state and operation, with property/method noted as the OOP name for the same split
- Graph theory: entity and relation
- Chinese writing: 形/声 (form/sound) — now lowercase throughout the paper, not capitalized, per
  Wen's instruction that capitalizing FORM/SOUND reads as if we're coining new terminology rather
  than naming something that already recurs everywhere
- Mathematics (set theory): a set and its operators — flagged by Wen as the most fundamental of all
  the pairings in terms of formal representation, since an algebraic structure just *is* a carrier
  set plus its operators; framed as the case every other pairing above specializes

State-class/operation-class is then introduced as *this paper's* schema-level term specifically
because it's the least domain-loaded of all these equally-valid pairings — not an improvement on
form/function or 形/声, just the most neutral label for the same shape. Compiled clean at every step
(30 pages final).

**Follow-up (2026-07-08, cont'd): reorder + graph-first bookend + Novak & Gowin softening.**

- Reordered the §3.2 domain list per Wen's call: math (set theory) first, then physics, chemistry,
  biology/medicine, computer science, Chinese writing, English morphology, and finally **graph
  theory last** — a deliberate bookend (starts with math, ends with math/graph theory) since graph
  theory is literally the representation this paper's own concept-graph schema uses. The setup
  sentence and the graph-theory bullet itself both now name this connection explicitly.
- Wen's insight that motivated this: a concept-graph is relatable to *any* subject-domain expert
  because authoring one only asks for primitives, derived concepts, their composition relations, and
  a target concept — relationships every expert already reasons about; expressing that as a
  graph/network is simply the most efficient re-representation of domain knowledge for teaching.
- Wen flagged that **Novak & Gowin (1984), "Learning How to Learn,"** likely already covers this
  territory — concept mapping. Checked: confirmed via web search. Novak & Gowin's concept maps
  represent a domain as concepts linked by labeled propositions, hierarchical (general → specific),
  built on Ausubel's theory of meaningful learning — a graph-of-concepts-and-relations, four decades
  before this paper. The paper already credited this correctly in §2.1 (added in an earlier session),
  distinguishing ConceptBook's contribution (machine-executable specification + generation) from
  Novak's graph-for-study-and-planning. Per Wen's request, added it **earlier** too: a new paragraph
  in §1.3 (Introduction) right after the ZiNets/reductionism origin story, explicitly softening the
  originality claim — Wen arrived at the graph-of-knowledge idea independently (via physics
  reductionism, not educational theory) and had not encountered Novak & Gowin before; the paper now
  says this directly and reframes the contribution as narrower and dated to this moment: *"a concrete
  implementation for the era of large language models... not the graph-of-concepts idea itself."*
  Also added a third tie-in in §7.3 (Discussion), linking §3.2's cross-domain duality argument to the
  Novak & Gowin credit and the re-representation claim into one thread instead of three separate ones.
- **Book, for Wen to read:** [Learning How to Learn — Internet Archive](https://archive.org/details/learninghowtolea00jose)
  (free to borrow/stream, controlled digital lending — legitimate). Cambridge Core has the official
  publisher page too: https://www.cambridge.org/core/books/learning-how-to-learn/D4E082D454735D8CC7FEDADFA25A3B99
  (avoid Scribd copies found in the same search — sourced from Libgen, not a legitimate copy).
- Also fixed a stale claim while reviewing the full draft: §7.5 Future Work's "Multi-language books"
  bullet still said "only English books have been generated to date... not yet exercised end-to-end,"
  which the English-Morphology→Chinese run earlier this session had already disproven. Updated to
  report what actually happened and honestly scope what's still open (more languages/domains, human
  quality review of non-English output).
- Compiled clean at every step; 31 pages final.
