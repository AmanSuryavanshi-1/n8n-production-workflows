# n8n Workflow Backup to GitHub (V5): Implementation & Setup Guide


<img src="https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/v5_dual_stream_architecture.webp" width="700" alt="Dual Stream Architecture" />

## Step-by-Step for Everyone (Technical & Non-Technical)

> [!NOTE]
> This workflow creates **one-way backups from n8n to GitHub**. Restoring from GitHub is a manual process (File > Import).


---

## **Before You Start: What You'll Need**

### **Prerequisites Checklist**

- [ ] **n8n Instance** (v1.0 or higher) running and accessible
- [ ] **GitHub Account** with a repository where you want to backup workflows
- [ ] **GitHub Personal Access Token** (fine-grained, `Contents: Read/Write`)
  - Create one: Settings → Developer Settings → Personal Access Tokens → Fine-grained tokens
  - Permissions needed: `Contents` (read/write), `Metadata` (read)
- [ ] **n8n API Key** (for accessing workflow data)
  - Found in: n8n Settings → API Key section
- [ ] **10 minutes** of uninterrupted time
- [ ] **One test workflow** to verify the backup works

---

## **Part 1: GitHub Setup (2 Minutes)**

### **Step 1A: Create or Use Existing Repository**

You need a GitHub repository where backups will be stored.

**Option A: Use Existing Repository**
- Navigate to your repo on GitHub
- Make sure you have write permissions
- Copy the repo URL

**Option B: Create New Repository**
1. Go to GitHub.com
2. Click "+" → "New repository"
3. Name: `n8n_Workflows` (or your preference)
4. Description: "Automated n8n workflow backups"
5. Visibility: Private (if you have credentials) or Public (if redacted)
6. Click "Create repository"

### **Step 1B: Create Personal Access Token**

1. Go to GitHub → Settings (top right)
2. Select "Developer settings" (left sidebar)
3. Click "Personal access tokens" → "Fine-grained tokens"
4. Click "Generate new token"
5. **Token Details:**
   - Name: `n8n-backup-token`
   - Expiration: 30 days (or your preference)
   - Description: "For n8n GitHub backup automation"

6. **Permissions:**
   - Repository access: Select your backup repo
   - Contents: ✅ Read & Write
   - Metadata: ✅ Read

7. Click "Generate token"
8. **SAVE THIS TOKEN** (you'll need it in 3 minutes)

---

## **Part 2: n8n Setup (5 Minutes)**

### **Step 2A: Import the Workflow**

1. Open your n8n instance
2. Go to Workflows → "+" (New Workflow)
3. Click "Import from URL" or "Import from File"
4. Paste the workflow URL or upload the `.json` file
5. Click "Import"

You should now see the "Antigravity - GitHub Backup V5" workflow in your dashboard.

<img src="https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/v5_canvas_overview.webp" width="700" alt="V5 Canvas Overview" />


### **Step 2B: Add GitHub Credentials**

1. Open the imported workflow
2. Click on any **HTTP Request** node (there are several)
3. In the right panel, look for "Credentials"
4. Click "Create New Credential"
5. Select "GitHub"
6. Fill in:
   - **Authentication**: Choose "Personal Access Token"
   - **Token**: Paste the GitHub token you created in Step 1B
7. Click "Save"
8. **Repeat this for all HTTP Request nodes** (you'll see 3-4 nodes that need GitHub credentials)

*Easy way to find them:*
- Look for nodes with red "!" icon (missing credentials)
- Fix them one by one

### **Step 2C: Configure the Workflow Constants**

1. Open the "1. Config & Scrub" node (in the bottom section)
2. Find this JavaScript code:
   ```javascript
   const REPO_OWNER = 'YourUsername';
   const REPO_NAME = 'YourRepo';
   ```

3. Replace with your actual values:
   ```javascript
   const REPO_OWNER = 'your-github-username';     // Or org name
   const REPO_NAME = 'n8n_Workflows';              // Your repo name
   ```

4. Click "Save"

**Example:**
```javascript
// If your GitHub repo is: github.com/john-doe/my-automations
const REPO_OWNER = 'john-doe';
const REPO_NAME = 'my-automations';
```

### **Step 2D: Test Manual Trigger**


*Verified Execution Success*

1. At the top of the workflow, click "Execute Workflow"
2. Watch the execution unfold
3. **Expected Result:** No errors, workflow completes in ~10 seconds
4. **Check GitHub:** Navigate to your repo. You should see a new folder created!

---

## **Part 3: Tagging Your Workflows (5 Minutes)**

This is crucial for organization. Without proper tags, all workflows go to `_Unsorted/` folder.

### **Step 3A: Understanding Tags**

Tags control WHERE your files are saved:

```
Tag: Project: BIP
  ↓
GitHub Folder: BIP/

Tag: Project: Internal + Tag: Sub: GitHub/Backups
  ↓
GitHub Folder: Internal/GitHub/Backups/
```

### **Step 3B: Apply Tags to Your Workflows**

<img src="https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/split_tag_organization_flow.webp" width="700" alt="Split Tag Logic" />


1. Go to n8n Dashboard → Workflows
2. Click on any workflow you want to backup
3. Look for "Tags" section (usually on the left)
4. Click "Add Tag"

**For your Twitter automation workflow, use:**
- Primary Tag: `Project: BIP` (makes folder "BIP/")
- Secondary Tag: `Status: Prod` (marks as production)

*Result: BIP/TwitterBot/workflow.json*

**For your internal automation (e.g., database backup):**
- Primary Tag: `Project: Internal`
- Secondary Tag: `Sub: Ops/Critical`
- Tertiary Tag: `Status: Dev`

*Result: Internal/Ops/Critical/DatabaseBackup/workflow.json*

### **Step 3C: Tag Naming Rules**

Follow these rules for professional organization:

| Rule | Example | Result |
|------|---------|--------|
| **Project Tag** | `Project: BIP` | BIP/ folder |
| **Sub Tag** | `Sub: Twitter/Social` | BIP/Twitter/Social/ |
| **Status Tag** | `Status: Prod` | (no folder, filtering only) |
| **Variants** | Name: `Twitter - V1 (OpenAI)` | BIP/Twitter - V1 (OpenAI)/ |

---

## **Part 4: Activate & Schedule (2 Minutes)**

### **Step 4A: Activate the Workflow**

1. Open the GitHub Backup V5 workflow
2. Top right corner, toggle "Active" switch to **ON**
3. You should see a green checkmark

**Why Important:** The workflow needs to be active to call itself via webhook.

### **Step 4B: Set Up Schedule (Optional)**

By default, the workflow runs **daily at midnight**. To change:

1. Click on "Schedule Trigger" node (top-left)
2. Edit the schedule:
   - **Every day at 3 AM?** Set custom time
   - **Every hour?** Change interval
   - **Manually only?** Keep it as is (you can always click "Execute")

---

## **Part 5: Verify & Monitor (Ongoing)**

### **Step 5A: First Backup Verification**

1. Wait for the next scheduled run OR click "Execute Workflow"
2. Go to GitHub repo
3. You should see folders named after your "Project" tags:
   ```
   n8n_Workflows/
   ├── BIP/
   │   └── TwitterBot/
   │       └── workflow.json
   ├── Internal/
   │   └── Ops/
   │       └── Critical/
   │           └── DatabaseBackup/
   │               └── workflow.json
   ├── _Unsorted/
   │   └── UntaggedWorkflow/
   │       └── workflow.json
   ```

   > <img src="https://cdn.jsdelivr.net/gh/AmanSuryavanshi-1/portfolio-assets@main/N8N-GithubBackup/v5_real_repo_structure.webp" width="500" alt="GitHub Repo Structure" />
   > *Verified GitHub Folder Hierarchy*


### **Step 5B: Monitor Backups**

**Check Execution Logs**

1. Open the workflow
2. Click "Executions" tab
3. Look for:
   - ✅ Green checkmarks (success)
   - ⚠️ Yellow warnings (completed with warnings)
   - ❌ Red errors (failed)

**What Each Status Means**

| Status | Meaning | Action |
|--------|---------|--------|
| ✅ Success | All workflows backed up | Nothing needed |
| ⚠️ Partial | Some failed, some passed | Check error logs |
| ❌ Failed | All workflows failed | Likely credential issue |

### **Step 5C: Handle Errors**

**Error: "Cannot find credential"**
- Fix: Go through Step 2B again (assign GitHub credential to red-marked nodes)

**Error: "Repository not found (404)"**
- Fix: Verify REPO_OWNER and REPO_NAME are correct in Step 2C

**Error: "Rate limit exceeded"**
- Normal: This means workflow ran before. Wait a bit.
- Fix: The "Wait 2s" node protects you, so this is rare

**Error: "Webhook not responding"**
- Fix: Make sure workflow is Active (Step 4A)

---

## **Part 6: Customization (10 Minutes)**

### **6A: Change Rate Limit (If You Want Faster Backups)**

**WARNING: Don't go too fast!**

1. Open "Wait (Rate Limit)" node
2. Change "2 seconds" to "1.5 seconds" or "1 second"
3. **DON'T go below 1 second** (risks hitting GitHub limits)

**Math**: 
- 2 seconds = 30 req/min (safe, GitHub limit)
- 1.5 seconds = 40 req/min (slightly risky)
- 1 second = 60 req/min (RISKY, will error)

### **6B: Change Backup Folder Structure**

Want all backups in a `Backups/YYYY-MM-DD/` folder?

1. Open "1. Config & Scrub" node
2. Find this line:
   ```javascript
   const targetPath = `${root}/${sub}${safeName}/workflow.json`;
   ```

3. Change to:
   ```javascript
   const today = new Date().toISOString().split('T')[0];
   const targetPath = `Backups/${today}/${root}/${sub}${safeName}/workflow.json`;
   ```

This creates date-stamped backups.

### **6C: Add Slack Notifications on Failure**

1. Open the "Error Trigger" node (bottom right)
2. Add a "Send Slack Message" node after it
3. Configure with your Slack webhook URL
4. Message template:
   ```
   ❌ Workflow Backup Failed
   Workflow: {{workflow_name}}
   Error: {{error_message}}
   Time: {{timestamp}}
   ```

---

## **Part 7: Maintenance (Monthly Check)**

### **Monthly Tasks (5 Minutes)**

- [ ] **Review GitHub commits** — Verify files are being updated
- [ ] **Check execution logs** — Any recurring errors?
- [ ] **Update tags** — Any new workflows added to n8n?
- [ ] **Refresh credentials** — GitHub tokens expire (if using personal access tokens)

### **Quarterly Tasks (15 Minutes)**

- [ ] **Audit redacted credentials** — Manual spot-check of backed-up workflows
- [ ] **Review folder structure** — Reorganize if too many `_Unsorted/` files
- [ ] **Update workflow name** — Rename n8n side to match GitHub backup

---

## **Troubleshooting Guide**

### **Symptom: Backups Not Appearing on GitHub**

**Diagnosis:**
1. Open workflow, click "Executions"
2. Did it run? If no → Workflow not active
3. Did it error? If yes → Missing credentials

**Fix:**
- Verify "Active" toggle is ON (Step 4A)
- Verify GitHub credential assigned to HTTP nodes (Step 2B)

---

### **Symptom: "File Already Exists" Error**

This is a GitHub conflict, not your fault. The system recovers automatically.

**What happened:** File SHA changed between your read and write. System retries with latest SHA.

**Result:** ✅ File gets updated on retry (you'll see success in next execution)

---

### **Symptom: File Moved on GitHub, Now There Are Duplicates**

This shouldn't happen with V5. The system searches by workflow ID.

**Possible cause:** Workflow had no ID when moved

**Fix:**
- Manually delete the duplicate file on GitHub
- Re-run backup. System will find the ID-based location.

---

### **Symptom: Credentials Visible in Backed-Up Files**

❌ **This should NEVER happen.** If it does:

1. Regenerate any leaked credentials immediately
2. Notify your security team
3. Check "1. Config & Scrub" node — SENSITIVE_REGEX might not be catching your credential format
4. Add custom regex pattern

Example: If you have a field called `myApiToken`:
```javascript
const SENSITIVE_REGEX = /password|token|secret|api_?key|auth|bearer|credential|private_?key|myApiToken/i;
```

---

## **Advanced: Restore from GitHub Backup**

Need to restore a workflow? Follow these steps:

1. Open GitHub repo
2. Find the workflow file (e.g., `BIP/TwitterBot/workflow.json`)
3. Click the file, then "Raw" to view raw JSON
4. Copy all content
5. Go to n8n → New Workflow → "+" → "Import from URL" or paste JSON
6. Edit credentials and activate

---

## **FAQ**

### **Q: Will the backup workflow interfere with my other automations?**
**A:** No. It uses a different API endpoint (webhook vs. REST).

---

### **Q: What happens if I delete a workflow in n8n?**
**A:** The backup file stays on GitHub. You can restore anytime.

---

### **Q: Can I make my GitHub repo public?**
**A:** Yes, if credentials are scrubbed! Check a sample file first. The system should redact all sensitive data.

---

### **Q: How much does this cost?**
**A:** Only GitHub API costs (free tier: 60 req/hour, plenty). n8n costs same whether backup runs or not.

---

### **Q: What if I have 1000 workflows?**
**A:** No problem. 
- 1000 × 2 seconds = 2000 seconds = ~33 minutes
- All within rate limits
- Runs daily at midnight, you don't notice

---

## **Quick Checklist: You're Done When...**

- [ ] GitHub repo created
- [ ] GitHub token generated and saved
- [ ] Workflow imported into n8n
- [ ] GitHub credentials assigned to HTTP nodes
- [ ] Constants updated (REPO_OWNER, REPO_NAME)
- [ ] Workflow is Active
- [ ] Manual test executed successfully
- [ ] Files appear on GitHub
- [ ] Tags applied to at least 3 workflows
- [ ] Scheduled backup time set
- [ ] First automatic backup confirmed

---

## **You're All Set!**

From this point on:
- ✅ Backups run automatically
- ✅ Files organized by tags
- ✅ Credentials redacted
- ✅ Rate limits respected
- ✅ Zero manual work needed

**Questions?** Check the Technical Documentation for deeper explanations.

**Ready to share your automations?** Your GitHub repo is now auditable and portfolio-ready!

---

**Last Updated**: January 2026  
**Version**: V5.0  
**Support**: Check GitHub Backup V5 Technical Documentation