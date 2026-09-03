# Putting your calculator online with GitHub — for complete beginners

You have never used GitHub. That's fine. This guide assumes nothing.

**What GitHub is:** a website where people store files. If you put a file called
`index.html` in a public folder there, GitHub will also serve it as a free website.
That's all we're using it for.

**What you need before you start:**
- The file `github_upload_package.zip`, unzipped, so you can see the files inside it
- An email address
- About 15 minutes

**The words you'll see, translated:**
- *Repository* (or "repo") = a folder on GitHub
- *Commit* = save
- *Branch: main* = the normal version of your folder (ignore the concept, just pick "main" when asked)
- *GitHub Pages* = the free website feature

---

## Part 1 — Make an account  (3 minutes)

1. Go to **github.com** in your browser.
2. Click **Sign up** (top right).
3. Enter your email, create a password, and choose a username.
   - Your username becomes part of your website's address, so pick something you'd
     be happy to put on a slide. `maria-lopez` is good. `dragonslayer4444` is not.
4. Complete the verification puzzle, enter the code GitHub emails you.
5. If it asks questions about your team size or interests, answer anything or look
   for **Skip**. Choose the **Free** plan if asked.

You now have an account. You'll land on a page with a left-hand sidebar — this is
your home page.

## Part 2 — Make a folder for the calculator  (2 minutes)

1. Click the **+** button in the very top-right corner of the page.
2. Click **New repository**.
3. In **Repository name**, type exactly:  `llm-cost-calculator`
   (all lowercase, with the hyphens)
4. Make sure **Public** is selected. (Public is what makes the free website work.
   People can see the files — that's fine, it's a calculator, not your accounts.)
5. Tick the box **"Add a README file"**.
6. Click the green **Create repository** button at the bottom.

You're now looking at your new, nearly-empty folder.

## Part 3 — Put the files in  (3 minutes)

1. Find the button that says **Add file** — it's near the green **<> Code** button.
   Click it, then click **Upload files**.
2. Open the unzipped package folder on your computer. Drag these files into the
   dotted upload box in your browser, all at once:
   - `index.html`
   - `README.md`
   - `DEVELOPER_BRIEF.md`  (optional, but useful later)

   **Do not upload `LLM_Hosting_Cost_Calculator.xlsx`.** The site is public;
   anything in it can be downloaded by anyone. The workbook stays with Mark,
   and the website tells visitors to contact him for it.
3. Wait for the file names to appear in the list under the box.
4. Scroll down and click the green **Commit changes** button. ("Commit" just
   means save.)

**One thing that trips people up:** the website only works if the main file is
called exactly `index.html`. If your computer renamed it during download — to
`index (1).html`, for example — rename it back before uploading.

## Part 4 — Switch the website on  (2 minutes)

1. Near the top of your repository you'll see tabs: Code, Issues, Pull requests...
   Click the last one, **Settings** (it has a little gear icon).
2. In the menu on the left side, click **Pages**.
3. Under **Build and deployment**, find **Source** and choose
   **Deploy from a branch**.
4. Two dropdowns appear under **Branch**. Set the first to **main** and the second
   to **/ (root)**. Click **Save**.

## Part 5 — See it  (2 minutes)

1. Wait one to two minutes. Make a cup of tea.
2. Refresh the Pages settings screen. A box appears at the top saying
   **"Your site is live at..."** with your address. It will look like:

   **https://**your-username**.github.io/llm-cost-calculator/**

3. Click **Visit site**. That's your calculator, live on the internet.

Share that address with anyone. When someone builds a specific estimate on it, the
address in their browser updates too — so copying the address bar shares that exact
scenario, not just the blank calculator.

---

## Afterwards

**To replace a file later** (say, an updated workbook): go to your repository,
click **Add file → Upload files**, drag in the new file with the same name, and
click **Commit changes**. GitHub replaces the old one. The website updates itself
within a minute or two.

**To take the website down:** Settings → Pages → set Source back to **None**.
To delete everything: Settings → scroll to the bottom ("Danger Zone") →
**Delete this repository** → follow the confirmation steps.

**About the prices:** everything was verified in September 2026. The calculator
watches its own age — after about six weeks it shows visitors a yellow warning
that the numbers may be stale. When you want them refreshed, give
`DEVELOPER_BRIEF.md` to whoever helps you with technical things; it contains the
complete instructions for automating weekly price updates.

## If something goes wrong

| What you see | What it means | What to do |
|---|---|---|
| "404 — there isn't a GitHub Pages site here" | The site hasn't finished building, or Pages isn't on | Wait 2 minutes and refresh. Still broken? Redo Part 4. |
| The website shows a list of files instead of the calculator | The main file isn't named `index.html` | Rename it exactly `index.html` and upload again |
| GitHub asks about "branches" or "pull requests" | An advanced feature you don't need | Close the prompt; you only ever need Add file → Upload files |

You cannot break anything permanently. The worst case is deleting the repository
and doing these 15 minutes again.
