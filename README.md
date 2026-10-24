# 🎯 Opportunity Radar

> Live opportunity radar for Indian college students. Discovers currently open internships, hackathons, competitions, scholarships, fellowships, and open-source programs matched to their profile using live web search and Gemini intelligence.

---

## ⚡ What Opportunity Radar Does

1. **Understands the Student**: Analyzes student background (degree, year, college, skills, focus areas) and query intent using **Gemini 2.5 Flash** to identify search keywords, study level, and location preferences.
2. **Scans 20 Trusted Portals in Real-Time**: Queries live web indexes via the **TinyFish Search API** and grabs active listing pages across 20 curated Indian and global platforms.
3. **Triages & Filters Web Noise**: An AI triage step rejects listicles ("top 10", "tips"), past recaps, and expired archives to pinpoint high-signal opportunity URLs.
4. **Deep Page Content Extraction**: Fetches full markdown page texts with the **TinyFish Fetch API** (with 30-minute in-memory caching and concurrency limits).
5. **Rigorous Opportunity Verification**: Gemini inspects the live page text to extract only authentic, currently open programs with explicit deadlines, eligibility rules, stipends/prizes, and direct links.
6. **Code-Level Deadline & Fit Ranking**: Strictly purges expired deadlines (`deadline_iso < today`). Dedupes by organization and ranks opportunities by personalized student fit score (1–10) and nearest closing deadline.

---

## 🚀 Quickstart in 4 Steps

### Step 1: Install Dependencies
```bash
npm install
```

### Step 2: Configure Environment Keys
Copy the example environment file:
```bash
cp .env.example .env.local
```
Edit `.env.local` with your API keys:
```env
# TinyFish API Key (from https://agent.tinyfish.ai/api-keys)
TINYFISH_API_KEY=your_actual_tinyfish_api_key

# Google Gemini API Key (from https://aistudio.google.com)
GEMINI_API_KEY=your_actual_gemini_api_key

# Optional: Gemini Model (defaults to gemini-2.5-flash)
GEMINI_MODEL=gemini-2.5-flash
```

### Step 3: Test Trusted Sources
Run the built-in diagnostic test to verify TinyFish connectivity and check which sources return clean text:
```bash
npm run test:sources
```
This fetches all 20 source URLs in batches of 10 and prints an ASCII summary table detailing status (`ok`, `thin`, `failed`), characters returned, and a text preview.

### Step 4: Launch Development Server
```bash
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser.

---

## 🛡️ Trusted Sources (20 Curated Portals)

All sources are defined in [`config/sources.ts`](file:///c:/Users/it%20admin/Desktop/Worshop/config/sources.ts):

| Category | Source Name | URL | Domain |
|---|---|---|---|
| **Internships** | Internshala | `https://internshala.com/internships/` | `internshala.com` |
| **Internships** | AICTE Internship Portal | `https://internship.aicte-india.org/internships` | `internship.aicte-india.org` |
| **Internships** | PM Internship Scheme | `https://pminternship.mca.gov.in` | `pminternship.mca.gov.in` |
| **Hackathons** | Devfolio | `https://devfolio.co/hackathons` | `devfolio.co` |
| **Hackathons** | Hack2skill *(jsHeavy)* | `https://hack2skill.com/hackathons-listing` | `hack2skill.com` |
| **Hackathons** | HackCulture | `https://www.hackculture.io` | `hackculture.io` |
| **Hackathons** | MLH | `https://www.mlh.com/events` | `mlh.com` |
| **Hackathons** | lablab.ai | `https://lablab.ai/ai-hackathons` | `lablab.ai` |
| **Hackathons** | DoraHacks *(jsHeavy)* | `https://dorahacks.io/hackathon` | `dorahacks.io` |
| **Competitions** | Unstop *(jsHeavy)* | `https://unstop.com/competitions` | `unstop.com` |
| **Competitions** | Kaggle *(jsHeavy)* | `https://www.kaggle.com/competitions` | `kaggle.com` |
| **Competitions** | MyGov Innovate India | `https://innovateindia.mygov.in/` | `innovateindia.mygov.in` |
| **Scholarships** | Buddy4Study *(jsHeavy)* | `https://www.buddy4study.com/online-application-form` | `buddy4study.com` |
| **Scholarships** | National Scholarship Portal | `https://scholarships.gov.in/Students` | `scholarships.gov.in` |
| **Scholarships** | DAAD India | `https://www.daad.in/en/find-funding/scholarship-database/` | `daad.in` |
| **Fellowships** | IASc Summer Research Fellowship | `https://webjapps.ias.ac.in/fellowship2027/index.html` | `ias.ac.in` |
| **Fellowships** | Teach For India | `https://teachforindia.org/how-to-be-a-fellow` | `teachforindia.org` |
| **Open Source** | Google Summer of Code | `https://developers.google.com/open-source/gsoc/timeline` | `developers.google.com` |
| **Open Source** | GirlScript Summer of Code | `https://gssoc.girlscript.org` | `gssoc.girlscript.org` |
| **Open Source** | Outreachy | `https://www.outreachy.org/apply/project-selection/` | `outreachy.org` |

### How to Add or Remove Sources
To add or modify a source, simply edit [`config/sources.ts`](file:///c:/Users/it%20admin/Desktop/Worshop/config/sources.ts):
```typescript
{
  name: "New Portal",
  url: "https://newportal.in/opportunities",
  domain: "newportal.in",
  category: "Internships", // One of: "Internships" | "Hackathons" | "Competitions" | "Scholarships" | "Fellowships" | "Open Source"
  jsHeavy: false, // Set to true if portal requires heavy client-side React rendering
}
```

### Note on `jsHeavy` Sources
Several portals (such as **Unstop**, **Hack2skill**, **Kaggle**, **Buddy4Study**, and **DoraHacks**) rely on client-side React/SPA hydration. 
- With standard HTTP fetch, these pages may return minimal or "thin" text (< 500 characters).
- In the UI's collapsible **Sources Panel**, these sources are tagged with a `JS` badge.
- These sources are prime candidates for the interactive **TinyFish Agent API** in a future version to interactively click through tabs and dynamic pagination.

---

## 🔍 Detailed Server Console Logging

Opportunity Radar logs diagnostics at every step to make pipeline tracing and debugging seamless:
- `[Gemini Understand]`: Extracted keywords, student profile level, and the 5 generated search queries.
- `[TinyFish Search]`: Search queries, domain filters, and raw response JSON from the initial call.
- `[Radar Gather]`: Total unique URLs discovered from keyword search and listing portals.
- `[Gemini Triage]`: URLs evaluated, which 8 were picked as specific opportunity pages, and reasons logged.
- `[TinyFish Fetch]`: Batch sizes, cache hits (30-minute in-memory cache), and characters returned per URL.
- `[Gemini Extract]`: Extraction prompts sent per page (concurrency pool of 4) and opportunities found.
- `[Validator]`: Count and titles of expired opportunities dropped (`deadline_iso < today`), deduplication statistics, and final top 30 ranked list.

---

## 🛡️ Security & Privacy

- **Zero Client-Side Keys**: All calls to TinyFish and Google Gemini take place exclusively inside server-side API routes (`/api/radar`).
- **No Keys in Client Bundles**: API keys never reach the browser.
- **Friendly Error Handling**: 
  - `401`: "Your API key looks wrong"
  - `429`: "Too many requests, wait a minute"
  - `402`: "Daily free limit reached"
  - Network timeout / unreachable: "Couldn't reach TinyFish"
