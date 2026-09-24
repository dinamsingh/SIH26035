# PART 1 — COMPLETE SOFTWARE REQUIREMENTS SPECIFICATION (SRS)

## 1. System Vision
**System Name:** Automated NAWI Compliance & Reporting System (ANCRS)
**The System is a** secure, web-based regulatory compliance platform designed to digitize the physical testing workflow of Non-Automatic Weighing Instruments (NAWIs). It allows laboratory technicians to input raw test observations, mathematically evaluates these observations against version-controlled legal metrology rules (based on OIML R 76), handles exact decimal precision calculations including Maximum Permissible Error (MPE) thresholds, and generates historically reproducible, legally acceptable, standardized PDF test reports.
**Primary Users:** Laboratory Technicians, Reviewing/Approving Officers, System Administrators.
**Boundaries:** The system strictly handles data entry, calculation, compliance logic, and report generation. It does not interface directly with weighing hardware (IoT sensors) and is strictly limited to NAWIs, explicitly excluding Automatic Weighing Instruments (AWIs).

## 2. Problem Statement Analysis & Traceability
| SIH Requirement | SRS Requirement ID | System Module | Acceptance Method |
| :--- | :--- | :--- | :--- |
| Eliminate manual test calculation errors | FR-CALC-01, FR-RULE-01 | Calculation & Rule Engine | Matrix of defined OIML formula tests vs manual baseline. |
| Digitize test report generation | FR-REP-01, FR-REP-02 | Report Generation | Output matches exactly with the legal template. |
| Prevent historical data corruption | FR-VER-01, FR-AUD-01 | Versioning & Audit | Recalculating a 2022 report uses 2022 rules, not 2024 rules. |
| Enforce standard workflow | FR-WF-01 | Workflow / State | State transition table verification. |

## 3. Scope
### 3.1. In Scope
* Capture of instrument metadata (Manufacturer, Model, Class, Max, Min, e, d).
* Capture of laboratory environmental conditions.
* Structured entry of raw test observations (eccentricity, repeatability, weighing tests).
* Deterministic mathematical calculations & compliance evaluations against configured rules.
* Generation of standardized PDF test reports (and editable formats, if verified by Dept).
* Document attachment capabilities (circuit diagrams, instrument photos).
* Role-based access, strict workflow states, approval signatures, and audit trails.
* Explicit rule engine versioning ensuring historical reproducibility.

### 3.2. Out of Scope
* **Automatic physical test execution / IoT load-cell integration:** Not requested; introduces hardware dependency risks out of scope for the core compliance algorithm.
* **Automatic Weighing Instruments (AWI):** Different physics, different OIML standards (R 51, R 134). Scaling safely means excluding them from MVP.
* **General-purpose billing / POS software:** Irrelevant to laboratory model approval workflows.
* **Automatic extraction/migration of legacy Excel reports:** Scope is limited to new workflows to ensure data integrity constraints are met.
* **AI-driven decision making:** Regulatory compliance requires 100% deterministic, explainable, and legal physics-based mathematics, not probabilistic AI.

## 4. Definitions
* **NAWI:** Non-Automatic Weighing Instrument; requires human intervention to accept a weighing result (e.g., retail scale, static weighbridge).
* **AWI:** Automatic Weighing Instrument; weighs without human intervention (e.g., conveyor checkweigher).
* **OIML:** International Organization of Legal Metrology.
* **R 76:** OIML Recommendation specifying metrological requirements for NAWIs.
* **MPE:** Maximum Permissible Error; the legal tolerance threshold for a given test load.
* **e (Verification scale interval) & d (Actual scale interval):** Crucial foundational variables for instrument classification and MPE calculation.

## 5. System Users and Roles
| Role | Responsibilities | Permissions | Restricted Actions |
| :--- | :--- | :--- | :--- |
| **Laboratory Technician** | Performs physical tests, enters raw observations, uploads photos. | Create Test Record, Input Data, Submit for Review, View Reports. | Cannot approve/certify reports. Cannot alter rules. Cannot alter data after submission. |
| **Reviewing Officer** | Validates technician data, confirms compliance logic, grants final approval. | View all test records, Return to Technician, Approve Record, Export PDF. | Cannot alter raw test data directly (must return it). Cannot alter rules. |
| **System Administrator** | Modifies OIML/Legal metrology rule sets, manages users. | Manage User Roles, Create/Retire Rule Versions. | Cannot create or approve test reports. |

## 6. Complete End-to-End Workflow
**Workflow Diagram (Conceptual):**
Manufacturer Request -> Test Case Creation -> Instrument Identification (Class, Max, Min, e) -> Setup Lab Conditions -> Select Tests -> (Loop) Enter Observation -> System Validates & Calculates Error -> System Compares via Rules Engine against MPE -> Compliance Result -> Submit to Reviewer -> [Reviewer Return for Correction] OR [Reviewer Approve] -> Generate PDF Report -> Archive to Repository

## 7. Instrument Scope (The Classification Rule)
The system shall enforce a strict classification gateway:
1. **Operating Mode Check:** "Does this instrument require human intervention to record/accept the final weight?"
2. **If NO (AWI):** System flags "Out of Scope" (unless AWI modules are added in future phases).
3. **If YES (NAWI):** System proceeds to OIML R 76 testing track.
*Supported NAWI Configurations:* Retail electronic scales, bench/platform scales, static truck weighbridges (strictly operating in static mode). 
*TBD:* Whether mechanical NAWIs are in scope, or exclusively electronic variants.

## 8. Regulatory Requirements Matrix
| Authority / Document | Scope | Relevance to System |
| :--- | :--- | :--- |
| **Legal Metrology Act, 2009 (India)** | Statutory Law | Mandates use of legally verified models; provides ultimate legal authority. |
| **Legal Metrology (Approval of Models) Rules, 2011** | Administrative Regulations | Mandates that testing MUST occur; defines who can approve models. |
| **OIML Recommendation R 76-1** | International Technical Standard | Provides the specific mathematical formulas, error tolerances (MPE), and test step procedures. |
*Crucial Distinction:* OIML R 76 is the technical guideline. The *LM (Approval of Models) Rules, 2011* provides the legal mandate. The system uses R 76 to satisfy LM 2011.

## 9. OIML R76 Requirements (Conceptual Rule Examples)
*Note: Numbers are placeholders pending authoritative confirmation.*
* **Rule ID:** OIML-R76-MPE-01
* **Description:** MPE Class III Load <= 500e.
* **Applicability:** Class = III, 0 <= Load <= 500e.
* **Calculation/Decision:** If |Indication Error| <= 0.5e THEN PASS ELSE FAIL.
* **Source / Version:** OIML R 76-1 (Edition TBD).
* **Verification Method:** Unit test asserting calculation matrix against trusted NPL tables.


## 10. Functional Requirements

### 10.1. Instrument & Test Case Management
* **FR-INST-01:** The system shall require users to define the Instrument Class (I, II, III, IIII) before any test can be initiated.
* **FR-INST-02:** The system shall require the input of Max, Min, e, and d parameters prior to test generation.
* **FR-INST-03:** The system shall enforce logical constraints (e.g., Min >= lower bound requirement for Class) based on selected Rule Version.

### 10.2. Observation Entry & Validation
* **FR-OBS-01:** The system shall provide data entry grids specifically mapped to applicable OIML R76 tests (e.g., Eccentricity, Repeatability).
* **FR-OBS-02:** The system shall prevent Submission if mandatory observation fields for applicable tests are mathematically invalid (e.g., character strings in numeric fields).

### 10.3. Calculation Engine
* **FR-CALC-01:** The system shall calculate the "Indication Error" accurately based strictly on OIML Annex procedures (handling zero-setting, tare, and rounding adjustments).
* **FR-CALC-02:** The system shall compute absolute values natively ensuring multi-precision decimal correctness avoiding standard floating-point degradation.

### 10.4. Rule Engine & Compliance Evaluation
* **FR-RULE-01:** The system shall dynamically select the applicable MPE limit based on Instrument Class, verification scale interval (e), and applied load.
* **FR-RULE-02:** The system shall output a discrete compliance state (PASS / FAIL) for every individual test observation row.
* **FR-RULE-03:** The system shall aggregate row-level compliance into an overall test-section compliance state.

### 10.5. Review, Approval & Audit
* **FR-REV-01:** The system shall allow a Reviewing Officer to transition a test record state from Submitted to Approved or Returned for Correction.
* **FR-REV-02:** The system shall freeze all raw data fields upon Approved state transition.

### 10.6. Report Generation
* **FR-REP-01:** The system shall generate a combined PDF report containing Manufacturer Details, Instrument Parameters, Environmental Conditions, and formatted Test Observation tables.
* **FR-REP-02:** The report shall explicitly print the Rule Version utilized to calculate the compliance outcome.
* **FR-REP-03:** Upon generation, the PDF shall receive a unique, traceable document identifier.

### 10.7. Rule Versioning
* **FR-VER-01:** The system shall inextricably link every created Test Case to a specific, active Rule Version ID at the moment of creation.

## 11. Non-Functional Requirements (NFR)
* **NFR-SEC-01 (Security):** All passwords shall be hashed; RBAC shall enforce strict separation of duties.
* **NFR-PERF-01 (Performance):** Compliance evaluation routines shall execute in under 2 seconds per test suite to prevent UI blocking.
* **NFR-DATA-01 (Data Integrity):** Floating-point values representing weights/loads shall utilize exact decimal data types (precision configurations TBD by Metrology standards).
* **NFR-REP-01 (Reproducibility):** Recalculating a legacy test record shall consistently yield the identical compliance output regardless of contemporary rule alterations.
* **NFR-AUD-01 (Auditability):** Every state transition of a Test Record shall log the User ID, Timestamp (UTC), Old State, and New State.

## 12. Conceptual Data Requirements
*Note: This is a conceptual entity relationship map, NOT a database schema.*
* **User Entity:** Credentials, RBAC Role, Lab Location.
* **Instrument Definition Entity:** Manufacturer details, Max, Min, e, d, Class constraints.
* **Rule Engine Version Entity:** Activation Date, Deactivation Date, Configuration JSON/Payload for logic matrices.
* **Test Case Entity:** State (Draft/Submitted/Approved), Assigned Rule Version ID, Environmental specifics.
* **Observation Entity:** Parent Test Case ID, Test Type (e.g., Eccentricity), Applied Load, Screen Indication, Calculated Error, Local Compliance Result.
* **Attachment Entity:** Binary blobs (photos, PDFs) mapped to Test Case ID.

## 13. Rule Engine Requirements
The core innovation module. The Rule Engine must conceptually decouple pure logic from the UI.
* **Selection:** Queries the active database for the "Active Rule Set" at the date of test initiation.
* **Calculation:** Employs a deterministic function eval(Input, Rule) -> Result.
* **Missing Rule:** If an instrument is configured outside standard OIML logic bounds, the engine must HALT and throw an "Unsupported Configuration" warning rather than assuming defaults.
* **Explainability:** The engine must return a trace object along with its result. Example: Result: FAIL. Reason: Computed error (1.8g) exceeds selected threshold MPE (1.5g) for Class III at load 500e under RuleSetv1.2.

## 14. Calculation Requirements
* **Precision:** Calculations MUST utilize exact-precision math libraries (e.g., BigDecimal / decimal.js). Native loat data types are strictly prohibited for metrological calculations.
* **Unit Safety:** Inputs and rule thresholds must explicitly carry units. 5 (kg) vs 5000 (g). 
* **Rounding:** The methodology for evaluating rounding errors as directed by OIML R76 must be verifiably applied. TBD specific OIML formula selection for rounding adjustments.

## 15. Compliance Result Requirements (Explainability)
A result shall never present merely as "FAIL". It must expose:
1. Applied Load
2. Recorded Indication
3. Derived Error
4. Applicable MPE Limit
5. Final Comparison Statement (e.g., |2.0| > 1.5 -> FAIL).

## 16. Report Requirements
* **Contents:** Must unify Instrument Definitions, Lab Conditions, Test Observations and Final Compliance.
* **Regulatory Compliance:** The layout must eventually identically mirror the official Legal Metrology department's approved annexure template (Template specifics - TBD).
* **Security:** PDF outputs must be locked from editing and preferably digitally signed to ensure evidentiary integrity.

## 17. Workflow State Model
* **DRAFT:** Technician creates the test. Data is mutable.
* **TESTING:** Technician enters observations. Rules engine runs interactively.
* **SUBMITTED FOR REVIEW:** Record locked to Technician. Visible to Reviewer.
* **RETURNED FOR CORRECTION:** Unlocked for specified technician corrections.
* **APPROVED:** Locked globally. Cryptographically sealed. Report generation authorized.
* **ARCHIVED:** Post-retention lifecycle state.


## 18. Audit and Data Integrity Specifications
* **18.1 Cryptographic Record Locking:** Upon reaching the Approved state, a SHA-256 cryptographic hash of all raw observation records, calculated errors, instrument metadata, and rule version ID shall be generated and permanently anchored to the test record.
* **18.2 Append-Only Audit Log:** The system shall maintain an immutable, append-only audit trail recording every state transition, data modification attempt, user authentication event, and report generation action with UTC timestamps and user attribution.
* **18.3 Tamper Detection:** Any retroactive modification to historical observation data shall invalidate the cryptographic seal and immediately flag the test record as corrupted/tampered.

## 19. Security and Role-Based Access Control (RBAC) Requirements
* **19.1 Separation of Duties:** A user acting in the Technician role shall not possess privileges to transition a record to Approved. Approval authority is strictly restricted to Reviewing Officer / Verifier and Admin roles.
* **19.2 Least Privilege Principle:** Access to instrument master data, rule configurations, and laboratory calibration constants shall be restricted based on formal organizational hierarchy.
* **19.3 Credential & Session Management:** Session tokens shall be cryptographically signed, short-lived, and revocable upon logout or privilege modification.

## 20. Error Handling & Boundary Condition Specifications
* **20.1 Out-of-Range Inputs:** Entry of applied load values exceeding  + 9e$ (the legal limit of operation before balance cutoff) shall be rejected with immediate validation feedback.
* **20.2 Division-by-Zero & Undefined Interval Checks:** Scale interval ratios /d$ not adhering to valid powers of 10 or specified OIML ratios shall trigger a hard calculation block.
* **20.3 Incomplete Test Execution:** Test runs missing required observation steps (e.g., incomplete 5-position eccentricity grid) shall be blocked from transitioning to Submitted for Review.

## 21. Assumptions and Constraints
* **21.1 Statutory Primacy:** In any instance of direct conflict between international OIML R 76-1 guidelines and the Indian Legal Metrology (General) Rules 2011, Indian statutory law shall take absolute operational precedence.
* **21.2 Static Verification Environment:** Phase 1 specifications assume testing takes place in controlled laboratory conditions with pre-verified working standards (weights).
* **21.3 Offline Operation:** The client interface must allow local observation caching during intermittent laboratory network connectivity, synchronizing with the central verification ledger upon reconnect.

## 22. Acceptance Criteria & Verification Matrix
* **AC-01 (Rule Determinism):** 100% of benchmark test vectors from OIML R 76-1 Annex B/C must yield identical, reproducible Pass/Fail evaluations across 1,000 independent calculation executions.
* **AC-02 (Precision Accuracy):** Mathematical calculations of  = I + 0.5e - \Delta L - L$ and  = E - E_0$ must maintain zero rounding error up to 8 decimal places prior to metrological truncation.
* **AC-03 (Audit Integrity):** Any simulated bit-flip in the persisted test observation payload must result in verification failure during cryptographic checksum evaluation.

## 23. Requirements Traceability Matrix (RTM)

| Requirement ID | Requirement Category | Regulatory / Problem Reference | System Module | Verification Method |
| :--- | :--- | :--- | :--- | :--- |
| **FR-INST-01** | Instrument Classification | OIML R 76-1 Clause 3.1 | Instrument Master | Automated Schema Validation |
| **FR-OBS-01** | Test Grid Generation | OIML R 76-1 Clause A.4 | Observation Module | Visual & Boundary Testing |
| **FR-CALC-01** | Error Calculation ($) | OIML R 76-1 Clause A.4.4.3 | Calculation Engine | Mathematical Test Vectors |
| **FR-RULE-01** | Dynamic MPE Lookup | OIML R 76-1 Table 6 / LM Rules | Rule Engine | Deterministic Matrix Test |
| **FR-REV-01** | Multi-Tier Approval | LM (Approval of Models) 2011 | Workflow Engine | State Transition Testing |
| **FR-REP-01** | Standardized PDF Output | LM (General) Rules Seventh Sched | Report Generator | Layout & Content Audit |
| **FR-VER-01** | Historical Rule Versioning | Legal Evidentiary Standard | Versioning Service | Regression & Replay Testing |

---

# PART 2 — Multi-Phase Execution Plan

## 24. Comprehensive Multi-Phase Execution Breakdown

`
+-----------------------------------------------------------------------------------+
|                           PHASE-WISE EXECUTION ROADMAP                            |
+-----------------------------------------------------------------------------------+
| Phase 0: Regulatory Ingestion & Rule Modeling                                     |
| Phase 1: Precision Mathematical Specification & Test Vectors                      |
| Phase 2: Domain Entity & Data Modeling                                            |
| Phase 3: Rule Engine & Explainability Architecture                                |
| Phase 4: Workflow State Machine & Lifecycle Design                                |
| Phase 5: Cryptographic Audit & Integrity Architecture                             |
| Phase 6: Report Generation Engine & PDF Template Design                           |
| Phase 7: Verification & Validation (V&V) Test Harness Design                      |
+-----------------------------------------------------------------------------------+
|                        --- IMPLEMENTATION BOUNDARY ---                            |
+-----------------------------------------------------------------------------------+
| Phase 8: Core Mathematical & Rule Engine Implementation                           |
| Phase 9: Data Persistence Layer Implementation                                    |
| Phase 10: Workflow & RBAC Service Implementation                                  |
| Phase 11: PDF Engine & Digital Certificate Generation                             |
| Phase 12: User Interface & Frontend Observation Grid                              |
| Phase 13: End-to-End System Integration & Metrological Audit                      |
| Phase 14: Hackathon Demonstration, Packaging & Documentation                      |
+-----------------------------------------------------------------------------------+
`

### Phase 0: Regulatory & Standards Ingestion / Formal Rule Modeling
* **Objective:** Formalize legal requirements, MPE tables, and test protocols from OIML R 76-1 and Legal Metrology Rules (2011) into structured rule definitions.
* **Key Deliverables:** Formalized Rule Matrix JSON schemas, statutory reference mappings.

### Phase 1: Precision Mathematical Specification & Test Vector Definition
* **Objective:** Define exact-precision calculation formulas (zero correction, turning point determination, eccentricity error, repeatability variance) and compile comprehensive synthetic test vectors.
* **Key Deliverables:** Mathematical Specification Document, 50+ golden metrological test vectors with expected decimal outputs.

### Phase 2: Domain Entity & Conceptual Data Architecture
* **Objective:** Model core metrological domain entities, attributes, relationships, and immutable historical versioning links.
* **Key Deliverables:** Entity Relationship Models, Entity Data Dictionaries, State transition schemas.

### Phase 3: Rule Engine Design & Explainability Architecture
* **Objective:** Architect the decoupled deterministic rule evaluation pipeline capable of returning transparent calculation traces alongside Pass/Fail verdicts.
* **Key Deliverables:** Rule Engine execution flowcharts, Trace object structure definitions.

### Phase 4: Workflow State Machine & Lifecycle Design
* **Objective:** Detail the state transitions (Draft -> Testing -> Submitted -> Approved -> Archived), actor permissions, lock-out triggers, and rejection loop mechanics.
* **Key Deliverables:** Formal State Transition Tables, Guard condition definitions.

### Phase 5: Audit, Provenance & Tamper-Evident Integrity Architecture
* **Objective:** Specify cryptographic hashing algorithms, digital signature models, append-only audit structures, and certificate verification protocols.
* **Key Deliverables:** Cryptographic Hash Sequence diagrams, Tamper-detection specifications.

### Phase 6: Report Generation Engine & PDF Template Design
* **Objective:** Design the pixel-accurate report layout matching Seventh Schedule Legal Metrology certificates and OIML R 76 Annex format.
* **Key Deliverables:** Standardized Certificate Layout Specs, dynamic data binding blueprints.

### Phase 7: Verification & Validation (V&V) Strategy
* **Objective:** Formulate comprehensive unit, integration, metrological vector, and edge-case testing protocols prior to code development.
* **Key Deliverables:** Master Test Plan, Edge Case Test Matrix.

### Phase 8 to Phase 14: Implementation & Deployment Phases
*(Executed strictly after formal sign-off of Phases 0-7 specifications)*
* **Phase 8:** Core Metrological Engine (Exact Decimal Math & Rules).
* **Phase 9:** Data Persistence & Versioning Repositories.
* **Phase 10:** Workflow, RBAC & State Enforcement Services.
* **Phase 11:** PDF Compilation & Digital Watermarking Subsystem.
* **Phase 12:** Responsive Laboratory Data-Entry UI.
* **Phase 13:** Metrological End-to-End Stress & Regression Testing.
* **Phase 14:** Production Packaging, Deployment & Hackathon Demo Preparation.

## 25. Phase Dependency Graph

`
[Phase 0: Rules Modeling]  ---> [Phase 1: Math Specs & Vectors]
         |                                 |
         v                                 v
[Phase 2: Data Modeling]   ---> [Phase 3: Rule Engine Arch]
         |                                 |
         v                                 v
[Phase 4: Workflow Design] ---> [Phase 5: Audit Arch] ---> [Phase 6: Report Design]
                                                                  |
                                                                  v
                                                       [Phase 7: V&V Strategy]
                                                                  |
==================== IMPLEMENTATION GATEWAY ======================+
                                                                  |
                                                       [Phase 8: Core Engine Code]
                                                                  |
                                                       [Phase 9: Data Layer Code]
                                                                  |
                                                       [Phase 10: Workflow Code]
                                                                  |
                                                       [Phase 11: Report Code]
                                                                  |
                                                       [Phase 12: Frontend UI]
                                                                  |
                                                       [Phase 13: E2E Testing]
                                                                  |
                                                       [Phase 14: Final Demo]
`

## 26. Formal Minimum Viable Product (MVP) Definition

### Included in MVP Scope (Hackathon Deliverable)
1. **Instrument Profiles:** Class I, II, III, and IIII Single-Range NAWIs ($, $, $, $).
2. **Core Test Suite:**
   * Weighing Performance Test (Increasing and Decreasing loads up to $).
   * Eccentricity Test (Center, Front-Left, Front-Right, Rear-Right, Rear-Left at /3$).
   * Repeatability Test (3-10 repeated loads at $\approx 0.5 Max$ and $\approx Max$).
   * Tare & Zero-Setting Performance Verification.
3. **Calculation & Rule Evaluation:** Precision decimal calculation with step-by-step mathematical trace and MPE boundary checks.
4. **State Lifecycle:** Complete lifecycle transitions (Draft -> Submitted -> Approved / Returned).
5. **Report Generation:** Downloadable, locked PDF certificate matching Legal Metrology annexure formatting.
6. **Rule Versioning:** Support for at least 2 distinct rule sets (e.g., OIML R 76-1:2006 vs Legal Metrology Rules 2011).

### Deferred to Post-MVP (Phase 2+)
* Multi-Interval (, e_2$) and Multiple-Range Instruments.
* Direct IoT / RS-232 live serial port capture from physical scale indicators.
* Automated Climatic Chamber temperature/humidity sensor logging integration.
* Advanced Multi-Language Localization (Regional Indian Languages).

## 27. Phase-Wise Testing Strategy

| Phase | Target Module | Testing Methodology | Success Gate |
| :--- | :--- | :--- | :--- |
| **Phase 8** | Math & Rule Engines | 100% automated unit tests against 50+ OIML golden vectors | 0.000% deviation from expected results |
| **Phase 9** | Persistence Layer | Concurrency testing, version integrity checks | Zero data corruption during concurrent writes |
| **Phase 10** | Workflow Engine | State machine transition fuzzing, permission breach injection | Zero illegal state transitions allowed |
| **Phase 11** | Report Generator | Visual regression, font rendering, checksum verification | 100% PDF generation without layout truncation |
| **Phase 12** | Frontend Observation | Usability testing, client-side boundary validation | Clear error highlighting on invalid load inputs |
| **Phase 13** | End-to-End System | Complete end-to-end testing from instrument entry to PDF | Flawless end-to-end execution across all test classes |

## 28. Critical Risk Register

| Risk ID | Risk Description | Severity | Probability | Mitigation Strategy |
| :--- | :--- | :--- | :--- | :--- |
| **RSK-01** | Floating-point rounding error in borderline MPE evaluation | **CRITICAL** | HIGH | Enforce strict arbitrary-precision decimal types (Decimal / BigDecimal) across all layers. |
| **RSK-02** | Retrospective data corruption upon rule modification | **CRITICAL** | MEDIUM | Immutable rule versioning; test records link to static rule snapshots at creation time. |
| **RSK-03** | PDF formatting mismatch with official Legal Metrology standards | HIGH | MEDIUM | Standardize template against official Gazette Seventh Schedule specifications. |
| **RSK-04** | Discrepancies between OIML R 76 and Indian Legal Metrology Rules | HIGH | LOW | Implement explicit rule-set selector with statutory override flags. |
| **RSK-05** | Unauthorized modification of approved test records | **CRITICAL** | LOW | Cryptographic SHA-256 seal and database-level immutable record locking on approval. |

## 29. Formal Open Questions & Regulatory Ambiguities Checklist
* **Q-01:** Does the targeted Indian Legal Metrology jurisdiction mandate the exact Seventh Schedule format or allow customized digital certificates containing equivalent data?
* **Q-02:** What is the precise statutory policy on rounding intermediate turning points versus final indication errors during field verification?
* **Q-03:** Are digital signatures required to be Indian Information Technology Act (Aadhaar / eSign / DSC Class 3) compliant, or is a system-generated cryptographic watermark sufficient for the hackathon MVP?

## 30. Explicit Pre-Development Boundaries & Governance

* **What We Know with Certainty:**
  1. OIML R 76-1:2006 mathematical formulations and MPE tiers for Classes I, II, III, IIII.
  2. The statutory role of the Indian Legal Metrology Act (2009) and Model Approval Rules (2011).
  3. The necessity of separating pure deterministic rule logic from UI and data persistence.
* **What We Assume for Design:**
  1. Standard lab conditions (20°C ± 5°C) apply unless environmental influence tests are specifically activated.
  2. Single-range instruments constitute >85% of standard field model approval test cases.
* **What We Must Verify Before Coding Phase:**
  1. Exact state government annexure forms for model verification test certificates.
* **What We Can Design Now:**
  1. Complete system architecture, data models, rule schemas, and workflow state machines.
* **What We Must NOT Implement Yet:**
  1. No writing of backend application code, frontend framework files, or database deployment scripts until the architecture and rule matrices are fully validated.

---
