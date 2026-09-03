# How to publish your cost calculator — no technical knowledge needed

You have two files to put online: `index.html` (the calculator website) and
`LLM_Hosting_Cost_Calculator.xlsx` (the Excel workbook people can download from it).
GitHub will host both for free. Every step below happens in your web browser —
you will not type any commands.

Total time: about 10 minutes.

## Step 1 — Create a GitHub account (skip if you have one)

1. Go to **github.com** and click **Sign up**.
2. Use your work email, pick a username (it becomes part of your website address,
   so something professional like `janedoe-acme` is better than `xxgamer99xx`).
3. Choose the **Free** plan.

## Step 2 — Create a place for the files (a "repository")

1. Once signed in, click the **+** in the top-right corner, then **New repository**.
2. Repository name: `llm-cost-calculator` (lowercase, exactly like that).
3. Leave it set to **Public** — this is what makes the free website work.
4. Tick **"Add a README file"**.
5. Click the green **Create repository** button.

## Step 3 — Upload the two files

1. On the page that opens, click **Add file** (near the green Code button), then
   **Upload files**.
2. Drag **`index.html`** and **`LLM_Hosting_Cost_Calculator.xlsx`** from the
   package folder into the upload box. Also drag **`README.md`** and say yes when
   it asks to replace the existing one.
3. Click the green **Commit changes** button at the bottom.

> The file names matter. If your computer renamed the download to `index (1).html`,
> rename it back to exactly `index.html` before uploading.

## Step 4 — Turn on the website

1. In your repository, click **Settings** (the tab with the gear icon).
2. In the left-hand menu, click **Pages**.
3. Under **Build and deployment → Source**, choose **Deploy from a branch**.
4. Under **Branch**, choose **main** and folder **/ (root)**, then click **Save**.

## Step 5 — Visit your calculator

Wait one to two minutes, then go to:

```
https://YOUR-USERNAME.github.io/llm-cost-calculator/
```

(Replace YOUR-USERNAME with the username you chose. The Pages settings screen also
shows this address once it's live — refresh the page if you don't see it yet.)

That address is safe to share with anyone: colleagues, your boss, a board deck.
Every estimate someone builds gets its own web address too, so you can send a
specific scenario by just copying the address bar.

## If something doesn't work

- **404 / page not found** — wait two more minutes and refresh; the first build is slow.
  Still nothing? Re-check Step 4, and confirm the file is called exactly `index.html`.
- **The page loads but the Excel download doesn't** — check the workbook uploaded with
  its exact name: `LLM_Hosting_Cost_Calculator.xlsx`.
- **You want to take it down** — Settings → Pages → change Source to **None**. Or delete
  the whole repository under Settings → General → Danger Zone.

## About the prices going stale

Every price in the calculator was verified in **August 2026**. Prices in this market
move often. The website knows its own age: after about six weeks it automatically
shows a yellow warning banner telling visitors the numbers may be out of date, so it
will never quietly pretend to be current.

When you want the numbers refreshed, hand the file `DEVELOPER_BRIEF.md` (in this
package) to whoever helps you with technical work — it tells them exactly how to
rebuild the site with live weekly price updates from Google's and Microsoft's own
pricing systems. Until then, the calculator is honest about its vintage, and the
relationships it teaches (idle time dominates, people cost more than GPUs for small
models, pay-per-token wins at low volume) stay true even as individual prices drift.
