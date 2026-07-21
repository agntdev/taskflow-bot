# TaskMaster Bot — Bot specification

**Archetype:** workflow

**Voice:** warm and concise — write every user-facing message, button label, error, and empty state in this voice.

A personal task and reminder manager that accepts natural-language input, smart-schedules tasks with configurable notifications, and tracks goals with milestones—all within Telegram.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- individuals managing daily tasks
- people setting personal goals
- users who prefer natural-language input

## Success criteria

- user can add a task with /add tomorrow at 3pm
- user receives a notification 15 minutes before a task is due
- user can mark a task complete with an inline button
- user can view goals and update progress
- user can configure notification timing in /settings

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **Start** (command, actor: user, command: /start) — Open the main menu with quick-start guide
- **Add task** (command, actor: user, command: /add) — Add a new task with natural language parsing
  - inputs: task description
  - outputs: task confirmation with due date
- **Set reminder** (command, actor: user, command: /remind) — Set a reminder with natural language
  - inputs: reminder description
  - outputs: reminder confirmation
- **List tasks** (command, actor: user, command: /list) — Show pending tasks, filtered by category or due date
  - inputs: optional category filter
  - outputs: task list with inline buttons
- **Complete task** (command, actor: user, command: /complete) — Mark a task as done
  - inputs: task ID or selection
  - outputs: completion confirmation
- **Delete task** (command, actor: user, command: /delete) — Remove a task
  - inputs: task ID or selection
  - outputs: deletion confirmation
- **Goals** (command, actor: user, command: /goals) — View and manage goals
  - outputs: goals list with progress
- **Settings** (command, actor: user, command: /settings) — Configure preferences
  - outputs: settings menu
- **Mark complete** (button, actor: user, callback: task:complete) — Mark a task as done from the list view
  - inputs: task ID
  - outputs: completion confirmation
- **Edit task** (button, actor: user, callback: task:edit) — Edit a task's details
  - inputs: task ID
  - outputs: edit menu
- **Delete task** (button, actor: user, callback: task:delete) — Remove a task from the list view
  - inputs: task ID
  - outputs: deletion confirmation
- **Add goal** (button, actor: user, callback: goal:add) — Create a new goal
  - inputs: goal description
  - outputs: goal confirmation
- **Update progress** (button, actor: user, callback: goal:progress) — Update a goal's progress
  - inputs: goal ID, progress percentage
  - outputs: progress confirmation

## Flows

### Add task
_Trigger:_ /add

1. Parse natural language for task description and due date
2. Create task entity with parsed data
3. Save to database
4. Send confirmation message

_Data touched:_ tasks

### Set reminder
_Trigger:_ /remind

1. Parse natural language for reminder description and time
2. Create reminder entity
3. Schedule notification
4. Send confirmation message

_Data touched:_ reminders

### List tasks
_Trigger:_ /list

1. Query tasks filtered by category or due date
2. Sort by due date, priority, creation time
3. Send task list with inline buttons

_Data touched:_ tasks

### Complete task
_Trigger:_ Inline button or /complete

1. Mark task as completed
2. Update database
3. Send confirmation message

_Data touched:_ tasks

### Delete task
_Trigger:_ Inline button or /delete

1. Delete task from database
2. Send confirmation message

_Data touched:_ tasks

### View goals
_Trigger:_ /goals

1. Query user's goals
2. Calculate progress for each goal
3. Send goals list with progress indicators

_Data touched:_ goals

### Configure settings
_Trigger:_ /settings

1. Show settings menu
2. Update user preferences
3. Send confirmation

_Data touched:_ user_preferences

### Archive old tasks
_Trigger:_ Scheduled task

1. Query tasks older than 90 days
2. Mark as archived
3. Update database

_Data touched:_ tasks

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **tasks** _(retention: persistent)_ — User's tasks with due dates, priorities, and categories
  - fields: id, title, description, due_date, priority, status, category, created_at, archived
- **reminders** _(retention: persistent)_ — Scheduled notifications tied to tasks or standalone
  - fields: id, task_id, description, scheduled_time, sent, created_at
- **goals** _(retention: persistent)_ — Long-term objectives with milestones and progress tracking
  - fields: id, title, description, target_date, progress, status, created_at
- **categories** _(retention: persistent)_ — User-defined groups for task organization
  - fields: id, name, user_id, created_at
- **user_preferences** _(retention: persistent)_ — User-specific settings for notifications and defaults
  - fields: user_id, default_category, notification_time_before, email_notifications, created_at

## Integrations

- **Telegram** (required) — Bot API messaging
- **SQLite** (required) — Data persistence
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- View all tasks, reminders, goals, and categories
- Delete any task, reminder, or goal
- Archive old tasks
- Configure notification timing
- Set default category
- Enable/disable email notifications

## Notifications

- 15 minutes before a task is due
- When a reminder time is reached
- Goal milestone progress updates

## Permissions & privacy

- User data is isolated per Telegram user
- No data sharing with third parties
- Tasks older than 90 days are archived automatically
- User can delete any data at any time

## Edge cases

- Natural language parsing fails for date/time
- User tries to complete a non-existent task
- Reminder time has already passed
- User sets a task due date in the past
- Goal progress exceeds 100%
- User has no tasks when /list is called
- User tries to add a task with invalid category

## Required tests

- Add a task with natural language 'Buy groceries tomorrow at 5pm'
- Set a reminder 'Remind me to call mom in 2 hours'
- List tasks filtered by category 'Work'
- Mark a task complete via inline button
- Delete a task via inline button
- Create a goal with progress tracking
- Configure notification time in /settings
- Archive tasks older than 90 days

## Assumptions

- Users understand natural language date parsing
- Default category 'Personal' is acceptable for most users
- 15 minutes before due time is a reasonable notification timing
- Tasks older than 90 days can be archived automatically
- English interface with emoji-friendly UI is sufficient
