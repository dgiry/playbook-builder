# 📖 Playbook Builder

**Structured IR response drafts for security teams.**

→ **[Live tool](https://dgiry.github.io/playbook-builder)**

> A planning aid — not a replacement for analyst judgment. Review and adapt the output to your environment before executing.

---

## What it does

Given a MITRE technique, incident description, or SIGMA rule — generate a structured 6-phase IR playbook draft as a starting point for your response. Each phase includes concrete steps, runnable commands, evidence checklist, and specific escalation criteria.

| Input | Example |
|-------|---------|
| **MITRE Technique** | T1059.001, T1003.001, T1486, or technique name |
| **Incident Description** | "Ransomware alert on FS-03, 4000 files encrypted, user: svc_backup" |
| **SIGMA Rule** | Paste any SIGMA YAML — playbook built around the exact detection logic |

| Output | Description |
|--------|-------------|
| **6 IR Phases** | Detection · Triage · Containment · Eradication · Recovery · Post-Incident |
| **Actionable Steps** | 4-6 concrete steps per phase — no vague "investigate further" steps |
| **Runnable Commands** | SPL / KQL / Bash / PowerShell / Python — real syntax, labeled placeholders |
| **Evidence Checklist** | Specific logs, artifacts, and forensic items to preserve per phase |
| **Escalation Criteria** | Specific observable conditions (not generic thresholds) + escalation target |
| **MITRE ATT&CK** | Min. 2 techniques with tactic, sub-techniques, and clickable links |
| **Recommended Tools** | Real tools (Volatility, KAPE, Velociraptor…) tagged by phase |
| **Executive Summary** | Structured 4-point format: what / who / done / status — CISO-ready |
| **Post-Mortem Questions** | 3-5 incident-specific lessons-learned prompts as a checklist |
| **⬇ Download .md** | Full playbook as Markdown — ready for Notion / Confluence / Jira |
| **⬇ Download .json** | Raw structured JSON for integration |

---

## Options

| Option | Values | Notes |
|--------|--------|-------|
| **Environment** | Corporate Windows / Hybrid / Cloud / Linux / OT-ICS | Adapts tools, commands, and log sources per env |
| **Focus** | Full playbook / Containment-focused / Recovery-focused | Weights phase depth accordingly |
| **Model** | gpt-4o-mini (fast) / gpt-4o (best) | |

**OT/ICS note:** when this environment is selected, every containment and eradication command is prefixed with `⚠️ Validate with OT engineering before execution` to prevent production impact.

---

## Usage

```
https://dgiry.github.io/playbook-builder
```

Enter a trigger → click **Generate Playbook** → review, adapt, download .md or copy per phase.

Requires an OpenAI API key. Stored in `localStorage` only — never transmitted except directly to OpenAI. Key shared with the full pipeline (`cv_oai_key`).

### Deep-link support

Pre-fills the correct tab from any external link:

```
?technique=T1059.001
?incident=Ransomware+alert+on+FS-03...
?sigma=<url-encoded SIGMA YAML>
```

**sigma-generator integration:** after generating a SIGMA rule, the "Build Playbook →" bridge in sigma-generator automatically links here with `?sigma=` pre-filled from the generated rule.

---

## Part of the SecOps pipeline

```
🧪 defang-ioc  →  🔭 ioc-pivot  →  🔬 cve-enricher  →  🚨 alert-explainer  →  ✍️ sigma-generator  →  📖 playbook-builder
   (extract)        (pivot)           (enrich)              (explain)               (detect)                (respond)
```

- **[🧪 Defang IOC](https://dgiry.github.io/defang-ioc)** — extract and defang/refang IOCs
- **[🔭 IOC Pivot Hub](https://dgiry.github.io/ioc-pivot)** — pivot any IOC to 30+ threat intel platforms
- **[🔬 CVE Enricher](https://dgiry.github.io/cve-enricher)** — full CVE context: KEV, threat actors, patch priority
- **[🚨 Alert Explainer](https://dgiry.github.io/alert-explainer)** — understand any SIEM alert in plain English
- **[✍️ SIGMA Generator](https://dgiry.github.io/sigma-generator)** — describe an attack, get a detection rule → bridges directly to playbook-builder

---

## Deploy your own

Static HTML — works on GitHub Pages, Netlify, Vercel, or any web server.

```bash
git clone https://github.com/dgiry/playbook-builder
# open index.html in your browser
```

## License

MIT
