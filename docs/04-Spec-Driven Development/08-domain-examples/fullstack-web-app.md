# Example: Full-Stack Web App Spec

> A complete SDD example for a Next.js task management application.

---

## Constitution (excerpt)

```markdown
# Task Manager Pro — Constitution

## Tech Stack
- Frontend: Next.js 14 (app router), TypeScript 5.6, Tailwind CSS 4
- Backend: Next.js API routes with Prisma ORM
- Database: PostgreSQL 16
- Auth: NextAuth v5 with email + Google OAuth
- Testing: Vitest (unit), Playwright (E2E)

## Standards
- TypeScript strict mode, no `any`
- All API endpoints return consistent JSON format
- Test coverage >80% for new features
- WCAG 2.1 AA accessibility
```

---

## Feature Spec: Task Board

```markdown
# Feature: Task Board View

## Summary
Users need a Kanban-style board to visualize and manage their tasks across
different statuses. The board enables drag-and-drop task movement between
columns, providing an intuitive workflow management experience.

## User Stories

### US-1: View Task Board
As a logged-in user, I want to see my tasks organized in columns by status
so that I can visualize my workflow at a glance.

### US-2: Drag Tasks Between Columns
As a user, I want to drag tasks from one column to another so that I can
quickly update task status.

### US-3: Create Task from Board
As a user, I want to add new tasks directly from the board so that I don't
have to navigate away from my workflow view.

### US-4: Filter Board
As a user, I want to filter tasks by assignee, label, and priority so that
I can focus on relevant work.

## Acceptance Criteria

### US-1: View Task Board
- AC-1.1: Given I am logged in
  When I navigate to /board
  Then I see columns for: To Do, In Progress, In Review, Done
- AC-1.2: Given I have tasks in multiple statuses
  When the board loads
  Then tasks appear in their correct columns, sorted by priority (high→low)
- AC-1.3: Given a column has more than 10 tasks
  When viewing the board
  Then only 10 tasks are shown with a "Load more" button

### US-2: Drag Tasks Between Columns
- AC-2.1: Given I drag a task from "To Do" to "In Progress"
  When I drop it
  Then the task status updates to "in_progress" in the database
  And the task appears in the "In Progress" column
- AC-2.2: Given I drag a task to the same column
  When I drop it
  Then the task reorders within the column (no status change)
- AC-2.3: Given I drag and the network request fails
  When the error occurs
  Then the task returns to its original position
  And an error toast shows "Failed to update task. Please try again."

### US-3: Create Task from Board
- AC-3.1: Given I click "+" on any column
  When I enter a title and press Enter
  Then a new task is created with that column's status
  And it appears at the top of the column
- AC-3.2: Given I create a task with empty title
  When I press Enter
  Then the creation is cancelled (no empty tasks)

### US-4: Filter Board
- AC-4.1: Given I select a filter
  When the filter is applied
  Then only matching tasks show across all columns
  And the filter persists in the URL query string

## Edge Cases
| Scenario | Expected Behavior |
|----------|------------------|
| User has 0 tasks | Show empty state: "No tasks yet. Create your first task!" |
| Column has 100+ tasks | Paginate at 10, "Load more" button |
| Two users drag same task simultaneously | Last write wins; loser sees error toast |
| User loses network during drag | Task returns to original position |
| Task is deleted by another user while on board | Remove from board on next poll/refresh |

## Non-Functional Requirements
- Board renders in <500ms with 50 tasks
- Drag-and-drop works on touch devices (mobile/tablet)
- Keyboard accessible: Tab to focus tasks, Enter to open, Arrow keys to move
- Board state synchronized every 30 seconds

## Out of Scope (v1)
- Swimlanes (group rows by project)
- Custom columns (fixed: To Do, In Progress, In Review, Done)
- Subtasks displayed on board
- Board sharing with non-team members
- WIP limits on columns
```

---

## Plan (excerpt)

```markdown
# Plan: Task Board View

## Architecture
- React component tree: BoardPage → BoardColumn × 4 → TaskCard × N
- Drag-and-drop: @dnd-kit library (accessible, touch-friendly)
- State management: React Query for server state, local state for optimistic updates

## API Endpoints
- GET /api/tasks?status=<status>&page=<n> — Paginated task list by status
- PATCH /api/tasks/:id — Update task status and position
- POST /api/tasks — Create task with initial status

## Data Model Changes
- Add `position` column to tasks table (integer, for ordering within column)
- Add composite index on (user_id, status, position)

## Testing Strategy
- Unit: TaskCard, BoardColumn components
- Integration: API endpoints with database
- E2E: Full drag-and-drop flow with Playwright
```

---

## Tasks (excerpt)

```markdown
# Tasks: Task Board View

## Task 1: Add position column migration
Create Prisma migration adding `position` (Int) to Task table.
Files: prisma/migrations/xxx_add_position.sql
Acceptance: Migration runs, column exists, index created

## Task 2: Update Task API for position
Add PATCH /api/tasks/:id to support status + position updates.
Files: src/app/api/tasks/[id]/route.ts
Acceptance: API updates status and position atomically

## Task 3: Create TaskCard component
Build card component displaying title, priority badge, assignee avatar.
Files: src/components/board/TaskCard.tsx
Acceptance: Renders all fields, accessible with keyboard focus

## Task 4: Create BoardColumn component
Build column with header, task list, "Load more" button.
Files: src/components/board/BoardColumn.tsx
Acceptance: Shows tasks sorted by position, paginates at 10

## Task 5: Implement drag-and-drop
Wire @dnd-kit for cross-column drag with optimistic updates.
Files: src/components/board/Board.tsx
Acceptance: Drag updates status, failure reverts position

... (Tasks 6-10: filtering, empty states, E2E tests, etc.)
```

---

*Next: [API Service Spec →](api-service.md)*
