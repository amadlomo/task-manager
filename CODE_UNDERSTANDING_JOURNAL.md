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

## Architecture & General Data Flow Analysis

This section maps out the general path that data takes from initial user command line input to output, how state is structured, how data is transformed at each layer, and guidance on debugging or extending this flow.

### 1. Global Data Flow Diagram (Input to Output)

```text
               +--------------------------------------+
               |            Terminal Input            |
               |  e.g., python cli.py create <args>   |
               +------------------+-------------------+
                                  |
                                  v
               +--------------------------------------+
               |          CLI Layer (cli.py)          |
               | - argparse validates basic arguments  |
               | - Raw strings parsed & split         |
               +------------------+-------------------+
                                  | (Primitive Types)
                                  v
               +--------------------------------------+
               |    TaskManager (task_manager.py)     |
               | - Instantiates TaskPriority & Status |
               | - Parses ISO-format due-date strings |
               | - Builds domain entities             |
               +------------------+-------------------+
                                  | (Domain Object)
                                  v
               +--------------------------------------+
               |      TaskStorage (storage.py)        |
               | - Holds active in-memory state map   |
               | - Marshals data using Custom Encoder |
               | - Saves stream to disk in JSON       |
               +------------------+-------------------+
                                  | (Serialized Strings)
                                  v
                          [ tasks.json ]
```

### 2. State Management & Lifecycle
* **Short-lived In-memory State**: Managed in-memory as a dictionary mapping UUID strings to `Task` entity instances inside `TaskStorage.tasks` (within `storage.py`). Because this is a CLI program, this in-memory state dictionary lives only for the brief duration of a single command-line execution.
* **Persistent State**: Persisted permanently inside a JSON flat-file (`tasks.json`).
* **Write-Through Persistence**: Any action that mutates task state (adding tasks, modifying tags, updating priority, or marking as completed) triggers an immediate, synchronous write-through operation to `tasks.json` by invoking `self.storage.save()`.

### 3. Core Data Transformations

| Layer / Step | Source Representation | Destination Representation | Method / Component |
| :--- | :--- | :--- | :--- |
| **CLI Input Cleaning** | Comma-separated tag string (`"work,todo"`) | List of clean strings (`['work', 'todo']`) | `cli.py` splitting list |
| **Enum Coercion** | Raw integer priority (`3`) | Type-safe Enum (`TaskPriority.HIGH`) | `task_manager.py` |
| **Date Parsing** | ISO format string (`"2026-08-12"`) | Native Python `datetime` object | `task_manager.py` |
| **Serialization** | `Task` Domain Object | Clean JSON string map with primitives | `storage.py` (`TaskEncoder`) |
| **Deserialization** | Plain JSON string map with primitives | Reconstituted `Task` domain instance | `storage.py` (`TaskDecoder`) |

### 4. Debugging & Troubleshooting Guide
* **Catch Deserialization Errors**: If the application crashes on startup or task lists appear empty, inspect `TaskStorage.load()`. Temporarily replace the catch-all `except Exception:` block with `except Exception as e: import traceback; traceback.print_exc()` to expose invalid JSON formats or key errors.
* **Inspect the Database File**: Directly view `tasks.json` to verify that timestamps are in standard ISO-8601 format (e.g. `"2026-08-12T10:00:00.000000"`) and priorities are saved as raw integers (`1` through `4`), ensuring serialization is working as intended.
* **Check Exception Propagation**: Remember that `TaskStorage.save()` swallows filesystem and permission errors. For deep debugging of save issues, temporarily comment out the `try-except` block in `save()` to let errors bubble up and crash the process, revealing the underlying OS exception.

### 5. Modification & Extensibility Checklist
When planning to alter the application's data flow, ensure you address each of the following points:
1. **Domain Model updates (`models.py`)**: Update `Task.__init__` with the new field and set a sane default value for backwards compatibility.
2. **Custom JSON Serialization/Deserialization (`storage.py`)**:
   * Add the new field to `TaskEncoder.default()` if it utilizes a custom/non-primitive type.
   * Add the field extraction logic into `TaskDecoder.object_hook()`, defining fallbacks if the key is missing from legacy `tasks.json` entries.
3. **Orchestration Layer (`task_manager.py`)**: Add the argument to high-level methods (like `create_task` or a new `update_task_field` method) and enforce necessary type conversions.
4. **CLI Layer (`cli.py`)**: Add corresponding command-line arguments to the argparse subparsers, clean the inputs, and pass them down the chain.

## Task Completion Data Flow and State Analysis

To understand how the application handles marking a task as complete, we can map out the transition from user command to persistent storage.

### 1. Data Flow Diagram: Task Completion

```text
               +--------------------------------------+
               |            Terminal Input            |
               |   python cli.py status <ID> done     |
               +------------------+-------------------+
                                  |
                                  v
               +--------------------------------------+
               |          CLI Layer (cli.py)          |
               | - Receives command "status"          |
               | - Extracts task_id and status "done" |
               +------------------+-------------------+
                                  |
                                  v (Method Call)
               +--------------------------------------+
               |    TaskManager (task_manager.py)     |
               | - Instantiates TaskStatus("done")    |
               | - Fetches task from storage          |
               | - Invokes domain transition method   |
               +------------------+-------------------+
                                  |
                                  v (Mutates Object)
               +--------------------------------------+
               |       Domain Model (models.py)       |
               | - Mutates task.status to DONE        |
               | - Sets task.completed_at & updated_at|
               +------------------+-------------------+
                                  |
                                  v (Persists State)
               +--------------------------------------+
               |      TaskStorage (storage.py)        |
               | - Invokes self.storage.save()        |
               | - TaskEncoder converts to primitive  |
               | - Writes entire list to tasks.json   |
               +------------------+-------------------+
                                  |
                                  v
                          [ tasks.json ]
```

### 2. State Changes During Task Completion

When a task's status is updated to `done`, the following state changes occur within the `Task` domain instance (defined in `models.py` inside `mark_as_done()`):

* **`task.status`**: Transitioned from `TaskStatus.TODO`, `TaskStatus.IN_PROGRESS`, or `TaskStatus.REVIEW` to `TaskStatus.DONE`.
* **`task.completed_at`**: Transitioned from `None` to the current system timestamp (`datetime.now()`).
* **`task.updated_at`**: Updated to the exact same system timestamp as `completed_at` (`datetime.now()`).

### 3. Entry Points & Components Involved

* **Command Entry Point (`cli.py`)**:
  * Parser: `update_status_parser = subparsers.add_parser("status")`
  * Action Router:
    ```python
    elif args.command == "status":
        if task_manager.update_task_status(args.task_id, args.status):
            print(f"Updated task status to {args.status}")
    ```
* **Orchestrator Control Flow (`task_manager.py`)**:
  * Action Handler:
    ```python
    def update_task_status(self, task_id, new_status_value):
        new_status = TaskStatus(new_status_value)
        if new_status == TaskStatus.DONE:
            task = self.storage.get_task(task_id)
            if task:
                task.mark_as_done()
                self.storage.save()
                return True
        else:
            return self.storage.update_task(task_id, status=new_status)
    ```
* **Domain Entity (`models.py`)**:
  * State Transition Logic:
    ```python
    def mark_as_done(self):
        self.status = TaskStatus.DONE
        self.completed_at = datetime.now()
        self.updated_at = self.completed_at
    ```
* **Persistence Handler (`storage.py`)**:
  * Repository: `TaskStorage` triggers `self.save()` which serialized the mutated domain model to JSON using `TaskEncoder`.

### 4. Potential Points of Failure

1. **Unknown/Invalid Task ID**:
   * *Symptom*: If the user provides a non-existent `task_id`, `self.storage.get_task(task_id)` returns `None`.
   * *Resolution*: `update_task_status` returns `False`, and `cli.py` prints `"Failed to update task status. Task not found."`
2. **Silent Failure on File Write / disk full**:
   * *Symptom*: If `self.storage.save()` raises an OS exception (e.g. disk is full, file permissions denied), the exception is caught and printed: `Error saving tasks: ...` inside `storage.py`.
   * *Vulnerability*: However, the exception does not propagate, and `save()` does not return a status. `update_task_status()` still returns `True`, meaning `cli.py` prints `"Updated task status to done"`, deceiving the user into believing the change was successfully persisted when it actually failed.
3. **Invalid Status Transition Values**:
   * *Symptom*: If a status value not allowed by argparse is bypassed/passed programmatically, the line `TaskStatus(new_status_value)` raises a `ValueError` which is unhandled and will crash the execution thread.

### 5. How the Application Persists These Changes

* **Immediate Synchronous Save**: The system follows a write-through pattern. As soon as `task.mark_as_done()` is executed, `self.storage.save()` is called.
* **JSON Serialization**: `TaskEncoder` processes the dictionary:
  * Encodes the Enum `TaskStatus.DONE` to its raw string value `"done"`.
  * Encodes Python `datetime` instances (`completed_at`, `updated_at`) to ISO-8601 string representations (e.g., `"2026-08-12T15:30:45.123456"`).
* **Write to Flat-File**: The entire memory map of tasks is marshalled and completely overwrites `tasks.json` with indentation for human-readability.

---

## Junior Developer Reflection & Conceptual Journal

This section documents the onboarding learning process, mapping out how my understanding developed and how I conceptualize this system as a beginner.

### 1. How the Explanations Reshaped My Understanding
Initially, the codebase felt like a disparate collection of scripts. Seeing the detailed data mapping made me realize that **data flow is like a relay race**. Instead of one giant block of code trying to manage files, arguments, and validation, each class behaves as a specialized runner:
* The **CLI (`cli.py`)** acts as the front of house. It gathers, cleans, and structures user inputs.
* The **Orchestrator (`task_manager.py`)** converts raw values to domain Enums and dates.
* The **Domain Entity (`models.py`)** applies the actual business mutations (updating statuses, tracking timestamps).
* The **Repository (`storage.py`)** handles persistence, formatting the domain's state into file-safe primitives.

### 2. What Aspects Were Most Challenging
The concept of **JSON Hydration (Serialization & Deserialization)** was the most complex part to grasp. 
* **The Challenge**: I didn't initially realize that JSON is a highly restrictive format that only understands primitive types like strings, numbers, arrays, and booleans.
* **The Solution**: Realizing that Python's nested `datetime` structures and Custom Enum classes must be carefully dismantled (serialized via `TaskEncoder`) and reconstructed (hydrated via `TaskDecoder.object_hook`) on program startup helped me make sense of why these utility classes exist.

### 3. Explaining This System to Another Junior Developer (The Cafe Analogy)
If I were explaining this system to another junior developer, I would use a restaurant/cafe analogy:
* **`cli.py` is the Waiter**: They stand at the front, take a customer's order (e.g., `"status 1 done"`), ensure the order makes sense, and write it down on a ticket.
* **`task_manager.py` is the Kitchen Manager**: They read the ticket, translate the order into kitchen-specific instructions (such as translating `"done"` into a physical status change), and hand the task to the chef.
* **`models.py` is the Chef**: They actually assemble or mutate the dish (the `Task`), stamp it with timestamps, and keep track of its internal quality metrics.
* **`storage.py` is the Freezer**: Fresh food spoils when the cafe closes (when the script finishes execution). Therefore, we convert the food to a packaged format (JSON serialization) to freeze it. When the cafe opens tomorrow, we thaw the food back into a hot dish (JSON deserialization/hydration) so it can be served again.

### 4. Testing My Understanding
I validated my mental model by tracing failure cases through the codebase:
* **Invalid task ID**: Traced that searching for a non-existent task returns `None` at the repository layer. The Manager catches this, cancels the update, returns `False`, and prevents disk writes.
* **Argparse validation**: Verified that the CLI parser restricts task status and priority inputs directly at the terminal level, avoiding unhandled errors inside deeper domain logic.
* **Automated verification**: Ran the unit test suites via `python -m unittest discover tests` and verified that all 55 tests pass, confirming the system aligns perfectly with the mapped rules.

### 5. Identified Areas for Improvement
Based on my new understanding, I identified two valuable improvements we could add to make this application production-grade:
1. **Propagate Save Statuses (Avoid Silent Lies)**: Currently, if saving fails due to an OS-level issue (e.g., permission errors or full disk), the exception is swallowed inside `TaskStorage.save()`. The orchestrator assumes the update was successful, returning `True` so the CLI prints `"Updated task status to done"`. We should change `save()` to return a boolean status (or bubble the exception up) so that users are warned if their changes failed to persist.
2. **Implement Safe/Atomic Writes**: Since `save()` completely overwrites the target file in one pass, any system interrupt or crash during the write will result in total data loss or file corruption. Writing to a temporary file (`tasks.json.tmp`) and atomically swapping/renaming it on successful write would eliminate this vulnerability.


