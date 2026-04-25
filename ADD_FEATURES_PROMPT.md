# Prompt: Add 4 Features to Existing Single-File React App

Paste this entire prompt into a Claude conversation along with your existing `index.html` file.

---

I have a single-file React app (`index.html`) that uses React 18 via CDN with no build step. I want you to add four features to it. The app already has its own UI, layout, and styling — **do not redesign or restyle anything that already exists**. Wire each feature into wherever it naturally fits in the existing structure. Adapt all code snippets to match the conventions already in the file (variable names, spacing, component patterns, etc.).

Here are the four features to add:

---

## Feature 1: Live UTC Clock + Elapsed Timer (ClockBar)

Add a `ClockBar` component. It displays two things: a live UTC clock that ticks every second, and an incident elapsed timer that counts up from a start timestamp.

**Props:**
- `timerStart` — ms timestamp (from `Date.now()`) when the timer started, or `null` if not running
- `onTimerStart` — callback, called with no arguments when the user starts the timer
- `onTimerStop` — callback, called with no arguments when the user stops and resets the timer
- `compact` — boolean, reduces sizing for use in tight header bars
- `hideTimer` — boolean, hides the elapsed section entirely (shows only the UTC clock)

**Behavior:**
- Use `setInterval` inside a `useEffect` to tick every 1000ms
- UTC clock format: `HH:MM:SS UTC` using `getUTCHours()`, `getUTCMinutes()`, `getUTCSeconds()`
- Elapsed format: `HH:MM:SS`, counting seconds from `timerStart` to `Date.now()`. When `timerStart` is null, show `––:––:––`
- Show a start/stop toggle button: `▶ START` when idle, `■ STOP` when running
- Show a small pulsing red dot while the timer is running (use a CSS keyframe animation called `rec-dot`)
- Timer state lives in the parent — `ClockBar` only receives and displays it, never owns it

**Code to use directly:**

```javascript
function ClockBar({ timerStart, onTimerStart, onTimerStop, compact, hideTimer }) {
  const [utcTime, setUtcTime] = React.useState('');
  const [elapsed, setElapsed] = React.useState('––:––:––');

  React.useEffect(() => {
    function tick() {
      const now = new Date();
      const p = n => String(n).padStart(2, '0');
      setUtcTime(p(now.getUTCHours()) + ':' + p(now.getUTCMinutes()) + ':' + p(now.getUTCSeconds()) + ' UTC');
      if (timerStart) {
        const diff = Math.max(0, Math.floor((Date.now() - timerStart) / 1000));
        setElapsed(p(Math.floor(diff / 3600)) + ':' + p(Math.floor((diff % 3600) / 60)) + ':' + p(diff % 60));
      } else {
        setElapsed('––:––:––');
      }
    }
    tick();
    const id = setInterval(tick, 1000);
    return () => clearInterval(id);
  }, [timerStart]);

  const isRunning = !!timerStart;

  // Render: show elapsed section (unless hideTimer), then a divider, then the UTC clock.
  // The elapsed section contains: label "ELAPSED", the elapsed value, the rec-dot (if running),
  // and the start/stop button.
  // Wire onTimerStart / onTimerStop to the button's onClick.
  // Apply compact sizing based on the compact prop.
  // Style to match your existing UI — layout and colors are up to you.
}
```

Add this CSS for the pulsing dot:
```css
@keyframes rec-pulse {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.2; }
}
.rec-dot { animation: rec-pulse 1.2s ease-in-out infinite; }
```

Place `ClockBar` in your main header. Pass `hideTimer={true}` if you only want the UTC clock there. Also place it (with the timer visible) in the Incident View header if you build Feature 3.

---

## Feature 2: Per-Card Time Tracking in the Incident View

Each action card in the Incident View must track timestamps automatically as its status changes. No user input needed — timestamps are set by the app when a card moves between columns.

**Card data shape** (the fields that matter for time tracking):
```javascript
{
  id: uid(),
  status: 'todo',          // 'todo' | 'inprogress' | 'blocked' | 'completed'
  techName: '',            // the action label
  tacticShortcode: '',     // phase label (e.g. 'I', 'C', 'E', 'R', 'AD')
  owner: '',
  notes: '',
  images: [],
  todoAt:      Date.now(), // set when first created
  assignedAt:  null,       // set when owner is first assigned
  startedAt:   null,       // set when first moved to 'inprogress'
  blockedAt:   null,       // set when moved to 'blocked'
  completedAt: null,       // set when first moved to 'completed'
}
```

**moveCard function** — use this exact logic so timestamps are only ever set once (never overwritten on a second visit to the same status):
```javascript
function moveCard(cardId, newStatus) {
  updateInc(activeId, i => ({
    ...i,
    cards: i.cards.map(c => {
      if (c.id !== cardId) return c;
      const t = Date.now();
      return {
        ...c,
        status: newStatus,
        todoAt:      newStatus === 'todo'        && !c.todoAt      ? t : c.todoAt,
        startedAt:   newStatus === 'inprogress'  && !c.startedAt   ? t : c.startedAt,
        blockedAt:   newStatus === 'blocked'                        ? t : c.blockedAt,
        completedAt: newStatus === 'completed'   && !c.completedAt  ? t : c.completedAt,
      };
    }),
  }));
}
```

**Live elapsed ticker on each incident:** Add a `now` state variable that ticks every second, and use it to show how long each incident has been open:
```javascript
const [now, setNow] = React.useState(Date.now());
React.useEffect(() => {
  const t = setInterval(() => setNow(Date.now()), 1000);
  return () => clearInterval(t);
}, []);

// Then in your render, per incident:
function fmtElapsed(startMs, nowMs) {
  const diff = Math.max(0, Math.floor((nowMs - startMs) / 1000));
  const p = n => String(n).padStart(2, '0');
  return p(Math.floor(diff / 3600)) + ':' + p(Math.floor((diff % 3600) / 60)) + ':' + p(diff % 60);
}
// Usage: fmtElapsed(incident.createdAt, now)  →  "01:24:07"
```

**Export helpers** — use these when building the CSV export:
```javascript
const p = n => String(n).padStart(2, '0');

function fmtUTC(ms) {
  if (!ms) return '';
  const d = new Date(ms);
  return `${d.getUTCFullYear()}-${p(d.getUTCMonth()+1)}-${p(d.getUTCDate())} ` +
         `${p(d.getUTCHours())}:${p(d.getUTCMinutes())}:${p(d.getUTCSeconds())} UTC`;
}

function fmtDuration(ms) {
  if (!ms || ms < 0) return '';
  const s = Math.floor(ms / 1000);
  return `${p(Math.floor(s / 3600))}:${p(Math.floor((s % 3600) / 60))}:${p(s % 60)}`;
}

function lastActionMs(card) {
  return Math.max(...[card.todoAt, card.assignedAt, card.startedAt, card.blockedAt, card.completedAt].filter(Boolean));
}
```

---

## Feature 3: Incident View

Add a full-screen Incident View that replaces the main view when the user enters it. It is triggered by a button in your existing UI (e.g. "Open Incident View" or similar — place it wherever makes sense). It can be toggled with a boolean state variable like `showIncident`.

**Component signature:**
```javascript
function IncidentView({ matrix, theme, dark, onClose, timerStart, onTimerStart, onTimerStop, onThemeChange }) { }
```

**State inside IncidentView:**
```javascript
const [incidents, setIncidents]         = React.useState([...]);  // see Feature 4 for init
const [activeId, setActiveId]           = React.useState(...);    // see Feature 4 for init
const [now, setNow]                     = React.useState(Date.now());
const [showAdd, setShowAdd]             = React.useState(false);
const [editingIncId, setEditingIncId]   = React.useState(null);
const [editIncName, setEditIncName]     = React.useState('');
const [collapsedGroups, setCollapsedGroups] = React.useState(new Set());
const [sidebarOpen, setSidebarOpen]     = React.useState(true);
```

**Derived values:**
```javascript
const activeInc = incidents.find(i => i.id === activeId) || incidents[0];
```

**Incident management functions:**
```javascript
function createIncident() {
  const id = uid();
  const n  = incidents.length + 1;
  setIncidents(p => [...p, { id, name: `INC-00${n} New Incident`, createdAt: Date.now(), playbookId: null, cards: [] }]);
  setActiveId(id);
}

function deleteIncident(id) {
  setIncidents(p => {
    const next = p.filter(i => i.id !== id);
    if (next.length === 0) {
      const nid = uid();
      setActiveId(nid);
      return [{ id: nid, name: 'INC-001 New Incident', createdAt: Date.now(), playbookId: null, cards: [] }];
    }
    if (activeId === id) setActiveId(next[0].id);
    return next;
  });
}

function updateInc(id, fn) {
  setIncidents(p => p.map(i => i.id === id ? fn(i) : i));
}

function resetAll() {
  if (!window.confirm('Reset all incidents? This cannot be undone.')) return;
  const freshId = uid();
  setIncidents([{ id: freshId, name: 'INC-001 New Incident', createdAt: Date.now(), playbookId: null, cards: [] }]);
  setActiveId(freshId);
  onTimerStop();
  try {
    localStorage.removeItem('ir_navigator_incidents');
    localStorage.removeItem('ir_navigator_activeId');
    localStorage.removeItem('ir_navigator_timer');
  } catch(e) {}
}
```

**Card management** — use `moveCard` from Feature 2, plus:
```javascript
function addCard(techName, tacticShortcode, tacticName) {
  const card = {
    id: uid(), techName, tacticShortcode, tacticName,
    status: 'todo', owner: '', notes: '', images: [],
    todoAt: Date.now(), assignedAt: null, startedAt: null, blockedAt: null, completedAt: null,
  };
  updateInc(activeId, i => ({ ...i, cards: [...i.cards, card] }));
}

function removeCard(cardId) {
  updateInc(activeId, i => ({ ...i, cards: i.cards.filter(c => c.id !== cardId) }));
}

function setOwnerFn(cardId, owner) {
  updateInc(activeId, i => ({
    ...i,
    cards: i.cards.map(c => c.id !== cardId ? c : {
      ...c, owner,
      assignedAt: owner && !c.assignedAt ? Date.now() : (owner ? c.assignedAt : null),
    }),
  }));
}

function setNotesFn(cardId, notes) {
  updateInc(activeId, i => ({
    ...i, cards: i.cards.map(c => c.id === cardId ? { ...c, notes } : c),
  }));
}
```

**Layout (UI agnostic — structure only):**
- A fixed header with: back/close button (`onClose`), incident name (click-to-edit inline text), export buttons, reset button, theme toggle (if `onThemeChange` is provided), and `ClockBar` with the timer visible.
- A collapsible left sidebar listing all incidents. Each entry shows name, created time, card count. Active incident is highlighted. Include a "+ New Incident" button.
- A main kanban area with 4 columns: **TO DO** (`todo`), **IN PROGRESS** (`inprogress`), **BLOCKER / HOLD** (`blocked`), **COMPLETED** (`completed`). All 4 columns must be visible — size each at 25% width (or `flex: 1`) so they all fit. Allow horizontal scroll as a fallback.
- Cards within each column are grouped by `tacticShortcode` (phase). Groups are collapsible via `toggleGroup`.
- Each card shows: phase label, action name, owner field, status move buttons, notes/screenshot expand section, and a remove button.

**Inline name editing:**
```javascript
// Clicking the incident name enters edit mode:
// setEditingIncId('topbar'); setEditIncName(activeInc.name);
// On blur or Enter: save the new name back into incidents state.
// On Escape: cancel.
```

**Export Timeline (CSV):**
```javascript
function exportTimeline() {
  const inc = activeInc;

  function lastActionMs(c) {
    return Math.max(...[c.todoAt, c.assignedAt, c.startedAt, c.blockedAt, c.completedAt].filter(Boolean));
  }

  const completed = inc.cards.filter(c => c.status === 'completed').sort((a,b) => (a.completedAt||0) - (b.completedAt||0));
  const rest      = inc.cards.filter(c => c.status !== 'completed').sort((a,b) => lastActionMs(a) - lastActionMs(b));
  const sorted    = [...completed, ...rest];

  const escape = v => `"${String(v || '').replace(/"/g, '""')}"`;

  const headers = ['Started Time (UTC)', 'Completed Time (UTC)', 'Duration (HH:MM:SS)', 'Last Action Time (UTC)', 'Status', 'Phase', 'Action', 'Owner', 'Notes', 'Screenshots'];

  const metaRows = [
    ['Incident', escape(inc.name)],
    ['Created',  escape(fmtUTC(inc.createdAt))],
    ['Total Incident Duration', escape(timerStart ? fmtDuration(Date.now() - timerStart) : 'Timer not used')],
    ['Exported', escape(fmtUTC(Date.now()))],
    [],
    headers.map(escape),
  ];

  const dataRows = sorted.map(c => {
    const duration = c.startedAt && c.completedAt ? fmtDuration(c.completedAt - c.startedAt) : '';
    const statusLabel = c.status === 'inprogress' ? 'In Progress' : c.status === 'todo' ? 'To Do' : c.status === 'blocked' ? 'Blocked' : 'Completed';
    return [
      escape(fmtUTC(c.startedAt)),
      escape(fmtUTC(c.completedAt)),
      escape(duration),
      escape(fmtUTC(lastActionMs(c))),
      escape(statusLabel),
      escape(c.tacticShortcode || ''),
      escape(c.techName || ''),
      escape(c.owner || 'Unassigned'),
      escape(c.notes || ''),
      escape((c.images || []).length),
    ];
  });

  const csv = [...metaRows, ...dataRows].map(r => r.join(',')).join('\r\n');
  const a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([csv], { type: 'text/csv' }));
  a.download = `timeline-${inc.name.replace(/\s+/g, '-')}.csv`;
  a.click();
}
```

**Export JSON:**
```javascript
function exportIncident() {
  const inc = activeInc;
  const data = {
    incident:  inc.name,
    createdAt: new Date(inc.createdAt).toISOString(),
    exportedAt: new Date().toISOString(),
    summary: {
      todo:       inc.cards.filter(c => c.status === 'todo').length,
      inProgress: inc.cards.filter(c => c.status === 'inprogress').length,
      blocked:    inc.cards.filter(c => c.status === 'blocked').length,
      completed:  inc.cards.filter(c => c.status === 'completed').length,
    },
    actions: inc.cards.map(c => ({
      name:        c.techName,
      phase:       c.tacticShortcode,
      status:      c.status,
      owner:       c.owner || null,
      notes:       c.notes || null,
      todoAt:      c.todoAt      ? new Date(c.todoAt).toISOString()      : null,
      startedAt:   c.startedAt   ? new Date(c.startedAt).toISOString()   : null,
      completedAt: c.completedAt ? new Date(c.completedAt).toISOString() : null,
    })),
  };
  const a = document.createElement('a');
  a.href = URL.createObjectURL(new Blob([JSON.stringify(data, null, 2)], { type: 'application/json' }));
  a.download = `incident-${inc.name.replace(/\s+/g, '-')}.json`;
  a.click();
}
```

---

## Feature 4: Browser Persistence (localStorage)

All incident data and the timer must survive a page refresh. Use three localStorage keys.

**In the root App component**, manage the timer:
```javascript
const LS_TIMER = 'ir_navigator_timer';

const [timerStart, setTimerStart] = React.useState(() => {
  try {
    const saved = localStorage.getItem(LS_TIMER);
    if (saved) return parseInt(saved, 10);
  } catch(e) {}
  return null;
});

function startTimer() {
  const t = Date.now();
  setTimerStart(t);
  try { localStorage.setItem(LS_TIMER, String(t)); } catch(e) {}
}

function stopTimer() {
  setTimerStart(null);
  try { localStorage.removeItem(LS_TIMER); } catch(e) {}
}
```

Pass `timerStart`, `startTimer`, `stopTimer` as props all the way down to `ClockBar` and `IncidentView`.

**Inside IncidentView**, persist incidents and the active incident ID:
```javascript
const LS_INC = 'ir_navigator_incidents';
const LS_AID = 'ir_navigator_activeId';

// Load on mount:
const [incidents, setIncidents] = React.useState(() => {
  try {
    const saved = localStorage.getItem(LS_INC);
    if (saved) {
      const parsed = JSON.parse(saved);
      if (Array.isArray(parsed) && parsed.length > 0) return parsed;
    }
  } catch(e) {}
  // Default: one blank incident
  const id = uid();
  return [{ id, name: 'INC-001 New Incident', createdAt: Date.now(), playbookId: null, cards: [] }];
});

const [activeId, setActiveId] = React.useState(() => {
  try {
    const saved = localStorage.getItem(LS_AID);
    if (saved) return saved;
  } catch(e) {}
  return incidents[0]?.id || null;
});

// Save on every change:
React.useEffect(() => {
  try { localStorage.setItem(LS_INC, JSON.stringify(incidents)); } catch(e) {}
}, [incidents]);

React.useEffect(() => {
  try { localStorage.setItem(LS_AID, activeId); } catch(e) {}
}, [activeId]);
```

**On Reset All**, clear all three keys:
```javascript
localStorage.removeItem('ir_navigator_incidents');
localStorage.removeItem('ir_navigator_activeId');
localStorage.removeItem('ir_navigator_timer');
```

**Rules:**
- Always wrap localStorage reads and writes in `try/catch` — private browsing or storage quota errors should fail silently, never crash the app.
- Never read from localStorage after mount — only write. The initial state is set once via the lazy `useState` initializer.
- The timer key is managed in `App`, not in `IncidentView`, so the timer persists regardless of whether the user is in the Matrix view or Incident View.

---

## Helper functions (add these globally if not already present)

```javascript
function uid() {
  return Math.random().toString(36).slice(2, 8);
}

function dateStamp() {
  const d = new Date();
  const p = n => String(n).padStart(2, '0');
  return `${d.getUTCFullYear()}${p(d.getUTCMonth()+1)}${p(d.getUTCDate())}`;
}
```

---

## Summary of wiring

1. Add `ClockBar` as a standalone component near the top of the file.
2. Add timer state (`timerStart`, `startTimer`, `stopTimer`) to the root `App` component with localStorage persistence.
3. Render `ClockBar` (with `hideTimer` if desired) in the existing main header, passing `timerStart`, `startTimer`, `stopTimer`.
4. Add `showIncident` boolean state to `App`. Add a button to toggle it.
5. Render `IncidentView` when `showIncident` is true, passing all required props. It should cover/replace the main view.
6. All localStorage logic lives in `App` (timer) and `IncidentView` (incidents). No other components touch localStorage.
