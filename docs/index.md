# AI for Physical Infrastructure

> **"Uber didn't build the roads."**

## 1. Strategic Vision

### The Infrastructure Analogy
Uber didn't build the roads, bridges, or tunnels. They built a service layer *on top* of existing public infrastructure to revolutionize transportation. 

Similarly, our value proposition is **AI for Infrastructure** (Telecom, Energy, Utilities). We do not build the "AI roads"—the foundation models, GPU clusters, and training pipelines. We build the verification and application layer that allows physical infrastructure companies to safely use these powerful new tools.

### Why We Build *On* the Giants
Building foundation models is a capital-intensive "race to the bottom." Instead, we leverage **Google Cloud Vertex AI** to treat intelligence as a reliable, scalable utility.

#### 1. No Vendor or Model Lock-in
The AI landscape shifts weekly. By building on the **Vertex Model Garden**, we completely decouple our business logic from specific model providers.
*   **Gemini Models:** Utilized for massive context windows and native multimodal (text/image/video) reasoning.
*   **Claude (Anthropic):** Available as a first-class citizen for high-nuance tasks.
*   **Open Models (Llama, Mistral):** Deployable for cost-sensitive, high-volume, or data-sovereign workloads.

*We can swap the engine, but the vehicle—our Claims Platform—remains the same.*

#### 2. Economic Efficiency
We operate on a purely OPEX, pay-as-you-go model (see [Vertex AI Pricing](https://cloud.google.com/vertex-ai/generative-ai/pricing)).
*   **Zero CapEx:** We do not purchase H100s, manage data center cooling, or depreciate hardware.
*   **Dynamic Optimization:** We route simple extraction tasks to cost-effective models (e.g., Gemini Flash) and complex reasoning to "Pro" or "Ultra" variants. This allows us to protect margins while scaling from 10 to 10 million documents without architectural changes.

---

## 2. Technical Execution: The Business of Attesting Truth

### Vision: Attestation & Canonical Models
The core business goal is to shift from ephemeral data extraction to a system that enables both structured attestation and interactive document engagement. Customers and internal users want to be able to chat with documents to ask questions and refine them at the same time for feedback loops - this functionality serves practical business needs, not just convenience. The system also moves away from rigid, hard-coded automation pipelines where random pieces of data are extracted and manually assembled to build CAD models. Current approaches follow very specific, rigid patterns that require extensive custom code for each document type, creating maintenance burdens and limiting scalability. Our AI attestation approach allows us to scale both in domain knowledge and code-wise. We propose a system where every piece of data is a signed, evidence-backed **Claim** that can be verified and trusted.

### Why This Scales
*   **Decoupling Extraction from Knowledge:** By building a canonical site model through fused claims, the system handles N-customers and N-document types without constant code intervention.
*   **Verifiable Facts:** As more data is processed through the system, confidence scores increase, creating a self-reinforcing cycle of accuracy and reliability that the business can rely on for high-stakes decisions like engineering and construction.

---

## 3. Architecture & Components

The architecture follows a **Supervisor/Worker** pattern, utilizing **Domain-Driven Design (DDD)** to isolate business rules from technical orchestration.

### 3.1 Component Breakdown
*   **The Supervisor (Planner):** Acts as a "planner over missing claims." It identifies what evidence is required to reach a target schema version and delegates tasks dynamically.
*   **Domain Workers:** Specialized agents (e.g., RFDS Agent, Lease Agent) that are stateless and focused on generating specific claims.
*   **Stateless by Design:** We explicitly reject the "Stateful Graph" anti-pattern (like complex LangGraph chains). State is heavy, brittle, and difficult to debug. Instead, we treat state as a natural byproduct of the ecosystem:
    *   **Context:** The immediate working set of data (e.g., the current document).
    *   **Session:** The ephemeral interaction history (e.g., "Why did you choose that value?").
    *   **Memory:** The long-term persistent store of validated truths (The Database).

### 3.2 The "Context Assembly" Anti-Pattern
Many AI systems fail because they rely on "Context Assembly"—manually stitching together strings from various sources to feed an LLM. This is fragile and error-prone.
*   **Our Approach:** We use native primitives. **Vertex AI Context Caching** allows us to "freeze" massive documents into the model's native memory, eliminating the need to constantly re-upload or manually parse text. This reduces latency and cost while improving accuracy.

### 3.3 Input/Output Examples
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

**Example Output (Aggregated Claims - Comprehensive RFDS Structure):**
```json
{
  "document_metadata": {
    "title": "RF Design Study for Site WSUTH0035137",
    "specification_id": "AE201.24-10-25.WSUTH0035137",
    "revision": "A",
    "authoring_organization": "Example Engineering Corp",
    "publication_date": "2024-10-25",
    "applicable_system": "Cellular Network Infrastructure",
    "classification": "Internal Use",
    "referenced_standards": ["3GPP TS 36.101", "FCC Part 24", "ETSI EN 301 908-1"],
    "source_pages": [1, 2, 3, 4, 5]
  },

  "rf_system_overview": {
    "system_type": "LTE/5G NR Base Station",
    "application_domain": "Cellular Network Infrastructure",
    "operating_modes": ["FDD", "TDD"],
    "architecture_description": "MIMO 4x4 with Beamforming",
    "frequency_bands": [
      {
        "band_name": "Band 13",
        "lower_frequency": {
          "value": 746,
          "unit": "MHz",
          "si_value_hz": 746000000
        },
        "upper_frequency": {
          "value": 756,
          "unit": "MHz",
          "si_value_hz": 756000000
        },
        "confidence": 0.95,
        "source_pages": [4]
      }
    ]
  },

  "performance_specifications": {
    "frequency_characteristics": {
      "channel_bandwidth": [
        {
          "value": 20,
          "unit": "MHz",
          "confidence": 0.92,
          "source_pages": [6]
        }
      ],
      "frequency_stability": [
        {
          "value": 0.1,
          "unit": "ppm",
          "confidence": 0.88,
          "source_pages": [7]
        }
      ]
    },

    "power_and_gain": {
      "output_power": [
        {
          "value": 43,
          "unit": "dBm",
          "confidence": 0.94,
          "source_pages": [8]
        }
      ],
      "antenna_gain": [
        {
          "value": 18,
          "unit": "dBi",
          "confidence": 0.91,
          "source_pages": [9]
        }
      ]
    },

    "signal_quality": {
      "adjacent_channel_rejection": [
        {
          "value": 45,
          "unit": "dB",
          "confidence": 0.89,
          "source_pages": [10]
        }
      ]
    }
  },

  "environmental_mechanical": {
    "operating_temperature": [
      {
        "min_value": -40,
        "max_value": 65,
        "unit": "Celsius",
        "confidence": 0.96,
        "source_pages": [12]
      }
    ],
    "physical_dimensions": {
      "length": {
        "value": 600,
        "unit": "mm",
        "confidence": 0.90,
        "source_pages": [13]
      },
      "width": {
        "value": 400,
        "unit": "mm",
        "confidence": 0.90,
        "source_pages": [13]
      },
      "height": {
        "value": 200,
        "unit": "mm",
        "confidence": 0.90,
        "source_pages": [13]
      }
    }
  },

  "compliance_regulatory": {
    "regulatory_standards": ["FCC Part 24", "IC RSS-129", "CE Marking"],
    "certification_requirements": ["FCC ID", "ISED Certification"],
    "safety_constraints": ["UL 62368-1", "IEC 62368-1"]
  },

  "tables": [
    {
      "table_title": "Power Amplifier Specifications",
      "columns": ["Parameter", "Min", "Typ", "Max", "Unit"],
      "rows": [
        ["Output Power", "-", "40", "43", "dBm"],
        ["Gain", "15", "18", "20", "dB"]
      ],
      "source_pages": [8]
    }
  ],

  "extraction_metadata": {
    "model_version": "gemini-3.0-pro",
    "extraction_timestamp": "2024-11-05T10:30:00Z",
    "confidence_aggregate": 0.92
  }
}
```

This represents the final aggregated output after multiple individual claims have been collected and combined into a comprehensive document representation.

---

## 4. Workflow Pattern: Rough Spec to Refined Output

The system is designed to take a "rough spec" (raw document sets) and transform them into a "refined output" ready for technical drawings or engineering analysis.

### 4.1 Looper Agents
Rather than retrying entire workflows when a task fails, we use **Looper Agents** for individual task refinement.
*   **Task Isolation:** A single task is given to a worker.
*   **Internal Refinement:** The worker loops internally—reviewing its own work, checking against constraints, and refining the output until it meets the required confidence threshold.
*   **Efficiency:** This prevents the "cascading failure" problem common in linear pipelines.

### 4.2 Intermediary Checks
The architecture allows for specialized agents to intervene at any point in the lifecycle:
*   **Lease Agents:** Validate claims against lease agreement constraints.
*   **Proposal Agents:** Ensure the evolving site model aligns with the project proposal.
*   **Infrastructure Agents:** Check for buildability against existing infrastructure standards.

### 4.3 No Vendor Lock-in (Technical Layer)
By defining truth through a **Claims Protocol** rather than model-specific prompts, the platform remains highly portable. We can swap underlying LLMs (e.g., from Gemini to GPT or local models) or OCR engines without rewriting the business logic, as long as they adhere to the input/output contract.

---

## 5. Competitive & Strategic Advantage
*   **Platform Native:** We build *on* the platform (Vertex AI) rather than *against* it. We avoid heavy middleware libraries that enforce their own state management paradigms.
*   **Stateless Scalability:** By relying on native Cloud primitives (Cloud Run, Vertex AI, Pub/Sub), our agents scale to zero and handle massive concurrency without the "State Management Tax."
*   **Deterministic Foundation:** We use DocAI for structural extraction, providing a grounded foundation for the probabilistic LLM layers.
*   **Human-in-the-Loop (HITL):** Discrepancies (e.g., SA vs. RFDS height conflicts) are handled via managed "Pause and Resume" workflows, treating human resolution as a high-confidence claim.

---

## 6. Observability: Trust at Scale

Observability in AI infrastructure is not just about system logs; it is about exposing the "Chain of Thought" and "Chain of Custody" for every decision. We provide tailored views for every stakeholder to ensure trust is verifiable, not assumed.

### 6.1 For Executives: ROI & Risk
*   **Cost-Per-Fact:** Granular tracking of token consumption allows us to calculate the exact cost to extract a specific data point (e.g., "It cost $0.03 to validate the antenna height").
*   **Throughput Metrics:** Dashboard views of site processing velocity—shifting from "sites per month" to "sites per hour."
*   **Risk Heatmaps:** Identification of projects with low-confidence claims, allowing leadership to focus human QA resources where they matter most.

### 6.2 For Operations: The Control Plane
*   **Job Lifecycle Tracking:** Real-time status of every site (Queued -> Processing -> Waiting for Human -> Complete).
*   **Bottleneck Detection:** Identification of specific agents (e.g., the Lease Validator) that are consistently stalling or requesting human help, signaling a need for process refinement.

### 6.3 For Domain Experts: The "Glass Box" Experience
*   **Deep Provenance:** We never present a value without its source. Clicking a "Claim" (e.g., Antenna Azimuth) instantly opens the source PDF and draws a bounding box around the specific text or table cell used to derive that value.
*   **Audit Trails:** A tamper-proof history of the "Truth." We distinguish between values extracted by AI (with confidence scores) and values overridden or confirmed by a Human Engineer.
    *   *v1 (AI):* 150.0 ft (80% confidence)
    *   *v2 (Human):* 155.0 ft (Reason: "Updated per latest redlines")

### 6.4 For Developers: Tracing & Debugging

*   **Full Stack Tracing:** Leveraging Vertex AI Traces to see the full request/response lifecycle of every agent interaction.

*   **Eval-Driven Development:** Continuous monitoring of agent performance against "Golden Sets" to ensure that model upgrades (e.g., to **Gemini 3**) actually improve accuracy before deployment.
