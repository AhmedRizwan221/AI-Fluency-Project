# Delegation Plan: Product Review & Rating System

**Course:** AI Fluency — Delegation Planning Exercise
**Date:** July 2026

---

## Project Vision

A Product Review & Rating System added to an existing, running MERN e-commerce app.

**Stack context:**
- Backend: Node/Express/MongoDB (Mongoose), JWT auth already in place (`protect`/`isAdmin` middleware exists)
- Frontend: React + Redux Toolkit + Tailwind CSS
- Deployment target (future): MongoDB Atlas + Render/Railway/Heroku

**Core business rules:**
- Only **verified purchasers** (user has an order with status **delivered/completed** containing the product) can review it
- A review = **star rating (1–5) + written text**
- Users can **edit/delete their own reviews** anytime, no time limit
- **No moderation** — reviews publish instantly
- **Admins can delete any review** (no other special admin powers)

**Rating display:**
- Overall product rating = **simple average**, recalculated on every add/edit/delete, cached on the `Product` document
- Reviews sorted **highest-rated first** by default
- **Pagination** for review lists (no star-filtering)

**Definition of done:** Working code (backend + frontend) + automated tests + documentation (README + Postman collection). Deployment is a separate, deferred effort.

---

## Task Breakdown & Delegation Decisions

| # | Task | Delegation Decision | Rationale |
|---|------|---------------------|-----------|
| 1 | **Data modeling** — Review schema + Product/User/Order relationships | User shares existing model files → **AI drafts** schema | Only the user has ground truth on existing schema shapes; AI is strong at schema mechanics once given real context |
| 2 | **Verified-purchase logic** — query Order history, define eligibility | **User owns fully** | Core business logic with real ambiguity (what counts as "verified"); user has context AI lacks and wants ownership of this judgment call |
| 3 | **CRUD API endpoints** — create/read/update/delete review routes | **AI drafts all 4 endpoints**, user reviews against their conventions | Boilerplate-heavy once business rules are settled; good speed win for AI, user acts as quality gate |
| 4 | **Rating aggregation** — recalculate average, cache on Product | **User drafts**, AI reviews the aggregation query logic | Architecture choice made: explicit function called from controllers (not Mongoose hooks) for traceability; good hands-on skill to build |
| 5 | **Authorization (ownership check)** — edit/delete gated to review owner | **User writes it himself** (one-liner); reuses existing `protect`/`isAdmin` | Trivial once existing auth middleware is reused — no need to delegate |
| 6 | **Backend testing** (Jest/Supertest) | **AI drafts test cases + implementation**, user reviews for gaps | Efficient starting point, but user must verify tests actually cover real edge cases (delivered-vs-pending, ownership, admin delete) — not just generic CRUD happy paths |
| 7 | **Frontend Redux slice** — state + async thunks for reviews | **User writes it himself** | Core transferable skill (Redux Toolkit patterns) worth building muscle memory for |
| 8 | **Frontend UI components** — review form, list, star-rating display | **AI drafts star-rating component only**; user builds the rest | Star selector has real interaction-logic complexity (hover states, accessibility); layout/composition work is more straightforward and worth owning |
| 9 | **Documentation** — README + Postman collection | **AI drafts README**; user builds Postman collection himself | Building the Postman collection forces manual re-verification of every endpoint |
| 10 | **Deployment prep** — env vars, CORS, first deployment | **Deferred / out of scope for now** | This is the app's *first* deployment, not just a feature addition — bigger scope than expected, worth its own focused session later |

---

## Patterns Observed

A few recurring themes worth noting for future delegation decisions:

- **"AI drafts, human reviews"** worked well for boilerplate-heavy, low-ambiguity work once rules were defined (Tasks 1, 3, 6, 9).
- **"Human owns fully"** was chosen for anything with real business-logic ambiguity or clear learning value (Tasks 2, 4, 5, 7).
- **Scope correction happened live**: Task 10 was assumed to be light verification but turned out to be a full first deployment — a reminder to verify assumptions about "how big is this really" before assigning it.
- **Context-sharing was a prerequisite**, not an afterthought: several tasks (1, 3, 7) explicitly required the user to hand over existing code/schemas before AI could contribute usefully.

---

## Next Steps

1. Share existing `Product`, `User`, `Order` model files → begin Task 1
2. Work through tasks roughly in the order above (natural dependency chain: schema → business logic → API → aggregation → auth → tests → frontend)
3. Revisit deployment (Task 10) as a separate planning session once core feature is complete
