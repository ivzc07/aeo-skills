# AEO Usage Examples

This document provides detailed examples of using AEO skills in various scenarios.

## Example 1: Small Autonomous Task

**User Request:** "Add email validation to the signup form"

### Session Flow

```
User: /aeo

AEO-Core: [Loading aeo-spec-validator]
          [Loading aeo-core]

          --- PHASE 0: SPEC VALIDATION ---

AEO-Spec-Validator: Analyzing task...
                   Task: "Add email validation to the signup form"

                   Scoring:
                   • Objective clarity: 15/15 (explicit)
                   • Acceptance criteria: 12/15 (some details missing)
                   • Context: 8/10 (partial context)
                   • Dependencies: 5/10 (some mentioned)
                   • Tech stack: 5/10 (general)
                   • Test requirements: 8/10 (general testing)
                   • Performance/security: 5/10 (missing perf req)
                   • Scope: 10/10 (single file)
                   • Complexity: 8/10 (simple)

                   Spec Score: 78/100 - MINOR GAPS

                   Assumptions:
                   • Using existing test framework
                   • Standard RFC 5322 validation
                   • No special performance requirements
                   • Using React Hook Form

                   Proceeding with these assumptions...

AEO-Core: --- PHASE 1: CONFIDENCE CALCULATION ---

          Base Confidence: 0.50
          Clarity: +0.30 (acceptance criteria, tech stack)
          Context: +0.10 (similar task in memory)
          Risk: -0.00 (low risk area)
          Spec Adjustment: +0.10 (spec ≥80)
          Security Multiplier: ×1.0 (not security-critical)

          Final Confidence: 0.92

          Decision: AUTONOMOUS (≥0.85)

AEO-Core: ✅ Confidence: 0.92 - Proceeding autonomously

          [Implementing email validation...]

AEO-Core: [Implementation complete]
          [Invoking aeo-qa-agent]

AEO-QA-Agent: Scanning files...

              🔧 AUTO-FIXES APPLIED
              • Removed 2 console.log statements (signup.tsx:23, 45)
              • Extracted constant MAX_EMAIL_LENGTH (validation.ts:12)

              ✅ QA REVIEW PASSED

              Changed Files: 2
              Security Issues: 0
              Code Smells: 0
              Test Coverage: ✓
              Architecture: ✓

AEO-Core: Ready to commit changes.

User: Yes

AEO-Core: [Committing changes]
          [Recording signal to memory]

Signal: {
  "timestamp": "2026-01-22T10:30:00Z",
  "task_id": "task-456",
  "predicted_confidence": 0.92,
  "actual_difficulty": "easy",
  "success": true,
  "adjustment": +0.05
}

✅ Task Complete
```

### Outcome

- **Execution Mode:** Fully autonomous
- **Time:** ~3 minutes
- **Human Interaction:** 1 confirmation
- **Confidence:** 0.92 (correctly predicted easy task)
- **Learning:** +0.05 adjustment (future similar tasks more confident)

---

## Example 2: Medium Task with Advisory Mode

**User Request:** "Create a user dashboard showing metrics"

### Session Flow

```
User: /aeo

AEO-Core: [Loading skills...]

          --- PHASE 0: SPEC VALIDATION ---

AEO-Spec-Validator: Analyzing task...
                   Task: "Create a user dashboard showing metrics"

                   Scoring:
                   • Objective clarity: 10/15 (somewhat vague)
                   • Acceptance criteria: 8/15 (general criteria)
                   • Context: 5/10 (which metrics?)
                   • Dependencies: 5/10 (data source unclear)
                   • Tech stack: 8/10 (React, Chart.js mentioned)
                   • Test requirements: 7/10 (some testing)
                   • Performance/security: 5/10 (missing)
                   • Scope: 5/10 (multiple pages, unclear)
                   • Complexity: 5/10 (moderate)

                   Spec Score: 58/100 - MAJOR GAPS

                   ❌ SPEC NEEDS CLARIFICATION

                   Missing Details:
                   • Which metrics to show? (users, revenue, engagement?)
                   • Data source? (API, database?)
                   • Dashboard pages? (single page, tabs?)
                   • Time range? (day, week, month?)
                   • Interactivity? (filters, export?)

                   Please clarify:
                   1. Which metrics should be displayed?
                   2. What is the data source?
                   3. Single page or multiple tabs?
                   4. What time ranges should be supported?

User: User engagement metrics - DAU, MAU, session length. Data from Mixpanel API. Single page with time range picker (day/week/month).

AEO-Spec-Validator: Recalculating with added context...

                   Updated Score: 82/100

AEO-Core: --- PHASE 1: CONFIDENCE CALCULATION ---

          Base Confidence: 0.50
          Clarity: +0.30 (now well-defined)
          Context: +0.00 (no similar tasks in memory)
          Risk: -0.10 (new feature, moderate scope)
          Spec Adjustment: +0.10 (spec ≥80)
          Security Multiplier: ×1.0 (not security-critical)

          Final Confidence: 0.78

          Decision: ADVISORY (0.70-0.84)

AEO-Core: ⚡ ADVISORY - Confidence: 0.78

          [Acceptance criteria defined]
          [Tech stack: React, Chart.js, Mixpanel API]
          [Data: DAU, MAU, session length]
          [Scope: Single page with time range picker]

          I'll proceed autonomously but will pause if issues arise.

          [Implementing dashboard...]

          [After 5 minutes, encounters issue]

AEO-Failure-Patterns: Error: Mixpanel API rate limit exceeded
                      Pattern matched: Rate limiting (confidence: 0.85)

                      Auto-fixing: Adding request queuing and retry logic...

                      ✅ Fixed: Implemented rate limit handling

          [Implementation continues...]

AEO-Core: [Implementation complete]
          [Invoking aeo-qa-agent]

AEO-QA-Agent: Scanning files...

              ✅ QA REVIEW PASSED

              Changed Files: 3
              Security Issues: 0
              Code Smells: 0 (1 auto-fixed)
              Test Coverage: ✓
              Architecture: ✓

AEO-Core: Ready to commit changes.

User: Yes

AEO-Core: [Committing changes]
          [Recording signal to memory]

Signal: {
  "timestamp": "2026-01-22T11:15:00Z",
  "task_id": "task-457",
  "predicted_confidence": 0.78,
  "actual_difficulty": "medium",
  "success": true,
  "adjustment": 0.00
}
```

### Outcome

- **Execution Mode:** Advisory with one clarification
- **Time:** ~12 minutes
- **Human Interaction:** 2 interactions (clarification, commit approval)
- **Confidence:** 0.78 (accurately predicted medium difficulty)
- **Auto-fix Applied:** Rate limit handling

---

## Example 3: Large Task Requiring Escalation

**User Request:** "Build a complete project management system"

### Session Flow

```
User: /aeo

AEO-Core: [Loading skills...]

          --- PHASE 0: SPEC VALIDATION ---

AEO-Spec-Validator: Analyzing task...
                   Task: "Build a complete project management system"

                   Scoring:
                   • Objective clarity: 3/15 (extremely vague)
                   • Acceptance criteria: 0/15 (none defined)
                   • Context: 2/10 (no context)
                   • Dependencies: 0/10 (none listed)
                   • Tech stack: 0/10 (none specified)
                   • Test requirements: 0/10 (none)
                   • Performance/security: 0/10 (none)
                   • Scope: 0/10 (massive scope)
                   • Complexity: 0/10 (high complexity)

                   Spec Score: 5/100 - UNACCEPTABLE

AEO-Core: ❌ CANNOT EXECUTE - INSUFFICIENT CONFIDENCE

          Confidence: 0.18
          Threshold: 0.70

          Why:
          • Spec score: 5/100 (below 40 threshold)
          • No acceptance criteria defined
          • Extremely vague scope
          • No tech stack specified
          • No dependencies listed

          What's needed:
          1. Clear acceptance criteria
          2. Feature scope definition
          3. Tech stack selection
          4. Dependency identification
          5. Test requirements

          Example of a good spec:
          "Create a project management system with the following features:
           - User authentication (JWT)
           - Project CRUD operations
           - Task assignment with due dates
           - Progress tracking dashboard

           Tech stack:
           - Backend: Node.js, Express, PostgreSQL
           - Frontend: React, TypeScript
           - Testing: Jest, Cypress

           Acceptance Criteria:
           - Users can create, read, update, delete projects
           - Tasks can be assigned to users with due dates
           - Dashboard shows project and task progress
           - All actions authenticated with JWT"

AEO-Escalation: [Invoking escalation]

               ❌ CANNOT PROCEED - SPEC TOO UNCLEAR

               Spec Score: 5/100
               Threshold: 40 (Minimum)

               Why this was refused:
               • Objective vague (3/15) - "complete system" too broad
               • No acceptance criteria (0/15)
               • Missing context (2/10)
               • No dependencies listed (0/10)
               • Ambiguity penalty: -5

               Options:
               1. Provide improved spec - I'll wait for your detailed specification
               2. Answer clarifying questions - I'll ask specific questions to build spec
               3. Use template - I'll provide a spec template for you to fill

               Recommended: Option 2 - Answer questions to build spec together

User: 2

AEO-Escalation: Let me ask some clarifying questions to build a good spec:

               Question 1: What specific features do you need in the project management system?

User: We need project creation, task management, team assignment, file attachments, and progress reporting.

AEO-Escalation: Question 2: Who are the users? (admins, project managers, team members?)

User: Project managers create projects, team members see assigned tasks, admins manage users.

AEO-Escalation: Question 3: What tech stack should we use?

User: Node.js backend with Express, React frontend, PostgreSQL database.

AEO-Escalation: Question 4: Any specific requirements for authentication, file storage, notifications?

User: JWT authentication, S3 for file storage, email notifications for due dates.

AEO-Escalation: [Building improved spec...]

               Let me create a comprehensive spec based on your answers:

               **Project Management System Specification**

               **Objective:** Build a web-based project management system for teams

               **Features:**
               1. User authentication (JWT)
               2. Project CRUD (create, read, update, delete)
               3. Task management with due dates
               4. Team member assignment
               5. File attachments (S3)
               6. Email notifications for due dates
               7. Progress reporting dashboard

               **Users:**
               - Admins: Manage users, projects
               - Project Managers: Create projects, assign tasks
               - Team Members: View tasks, update status

               **Tech Stack:**
               - Backend: Node.js, Express, PostgreSQL
               - Frontend: React, TypeScript
               - Storage: AWS S3
               - Auth: JWT with bcrypt

               **Acceptance Criteria:**
               - Users can register/login with email
               - Project managers can create projects
               - Tasks can be assigned to team members
               - Files can be attached to tasks (max 10MB)
               - Email notifications sent for due dates
               - Dashboard shows project progress

               Does this specification look good? Should I proceed with breaking this down into stories?

User: Yes, that looks good.

AEO-Core: [Recalculating confidence...]
          New confidence: 0.42

          Still below threshold (0.70) - too large for autonomous execution

          Recommendation: Break into 5-7 smaller stories

AEO-Escalation: [Proposing breakdown...]

               Options:
               1. Break into stories now - I'll create story breakdown
               2. Use Ralph execution loop - Ralph will manage breakdown and execution
               3. Create detailed project plan - Full architecture first

               Recommended: Option 2 - Use Ralph for execution management

User: 2

AEO-Core: [Invoking Ralph integration]
          [Generating PRD from spec...]

          PRD generated successfully.

          Ralph will now manage execution:
          - Break down into stories
          - Execute story by story
          - AEO will validate each task
          - QA review before commits

          Run: ralph execute prd.json

User: [Runs ralph execute prd.json]

[Ralph Session]:
ralph> Reading PRD...
ralph> Breaking down into 6 stories with 24 tasks...

[Story 1: Authentication System]
ralph> [Task 1.1: Set up user registration]
aeo> Spec score: 88/100 - well-defined
aeo> Confidence: 0.89 - AUTONOMOUS
ralph> ✅ Implementing...
aeo> QA: ✅ Passed - 1 auto-fix applied
ralph> ✅ Committed

ralph> [Task 1.2: Implement login with JWT]
aeo> Spec score: 85/100
aeo> Confidence: 0.87 - AUTONOMOUS
ralph> ✅ Implementing...
aeo> QA: ✅ Passed
ralph> ✅ Committed

[... continues through all stories ...]

ralph> All stories complete!
ralph> 24 tasks executed
ralph> 17 autonomous executions
ralph> 5 required clarification
ralph> 2 escalated for human decision
ralph> 0 failed

AEO-Core: [Recording all signals to memory]
          [Calculating final adjustments]

          Summary:
          Average confidence: 0.76
          Average actual difficulty: medium
          Success rate: 100%
          Learning adjustment: +0.02 (system becoming more confident)
```

### Outcome

- **Execution Mode:** Escalation → Ralph-managed breakdown
- **Time:** ~2 hours (across multiple sessions)
- **Human Interaction:** 5 interactions (spec clarification, breakdown confirmation)
- **Confidence:** Started at 0.18, improved to 0.42 after clarification
- **Learning:** System learned that well-defined specs enable autonomous execution

---

## Example 4: Security Veto and Fix

**User Request:** "Add API endpoint to get user data"

### Session Flow

```
User: /aeo

AEO-Core: [Standard validation flow]
          Confidence: 0.85 - AUTONOMOUS

          [Implementing GET /api/users/:id endpoint]

AEO-Core: [Implementation complete]
          [Invoking aeo-qa-agent]

AEO-QA-Agent: Scanning files...

              🚨 VETO - SECURITY ISSUE DETECTED

              File: src/routes/users.js:47
              Issue: SQL Injection vulnerability

              Current Code:
              ```javascript
              app.get('/api/users/:id', (req, res) => {
                const query = `SELECT * FROM users WHERE id = ${req.params.id}`
                const users = await db.query(query)
                res.json(users)
              })
              ```

              Why this is dangerous:
              • User input directly interpolated into SQL
              • Attacker can execute arbitrary SQL
              • Can expose all user data

              Required Fix:
              ```javascript
              app.get('/api/users/:id', (req, res) => {
                const query = 'SELECT * FROM users WHERE id = $1'
                const users = await db.query(query, [req.params.id])
                res.json(users)
              })
              ```

              Action: BLOCKED - Fix required before commit

AEO-Escalation: [Presenting options]

               Options:
               1. Fix immediately - I'll apply the secure version now
               2. Review fix together - I'll show you the fix for approval
               3. Explain more - I'll provide detailed explanation

               Recommended: Option 1 - Fix immediately (critical security issue)

User: 1

AEO-QA-Agent: Applying fix...

              [Updating src/routes/users.js:47]
              ✅ Fixed: Changed to parameterized query

              Re-scanning...
              ✅ QA REVIEW PASSED

              Security Issues: 0
              [All other checks passed]

AEO-Core: Ready to commit secure version.

User: Yes

AEO-Core: [Committing with security fix]
```

### What Happened

1. **AEO detected vulnerability:** SQL injection in user-provided code
2. **QA vetoed the change:** Blocked commit until fixed
3. **Escalation presented options:** With recommendation to fix immediately
4. **Auto-fix applied:** Changed to parameterized query
5. **Re-review passed:** Security issue resolved
6. **Safe commit:** Only secure code committed

### Learning Signal Recorded

```json
{
  "escalation_type": "qa_veto",
  "issue": "sql_injection",
  "resolution": "auto_fixed",
  "success": true,
  "learning": "Security vetos with clear fixes should auto-immediately"
}
```

---

## Example 5: Error Pattern Auto-Fix

**User Request:** "Fix the failing tests"

### Session Flow

```
User: /aeo

AEO-Core: [Standard validation]
          Confidence: 0.88 - AUTONOMOUS

          [Running tests...]

Test Output:
❌ Error: Timeout - Async callback was not invoked within the 5000ms timeout

AEO-Failure-Patterns: [Pattern matching]
                      Error matched: "Async callback timeout"
                      Confidence: 0.90
                      Pattern ID: #4

                      Auto-fixing...
                      [Reading test file]

                      Current code:
                      ```javascript
                      test('fetches user data', () => {
                        fetchUser('user-123')
                          .then(user => {
                            expect(user.name).toBe('Test User')
                          })
                      })
                      ```

                      Issue: Missing done() callback or async/await

                      Applying fix:
                      ```javascript
                      test('fetches user data', async () => {
                        const user = await fetchUser('user-123')
                        expect(user.name).toBe('Test User')
                      })
                      ```

                      ✅ Fixed

                      [Re-running tests...]
                      ✅ All tests passing

AEO-QA-Agent: Scanning files...
              ✅ QA REVIEW PASSED

AEO-Core: Ready to commit.

User: Yes

AEO-Core: [Committing with test fix]
```

### What Happened

1. **Error detected:** Jest async timeout
2. **Pattern matched:** 20 core patterns (ID #4, confidence 0.90)
3. **Auto-fix applied:** Converted to async/await
4. **Tests passed:** Fix resolved the issue
5. **QA approved:** Code review passed
6. **Learning recorded:** Pattern success count incremented

---

## Example 6: Cost Management During Execution

**User Request:** "Refactor the database schema"

### Session Flow

```
User: /aeo

AEO-Cost-Governor: [Checking budget...]

                    Daily Budget: $10.00
                    Used: $8.47 (84.7%)
                    Remaining: $1.53

                    Task Estimate: ~$2.50
                    ⚠️ ALERT: Task may exceed daily budget

AEO-Escalation: [Presenting options]

                    Options:
                    1. Continue anyway - May exceed budget
                    2. Use Haiku instead - Estimated cost ~$0.50
                    3. Break into smaller tasks - Spread across multiple days

                    Recommended: Option 3 - Break into smaller tasks

User: 3

AEO-Cost-Governor: Suggesting subtask breakdown:

                    Day 1 (budget remaining: $1.53):
                    1. Analyze current schema ($0.30)
                    2. Design new schema structure ($0.40)
                    3. Create migration plan ($0.50)

                    Day 2 (new budget: $10.00):
                    4. Implement migration v1 ($1.00)
                    5. Test migration rollback ($0.50)

                    Day 3 (new budget: $10.00):
                    6. Run full migration ($1.00)
                    7. Update application code ($1.50)

                    Total estimated: $5.20 over 3 days

User: Good plan, proceed with Day 1.

AEO-Core: [Proceeding with subtasks...]
          [Completes Day 1 tasks within budget]

AEO-Cost-Governor: [Recording costs]
                    Day 1 used: $1.21
                    Remaining: $0.32

                    [Archiving daily log to aeo-costs.jsonl]

AEO-Core: Day 1 complete. Resume tomorrow for Day 2?
```

### What Happened

1. **Cost detected:** Task would exceed daily budget
2. **Escalation presented:** Options to manage costs
3. **Breakdown chosen:** Split into 3-day subtask plan
4. **Day 1 completed:** Within remaining budget
5. **Resumed next day:** Fresh daily budget for Day 2

### Cost Tracking Recorded

```json
{"date":"2026-01-22","task_id":"task-day1-1","model":"claude-sonnet-4.5","cost_usd":0.30}
{"date":"2026-01-22","task_id":"task-day1-2","model":"claude-sonnet-4.5","cost_usd":0.40}
{"date":"2026-01-22","task_id":"task-day1-3","model":"claude-sonnet-4.5","cost_usd":0.51}
// Day 1 total: $1.21 (under $1.53 remaining)
```

---

## Summary of Modes

| Example | Confidence | Mode | Human Interactions | Outcome |
|---------|------------|------|-------------------|----------|
| 1. Email validation | 0.92 | Autonomous | 1 | ✅ Success (easy) |
| 2. Dashboard | 0.78 | Advisory | 2 | ✅ Success (medium) |
| 3. Project management | 0.18 → 0.42 | Escalation → Ralph | 5 | ✅ Success (broken down) |
| 4. API endpoint | 0.85 | Autonomous → QA Veto | 1 | ✅ Success (security fix) |
| 5. Test fix | 0.88 | Autonomous | 1 | ✅ Success (auto-fixed) |
| 6. Schema refactor | N/A | Cost escalation | 2 | ✅ Success (split across days) |

## Key Takeaways

1. **Well-defined specs = higher confidence** - Example 1 vs Example 3
2. **AEO learns from outcomes** - Each task records learning signal
3. **Security is non-negotiable** - QA veto overrides everything
4. **Errors get auto-fixed** - When pattern confidence ≥0.85
5. **Cost awareness** - Budget constraints trigger escalation
6. **Escalation presents clear options** - Never leaves human stuck
