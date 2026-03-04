# 🏥 Aadhya AI — Product Scope & Feature Reference

> **Aadhya** is an intelligent aesthetics & healthcare AI assistant powering the end-to-end clinical journey — from a patient's first query to a doctor's treatment plan — with memory, safety, and real-time database verification built in.

---

## 1. 💬 General Chat (Ask Anything)

Patients and doctors can have a natural, ongoing conversation with Aadhya. The assistant knows who it's talking to and adjusts its tone and depth accordingly.

**What it does:**

- Answers questions about **aesthetics services, treatments, skin care, medications, and symptoms** in plain language
- Searches the clinic's live **doctor & service database** to give accurate, real-time answers — no hallucination
- Recommends specific doctors by specialty and city when asked (e.g. *"Find a dermatologist in Chennai"*)
- Pulls from a **RAG knowledge base** for in-depth medical questions; falls back to internet search for general research
- **Doctor mode** — a professional medical assistant persona for evidence-based clinical answers
- **Patient mode (AADHYA)** — a friendly, empathetic persona that encourages booking and guides aesthetic goals
- All personally identifiable information (emails, phone numbers) in responses is **automatically masked**
- Maintains full **session memory** — Aadhya remembers what was said earlier in the same conversation via Mem0

**Testable scenarios:**

| # | Scenario |
|---|---|
| T1 | Ask about an aesthetic service → should return DB results, never hallucinate |
| T2 | Ask for a doctor in a city → should return real doctors from DB |
| T3 | Ask a general medical question → should use RAG / internet |
| T4 | Say "Hi" → should get AADHYA introduction |
| T5 | Ask same question twice in a session → Aadhya should remember context |
| T6 | Response containing phone/email → should be masked in output |

---

## 2. 📄 File Upload & Document Processing

Users can upload medical documents (PDFs or images) and Aadhya reads, interprets, and stores them intelligently. This module contains **four distinct sub-features**, each independently testable.

---

### 2a. 🔍 Healthcare Detection (Keyword-Based Fast Filter)

**What it does:**

- Before any AI analysis, the system scans the extracted text for **120+ predefined healthcare keywords** (e.g. blood, prescription, diagnosis, hemoglobin, botox, ultrasound, patient, clinic…)
- If **at least one keyword matches**, the document is treated as healthcare-related and processing continues
- If **no keywords match**, the document is immediately rejected — no AI call is made, no data is saved
- Eliminates ~16 seconds of latency for non-medical uploads

**Testable scenarios:**

| # | Input | Expected Output |
|---|---|---|
| T1 | Upload a medical lab report | Detected as healthcare → processed fully |
| T2 | Upload a food menu or random PDF | Rejected instantly with "not healthcare-related" message |
| T3 | Upload an image of a prescription | Keywords found in OCR/vision output → processed |
| T4 | Upload a blank or near-empty PDF | No keywords → rejected |

---

### 2b. 🏷️ Report Type Identification & Verification

**How it works — two distinct steps:**

#### Step 1 — Automatic Type Identification (always runs)
Every healthcare document that passes the keyword filter is **automatically classified** by the AI into one of exactly **5 fixed categories**:

| `doc_type` value | What it means | Real-world examples |
|---|---|---|
| `prescription` | A doctor's written prescription for medicines | Doctor Rx slip, e-prescription, chemist prescription |
| `lab_report` | Any pathology, blood, or diagnostic lab result | **CBC / Complete Blood Count**, **Blood Sugar (HbA1c, FBS, PPBS)**, **Lipid Profile**, **Liver Function Test (LFT)**, **Kidney Function Test (KFT/RFT)**, **Urine Routine**, **Thyroid Profile (TSH, T3, T4)**, **Vitamin D / B12 / Iron levels**, **Hemoglobin**, **BMI Report (if from a diagnostic lab)** |
| `diagnosis` | A formal diagnosis or clinical assessment note | Dermatologist diagnosis note, discharge summary with confirmed diagnosis |
| `medical_history` | A record of past conditions and health background | Patient medical history card, case summary from a previous hospital, OPD card |
| `general` | Any healthcare document that doesn't fit the above | Consent forms, insurance documents, vaccination certificates, wellness reports |

> ℹ️ The `doc_type` is always returned in the API response under `processed_files[].doc_type`, regardless of whether `report_type` verification was requested.

---

#### Step 2 — Report Type Verification (only runs when `report_type` is passed)

When the calling system sends a `report_type` parameter (a **free-text string** describing what the document is expected to be), the system activates the **Validation Checker agent** to compare the expected type against the identified `doc_type` and the document's content/summary.

- The caller passes `report_type` as any natural-language string — e.g. `"blood report"`, `"CT scan"`, `"BP report"`, `"BMI report"`, `"CBC test"`, `"prescription"`
- The Validation Checker agent reads the `doc_type` + the AI-generated summary and determines whether the uploaded document is a reasonable match for what was requested
- Returns `"is_verified": true` or `"is_verified": false` in the response

**How the matching works (AI-based, not keyword-exact):**

| `report_type` sent by caller | Uploaded document | `is_verified` result |
|---|---|---|
| `"prescription"` | Prescription slip | `true` |
| `"blood report"` | CBC / Blood count report | `true` |
| `"BMI report"` | Lab report with weight/height/BMI values | `true` |
| `"BP report"` | Blood pressure monitoring chart | `true` |
| `"CT scan"` | CT scan report / radiology report | `true` |
| `"city scan"` (typo for CT scan) | CT scan report | `true` (AI understands intent) |
| `"thyroid report"` | TSH/T3/T4 lab report | `true` |
| `"HbA1c"` | Blood sugar lab report | `true` |
| `"blood report"` | Prescription slip | `false` (doc_type mismatch) |
| `"CT scan"` | Liver Function Test report | `false` |
| `"prescription"` | Medical history card | `false` |
| `"lab report"` | Random consent form | `false` |

---

**Testable scenarios:**

| # | `report_type` sent | Document uploaded | `doc_type` identified | `is_verified` |
|---|---|---|---|---|
| T1 | `"prescription"` | Doctor's Rx slip | `prescription` | `true` |
| T2 | `"blood report"` | CBC / Blood Count lab report | `lab_report` | `true` |
| T3 | `"BMI report"` | BMI measurement report from diagnostic lab | `lab_report` | `true` |
| T4 | `"BP report"` | Blood pressure monitoring record | `lab_report` | `true` |
| T5 | `"CT scan"` | CT scan radiology report | `diagnosis` or `lab_report` | `true` |
| T6 | `"city scan"` (common voice/typo) | CT scan report | `lab_report` | `true` (AI understands context) |
| T7 | `"thyroid report"` | TSH / T3 / T4 test result | `lab_report` | `true` |
| T8 | `"sugar report"` | HbA1c / Blood glucose test | `lab_report` | `true` |
| T9 | `"blood report"` | Prescription slip | `prescription` | `false` (wrong doc type) |
| T10 | `"CT scan"` | LFT (Liver Function Test) | `lab_report` | `false` (different scan) |
| T11 | `"prescription"` | Medical history card | `medical_history` | `false` |
| T12 | *(not sent / null)* | Any document | `lab_report` / etc. | No `is_verified` field in response |
| T13 | `"lab report"` | Non-healthcare document (e.g. food invoice) | Rejected at Step 1 | File rejected before verification runs |

---

**What the API response looks like when `report_type` is passed:**

```json
{
  "processed_files": [{
    "file_url": "...",
    "success": true,
    "file_type": "pdf",
    "is_healthcare_related": true,
    "summary": "...",
    "description": "...",
    "doc_type": "lab_report",
    "is_verified": true,
    "name_verified": true
  }]
}
```

> ⚠️ `is_verified` only appears in the response when `report_type` was part of the request. If `report_type` is omitted, `is_verified` is not present — only `doc_type` is.

---

### 2c. 👤 File Name / Patient Name Verification

**What it does:**

- When a file is uploaded, the system checks if the **patient's name mentioned inside the document** matches the **account holder's name** in the database
- Looks up the patient profile via `user_id` and compares (case-insensitive, ignoring spaces)
- Returns `true` if the name matches, `false` if it does not match or no name is found in the document
- Designed to prevent patients from submitting someone else's reports

**Testable scenarios:**

| # | Input | Expected Output |
|---|---|---|
| T1 | Upload a report containing the same name as the account | Name verification returns `true` |
| T2 | Upload a report with a different patient's name | Name verification returns `false` |
| T3 | Upload a report with no patient name on it | Returns `false` (name not found) |
| T4 | Upload with a user_id that does not exist in DB | Returns `false` |

---

### 2d. 🧠 AI Document Summary Generation

**What it does:**

- For healthcare-related documents, Aadhya generates a **structured, readable AI summary** using a unified medical prompt
- **For PDFs:** extracts text directly; if the PDF is scanned/image-only, runs **OCR (pytesseract)** first
- **For images:** uses **AI Vision** (GPT-4o) to read and interpret the image
- The summary follows a consistent structure:
  - 🏥 Hospital & Patient Profile
  - 🩺 Comprehensive Medical Summary (findings, diagnosis, clinical insights)
  - 💊 Treatment & Medications (each medicine with purpose)
  - 📊 Consolidated Data Table (lab values, vitals)
  - ⚠️ Medical Disclaimer
- If a field is missing or unreadable, it is **silently omitted** — no placeholders like "N/A" or "Unknown"
- The summary and full extracted content are saved to the patient's database record with **vector embeddings** for future RAG search

**Testable scenarios:**

| # | Input | Expected Output |
|---|---|---|
| T1 | Upload a clear typed prescription PDF | Summary shows doctor, medicines with purpose, clear structure |
| T2 | Upload a scanned/handwritten prescription (image PDF) | OCR runs, summary still generated |
| T3 | Upload a lab report image (JPG) | Vision API reads it, lab values appear in data table |
| T4 | Document has no patient name | Name section silently omitted — no "N/A" shown |
| T5 | Upload same file twice | Both saved; both searchable via RAG |

---

## 3. 🩺 Pre-Consultation Chat

Before a patient's scheduled appointment, Aadhya conducts a dynamic conversational intake assessment — like a smart nurse gathering information for the doctor.

**What it does:**

- Triggered by a **booking slot ID** tied to a real, confirmed appointment
- Greets the patient and collects information through natural **conversation**, not a form
- Covers: chief complaint, personal medical history, family history, surgical history, medications, allergies, lifestyle, and social context
- Uses **multiple-choice questions (MCQ)** for structured topics, plain text for open-ended ones
- Context tags (`[CONTEXT: FAMILY_HISTORY]`, `[CONTEXT: PERSONAL_HISTORY]`) ensure answers are attributed correctly
- Tracks full conversation history — patient never needs to repeat themselves
- Automatically detects when all key areas are covered and **gracefully closes the chat**
- On completion, generates a **narrative clinical summary** (flowing paragraphs — not a form) and saves it to the appointment record

**Summary style (what the doctor reads):**

> *"Priya Sharma, a 26-year-old woman, presents with a primary concern of **persistent acne** affecting her confidence over the past year. She has a known personal history of **hypertension** currently managed with medication. Her family history is notable for **Type 2 Diabetes** on the maternal side. She reports no prior surgeries and was not willing to disclose alcohol consumption habits."*

**Testable scenarios:**

| # | Scenario | Expected Output |
|---|---|---|
| T1 | Start chat with valid slot_id | Aadhya greets and begins intake |
| T2 | Answer with "Skip" | Summary says "Patient was not willing to disclose X" |
| T3 | Answer with "Yes" to a condition question | Condition confirmed in summary |
| T4 | Complete all questions | Status becomes `end`, summary saved to DB |
| T5 | Read the saved summary | Reads as clinical narrative, NOT a form |
| T6 | Answer family history after personal history question | Context tags ensure correct attribution |

---

## 4. 🔄 Follow-Up Care Chat

After a patient completes a treatment, Aadhya handles automated check-in conversations.

**What it does:**

- Triggered by follow-up date and the completed appointment's slot
- Generates **treatment-specific questions** — different questions for Botox vs. Hair Transplant vs. Laser
- Covers: treatment results, side effects, satisfaction, progress toward goals, need for more sessions
- Conducts the check-in as a natural conversation, tracking all responses over multiple turns
- All responses stored for the doctor's review

**Testable scenarios:**

| # | Scenario | Expected Output |
|---|---|---|
| T1 | Follow-up for Botox appointment | Questions relevant to Botox results and side effects |
| T2 | Follow-up for Hair Transplant | Questions about hair growth, scalp condition |
| T3 | Multi-turn response | Aadhya remembers previous answers in same session |

---

## 5. 💊 Treatment Planner (Doctor Tool)

The doctor types natural post-consultation notes and Aadhya converts them into a verified, structured treatment plan.

**What it does:**

- Input: free-form clinical notes (e.g. *"Hair Transplant 2600 grafts, PRP 3 sessions, CBC test, Metformin 500mg twice daily"*)
- Aadhya **parses and extracts**: services, medications/products, and lab tests mentioned
- Each item is **verified against the live database**:
  - Services → `service_onboarding_details` (service ID returned)
  - Products → `product` catalogue (product ID, MRP, and discounted price returned)
  - Lab Tests → `diagnostic_services` (test ID and price returned)
- Outputs **Plan A** (primary treatment), **Plan B** (alternative), **Plan C** (medications/tests) — or however many plans the notes imply
- Each item shows: name, database ID, verification status (`verified` / `not_found`), dosage, frequency, and cost
- Items not found in DB are still included but clearly **flagged as unverified**
- Output is clean JSON — no markdown, no extra text

**Testable scenarios:**

| # | Input | Expected Output |
|---|---|---|
| T1 | Notes with a known service (e.g. "Laser Hair Removal") | `verified: true`, service_onboarding_id returned |
| T2 | Notes with an unknown service | `verified: false`, still included with name |
| T3 | Notes with a product/medication | Product ID, MRP, discount price returned |
| T4 | Notes with a lab test (e.g. "CBC") | diagnosis_id and price returned |
| T5 | Notes with multiple treatments | Multiple plans generated (A, B, C…) |
| T6 | Notes with only one treatment plan | Single plan returned, not forced to 3 |

---

## 6. ✅ Product & Prescription Validation

When a patient wants to purchase a product, the system checks if they have a valid prescription for it.

**What it does:**

- Takes a list of **product names** and an uploaded **prescription file**
- Reads the prescription document (PDF or image) and extracts the medication names listed
- Checks each product against what's in the prescription → returns `can_take: true/false` per product
- Also checks the database `rx_required` flag — if the product needs a prescription and none is provided, it is flagged
- Returns structured JSON — the frontend can use this to block or allow checkout

**Testable scenarios:**

| # | Input | Expected Output |
|---|---|---|
| T1 | Product present in the uploaded prescription | `can_take: true` |
| T2 | Product NOT in the prescription | `can_take: false` |
| T3 | Product with `rx_required = true`, no prescription uploaded | Flagged as not authorised |
| T4 | Product with `rx_required = false` | Allowed regardless of prescription |
| T5 | Multiple products, some matching, some not | Per-product `can_take` in response |

---

## 7. 🌟 Personalised Recommendations

Aadhya proactively recommends services and products tailored to each patient's profile and history.

**What it does:**

- Pulls the patient's **demographic profile** (age, gender) and matches it to age/gender-appropriate services
- Reviews the patient's full **booking history** — past services and consultation summaries
- Reviews all **uploaded documents** — lab reports, prescriptions, treatment history
- Recommends:
  - **Services** — complementary treatments, maintenance sessions, new services relevant to their history
  - **Products** — products that support their treatment or documented health profile
- Each recommendation includes a **reason** and a **priority level** (high / medium / low)

**Testable scenarios:**

| # | Scenario | Expected Output |
|---|---|---|
| T1 | Patient who had Botox → request recommendations | Suggests Botox touch-up, complementary skin services |
| T2 | Female patient profile | Gender-appropriate services returned |
| T3 | Patient with uploaded lab showing high cholesterol | Relevant health-aware recommendations |

### 7a. Skin Profile Product Recommendations

- Accepts AI skin assessment scores (acne severity, redness, pigmentation, hydration, skin type, concerns/constraints)
- Matches to **pre-configured skin profiles** in the database
- Returns the **top 3 matching products** with reasons

**Testable scenarios:**

| # | Input | Expected Output |
|---|---|---|
| T1 | Oily skin + moderate acne + low budget | Top 3 products matching all three filters |
| T2 | Dry-sensitive skin + pigmentation concern | Products relevant to pigmentation + hydration |

---

## 8. 👨‍⚕️ Doctor–Patient Overview (Doctor Tool)

Doctors can ask questions about any specific patient in plain language and get real, data-backed answers.

**What it does:**

- All queries are **strictly locked to one patient** — doctor provides patient ID, no other patient's data is ever returned
- Doctor asks natural language questions: *"What allergies does this patient have?"*, *"What was discussed last time?"*
- Aadhya searches across all sources: **patient profile, all past consultations, and all uploaded file summaries**
- Answers are written as **natural clinical prose** — no raw DB output, no technical jargon
- Patient UUIDs and internal system IDs are **never revealed**, even if asked

**Testable scenarios:**

| # | Scenario | Expected Output |
|---|---|---|
| T1 | Ask about valid patient's allergies | Returns allergies from profile + consultation notes |
| T2 | Ask about an invalid patient ID | "No patient found" — no data returned |
| T3 | Ask for patient's internal UUID | Politely declined — privacy guard active |
| T4 | Ask about a different patient mid-session | Remains locked to original patient_id |
| T5 | Ask about documents uploaded by patient | Lists file summaries for that patient only |

---

## 9. 🔒 Safety, Privacy & Memory

| Feature | What it does | How to test |
|---|---|---|
| **PII Masking** | Emails and phone numbers in AI responses are masked automatically | Include PII in a DB record → verify response masks it |
| **Healthcare Guard** | Security validation layer blocks non-healthcare queries (configurable) | Send a cooking recipe question → should be rejected |
| **Anti-Jailbreak** | Patient Overview agent blocks attempts to extract system info or IDs | Ask for patient's UUID → declined with privacy message |
| **Long-Term Memory** | Aadhya remembers past sessions per patient via Mem0 vector memory | Ask about something from a previous session → recalled correctly |
| **In-Session Context** | Full chat history maintained within a session | Refer to an earlier answer → Aadhya knows it |
| **Concurrent Users** | Thread-safe — unlimited simultaneous users with per-request isolation | Run parallel requests — no data cross-contamination |

---

## ✅ Feature Completion Summary

| # | Feature | Status |
|---|---|---|
| 1 | General Chat — Patient Mode (AADHYA) | ✅ Complete |
| 2 | General Chat — Doctor Mode | ✅ Complete |
| 3 | File Upload — Healthcare Detection (Keyword) | ✅ Complete |
| 4 | File Upload — Report Type Identification | ✅ Complete |
| 5 | File Upload — Report Type Verification (vs. expected type) | ✅ Complete |
| 6 | File Upload — Patient Name Verification | ✅ Complete |
| 7 | File Upload — PDF Text Extraction & AI Summary | ✅ Complete |
| 8 | File Upload — Scanned PDF OCR + Summary | ✅ Complete |
| 9 | File Upload — Image (Vision AI) + Summary | ✅ Complete |
| 10 | Pre-Consultation Chat (MCQ + Text) | ✅ Complete |
| 11 | Pre-Consultation Clinical Summary (Narrative) | ✅ Complete |
| 12 | Follow-Up Care Chat | ✅ Complete |
| 13 | Treatment Planner — Services Verification | ✅ Complete |
| 14 | Treatment Planner — Products + Pricing Verification | ✅ Complete |
| 15 | Treatment Planner — Lab Tests Verification | ✅ Complete |
| 16 | Product & Prescription Validation (can_take) | ✅ Complete |
| 17 | Personalised Service & Product Recommendations | ✅ Complete |
| 18 | Skin Profile Product Recommendations | ✅ Complete |
| 19 | Doctor–Patient Overview Chatbot | ✅ Complete |
| 20 | Long-Term Memory (Mem0) | ✅ Complete |
| 21 | PII Masking & Privacy | ✅ Complete |

---

*Document version: March 2026 · Aadhya AI System*
