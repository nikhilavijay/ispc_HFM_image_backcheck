# ispc_HFM_image_backcheck
Backcheck of plot x date images from ISPC Kharif 2026 season

# Plot photo backcheck: setup guide

The site has two parts:

- **Apps Script** (`Code.gs`) sits inside the Google Sheet. It hands out plots, sends the two photos and writes answers back to the sheet.
- **The web page** (`index.html`) is hosted on Vercel from a GitHub repository. This is the link backcheckers open.

Set up Apps Script first. It gives you the web app URL that the page needs.

---

## Part 1: Apps Script (the sheet side)

### 1.1 Add the script to the sheet

1. Open the monitoring Google Sheet.
2. Go to **Extensions → Apps Script**. A new tab opens with a file called `Code.gs`.
3. Delete everything in that file and paste in the full contents of `Code.gs` from this folder.
4. Rename the project at the top left (for example, "Plot backcheck").

### 1.2 Edit the config block

At the top of `Code.gs`, change these values to match your sheet:

| Setting | What to put |
|---|---|
| `SHEET_NAME` | The tab name holding the data, exactly as it appears (e.g. `Sheet1`) |
| `SECRET` | An access key you'll share with backcheckers. Choose something not easy to guess. |
| `ID_COL` | Header of the plot ID column |
| `DATE_COL` | Header of the monitoring date column |
| `IMAGE_COLS` | Headers of the two image columns, in the order they should appear |

Header names must match row 1 exactly, including capitalisation and spaces. You don't need to create the `bc_` answer columns. The script adds them at the end of the header row the first time it runs.

### 1.3 Turn on the Drive API

1. In the left sidebar, click **+** next to **Services**.
2. Choose **Drive API**, keep version **v3**, and click **Add**.

### 1.4 Grant permissions

1. In the toolbar's function dropdown, select `authorize`, then click **Run**.
2. Click **Review permissions** and choose your J-PAL Google account.
3. If you see "Google hasn't verified this app", click **Advanced → Go to Plot backcheck (unsafe)**. This is your own script, so it's safe.
4. Click **Allow**.
5. Open **Execution log**. You should see a list of your column headers, and the `bc_status`, `bc_by`, `bc_claimed_at` and `bc_submitted_at` columns should now be in the sheet.

### 1.5 Deploy as a web app

1. Click **Deploy → New deployment**.
2. Click the gear icon next to "Select type" and choose **Web app**.
3. Set **Execute as: Me** and **Who has access: Anyone**.
   (Use "Anyone", not "Anyone with Google account", or the page can't reach it. The access key is what keeps strangers out.)
4. Click **Deploy** and copy the **Web app URL**. It ends in `/exec`.

**Quick check:** paste this into a browser, using your own URL and key:

```
https://script.google.com/macros/s/XXXX/exec?action=next&key=YOUR_SECRET&by=test
```

A page full of text starting with `{"row":` means it works. Note that this check reserves one plot for the user "test" for 15 minutes.

---

## Part 2: Put the page on GitHub

### 2.1 Add the URL to the page

Open `index.html` in any text editor (Notepad or TextEdit is fine). Near the bottom, find:

```js
API_URL: 'PASTE_YOUR_APPS_SCRIPT_WEB_APP_URL_HERE',
```

Replace the placeholder with your `/exec` URL and keep the quote marks. Save the file.

You can also edit the questions here, in the `QUESTIONS` list. Each `col` value becomes a sheet column and must start with `bc_`.

### 2.2 Create the repository

1. Sign in at [github.com](https://github.com), then click **+ → New repository** at the top right.
2. Name it (for example, `plot-backcheck`) and choose **Private**.
3. Click **Create repository**.
4. On the next page, click **uploading an existing file**.
5. Drag in **`index.html` only**. Don't upload `Code.gs`, because it contains your access key. This README is optional.
6. Click **Commit changes**.

---

## Part 3: Host it on Vercel

### 3.1 Connect GitHub to Vercel

1. Go to [vercel.com](https://vercel.com) and click **Sign up** (or **Log in**) **with GitHub**.
2. When asked, allow Vercel to access your repositories. You can limit it to just `plot-backcheck`.

### 3.2 Import the project

1. On the Vercel dashboard, click **Add New… → Project**.
2. Find `plot-backcheck` in the list and click **Import**.
3. Set **Framework Preset** to **Other**. Leave the build command, output directory and root directory empty.
4. Click **Deploy**.

After about 30 seconds you'll get a link like `plot-backcheck.vercel.app`. This is the link you send to backcheckers, along with the access key.

To change the address, go to **Project → Settings → Domains**.

### 3.3 Test it

1. Open the link and enter a test name and the access key.
2. Answer the questions for one plot and click **Save and next plot**.
3. In the sheet, check that the plot's row now has the answers, `bc_status = done`, your name and a timestamp.
4. Clear those test values from the `bc_` columns of that row so the plot goes back into the queue.

It's safest to do your first full test on a copy of the sheet (**File → Make a copy**). A copy has its own Apps Script, so repeat Part 1 for it and point a test version of the page at that URL.

---

## Making changes later

**Changing the page** (questions, wording, design): edit `index.html` on GitHub by opening the file, clicking the pencil icon, editing and clicking **Commit changes**. Vercel redeploys automatically within a minute.

**Changing `Code.gs`**: edit it in the Apps Script editor, then go to **Deploy → Manage deployments**, click the pencil icon, set **Version: New version** and click **Deploy**. Saving the code alone does nothing to the live site. Don't create a *new* deployment, because that gives a new URL and the page would still point to the old one.

**Changing the access key**: update `SECRET` in `Code.gs` and redeploy as above. Backcheckers will be signed out and asked for the new key.

**Adding plots**: just add rows to the sheet. They appear on the site automatically.

**Re-checking a plot**: clear its `bc_status` cell. It goes back into the queue.

---

## Troubleshooting

| What you see | Likely cause and fix |
|---|---|
| "Could not reach the sheet" on every load | `API_URL` is wrong, or the deployment's access isn't set to **Anyone**. Recheck step 1.5 and step 2.1. |
| "Access key is wrong" | The key typed doesn't match `SECRET`. If you changed `SECRET`, check that you redeployed with a new version. |
| "Photo couldn't be loaded" | The image cell has no Drive link or file ID, the Drive API isn't added (step 1.3), or your account can't open that file. |
| Answers land in new columns instead of existing ones | A `col` value in `QUESTIONS` doesn't exactly match your existing header. |
| "Sheet not found" | `SHEET_NAME` doesn't match the tab name. |
| Code changes don't show up | For `Code.gs`, deploy a new version. For `index.html`, check the latest deployment under the project's **Deployments** tab in Vercel. |

**Limits worth knowing:**

- Apps Script runs as your account, so its daily quotas apply. Normal backchecking volume is far below them.
- Each plot takes a few seconds to load because both photos are fetched from Drive.
- Vercel's free Hobby plan is meant for personal, non-commercial use. Check with your team whether an organisational account is needed.

