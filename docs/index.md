# AI Infrastructure Proposal: The Business of Attesting Truth

## 1. Vision: Attestation & Canonical Models
The core business goal is to shift from ephemeral data extraction to **attesting truth** and building **canonical models** that scale. Traditional AI implementations often result in untyped, unversioned outputs that are difficult to verify. We propose a system where every piece of data is a signed, evidence-backed **Claim**.

### Why This Scales
*   **Decoupling Extraction from Knowledge:** By building a canonical site model through fused claims, the system handles N-customers and N-document types without constant code intervention.
*   **Verifiable Facts:** Instead of "chatting with documents," the system produces a structured representation of reality (e.g., a tower's physical configuration) that the business can rely on for high-stakes decisions like engineering and construction.

---

## 2. Architecture & Components

The architecture follows a **Supervisor/Worker** pattern, utilizing **Domain-Driven Design (DDD)** to isolate business rules from technical orchestration.

### 2.1 Component Breakdown
*   **The Supervisor (Planner):** Acts as a "planner over missing claims." It identifies what evidence is required to reach a target schema version and delegates tasks dynamically.
*   **Domain Workers:** Specialized agents (e.g., RFDS Agent, Lease Agent) that are stateless and focused on generating specific claims.
*   **Memory Handling:**
    *   **Stateless Workers:** We avoid "chat history" bloat. Workers receive only the necessary context for their specific task.
    *   **Structured State:** We leverage managed memory stores and Vertex AI Sessions to maintain a verifiable event history without re-injecting massive prompt histories.
    *   **Long-Term Memory:** Domain knowledge and user preferences are persisted independently of the session state.

### 2.2 Input/Output Examples
The system uses a standardized protocol to ensure agents remain composable and the system remains vendor-agnostic.

**Example Input (Task Configuration):**
```json
{
  "site_id": "WSUTH0035137",
  "attachments": [
    "s3://inorsa-dev/agent-nora/ansco-data-validation-demo/WSUTH0035137/AE201.24-10-25.WSUTH0035137.IDL04374.pdf",
    "s3://inorsa-dev/agent-nora/ansco-data-validation-demo/WSUTH0035137/DE113.24-10-25.WSUTH0035137.IDL04374.pdf",
    "s3://inorsa-dev/agent-nora/ansco-data-validation-demo/WSUTH0035137/LS106 - WSUTH0035137.pdf",
    "s3://inorsa-dev/agent-nora/ansco-data-validation-demo/WSUTH0035137/RFDS - WSUTH0035137 - 12781456.pdf"
  ]
}
```

**Example Output (The Claim):**
```json
{
  "claim_id": "c_12345",
  "key": "top_of_antenna_height",
  "value": 150.5,
  "unit": "ft",
  "confidence": 0.98,
  "rationale": "Extracted from Elevation Table on page 4 of AE201.",
  "provenance": {
    "document": "AE201.24-10-25.WSUTH0035137.IDL04374.pdf",
    "page": 4,
    "bbox": [120, 450, 150, 470]
  }
}
```

---

## 3. Workflow Pattern: Rough Spec to Refined Output

The system is designed to take a "rough spec" (raw document sets) and transform them into a "refined output" ready for technical drawings or engineering analysis.

### 3.1 Looper Agents
Rather than retrying entire workflows when a task fails, we use **Looper Agents** for individual task refinement.
*   **Task Isolation:** A single task is given to a worker.
*   **Internal Refinement:** The worker loops internally—reviewing its own work, checking against constraints, and refining the output until it meets the required confidence threshold.
*   **Efficiency:** This prevents the "cascading failure" problem common in linear pipelines.

### 3.2 Intermediary Checks
The architecture allows for specialized agents to intervene at any point in the lifecycle:
*   **Lease Agents:** Validate claims against lease agreement constraints.
*   **Proposal Agents:** Ensure the evolving site model aligns with the project proposal.
*   **Infrastructure Agents:** Check for buildability against existing infrastructure standards.

### 3.3 No Vendor Lock-in
By defining truth through a **Claims Protocol** rather than model-specific prompts, the platform remains highly portable. We can swap underlying LLMs (e.g., from Gemini to GPT or local models) or OCR engines without rewriting the business logic, as long as they adhere to the input/output contract.

---

## 4. Competitive & Strategic Advantage
*   **Platform Native:** We build *on* the platform (Vertex AI) rather than *against* it with middleware libraries like LangGraph, reducing "maintenance tax" and technical debt.
*   **Deterministic Foundation:** We use DocAI for structural extraction, providing a grounded foundation for the probabilistic LLM layers.
*   **Human-in-the-Loop (HITL):** Discrepancies (e.g., SA vs. RFDS height conflicts) are handled via managed "Pause and Resume" workflows, treating human resolution as a high-confidence claim.
