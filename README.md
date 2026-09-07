# Bhumix

**Land Record Digitization and Validation System** - built for Internal Smart India Hackathon 2026 (PS ID: SIH26018, Theme: Smart Automation, Category: Software).

Bhumix digitizes scanned/handwritten land records using a Vision-Language Model, translates fields into regional languages, and routes them through an official verification workflow before committing them as version-controlled, audit-logged records. Scoped initially to **Delhi's land record system** (governed by the Delhi Land Reforms Act, 1954 / Delhi Land Revenue Act, using Khatauni-format records).

## The Problem

Land records in Delhi (and much of India) are still largely paper-based and inconsistently digitized. Manual entry is slow and error-prone, rigid OCR fails on damaged or handwritten documents, and there's no standard way to serve the same record in multiple regional languages. This causes delayed mutations, ownership disputes, and friction when citizens try to use land records as collateral for loans.

## Solution

1. **Gemini VLM extraction** — reads scanned documents (including handwritten/damaged ones) and returns structured JSON, outperforming rule-based OCR.
2. **Sarvam AI translation** — converts extracted fields into a regional language on demand, so a record is captured once and served in any language.
3. **Human-in-the-loop review** — a revenue official verifies extracted data before it's committed.
4. **Version-controlled storage** — approved records are saved to Supabase with an append-only audit log; nothing is overwritten, only superseded.

## Workflow

```
Upload (scanned image/PDF)
      ↓
Gemini VLM → extracts fields as JSON
      ↓
Sarvam AI → translates to regional language
      ↓
Official review → verifies and approves
      ↓
Supabase → stores record + audit log entry
      ↓
Dashboard / API → serves latest verified records
```

Corrections after approval don't overwrite history — a new version is saved, the prior approved version is retained, and the audit log records who changed what and why.

## Tech Stack

| Component | Technology |
|---|---|
| Document upload | Web form |
| Field extraction | Gemini VLM (Hugging Face endpoint) |
| Regional translation | Sarvam AI (Hugging Face endpoint) |
| Database & file storage | Supabase (Postgres + Storage) |
| Audit & version control | Supabase append-only `audit_log` |
| Dashboard / API | Supabase-backed REST API |
| Access control | Role-based auth — Patwari / Tehsildar / SDM / Admin |

## Roles & Permissions

Modeled on Delhi's revenue department hierarchy, simplified to three field roles for this MVP

| Action | Patwari | Tehsildar | SDM | Admin |
|---|:---:|:---:|:---:|:---:|
| Upload document | ✓ | ✗ | ✗ | ✓ |
| View own uploads | ✓ | ✓ | ✓ | ✓ |
| View all district records | ✗ | ✓ | ✓ | ✓ |
| Edit fields before approval | ✗ | ✓ | ✗ | ✓ |
| Approve / reject record | ✗ | ✗ | ✓ | ✓ |
| Edit already-approved record | ✗ | ✓ | ✗ | ✓ |
| Reassign record | ✗ | ✓ | ✓ | ✓ |
| Manage accounts / roles | ✗ | ✗ | ✗ | ✓ |
| View audit log | ✗ | ✓ | ✓ | ✓ |

*"Admin" is a system-only role (app superuser), not a real revenue department post — worth being explicit about that distinction if asked.*

## Security Model

- Post-approval edit rights limited to Tehsildar and Admin only.
- Versioning, not overwriting — the original approved record is never lost.
- Every correction is logged in the append-only audit log, with no invisible-edit path for any role.

## Notes on the Domain Model

Delhi's actual revenue hierarchy has more layers than this MVP models:

```
District Magistrate (DM) → SDM → Tehsildar → Naib Tehsildar → Revenue Inspector/Kanungo → Patwari
```

We've simplified this to a 3-role chain (**Patwari → Tehsildar → SDM**) for the hackathon build. Naib Tehsildar and Revenue Inspector/Kanungo responsibilities are folded into the Tehsildar review step for now — a natural next step would be to reintroduce them as intermediate approval tiers if this were extended toward production. Land record terminology used here (Khatauni, mutation) follows the Delhi Land Reforms Act, 1954; other states use different formats (e.g. Jamabandi in Punjab, 7/12 extract in Maharashtra) and would need separate schema handling.

## Impact

- **Citizens** — faster mutation, ownership proof, easier loan access
- **Patwaris & Tehsildars** — less manual entry, faster verification
- **Government** — supports DILRMP goals, cuts dispute backlog

## Feasibility

- Built on existing, proven APIs (Gemini, Sarvam AI, Supabase) - no model training from scratch
- Low-cost pilot via Hugging Face-hosted endpoints
- Modular design - extraction, translation, storage, and review stages can scale or be swapped independently

## References

- [Digital India Land Records Modernization Programme (DILRMP)](https://dilrmp.gov.in/)
- SIH2026 Portal - PS ID SIH26018, Ministry of Rural Development
- [Google Gemini API](https://ai.google.dev/)
- [Sarvam AI](https://www.sarvam.ai/)
- [Delhi land records service reference](https://dmsouth.delhi.gov.in/services-offered/)
- [Supabase](https://supabase.com/)

## Team

**Bhumix** - Internal Smart India Hackathon 2026

**Members** - Soumyadip Debnath (Team Lead), Sunil Kumar Swami, Varun Bhasin, Srija Das, Shreya Bansal, Shreyansh Bhatnagar. 

