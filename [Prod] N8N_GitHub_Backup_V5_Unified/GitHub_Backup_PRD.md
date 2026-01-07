# 🚀 n8n Workflow Backup to GitHub (V5): Product Requirements Document
## Project: Ultimate Secure One-Way Archival for n8n (Unified V5.0)
**Status:** 🏗️ In Development  |  **Owner:** AmanSuryavanshi  |  **Version:** 5.0 (Unified "Loop-to-Webhook")

---

### 1. Executive Summary
**The Vision:** A single, self-contained workflow file that provides industry-standard stability (Rate Limits, Isolated Execution) without the complexity of managing multiple files.
**Key Evolution (V5.0):** **"Loop-to-Webhook" Pattern.** The workflow triggers itself via Webhook for each item, acting as both Orchestrator and Worker in one canvas.

### 2. Context Verification (User Requirements)
*   **Tags & Sub-tags:** ✅ `Project:` and `Sub:` tags mapped to deeply nested folders.
*   **Security:** ✅ `SENSITIVE_REGEX` redacts all credentials before push.
*   **Folder Structuring:** ✅ Automatic folder creation based on tags.
*   **Auto Folder Change (Local):** ⚠️ **Smart Search Strategy:** If a file is renamed/moved locally (tags change), the system searches GitHub by `ID`.
    *   *Match Found?* It updates the file *in place* (at the old location). **Why?** To prevent duplicate files (Ghost files) and preserve Git history.
    *   *No Match?* It creates a new file at the new Tag-based location.
*   **2-Way Sync:** ⛔ **Manual Only.** This is a **One-Way Backup System (n8n → GitHub)**. Automated restore is disabled to prevent accidental overwrite of live workflows. Restore is done via "File > Import" or Git.

### 3. Architecture: The "Loop-to-Webhook" Pattern

#### 3.1 The Concept
Instead of two files, we use two **disconnected streams** in one canvas:
1.  **Stream A (Orchestrator):** `Schedule` -> `Fetch` -> `Split` -> `Wait` -> `HTTP Request (Call Stream B)`.
2.  **Stream B (Worker):** `Webhook Trigger` -> `Logic` -> `Push` -> `Error Handler`.

#### 3.2 Benefits
*   **Single File:** Easy to share/import.
*   **Isolated Execution:** If Item #5 fails, Item #6 still runs.
*   **Rate Limited:** The "Wait" node in Stream A protects GitHub API.
*   **Clean Layout:** "Top Stream" = Management, "Bottom Stream" = Logic.

### 4. Technical Specs
*   **Rate Barrier:** `Wait` 2 Seconds (Max 30 requests/minute).
*   **Webhook Method:** `POST` to local webhook URL with `workflowJson` body.
*   **Error Handling:** `Error Trigger` node in Stream B sends Alert (Slack/Email).

### 5. Implementation Node Map

#### Stream A: Manager
| Node | Purpose |
| :--- | :--- |
| `Schedule` | Daily Trigger |
| `Fetch All` | Get Workflows |
| `Filter` | Remove Archived |
| `Split` | Batch (Size=1) |
| `Wait` | Rate Limit (2s) |
| `Call Worker` | HTTP Request (POST Webhook) |

#### Stream B: Worker
| Node | Purpose |
| :--- | :--- |
| `Webhook` | Receive Job |
| `Scrub` | Config & Redact |
| `Search` | Find by ID |
| `Diff` | Compare Content |
| `Push` | Upsert to GitHub |
| `Error` | Catch Failures |

---
