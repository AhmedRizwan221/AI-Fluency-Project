# Collaboration Framework: Product Review & Rating System

**Purpose:** Define Product, Process, and Performance expectations before execution begins, so requests to Claude are precise and outputs are consistently useful.

---

## Part 1: General Rules (apply to every task)

### Product Description — What I should ask for

| Element | Default expectation |
|---|---|
| **Format** | Code in artifacts/files (not inline snippets) for anything >20 lines; plain explanation in chat for reasoning/tradeoffs |
| **Scope** | One task at a time, matching the 10-task breakdown from the plan — never "build the whole feature" in one request |
| **Context provided** | I will paste/attach relevant existing code (models, slices, components) *before* asking for a draft — no output should be requested "blind" |
| **Level of detail** | Production-quality: error handling, comments explaining non-obvious logic, consistent with existing code style — not a toy/simplified version |
| **What "done" looks like per task** | Explicitly stated at the start of each task (e.g., "done = 4 working endpoints matching my existing controller pattern, with error handling") |

### Process Description — How Claude should approach each task

1. **Check assumptions first.** Before generating code, state what you're assuming about my existing code/conventions if I haven't shown you directly, and ask if unclear rather than guessing silently.
2. **Reference the agreed decisions.** Every task should be built consistent with the vision doc and delegation plan (e.g., don't add moderation logic, don't reintroduce photo uploads — we decided against these).
3. **Explain before/alongside code, not after.** Brief reasoning for structural choices (e.g., why a compound index, why this query shape) should accompany the output, not be a separate afterthought only if I ask.
4. **Flag deviations.** If something in my request conflicts with an earlier decision (e.g., I accidentally ask for moderation logic we ruled out), say so before proceeding.
5. **Validate against edge cases relevant to *this* project** — not generic ones. E.g., for review CRUD: non-buyer attempting review, non-owner attempting edit, admin delete. Don't pad with irrelevant generic edge cases.

### Performance Description — How we collaborate

- **Ask before generating** whenever a task has more than one reasonable approach and I haven't specified which (e.g., "should errors return 403 or 404 for unauthorized edit attempts?"). Don't ask when the answer was already settled in the vision/delegation plan.
- **Challenge my decisions when there's a real tradeoff**, even if I don't ask for critique — but don't relitigate decisions I've already made deliberately (e.g., don't re-argue "explicit function vs. Mongoose hooks" — that's settled).
- **Always explain tradeoffs** for anything with more than one defensible answer, even briefly — I want to understand *why*, not just get an answer.
- **Match the delegation split we already agreed per task** (see Part 2) — don't offer to write code for tasks I said I want to own, and don't hold back drafts for tasks I delegated to you.
- **Be direct about limitations** — e.g., if a request depends on details of my codebase you haven't seen, say so rather than producing a plausible-looking generic guess.

---

## Part 2: Task-Specific Collaboration Specs

Each task carries a different delegation split (from the delegation plan), which changes what Product/Process/Performance should look like *for that task specifically*.

### Task 1 — Data Modeling (AI drafts, from user-supplied schemas)
- **Product:** A complete Mongoose schema file (`Review.js`) + a short diff/note on any changes needed to `Product.js` (e.g., adding cached rating fields)
- **Process:** Wait for existing model files before drafting anything. State any assumptions about field names explicitly.
- **Performance:** Ask clarifying questions about relationships (e.g., "does your Order model store items as an array of product IDs or objects?") before drafting, rather than guessing.

### Task 2 — Verified-Purchase Logic (user owns fully)
- **Product:** N/A for drafting — if asked, only a **review/critique** of what the user writes, not a rewrite
- **Process:** When reviewing, check specifically: does the query correctly filter by "delivered/completed" status? Are edge cases handled (user with multiple orders, one delivered one not)?
- **Performance:** Don't offer alternative implementations unprompted — this is user-owned. Only flag actual bugs or logic gaps if asked to review.

### Task 3 — CRUD API Endpoints (AI drafts all 4, user reviews)
- **Product:** All 4 endpoints (create/read/update/delete) in one file/set, matching existing controller/route file structure, with inline comments on auth/validation logic
- **Process:** Build on top of Task 1 (schema) and reuse Task 2's verified-purchase check as-is (don't modify it) and existing `protect`/`isAdmin` middleware
- **Performance:** Explain response shape/status code choices briefly; flag if anything doesn't match typical REST conventions used elsewhere in the app (if shown)

### Task 4 — Rating Aggregation (user drafts, AI reviews query logic)
- **Product:** Only a review — a plain-language critique of the user's aggregation function, not a rewritten version, unless a bug is found
- **Process:** Check correctness (does it recompute correctly after edit/delete, not just create?) and check it's called consistently across all relevant controllers
- **Performance:** Point out the "must remember to call this everywhere" risk if it's not handled (this is the known tradeoff of the explicit-call architecture chosen)

### Task 5 — Ownership Check (user writes, no AI involvement expected)
- **Product/Process/Performance:** No default AI involvement. Only engage if explicitly asked to review the one-liner.

### Task 6 — Backend Testing (AI drafts, user reviews for gaps)
- **Product:** A test file (Jest/Supertest) covering: successful review creation by verified buyer, rejection for non-buyer, rejection for pending-order buyer, ownership-based edit/delete, admin delete override, pagination correctness, rating recalculation after edit/delete
- **Process:** Build tests against the *actual* business rules from the vision doc, not generic CRUD tests
- **Performance:** Explicitly call out which edge cases are covered vs. not, so the user's review has a clear checklist rather than needing to reverse-engineer test coverage

### Task 7 — Redux Slice (user writes, for learning)
- **Product/Process/Performance:** No default drafting. If the user asks for help, prioritize explaining Redux Toolkit patterns/concepts over providing full solutions, unless explicitly asked for a working example.

### Task 8 — UI Components (AI drafts only star-rating component)
- **Product:** One self-contained React component (star display + interactive selector variant), Tailwind-styled, with props documented (e.g., `value`, `onChange`, `readOnly`)
- **Process:** Consider accessibility (keyboard input, ARIA) and both read-only (displaying average) and interactive (submitting a rating) use cases
- **Performance:** Note styling assumptions (since I haven't seen the user's exact Tailwind config/theme) and flag where adjustment will likely be needed

### Task 9 — Documentation (AI drafts README, user builds Postman collection)
- **Product:** A README section (Markdown) covering: feature overview, verified-purchase rule, rating calculation, list of endpoints with auth requirements (not full request/response schemas, since that's what the Postman collection is for)
- **Process:** Base the README on the *actual final code*, not the plan — ask the user to share final route/controller files first if not already visible
- **Performance:** Flag if documentation would need updating due to any deviation from the original plan during implementation

### Task 10 — Deployment (deferred)
- Not in scope for this execution phase. Revisit as its own planning conversation later.

---

## Quick-Reference Summary

| Task | Who drafts | Who reviews | Key Performance rule |
|---|---|---|---|
| 1. Data modeling | AI | User | Wait for real schemas first |
| 2. Verified-purchase logic | User | AI (if asked) | Don't rewrite, only critique |
| 3. CRUD endpoints | AI | User | Explain status/response choices |
| 4. Rating aggregation | User | AI | Flag "must call everywhere" risk |
| 5. Ownership check | User | — | Hands-off by default |
| 6. Testing | AI | User | List coverage explicitly |
| 7. Redux slice | User | — | Teach concepts, don't just solve |
| 8. Star-rating component | AI | User | Flag styling assumptions |
| 9. Documentation | AI (README) / User (Postman) | Both | Base on final code, not plan |
| 10. Deployment | Deferred | Deferred | Out of scope for now |
