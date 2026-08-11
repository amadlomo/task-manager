# Task Manager Codebase Discoveries

This document tracks my understanding of the Task Manager project.

## Project Structure

- `README.md`: Provides basic setup and usage instructions.
- `cli.py`: The command-line interface entry point.
- `task_manager.py`: Main logic for task operations (`TaskManager` class).
- `models.py`: Defines data models (`Task`, `TaskPriority`, `TaskStatus`).
- `storage.py`: Manages data persistence to a JSON file (`TaskStorage` class, `TaskEncoder`/`TaskDecoder`).
- `task_parser.py`: Implements free-form text parsing to create tasks.
- `task_priority.py`: Calculates priority scores and sorts tasks.
- `task_list_merge.py`: Implements logic for merging two task lists with conflict resolution.
- `tests/`: Contains unit tests.

## Key Components

### 1. Data Models (`models.py`)
- `Task`: The central class representing a task. Includes `id` (uuid), `title`, `description`, `priority` (Enum), `status` (Enum), timestamps (`created_at`, `updated_at`, `due_date`, `completed_at`), and `tags`.
- `TaskPriority`: Enum for priority levels (LOW=1, MEDIUM=2, HIGH=3, URGENT=4).
- `TaskStatus`: Enum for status (TODO, IN_PROGRESS, REVIEW, DONE).

### 2. Persistence (`storage.py`)
- `TaskStorage`: Handles loading and saving tasks to a JSON file (`tasks.json`).
- `TaskEncoder` / `TaskDecoder`: Custom JSON (de)serialization logic for `Task` objects and datetimes.

### 3. Core Logic (`task_manager.py`)
- `TaskManager`: Interfaces with `TaskStorage` to provide high-level CRUD operations (create, list, update, delete, get details) and statistics generation.

### 4. CLI (`cli.py`)
- Uses `argparse` to handle user commands.
- Provides commands: `create`, `list`, `status` (update), `priority` (update), `due` (update), `tag`, `untag`, `show`, `delete`, `stats`.
- Formats tasks for output.

### 5. Utilities
- `task_parser.py`: Parses strings like `"Buy milk @shopping !2 #tomorrow"` to create `Task` objects.
- `task_priority.py`: Calculates an importance score based on priority, due date, status, tags, and update time.
- `task_list_merge.py`: Merges local/remote task lists, handles conflicts (newest update wins, status changes merge rules).

## Onboarding Exploration Exercise & Findings

To reinforce your understanding of the codebase and verify that the components interact as described, complete this hands-on exploration.

### Exercise Instructions

#### Step 1: Run the Unit Tests
Verify your environment is correctly set up and all core behavior functions correctly by executing:
```bash
python -m unittest discover tests
```

#### Step 2: Create a Task via the CLI
Create a task with Medium priority, a due date, and some tags:
```bash
python cli.py create "Learn Python MVC Design" --description "Review Models and CLI implementation" --priority 2 --due "2026-12-31" --tags "onboarding,learning"
```
I created a task usingcommand line python cli.py create "Learn Python MVC Design" --description "Review Models and CLI implementation" --priority 2 --due "2026-12-31" --tags "onboarding,learning" and the results were saved in a tasks.json file. The results are:
  ...
  6b5313bd - !! Learn Python MVC Design
  Review Models and CLI implementation
  Due: 2026-12-31 | Tags: onboarding, learning
  Created: 2026-08-09 15:47
when I run python cli.py list.

#### Step 3: Inspect the Saved File (`tasks.json`)
A `tasks.json` file should now have been created in the project root folder.
Open it and analyze how:
1. The custom `TaskEncoder` transformed class attributes.
2. The UUID, Enums, and ISO datetimes are represented as JSON primitives.

#### Step 4: Show Task Details and Generate Statistics
Verify that the task status and details can be read from the CLI:
```bash
python cli.py show <your_task_id>
python cli.py stats
```

#### Step 5: Complete the Task
Update the status of the task to `done` and verify the timestamp changes:
```bash
python cli.py status <your_task_id> done
```
Re-inspect `tasks.json` and check how `completed_at` and `updated_at` timestamps were automatically updated to the current system time.

---

### Verification Findings


### My Understanding of the Exercise

From this exercise, I understand that the main module of the program is called **`cli.py`**. I noticed that the program uses different positional arguments, which are basically commands that tell the program what action to perform.

The commands I identified are:

* **`create`** – used to create a new task.
* **`list`** – used to display or list all the tasks.
* **`status`** – used to update the status of a task.
* **`priority`** – used to update the priority of a task. I also noticed that there are **four different task priority levels**.
* **`due`** – used to update or set the due date of a task.
* **`tag`** – used to add a tag to a task.
* **`untag`** – used to remove a tag from a task.
* **`show`** – used to display the details of a task.
* **`delete`** – used to delete a task.
* **`stats`** – used to display statistics about the tasks.

I also understand that Python files normally use the **`.py`** file extension, which is why the main program file is called **`cli.py`**.

Overall, I understand that `cli.py` acts as the main entry point for the application, and the different commands allow the user to manage tasks from the command line.

### AI understanding
1. **Test Execution Findings:**
   - Command: `python -m unittest discover tests`
   - Results: All 55 tests passed successfully, confirming the business logic for parsing, prioritization, list merging, storage, and status updates is robust.

2. **File Generation & Persistence (`tasks.json` Schema):**
   - When a task is created, `TaskStorage` invokes `TaskEncoder`, which serializes the task into a standard JSON file (`tasks.json`).
   - Datetime attributes (`created_at`, `updated_at`, `due_date`) are saved in ISO-8601 format strings (e.g., `YYYY-MM-DDTHH:MM:SS.mmmmmm`).
   - Enums are saved as their underlying raw values (`TaskPriority` values map to integers `1-4`, and `TaskStatus` values map to strings like `"todo"`, `"in_progress"`).
   - Task IDs are 36-character standard UUID strings generated via Python's `uuid.uuid4()`.

3. **Lifecycle Tracking Behavior:**
   - Transitioning a task status to `done` invokes the model's `mark_as_done()` method.
   - This method sets `completed_at` and `updated_at` to the precise execution time, and saves the task back to `tasks.json`.

## System Architecture & Design Patterns Summary

### Execution Flow
- **Task Creation:** CLI (`cli.py`) processes user inputs $\rightarrow$ delegates to `TaskManager.create_task()` $\rightarrow$ instantiates a `Task` with a unique UUID and timestamps $\rightarrow$ saves to disk via `TaskStorage` using custom JSON encoding.
- **Task Updates:** CLI routes requests to `TaskManager` $\rightarrow$ delegates status/priority/due date modifications to `TaskStorage.update_task()` (invoking model-level timestamp and status state transitions) $\rightarrow$ persists the changes back to `tasks.json`.

### Persistence & Storage Mechanisms
- **JSON Flat-File Database:** Persists tasks in `tasks.json`.
- **Custom Serialization/Deserialization (`TaskEncoder`/`TaskDecoder`):** Custom serialization logic converts complex domain objects (including nested Python datetime objects and Enum attributes like `TaskPriority` and `TaskStatus`) to and from compatible JSON primitives.

### Discovered Design Patterns
1. **Rich Domain Model (Domain-Driven Design):** The `Task` class encapsulates its state transitions (such as `mark_as_done()`), validating invariants and auto-managing creation, update, and completion timestamps.
2. **Repository Pattern (`TaskStorage`):** Decouples the business logic from persistence concerns. The storage layer exposes clean interfaces (`add_task`, `get_task`, `update_task`) to query and modify state independently of JSON persistence details.
3. **Decoupled Strategy Pattern (`task_priority.py`):** Calculates multi-criteria task importance scores dynamically, completely separated from core models or database storage layers.
4. **Conflict Reconciliation Engine (`task_list_merge.py`):** Merges disconnected local and remote task databases with custom deterministic rules (e.g., LWW, sticky task completion, set unions for tags).
