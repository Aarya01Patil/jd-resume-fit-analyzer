# 🚀 Agentic Workflow: JD-Resume Fit Analyzer

**Name:** Aarya Patil

**Roll Number:** 23BCS10170

**Submission Type:** Individual Submission

**Tool Used:** n8n (Low-code workflow automation)

## 📌 Problem Statement

**The User:** HR Professionals, Technical Recruiters, and Hiring Managers.
**The Pain Point:** Recruiters spend an overwhelming amount of time manually screening hundreds of unstructured resumes against highly specific Job Descriptions (JDs). This manual process is slow, prone to human error, and often introduces implicit bias, leading to missed talent or wasted interview slots.
**The Solution:** An automated, agentic HR assistant that acts as a first-round screening pipeline. It processes candidate submissions, systematically extracts data, evaluates the fit using LLMs, and routes the application autonomously based on predefined logic.
**Expected Output:** A structured HR database logging all applicants, prioritized human-in-the-loop alerts for top candidates, and personalized, AI-generated feedback automatically emailed to applicants based on their performance.

---

## 🏗️ Workflow Architecture & Logic

This workflow abandons the "single massive prompt" approach in favor of a multi-step agentic pipeline. It separates extraction, reasoning, validation, and action into distinct nodes.

### Step-by-Step Breakdown:

1. **Intake & Trigger (`1 · Form Trigger`):**
* Captures candidate details (Name, Email), the target Job Description (text), and the candidate's Resume (PDF upload).


2. **Deterministic Data Extraction (`2 · PDF Parser`):**
* A native utility node that strips raw text from the uploaded PDF without relying on AI, ensuring perfectly accurate text transfer.


3. **Agent 1: The Extractor (`3 · AI Extractor` & `3b · Parse Extractor`):**
* **Role:** Data Structurer.
* **Task:** Ingests the raw resume text and outputs a strictly defined JSON schema (Skills, Experience, Education, Certifications).
* **Sanitization:** Node `3b` deterministically catches the AI's output, strips any markdown formatting, and validates the JSON before passing it downstream.


4. **Agent 2: The Analyst (`4 · AI Fit Scorer`):**
* **Role:** Evaluator.
* **Task:** Compares the structured resume JSON against the raw Job Description. It identifies matched skills, missing skills, and overall strengths, ultimately generating a numeric score (0-100) and a recommended fit category.


5. **The Guardrail (`5 · Score Validator (Deterministic)`):**
* A custom JavaScript node that enforces strict rules on the AI's output. It verifies the score is a valid integer between 0 and 100, and standardizes the categorization (`strong_fit`, `partial_fit`, `low_fit`) to prevent AI hallucinations from breaking the workflow routing.




6. **The Router (`6 · Router (by Score Category)`):**
* A switch node that evaluates the deterministic fit category and routes the workflow down one of three specialized branches.




7. **Execution Branches:**
* 🟢 **Branch A: Strong Fit (70-100):** Sends an internal alert to the Recruiter for a human-in-the-loop review, and sends a positive acknowledgment email to the candidate.
* 🟡 **Branch B: Partial Fit (40-69):** Triggers **Agent 3: The Career Coach (`6b · AI Recommender`)**. This AI generates a highly personalized, actionable improvement plan (course recommendations, portfolio ideas) which is instantly emailed to the candidate.
* 🔴 **Branch C: Low Fit (0-39):** Triggers a deterministic, polite auto-rejection email.


8. **System Logging (`Log to Google Sheets`):**
* Regardless of the branch, every execution appends a structured row to the HR Dashboard in Google Sheets, logging the timestamp, score, missing skills, and AI recommendation.





---

## 🤖 Agentic Practices Demonstrated

This workflow explicitly hits the core concepts of agentic design:

* **Task Decomposition & Role Definition:** AI is not used as a monolithic chatbot. It is split into specialized agents (Extractor, Analyst, Coach), each with a highly constrained prompt and specific objective.


* **Structured Outputs:** The Extractor and Analyst agents are forced to return strict JSON arrays and objects, allowing seamless integration with standard code logic.


* **AI vs. Deterministic Logic:** AI is restricted to tasks requiring reasoning (parsing unstructured text, evaluating skill gaps, drafting coaching advice). Deterministic logic (JavaScript code nodes, Switch nodes) is used for control flow, mathematical validation, and API routing.


* **Human-in-the-Loop (HITL):** The system does not automatically schedule interviews. High-scoring candidates generate a structured alert to a human recruiter, leaving the final high-stakes decision to a person.


* **Tool & API Integration:** Seamlessly integrates native Form triggers, local file parsing, Google Sheets (OAuth2), and Gmail (OAuth2).



---

## 💻 Setup & Installation Instructions

1. Clone or download this repository.
2. Ensure you have an active instance of [n8n](https://n8n.io/).
3. Import the `JD-Resume Fit Analyzer.json` file into your n8n workspace.
4. **Configure Credentials:**
* Add your OpenAI API key for the LLM nodes.
* Authenticate your Google Account for the Google Sheets and Gmail nodes.


5. **Configure the Dashboard:**
* Create a new Google Sheet with the following headers: *Timestamp, Candidate Name, Candidate Email, Overall Score, Fit Category, Matched Skills, Missing Skills, Recommendation*.
* Update the `Log to Google Sheets` node with your specific Document ID.


6. Click **Execute Workflow**, open the provided Form URL, and submit a test application!

---

## 🔮 Limitations & Future Enhancements

* **Token Limits:** Excessively long, multi-page CVs could theoretically exceed the context window of smaller LLM models during the extraction phase.
* **Fallback Mechanisms:** While `Parse Extractor JSON` catches markdown errors, a true production system would feature an automated retry-loop if the AI fails to return valid JSON.
* **Integration Expansion:** Future versions could replace Google Sheets with a direct API integration into an Applicant Tracking System (ATS) like Workable or Greenhouse.
