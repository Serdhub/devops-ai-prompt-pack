# Contributing

Thank you for your interest in contributing to the DevOps AI Prompt Pack.

This repository contains a **free sample** from a commercial product. Contributions are accepted in the form of:

- Bug reports (broken prompt structure, missing fields, technical inaccuracies)
- Suggestions for the free sample content
- Technical corrections to existing prompts

---

## What we accept

### Technical corrections

If a prompt contains a technically incorrect statement, command, or diagnostic step, open an issue with:

- The specific prompt title and section (e.g., "Sample 2 — DNS, Verification Steps, step 3")
- The current text that is incorrect
- The correct text with a brief explanation
- A reference (documentation, CVE, RFC, or your own reproducible test) supporting the correction

### Reproducibility requirement

All technical corrections must be reproducible. Claims that cannot be verified with a concrete command, metric, or documented behavior will not be accepted.

### Broken prompt structure

If a prompt is missing a required field (Scenario, Symptoms, Required Inputs, Prompt block, Expected Output, Verification Steps, Production Warning, or Senior Insight), open an issue describing which field is missing and on which prompt.

---

## What we do not accept

- New prompt submissions for the free sample (the sample selection is fixed)
- AI-generated corrections without human verification
- Contributions containing credentials, API keys, hostnames, IP addresses, or any real infrastructure configuration
- Claims that a prompt "does not work" without specific, reproducible reproduction steps

---

## How to report an issue

Use GitHub Issues. The title format should be:

```
[CORRECTION] Prompt title — Section — Brief description
```

Example:

```
[CORRECTION] DNS Resolution Failing — Verification Steps — conntrack metric name is incorrect
```

---

## No pull requests on the product file

The `FREE_SAMPLE.md` file is derived from the commercial product (`DEVOPS_AI_PROMPT_PACK_V1_FINAL.md`). Pull requests that modify prompt content without a corresponding verified issue will not be merged.

---

## Code of conduct

Technical discussions only. Be precise, be verifiable, be professional.

---

*Serdhub · LAB-001*
