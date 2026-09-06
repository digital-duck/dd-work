# SPL Solver × Recipe Reference Table

Consolidated from `solver-research-plan.md` §3.1–§3.3.  
Recipe status from **A.3 Recipe Index** (authoritative). Rows marked _planned_ have no corresponding entry in A.3.  
GitHub base: `https://github.com/digital-duck/SPL.py/tree/main/cookbook/`

---

| ID | Solver | Problem class | Recipe ID | ASSERT predicate | Real-world domains | Ref Links |
|:---:|---|---|:---:|---|---|---|
| S001 | **PuLP + CBC** | LP / MIP — linear objectives, linear constraints | r78, r100, r101, r109 | `status == "Optimal"` | Production planning, staffing, blending, supply sourcing, sustainability, synthetic catalog | [PuLP](https://coin-or.github.io/pulp/) · [CBC](https://github.com/coin-or/Cbc) · [COIN-OR](https://www.coin-or.org/) |
| S002 | **Google OR-Tools** (CP routing) | NP-hard TSP / VRP — vehicle routing | r87 | feasible route found | Delivery routing, technician dispatch, last-mile logistics | [OR-Tools](https://developers.google.com/optimization) · [GitHub](https://github.com/google/or-tools) |
| S003 | **python-constraint** | CSP — constraint satisfaction over finite domains | r82 | `status == "Unique"` | Shift exclusions, SKU incompatibility, eligibility rules | [docs](https://python-constraint.github.io/python-constraint/) · [GitHub](https://github.com/python-constraint/python-constraint) |
| S004 | **networkx** | Graph algorithms — shortest path, bipartite matching | r81 | ground-truth match + round-trip | Org-chart reachability, dependency graphs, supply-chain topology | [networkx.org](https://networkx.org/) · [docs](https://networkx.org/documentation/stable/) |
| S005 | **pint** (UnitRegistry) | Dimensional analysis — unit safety | r83 | no `DimensionalityError` raised | Engineering calculators, BOM calculations, multi-unit pipelines | [pint](https://pint.readthedocs.io/) · [GitHub](https://github.com/hgrecco/pint) |
| S006 | **sqlite3** (stdlib) | Text-to-SQL — NL → query → verified answer | r84, r94 | row-diff vs ground truth | Chat-with-database, NL reporting, data governance | [Python docs](https://docs.python.org/3/library/sqlite3.html) · [SQLite](https://www.sqlite.org/) |
| S007 | **decimal** (stdlib) | Exact financial arithmetic | r86 | cent-level tolerance | Invoicing, loan amortization, customer-facing dollar figures | [Python docs](https://docs.python.org/3/library/decimal.html) · [IBM spec](https://speleotrove.com/decimal/) |
| S008 | **pytest + Hypothesis** | Code-from-spec correctness; property-based falsification | r79, r85 | all tests pass / no counter-example found | LLM-written scripts, data transforms, property-based testing | [pytest](https://docs.pytest.org/) · [Hypothesis](https://hypothesis.readthedocs.io/) |
| S009 | **NumPy + recomputation** | Physics / materials — conservation law recomputation | r90, r92 | 1% tolerance on recomputed invariant | Engineering simulations, scientific computing, crystal density | [numpy.org](https://numpy.org/) · [docs](https://numpy.org/doc/stable/) |
| S010 | **SymPy** | Symbolic math — algebra, calculus, ODEs, Taylor series | r67, r77 | `simplify(result − expected) == 0` | STEM education, scientific computing, equation solvers, CAS | [sympy.org](https://www.sympy.org/) · [docs](https://docs.sympy.org/) · [GitHub](https://github.com/sympy/sympy) |
| S011 | **SageMath** | Advanced math — Laplace transforms, ODEs, infinite sums, Galois groups | r75, r77 | symbolic identity or round-trip re-transform | Research computing, number theory, algebraic topology | [sagemath.org](https://www.sagemath.org/) · [docs](https://doc.sagemath.org/) · [GitHub](https://github.com/sagemath/sage) |
| S012 | **Google OR-Tools CP-SAT** | Job-shop / shift scheduling — distinct from routing | r98 | makespan minimized | Machine scheduling, shift optimization | [CP-SAT guide](https://developers.google.com/optimization/cp) · [OR-Tools](https://developers.google.com/optimization) |
| S013 | **cvxpy** | Convex optimization / portfolio — efficient frontier, multi-obj scalarization | r99 | `problem.status == "optimal"`; sweep `target_return` to trace efficient frontier | Portfolio optimization, finance, energy dispatch, robust control | [cvxpy.org](https://www.cvxpy.org/) · [GitHub](https://github.com/cvxpy/cvxpy) · [paper](https://www.jmlr.org/papers/v17/15-408.html) |
| S014 | **Z3** (Microsoft SMT) | Theorem prover — eligibility rules, pricing tiers, compliance logic | r102 | rules satisfiable; no contradiction found | Eligibility rules, pricing tiers, compliance logic, contract verification | [Z3 guide](https://microsoft.github.io/z3guide/) · [GitHub](https://github.com/Z3Prover/z3) · [Python API](https://z3prover.github.io/api/html/namespacez3py.html) · [Programming Z3](https://z3prover.github.io/papers/programmingz3.html) |
| S015 | **Great Expectations (GX)** | Data quality framework — expectation suite validation | r103 | all expectations pass | Data governance, pipeline quality gates, data contracts | [Great Expectations](https://greatexpectations.io/) · [docs](https://docs.greatexpectations.io/) |
| S016 | **pandera** | Dataset validation — statistically typed dataframes | r104 | schema validates (no `SchemaError`) | Data processing pipelines, typed dataframes, ML feature stores | [pandera](https://pandera.readthedocs.io/) · [GitHub](https://github.com/unionai-oss/pandera) |
| S017 | **SWI-Prolog** | Logic programming — contract / compliance satisfiability | r105 | query succeeds / clause satisfiable | Insurance, legal, contract logic, business rule engines | [swi-prolog.org](https://www.swi-prolog.org/) · [docs](https://www.swi-prolog.org/pldoc/index.html) |
| S018 | **Shapely / PostGIS** | Geospatial — polygon intersection, service-area coverage | r106 | intersection / containment predicate verified | Zoning constraints, service territory verification, geocomputations | [Shapely](https://shapely.readthedocs.io/) · [PostGIS](https://postgis.net/) · [GitHub](https://github.com/shapely/shapely) |
| S019 | **pymoo** | Multi-objective evolutionary — NSGA-II, MOEA/D, reference-point methods | r107 | solution not dominated; hypervolume contribution > 0 | Supply chain (cost / service / carbon), workforce scheduling, operations | [pymoo.org](https://pymoo.org/) · [GitHub](https://github.com/anyoptimization/pymoo) · [paper](https://ieeexplore.ieee.org/document/9078759) |
| S020 | **platypus** | Multi-objective evolutionary — lighter alternative to pymoo | _planned_ | Pareto dominance check | Same domains as S019; preferred when pymoo dependency weight is a concern | [GitHub](https://github.com/Project-Platypus/Platypus) · [docs](https://platypus.readthedocs.io/) |
| S021 | **Pyomo + GLPK** (Extensive Form) | Two-stage stochastic LP/IP — decide now → observe scenario → recourse | r117 | `status == "optimal"` (Pyomo certified) | Inventory under demand uncertainty, energy dispatch, supply chain under disruption risk | [pyomo.org](http://www.pyomo.org/) · [GitHub](https://github.com/Pyomo/pyomo) · [GLPK](https://www.gnu.org/software/glpk/) |
| S022 | **python-mip** | MILP with scenario-based robust constraints | r108 | `model.status == OptimizationStatus.OPTIMAL` | Supply chain resilience, project scheduling with risk budgets | [python-mip.com](https://www.python-mip.com/) · [GitHub](https://github.com/coin-or/python-mip) |
| S023 | **nashpy** | Nash equilibria — 2-player normal-form games | r110 | no profitable unilateral deviation (support enumeration) | Pricing strategy, procurement auctions, market entry | [nashpy](https://nashpy.readthedocs.io/) · [GitHub](https://github.com/drvinceknight/Nashpy) |
| S024 | **pygambit** | N-player normal-form games — dominant strategy, pure Nash enumeration | r115 | no profitable unilateral deviation for any player | Multi-firm pricing triopoly, platform competition, 3-player auctions | [gambit-project.org](http://gambit-project.org/) · [GitHub](https://github.com/gambitproject/gambit) · [pygambit](https://gambit.readthedocs.io/en/latest/pygambit.html) |
| S025 | **OpenSpiel + CFR** | Imperfect-information games — hidden cards, sealed bids, private signals | r116 | exploitability < 0.05 (distance from Nash equilibrium) | Poker strategy, sealed auctions, multi-round negotiation with private info | [GitHub](https://github.com/google-deepmind/open_spiel) · [paper](https://arxiv.org/abs/1908.09453) · [CFR](https://poker.cs.ualberta.ca/publications/NIPS07-cfr.pdf) |
| S026 | **optuna** (TPE) | Black-box optimization — when objective has no analytical form | r112 | best trial value + convergence plateau | Pricing model tuning, campaign budget allocation, hyperparameter strategy | [optuna.org](https://optuna.org/) · [GitHub](https://github.com/optuna/optuna) |
| S027 | **scikit-optimize** (GP+EI) | Bayesian optimization — adaptive sequential experimentation | r113 | posterior mean improvement > threshold | A/B strategy testing, CRO, product feature tradeoffs | [scikit-optimize](https://scikit-optimize.github.io/) · [GitHub](https://github.com/scikit-optimize/scikit-optimize) |
| S028 | **scipy.optimize** | Constrained nonlinear — SLSQP, Nelder-Mead, differential_evolution | r114 | `result.success == True`; recompute objective independently | Nonlinear pricing, logistics with quadratic costs, demand curve fitting | [scipy docs](https://docs.scipy.org/doc/scipy/reference/optimize.html) · [GitHub](https://github.com/scipy/scipy) |

---

## Recipe → Solver reverse index

| Recipe | Description | Solver (ID) | Status |
|:---:|---|---|:---:|
| r67 | sympy_basics | SymPy (S010) | ✓ |
| r75 | sage_math | SageMath (S011) | ✓ |
| r76 | lean_proof | Lean 4 (no Sxxx — formal proof kernel) | ✓ |
| r77 | neurosymbolic | SymPy + SageMath (S010, S011) | ✓ |
| r78 | pulp_lp | PuLP + CBC (S001) | ✓ |
| r79 | pytest_codegen | pytest (S008) | ✓ |
| r81 | networkx_graph | networkx (S004) | ✓ |
| r82 | constraint_csp | python-constraint (S003) | ✓ |
| r83 | pint_units | pint (S005) | ✓ |
| r84 | text2sql | sqlite3 (S006) | ✓ |
| r85 | hypothesis_test | Hypothesis / pytest (S008) | ✓ |
| r86 | decimal_finance | decimal (S007) | ✓ |
| r87 | ortools_vrp | OR-Tools CP routing (S002) | ✓ |
| r90 | numpy_physics | NumPy (S009) | ✓ |
| r92 | numpy_materials | NumPy (S009) | ✓ |
| r94 | sql_quality | sqlite3 (S006) | ✓ |
| r98 | ortools_jobshop | OR-Tools CP-SAT (S012) | ✓ |
| r99 | cvxpy_portfolio | cvxpy (S013) | ✓ |
| r100 | supply_sourcing | PuLP + CBC (S001) | ✓ |
| r101 | production_sustainability | PuLP + CBC (S001) | ✓ |
| r102 | z3_compliance | Z3 (S014) | ✓ |
| r103 | gx_dataquality | Great Expectations (S015) | ✓ |
| r104 | pandera_validate | pandera (S016) | ✓ |
| r105 | prolog_contract | SWI-Prolog (S017) | ✓ |
| r106 | shapely_geozone | Shapely (S018) | ✓ |
| r107 | workforce_3obj | pymoo NSGA-II (S019) | ✓ |
| r108 | robust_milp | python-mip (S022) | ✓ |
| r109 | synthetic_problems | PuLP + CBC (S001) | ✓ |
| r110 | nash_game_theory | nashpy (S023) | ✓ |
| r111 | stackelberg_game | pure-Python backward induction (no Sxxx) | ✓ |
| r112 | optuna_blackbox | optuna (S026) | ✓ |
| r113 | bayesian_opt | scikit-optimize (S027) | ✓ |
| r114 | scipy_nonlinear | scipy.optimize (S028) | ✓ |
| r115 | gambit_3player | pygambit / brute-force (S024) | ✓ |
| r116 | openspiel_cfr | OpenSpiel CFR (S025) | ✓ |
| r117 | pyomo_stochastic | Pyomo + GLPK Extensive Form (S021) | ✓ |
