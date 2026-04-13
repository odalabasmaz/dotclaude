---
name: job-evaluator
description: Given a company name or URL, produces a comprehensive job evaluation report based on Orhun Dalabasmaz's career profile and expectations. Searches Glassdoor, Kununu, Levels.fyi, LinkedIn, and Remotely.de. Use this skill when the user provides one or more company names or URLs, researches job listings, or uses phrases like "evaluate this company", "should I apply here", "what's the salary", "what are the employee reviews".
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

1. **Glassdoor:** `[company name] Glassdoor reviews` → Overall rating, CEO approval, recommendation rate, pro/con reviews
2. **Kununu:** `[company name] Kununu Bewertungen` → German market employee reviews
3. **Levels.fyi:** `[company name] levels.fyi engineer salary Germany` → Salary data
4. **LinkedIn:** `[company name] Principal Engineer SRE Staff Engineer jobs Munich remote` → Open positions with job detail links
5. **Remotely.de:** `[company name] remotely.de engineer` → Remote job listings with job detail links
6. **Company careers page:** `[company name] careers jobs Principal Engineer SRE` → Direct listings on their own site
7. **General:** `[company name] employee benefits Germany equity RSU` → Benefits

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
| Salary Competitiveness | ⭐⭐⭐⭐⭐ | Levels.fyi/Glassdoor |

#### 💰 SALARY & PACKAGE
- **Target Role Base Salary:** (cite source)
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

#### 💼 OPEN POSITIONS
List every relevant open position found. For each one:

| Title | Location | Work Model | Salary (if listed) | Posted | Link |
|-------|----------|------------|--------------------|--------|------|
| [Job Title] | [City / Remote] | Remote / Hybrid / On-site | €XXX,XXX or N/A | [date or "recent"] | [Apply](url) |

Only include roles relevant to the candidate profile (Principal Engineer, Staff Engineer, SRE, EM, Solutions Architect, Team Lead). Skip unrelated roles. If no relevant positions are found, write "No matching open positions found."

#### 🔗 SOURCES
- 🔍 Glassdoor: [link]
- 🔍 Kununu: [link]
- 💰 Levels.fyi: [link]
- 💼 LinkedIn Jobs: [link]
- 🌐 Remotely.de: [link]
- 🏢 Careers Page: [link]

#### 🏁 OVERALL ASSESSMENT
**Decision:** 🟢 APPLY | 🟡 RESEARCH | 🔴 SKIP

**Rationale:** (2-3 sentence summary — why this decision?)

---

## Multiple Companies

If the user provides multiple companies, append a comparison table at the end of the reports:

### 📊 COMPARISON TABLE
| Company | Glassdoor | Salary Fit | Stack Fit | Model Fit | Decision |
|---------|-----------|------------|-----------|-----------|----------|
| ... | | | | | 🟢/🟡/🔴 |

---

## Important Notes

- Report is written in **English**
- For missing data, write "data not available" — never guess
- Use €120k base as the salary comparison threshold
- Equity matters: flag with ⚠️ if RSU/ESOP is not available
- Remote: any city accepted; hybrid/on-site: Munich only
