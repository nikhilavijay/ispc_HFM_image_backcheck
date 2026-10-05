# ispc_HFM_image_backcheck
Backcheck of plot x date images from ISPC Kharif 2026 season

# Plot photo backcheck: setup guide

The site has two parts:

- **Apps Script** (`Code.gs`) sits inside the Google Sheet. It hands out observations one at a time, sends the photos and the pani-pipe reading, and writes answers back to the sheet.
- **The web page** (`index.html`) is hosted on Vercel from a GitHub repository. This is the link backcheckers open.

**What backcheckers see:** they choose their name from a dropdown and enter the access key. For each observation they see only the plot photo and pipe photo, with no KEY, farmer or village details. They answer the six questions. The pani-pipe reading (`d4_wl_above_` minus `d5_wl_below_`) then appears, followed by the last question. After that they click **Save and show next images**. An observation marked `done` never appears again.

Set up Apps Script first, because the page needs its URL.

---

## Part 1: Apps Script

### 1.1 Check the sheet

Row 1 must contain these headers. Capitals and extra spaces don't matter, but the wording must match.

- `KEY` and `plot_num`. One KEY covers several plots, so the two together identify a row. (The sheet has `KEY` twice; the first one is used.)
- `Hyperlinked Image - 2` (plot photo) and `Hyperlinked Image - 1` (pipe photo)
- `d4_wl_above_` and `d5_wl_below_`
- `bc_Checked by:`, where the backchecker's name is written
- The seven answer columns:
  - `bc_Image Quality of Plot Image (Max Score: 5)`
  - `bc_Image Quality of Pipe Image (Max Score: 5)`
  - `bc_Is the field Flooded across the plot?`
  - `bc_Is Drying /Flooding correctly recorded as per photo?`
  - `bc_Is water level in pani-pipe visible?`
  - `bc_Is the panipipe reading approximately correct ( x cm below soil level)?`
  - `bc_Comments`

The script adds three tracking columns at the end of row 1 the first time it runs: `bc_status`, `bc_claimed_at` and `bc_submitted_at`. Don't delete them.

If an observation has only one photo, leave the other image cell blank. Questions about the missing photo are skipped and recorded as `NA`.

### 1.2 Add the script

1. In the sheet, go to **Extensions → Apps Script**.
2. Delete everything in `Code.gs` and paste in the full contents of `Code.gs` from this folder.
3. In the `CONFIG` block at the top, change `SECRET` to an access key of your choice. If the data isn't on the first tab, set `SHEET_NAME` to the tab's name.
4. In the left sidebar, click **+** next to **Services**, choose **Drive API** (v3), and click **Add**.
5. Save the file (Ctrl/Cmd + S).

### 1.3 Grant permissions and check the columns

1. In the function dropdown in the toolbar, select `authorize`, then click **Run**.
2. Click **Review permissions** and choose your account. If you see "Google hasn't verified this app", click **Advanced → Go to (project name) (unsafe)**. This is your own script, so it's safe. Then click **Allow**.
3. Open the **Execution log**. Every line should say `found`. If one says `MISSING`, fix that header in the sheet, or the matching name in `CONFIG`, and run again.

### 1.4 Deploy as a web app

1. Click **Deploy → New deployment**. Click the gear icon and choose **Web app**.
2. Set **Execute as: Me** and **Who has access: Anyone**. It must be "Anyone"; the access key is what keeps others out.
3. Click **Deploy** and copy the **Web app URL**. It ends in `/exec`.

**Quick check:** open this in your browser, using your own URL and key:

```
https://script.google.com/macros/s/XXXX/exec?action=next&key=YOUR_SECRET&by=test
```

Text starting with `{"row":` means it works. This reserves one observation for "test" for 15 minutes. To release it straight away, clear `bc_status` for that row.

Don't use the **Run** button on `doGet` to test. It fails in the editor because there's no web request.

---

## Part 2: Prepare the page

Open `index.html` in a text editor. Find the `CONFIG` block near the bottom of the file and make these changes:

1. Set `API_URL` to your `/exec` URL, keeping the quote marks.
2. Replace the placeholder `BACKCHECKERS` with the real names, for example:
   ```js
   BACKCHECKERS: ['Asha', 'Ravi', 'Meena'],
   ```
3. Optionally edit the question wording (`label`) or hints. Don't change `col`, which must match the sheet header.

Save the file.

---

## Part 3: Put it on GitHub

1. At [github.com](https://github.com), click **+ → New repository**. Name it (for example `plot-backcheck`), choose **Private**, and click **Create repository**.
2. Click **uploading an existing file** and drag in **`index.html`**. Leave out `Code.gs`, because it contains your access key.
3. Click **Commit changes**.

---

## Part 4: Host it on Vercel

1. At [vercel.com](https://vercel.com), click **Sign up / Log in with GitHub**. Allow access to the `plot-backcheck` repository.
2. Click **Add New… → Project**, find `plot-backcheck`, and click **Import**.
3. Set **Framework Preset** to **Other** and leave every build setting empty. Click **Deploy**.
4. You'll get a link like `plot-backcheck.vercel.app`. Share it with backcheckers, along with the access key.

### Test before sharing

1. Open the link, choose a name, enter the key, and complete one observation.
2. Check that row in the sheet. It should have all seven answers, the name in `bc_Checked by:`, `bc_status = done` and a timestamp.
3. Clear those cells so the observation goes back into the queue.

---

## Making changes later

**Page changes** (names, question wording): open `index.html` on GitHub, click the pencil icon, edit, and click **Commit changes**. Vercel updates the site within a minute.

**`Code.gs` changes:** edit in Apps Script, then go to **Deploy → Manage deployments**, click the pencil icon, set **Version: New version**, and click **Deploy**. Saving alone doesn't update the live site. Don't make a *new* deployment, because that changes the URL.

**Re-checking an observation:** clear its `bc_status` cell.

**New observations:** add rows to the sheet. They appear on the site automatically.

---

## Troubleshooting

| What you see | Likely cause and fix |
|---|---|
| "Could not reach the sheet" | `API_URL` is wrong, or the deployment isn't set to **Anyone** (step 1.4). |
| "Access key is wrong" | The key doesn't match `SECRET`. If you changed `SECRET`, check that you deployed a new version. |
| "These columns are not in the sheet: …" | A header in the sheet doesn't match `CONFIG` in `Code.gs`. Rename one so they match. |
| "These answer columns are not in the sheet: …" | A `col` in `index.html` doesn't match the sheet header. |
| "Plot photo couldn't be loaded" | The image cell has no working Drive link, the Drive API isn't added (step 1.2), or your account can't open that file. |
| Reading shows "No reading recorded" | Both `d4_wl_above_` and `d5_wl_below_` are blank or non-numeric for that row. |
| Changes don't show | For `Code.gs`, deploy a new version. For `index.html`, check the latest deployment under **Deployments** in Vercel. |

**Notes**

- Each observation takes a few seconds to load, because the photos are fetched from Drive and resized.
- Vercel's free Hobby plan is for personal, non-commercial use. Check with your team whether an organisational account is needed.

