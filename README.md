# Problem Statement

The problem of automating parts of the job-search to manage job-related tasks through an “agentic” solution.
The goal is to reduce the manual effort involved , rather than reading job descriptions and manually crafting a cover letter.
This AI agent can take inputs (e.g. resume + job description), reason about compatibility, and produce structured results .

This is important because:
Many people waste a lot of time tailoring applications manually; automation can drastically speed up and simplify the process.
An “agentic” solution can combine reasoning, context, and automation: not just generate text, but “decide” what’s relevant, restructure content, and output structured data.

## Why agents?

Compared to a single generative LLM prompt, an agentic system can better manage complexity, incorporate reasoning, and produce structured outputs that are easier to consume, automate further (e.g. store in database / pipeline), and reuse.

## What we've created

The "agentic" workflow involves: taking user inputs (resume + job description), feeding them into an LLM (likely via an API), having the LLM reason about match/compatibility, then generate tailored content (bullet points for resume, cover letter), then output a structured JSON comprising relevant fields ( cover letter).

## Demo

When you run the notebook:
You provide the main inputs: your resume .
The agent compares it and computes a match score: indicating how well your profile fits the job.
It then suggests customized bullet points that you could add or highlight in your resume (to better align with the job).
Finally, it auto-generates a personalized cover letter, tailored to that specific job, using info from your resume .
All output is packaged into a structured JSON, which includes fields like match score, suggestions , cover letter .

This makes the output readable . So , you could plug it into a tracking system, use it to batch-apply for jobs .

# The Build

How we created it, what tools or technologies we used.
Recruit-Radar is constructed as an Autonomous Multi-Agent System using Python. Instead of a linear script, it uses an "agentic" architecture where specialized AI workers collaborate to parse data, hunt for information, verify credibility, and generate output.

## 1. The Core Tech Stack

Framework: CrewAI (for orchestrating agents, tasks, and process flows).

LLM (The Brain): Google Gemini 2.5 Flash (via google-generativeai). Selected for its high speed, low latency, and cost-effectiveness for processing large context windows (resumes).

Search Engine: Serper.dev (Google Search API wrapper). Used to perform advanced Boolean queries to find recruiter data in real-time.

Data Processing:

pdfplumber: For extracting raw text from resume PDFs.

csv: For structured data storage of verified leads.

re (Regex): For pattern matching email addresses.

## 2. The Agentic Workflow

The system is built on a Sequential Process pipeline involving four distinct agents, each with a specific "Persona" and set of tools:

### A. The Analyst (Role Discovery)

Function: Ingests the raw resume text (resume.txt) and acts as an ATS (Application Tracking System).

Logic: It analyzes skills and experience to infer the top 3–5 most relevant job titles, ensuring the search queries are targeted rather than generic.

### B. The Hunter (Lead Extraction)

Function: Executes high-precision Google Dorks (Boolean search queries).

Logic: It utilizes 10+ specific search templates (e.g., site:linkedin.com/posts "hiring" "@gmail.com").

Constraint: Heavily prompted with Negative Constraints to ignore spam, student posts, or "looking for work" comments.

### C. The Verifier (Data Cleaning & Validation)

Function: The quality control layer. It parses the raw snippets found by the Hunter.

The "3-Level Validation" Logic:

Pattern Check: Validates email format and domain (prioritizing corporate emails or known recruiter personal emails).

Keyword Check: Ensures the surrounding text contains authority keywords (e.g., "Talent Acquisition," "HR Manager").

Spam Filter: Rejects snippets containing candidate-side language (e.g., "hire me," "fresher," "interested").

Output: Saves confirmed leads to leads.csv using a custom-built CSVExportTool.

### D. The Copywriter (Outreach)

Function: Generates personalized cold emails.

Logic: Reads the user's resume and the verified lead's context. It maps the candidate's specific strengths to the recruiter's specific post, generating a unique email saved to cold_emails.md.

## 3. Custom Tooling

While standard tools were used for searching, a custom CSVExportTool was built to bridge the gap between the LLM's text output and structured file storage.

Mechanism: The LLM formats a string as Company|Email|Context|Source, and the tool parses this delimiter to append a clean row to the CSV file, ensuring no data loss during the agent handover.

# Future Scope

If I had more time, here are some possible improvements I'd do :

Turn notebook into a proper application ( web UI / web app) - instead of a Jupyter notebook: wrap it into a script or web interface that would make it more user-friendly and accessible.

Support batch processing - allow feeding multiple resumes + job descriptions in bulk, producing outputs ( cover letters) for many jobs in one go.

Add better context / memory / personalization - maintain a “profile database” of users: previous applications, job history, preferences , track applications over time.

Add feedback loop - track which jobs you applied to, whether you got interview calls / rejections; use that feedback to improve matching logic (agent learns over time which kinds of jobs suit you).

Expand to multi-agent architecture , instead of a single monolithic agent: maybe one agent for parsing resume, one for parsing job description, another for matching & scoring, a third for cover letter generation , making system modular, easier to maintain/extend.
