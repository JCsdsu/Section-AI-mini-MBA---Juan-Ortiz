# Project Memo — AI-Powered Staffing Match & Summary Tool

## Project Overview
This project proposes an internal web application for the staffing company I work for (Soundings Connect) that integrates directly with our Zoho CRM via API to automate and enhance the candidate–job matching process. 

Currently, we use Zoho CRM to store candidates and jobs within separate modules. The job module has standard fields that are filled out by the client to describe their needs. The candidate module has standard fields that describe their skills and profile. We then run manual reports with to search for candidates within our network that meet the needs of the client. This is the process we'd like to partially automate. 

The new system will:

- Match candidates to open jobs using both deterministic skills matching and AI-powered cultural fit analysis.
- Generate standardized job descriptions from CRM job data (used to pitch to the candidates).
- Produce concise candidate summaries after interviews, combining skills, strengths, and qualitative notes (used to pitch to clients).

**Problem:**  
Our recruiters currently spend significant time manually:
- Reviewing candidate profiles for relevant skills.
- Assessing “culture fit” based on CliftonStrengths compatibility with client contacts and reviewing interview transcipts.
- Writing consistent, candidate-ready job descriptions and client-ready candidate summaries.

**Opportunity:**  
By automating these steps with AI, we can:
- Save hours of manual work.
- Increase match quality and consistency.
- Provide richer, data-informed recommendations to clients.
- Standardizes output.
- Reduces bias from recruiters. 

---

## Use Case & Workflow
**Primary users:** Internal recruiters and staffing coordinators.

**Workflow:**
1. **Job intake** — Zoho CRM contains open job records with top 10 required skills, client contact details (including their CliftonStrenghts), and other metadata.
2. **Candidate matching** —  
   - **2a:** Deterministic scoring based on overlap between candidate skills (top 10) and job skills (top 10).  
   - **2b:** AI-driven cultural fit analysis based on compatibility between candidate CliftonStrengths (top 5) and client contact CliftonStrengths, and interview transcripts.
3. **Job description generation** — LLM creates a standardized, one-page description from CRM job record data. This is used to pitch the job to the candidates. 
4. **Shortlist creation** — Top 5 candidates ranked by combined skill and fit score.
5. **Interview stage** — Recruiter interviews candidates, uploads full transcript to the system. This transcript is obtained by using Fireflies AI (already in use internally). 
6. **Candidate summary generation** — LLM produces a structured, client-ready one-page summary.

---

## AI Features to Be Implemented

### 1. Prompt Engineering
- **Few-shot prompting** for CliftonStrengths compatibility scoring, using expert-provided examples of “good fit” and “poor fit” pairings.
- **Chain-of-thought prompting** to have the LLM explain compatibility reasoning before outputting a final score.

### 2. Structured Outputs
- JSON schema for:
  - Job descriptions (sections: Overview, Responsibilities, Skills Required, Cultural Fit Notes).
  - Candidate summaries (sections: Skills Match Score, Cultural Fit Score, Interview Highlights, Fit Rationale).

### 3. Retrieval-Augmented Generation & Vector Database
- Store expert-authored CliftonStrengths compatibility guidelines in a vector database (OpenAI embedding functionality).
- When assessing cultural fit, the system retrieves relevant compatibility patterns and includes them in the LLM context.

### 4. Evaluation Frameworks
- Build a ground truth dataset of past successful placements with:
  - Job data.
  - Candidate data.
  - Actual hire outcomes.
- Evaluate LLM cultural fit scoring against historical matches.
- Use **LLM-as-a-judge** for ongoing prompt refinement.

### 5. Observability Tools
- Log all LLM calls (input, output, token usage, latency, and cost).
- Track changes in cultural fit scores over time.
- A/B test prompt variations for compatibility scoring.

---

## Technical Approach

### Data Flow
1. **Zoho CRM API** → Python backend retrieves:
   - Job records.
   - Candidate profiles.
   - Client contact details (including CliftonStrengths).
2. **Skills Matching**:
   - Python function computes skill overlap score (0–10).
3. **Cultural Fit Matching**:
   - Query vector DB for relevant compatibility guidelines.
   - Pass guidelines + candidate & client strengths into LLM prompt.
4. **Job Description Generation**:
   - Pass job data into LLM with structured output instructions.
5. **Interview Summary**:
   - Upload transcript, pass into LLM with structured output instructions.
6. **Frontend**:
   - Internal web dashboard (Streamlit) for recruiters to view ranked matches, generated job descriptions, and summaries.

### Tools & Services
- **Language Model:** OpenAI GPT-5.
- **Vector Store:** OpenAI API.
- **Backend:** Python (FastAPI).
- **Frontend:** Streamlit.
- **CRM Integration:** Zoho CRM API.
- **Logging & Observability:** LangSmith, Weights & Biases, or custom logging (based on my research).

---

## Example Prompt & Expected Outputs 

### Initial Prompt

**Recruiter enters desired Job ID:**  

**The web app will run:**
1. **Data retrieval (Zoho CRM):** The system pulls the job record (title, responsibilities, required skills), the client contact (including CliftonStrengths), and the eligible candidate pool (top 10 skills, top 5 strengths). It is important to note, these objects in CRM cannot be saved unless they have all the relevant info, so we do not need guardrails for missing data. If the job ID does not exist, a predefined error message may be displayed
2. **Skills scoring (deterministic):** A simple function counts skill overlap between the job’s top 10 skills and each candidate’s top 10, producing a score from 0 to 10. Cursor can be used to write this simple function initially. 
3. **Cultural fit scoring (LLM + guidelines):**  
   - The system retrieves relevant **CliftonStrengths compatibility guidelines** from a small internal knowledge base (RAG).  
   - The LLM uses those guidelines plus the client/candidate strengths to produce a **cultural fit score** (0–10) and a brief rationale in plain language.
4. **Aggregation & ranking:** Each candidate receives a combined score (e.g., weighted average of skills and cultural fit). The top five are selected. 
5. **Standardized job description:** The LLM generates a one-page job description from the CRM record using a consistent structure (title, overview, responsibilities, required skills, cultural fit notes). This is manually passed on to the candidates to gauge interest. 

---

### Follow-Up Prompt (After Interviews)

**Recruiter says:**  
“I’ve uploaded interview transcripts for the shortlisted candidates. Generate a one-page summary per candidate that includes key skills evidence, behavioral signals, and a brief ‘Reasons to Hire’ section.” I have thought this prompt may be better stored in the program itself, in chunks, so we can monitor output consistency and tweak as needed. 

**The web app will run:**
1. The system associates each transcript with the corresponding candidate and job.
2. The LLM extracts evidence of skills, examples/quotes from the interview, and behavioral indicators aligned to the role.
3. The LLM produces a consistent one-page summary for each candidate: skills match highlights, cultural fit recap, interview takeaways, and reasons they’re a strong fit for the client.

**Summary of output:**
- For each candidate, a polished, one-page summary that can be shared internally or adapted for the client, with clear sections and concise, evidence-based language.

---

### Formatting Strategy

- Standardized sections with chunking and structured output to reduce editing:  
  - Job Description: *Title, Overview, Responsibilities, Required Skills, Cultural Fit Notes.*  
  - Candidate Cards: *Skills Score, Cultural Fit Score, Overlap Skills, Strengths, Fit Rationale.*  
  - Interview Summary: *Skills Evidence, Behavioral Signals, Fit Rationale, Reasons to Hire.*

---

## Evaluation Strategy

### What we evaluate
**Matching quality**
  - Skills score validity: Does the deterministic score correlate with actual shortlist/hire outcomes?
  - Cultural fit accuracy: Do LLM fit scores align with expert expectations and past successful placements?
**Content quality**
  - Job descriptions: Clarity, completeness, and consistency.
  - Candidate summaries: Evidence-based, concise, and client-ready.
**User impact**
  - Time saved per role, recruiter satisfaction, shortlist acceptance rate by hiring managers.

### How we evaluate
**Ground-truth dataset (Expert-driven):**  
- Curate a set of historic roles with their finalists/hire, including job data, candidate skills/strengths, interview notes, and the final    decision. This is the “expert” pillar in action.
**Techniques**
  - String/structure checks for deterministic parts (e.g., skills overlap is computed correctly).
  - LLM-as-a-judge for cultural fit rationales against expert-written rubrics/guidelines.
  - Human review by recruiters for usefulness and accuracy on a sample of outputs.
  - Combination of the above to balance objectivity and real-world nuance.
**Metrics**
  - Rank correlation (e.g., Spearman) between our ranked list and actual hire/shortlist.
  - Alignment score from LLM judge vs expert rubric.
  - Recruiter satisfaction (1–5) on job descriptions and candidate summaries. We do already have 1-5 surveys from clients implemented.
  - Operational: JSON/structure compliance rate, re-run reproducibility.
**Targets & cadence**
  - Start with an 80% pass rate on internal thresholds, then raise as we iterate.
  - A/B prompt versioning: Promote only if the candidate version beats the control on pre-agreed metrics.

---

## Observability Plan

### What we track
- For each run: job ID, candidate IDs considered, steps executed (retrieval → skills score → cultural fit → ranking → generation).
- Per step: start/end time, latency, errors, retries, prompt version, input/output sizes, token usage, and estimated cost.
- Retrieval health (RAG): similarity scores, missing guideline topics.
- Output quality signals: structured-output compliance, length, section completeness.
- Human-in-the-loop feedback: thumbs up/down, edits, comments; used to build better datasets.

### Continuous improvement loop
- Trace & log every run.
- Evaluate on curated datasets (evals pillar with objective targets).
- Expert review of edge cases and prompt examples.
- Iterate, update prompts/guidelines, A/B test, and promote winners based on metrics.


---

## Final Note
- One major flaw may be that skills and strenghts are self reported by candidates, and self requested by clients. Right now, our goal is to successfully match these. However, we may want to address the validity of the self reported/requested metrics. If we can provide education/testing to candidates to validate skills, and use data to help clients identify what skills they actually need, this will be much more powerful. 
