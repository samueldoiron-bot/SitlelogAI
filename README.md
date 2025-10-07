# README.md

**SiteLogAI — Prototype (Web React + Mock Backend)**

This repository contains a single-file React prototype and a simple Express mock backend to demonstrate the AI Daily Log flow.

## What is included

- `frontend/src/App.jsx` — React single-page prototype (Tailwind-style classes used). Interactive screens: Dashboard, New Log, AI Summary, History. Uses a mock `fetch` to `/api/summarize`.
- `server/index.js` — Minimal Express server with `/api/summarize` route (placeholder for OpenAI integration).
- `.env.example` — shows environment variables to set (OPENAI_API_KEY).
- `package.json` — combined scripts to run frontend (Vite) and backend concurrently.

---

## Quick start (local)

1. Copy these files into a project folder.
2. Install dependencies:

```bash
npm install
```

3. Set environment variables (create `.env` from `.env.example`):

```
OPENAI_API_KEY=your_api_key_here
```

4. Start the app:

```bash
npm run dev
```

5. Open `http://localhost:5173` (Vite default) — frontend will hit mock backend at `http://localhost:3000/api/summarize`.

---

## Where to go next

- Replace the mock summarizer with the OpenAI API or your chosen LLM.
- Add authentication (Google/Microsoft) and project/team models.
- Add persistent storage (Postgres / Firebase) and file uploads (S3).
- Implement mobile using React Native (share components/business logic).

---

/* ---------- package.json ---------- */

// package.json
{
  "name": "sitelogai-prototype",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "concurrently \"node server/index.js\" \"vite\"",
    "start": "node server/index.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "concurrently": "^8.0.1",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2"
  },
  "devDependencies": {
    "vite": "^5.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}

/* ---------- .env.example ---------- */

# .env.example
OPENAI_API_KEY=your_openai_api_key_here

/* ---------- server/index.js ---------- */

// Minimal Express server with a mock summarization endpoint.
// Replace the mock logic with an actual OpenAI call when ready.

const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();

const app = express();
app.use(cors());
app.use(bodyParser.json());

const PORT = 3000;

app.post('/api/summarize', async (req, res) => {
  try {
    const { text, crewSize, siteName } = req.body || {};

    // MOCK AI processing — simulate labelling and summarizing
    // Replace below with call to OpenAI's API.
    const summary = `Date: ${new Date().toLocaleDateString()}\nSite: ${siteName || 'Unknown'}\nCrew: ${crewSize || 'N/A'}\nSummary: ${text.substring(0, 200)}...`;

    // simulated labels
    const labels = [];
    if (/delay|delayed|late/i.test(text)) labels.push('Delay');
    if (/safety|incident|injury/i.test(text)) labels.push('Safety');
    if (/delivery|delivered|received/i.test(text)) labels.push('Delivery');

    // return a JSON structure similar to what an LLM would return
    return res.json({ summary, labels, status: 'ok' });
  } catch (err) {
    console.error(err);
    return res.status(500).json({ error: 'Server error' });
  }
});

app.listen(PORT, () => {
  console.log(`Mock backend listening on http://localhost:${PORT}`);
});

/* ---------- frontend/index.html ---------- */

<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>SiteLogAI Prototype</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>

/* ---------- frontend/src/main.jsx ---------- */

import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'
import './index.css'

createRoot(document.getElementById('root')).render(<App />)

/* ---------- frontend/src/index.css ---------- */

/* Minimal styles and utility classes — ideally replace with Tailwind setup. */
body { font-family: Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial; margin: 0; background: #f5f7fb; }
.container { max-width: 980px; margin: 24px auto; padding: 16px; }
.header { display:flex; align-items:center; justify-content:space-between; margin-bottom:16px }
.btn { padding: 8px 12px; border-radius:8px; background:#1e3a8a; color:white; border:none; cursor:pointer }
.card { background:white; padding:16px; border-radius:10px; box-shadow: 0 6px 18px rgba(0,0,0,0.06); }
.input, textarea { width:100%; padding:8px; border-radius:8px; border:1px solid #e5e7eb; }
.nav { display:flex; gap:8px }
.badge { display:inline-block; padding:4px 8px; border-radius:999px; background:#fde68a }

/* ---------- frontend/src/App.jsx ---------- */

import React, { useState } from 'react'

export default function App() {
  const [view, setView] = useState('dashboard')
  const [site, setSite] = useState('NB Refinery')
  const [crewSize, setCrewSize] = useState(6)
  const [text, setText] = useState('')
  const [generated, setGenerated] = useState(null)
  const [logs, setLogs] = useState([])
  const [loading, setLoading] = useState(false)

  async function handleGenerate() {
    setLoading(true)
    try {
      const resp = await fetch('http://localhost:3000/api/summarize', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text, crewSize, siteName: site })
      })
      const data = await resp.json()
      setGenerated(data)
      setLoading(false)
    } catch (err) {
      console.error(err)
      setLoading(false)
    }
  }

  function handleSave() {
    const newLog = {
      id: Date.now(),
      date: new Date().toLocaleString(),
      site,
      crewSize,
      text,
      generated
    }
    setLogs([newLog, ...logs])
    setText('')
    setGenerated(null)
    setView('history')
  }

  return (
    <div className="container">
      <header className="header">
        <div>
          <h1 style={{margin:0}}>SiteLog AI — Prototype</h1>
          <small className="badge">Quick demo</small>
        </div>
        <nav className="nav">
          <button className="btn" onClick={() => setView('dashboard')}>Dashboard</button>
          <button className="btn" onClick={() => setView('new')}>New Log</button>
          <button className="btn" onClick={() => setView('history')}>History</button>
        </nav>
      </header>

      {view === 'dashboard' && (
        <div className="card">
          <h2>Welcome, Sam</h2>
          <p>Site: <strong>{site}</strong></p>
          <p>Recent logs: {logs.length}</p>
          <div style={{marginTop:12}}>
            <button className="btn" onClick={() => setView('new')}>Create new daily log</button>
          </div>
        </div>
      )}

      {view === 'new' && (
        <div className="card">
          <h3>New Daily Log</h3>
          <div style={{display:'grid', gap:8}}>
            <label>Site</label>
            <input className="input" value={site} onChange={e => setSite(e.target.value)} />

            <label>Crew size</label>
            <input className="input" type="number" value={crewSize} onChange={e => setCrewSize(Number(e.target.value))} />

            <label>Notes (voice or text)</label>
            <textarea rows={6} className="input" value={text} onChange={e => setText(e.target.value)} />

            <div style={{display:'flex', gap:8}}>
              <button className="btn" onClick={handleGenerate} disabled={loading}>{loading ? 'Generating...' : 'Generate AI Summary'}</button>
              <button className="btn" onClick={() => { setText(''); setGenerated(null); }}>Clear</button>
            </div>

            {generated && (
              <div style={{marginTop:12, padding:12, border:'1px solid #e5e7eb', borderRadius:8}}>
                <h4>AI Summary</h4>
                <pre style={{whiteSpace:'pre-wrap'}}>{generated.summary}</pre>
                <p>Labels: {generated.labels.join(', ') || '—'}</p>
                <div style={{display:'flex', gap:8}}>
                  <button className="btn" onClick={handleSave}>Save Log</button>
                </div>
              </div>
            )}
          </div>
        </div>
      )}

      {view === 'history' && (
        <div className="card">
          <h3>Logs History</h3>
          {logs.length === 0 && <p>No logs yet.</p>}
          <div style={{display:'grid', gap:8}}>
            {logs.map(l => (
              <div key={l.id} style={{padding:12, border:'1px solid #eef2ff', borderRadius:8}}>
                <div style={{display:'flex', justifyContent:'space-between'}}>
                  <strong>{l.site}</strong>
                  <small>{l.date}</small>
                </div>
                <div style={{marginTop:8}}>
                  <pre style={{whiteSpace:'pre-wrap'}}>{l.generated?.summary || l.text}</pre>
                </div>
              </div>
            ))}
          </div>
        </div>
      )}

    </div>
  )
}


/* ---------- NOTES ---------- */

- This prototype is intentionally small and focused. It demonstrates the input → AI → formatted log flow.
- When you're ready, I can:
  - Implement the OpenAI integration server-side with robust prompt engineering and templates.
  - Add authentication (Firebase/Auth0) and database persistence (Postgres or Firebase).
  - Create mobile versions (React Native) reusing React logic and components.


<!-- End of document -->
