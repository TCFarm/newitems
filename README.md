# TC Farm — weekly new-item review

The page a purchasing reviewer uses to approve the week's new-item recommendations.
Live at **https://tcfarm.github.io/newitems/**

⛔ **THIS REPO HOLDS PAGE CODE ONLY.** No product data, no costs, no margins and no
credentials are committed here. Everything the page shows is fetched from the
**Claude Agents SharePoint site** *after* a Microsoft sign-in, so a public URL is
harmless — the data sits behind the tenant login, which is the requirement Jack set
(2026-09-03): *"there is a credential someone has to log in to so that a random
member of the public can't guess the URL and be able to see the data."*

The `clientId` and `tenantId` in `index.html` are **not secrets** — they identify
the app, they authorise nothing on their own, and this is a PKCE single-page app
with no client secret. The data is protected by the SharePoint ACL.

## Where everything lives

| Thing | Where |
|---|---|
| Owner project | `Claude Files/Procurement & Merchandising/` |
| Spec + build log | that project's `PROPOSAL-2026-09-02-newitem-weekly-app.md` |
| Weekly build | `newitem_week.py` — writes the payload and posts the Teams card |
| Payload | `payload_<order_sunday>.json` in the Claude Agents site's `Documents/NewItemReview/` |
| Decisions | SharePoint list **`NewItemReview`** on the same site |
| Images | `Documents/NewItemReview/images/` (RIVIR pack shots, uploaded per run) |
| The drain | `newitem_drain.py` — approvals become ProductPipeline input + order-queue lines |

## Deploying an update

The page is a single file. Push `index.html` to `main`; Pages rebuilds in ~50 s.
A fine-grained token with **Contents: read/write** is enough to push — it is *not*
enough to create the repo or enable Pages (both were done by hand, once).

## ⛔ One-time setup, recorded so nobody re-derives it

1. **Repo** — created by hand, public (Pages on a private repo needs a paid plan).
2. **Pages** — Settings → Pages → Deploy from a branch → `main` / `/ (root)`.
   ⚠️ GitHub refuses to enable Pages on an empty repo: push content first.
3. **Entra redirect URIs** — on the app *TC Farm Receiving*
   (`4a5939ef-bb89-4629-8a95-007509bb11a5`), under the **Single-page application**
   platform, add BOTH:
   * `https://tcfarm.github.io/newitems/`
   * `https://tcfarm.github.io/newitems/index.html`
   ⛔ It must be the SPA platform, not Web — Web expects a client secret and will
   reject a PKCE exchange. ⛔ Entra matches the redirect string EXACTLY, and the
   page sends `location.origin + location.pathname`, so a visitor who types
   `.../index.html` sends a different string than one who lands on `.../`. Both
   entries cover both. A mismatch fails as `AADSTS50011`.
   ✅ Verified live 2026-09-03.
4. **localhost** — `http://localhost:8000` is already registered, so the page can
   be tested locally with `python3 -m http.server 8000` from this directory.

## What the page does NOT do

⛔ It orders nothing and creates no product. Submitting records a **decision**. The
drain turns approvals into a ProductPipeline input CSV (its own review gates still
apply), a Finale import a human still runs, and a scheduled-order-queue line the
Sunday build picks up. A human still approves the order sheet itself.
