# IR Navigator — PICERL

A living incident response reference tool built on the PICERL framework. Browse phases, actions, and use cases at the live URL. When an alert comes in, switch to **Incident View** to track your actions in real time. Update this repository to instantly update what everyone sees.

---

## Live URL

```
https://rootxover.github.io/IR-REACT/
```

Replace `YOUR-USERNAME` with your actual GitHub username after setup.

---

## How this works (the short version)

The website loads data from the JSON files in this repository. When you edit a file here and commit, the live page updates automatically within about 60 seconds via GitHub Pages. No build step, no server, no technical skills required beyond editing a file in GitHub.

---

## Repository structure

```
react-custom2/
│
├── index.html              ← The app itself. Do not edit this.
├── index.json              ← Master index: lists all phases and use cases.
├── categories.json         ← Color-coded category definitions.
│
├── phases/                 ← One folder per PICERL phase.
│   ├── 01-preparation/
│   │   ├── _phase.json     ← Phase metadata and ordered list of actions.
│   │   ├── PR-001-practice.json
│   │   ├── PR-002-take-trainings.json
│   │   └── ...
│   ├── 02-identification/
│   ├── 03-containment/
│   ├── 04-eradication/
│   ├── 05-recovery/
│   └── 06-lessons-learned/
│
└── usecases/               ← One file per incident use case.
    ├── phishing.json
    ├── layer3-ddos.json
    ├── layer7-ddos.json
    ├── password-spray.json
    ├── malware.json
    ├── insider-threat.json
    └── session-hijacking.json
```

---

## The golden rule

> **Edit the JSON files to update the site. Never edit `index.html`.**

Everything an analyst would want to change — action names, procedures, URLs, categories, use case steps — lives in the JSON files. The HTML is just the interface.

---

## App features

### Matrix view
The main view displays a column-based matrix. Each column is a PICERL phase. Each cell is an action. Click an action to expand its detail sections (description, procedure, tools, etc.) inline. Hover over an action to see **edit** and **del** buttons.

### Themes
Cycle through 5 themes using the toggle button in the header: Dark (default), Light, Tactical (red/amber), Midnight (blue), Corporate (as boring as possible), Goose (Grumpy Goose).

### Category filter bar
Filter the matrix by category (Network, Email, File, Process, Configuration, Identity, General). Click a category pill to show only actions in that category. Click again to clear the filter.

### Use cases panel
The right-side panel lists all use cases. Each use case has three modes:

- **View** — highlights its actions on the matrix. Clicking a highlighted action opens its reference URL.
- **Edit** — click any action on the matrix to add/remove it from the use case. Drag rows in the panel to reorder.
- **Guide** — opens a full-page response guide with all actions grouped by PICERL phase, each expandable to show description and detail sections.

### Incident View
A full-screen action tracker for working live incidents. Switch to it when an alert requires active handling. See the **Working a live incident** section below for a full walkthrough.

### Actions
Each action has:
- **ID** — unique identifier (e.g. `PR-001`, `CO-012`)
- **Name** — display name
- **Description** — short summary shown in italics
- **Category** — one of the defined categories (changes the cell color)
- **Reference URL** — clicking the action opens this URL in a new tab
- **Sections** — collapsible detail panels (e.g. Procedure, Tools, Escalation, References). Each section has a title, body text, and optional URL. Sections have a **copy to clipboard** button.

### Header toolbar
- **Click the title** to rename the matrix
- **+ Phase** — add a new phase column
- **+ Action** — add a new action to any phase
- **Clear All** — reset all categories and use case assignments
- **Import JSON** — load a previously exported snapshot
- **Export JSON** — download a JSON snapshot of the current matrix state
- **Manage Categories** — add or delete categories with custom colors

### Keyboard shortcuts
- **Escape** — close the response guide overlay

---

## Working a live incident

### When to switch to Incident View

Switch to Incident View as soon as you have a confirmed or suspected incident that needs tracking. It's designed to be shared on screen during a call — everything a team needs to see (who owns what, what stage things are in, how long they've been there) is visible at a glance.

Use the **matrix view** to look things up and understand what an action does. Use **Incident View** to actually run the response.

### How to open it

Click the **Incident View** button on the right side of the category filter bar (the bar just below the header). The page switches to a full-screen Kanban board with three lanes: **To Do**, **In Progress**, and **Completed**.

### Loading a playbook

If the alert matches a known scenario (phishing, malware, password spray, etc.), load the matching playbook:

1. In the top bar of Incident View, find the **Load Playbook** dropdown
2. Select the playbook that matches the incident type
3. All relevant actions are automatically placed in the **To Do** lane, grouped by phase (Identification → Containment → Eradication → Recovery)

The phases always appear in ICER order. Work top to bottom — don't skip Identification just to jump to Containment.

### Adding actions manually

If the incident doesn't match a playbook, or you need to add something not in the library:

- Click **+ Add Action** at the bottom of the To Do lane
- Use the **From Library** tab to search the full action catalogue. Filter by phase or by category to narrow things down quickly.
- Use the **Custom Action** tab to type in a free-text action if the exact step isn't in the library.

Actions added manually sit in the correct ICER phase group alongside playbook actions.

### Working the board

**Starting an action:**
1. Find the action card in the To Do lane
2. Assign an owner — type a name in the **Assign owner** field on the card
3. Click **Start →** to move it to In Progress

The card automatically captures the timestamp when it was assigned and when it moved to In Progress.

**Adding notes:**
Each card has a **NOTES** field. Use this for findings, observations, or anything relevant to that specific action — IOCs found, tools used, outcomes. Notes persist as the card moves between lanes and are included in the exported report.

**Completing an action:**
Click **Complete ✓** on the card. A completion timestamp is recorded automatically.

**Keeping the lanes clean:**
Each lane groups actions under their phase header (Identification, Containment, etc.). Click the phase header to collapse that group when it's done — it stays visible as a count so you can still see progress at a glance. Click again to expand.

### Tracking multiple incidents

The sidebar on the left lists all open incidents. Click **+** to create a new one. Click any incident to switch to it. Double-click an incident name to rename it (e.g. `INC-00124 Phishing Wave`). Each incident is independent — its own board, its own playbook, its own cards.

### Exporting a record

Click **Export JSON** in the top bar to download a full record of the incident. The export includes every action, owner, status, timestamps (assigned, started, completed), and any notes added to each card. Hand this to Tier 2, attach it to a ticket, or use it for the post-incident review.

### Going back to the matrix

Click **← Back to Matrix** in the top bar to return. Your incident board is preserved for the session.

---

## Common tasks

### Update the procedure for an existing action

1. Navigate into the correct phase folder (e.g. `phases/03-containment/`)
2. Click the action file you want to update (e.g. `CO-002-block-external-ip-address.json`)
3. Click the **pencil icon** in the top right of the file view
4. Edit the fields — particularly the `sections` array
5. Scroll down, write a short commit message
6. Click **Commit changes**

The live page updates within about 60 seconds.

**Example action file:**
```json
{
  "id": "CO-002",
  "name": "Block External IP Address",
  "description": "Block an external IP address from being accessed by corporate assets",
  "url": "https://wiki.example.com/block-ip",
  "category": "network",
  "sections": [
    {
      "id": "s1",
      "title": "Procedure",
      "body": "1. Open the firewall management console\n2. Navigate to block rules\n3. Add the IP to the deny list\n4. Verify the rule is active\n5. Document in the incident ticket"
    },
    {
      "id": "s2",
      "title": "Tools",
      "body": "- Palo Alto Panorama\n- Firewall CLI: set security policy deny-ip <address>\n- Ticket system: create ticket type NET-BLOCK"
    },
    {
      "id": "s3",
      "title": "Escalation",
      "body": "If the IP is associated with a CDN or shared hosting provider, escalate to Tier 2 before blocking."
    }
  ]
}
```

**Tips for writing the `body` field:**
- Use `\n` to start a new line
- Use `- ` at the start of a line for a bullet point
- Use `1. ` at the start of a line for a numbered step
- Keep it plain text — no special formatting needed

---

### Add a new action to a phase

1. Go to the phase folder (e.g. `phases/03-containment/`)
2. Click **Add file > Create new file**
3. Name it following the pattern: `CO-027-your-action-name.json`
   - Use the phase prefix (`CO` for Containment, `PR` for Preparation, etc.)
   - Use the next available number
   - Use hyphens, not spaces
4. Paste this template and fill it in:

```json
{
  "id": "CO-027",
  "name": "Your Action Name",
  "description": "Short description of what this action does",
  "url": "",
  "category": "network",
  "sections": [
    {
      "id": "s1",
      "title": "Procedure",
      "body": "Describe the steps here."
    }
  ]
}
```

5. Commit the file
6. **Important:** Open `_phase.json` in the same folder and add your new filename to the end of the `actions` array:

```json
{
  "id": "t3",
  "name": "Containment",
  "shortcode": "C",
  "url": "",
  "actions": [
    "CO-001-patch-vulnerability.json",
    "CO-002-block-external-ip-address.json",
    "...",
    "CO-027-your-action-name.json"
  ]
}
```

7. Commit `_phase.json`

The action now appears on the live page.

---

### Phase prefixes reference

| Folder | Prefix | Phase |
|--------|--------|-------|
| `01-preparation` | `PR` | Preparation |
| `02-identification` | `ID` | Identification |
| `03-containment` | `CO` | Containment |
| `04-eradication` | `ER` | Eradication |
| `05-recovery` | `RE` | Recovery |
| `06-lessons-learned` | `LL` | Lessons Learned |

---

### Update a use case

Use cases define which actions belong to a specific incident type. Open a file in `usecases/`, e.g. `usecases/phishing.json`:

```json
{
  "id": "pb1",
  "name": "Phishing",
  "color": "#e94560",
  "description": "Response playbook for phishing campaigns targeting users via email.",
  "techIds": ["PR-001", "PR-003", "ID-025", "ID-026", "CO-013", "CO-015", "ER-003", "LL-001"]
}
```

- **`techIds`** — action IDs in the order they should appear. Add an ID to include an action. Remove it to exclude. Reorder by moving IDs around.
- **`description`** — shown in the use cases panel and the response guide header.
- **`color`** — hex color used for highlighting on the matrix and in the guide.

---

### Add a new use case

1. Go to the `usecases/` folder
2. Click **Add file > Create new file**
3. Name it (e.g. `ransomware.json`) and fill it in:

```json
{
  "id": "pb8",
  "name": "Ransomware",
  "color": "#68d391",
  "description": "Response playbook for ransomware incidents.",
  "techIds": []
}
```

Use a unique `id` (e.g. `pb8`, `pb9`, etc.). Available colors: `#e94560` (red), `#f6ad55` (amber), `#68d391` (green), `#63b3ed` (blue), `#b794f4` (purple), `#81e6d9` (teal), `#fc8181` (coral), `#a29bfe` (lavender), `#fbd38d` (gold), `#90cdf4` (sky).

4. Commit the file
5. Open `index.json` and add `"usecases/ransomware.json"` to the `playbooks` array:

```json
{
  "name": "Incident Response — PICERL",
  "categories": "categories.json",
  "playbooks": [
    "usecases/phishing.json",
    "usecases/layer3-ddos.json",
    "...",
    "usecases/ransomware.json"
  ],
  "phases": [ "..." ]
}
```

6. Commit `index.json`

You can also add use cases from the UI by clicking the **+** button in the use cases panel, but those changes are session-only — to persist them, update the JSON files.

---

### Add or update a category

Open `categories.json`. Each entry looks like this:

```json
{
  "id": "network",
  "label": "Network",
  "color": "#2b6cb0",
  "bg": "#1a365d",
  "text": "#63b3ed"
}
```

- `id` — used in action files (e.g. `"category": "network"`)
- `label` — what appears in the UI
- `color` — the border/accent color (hex)
- `bg` — the background color of the badge
- `text` — the text color inside the badge

To add a new category, copy an existing entry, paste it at the end (before the closing `]`), and change all fields. You can also use the **Manage** button in the category filter bar on the live page.

**Current categories:** Uncategorized, General, Network, Email, File, Process, Configuration, Identity.

---

### Add a reference link to an action

Open the action file and set the `url` field:

```json
{
  "id": "CO-002",
  "name": "Block External IP Address",
  "url": "https://your-internal-wiki.com/block-ip",
  "..."
}
```

On the live page, the action cell will show a link icon. Clicking the action opens that URL in a new tab.

---

### Add a reference link to a section

Each section within an action can have its own URL:

```json
{
  "id": "s1",
  "title": "Procedure",
  "body": "Steps go here...",
  "url": "https://wiki.example.com/detailed-procedure"
}
```

A small link icon appears next to the section title on the live page.

---

### Add a reference link to a phase

Open the `_phase.json` file for the phase and set the `url` field:

```json
{
  "id": "t1",
  "name": "Preparation",
  "shortcode": "P",
  "url": "https://wiki.example.com/preparation-phase",
  "actions": [ "..." ]
}
```

Clicking the phase header on the live page opens that URL. The header also gets a distinct visual treatment to indicate it's linked.

---

## JSON formatting rules (important)

JSON has strict formatting rules. If you break them, the page will fail to load. Here are the most common mistakes:

**Always use double quotes, never single quotes:**
```json
"name": "Block External IP"   ✓
'name': 'Block External IP'   ✗
```

**Every item in a list except the last needs a comma:**
```json
"actions": [
  "CO-001-patch-vulnerability.json",
  "CO-002-block-external-ip-address.json",
  "CO-003-block-internal-ip-address.json"
]
```
Note: no comma after the last item.

**The same rule applies to objects inside lists:**
```json
"sections": [
  {
    "id": "s1",
    "title": "Procedure",
    "body": "Step 1..."
  },
  {
    "id": "s2",
    "title": "Tools",
    "body": "Tool list..."
  }
]
```

**If you're unsure your JSON is valid**, paste it into [jsonlint.com](https://jsonlint.com) before committing.

---

## How data loading works

When the page loads, it:

1. Fetches `index.json` to get the list of phases and use cases
2. Fetches `categories.json`, all use case files, and all `_phase.json` files in parallel
3. For each phase, fetches every action file listed in its `_phase.json` in parallel
4. Assembles everything into a single matrix and renders the app

All data is read-only from the repo. Edits made in the UI (renaming, reordering, etc.) are session-only and lost on refresh unless you export a snapshot.

---

## How to check if your change worked

1. After committing, go to the **Actions** tab in GitHub to see the deployment status
2. Wait for the yellow dot to turn green — usually under 60 seconds
3. Hard refresh the live URL (Ctrl+Shift+R on Windows, Cmd+Shift+R on Mac)

---

## Something broke

**The page shows a load error:** A JSON file has invalid formatting. Check what you last committed and look for missing commas, mismatched quotes, or unclosed brackets. The error message will usually name the file that failed.

**My change isn't showing:** Hard refresh the page (Ctrl+Shift+R). GitHub Pages sometimes caches aggressively.

**I accidentally deleted something:** Go to the file in GitHub, click **History**, find the version before your change, and copy the content back.

---

## Who maintains what

| File / Folder | Who edits it |
|---|---|
| `index.html` | Senior engineer only |
| `index.json` | Senior analyst (adding new phases or use cases) |
| `categories.json` | Senior analyst |
| `phases/*/[action].json` | Any analyst |
| `phases/*/_phase.json` | Senior analyst (when adding/removing actions) |
| `usecases/*.json` | Any analyst |
