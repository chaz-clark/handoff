# Canvas-Toolbox Feature Requirements for NGAI Platform

**Handoff Document**
**From**: DS250 NGAI n8n Platform Team
**To**: Canvas-Toolbox Development Team
**Date**: 2026-06-29
**Purpose**: Request new features and enhancements to support NGAI platform integration

---

## Executive Summary

The NGAI (Non-General AI) platform is building a three-layer AI agent system (Peer/QC/Orchestrator) on top of n8n workflows that integrates deeply with Canvas LMS. The platform requires several new canvas-toolbox capabilities to enable:

1. **Peer agents** to interact with students using student-view Canvas materials
2. **QC agents** to auto-grade and evaluate critical thinking/questioning
3. **Orchestrator** to manage instructor approval workflows
4. **Academic integrity** handling via remedial assignments
5. **Course design improvement** through gap analysis

This document specifies 7 new features needed from canvas-toolbox, with priorities, complexity estimates, and acceptance criteria.

---

## Platform Context (Brief Overview)

**Architecture**: Three-layer NGAI model
- **Bottom Layer (Peer Agents)**: Student-facing chat agents, one per student, access student-view Canvas only
- **Middle Layer (QC Agents)**: Auto-grading agents, one per module, have full access including answer keys
- **Top Layer (Orchestrator)**: Teaching-TA agent, manages instructor approval workflow

**Integration Pattern**: n8n workflows call canvas-toolbox Python scripts via Execute Command node or REST API

**Canvas as Source of Truth**: All grades, submissions, and completion status live in Canvas. NGAI database (Postgres) stores conversation logs and analytics only.

**Key Requirement**: Canvas-toolbox must support differential access (student-view vs instructor-view) for same course/module content.

---

## Existing Canvas-Toolbox Features We'll Use

Before listing new requirements, acknowledging existing features that are perfect for NGAI:

✅ **Grading Skills** (grader_fetch, grader_push, grader_push_comments)
- Used by QC agents to auto-grade and push approved comments

✅ **De-identification** (grader_deidentify_comments)
- Used to anonymize student data in analytics database

✅ **Roster Management** (people API access)
- Used for student authentication and enrollment validation

✅ **Submission Fetching**
- Used to verify "I turned it in" claims and handle multiple submissions

✅ **all-comments.md Pattern**
- Orchestrator will use similar markdown file pattern for instructor approval

**Request**: Documentation on recommended usage patterns for n8n integration (best practices for calling from workflows, error handling, rate limiting)

---

## New Feature Requests

### Feature 1: Teaching Sheet Pseudo-Generation

**Priority**: 🔴 **CRITICAL** (blocks peer agent evaluation of "ability to teach")

**Use Case**:
The NGAI platform requires students to demonstrate both **ability to do** (submit working code) and **ability to teach** (explain concepts to peer agent). Many Canvas courses lack formal "teaching evaluation" rubrics. Canvas-toolbox should generate pseudo-teaching-sheets from existing course materials.

**User Story**:
_As an orchestrator agent, I want to generate teaching evaluation criteria from a module's assignments, quizzes, and learning objectives, so that peer agents can assess student understanding even when instructor hasn't created formal teaching rubrics._

**Technical Requirements**:

**Input**:
- Canvas course ID
- Module ID or assignment ID
- Access token (instructor-level)

**Process**:
1. Fetch module/assignment metadata (title, description, learning objectives)
2. Fetch associated quiz questions (if any) - extract key concepts
3. Analyze assignment instructions for verbs (explain, demonstrate, implement)
4. Use LLM (GPT-4 or Claude) to generate teaching evaluation criteria:
   - What should student be able to explain?
   - What questions should student be able to answer?
   - What edge cases should student understand?
5. Output structured teaching-sheet format

**Output Format**:
```yaml
module_id: "module_3"
module_name: "Data Visualization with Polars"
teaching_criteria:
  core_concepts:
    - "Explain the difference between .select() and .filter()"
    - "Describe when to use lazy vs eager evaluation"
  application:
    - "Walk through your code line-by-line explaining what each method does"
    - "Explain how you chose this visualization type"
  edge_cases:
    - "What happens if the dataset is empty?"
    - "How would you handle missing values in this column?"
evaluation_rubric:
  advanced: "Student explains all concepts clearly with examples"
  proficient: "Student explains most concepts, minor gaps"
  developing: "Student struggles to explain reasoning"
  beginning: "Student cannot articulate understanding"
```

**Acceptance Criteria**:
- [ ] CLI command: `uv run python lib/tools/teaching_sheet_generate.py --module-id 123 --output teaching_sheets/module_3.yaml`
- [ ] Generates YAML file with teaching criteria for any module
- [ ] LLM API key configurable via .env (supports OpenAI and Anthropic)
- [ ] Handles modules without quizzes (uses assignment descriptions only)
- [ ] Includes cost tracking (log tokens used to generate sheet)

**Complexity**: ⚠️ **MEDIUM** (requires LLM integration, new output format)

**Suggested Implementation**:
- Add new `lib/tools/teaching_sheet_generate.py`
- Reuse existing Canvas API fetching logic
- Add LLM client in `lib/skills/llm_client.py` (supports multiple providers)
- Output to `teaching_sheets/<module_id>.yaml`

**Dependencies**:
- Canvas API: modules, assignments, quizzes endpoints
- LLM API: OpenAI or Anthropic (configurable)

---

### Feature 2: Remedial Assignment Cloning

**Priority**: 🟡 **HIGH** (needed for academic integrity workflow)

**Use Case**:
When QC agent detects a student may have cheated (can do but can't teach), the system needs to assign a remedial version of the same assignment with modified parameters (different data, different questions, same concepts).

**User Story**:
_As a QC agent, when I detect a student submitted working code but can't explain it, I want to clone the original assignment with modified values/scenarios and assign it only to that student, so they have a chance to learn through doing._

**Technical Requirements**:

**Input**:
- Canvas course ID
- Original assignment ID
- Target student Canvas user ID
- Modification strategy: "change_values" | "change_scenario" | "change_dataset"
- Optional: LLM-generated variations prompt

**Process**:
1. Fetch original assignment (title, description, instructions, rubric)
2. Parse instructions for parameterizable elements:
   - Numbers (e.g., "Calculate profit with revenue=$1000" → "$1500")
   - Dataset names (e.g., "penguins.csv" → "diamonds.csv")
   - Scenarios (e.g., "ice cream sales" → "book sales")
3. Generate modified version:
   - Option A: Simple regex substitution (for numeric values)
   - Option B: LLM prompt to rewrite instructions maintaining difficulty
4. Create new Canvas assignment:
   - Title: "{Original Title} - Remedial"
   - Published: false (unpublished by default)
   - Due date: +7 days from now
   - Assign to: only the specified student
5. Return new assignment ID

**Output**:
```json
{
  "original_assignment_id": 456,
  "remedial_assignment_id": 789,
  "student_canvas_id": 123,
  "modifications_applied": [
    "Changed revenue from $1000 to $1500",
    "Changed dataset from penguins to diamonds"
  ],
  "published": false,
  "due_date": "2026-07-06T23:59:00Z"
}
```

**Acceptance Criteria**:
- [ ] CLI command: `uv run python lib/tools/remedial_clone.py --assignment-id 456 --student-id 123 --strategy change_values`
- [ ] Creates new unpublished assignment in Canvas
- [ ] Assigns only to specified student
- [ ] Logs modifications made (for instructor review)
- [ ] Option to use LLM for intelligent rewriting (vs simple substitution)
- [ ] Preserves rubric from original (or creates simplified version)

**Complexity**: ⚠️ **MEDIUM** (Canvas Assignments API, optional LLM, parameter detection logic)

**Suggested Implementation**:
- Add `lib/tools/remedial_clone.py`
- Use Canvas Assignments API (POST /api/v1/courses/:id/assignments)
- Assignment override API for student-specific due dates
- Reuse LLM client from Feature 1 for intelligent variations

**Dependencies**:
- Canvas API: assignments, assignment overrides
- Optional: LLM API for intelligent rewriting

---

### Feature 3: Course Design Gap Analysis Audit

**Priority**: 🟢 **MEDIUM** (improves course quality, not blocking MVP)

**Use Case**:
The NGAI platform works best when Canvas courses have proper scaffolding: clear rubrics, teaching evaluations, module prerequisites, balanced assignment distribution. Canvas-toolbox should audit a course and report gaps.

**User Story**:
_As an instructor or orchestrator, I want to audit my Canvas course structure to identify missing rubrics, unbalanced modules, and missing teaching evaluations, so I can improve course design before launching NGAI agents._

**Technical Requirements**:

**Input**:
- Canvas course ID
- Audit config (YAML or JSON):
  ```yaml
  checks:
    - missing_rubrics
    - missing_prerequisites
    - module_balance  # flag modules with >6 assignments or <2
    - assignment_descriptions  # flag assignments with <50 char description
    - teaching_evaluations  # check for teaching-related assessments
  ```

**Process**:
1. Fetch full course structure (modules, assignments, quizzes, rubrics)
2. Run checks:
   - **Missing Rubrics**: assignments worth >10 points with no rubric
   - **Missing Prerequisites**: modules with no prerequisite chain
   - **Module Balance**: modules with unusual assignment counts (too many/few)
   - **Weak Descriptions**: assignments with short or generic descriptions
   - **No Teaching Evals**: course has no assignments tagged "teaching" or "reflection"
3. Generate report with recommendations

**Output Format**:
```markdown
# Course Design Audit Report
**Course**: DS250 - Data Science Programming
**Audited**: 2026-06-29

## Summary
- ✅ 12/14 assignments have rubrics
- ⚠️ 3/7 modules have no prerequisites
- ❌ 0 teaching evaluation assignments found

## Detailed Findings

### Missing Rubrics (2)
- Assignment: "Final Project" (100 points) - **HIGH PRIORITY**
- Assignment: "Midterm Reflection" (20 points)

### Module Prerequisites
- Module 3: "Data Cleaning" has no prerequisite (recommend: Module 2)
- Module 5: "APIs" has no prerequisite (recommend: Module 4)

### Module Balance
- ⚠️ Module 7: "Machine Learning" has 8 assignments (avg: 3.5)
  - Recommendation: Split into ML Part 1 and Part 2

### Teaching Evaluations
- ❌ No teaching-focused assignments found
- Recommendation: Add "Teaching Sheet" after each module where student explains concepts to peer

## NGAI Compatibility Score: 6.5/10
Recommendations:
1. Add rubric to Final Project (CRITICAL for QC agent grading)
2. Create 7 teaching sheets (one per module) - use `teaching_sheet_generate.py`
3. Set module prerequisites to enforce linear progression
```

**Acceptance Criteria**:
- [ ] CLI command: `uv run python lib/tools/course_audit.py --course-id 12345 --output audit_report.md`
- [ ] Generates markdown report with findings + recommendations
- [ ] NGAI compatibility score (0-10) based on scaffolding quality
- [ ] Actionable recommendations with priority levels
- [ ] Checks configurable via YAML/JSON

**Complexity**: ⚠️ **MEDIUM-HIGH** (complex analysis logic, multiple Canvas API calls, heuristics)

**Suggested Implementation**:
- Add `lib/tools/course_audit.py`
- Create audit engine in `lib/skills/course_analyzer.py`
- Configurable checks via `config/audit_checks.yaml`
- Markdown report generator

**Dependencies**:
- Canvas API: courses, modules, assignments, quizzes, rubrics

---

### Feature 4: Differential Content Access (Student-View vs Instructor-View)

**Priority**: 🔴 **CRITICAL** (peer agents must not see answer keys)

**Use Case**:
Peer agents (bottom layer) should access Canvas content exactly as students see it (no answer keys, no unpublished assignments). QC agents (middle layer) need instructor-view with full access. Currently, canvas-toolbox uses instructor API token for everything.

**User Story**:
_As a peer agent, I want to fetch module content with student-view permissions so I cannot accidentally leak quiz answers or see instructor-only materials._

**Technical Requirements**:

**Input**:
- Canvas course ID
- Module/Assignment ID
- Access mode: `student_view` | `instructor_view`
- For student_view: optionally provide Canvas user ID to impersonate

**Process**:
- **Student View Mode**:
  - Use Canvas "student view" feature or test student account
  - Fetch only published content
  - Quiz questions WITHOUT answers
  - Assignment descriptions without solution files
  - No rubric answer keys (only rubric criteria)
- **Instructor View Mode**:
  - Current behavior (full access)
  - Quiz questions WITH answers
  - All files including solutions
  - Full rubrics

**Implementation Options**:

**Option A**: Use Canvas Student View API
- Canvas has `/api/v1/courses/:id/student_view_student` endpoint
- Creates/retrieves test student
- Make API calls as that student

**Option B**: Dual-token approach
- Store both instructor token and limited student token in .env
- Switch token based on access_mode parameter

**Option C**: API response filtering
- Fetch with instructor token
- Strip sensitive fields before returning (answers, solutions)

**Acceptance Criteria**:
- [ ] CLI command: `uv run python lib/tools/module_fetch.py --module-id 123 --access-mode student_view`
- [ ] Returns module content without answer keys when `access_mode=student_view`
- [ ] Returns full content including answers when `access_mode=instructor_view`
- [ ] Quiz questions in student_view do NOT include `correct_comments` or `answers` fields
- [ ] Clearly documented which fields are redacted in student_view

**Complexity**: ⚠️ **MEDIUM** (requires understanding Canvas permission model, API filtering logic)

**Suggested Implementation**:
- Modify existing fetch tools to accept `--access-mode` flag
- Add `lib/skills/student_view_filter.py` to strip sensitive fields
- Document which endpoints support student view API

**Dependencies**:
- Canvas API: student_view_student endpoint or dual-token setup

---

### Feature 5: Real-Time Comment De-Identification

**Priority**: 🟢 **LOW** (nice-to-have, workaround exists)

**Use Case**:
When peer agent discusses graded feedback with student, it needs to reference instructor comments. These comments may contain student names or identifying info. Canvas-toolbox should de-identify comments in real-time (faster than current batch process).

**User Story**:
_As a peer agent, when I fetch a student's graded assignment to discuss feedback, I want comments to be de-identified on-the-fly so I can quote them without privacy concerns._

**Current State**:
`grader_deidentify_comments.py` exists but designed for batch processing of all-comments files.

**Technical Requirements**:

**Input**:
- Raw comment text (string)
- Optional: student name to redact

**Output**:
- De-identified comment text

**Process**:
1. Detect and redact student names (replace with "[Student]")
2. Detect and redact instructor names (replace with "[Instructor]")
3. Redact email addresses
4. Redact any personally identifiable info
5. Preserve feedback content

**Example**:
```python
Input: "Great work, Sarah! Your explanation of Polars was clear. Email me at prof@university.edu if you have questions."

Output: "Great work, [Student]! Your explanation of Polars was clear. Email me at [Instructor Email] if you have questions."
```

**Acceptance Criteria**:
- [ ] Python function: `deidentify_text(text: str, student_name: str = None) -> str`
- [ ] Can be imported and called from n8n Python code node
- [ ] Handles common PII patterns (names, emails, phone numbers)
- [ ] Fast (<100ms for typical comment)
- [ ] Does not redact course content (e.g., "Polars" should stay)

**Complexity**: 🟢 **LOW** (mostly regex, reuse existing logic)

**Suggested Implementation**:
- Refactor `grader_deidentify_comments.py` to expose `deidentify_text()` function
- Make it importable: `from canvas_toolbox.lib.skills.deidentification import deidentify_text`
- Add unit tests with various PII patterns

**Dependencies**: None

---

### Feature 6: Quiz Question Differential Fetch

**Priority**: 🔴 **CRITICAL** (peer agents deliver quizzes, QC validates answers)

**Use Case**:
Peer agents need quiz questions to deliver conversationally. QC agents need questions + correct answers to validate student responses. Single fetch tool should support both modes.

**User Story**:
_As a peer agent, I want to fetch quiz questions without answers so I can deliver them to students. As a QC agent, I want the same quiz WITH answers so I can grade responses._

**Technical Requirements**:

**Input**:
- Canvas course ID
- Quiz ID
- Access mode: `questions_only` | `questions_with_answers`

**Output (questions_only)**:
```json
{
  "quiz_id": 789,
  "quiz_name": "Module 3 Quiz: Data Visualization",
  "questions": [
    {
      "id": 1,
      "question_text": "What does .select() do in Polars?",
      "question_type": "multiple_choice",
      "answers": [
        {"id": "a", "text": "Filters rows"},
        {"id": "b", "text": "Selects columns"},
        {"id": "c", "text": "Sorts data"}
      ]
      // NO "correct_answer" field
    }
  ]
}
```

**Output (questions_with_answers)**:
```json
{
  "quiz_id": 789,
  "questions": [
    {
      "id": 1,
      "question_text": "What does .select() do in Polars?",
      "correct_answer": "b",
      "correct_comments": "Correct! .select() chooses columns."
    }
  ]
}
```

**Acceptance Criteria**:
- [ ] CLI command: `uv run python lib/tools/quiz_fetch.py --quiz-id 789 --mode questions_only`
- [ ] Returns questions without answers when `mode=questions_only`
- [ ] Returns questions with answers when `mode=questions_with_answers`
- [ ] Handles all Canvas quiz question types (multiple choice, true/false, short answer, essay)
- [ ] Error if quiz is not yet published

**Complexity**: 🟢 **LOW-MEDIUM** (Canvas Quiz API, response filtering)

**Suggested Implementation**:
- Add `lib/tools/quiz_fetch.py`
- Use Canvas Quiz Questions API: `/api/v1/courses/:id/quizzes/:quiz_id/questions`
- Filter response based on `mode` parameter
- Reuse filtering logic from Feature 4

**Dependencies**:
- Canvas API: quizzes, quiz_questions

---

### Feature 7: Multi-Submission Tracker

**Priority**: 🟡 **HIGH** (students may resubmit multiple times)

**Use Case**:
Students can submit assignments multiple times in Canvas. When student tells peer agent "I turned it in", system needs to verify AND track which submission version. QC agent needs to grade the latest (or instructor-selected) submission.

**User Story**:
_As a QC agent, when a student has submitted an assignment 3 times, I want to fetch all submissions with timestamps so I can grade the most recent one (or the one instructor specified)._

**Current State**:
Canvas-toolbox likely fetches submissions, but may not expose multi-submission metadata clearly.

**Technical Requirements**:

**Input**:
- Canvas course ID
- Assignment ID
- Student Canvas user ID

**Output**:
```json
{
  "assignment_id": 456,
  "student_id": 123,
  "submission_count": 3,
  "submissions": [
    {
      "attempt": 1,
      "submitted_at": "2026-06-15T14:30:00Z",
      "grade": null,
      "workflow_state": "submitted"
    },
    {
      "attempt": 2,
      "submitted_at": "2026-06-16T10:15:00Z",
      "grade": 85,
      "workflow_state": "graded"
    },
    {
      "attempt": 3,
      "submitted_at": "2026-06-17T09:00:00Z",
      "grade": null,
      "workflow_state": "pending_review"
    }
  ],
  "latest_submission": {
    "attempt": 3,
    "url": "https://canvas.example.edu/...",
    "attachments": ["solution.py"]
  }
}
```

**Acceptance Criteria**:
- [ ] CLI command: `uv run python lib/tools/submission_tracker.py --assignment-id 456 --student-id 123`
- [ ] Returns all submissions with attempt numbers and timestamps
- [ ] Clearly marks which is the latest/gradeable submission
- [ ] Handles case where no submissions exist (returns empty list, not error)
- [ ] Includes submission file URLs for download

**Complexity**: 🟢 **LOW** (Canvas Submissions API is well-documented)

**Suggested Implementation**:
- Add `lib/tools/submission_tracker.py`
- Use Canvas Submissions API: `/api/v1/courses/:id/assignments/:assignment_id/submissions/:user_id`
- Include `?include[]=submission_history` to get all attempts
- Parse and format response

**Dependencies**:
- Canvas API: submissions

---

## Integration & Compatibility Requirements

### n8n Integration Pattern

**How n8n will call canvas-toolbox**:

**Option A (MVP)**: Execute Command Node
- n8n Docker container has Python + canvas-toolbox installed (custom Dockerfile)
- n8n Execute Command node runs: `uv run python lib/tools/teaching_sheet_generate.py --module-id 123`
- Parse stdout JSON/YAML into n8n variables
- Environment: `.env` from ds250-onln-master mounted as Docker volume

**Option B (Future)**: Canvas-Toolbox as Microservice
- Wrap canvas-toolbox in FastAPI REST API
- n8n calls via HTTP Request node: `POST /api/teaching-sheet/generate`
- Returns JSON response

**Request**: For Option A to work smoothly, all new tools should:
- ✅ Accept parameters via CLI flags (not interactive prompts)
- ✅ Output structured data to stdout (JSON or YAML, not human-readable tables)
- ✅ Use exit codes: 0 = success, non-zero = error
- ✅ Write logs/errors to stderr (not stdout, so n8n can parse stdout cleanly)
- ✅ Support `--help` flag with clear parameter documentation

**Example Good Output**:
```bash
$ uv run python lib/tools/quiz_fetch.py --quiz-id 789 --mode questions_only
{"quiz_id": 789, "questions": [...]}  # stdout = JSON
$ echo $?
0  # exit code
```

**Example Bad Output** (hard to parse in n8n):
```bash
$ uv run python lib/tools/quiz_fetch.py
Enter quiz ID: _  # interactive prompt - breaks automation
Fetching quiz...
Quiz: Module 3 Quiz  # human-readable, not structured
Questions: [...]  # mixed output
```

---

### Error Handling Standards

For all new tools, request consistent error format:

```json
{
  "error": true,
  "error_code": "QUIZ_NOT_FOUND",
  "message": "Quiz ID 789 not found in course 12345",
  "timestamp": "2026-06-29T10:30:00Z"
}
```

Exit code should be non-zero (e.g., 1 for generic error, specific codes for Canvas API errors).

---

### Canvas API Rate Limiting

NGAI platform will make frequent Canvas API calls:
- 100 students × 10 messages/day = potential 1000+ API calls/day
- Need to respect Canvas API rate limits (typically 3000 req/hour)

**Request**:
- Add rate limiting helper in canvas-toolbox
- Exponential backoff on 429 (rate limit) responses
- Optional caching layer for frequently-accessed content (module descriptions, rubrics)

**Example**:
```python
# lib/skills/rate_limiter.py
from canvas_toolbox.lib.skills.rate_limiter import with_rate_limit

@with_rate_limit
def fetch_module(module_id):
    # automatically throttles if approaching rate limit
    ...
```

---

## Timeline & Prioritization

### Phase 1 (MVP - Needed for pilot)
**Target**: 4-6 weeks

1. **Feature 4**: Differential Content Access (CRITICAL)
2. **Feature 6**: Quiz Question Differential Fetch (CRITICAL)
3. **Feature 1**: Teaching Sheet Pseudo-Generation (CRITICAL)
4. **Feature 7**: Multi-Submission Tracker (HIGH)

**Deliverable**: NGAI platform can run pilot with 10-20 students in one module

---

### Phase 2 (Production - Needed for full deployment)
**Target**: 8-10 weeks

5. **Feature 2**: Remedial Assignment Cloning (HIGH)
6. **Feature 3**: Course Design Gap Analysis (MEDIUM)
7. **Feature 5**: Real-Time Comment De-Identification (LOW)

**Deliverable**: NGAI platform can handle 100 students across full course with academic integrity features

---

## Testing & Validation

For each feature, canvas-toolbox team should provide:

1. **Unit tests** (pytest)
2. **Integration tests** against Canvas test instance
3. **Example CLI usage** in feature documentation
4. **Example output** (sample JSON/YAML for n8n parsing)

NGAI team will provide:
- Canvas sandbox course for testing (course ID to be shared)
- Sample scenarios for each feature
- n8n workflow test harness

---

## Questions & Collaboration

**NGAI Team Contact**: Chaz Clark (project lead)
**Canvas-Toolbox Repo**: [Link to canvas-toolbox GitHub]

**Open Questions** (for discussion):

1. **LLM Provider**: Should canvas-toolbox standardize on one LLM library (e.g., LangChain) or support multiple providers directly?
2. **Caching Strategy**: Should canvas-toolbox include Redis caching for Canvas API responses, or leave that to calling application (n8n)?
3. **Async Support**: Would async Python (asyncio) improve performance for batch operations?
4. **TypeScript Rewrite**: Any plans to rewrite canvas-toolbox in TypeScript for n8n custom nodes?

**How to Collaborate**:
- File GitHub issues for each feature with label `ngai-platform`
- Weekly sync meeting to review progress?
- Shared Slack/Discord channel for quick questions?

---

## Success Criteria

Canvas-toolbox successfully supports NGAI platform when:

✅ Peer agents can fetch student-view content without seeing answers
✅ QC agents can auto-grade using canvas-toolbox + LLM
✅ Teaching sheets can be generated for any module
✅ Remedial assignments can be cloned on-demand
✅ n8n workflows can call all tools via Execute Command with clean JSON output
✅ Rate limiting prevents Canvas API 429 errors
✅ All features have CLI `--help` documentation

---

**Thank you for considering these enhancements!** The NGAI platform's success depends heavily on canvas-toolbox's capabilities. We're excited to build this symbiotic partnership and contribute back to the canvas-toolbox project where possible.

---

**Appendix A: Sample n8n Workflow Calling Canvas-Toolbox**

```json
{
  "nodes": [
    {
      "name": "Execute Canvas-Toolbox",
      "type": "n8n-nodes-base.executeCommand",
      "parameters": {
        "command": "=uv run python /canvas-toolbox/lib/tools/quiz_fetch.py --quiz-id {{$json.quiz_id}} --mode questions_only"
      }
    },
    {
      "name": "Parse JSON Output",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "jsCode": "const output = $input.item.json.stdout;\nreturn JSON.parse(output);"
      }
    }
  ]
}
```

**Key Point**: Clean stdout (JSON only) makes n8n parsing trivial.

---

**Appendix B: Feature Dependency Graph**

```
Feature 4 (Differential Access) ─┬─> Feature 6 (Quiz Fetch)
                                 └─> Feature 1 (Teaching Sheets)
                                      └─> Feature 3 (Gap Analysis)

Feature 7 (Submission Tracker) ──> Feature 2 (Remedial Clone)

Feature 5 (Comment De-ID) ────> (standalone, no deps)
```

**Implementation Order**: Follow dependency graph for smoothest development.
