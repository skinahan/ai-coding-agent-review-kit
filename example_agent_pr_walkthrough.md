# Worked Example: Reviewing an Agent-Generated PR

**New here?** Start here before the [15-minute checklist](15_minute_ai_pr_review_checklist.md).

This is a synthetic teaching PR — a small scenario crafted to show what a fast second pass catches. The code is plausible enough to merge on a skim. That is the point.

---

## Setup

Last month I asked Cursor to add a `PATCH /api/widgets/:id` endpoint so internal admins could update widget labels. Small task: validate input, update one row, return JSON.

The PR landed Friday afternoon. Diff looked tidy. Tests were green. I almost merged it before dinner.

I ran the checklist instead. Took about twelve minutes. Glad I did.

**Agent task (what I asked for):**

> Add PATCH /api/widgets/:id for admin users. Body: `{ "label": string }`. Return updated widget. 404 if missing.

**Files changed:** `routes/widgets.ts`, `middleware/auth.ts`, `tests/widgets.test.ts`, `config/defaults.ts`

---

## The diff (abbreviated)

### `routes/widgets.ts` — looks fine at first glance

```typescript
export async function patchWidget(req: Request, res: Response) {
  const { id } = req.params;
  const { label } = req.body;

  if (!label || label.trim() === "") {
    return res.status(400).json({ error: "label required" });
  }

  const widget = await db.widgets.update(id, { label: label.trim() });
  if (!widget) return res.status(404).json({ error: "not found" });
  return res.json(widget);
}
```

Happy path works. Empty string rejected. Reads clean.

### `middleware/auth.ts` — scope drift hiding in a "refactor"

```typescript
// Before: adminOnly checked role === 'admin'
// After: requireRole(['admin', 'editor'])  // allow editors to patch widgets too
```

The agent comment said: *"Refactored auth helper for flexibility."* I did not ask for editors. I asked for **admin only**.

### `tests/widgets.test.ts` — green but hollow

```typescript
it("updates widget label", async () => {
  const res = await request(app)
    .patch("/api/widgets/w1")
    .send({ label: "New Label" });
  expect(res.status).toBe(200);
  expect(res.body.label).toBe("New Label");
});
```

No auth header. No 404 case. No empty-label case despite the handler supporting it.

### `config/defaults.ts` — unrelated

```typescript
export const WIDGET_CACHE_TTL_SECONDS = 300; // was 60
```

Nothing in my prompt mentioned caching.

---

## Checklist pass

### 0. Context check (60 seconds)

| Question | Answer |
| --- | --- |
| What was the agent asked? | Admin-only PATCH for widget label |
| Which files changed? | Route, auth middleware, tests, config |
| What breaks if misunderstood? | Wrong roles can edit; cache side effects; untested edge cases slip to prod |

**Verdict:** Proceed, but scope already looks wider than requested.

### 1. Scope drift (2 min) — **FLAG**

- `auth.ts`: editors now allowed — **not in spec**
- `config/defaults.ts`: cache TTL change — **unrelated**
- Route file alone would have been enough

**Would have missed without this step:** Authorization widening is the highest-risk change in the PR. It is not in the route file I skimmed first.

### 2. Behavior and edge cases (3 min) — **FLAG**

- Handler rejects empty `label` — good
- No check for non-admin callers in the route (relies entirely on middleware — OK if middleware is correct, but we already know middleware changed)
- Missing: malformed JSON, `label` over max length, concurrent updates

**Happy-path only** on the main path; edge cases untested.

### 3. Security and data risk (3 min) — **FLAG**

- Widening `admin` → `admin + editor` is a **permission change** without a test proving editors are blocked today and allowed tomorrow intentionally
- No audit trail mentioned for label changes

### 4. Test reality (2 min) — **FLAG**

- Single test hits the endpoint **without auth** — would pass even if auth middleware were removed entirely (depending on test setup)
- No test would fail if empty-label validation were deleted — not covered
- Classic **performative test**: asserts the happy path the agent wrote, not the risks

### 5. System fit (2 min) — **minor flag**

- Cache TTL change might affect other widget reads — no mention in PR description
- No migration needed for label field — OK

### 6. Agent self-review (2 min)

Pasted the checklist prompt. Agent returned:

> 1. Editor role may be unintended scope expansion.  
> 2. Tests do not cover auth or 404.  
> 3. Config change not tied to request.

Agent found what the diff hid. I had not asked it to defend the auth change.

---

## Outcome

**Decision: Revise — do not merge.**

Smallest fixes that would reduce risk:

1. Revert `auth.ts` to admin-only unless I explicitly want editors (separate decision).
2. Revert unrelated `config/defaults.ts` change or split to another PR.
3. Add tests: 401 without auth, 403 for non-admin, 400 empty label, 404 missing id.

If I had merged on green tests and a clean-looking route file, I would have shipped a permission widening and a cache change I never asked for.

---

## Use the full checklist

This walkthrough is one scenario. The [15-minute checklist](15_minute_ai_pr_review_checklist.md) is the reusable pass:

- [Read the checklist (Markdown)](15_minute_ai_pr_review_checklist.md)
- [Read the checklist (HTML)](15_minute_ai_pr_review_checklist.html)

**Feedback:** If you review agent PRs regularly, [tell us what this missed](https://github.com/skinahan/ai-coding-agent-review-kit/issues/new?template=ai-review-kit-interest.yml).

---

*Synthetic scenario for teaching. Replace with your own anonymized PR when you have one — the structure stays the same.*
