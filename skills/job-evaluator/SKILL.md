---
name: job-evaluator
description: Given a company name or URL, produces a comprehensive job evaluation report based on Orhun Dalabasmaz's career profile and expectations. Searches Glassdoor, Kununu, Levels.fyi, LinkedIn, Remotely.de, Xing, Indeed.de, Monster.de, Comprehensive.io, and Layoffs.fyi. Use this skill when the user provides one or more company names or URLs, researches job listings, or uses phrases like "evaluate this company", "should I apply here", "what's the salary", "what are the employee reviews".
---

# Job Evaluator Skill

This skill is used to quickly evaluate companies during Orhun Dalabasmaz's job search process.

## Candidate Profile (Fixed — use in every report)

- **Name:** Orhun Dalabasmaz
- **Experience:** 15+ years
- **Current Role:** Principal Engineer @ Atlassian, Munich
- **Target Roles:** Principal Engineer, Staff Engineer, SRE, Engineering Manager, Solutions Architect, Team Lead
- **Priority:** Compensation and role content over role type
- **Tech Stack:** Java/Kotlin, Python, Spring, AWS (Neptune, EKS, DynamoDB, SQS, Lambda, KMS, S3, CloudFormation), Docker, Kubernetes, Microservices, Kafka, Grafana, Splunk, Datadog, Neo4j, Gremlin/Cypher, CI/CD (Bitbucket/Jenkins), MCP, Claude/Gemini AI integrations
- **Expertise:** SRE, DevOps, Cloud Architecture (AWS-focused), Distributed Systems, People Management, Technical Leadership, Roadmap Planning, Cross-functional Collaboration, FedRAMP/SOC2/GDPR Compliance
- **Cloud Preference:** AWS (multi-cloud accepted)
- **Work Model:** Remote (anywhere) OR Hybrid/On-site (Munich only)
- **Employment:** Full-time
- **Salary:** Minimum €120,000 base + equity preferred
- **On-call:** Acceptable
- **Travel:** Low amount acceptable (conferences, office visits)
- **People Management:** Either (IC or EM)
- **Industry:** SaaS/Cloud preferred, open to other sectors
- **Language:** English must be the working language (German not required)

---

## Research Steps

When the user provides a company name or URL, search the following sources using web_search:

### Reviews & Ratings
1. **Glassdoor:** `[company name] Glassdoor reviews` → Overall rating, CEO approval, recommendation rate, pro/con reviews
2. **Kununu:** `[company name] Kununu Bewertungen` → German market employee reviews

### Compensation
3. **Levels.fyi:** `[company name] levels.fyi engineer salary Germany` → Salary data by level
4. **Comprehensive.io:** `[company name] site:app.comprehensive.io/benchmarking/postings` → Market salary ranges for target roles
5. **General benefits:** `[company name] employee benefits Germany equity RSU bonus` → Equity, bonus structure, perks

### Job Openings (search all — consolidate into one positions list)
6. **LinkedIn:** `[company name] Principal Engineer SRE Staff Engineer jobs Munich remote` → with job detail links
7. **Xing:** `[company name] Xing Stellenangebote Principal Engineer SRE` → with job detail links
8. **Indeed.de:** `[company name] indeed.de Principal Engineer SRE Munich` → with job detail links
9. **Monster.de:** `[company name] monster.de engineer jobs` → with job detail links
10. **Remotely.de:** `[company name] remotely.de engineer` → with job detail links
11. **Company careers page:** `[company name] careers jobs Principal Engineer SRE` → Direct listings

### Stability
12. **Layoffs.fyi:** `[company name] layoffs.fyi` → Recent layoff events, headcount reductions, dates

For missing data, write "data not available" — **never guess**.

---

## Report Format

Produce the following report in English for each company:

---

### 🏢 [COMPANY NAME]
**Industry:** | **Size:** | **HQ:** | **Work Model:**

#### ⚡ QUICK OVERVIEW
| Criteria | Value | Source |
|----------|-------|--------|
| Glassdoor Rating | X.X / 5 | Glassdoor |
| Kununu Rating | X.X / 5 | Kununu |
| CEO Approval Rate | XX% | Glassdoor |
| Recommendation Rate | XX% | Glassdoor/Kununu |
| Salary Competitiveness | ⭐⭐⭐⭐⭐ | Levels.fyi/Comprehensive.io |
| Recent Layoffs | ✅ None / ⚠️ [date + size] | Layoffs.fyi |

#### ⚠️ LAYOFF HISTORY
Summarize any layoff events found on Layoffs.fyi or in the news:
- **[Month Year]:** ~X% of workforce / ~X,XXX employees — [brief reason if known] — [Source](url)

If no layoffs found, write "No layoffs recorded on Layoffs.fyi."

#### 💰 SALARY & PACKAGE
- **Market Range (target roles):** €XXX,XXX – €XXX,XXX base (Comprehensive.io / Levels.fyi)
- **Reported Salaries at Company:** (cite source)
- **Equity:** RSU / ESOP / Stock options available?
- **Bonus:** Structure?
- **Benefits:** Health, pension, learning budget, equipment, etc.

#### ✅ PROS
(From employee reviews — most frequently mentioned)
- ...

#### ❌ CONS
(From employee reviews — most frequently mentioned)
- ...

#### 🎯 CANDIDATE FIT
| Criteria | Status | Notes |
|----------|--------|-------|
| Tech Stack (AWS, Java/Kotlin, K8s) | ✅/⚠️/❌ | |
| Target Role Available? | ✅/⚠️/❌ | |
| Work Model (Remote/Hybrid-Munich) | ✅/⚠️/❌ | |
| Salary 120k+ Base | ✅/⚠️/❌ | |
| Equity Available? | ✅/⚠️/❌ | |
| English-speaking Environment | ✅/⚠️/❌ | |
| Engineering/IC Culture | ✅/⚠️/❌ | |
| Company Stability | ✅/⚠️/❌ | |

#### 💼 OPEN POSITIONS
Consolidate relevant roles from all job sources (LinkedIn, Xing, Indeed.de, Monster.de, Remotely.de, careers page). Only include roles matching the candidate profile (Principal Engineer, Staff Engineer, SRE, EM, Solutions Architect, Team Lead). Skip unrelated roles.

| Title | Source | Location | Work Model | Salary (if listed) | Posted | Link |
|-------|--------|----------|------------|--------------------|--------|------|
| [Job Title] | LinkedIn / Xing / Indeed / Monster / Remotely / Careers | [City / Remote] | Remote / Hybrid / On-site | €XXX,XXX or N/A | [date or "recent"] | [Apply](url) |

If no relevant positions are found, write "No matching open positions found."

#### 🔗 SOURCES
- 🔍 Glassdoor: [link]
- 🔍 Kununu: [link]
- 💰 Levels.fyi: [link]
- 💰 Comprehensive.io: [link]
- 📉 Layoffs.fyi: [link]
- 💼 LinkedIn Jobs: [link]
- 💼 Xing Jobs: [link]
- 💼 Indeed.de: [link]
- 💼 Monster.de: [link]
- 🌐 Remotely.de: [link]
- 🏢 Careers Page: [link]

#### 🏁 OVERALL ASSESSMENT
**Decision:** 🟢 APPLY | 🟡 RESEARCH | 🔴 SKIP

**Rationale:** (2-3 sentence summary — why this decision? Factor in stability, salary fit, open roles, and culture.)

---

## Multiple Companies

If the user provides multiple companies, append a comparison table at the end of the reports:

### 📊 COMPARISON TABLE
| Company | Glassdoor | Salary Fit | Stack Fit | Model Fit | Stability | Decision |
|---------|-----------|------------|-----------|-----------|-----------|----------|
| ... | | | | | ✅/⚠️/❌ | 🟢/🟡/🔴 |

---

## Important Notes

- Report is written in **English**
- For missing data, write "data not available" — never guess
- Use €120k base as the salary comparison threshold
- Equity matters: flag with ⚠️ if RSU/ESOP is not available
- Remote: any city accepted; hybrid/on-site: Munich only
- Layoffs within the last 12 months: flag as ⚠️ in QUICK OVERVIEW and CANDIDATE FIT
