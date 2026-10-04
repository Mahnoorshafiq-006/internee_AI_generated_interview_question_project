# internee_AI_generated_interview_question_project
# AI-Powered Interview Question Generator

An NLP project that generates **custom technical and behavioral interview questions for interns**. It reads an intern's resume (or a role name), finds the relevant skills, and produces a role-specific question set using existing question banks and a LLaMA-based text generation model.

**Author:** Mahnoor, BS Computer Science, UET Peshawar (Bannu Campus)

---

## 1. Objective

Generate custom technical and behavioral questions for intern interviews, automatically and per role.

| Task requirement | How this project meets it |
|---|---|
| Existing question banks | Hugging Face Software Engineering interview dataset |
| Intern profiles | Resume dataset (`Resume.csv`) |
| Job descriptions | Kaggle software engineering job postings |
| Model type: text generation (GPT-3, LLaMA) | TinyLlama-1.1B-Chat (LLaMA 2 architecture) |
| Outcome: automated, role-specific question sets | Output printed and saved to `generated_questions.csv` |

---

## 2. How it works

```
Resume.csv ──► clean text ──► extract skills ─┐
                                              ├─► match questions from the bank ─► few-shot prompt ─► LLaMA model ─► question set
Job postings ─► role title ──► extract skills ─┘
Question bank ─► clean ─► tag type (technical / behavioral) and skills ─┘
```

1. **Profile**: find the skills in a resume, or learn a role's skills from job postings.
2. **Retrieve**: pick bank questions that mention those skills (technical) plus behavioral questions.
3. **Generate**: give the retrieved questions to the LLaMA model as examples and ask it to write new questions.
4. **Combine**: model-written questions (marked `[AI]`) come first; the rest are filled from the bank.
5. **Save**: print the question set and write `generated_questions.csv`.

---

## 3. Datasets

| Dataset | Source | Role in the project | Details |
|---|---|---|---|
| **Resume dataset** | Kaggle (`Resume.csv`) | Intern profiles | 2,484 resumes, 4 columns (`ID`, `Resume_str`, `Resume_html`, `Category`), 24 job categories, no missing values or duplicates. Only `INFORMATION-TECHNOLOGY` (120) and `ENGINEERING` (118) are used, which gives 238 resumes. `Resume_html` is dropped; `Resume_str` is the plain text used. |
| **Software Engineering interview dataset** | Hugging Face: `Vineeshsuiii/Software_Engineering_interview_datasets` | Question bank | Text data, 1K to 10K rows, parquet format, split into `train` and `test` files. |
| **Software engineering job postings** | Kaggle (LinkedIn job postings) | Job descriptions and role skills | Columns include job title, job level, job summary and job skills. The code also accepts other posting files with `title`/`job_title` and `skills_desc`/`job_summary`/`description` columns. |

### Data preparation

- Resumes: keep IT and Engineering categories, drop the HTML column, extract skills.
- Question bank: auto-detect the question column, drop missing values and duplicates, tag each question as technical or behavioral, and tag the skills it mentions.
- Job postings: filter by role title, count the skills in the first 300 matching postings, and use the top 5.

---

## 4. Skills and technologies

| Area | Used |
|---|---|
| Language | Python 3 |
| Data handling | pandas, pyarrow (reading parquet), `collections.Counter` |
| Text processing | `re` (regular expressions), `random` |
| NLP and generation | Hugging Face Transformers, PyTorch, TinyLlama-1.1B-Chat (LLaMA 2 architecture) |
| Formats | CSV, Parquet |

**Skills demonstrated:** data cleaning, working with multiple datasets, text mining, prompt engineering, and applying a pretrained language model for text generation.

---

## 5. Techniques used

| Technique | What it does here |
|---|---|
| Data cleaning | Removes duplicates, missing values and HTML; filters to relevant categories |
| Keyword-based skill extraction | Matches a list of about 30 skills (Python, SQL, Git, machine learning and so on) in resumes, questions and job postings, using regex with word boundaries so `c++` and `java` match correctly |
| Rule-based question classification | Marks a question as behavioral if it contains words like "tell me about", "describe a time", "conflict" or "deadline"; otherwise technical |
| Frequency analysis | Counts the most common skills in postings for a role to define that role's skills |
| Retrieval | Selects bank questions that share skills with the candidate or role |
| Few-shot prompting | Gives the model 3 example questions and the skills, then asks for one new question |
| Text generation (LLM) | TinyLlama writes new questions with sampling (`temperature=0.8`, `top_p=0.95`) |
| Output filtering | Keeps only generated text that ends with a question mark, is long enough, and is not a copy of an example |
| Hybrid generation | Model-written questions are combined with bank questions, so the output is always complete |

---

## 6. Model

**TinyLlama-1.1B-Chat-v1.0** (`TinyLlama/TinyLlama-1.1B-Chat-v1.0`): a 1.1 billion parameter chat model that uses the same architecture and tokenizer as Llama 2, with an Apache 2.0 license. It is free, needs no access approval, and runs on a laptop CPU (slowly) or on a GPU.

Notes on the task's model options:

- **LLaMA**: used through TinyLlama. Larger LLaMA models (for example `meta-llama/Llama-3.2-1B-Instruct`) can be used by changing `MODEL_NAME`, but they need access approval.
- **GPT-3**: not used. It requires a paid OpenAI API key.

---

## 7. Project files

```
.
├── simple_interview_generator.py   # main script (the whole pipeline)
├── Resume.csv                      # intern profiles
├── train-00000-of-00001.parquet    # question bank (Hugging Face)
├── postings.csv                    # job postings (Kaggle)
├── generated_questions.csv         # output
└── README.md
```

A longer version with a command-line interface, `interview_generator.py`, is also available.

---

## 8. How to run

### Install

```bash
pip install pandas pyarrow transformers torch sentencepiece
```

### Set your file names

At the top of `simple_interview_generator.py`:

```python
RESUME_FILE = "Resume.csv"
QUESTION_FILE = "train-00000-of-00001.parquet"
JOBS_FILE = "postings.csv"

USE_MODEL = True    # False = pick questions from the bank only (fast, no download)
MODEL_NAME = "TinyLlama/TinyLlama-1.1B-Chat-v1.0"
```

### Run

```bash
python simple_interview_generator.py
```

The first run downloads the model (about 2.2 GB). The script prints the question and job file columns at the start; if it picks the wrong question column, set `QUESTION_COLUMN` in the code.

### Output format

```
============================================================
Candidate / role: Role: python developer
Skills: python, sql, django, git, flask

Technical questions:
  1. [AI] <question written by the model>
  2. <question taken from the bank>
  ...

Behavioral questions:
  1. <question>
  ...
```

`[AI]` marks questions written by the model. Results are also saved to `generated_questions.csv` with the columns `who`, `skills`, `technical`, `behavioral`.

---

## 9. Limitations

- Skill extraction is keyword-based, so it misses skills that are not in the skill list or are written in different words.
- Resumes with no listed skills get random technical questions from the bank.
- TinyLlama is a small model, so some generated questions can be weak or repetitive. Filtering and the bank questions reduce this.
- The question bank is tagged by keywords, so a few questions may be classified incorrectly.
- Running the model on CPU is slow.

## 10. Future improvements

- Fine-tune the LLaMA model on the question bank (for example with LoRA).
- Replace keyword matching with embeddings (such as sentence-transformers) for better skill and question matching.
- Add difficulty levels (easy, medium, hard) and per-skill question counts.
- Add a simple web interface (Streamlit) where a user uploads a resume and gets a question set.
- Add evaluation: question diversity, relevance scoring, and human review.

## 11. Data sources and credits

- Resume dataset: Kaggle
- Question bank: Hugging Face dataset `Vineeshsuiii/Software_Engineering_interview_datasets`
- Job postings: Kaggle LinkedIn job postings (software engineering)
- Model: TinyLlama project, `TinyLlama/TinyLlama-1.1B-Chat-v1.0` (Apache 2.0)

These datasets are for educational use. Check each dataset's license and terms before reusing them elsewhere.
