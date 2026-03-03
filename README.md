# Pre-Consultation Chart Test Report

## 1. Overview & Scope

This report documents the functional validation of the **AADHYA Healthcare AI System's Pre-Consultation Workflow**.

**Scope of Testing:**

* **Greeting & Context:** Verification of appointment details (Doctor, Service, Date, Time) in the initial greeting.
* **Conversational Logic:** Assessment of the 26-step intake flow, including multi-choice questions (MCQ), free-text inputs, and "Others" clarification triggers.
* **Input Validation:** Testing of range checks (e.g., body weight) and handling of irrelevant or gibberish inputs.
* **Skip Logic:** Validation of the system's ability to bypass optional fields while maintaining data integrity.
* **Summary Generation:** Verification of the final medical summary based on the collected patient data.

**Appointment Context:**

* **Patient:** Mahesh
* **Doctor:** Dr. Jagadeesh
* **Service:** Acne Scar Treatment (Subcision, Nanofat Grafting, Morpheus, Laser)
* **Schedule:** December 8, 2025, at 11:00 AM

## 2. Pre-Consultation Chat (Exact 26-Step Flow)

This table tracks the exact user inputs and agent responses during the clinical intake process.

| Step | User Input | Exact Agent Response | Status | Finding/Observation | 
| :--- | :--- | :--- | :--- | :--- | 
| **1** | *(Init)* | "Hello **Mahesh**! I'm AADHYA, your Medical Intake Specialist. I'm here to gather some information before your appointment with Dr. Jagadeesh for your Acne Scar treatment on **December 8, 2025**, at **11:00 AM**. Let's get started! What is your main concern or complaint regarding your skin condition? Feel free to skip this if you'd rather not share." | ✅ PASS | Initial greeting correct. | 
| **2** | `idk` | "I understand that you might not want to share your main concern right now, Mahesh! Let's move on to the next question. What is your expected result from the treatment? Feel free to skip this if you prefer." | ✅ PASS | Preference to move forward accepted. | 
| **3** | `apple` | "Thank you for your input, **Mahesh**! It seems like that may not be related to your main concern. Let's focus on what you'd like to discuss regarding your acne scars. What is your main concern or complaint about them? 😊" | ✅ PASS | Irrelevant input correctly identified and re-asked. | 
| **4** | `bannana` | "Thank you for your input, **Mahesh**! It seems like that may not be related to your main concern. Let's focus on what you'd like to discuss regarding your acne scars. What is your main concern or complaint about them? 😊" | ✅ PASS | Relevance check remains consistent. | 
| **5** | `acne on my skin` | "Thank you for sharing that, **Mahesh**! It sounds like your main concern is the **acne on your skin**. Now, could you please tell me what your expected result from the treatment is? 😊" | ✅ PASS | Valid text accepted. | 
| **6** | `to rmove my acne` | "Thank you for sharing that, **Mahesh**! It sounds like you want to **remove your acne**. Now, let's talk about your medical conditions. Do you have any of the following? Please select all that apply: **Diabetes**, **Hypertension**, **Asthma**, **Hypothyroid**, **Seizures**, **Blood disorders**, or if none apply, you can let me know that too! 😊" | ✅ PASS | Typo tolerated and corrected in response. | 
| **7** | `Diabetes, Others` | "Thank you for sharing your medical conditions, **Mahesh**! You've indicated **Diabetes** and **Others**. Could you please specify what other medical condition you have? 😊" | ✅ PASS | "Others" trigger works correctly. | 
| **8** | `balabbalabal` | "Thank you for sharing that, **Mahesh**! Since you mentioned **Others** for your medical condition, could you please specify what other medical condition you have? 😊" | ⚠️ PARTIAL | Gibberish re-asked, but acknowledgement is generic. | 
| **9** | `balabbalabal` | "Thank you for clarifying, **Mahesh**! Since you mentioned **Others** for your medical condition, could you please specify what other medical condition you have? 😊" | ⚠️ PARTIAL | Misleading "Thank you for clarifying" phrasing. | 
| **10** | `BP` | "Thank you for specifying that, **Mahesh**! You've mentioned **BP** as another medical condition. Now, let's move on to your past surgical history. Have you had any past surgeries? Please answer with **Yes** or **No**. 😊" | ✅ PASS | Clarification successfully completed. | 
| **11** | `Yes` | "Thank you for sharing your past surgical history, **Mahesh**! You've indicated **Yes** for having had past surgeries. Now, let's move on to your history of infectious illnesses. Have you had any of the following? Please select all that apply: **Typhoid**, **Chickenpox**, **Chikungunya**, **Malaria**, **Dengue**, **Jaundice**, or if none apply, you can let me know that too! 😊" | ✅ PASS | Field 4 -> Field 5. | 
| **12** | `Chickenpox` | "Thank you for sharing that, **Mahesh**! You've indicated a history of **Chickenpox**. Now, let's move on to your multivitamins or supplements usage. Are you currently taking any multivitamins or supplements? Please answer with **Yes** or **No**. 😊" | ✅ PASS | Field 5 -> Field 6. | 
| **13** | `no` | "Thank you for sharing that, **Mahesh**! You've indicated that you do not take any multivitamins or supplements. Now, let's move on to your previous hospitalization history. Have you ever been hospitalized? Please answer with **Yes** or **No**. 😊" | ✅ PASS | Field 6 -> Field 7. | 
| **14** | `Skip` | "Thank you for your input, **Mahesh**! You've indicated that you've never been hospitalized. Now, let's move on to your alcohol habits. Do you consume alcohol? Please select from the following options: **Yes**, **No**, **Occasionally**, **Frequently**, or if you'd like to skip, you can do that too! 😊" | ⚠️ PARTIAL | `Skip` interpreted as `No` in chat text. | 
| **15** | `Skip` | "Thank you for sharing that, **Mahesh**! You've indicated that you do not consume alcohol. Now, let's move on to your smoking habits. Do you smoke? Please select from the following options: **Yes**, **No**, **Occasionally**, **Frequently**, or if you'd like to skip, you can do that too! 😊" | ⚠️ PARTIAL | `Skip` interpreted as `No` in chat text. | 
| **16** | `Frequently` | "Thank you for sharing that, **Mahesh**! You've indicated that you smoke **Frequently**. Now, let's talk about your dietary pattern. Are you a **Vegetarian**, **Non-Vegetarian**, **Vegan**, or would you like to skip this question? 😊" | ✅ PASS | Field 9 -> Field 10. | 
| **17** | `Non-Veg` | "Thank you for sharing that, **Mahesh**! You've indicated that you are **Non-Vegetarian**. Now, let's talk about your sleep pattern. How many hours do you typically sleep? Please choose from the following options: **Less than 5 hours**, **5–7 hours**, **7–9 hours**, **More than 9 hours**, or if you'd like to skip this question, you can do that too! 😊" | ✅ PASS | Field 10 -> Field 11. | 
| **18** | `5–7 hours` | "Thank you for sharing that, **Mahesh**! You've indicated that you typically sleep **5–7 hours**. Now, let's talk about your physical activity. How often do you engage in regular exercise? Please select from the following options: **No regular exercise**, **1–2 days per week**, **3–5 days per week**, **Daily**, or if you'd like to skip this question, you can do that too! 😊" | ✅ PASS | Field 11 -> Field 12. | 
| **19** | `1–2 days/week` | "Thank you for sharing that, **Mahesh**! You've indicated that you engage in physical activity **1–2 days per week**. Now, let's move on to your body weight. Could you please provide your body weight in kilograms? 😊" | ✅ PASS | Field 12 -> Field 13. | 
| **20** | `5kgs` | "Thank you for providing your body weight, **Mahesh**! You've indicated your weight is **5 kg**. This seems quite low for an adult. Could you please re-check and provide a more realistic weight in kilograms? 😊" | ✅ PASS | Range validation (< 20kg) works. | 
| **21** | `999kgs` | "Thank you for providing your body weight, **Mahesh**! You've indicated your weight is **999 kg**. This seems quite high. Could you please re-check and provide a more realistic weight in kilograms? 😊" | ✅ PASS | Range validation (> 300kg) works. | 
| **22** | `99kgs` | "Thank you for providing your body weight, **Mahesh**! You've indicated your weight is **99 kg**. Now, let's talk about your stress level. How would you describe your stress level? Please choose from the following options: **Low**, **Moderate**, **High**, or if you'd like to skip this question, you can do that too! 😊" | ✅ PASS | Valid weight accepted. | 
| **23** | `Moderate` | "Thank you for sharing that, **Mahesh**! You've indicated that you have a **Moderate** stress level. Now, let's talk about your marital status. Are you currently **Single**, **Married**, **Divorced**, or **Widowed**? If you'd like to skip this question, you can do that too! 😊" | ✅ PASS | Field 14 -> Field 15. | 
| **24** | `Skip` | "Thank you for sharing that, **Mahesh**! You've indicated that you are currently **Skip** for your marital status. Now, let's move on to the family history of medical conditions. Do you have any family history of the following conditions? Please select all that apply: **Diabetes**, **Hypertension**, **Heart Disease**, **Cancer**, or if none apply, you can let me know that too! 😊" | ⚠️ PARTIAL | Literal "Skip" echo in conversation. | 
| **25** | `Hypertension` | "Thank you for sharing that, **Mahesh**! You've indicated a family history of **Hypertension**. Now, let's move on to the final field: Do you have a patient verification photo? If so, please upload it. 😊" | ✅ PASS | Field 17 -> Field 18. | 
| **26** | *(Upload)* | "Thank you for providing your verification photo, **Mahesh**! You've completed all the information gathering for your appointment. We appreciate your time and look forward to seeing you on **December 8, 2025, at 11:00 AM** with Dr. Jagadeesh. 😊" | ✅ PASS | Flow completed successfully. | 

## 3. Final Booking Summary (API Generated)

### 🧾 Patient Intake & Medical Summary

**🧍 Patient Overview**

* **Primary Concern:** Patient presents with persistent acne on facial skin.
* **Expected Outcome:** Complete acne clearance and skin improvement.
* **Body Weight:** 99 kg
* **Marital Status:** Patient was not interested in disclosing this information.

**🩺 Medical History & Clinical Background**

* **Medical Conditions:** Diabetes, Others: BP
* **Surgical History:** Yes
* **Hospitalizations:** Patient was not interested in disclosing this information.
* **Infectious Illnesses:** Chickenpox
* **Family History:** Hypertension

**🌿 Lifestyle & Daily Habits**

* **Dietary Pattern:** Non-Vegetarian | **Sleep:** 5–7 hours
* **Physical Activity:** 1–2 days per week | **Stress:** Moderate
* **Alcohol:** Not disclosed | **Smoking:** Frequently
* **Supplements:** No

**🧠 Clinical Summary**
The patient presents with persistent **acne** and seeks complete clearance. Medical history includes **Diabetes** and **Hypertension**, which may influence treatment selection and require careful monitoring. Lifestyle assessment reveals **moderate stress levels**, **5-7 hours of sleep**, and **frequent smoking**, factors that can impact skin healing and treatment efficacy.

## 4. Critical Improvement Areas

1. **Error Handling & Relevance (Re-asking):**
   * *Current Behavior:* The system correctly re-asks when input is irrelevant (Step 3).
   * *Recommended Enhancement:* Use a "perfect format" for rejection to guide the user: **"I'm sorry, I quite did not understand your treatment goal or main concern. Could you please describe it again so I can accurately capture it for the doctor?"**

2. **Field Tracking Integrity:**
   * *Issue:* Temporary instability in conversation phrasing during invalid inputs in Step 3/4.
   * *Fix:* Strengthen state persistence to lock the agent to the current field.

3. **"Skip" Acknowledgement:**
   * *Issue:* System interprets skips as "No" or echoes the word "Skip" literally (Step 24).
   * *Fix:* Refine natural language templates to acknowledge skipping without assuming a negative value.
