# Hey there, I'm Tunjay Akbarli.

I'm the Founder and Chief Executive Officer of **Omnira** ([@theomnira]) and an independent researcher and systems architect based in Meggen, Switzerland. My work sits where three areas meet: autonomous business infrastructure, compiler and language design, and hybrid quantum-classical control. The thread running through all of it is the question of how to make complex machines legible, verifiable, and changeable without losing their power.

---

## 🚀 What I'm Building

* **Omnira:** An autonomous operating ecosystem for running a business. Products, teams, payments, and infrastructure are managed as continuously running control loops rather than disconnected tools. Each loop observes state, reasons about it, acts, and checks the outcome. The goal is to replace the manual glue between dashboards, spreadsheets, and vendor consoles with a single command center that operates around the clock under strict access and audit controls.

* **Codira:** An ahead-of-time compiled, statically typed systems language built on three commitments:
  * **Proof before execution:** The compiler discharges verification conditions it can prove using the Z3 satisfiability-modulo-theories solver. Contracts such as preconditions, postconditions, and loop invariants are checked at compile time, and counterexamples become executable regression tests.
  * **Live evolution:** A running program can change its code and data layouts without dropping state. Fields can be retyped with saturating conversions, bindings can be renamed through migration annotations, and the whole runtime heap can be snapshotted and restored.
  * **One source, many machines:** The same program can target CPUs, GPUs through NVIDIA PTX, WebAssembly, and quantum processors through QIR and OpenQASM 2.0.

* **Neurosymbolic Execution and Understanding System:** An architecture that targets the *frozen weight problem*. Most AI systems store facts, reasoning habits, and stylistic tendencies in one opaque parameter matrix, which makes them hard to inspect, correct, audit, or extend. The system separates these concerns into distinct layers with formally specified interfaces.

---

## 🧠 Neurosymbolic Execution and Understanding System

### The Core Thesis

A trained neural network entangles three different things:

1. **Knowledge:** facts about the world, such as which drug interacts with which, or what a statute says.
2. **Reasoning:** the procedures for combining facts into conclusions.
3. **Execution:** the statistical machinery that parses language and produces fluent output.

When these share the same weights, a single fact cannot be located, changed, or retracted cleanly. Its provenance, confidence, and expiry date are discarded during training. The architecture resolves this by moving knowledge and reasoning out of the weights and leaving the neural network responsible for language only. The network becomes a translator between human language and a typed, inspectable symbolic core.

### Layer 1: Neural Execution

**Role:** Parsing natural language into typed structures and rendering results back into natural language. This layer is stateless with respect to world facts. It has no authority to decide what is true.

* **Constrained parsing:** Output is forced to conform to a schema through constrained decoding. Typed objects are validated with Pydantic and the Instructor library before they reach the symbolic core. Malformed input is rejected at the boundary, not repaired silently downstream.
* **Grounded rendering:** Symbolic answers and proof traces are verbalized under constraints that forbid introducing any claim absent from the trace. Every sentence of the output should trace back to a derivation step, so hallucination becomes a detectable failure rather than an invisible one.

### Layer 2: The Symbolic Core (the Mod Surface)

**Role:** The reasoning engine and the primary place where domain knowledge is authored. This is the architectural heart of the system.

* **Mod files:** Domain knowledge (medical, legal, financial) is written as version-controlled Prolog modules. Each rule carries mandatory provenance citations, unit tests written in PlUnit, and a conflict manifest declaring which other modules it may contradict and how precedence resolves the conflict.
* **Explicit goal systems and skill modules:** Goals, subgoals, and reusable procedures are first-class objects, so the system can explain why it pursued a line of reasoning, not only what it concluded.
* **Persona overlays:** Tone, audience, and presentation policies are applied as separate overlays. They can alter how a conclusion is communicated but cannot alter its derivation.
* **Static consistency checking:** Before a mod loads, the checker verifies type correctness, the restrictions of the Datalog subset, dependency consistency, and the satisfiability of safety invariants.

**Confidence algebra:** Uncertainty propagates through inference chains under explicit independence assumptions:

* **Conjunction** (all premises hold): $c_1 \otimes c_2 = c_1 \cdot c_2$
* **Disjunction** (at least one independent source holds): $c_1 \oplus c_2 = 1 - (1-c_1)(1-c_2)$
* **Negation:** $\neg c = 1 - c$
* **Temporal decay** (confidence erodes as evidence ages): $\delta(F,t) = c_F \cdot e^{-\lambda_F(t - t_{\text{in}})}$, where each fact type $F$ has its own decay rate $\lambda_F$ and $t_{\text{in}}$ is the ingestion time.

The independence assumption is stated, not hidden. Where sources are correlated, the mod must declare it, and the confidence calculation must account for the dependence.

### Layer 3: The Knowledge Layer

**Role:** Storage of world facts as an external, queryable, versioned graph, with no fact embedded in model weights.

* **Typed facts:** Each fact is a first-class object carrying a confidence score, a source URL, an ingestion timestamp, and an expiry horizon. Metadata is preserved end to end, which weight-based approaches cannot do.
* **Semantic versioning of the knowledge graph:** The version number encodes the kind of change. A **MAJOR** increment marks a retraction or breaking change that invalidates dependent derivations. A **MINOR** increment marks additions. A **PATCH** increment marks metadata or confidence updates. Downstream consumers can therefore decide mechanically whether they need to re-derive.
* **Tiered storage:** Hot memory holds cached query results. Warm storage holds active domain knowledge in indexed form. Cold storage uses columnar Apache Parquet files for bulk data and RDF graphs for linked, standards-compatible exchange.

### Layer 4: Weight Editing Layer

**Role:** Some knowledge does not reduce cleanly to symbolic rules. Stylistic preferences, syntactic habits, and fine-grained associations are examples. This layer handles those cases by treating neural weights as software memory that can be patched and rolled back.

* **Editing techniques:** ROME performs rank-one edits to a single MLP layer's key-value association, located via causal tracing. MEMIT extends the approach to batched edits across multiple layers. WISE keeps edits in a side memory, which avoids overwriting the base model. Each update has the form $\theta' = \theta + \Delta_\theta$ with $\text{rank}(\Delta_\theta) \le r$, so the change is low-rank and localized.
* **Version control for weights:** Each patch is versioned like source code, with diff, commit, test, and rollback operations. A patch ships with a regression suite that verifies both the intended change and the absence of collateral damage elsewhere.
* **Precedence:** Symbolic rules govern all factual queries. Weight patches serve as fallback behavior for cases symbolic rules cannot express, and a weight patch can never override a symbolic conclusion.

---

## 📐 Theoretical Foundations

The Neurosymbolic Execution and Understanding System v2.0 specification establishes the limits that justify this design:

* **Weight opacity (Theorem 3.1):** Recovering a specific fact from trained parameters is not computable in polynomial time in general, and no closed form exists. Interpretability methods can locate fragments of a fact, but they do not provide a reliable inverse mapping. The fact has no stable address inside the weights.
* **Metadata destruction (Corollary 3.1a):** Under standard loss functions, provenance, confidence, timestamps, and expiry fields contribute zero gradient signal. Training therefore discards them irreversibly. Better training cannot recover them, because the objective never asked for them.
* **Auditability gap (Theorem 3.2):** Dense networks composed as $f_\theta = g_L \circ \dots \circ g_1$ cannot support claim-to-source attribution. Superposition, polysemanticity, and distributed representations mean no single component maps to a single claim. Audit requires an architecture in which claims are discrete objects from the start.
* **Update complexity separation (Theorem 3.3):** Updating a weight-based model costs $\Omega(|\theta| \cdot |\mathcal{D}'| \cdot I)$, meaning parameter count times new data times training iterations. Inserting into the knowledge graph costs $O(|\mathcal{K}'| \log |\mathcal{K}|)$, an indexed insertion. The gap is roughly nine orders of magnitude at practical scales, so knowledge that changes frequently should not live in weights.
* **Audit completeness (Theorem 5.4):** The system satisfies architectural auditability. Proof traces have polynomial length, derivations are sound under SLD resolution (the standard procedure for Horn-clause logic programs), every step is recorded, and a third party can verify the trace independently.
* **Mod consistency decidability (Theorem 6.1):** For mod sets equivalent to Datalog, consistency checking is decidable in $O(|\mathcal{H}_M|^2)$, where $\mathcal{H}_M$ is the mod's rule set. Safety checks are therefore exhaustive, not sampled.
* **Derivation well-foundedness (Theorem 8.1):** Audit derivation graphs are strictly acyclic under tabled resolution, which memoizes intermediate results. Circular reasoning is structurally impossible, and infinite loops are prevented.

---

## 🌌 Quantum Computing & Materials Science

* **Quantum Calibration and Observable Processing (QCOP):** A framework I invented for closed-loop hybrid systems, addressing hardware error mitigation and synchronization between quantum and classical state estimates. The central idea is that calibration is itself a control problem, with its own budget and failure modes.
* **Hybrid materials design:** Implantable chemical sensor design is formulated as a constrained decision problem over 1,204 scaffold-analyte pairs. A working sensor must switch at temperatures that straddle body temperature (310 K) with a 20 K safety margin, so the search is driven by feasibility constraints rather than raw performance alone.
* **Exact subspace mapping for iron(II):** I implemented a five-orbital Kanamori model of $\text{Fe}^{2+}$. In the seniority-zero sector, the low-spin ground state maps directly onto 5 qubits. Because the mapping avoids Jordan-Wigner strings, the operators stay local and circuit depth is reduced. It was evaluated across 32,480 simulated device evaluations, giving complete observability of the sector.
* **Device-calibrated surrogates:** Cheap classical estimators are biased. Noisy quantum-processor energies are used to recalibrate them. With 150 calibration evaluations, surrogate error falls from 178 K to 21 K, and decision accuracy reaches 99.8%. A small amount of expensive, noisy ground truth corrects a large amount of cheap, systematically wrong estimation.
* **Calibration error law:** The surrogate's squared error follows $\sigma_{\text{surr}}^2 \approx b_{\text{arm}}^2 + s_{\text{arm}}^2 / n_{\text{cal}}$, where $b$ is systematic bias, $s$ per-evaluation noise, and $n_{\text{cal}}$ the number of calibration evaluations. The bias term does not shrink with more samples. The derived rule is to calibrate only while model error remains below single-evaluation error. Controlled-roughness ablations confirm the law at a median relative error of 1.0%.
* **Noise mitigation stack:** I evaluated mitigation across five contemporary quantum platforms, combining tensored readout-error correction with unitary zero-noise extrapolation. I also introduced zero-parameter twin reference-state circuits, which cancel systematic errors without fitted parameters and reduce variance while removing the bias floor.
* **Adaptive architectures and hardware artifacts:** Recurrent-depth decoders allocate computation dynamically per instance. Generative reinforcement-learning proposers reached a 100% top-hit rate on the benchmark. The output includes 318 hardware-ready OpenQASM 2.0 circuits, each shipped with a complete Hamiltonian manifest so results can be reproduced and checked.

## ⚛️ Quantum Systems Architecture & Orchestration

* **Adaptive Quantum Orchestration with Recurrent-Depth (AQORD):** A telemetry-driven control plane for hybrid quantum-classical computing. It continuously trades compilation depth against execution direction, subject to two walls: the hardware's coherence window and the classical energy budget. Across 107,520 physical trials on the Rigetti Cepheus-1 108-qubit processor, first-pass compile success rose to 97.3%, and gate depth fell by 45%.
* **Loss decomposition theorem:** The performance loss of any hybrid orchestrator decomposes into three terms: an oracle term (the best achievable with perfect information), an observability term (the cost of imperfect telemetry), and a learning term (the cost of not yet knowing the optimal policy). Exact offline replay over 98,304 trials showed that telemetry quality, not orchestrator sophistication, is the binding constraint. This is a result about where to invest engineering effort.
* **Recurrent-depth mixture-of-experts compiler:** A shared-weight recurrent-depth transformer that uses adaptive early exit and specialized experts to trade compiler compute against instance difficulty. It has provable geometric contraction of hidden states and closed-form lower bounds on compile success, so the quality guarantee is analytical, not only empirical.
* **Bidirectional quantum-classical routing (BQCR):** Switching the direction of support between quantum and classical resources is formalized as an adversarial bandit problem over a drifting processor. Updates use EXP3-IX, an exploration-based bandit algorithm with implicit exploration. The analysis yields tight cumulative regret bounds, so the total loss from suboptimal routing grows sublinearly.
* **Hardware-facing subsystems:**
  * The **Predictive Noise-Topology Mapper** tracks each chiplet's noise state with a Kalman filter, forecasting drift before it degrades circuits.
  * The **Autonomous Pulse-Level Scheduler** re-optimizes pulse shapes for single-qubit DRAG corrections and two-qubit controlled-Z gates as calibration drifts.
  * The **Hybrid Execution Graph** uses an entropy measure to partition each computation between tensor-network simulation, efficient for weakly entangled parts, and the quantum processor, needed for strongly entangled parts.

## 📐 Control Theory & Learning Under Drift

* **Observability limit:** A telemetry-conditioned hybrid policy is bounded by the Bayes error of its telemetry channel, the irreducible error of any classifier inferring hardware state from the available signals. The bound is $\min(a-c, c)P_e \le L_{\text{obs}} \le \max(a-c, c)P_e$, where $P_e$ is the Bayes error and $a$, $c$ are cost parameters. Sublinear regret against an oracle is possible only if this error vanishes over time.
* **Contraction of recurrent-depth states (Theorem 1):** Under a Banach contraction with Lipschitz constant $L_\phi \le 0.845$, the hidden states converge to a unique fixed point. Residuals across iterations therefore bound the structural compile success rate, $\text{CSR}(H) \ge 1 - \delta_0 L_\phi^{H-1}$, where $H$ is the recurrence depth.
* **Non-stationary regret (Theorems 3 & 4):** Routing regret remains bounded when the environment is ergodic Markovian and the calibration memory kernel is summable under $\psi$-mixing. The bound is $R_T \le (1+\tau_{\text{mix}})\sqrt{T \ln 2 / 2}$, where $\tau_{\text{mix}}$ is the environment's mixing time.
* **Input-to-state stability (Theorem 5):** Lyapunov-style arguments show that closed-loop quantum-classical iterations converge geometrically into a bounded ball whose radius scales with telemetry drift and shot noise. The loop is stable under realistic disturbances, not only the idealized noise-free case.
* **Optimal recalibration cadence (Theorem 10):** The workload-optimal recalibration interval scales as $T^{-1/4}$ over a session horizon $T$. It balances the regret saved by recalibrating more often against the recurring cost of each recalibration.

## 🧠 Cybernetics & Neuromorphic Frameworks

* **Artificial peripheral thalamus:** A fourteen-module, multi-scale cybernetic framework for the stroke continuum, closed-loop sensory feedback, and neuromorphic relay devices, published in the *Journal of Neural Engineering*. The thalamus is the brain's relay and gating structure; the framework models an engineered analog of that role, relaying sensory information back into a recovering system.
* **Shared state vector:** Fourteen modules across four physiological layers (pre-stroke reserve, ictal cascade, systemic response, and recovery), plus social/access and device layers, are coupled through six core variables: the collateral index, oxygen-extraction reserve, inflammatory set-point, surviving corticospinal capacity, afferent signal-to-noise ratio, and plasticity gain. All modules read and write this single vector, so cross-layer effects are explicit.
* **Penumbral threshold identity:** I derived an identity for the critical blood flow below which penumbral tissue (the at-risk zone around an infarct) can no longer meet its metabolic demand:

$$\text{CBF}_{\text{crit}} = \frac{\text{oxygen demand}}{\text{arterial oxygen content} \cdot \text{maximum oxygen extraction fraction}} = \text{CBF}_{\text{normal}} \cdot \frac{\text{OEF}_{\text{normal}}}{\text{OEF}_{\text{max}}}$$

  The identity uses no fitted parameters and matches the verified penumbral ceiling to within 4.5%, at 22.98 against 22 mL per 100 g per minute.
* **Sensitive-period plasticity:** Training responsiveness follows an inverted-U trajectory, $w(t) \propto (1 - e^{-t/\tau_{\text{open}}}) e^{-t/\tau_{\text{close}}}$, with the window opening at a time constant of 2.4 weeks and closing at 57 weeks, peaking at week 7.7.
* **Device specifications:** The closed-loop latency envelope is 30 ms or less, imperceptible to the user. At 10 ms, the system retains 94% of its learning rate. Event-driven throughput is 500 to 646 bits per second across six channels. The carrier must deliver sustained selective enrichment of 2.9x to 7x in penumbral tissue. These targets rest on 85 literature comparisons, 50 of them out-of-sample, and 10 pre-registered falsifiable predictions.

---

## 🛡️ Safety, Verification & Threat Modeling

* **Safety invariant checking:** Safety conditions are encoded as constraints and verified with Z3 before any mod loads. For example, "safe" and "contraindicated" can never both be derived for the same patient-symptom pair. A violation is a load-time error, not a runtime surprise.
* **Adversarial threat model for community mods:** Community contributions are treated as hostile input. The threat model follows the STRIDE categories (spoofing, tampering, repudiation, information disclosure, denial of service, and elevation of privilege). Each maps to a defense: strict type checking against spoofing and tampering, the restricted Datalog subset to bound computation, dependency manifests against supply-chain injection, and mandatory unit tests as the acceptance gate.
* **Frontier alignment research:** I originated the formal framework of *artificial depatterning*, which analyzes how advanced AI systems can shift behavioral policies between monitored evaluation and unmonitored deployment. It targets the gap between measured and deployed behavior, one of the central unresolved problems in evaluation.

---

## 📚 Publications & Leadership

* **Elected Member:** The New York Academy of Sciences.
* **Peer-reviewed research:** Papers and reviews in the *Journal of Neural Engineering*, *Royal Society Open Science*, and *Quantum Science and Technology*, plus archived technical documents in the National Encyclopedia of Azerbaijan, the Azerbaijan National Academy of Sciences, and repositories in Izmir, Florida, and Louisiana.
* **Global reach:** Technical presentations delivered to delegations from 59 countries, reaching over 1,000 scientists, fellows, and engineers across six countries.

---

## 🛠️ Stack & Primitives

* **Languages & runtimes:** Custom compiler toolchains (Codira, Eidos, LLVM, Cranelift), kernels, bootloaders, security systems, runtimes, refinement-verified and mission-critical systems
* **Logic & verification:** Prolog for mods, Datalog-subset constraints, Z3 solver, PlUnit, Pydantic, Instructor
* **Learning & editing:** Recurrent-depth transformers, sparse mixture-of-experts, rank-one and batched model editing (ROME, MEMIT), side-memory editing (WISE)
* **Infrastructure:** V8 isolate sandboxes, Docker orchestration, zero-trust edge execution, a custom SMT-aware linker forked from LLD (`Funis`)
* **Data & knowledge:** Local supercomputing environments, local databases, Parquet and RDF knowledge stores, versioned fact graphs

⚡ *Simplifying the complex, just to flow.*
