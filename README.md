# Home Lesson Tutorials LMS

A Learning Management System for managing students, curriculum materials, and computer-based testing (CBT) exams. Built for a tutoring center environment where admins create student accounts, upload lessons, and run timed multiple-choice exams.

## How It Works

The system has two roles: **Admin** and **Student**. Admins manage everything (students, lessons, exams). Students log in to access their class materials and take exams.

### Student Lifecycle

1. **Admin creates a student** from the dashboard, entering the student's name, class, parent phone, and a temporary password.
2. The system auto-generates a User ID (e.g. `HLT26001`) — this is what the student uses to log in.
3. The student logs in with their User ID + password, sees their class dashboard with available lessons and exams.
4. The student can **download lecture notes**, **watch video tutorials**, and **take CBT exams**.
5. After submitting an exam, the student sees their score with a question-by-question breakdown.

### Admin Workflow

- **Dashboard** — central hub with sidebar navigation to all sections.
- **Students** — register new students, freeze/unfreeze accounts, soft delete, permanent delete, view individual profiles and exam history.
- **Subjects & Lessons** — upload lecture documents (PDF/DOCX/PPTX), link video tutorials, organize by subject and class.
- **Exams Module** — create timed exams, set attempt limits, build question banks (multiple choice A/B/C/D).
- **Results Portal** — view all submissions across all students with aggregate stats (total submissions, average score, pass rate).
- **Diagnostics** — audit all questions for missing/invalid correct answers and fix them inline.

### Exam Flow

1. Student clicks "Enter Examination Environment" on an available exam.
2. A **countdown timer** starts (set by the exam creator, e.g. 30 minutes). The timer persists across page refreshes via localStorage.
3. Questions appear one per card. Student selects A/B/C/D. The **question navigator** sidebar tracks which questions are answered (green) vs unanswered (red).
4. Anti-cheating measures: right-click disabled, developer tools shortcuts blocked, text selection disabled, page-leave warning.
5. Answers auto-save to localStorage every 30 seconds as a backup against accidental refreshes.
6. On submit, the system grades server-side (correct answers are never sent to the client), stores the result, and shows a full breakdown matrix.
7. If the timer expires, the exam auto-submits with the current selections.

### Account Statuses

| Status | Effect |
|--------|--------|
| `active` | Normal access |
| `frozen` | Blocked from logging in; admin can unfreeze |
| `deleted` | Hidden from normal queries (soft delete); can be permanently removed |

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js + Express 5 |
| Database | PostgreSQL (Neon) |
| Templating | EJS |
| Auth | bcryptjs + express-session |
| CSRF | @dr.pogodin/csurf + cookie-parser |
| File uploads | Multer (15MB limit) |
| Testing | Vitest |
| Frontend | Bootstrap 5.3 |

## Setup

### Prerequisites

- Node.js 18+
- A PostgreSQL database (e.g. [Neon](https://neon.tech))

### Installation

1. Clone the repo and install dependencies:

```bash
git clone <repo-url>
cd LMSforThaoban
npm install
```

2. Copy `.env.example` to `.env` and fill in your credentials:

```bash
cp .env.example .env
```

Your `.env` should look like:

```
PORT=3000
SESSION_SECRET=a-strong-random-string
DATABASE_URL=postgresql://user:password@host:5432/dbname?sslmode=require
```

3. Set up the database — run the schema against your PostgreSQL database:

```bash
psql $DATABASE_URL -f schema.sql
```

4. Create the first admin account:

```sql
INSERT INTO users (username, full_name, password_hash, role, account_status)
VALUES ('admin', 'Administrator', '<bcrypt-hash>', 'admin', 'active');
```

To generate a bcrypt hash quickly:

```bash
node -e "require('bcryptjs').hash('your-password', 10).then(h => console.log(h))"
```

5. Start the server:

```bash
npm start
```

The app runs at `http://localhost:3000`.

## Project Structure

```
app.js                        # Express server entry point (thin shell)
schema.sql                    # Full database schema (8 tables)
src/
  db/pool.js                  # PostgreSQL connection pool
  middleware/
    auth.js                   # Route guards (protectRoute, redirectIfLoggedIn)
    upload.js                 # Multer file upload config
    validate.js               # Reusable input validation middleware
  routes/
    auth.js                   # Landing page, login, logout
    admin.js                  # All admin routes (students, curriculum, exams, results, diagnostics)
    student.js                # Student dashboard, exam room, results
views/
  admin/                      # Admin pages (dashboard, students, curriculum, exams, questions, results, exam_health, student_details)
  auth/                       # Login page
  public/                     # Landing page
  student/                    # Student pages (dashboard, exam_room, exam_result, results)
public/                       # Static assets (CSS, images)
tests/                        # Vitest test files
```

## Available Scripts

| Command | Description |
|---------|-------------|
| `npm start` | Start production server |
| `npm run dev` | Start with nodemon (auto-reload) |
| `npm test` | Run Vitest test suite |

## Database Schema

8 tables: `users`, `student_profiles`, `classes`, `subjects`, `lessons`, `exams`, `questions`, `exam_attempts`. See `schema.sql` for the full DDL with constraints and indexes.

## Route Map

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| GET | `/` | Public | Landing page |
| GET/POST | `/login` | Public / redirectIfLoggedIn | Login form and handler |
| GET | `/logout` | Public | Destroy session |
| GET | `/admin/dashboard` | Admin | Admin home |
| GET/POST | `/admin/students` | Admin | List / create students |
| POST | `/admin/students/:id/toggle-freeze` | Admin | Freeze or unfreeze a student |
| POST | `/admin/students/:id/delete` | Admin | Soft delete a student |
| POST | `/admin/students/:id/permanent-delete` | Admin | Permanently remove a student |
| GET | `/admin/students/:id/details` | Admin | View student profile + history |
| GET/POST | `/admin/curriculum` | Admin | List / create lessons |
| POST | `/admin/curriculum/lesson/edit/:id` | Admin | Edit a lesson |
| POST | `/admin/curriculum/lesson/delete/:id` | Admin | Delete a lesson |
| GET/POST | `/admin/exams` | Admin | List / create exams |
| POST | `/admin/exams/:id/alter-attempts` | Admin | Update attempt limit |
| GET/POST | `/admin/exams/:id/questions` | Admin | View / add questions |
| POST | `/admin/exams/:examId/questions/edit/:questionId` | Admin | Edit a question |
| POST | `/admin/exams/:examId/questions/delete/:questionId` | Admin | Delete a question |
| GET | `/admin/result` | Admin | Results portal |
| GET/POST | `/admin/diagnostic/exam-health` | Admin | Audit and fix broken questions |
| GET | `/student/dashboard` | Student | Student home (lessons + exams) |
| GET | `/student/results` | Student | Personal exam history |
| GET | `/student/exams/:id` | Student | Enter exam room |
| POST | `/student/exams/:id/submit` | Student | Submit exam answers |
