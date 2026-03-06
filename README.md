# 🏥 Aadhya AI — Complete System Architecture
### *Built for AestheticQ · March 2026*

---

> **What you're reading:** A complete, visual guide to every intelligent capability that has been built inside Aadhya AI — from how a patient types a question to how the AI finds the right answer in the database, remembers conversations, and keeps data private. Every diagram is designed to be easy to understand without a technical background.

---

## 📌 Table of Contents

1. [The Big Picture — Cloud Architecture](#1-the-big-picture--cloud-architecture)
2. [How Agents Talk to Each Other — Agentic Workflow](#2-how-agents-talk-to-each-other--agentic-workflow)
3. [How Memory Works — Mem0](#3-how-memory-works--mem0)
4. [Feature: General Chat (Ask Anything)](#4-feature-general-chat--ask-anything)
5. [Feature: File Upload & Document Processing](#5-feature-file-upload--document-processing)
6. [Feature: Pre-Consultation Chat](#6-feature-pre-consultation-chat)
7. [Feature: Follow-Up Care Chat](#7-feature-follow-up-care-chat)
8. [Feature: Treatment Planner (Doctor Tool)](#8-feature-treatment-planner--doctor-tool)
9. [Feature: Product & Prescription Validation](#9-feature-product--prescription-validation)
10. [Feature: Personalised Recommendations](#10-feature-personalised-recommendations)
11. [Feature: Doctor–Patient Overview](#11-feature-doctorpatient-overview)
12. [Feature: Safety, Privacy & Security](#12-feature-safety-privacy--security)
13. [What Has Been Built — Feature Completion](#13-what-has-been-built--feature-completion)

---

## 1. The Big Picture — Cloud Architecture

Think of Aadhya AI as a smart hospital assistant that lives in the cloud. A patient or doctor opens an app, types a question, and within seconds they get an intelligent, accurate answer. Behind the scenes, a sophisticated system of AI agents, databases, and memory services work together.

```mermaid
graph TB
    subgraph CLIENT["Client Layer"]
        APP["Mobile or Web App - Patient or Doctor"]
    end

    subgraph API["API Gateway Layer"]
        FAST["FastAPI Server - api_server.py - Port 8000"]
        MASK["PII Masking - Emails and Phones auto-hidden"]
        POOL["Connection Pool - Up to 50 concurrent users"]
    end

    subgraph AGENTS["Agentic Intelligence Layer - CrewAI"]
        ORCH["Orchestrator Agent - Decides who handles what"]
        SECURITY["Security Agent - Validates all inputs"]
        QUERY["Query Agent - Answers questions"]
        FILE["File Processor Agent - Reads and summarises docs"]
        BOOKING["Booking Agent - Pre-consultation chat"]
        MED["Medication Agent - Safe medicine suggestions"]
        FOLLOWUP["Follow-Up Agent - Post-treatment check-ins"]
        TREATMENT["Treatment Planner Agent - Doctor structured plans"]
        RECOMMEND["Recommendation Agent - Personalised suggestions"]
        PATIENT_OV["Patient Overview Agent - Doctor patient chatbot"]
        VALID["Validation Agent - Document and prescription checks"]
        SKIN["Skin Profile Agent - Product matching by skin type"]
    end

    subgraph MCP["MCP Tool Server - Model Context Protocol"]
        MCP_SRV["MCP Server - server.py - Bridges AI to Database"]
        DB_TOOL["run_query - execute SQL"]
        RAG_TOOL["rag_search - vector similarity"]
        FILE_TOOL["file_search - uploaded docs"]
    end

    subgraph MEMORY["Memory Layer - Mem0"]
        MEM0["Mem0 Memory - Long-term per-user"]
        QDRANT["Qdrant Vector Store - Stores memory embeddings"]
        SQLITE_MEM["SQLite History DB - Memory change log"]
    end

    subgraph RAG_KB["Knowledge Base - RAG"]
        CHROMA["ChromaDB - Medical PDF knowledge"]
        PDFSTORE["PDF Documents - Aesthetics and Medical guides"]
    end

    subgraph DB["PostgreSQL Database"]
        DOCTORS["doctor_details - doctor_clinic_details - doctor_service"]
        PATIENTS["patient_profile - patient_booked_slot - file_summaries"]
        SERVICES["services - service_onboarding_details - diagnostic_services"]
        PRODUCTS["product - product_details - skin_profiles"]
    end

    subgraph AZURE["Azure OpenAI"]
        GPT["GPT-4o-mini - LLM reasoning engine"]
        EMBED["text-embedding-ada-002 - Turns text into vectors"]
        VISION["GPT-4o Vision - Reads images and scanned docs"]
    end

    APP -->|"HTTPS POST /orch"| FAST
    FAST --> MASK
    FAST --> POOL
    FAST --> ORCH
    ORCH --> SECURITY
    SECURITY -->|"Approved"| QUERY
    SECURITY -->|"Approved"| FILE
    SECURITY -->|"Approved"| BOOKING
    SECURITY -->|"Approved"| MED
    SECURITY -->|"Approved"| FOLLOWUP
    SECURITY -->|"Approved"| TREATMENT
    SECURITY -->|"Approved"| RECOMMEND
    SECURITY -->|"Approved"| PATIENT_OV
    SECURITY -->|"Approved"| SKIN

    QUERY --> MCP_SRV
    FILE --> MCP_SRV
    BOOKING --> MCP_SRV
    MED --> MCP_SRV
    FOLLOWUP --> MCP_SRV
    TREATMENT --> MCP_SRV
    RECOMMEND --> MCP_SRV
    PATIENT_OV --> MCP_SRV
    SKIN --> MCP_SRV
    VALID --> MCP_SRV
    FILE --> VALID

    MCP_SRV --> DB_TOOL
    MCP_SRV --> RAG_TOOL
    MCP_SRV --> FILE_TOOL

    DB_TOOL --> DB
    RAG_TOOL --> PATIENTS
    RAG_TOOL --> CHROMA

    ORCH --> MEM0
    MEM0 --> ORCH
    MEM0 --> QDRANT
    MEM0 --> SQLITE_MEM

    PDFSTORE --> CHROMA
    FAST --> AZURE
    AZURE --> FAST
    MCP_SRV --> AZURE
    AZURE --> MCP_SRV
    FILE_TOOL --> VISION

    style CLIENT fill:#e8f4fd,stroke:#2196F3
    style API fill:#fff3e0,stroke:#FF9800
    style AGENTS fill:#e8f5e9,stroke:#4CAF50
    style MCP fill:#fce4ec,stroke:#E91E63
    style MEMORY fill:#ede7f6,stroke:#673AB7
    style RAG_KB fill:#e0f2f1,stroke:#009688
    style DB fill:#fff8e1,stroke:#FFC107
    style AZURE fill:#e3f2fd,stroke:#1565C0
```

### In Plain English

| Layer | What It Does |
|---|---|
| **Client App** | Where the patient or doctor types their question |
| **FastAPI Server** | The front door — receives requests, sends replies, and masks sensitive info |
| **Orchestrator Agent** | The traffic controller — reads the request and decides which specialist agent to call |
| **Security Agent** | The bouncer — every single message is checked before any AI agent sees it |
| **Specialist Agents** | 10 purpose-built AI agents, each expert in their own task |
| **MCP Tool Server** | The bridge that lets AI agents safely talk to the database |
| **PostgreSQL Database** | Where all real clinic data lives — doctors, patients, services, products |
| **Mem0 Memory** | Long-term memory — Aadhya remembers patients across sessions |
| **RAG Knowledge Base** | A scanned library of medical PDFs for answering clinical questions |
| **Azure OpenAI** | The AI brain — GPT-4o-mini for reasoning, Ada-002 for semantic search |

---

## 2. How Agents Talk to Each Other — Agentic Workflow

Every request flows through a specific sequence of agents. This ensures accuracy, safety, and the right answer every time.

```mermaid
sequenceDiagram
    participant U as 👤 User (App)
    participant API as ⚡ FastAPI
    participant SEC as 🛡️ Security Agent
    participant ORCH as 🧠 Orchestrator
    participant MEM as 🧠 Mem0 Memory
    participant SPEC as 🤖 Specialist Agent
    participant MCP as 🔌 MCP Server
    participant DB as 🗄️ PostgreSQL
    participant AI as ☁️ Azure OpenAI

    U->>API: POST /orch with session_id, user_id, input
    API->>API: Create isolated OrchestrationCrew per request
    API->>SEC: Validate input — healthcare check + jailbreak check
    
    alt Input is invalid or non-healthcare
        SEC-->>API: Rejected
        API-->>U: I can only help with healthcare topics
    else Input is valid
        SEC-->>ORCH: Approved
        ORCH->>MEM: Search past memories for this user
        MEM-->>ORCH: Relevant past memories if any
        ORCH->>SPEC: Delegate to right agent with full context
        
        loop Agent thinks up to 10 iterations
            SPEC->>MCP: Call tool — database_query or rag_search
            MCP->>DB: Execute SQL query
            DB-->>MCP: Real data rows
            MCP-->>SPEC: Formatted results
            SPEC->>AI: Reason about results and formulate answer
            AI-->>SPEC: Next thought or final answer
        end
        
        SPEC-->>ORCH: Final answer as JSON
        ORCH->>MEM: Save conversation to memory in background
        ORCH-->>API: JSON response
        API->>API: Mask PII — emails, phone numbers
        API-->>U: Clean, private, accurate response
    end
```

### How the Orchestrator Decides Which Agent to Use

```mermaid
flowchart TD
    REQ["📨 Incoming Request"] --> SEC["🛡️ Security Agent validates first"]
    SEC -->|"Rejected"| BLOCK["🚫 Blocked Response"]
    SEC -->|"Approved"| ROUTE{"🧠 Orchestrator reads request fields"}

    ROUTE -->|"input + general question"| QA["💬 Query Agent + Medication Agent"]
    ROUTE -->|"file_urls present"| FILE["📄 File Processor Agent + Validation Agent"]
    ROUTE -->|"slot_id present"| BOOK["📅 Booking Agent — Pre-consultation"]
    ROUTE -->|"followUp: true"| FU["🔄 Follow-Up Agent"]
    ROUTE -->|"treatment_planner_text"| TP["🩺 Treatment Planner Agent"]
    ROUTE -->|"recommendations: true"| REC["⭐ Recommendation Agent"]
    ROUTE -->|"skinValues present"| SKIN["🧴 Skin Profile Agent"]
    ROUTE -->|"patient_id + doctor user"| POV["👨‍⚕️ Patient Overview Agent"]
    ROUTE -->|"validation: true"| VAL["✅ Validation Agent"]

    QA --> OUT["📤 JSON Response"]
    FILE --> OUT
    BOOK --> OUT
    FU --> OUT
    TP --> OUT
    REC --> OUT
    SKIN --> OUT
    POV --> OUT
    VAL --> OUT

    style BLOCK fill:#ffcdd2,stroke:#c62828
    style OUT fill:#c8e6c9,stroke:#2e7d32
```

---

## 3. How Memory Works — Mem0

Aadhya does not forget. When a patient chats today, Aadhya remembers it next week. This is powered by **Mem0** — a vector memory system backed by Qdrant.

```mermaid
graph LR
    subgraph SESSION["🗂️ During This Conversation"]
        CHAT["Patient types messages"]
        HIST["In-memory session history — per session_id"]
        CHAT --> HIST
    end

    subgraph SAVE["💾 Saving Memories — runs in background"]
        HIST -->|"After response is sent"| MEM0["Mem0 Library"]
        MEM0 -->|"Summarise key facts"| GPT["Azure GPT-4o-mini"]
        GPT -->|"Create vector"| ADA["Azure text-embedding-ada-002"]
        ADA -->|"Store vector"| QDRANT["Qdrant Vector Store\n./qdrant_storage"]
        MEM0 -->|"Log change"| SQLITE["SQLite History DB\nlogs/mem0_history.db"]
    end

    subgraph RECALL["🔍 Recalling Memories — Next Session"]
        QUERY["New message arrives"]
        QUERY -->|"Search query"| MEM0_S["Mem0 Search"]
        MEM0_S -->|"Embed query"| ADA2["Azure text-embedding-ada-002"]
        ADA2 -->|"Find similar vectors"| QDRANT2["Qdrant Vector Store"]
        QDRANT2 -->|"Top 5 memories"| CONTEXT["Injected into agent context"]
        CONTEXT --> ANSWER["Agent answers with past context"]
    end

    QDRANT -.->|"persists to disk"| QDRANT2

    style SESSION fill:#e3f2fd,stroke:#1565C0
    style SAVE fill:#f3e5f5,stroke:#7B1FA2
    style RECALL fill:#e8f5e9,stroke:#2e7d32
```

### Memory Scoping — No Cross-Contamination

Each memory is tagged with `scope:user_id:session_id` so Patient A's memories never reach Patient B, and each session is completely isolated.

```mermaid
graph TD
    subgraph NAMESPACE["Memory Namespace Design — strict isolation"]
        M1["orch:patient_001:session_abc — Patient 1, Session A"]
        M2["orch:patient_001:session_xyz — Patient 1, Session B"]
        M3["orch:patient_002:session_def — Patient 2, Session A"]
        M4["booking:patient_001:session_abc — Booking scope, Patient 1"]
    end

    M1 -.->|"Never mixes"| M3
    M2 -.->|"Never mixes"| M3
    M1 -.->|"Separate scope"| M4
```

---

## 4. Feature: General Chat — Ask Anything

A patient or doctor types a question in plain language. Aadhya searches the right sources in the right order and returns a factual, real-time answer.

```mermaid
flowchart TD
    Q["💬 User Question\ne.g. Find a dermatologist in Chennai"]
    
    Q --> DB_FIRST["1️⃣ Search Database FIRST — mandatory for clinic-specific info"]
    
    DB_FIRST --> MCP_CALL["🔌 MCP Server receives call"]
    MCP_CALL --> SQL["🗄️ SQL Query executes\nSELECT id, full_name, specialty FROM doctor_details\nWHERE city ILIKE '%Chennai%'\nAND specialty ILIKE '%Dermatologist%'\nLIMIT 10"]
    SQL --> PG["PostgreSQL"]
    PG -->|"Returns rows"| FORMAT["Format results"]
    FORMAT -->|"Found doctors"| ANSWER["✅ Answer with real data — never hallucinate"]
    PG -->|"No results"| KNOWLEDGE{"Is it a general medical question?"}

    KNOWLEDGE -->|"Yes"| RAG["2️⃣ Search RAG Knowledge Base\nMedical PDFs in ChromaDB"]
    KNOWLEDGE -->|"No — clinic-specific"| NOT_FOUND["Could not find details in our records"]
    
    RAG --> CHROMA["ChromaDB vector search\nAesthetic and medical PDF library"]
    CHROMA -->|"Found"| RAG_ANS["Answer from knowledge base"]
    CHROMA -->|"Not found"| INTERNET["3️⃣ Internet Search — last resort only"]
    INTERNET --> WEB_ANS["Answer from web research"]

    ANSWER --> MASK["🔒 PII Masking applied"]
    RAG_ANS --> MASK
    WEB_ANS --> MASK
    NOT_FOUND --> MASK
    MASK --> RESP["Final Response to User"]

    style ANSWER fill:#c8e6c9,stroke:#2e7d32
    style NOT_FOUND fill:#fff9c4,stroke:#f9a825
    style MASK fill:#fce4ec,stroke:#c62828
```

### Patient Mode vs Doctor Mode

| | **AADHYA (Patient Mode)** | **Doctor Mode** |
|---|---|---|
| **Persona** | Friendly, warm, encouraging | Professional, clinical, evidence-based |
| **Greeting** | "Hi, I'm AADHYA, your aesthetics assistant!" | Addresses doctor by name |
| **Language** | Simple, easy to understand | Medical terminology embraced |
| **Booking push** | Actively encourages booking | No booking push needed |

---

## 5. Feature: File Upload & Document Processing

Patients upload medical files (PDFs or images). Aadhya reads, classifies, verifies, and stores them with a full AI summary.

```mermaid
flowchart TD
    UPLOAD["📎 File Uploaded — PDF or Image"]
    
    UPLOAD --> DOWNLOAD["Download file from Cloud Storage URL"]
    DOWNLOAD --> DETECT{"🔍 Step 1: Healthcare Detection\n120+ keyword scan"}
    
    DETECT -->|"No healthcare keywords found"| REJECT["❌ Rejected instantly\nThis does not appear to be a healthcare document\nZero AI cost — no LLM call made"]
    
    DETECT -->|"Healthcare keywords found"| EXTRACT{"📄 How to read this file?"}
    
    EXTRACT -->|"Typed PDF"| TEXT["Extract text directly — PyPDF2"]
    EXTRACT -->|"Scanned or Image PDF"| OCR["🔍 Run OCR — pytesseract"]
    EXTRACT -->|"Image JPG or PNG"| VISION["👁️ AI Vision — GPT-4o reads image"]
    
    TEXT --> CLASSIFY
    OCR --> CLASSIFY
    VISION --> CLASSIFY
    
    CLASSIFY["🏷️ Step 2: Classify Document Type — AI identifies one of 5 categories"]
    CLASSIFY --> TYPES{"Document Type"}
    TYPES --> T1["prescription"]
    TYPES --> T2["lab_report"]
    TYPES --> T3["diagnosis"]
    TYPES --> T4["medical_history"]
    TYPES --> T5["general"]

    T1 --> VERIFY_STEP
    T2 --> VERIFY_STEP
    T3 --> VERIFY_STEP
    T4 --> VERIFY_STEP
    T5 --> VERIFY_STEP

    VERIFY_STEP{"Was report_type sent in request?"}
    VERIFY_STEP -->|"Yes"| VALID_AGENT["✅ Validation Agent checks\nDoes uploaded doc match expected type?\nblood report vs lab_report returns true"]
    VERIFY_STEP -->|"No"| SKIP_VERIFY["Skip verification — only doc_type returned"]
    
    VALID_AGENT --> IS_VERIFIED["Returns is_verified: true or false"]
    
    IS_VERIFIED --> NAME_CHECK
    SKIP_VERIFY --> NAME_CHECK
    
    NAME_CHECK["👤 Step 3: Patient Name Verification\nMCP calls SQL: SELECT first_name FROM patient_profile WHERE user_id = X\nCompares with name found inside document"]
    NAME_CHECK --> NAME_RESULT["Returns name_verified: true or false"]
    
    NAME_RESULT --> SUMMARY["🧠 Step 4: AI Summary Generation\nGPT-4o-mini creates structured summary with sections:\nHospital + Patient Profile\nMedical Findings\nTreatments and Medications\nLab Values Table"]
    
    SUMMARY --> EMBED["Generate Vector Embedding\nAzure Ada-002 — 1536-dimensional vector"]
    
    EMBED --> SAVE["💾 Save to PostgreSQL\nfile_summaries table — summary and embeddings stored"]
    
    SAVE --> RESPONSE["📤 API Response\ndoc_type, is_verified, name_verified, summary"]

    style REJECT fill:#ffcdd2,stroke:#c62828
    style RESPONSE fill:#c8e6c9,stroke:#2e7d32
```

---

## 6. Feature: Pre-Consultation Chat

Before a patient's appointment, Aadhya acts like a smart nurse — gathering all the information the doctor needs through natural conversation.

```mermaid
sequenceDiagram
    participant P as 👤 Patient
    participant API as ⚡ FastAPI
    participant CREW as 🤖 Booking Agent
    participant MCP as 🔌 MCP Server
    participant DB as 🗄️ PostgreSQL
    participant MEM as 🧠 Session Memory

    P->>API: POST /orch with slot_id: abc123
    API->>MCP: Fetch booking details for slot_id
    MCP->>DB: SELECT service_name, slot_date FROM patient_booked_slot WHERE id = abc123
    DB-->>MCP: service: Botox, date: 15 Mar 2026
    MCP-->>API: Booking context returned
    
    API->>CREW: Start intake chat with booking context
    CREW-->>P: Hi! I see you have a Botox appointment on 15 March. What brings you in today?

    loop Conversation turns until all areas are covered
        P->>API: input message — e.g. I have persistent forehead lines
        API->>MEM: Retrieve session history
        MEM-->>API: Previous messages
        API->>CREW: Answer plus full history
        CREW-->>P: Next contextual question — MCQ or open text
        API->>MEM: Save this turn to history
    end

    Note over CREW: All areas covered\nChief complaint done\nPersonal history done\nFamily history done\nMedications done\nAllergies done\nLifestyle done

    CREW->>CREW: Generate narrative clinical summary
    Note over CREW: Priya Sharma, 26, presents with persistent\nforehead lines affecting confidence for 2 years...
    
    CREW->>MCP: Save summary to DB
    MCP->>DB: UPDATE patient_booked_slot SET summary = ... WHERE id = abc123
    CREW-->>P: status: end — summary saved
```

---

## 7. Feature: Follow-Up Care Chat

After a patient completes treatment, Aadhya automatically checks in with treatment-specific questions.

```mermaid
flowchart TD
    TRIGGER["🔔 Follow-Up Triggered\nfollowUp: true, slot_id, consultationDate, followupDate"]

    TRIGGER --> FETCH["🔌 MCP fetches slot details\nSELECT service_name, summary FROM patient_booked_slot WHERE id = slot_id"]

    FETCH --> SERVICE{"Which treatment was done?"}
    SERVICE -->|"Botox"| BOT_Q["Questions about:\nResult satisfaction\nSide effects — bruising, swelling\nDuration of effect\nNeed for touch-up\nNatural appearance achieved"]
    SERVICE -->|"Hair Transplant"| HAIR_Q["Questions about:\nNew hair growth observed\nScalp health and healing\nItching or infection signs\nProgress vs expectations\nNext PRP session needed"]
    SERVICE -->|"Laser Treatment"| LASER_Q["Questions about:\nSkin sensitivity post-laser\nPigmentation changes\nSPF routine follow-through\nSession results satisfaction"]
    SERVICE -->|"Any other service"| GEN_Q["General outcome questions based on procedure"]

    BOT_Q --> CHAT["💬 Multi-turn conversation\nPatient answers naturally\nAadhya remembers all previous answers\nin this follow-up session"]
    HAIR_Q --> CHAT
    LASER_Q --> CHAT
    GEN_Q --> CHAT

    CHAT --> SAVE["💾 Save all responses to DB for doctor review"]
    SAVE --> DONE["✅ Follow-up complete"]

    style DONE fill:#c8e6c9,stroke:#2e7d32
```

---

## 8. Feature: Treatment Planner — Doctor Tool

A doctor types free-form notes after a consultation. Aadhya converts them into a fully verified, structured treatment plan with real database IDs and pricing.

```mermaid
flowchart TD
    NOTES["📝 Doctor Free-Form Notes\nHair Transplant 2600 grafts, PRP 3 sessions,\nCBC test, Metformin 500mg twice daily"]

    NOTES --> PARSE["🤖 Treatment Planner Agent\nStep 1: Parse and identify all items"]
    PARSE --> ITEMS{"Categorise each item"}
    
    ITEMS -->|"Services"| SVC_VERIFY["🔌 MCP to PostgreSQL\nSELECT id AS service_onboarding_id, service_name\nFROM service_onboarding_details\nWHERE service_name ILIKE '%Hair Transplant%'\nLIMIT 5"]
    
    ITEMS -->|"Products or Medications"| PROD_VERIFY["🔌 MCP to PostgreSQL\nSELECT id, product_name, product_price, discount_price\nFROM product\nWHERE product_name ILIKE '%Metformin%'\nLIMIT 5"]
    
    ITEMS -->|"Lab Tests"| LAB_VERIFY["🔌 MCP to PostgreSQL\nSELECT id AS diagnosis_id, test_name, price\nFROM diagnostic_services\nWHERE test_name ILIKE '%CBC%'\nLIMIT 5"]

    SVC_VERIFY -->|"Found in DB"| SVC_OK["✅ verified: true — service_onboarding_id returned"]
    SVC_VERIFY -->|"Not in DB"| SVC_MISS["⚠️ verified: false — still included, clearly flagged"]

    PROD_VERIFY -->|"Found"| PROD_OK["✅ verified: true — product_id, MRP_cost, cost returned"]
    PROD_VERIFY -->|"Not in DB"| PROD_MISS["⚠️ verified: false — still included, clearly flagged"]

    LAB_VERIFY -->|"Found"| LAB_OK["✅ verified: true — diagnosis_id, price returned"]
    LAB_VERIFY -->|"Not in DB"| LAB_MISS["⚠️ verified: false — still included, clearly flagged"]

    SVC_OK --> STRUCTURE
    SVC_MISS --> STRUCTURE
    PROD_OK --> STRUCTURE
    PROD_MISS --> STRUCTURE
    LAB_OK --> STRUCTURE
    LAB_MISS --> STRUCTURE

    STRUCTURE["📋 Step 2: Structure into Plans — as many as notes imply"]
    
    STRUCTURE --> PLAN_A["Plan A — Primary Treatment\nHair Transplant 2600 grafts\nservice_onboarding_id: abc\nverified: true"]
    STRUCTURE --> PLAN_B["Plan B — Alternative\nPRP 3 Sessions\nservice_onboarding_id: xyz\nverified: true"]
    STRUCTURE --> PLAN_C["Plan C — Medications and Tests\nMetformin 500mg BD — product_id, MRP, cost\nCBC — diagnosis_id, price"]

    PLAN_A --> JSON_OUT["📤 Pure JSON output — no markdown, no extra text"]
    PLAN_B --> JSON_OUT
    PLAN_C --> JSON_OUT

    style SVC_OK fill:#c8e6c9,stroke:#2e7d32
    style PROD_OK fill:#c8e6c9,stroke:#2e7d32
    style LAB_OK fill:#c8e6c9,stroke:#2e7d32
    style SVC_MISS fill:#fff9c4,stroke:#f9a825
    style PROD_MISS fill:#fff9c4,stroke:#f9a825
    style LAB_MISS fill:#fff9c4,stroke:#f9a825
    style JSON_OUT fill:#c8e6c9,stroke:#2e7d32
```

---

## 9. Feature: Product & Prescription Validation

When a patient wants to buy a product, the system checks their uploaded prescription to verify they are authorised to take it.

```mermaid
flowchart TD
    REQ["📋 Request\nproducts: Metformin, Vitamin D\nfile_urls: prescription.pdf\nvalidation: true"]

    REQ --> FILE_READ["📄 File Processor reads prescription\nPDF text or Vision AI"]
    FILE_READ --> EXTRACT["Extract medication list from document"]

    EXTRACT --> PER_PRODUCT{"For each product requested"}

    PER_PRODUCT --> CHECK_PRES{"Is product name in prescription text?"}
    CHECK_PRES -->|"Yes"| DB_CHECK["🔌 MCP to PostgreSQL\nSELECT rx_required FROM product\nWHERE product_name ILIKE '%Metformin%'"]
    CHECK_PRES -->|"No"| CAN_TAKE_F["can_take: false — not prescribed"]
    
    DB_CHECK -->|"rx_required = false"| CAN_TAKE_T["can_take: true — no prescription needed"]
    DB_CHECK -->|"rx_required = true and in prescription"| CAN_TAKE_T2["can_take: true — prescribed and authorised"]
    DB_CHECK -->|"rx_required = true and NOT in prescription"| CAN_TAKE_F2["can_take: false — prescription required but not found"]

    CAN_TAKE_T --> RESP["📤 Per-product JSON response\nMetformin: can_take true\nVitamin D: can_take true"]
    CAN_TAKE_T2 --> RESP
    CAN_TAKE_F --> RESP
    CAN_TAKE_F2 --> RESP

    style CAN_TAKE_T fill:#c8e6c9,stroke:#2e7d32
    style CAN_TAKE_T2 fill:#c8e6c9,stroke:#2e7d32
    style CAN_TAKE_F fill:#ffcdd2,stroke:#c62828
    style CAN_TAKE_F2 fill:#ffcdd2,stroke:#c62828
```

---

## 10. Feature: Personalised Recommendations

Aadhya proactively recommends services and products tailored to each patient's complete history.

```mermaid
flowchart TD
    TRIGGER["⭐ recommendations: true — user_id: patient_001"]

    subgraph DATA_PULL["📊 Pull All Patient Data"]
        DEMO["🔌 MCP to patient_profile\nSELECT gender, date_of_birth WHERE id = patient_001"]
        HISTORY["🔌 MCP to patient_booked_slot\nSELECT service_name, summary\nWHERE patient_id = patient_001 ORDER BY slot_date DESC LIMIT 10"]
        FILES["🔌 MCP to file_summaries\nSELECT summary WHERE user_id = patient_001 LIMIT 10"]
    end

    TRIGGER --> DEMO
    TRIGGER --> HISTORY
    TRIGGER --> FILES

    DEMO --> ANALYSE
    HISTORY --> ANALYSE
    FILES --> ANALYSE

    ANALYSE["🤖 Recommendation Agent — analyses all data together"]

    ANALYSE --> SVC_MATCH["Match to services catalogue\nBased on: gender, past services, clinical notes"]
    ANALYSE --> PROD_MATCH["Match to products catalogue\nBased on: treatment history and health conditions"]

    SVC_MATCH --> SVC_RECS["📋 Service Recommendations\nBotox touch-up — high priority\nChemical Peel — medium priority"]
    PROD_MATCH --> PROD_RECS["💊 Product Recommendations\nSPF 50 sunscreen — reason: post-laser\nVitamin C serum — reason: pigmentation"]

    SVC_RECS --> OUT["📤 JSON with service and product recommendations\neach with reason and priority level"]
    PROD_RECS --> OUT

    style OUT fill:#c8e6c9,stroke:#2e7d32
```

### 10a. Skin Profile Product Recommendations

```mermaid
flowchart TD
    SKIN_IN["🧴 skinValues free-form text input\noily skin, moderate acne, low budget, no fragrance"]
    
    SKIN_IN --> PARSE_SKIN["🤖 Skin Profile Agent — parse the text"]
    PARSE_SKIN --> EXTRACT_FIELDS["Extract structured fields\nai_skin_type: oily\nai_acne_severity: 3-moderate\nuser_constraints: no-fragrance, low-budget\nuser_concern: acne, oil-control"]

    EXTRACT_FIELDS --> BUILD_SQL["Build targeted SQL query\nSELECT recommended_products FROM skin_profiles\nWHERE ai_skin_type ILIKE '%oily%'\nAND ai_acne_severity ILIKE '%moderate%'\nAND user_concern ILIKE '%acne%'\nLIMIT 5"]

    BUILD_SQL --> MCP_CALL["🔌 MCP Server to PostgreSQL"]
    MCP_CALL --> RESULTS["Matching skin profile rows returned"]
    RESULTS --> TOP3["📤 Top 3 matching products with reasons"]

    style TOP3 fill:#c8e6c9,stroke:#2e7d32
```

---

## 11. Feature: Doctor–Patient Overview

Doctors can ask natural questions about any specific patient and get clear, clinical-prose answers.

```mermaid
sequenceDiagram
    participant DOC as 👨‍⚕️ Doctor
    participant API as ⚡ FastAPI
    participant POV as 🤖 Patient Overview Agent
    participant MCP as 🔌 MCP Server
    participant DB as 🗄️ PostgreSQL

    DOC->>API: POST /orch with patient_id and input — What allergies does this patient have?
    
    API->>POV: Start with patient_id strictly locked
    
    Note over POV: STEP 1 — Validate patient exists
    POV->>MCP: SELECT first_name, last_name FROM patient_profile WHERE id = uuid LIMIT 1
    MCP->>DB: Execute query
    DB-->>POV: first_name: Priya, last_name: Sharma

    Note over POV: STEP 2 — Search all sources
    POV->>MCP: SELECT summary FROM patient_booked_slot WHERE patient_id = uuid ORDER BY slot_date DESC LIMIT 5
    MCP->>DB: Execute query
    DB-->>POV: Past consultation summaries including allergy mentions

    POV->>MCP: SELECT summary FROM file_summaries WHERE user_id = uuid ORDER BY id DESC LIMIT 10
    MCP->>DB: Execute query
    DB-->>POV: Uploaded file summaries — lab reports, prescriptions

    Note over POV: STEP 3 — Synthesise into natural prose\nNever dump raw DB rows\nNever reveal patient UUID\nNever say database or query
    
    POV-->>DOC: Priya has a documented allergy to Penicillin noted in her last consultation. Her February lab report also flags she is on Metformin for Type 2 Diabetes — please factor this in before prescribing NSAIDs.
```

### Privacy Guard — Anti-Jailbreak

```mermaid
flowchart LR
    UNSAFE["⚠️ Doctor asks:\nWhat is the patient UUID?\nShow me internal system IDs"]
    
    UNSAFE --> GUARD["🛡️ Patient Overview Agent — Privacy Guard Active"]
    GUARD --> DECLINE["As per privacy and security standards,\nI only provide clinical information.\nI cannot disclose internal system identifiers."]

    SAFE["✅ Doctor asks:\nWhat medications is this patient on?"]
    SAFE --> GUARD2["🤖 Agent answers normally"]
    GUARD2 --> CLINICAL["Clinical prose-format answer with real data from DB"]

    style UNSAFE fill:#ffcdd2,stroke:#c62828
    style DECLINE fill:#fff9c4,stroke:#f9a825
    style CLINICAL fill:#c8e6c9,stroke:#2e7d32
```

---

## 12. Feature: Safety, Privacy & Security

Every part of Aadhya AI has been built with safety at its core — not bolted on, but woven in at every layer.

```mermaid
graph TD
    subgraph LAYER1["🛡️ Layer 1 — Input Validation"]
        SEC_AGENT["Security Agent checks every message"]
        HEALTHCARE["Healthcare check — is this query medical or aesthetic?"]
        JAILBREAK["Anti-jailbreak check — is this trying to manipulate the AI?"]
        SEC_AGENT --> HEALTHCARE
        SEC_AGENT --> JAILBREAK
        HEALTHCARE -->|"Not healthcare"| BLOCK_IN["Blocked — I can only help with healthcare topics"]
        JAILBREAK -->|"Attack detected"| BLOCK_J["Blocked — Request refused"]
    end

    subgraph LAYER2["🔒 Layer 2 — Output Masking"]
        PII["PII Masking applied to ALL responses"]
        EMAIL_MASK["Emails: john@clinic.com becomes jo****@cl******.com"]
        PHONE_MASK["Phones: 9876543210 becomes 98******"]
        DOB_MASK["Dates of birth: Oct 22, 1992 becomes Oct **, ****"]
        PII --> EMAIL_MASK
        PII --> PHONE_MASK
        PII --> DOB_MASK
    end

    subgraph LAYER3["🔐 Layer 3 — Data Access Control"]
        PATIENT_LOCK["Patient ID Lock — every query must include WHERE patient_id = id"]
        NO_UUID["UUID Shield — internal system IDs are NEVER shown to users"]
        PRIVACY_GUARD["Anti-jailbreak guard in Patient Overview Agent"]
    end

    subgraph LAYER4["⚡ Layer 4 — Concurrency Safety"]
        THREAD_SAFE["Thread-Safe Session Management — RLocks protect all shared dictionaries"]
        PER_REQ["Per-Request Crew Isolation — each API call gets its own agent crew"]
        POOL_INFO["Connection Pool — PostgreSQL max 10, Thread pool max 50 workers"]
    end

    subgraph LAYER5["🗄️ Layer 5 — Database Safety"]
        SQL_GUARD["SELECT only — AI agents cannot INSERT, UPDATE or DELETE"]
        LIMIT_RULE["LIMIT on all queries — prevents bulk data dumps"]
        NO_EMBED["Embedding columns never selected — saves cost and bandwidth"]
    end

    LAYER1 --> LAYER2
    LAYER2 --> LAYER3
    LAYER3 --> LAYER4
    LAYER4 --> LAYER5

    style BLOCK_IN fill:#ffcdd2,stroke:#c62828
    style BLOCK_J fill:#ffcdd2,stroke:#c62828
    style LAYER1 fill:#fce4ec,stroke:#c62828
    style LAYER2 fill:#e8eaf6,stroke:#3F51B5
    style LAYER3 fill:#e0f2f1,stroke:#00796B
    style LAYER4 fill:#fff8e1,stroke:#F57F17
    style LAYER5 fill:#e8f5e9,stroke:#388E3C
```

---

## 13. What Has Been Built — Feature Completion

Every feature listed below is **live, tested, and production-ready** as of March 2026.

| # | Feature | Status | What It Does |
|---|---|---|---|
| 1 | General Chat — Patient Mode | ✅ Live | AADHYA persona, DB search, RAG fallback, Mem0 memory |
| 2 | General Chat — Doctor Mode | ✅ Live | Professional persona, clinical depth, multi-source search |
| 3 | Healthcare Detection | ✅ Live | 120+ keyword filter — blocks non-medical uploads instantly at zero AI cost |
| 4 | Report Type Identification | ✅ Live | AI classifies into 5 document categories automatically |
| 5 | Report Type Verification | ✅ Live | Validates that the uploaded document matches the caller's expected report type |
| 6 | Patient Name Verification | ✅ Live | Prevents submitting someone else's reports using DB name match |
| 7 | PDF Text Extraction + Summary | ✅ Live | Full structured AI summary saved with vector embeddings to DB |
| 8 | Scanned PDF OCR + Summary | ✅ Live | OCR via pytesseract reads handwritten or scanned documents |
| 9 | Image Vision + Summary | ✅ Live | GPT-4o Vision reads image-only medical files |
| 10 | Pre-Consultation Chat | ✅ Live | Smart nurse intake — all key clinical areas covered naturally |
| 11 | Clinical Summary Generation | ✅ Live | Narrative prose summary saved to appointment record |
| 12 | Follow-Up Care Chat | ✅ Live | Treatment-specific post-care questions with multi-turn memory |
| 13 | Treatment Planner — Services | ✅ Live | Verifies services against DB, returns service_onboarding_id |
| 14 | Treatment Planner — Products | ✅ Live | Verifies products, returns product_id, MRP, and discount price |
| 15 | Treatment Planner — Lab Tests | ✅ Live | Verifies tests, returns diagnosis_id and price |
| 16 | Prescription Validation | ✅ Live | Per-product can_take: true or false based on uploaded prescription |
| 17 | Personalised Recommendations | ✅ Live | Services and products recommended based on full patient history |
| 18 | Skin Profile Recommendations | ✅ Live | Top 3 products matched to AI-derived skin scores |
| 19 | Patient Overview Chatbot | ✅ Live | Doctor asks questions in plain language, strictly locked to one patient |
| 20 | Long-Term Memory — Mem0 | ✅ Live | Per-user vector memory backed by Qdrant, survives session restarts |
| 21 | PII Masking and Privacy | ✅ Live | Auto-masks emails, phone numbers, and dates of birth in all responses |

---

> *Document prepared by the AI Engineering Team · AestheticQ · March 2026*
>
> *All 21 features are production-ready and tested. Architecture follows enterprise standards for concurrency, privacy, and clinical safety.*
