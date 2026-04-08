# 📖 Playbook Builder

**Trigger an incident. Get a complete IR playbook.**

→ **[Live tool](https://dgiry.github.io/playbook-builder)**

---

## What it does

Given a MITRE technique, incident description, or SIGMA rule — generate a complete 6-phase incident response playbook with actionable steps, runnable commands, forensic evidence checklist, escalation criteria, and tool recommendations.

| Input | Example |
|-------|---------|
| **MITRE Technique** | T1059.001, T1003.001, T1486, or technique name |
| **Incident Description** | "Ransomware alert on file server, 4000 files encrypted" |
| **SIGMA Rule** | Paste any SIGMA YAML — playbook built around the detection logic |

| Output | Description |
|--------|-------------|
| **6 IR Phases** | Detection · Triage · Containment · Eradication · Recovery · Post-Incident |
| **Actionable Steps** | 4-6 concrete steps per phase with real tool commands |
| **Runnable Commands** | SPL / KQL / Bash / PowerShell / Python per step |
| **Evidence Checklist** | Specific logs, artifacts, forensic items to preserve |
| **Escalation Criteria** | When to escalate and to whom, per phase |
| **MITRE ATT&CK** | Min. 2 techniques mapped with tactic and clickable links |
| **Recommended Tools** | Real tools (Volatility, KAPE, Velociraptor…) per phase |
| **Executive Summary** | 3-4 sentences for CISO/management audience |
| **Post-Mortem Questions** | 3-5 specific lessons-learned prompts as a checklist |
| **⬇ Download .md** | Full playbook as Markdown — ready for Notion / Confluence / Jira |
| **⬇ Download .json** | Raw structured JSON for integration |

---

## Options

| Option | Values |
|--------|--------|
| **Environment** | Corporate Windows / Hybrid / Cloud / Linux / OT-ICS |
| **Focus** | Full playbook / Containment-focused / Recovery-focused |
| **Model** | gpt-4o-mini (fast) / gpt-4o (best) |

---

## Usage

```
https://dgiry.github.io/playbook-builder
```

Enter a trigger → click **Generate Playbook** → download .md or copy per phase.

Requires an OpenAI API key. Stored in `localStorage` only — never transmitted except directly to OpenAI. Key shared with the full pipeline (`cv_oai_key`).

Deep-link support: `?technique=T1059.001` · `?incident=description` · `?sigma=yaml`

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
- **[✍️ SIGMA Generator](https://dgiry.github.io/sigma-generator)** — describe an attack, get a detection rule

---

## Deploy your own

Static HTML — works on GitHub Pages, Netlify, Vercel, or any web server.

```bash
git clone https://github.com/dgiry/playbook-builder
# open index.html in your browser
```

## License

MIT
