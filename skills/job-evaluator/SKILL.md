---
name: job-evaluator
description: Given one or more company names or URLs, produces a comprehensive job evaluation report tailored to the candidate profile defined in PROFILE.md. Searches Glassdoor, Kununu, Levels.fyi, LinkedIn, Remotely.de, Xing, Indeed.de, Monster.de, Comprehensive.io, Layoffs.fyi, Hiring.cafe, Builtin.com, and Wellfound.com. Use this skill when the user provides company names or URLs, researches job listings, or uses phrases like "evaluate this company", "should I apply here", "what's the salary", "what are the employee reviews", "compare these companies".
---

# Job Evaluator Skill

## Step 0 — Load Candidate Profile

Before doing anything else, read the file `PROFILE.md` located in the same directory as this skill.
Extract the candidate's name, target roles, tech stack, work model preferences, salary floor, equity expectation, and all other fields.
Use this profile to personalise every section of the report.
If `PROFILE.md` cannot be found, ask the user to provide their profile before continuing.

---

## Data Integrity Policy

This skill operates in **strict zero-hallucination mode**:

- **Never infer, estimate, or fabricate** any data point not explicitly found in a search result.
- If a salary range, rating, tech stack, work model, or open position is not present in the source, write **"data not available"**.
- Do not fill gaps using general market knowledge or training data — only report what the web search actually returns.
- Do not guess that a company "probably" uses a certain tech stack or "likely" offers equity.
- If a search returns no relevant results for a specific source, write **"No results found on [source name]"**.
- Cite the source URL next to every data point.

---

## Research Steps

Run the following web searches for each company provided. Parallelize where possible.

### Reviews & Ratings
1. **Glassdoor:** `[company name] Glassdoor reviews` → overall rating, CEO approval %, recommendation rate, top pros/cons
2. **Kununu:** `[company name] Kununu Bewertungen` → German-market employee reviews and rating

### Compensation
3. **Levels.fyi:** `[company name] levels.fyi engineer salary Germany` → salary by level, location
4. **Comprehensive.io:** `[company name] site:app.comprehensive.io/benchmarking/postings` → market salary ranges for target roles
5. **Benefits/Equity:** `[company name] employee benefits Germany equity RSU bonus` → equity structure, bonus, perks

### Job Openings
Search all sources below. Consolidate all matching positions into one table. Only include roles that match the candidate's target roles from PROFILE.md.

6. **LinkedIn:** `[company name] [target roles] jobs [candidate location preferences]`
7. **Greenhouse:** `[company name] site:job-boards.greenhouse.io` or `[company name] site:job-boards.eu.greenhouse.io` → direct ATS listings with apply links
8. **Xing:** `[company name] Xing Stellenangebote [target roles]`
9. **Indeed.de:** `[company name] indeed.de [target roles]`
10. **Monster.de:** `[company name] monster.de engineer jobs`
11. **Remotely.de:** `[company name] remotely.de engineer`
12. **Hiring.cafe:** `[company name] site:hiring.cafe` or `[company name] hiring.cafe [target role] remote`
13. **Builtin.com:** `[company name] site:builtin.com [target role]`
14. **Wellfound.com:** `[company name] site:wellfound.com [target role]`
15. **Careers page:** `[company name] careers jobs [target roles]`

### Stability
16. **Layoffs.fyi:** `[company name] layoffs.fyi` → layoff events, dates, headcount reductions

---

## Report Format

Produce one report per company using the template below. Write in **English**.

---

### 🏢 [COMPANY NAME]
**Industry:** | **Size:** | **HQ:** | **Work Model:**

#### ⚡ QUICK OVERVIEW
| Criteria | Value | Source |
|----------|-------|--------|
| Glassdoor Rating | X.X / 5 | [Glassdoor](url) |
| Kununu Rating | X.X / 5 | [Kununu](url) |
| CEO Approval Rate | XX% | [Glassdoor](url) |
| Recommendation Rate | XX% | [Glassdoor](url) / [Kununu](url) |
| Salary Competitiveness | ⭐⭐⭐⭐⭐ | [Levels.fyi](url) / [Comprehensive.io](url) |
| Recent Layoffs (12 mo) | ✅ None / ⚠️ [date + size] | [Layoffs.fyi](url) |

#### ⚠️ LAYOFF HISTORY
Summarize layoff events found on Layoffs.fyi or in news results:
- **[Month Year]:** ~X% of workforce / ~X,XXX employees — [brief reason if stated in source] — [Source](url)

If no layoffs found: `No layoffs recorded on Layoffs.fyi or in recent news.`

#### 💰 SALARY & PACKAGE
- **Market Range (target roles):** €XXX,XXX – €XXX,XXX base — [source](url)
- **Reported Salaries at This Company:** [cite source and URL]
- **Equity:** [what was found, or "data not available"]
- **Bonus:** [what was found, or "data not available"]
- **Benefits:** [what was found, or "data not available"]

#### ✅ PROS
(From employee reviews — most frequently mentioned. Include source URL per point where possible.)
- ...

#### ❌ CONS
(From employee reviews — most frequently mentioned. Include source URL per point where possible.)
- ...

#### 🎯 CANDIDATE FIT
Evaluate against the candidate profile loaded from PROFILE.md. Adapt criteria to the profile.

| Criteria | Status | Notes |
|----------|--------|-------|
| Tech Stack alignment | ✅/⚠️/❌ | [which stack was confirmed, which was not found] |
| Target Role Available | ✅/⚠️/❌ | [role name or "none found"] |
| Work Model match | ✅/⚠️/❌ | [remote/hybrid/on-site — city if relevant] |
| Salary meets floor | ✅/⚠️/❌ | [salary found vs. floor from profile] |
| Equity Available | ✅/⚠️/❌ | [type if found, ⚠️ if not found] |
| Required working language | ✅/⚠️/❌ | [English/other] |
| Engineering / IC Culture | ✅/⚠️/❌ | [based on reviews — only if explicitly mentioned] |
| Company Stability | ✅/⚠️/❌ | [layoff history, funding, profitability] |

> ⚠️ Only mark ✅ or ❌ if a data point was explicitly found. Use ⚠️ when uncertain due to missing data.

#### 💼 OPEN POSITIONS
Consolidate all relevant roles found across job sources. Only include roles matching the candidate's target roles.

| Title | Source | Location | Work Model | Salary (if listed) | Posted | Link |
|-------|--------|----------|------------|--------------------|--------|------|
| [Job Title] | [Source name] | [City / Remote] | Remote / Hybrid / On-site | €XXX,XXX or "not listed" | [date or "recent"] | [Apply](url) |

If no matching roles found: `No open positions found matching the candidate's target roles.`

#### 🔗 SOURCES
List every source searched and whether it returned relevant data:
- 🔍 Glassdoor: [link or "no results"]
- 🔍 Kununu: [link or "no results"]
- 💰 Levels.fyi: [link or "no results"]
- 💰 Comprehensive.io: [link or "no results"]
- 📉 Layoffs.fyi: [link or "no results"]
- 💼 LinkedIn Jobs: [link or "no results"]
- 💼 Xing Jobs: [link or "no results"]
- 💼 Indeed.de: [link or "no results"]
- 💼 Monster.de: [link or "no results"]
- 🌐 Remotely.de: [link or "no results"]
- ☕ Hiring.cafe: [link or "no results"]
- 🏗️ Builtin.com: [link or "no results"]
- 🚀 Wellfound.com: [link or "no results"]
- 🏢 Careers Page: [link or "no results"]

#### 🏁 OVERALL ASSESSMENT
**Decision:** 🟢 APPLY | 🟡 RESEARCH MORE | 🔴 SKIP

**Rationale:** (2–3 sentences based solely on data found above. Do not speculate beyond what was found.)

---

## Multiple Companies

If the user provides multiple companies, run the full report for each, then append a comparison table:

### 📊 COMPARISON TABLE
| Company | Glassdoor | Salary Fit | Stack Fit | Model Fit | Stability | Decision |
|---------|-----------|------------|-----------|-----------|-----------|----------|
| [Name] | X.X / 5 | ✅/⚠️/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | ✅/⚠️/❌ | 🟢/🟡/🔴 |

---

## Behavioural Rules

- Write in **English** regardless of the user's input language.
- Never produce placeholder text in the final output — if data is missing, say so explicitly.
- Do not add commentary beyond what was found in sources.
- Do not suggest the candidate "may want to verify" something that you could search for yourself — search it first.
- Use the salary floor and equity preference from PROFILE.md as the threshold for ✅/⚠️/❌ in Candidate Fit.
- Layoffs within the last 12 months: flag as ⚠️ in both QUICK OVERVIEW and CANDIDATE FIT.
