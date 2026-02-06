# Quiz Web Application — Exam Prep Platform

A production-ready product design for a university-focused quiz platform that turns PDF study materials into engaging, timed, step-by-step quizzes with leaderboards.

---

## 1) Product Overview
**Goal:** A fast, student-friendly exam simulator and AI tutor that generates quizzes from uploaded PDFs and motivates learners with progress, feedback, and rankings.

**Key UX Principles**
- **Minimal, focused UI** with only the current question visible.
- **Immediate feedback** after each answer (correct/incorrect + explanation).
- **Motivational micro-texts** (“Keep going!”, “You’re doing great!”).
- **Responsive design** for mobile and desktop.

---

## 2) System Architecture

### Frontend (Web + Mobile Web)
- **Framework:** React (Next.js) or Vue (Nuxt)
- **Core modules**:
  - PDF upload + language selector
  - Quiz experience (step-by-step question flow)
  - Results + performance summary
  - Leaderboard + user ranking
  - Profile/history
- **State Management:** Redux / Zustand / Pinia
- **Internationalization:** i18n with Uzbek, English, Russian

### Backend (API + AI Services)
- **API Layer:** Node.js (NestJS / Express) or Python (FastAPI)
- **Services:**
  1. **PDF Processing Service**
     - Extracts text + metadata from PDF
  2. **Concept Extraction Service**
     - Summarizes and extracts key concepts
  3. **Quiz Generation Service**
     - Generates MCQ + True/False questions with explanations
  4. **Quiz Engine**
     - Controls timing, scoring, persistence
  5. **Leaderboard Service**
     - Computes ranking based on score, then time

### Storage
- **Database:** PostgreSQL
- **File Storage:** S3 / MinIO (PDFs + derived files)
- **Cache:** Redis (fast quiz sessions & leaderboard)

---

## 3) Database Schema (PostgreSQL)

### users
| Field | Type | Notes |
|------|------|------|
| id | uuid (PK) | Unique user ID |
| name | varchar | Full name |
| email | varchar (unique) | Login |
| password_hash | varchar | Auth |
| created_at | timestamp | |

### pdf_files
| Field | Type | Notes |
|------|------|------|
| id | uuid (PK) | |
| user_id | uuid (FK) | Uploader |
| filename | varchar | |
| storage_url | varchar | S3/MinIO path |
| detected_language | varchar | en/ru/uz |
| uploaded_at | timestamp | |

### quizzes
| Field | Type | Notes |
|------|------|------|
| id | uuid (PK) | |
| user_id | uuid (FK) | Owner |
| pdf_id | uuid (FK) | Source file |
| title | varchar | Quiz name |
| total_questions | int | |
| time_limit_sec | int | nullable |
| created_at | timestamp | |

### questions
| Field | Type | Notes |
|------|------|------|
| id | uuid (PK) | |
| quiz_id | uuid (FK) | |
| type | enum (mcq, tf) | |
| prompt | text | |
| options | jsonb | Array for MCQ |
| correct_answer | varchar | Index or boolean |
| explanation | text | |
| order_index | int | Question order |

### quiz_results
| Field | Type | Notes |
|------|------|------|
| id | uuid (PK) | |
| user_id | uuid (FK) | |
| quiz_id | uuid (FK) | |
| score | int | Correct answers |
| total | int | Total questions |
| accuracy | numeric | % |
| time_spent_sec | int | |
| performance_level | varchar | Excellent/Good/etc |
| created_at | timestamp | |

### leaderboard_entries
| Field | Type | Notes |
|------|------|------|
| id | uuid (PK) | |
| user_id | uuid (FK) | |
| quiz_id | uuid (FK) | |
| score | int | |
| time_spent_sec | int | Tiebreaker |
| rank | int | Calculated |
| created_at | timestamp | |

---

## 4) API Endpoints

### Auth
- `POST /api/auth/register`
- `POST /api/auth/login`

### PDF Upload + Quiz Generation
- `POST /api/pdf/upload`
  - Body: `multipart/form-data` (PDF + optional language)
- `POST /api/quiz/generate`
  - Body: `{ pdf_id, language, question_count, time_limit_sec }`

### Quiz Flow
- `GET /api/quiz/:quiz_id`
  - Returns quiz metadata + first question
- `POST /api/quiz/:quiz_id/answer`
  - Body: `{ question_id, answer }`
  - Returns correctness + explanation + next question

### Results + Leaderboard
- `GET /api/quiz/:quiz_id/result`
- `GET /api/leaderboard/:quiz_id`
- `GET /api/leaderboard/:quiz_id/me`

---

## 5) AI Logic & Processing Pipeline

### Step 1: PDF Parsing
- **Tools:** pdfplumber / PyMuPDF / AWS Textract
- Extract:
  - full text
  - headings (sections/chapters)
  - key phrases

### Step 2: Concept Extraction
- Use embedding + summarization:
  - chunk text (500–1000 tokens)
  - summarize per chunk
  - extract high-signal concepts + definitions

### Step 3: Question Generation
- **Input:** concepts + summaries
- **Output:** MCQ + True/False with explanations

**MCQ rules**
- 4 options
- 1 correct
- 3 plausible distractors

**True/False rules**
- Based on a single statement
- Explanation must justify truthfulness

### Step 4: Answer Validation
- Validate against stored `correct_answer`
- Return correctness + explanation

---

## 6) Sample JSON Response (Quiz Question)

```json
{
  "quiz_id": "a1b2c3",
  "question": {
    "id": "q12",
    "type": "mcq",
    "prompt": "Which of the following best describes Newton’s Second Law?",
    "options": [
      "Force equals mass times acceleration",
      "Energy equals mass times the speed of light",
      "For every action there is an equal and opposite reaction",
      "Objects in motion stay in motion unless acted upon"
    ],
    "correct_answer": 0,
    "explanation": "Newton’s Second Law states F = m × a, relating force to mass and acceleration."
  },
  "progress": {
    "current": 3,
    "total": 20
  },
  "feedback": {
    "result": "correct",
    "message": "✅ Correct! Keep going!"
  }
}
```

---

## 7) Quiz Flow (Step-by-Step UX)
1. User uploads PDF + selects language (auto-detect by default)
2. System generates quiz
3. Questions appear one-by-one
4. User answers → immediate feedback
5. Next question auto loads
6. Final results + ranking shown

---

## 8) Performance Evaluation

**Performance Level**
- **Excellent:** 90–100%
- **Good:** 75–89%
- **Average:** 50–74%
- **Needs Improvement:** < 50%

---

## 9) Motivational Micro-Texts
- “Keep going!”
- “You’re doing great!”
- “Almost there!”
- “Nice work!”

---

## 10) Competitive Leaderboard

**Ranking Logic**
- Sort by score (desc)
- If tie, sort by time_spent_sec (asc)

**Leaderboard Output Example**
- “You are 5th out of 120 students.”
- Top 10 shown with score + time

---

## 11) Final Product Feel
- **Exam simulator** with real-time scoring
- **Competitive platform** with rankings
- **AI tutor** giving feedback + explanations
