# 🛡️ Extracting and Interpreting Browser Artifacts on Windows

A hands-on project pulling apart browser artifacts on a Windows machine — **history**, **saved passwords**, **cookies**, **cache**, and **download records** — using **BrowsingHistoryView**, **WebBrowserPassView**, **ChromeCacheView**, and **DB Browser for SQLite**.

---

## 🎯 Project Overview

Browsers store far more than most people realise. Every site visited, every saved password, every cookie, every file downloaded — it's all on disk in plain SQLite databases inside the user profile. For an investigator that's a goldmine; for the user it's a privacy footprint they probably don't think about. I wanted to see what that footprint actually looks like, where the data lives, what format it's in, and how much of it you can pull out with free tools.

I chose Edge because I use it a lot and it's Chromium-based, so anything that works on it works the same way on Chrome, Brave, and Opera. I've worked my way from the most visible artifact (URL history) to the most internal (the cache index). The order matters: history gives you the big-picture timeline, and each subsequent artifact sharpens the picture — passwords tell you what the user was logging into, cookies tell you which sites set tracking, the cache tells you what specific resources got loaded, and downloads tell you what did the user transferred onto the disk.

A few things I picked up along the way that I didn't expect at the start: the database files are easier to work with directly than through the GUI tools in some cases; Chromium timestamps look like nonsense until you know what they are; and old "deprecated" tools like WebBrowserPassView still work on current browsers more often than you'd think.

---

## 📋 Pre-requisites

- Basic familiarity with Windows and the Command Prompt
- A Windows machine (physical or virtual) with administrative privileges
- At least one browser with some real activity on it (history, saved passwords, downloads) — otherwise there's nothing to look at

---

## 🖥️ Lab Environment

- **Operating System:** Windows 11 — running as a VM on macOS
- **Working directory:** `Desktop\Project 4`
- **Browser analysed:** Microsoft Edge (default browser on the VM)

---

## 🧰 Tools Used

| Tool | Purpose |
|---|---|
| **BrowsingHistoryView** | NirSoft utility — extracts and displays browser history across Chrome, Edge, Firefox, IE, Opera, and others from one place |
| **WebBrowserPassView** | NirSoft utility — recovers passwords stored in installed browsers |
| **ChromeCacheView** | NirSoft utility — reads and lists entries from the cache folder |
| **DB Browser for SQLite** | Open-source GUI for SQLite databases — used to inspect the raw cookies and downloads tables that Chromium-based browsers store on disk |

### Installing BrowsingHistoryView

1. Download the 64-bit zip from the [NirSoft website](https://www.nirsoft.net/utils/browsing_history_view.html). Portable — no installer.



<p align="center">
  <img src="https://github.com/user-attachments/assets/ee2882cc-9ee6-4141-a39f-15c1ab3aefd0" alt="NirSoft BrowsingHistoryView download page with browsinghistoryview-x64.zip in the Edge downloads pane" width="700"><br>
  <em>Figure: Downloading BrowsingHistoryView (64-bit) from NirSoft</em>
</p>

2. Extract the contents to `Desktop\Project 4`. Easy to find.

### Installing WebBrowserPassView

1. Download the zip from the [NirSoft website](https://www.nirsoft.net/utils/web_browser_password.html). No installer — it's a portable tool(exe).

<p align="center">
  <img src="https://github.com/user-attachments/assets/f13387d5-0d27-4f0a-a56a-a88c48b9b2e8" alt="NirSoft WebBrowserPassView download page with webbrowserpassview.zip in the Edge downloads pane" width="700"><br>
  <em>Figure: Downloading the WebBrowserPassView zip from NirSoft</em>
</p>



2. Extract the contents to `Desktop\Project 4`. The folder holds the `.exe` and a couple of helper files.

> 💡 **The zip is password-protected.** NirSoft puts a password on the archive to stop browsers and AV scanners from inspecting the contents on download (the tool would otherwise get quarantined even before the download is completed). The password is shown directly on the download page (`wbpv28821@`). Worth a heads-up because Windows Explorer's built-in zip handler doesn't always make the password prompt obvious; 7-Zip is more reliable here.


### Installing ChromeCacheView

1. Download the zip from the [NirSoft website](https://www.nirsoft.net/utils/chrome_cache_view.html). Portable, no installer — same as the other NirSoft tools.


<p align="center">
  <img src="https://github.com/user-attachments/assets/34fdd14a-d41b-4245-a20a-78f2f4367971" alt="NirSoft ChromeCacheView download page with chromecacheview.zip in the Edge downloads pane" width="700"><br>
  <em>Figure: Downloading ChromeCacheView from NirSoft</em>
</p>

2. Extract to `Desktop\Project 4` alongside the other NirSoft tools.


### Installing DB Browser for SQLite

1. Download the 64-bit standard installer (`DB.Browser.for.SQLite-v3.13.1-win64.msi`, latest release at the time) from [sqlitebrowser.org/dl](https://sqlitebrowser.org/dl/). The page also offers portable `.zip` builds and 32-bit / ARM64 variants if you need them.


<p align="center">
  <img src="https://github.com/user-attachments/assets/4ad194b8-9b65-4afd-8e44-aae6bfa8dcbd" alt="DB Browser for SQLite download page showing the 3.13.1 Windows builds, with the 64-bit MSI in the Edge downloads pane" width="700"><br>
  <em>Figure: Downloading DB Browser for SQLite 3.13.1 (64-bit MSI)</em>
</p>

2. Run the MSI and accepted the defaults settings.

<p align="center">
  <img src="https://github.com/user-attachments/assets/d432876d-abcb-4dcf-ab9f-453cfd6becda" alt="DB Browser for SQLite Setup Wizard completion screen" width="700"><br>
  <em>Figure: DB Browser for SQLite installation complete</em>
</p>

> 💡 **Note:** Chromium browsers (Chrome, Edge, Brave, Opera) lock their SQLite databases while the browser is running. Close the browser before opening `Cookies`, `History`, or `Login Data` in DB Browser — or copy the file somewhere else first and open the copy.

---

## 🔬 Exercises

### Exercise 1 — Extracting Browser History

**🎯 Objective:** Pull the browser history off the machine using BrowsingHistoryView.

**Background:** Modern browsers keep a local SQLite database of every URL visited, with timestamps and visit counts. Chromium-based browsers store this in a file called `History` inside the user profile. BrowsingHistoryView can read all of them from one place.

**Expected Output:** A unified, sortable list of every URL visited across the selected browsers, with timestamps and visit counts.

**Steps:**

1. **Run `BrowsingHistoryView.exe`** from `Desktop\Project 4`. The tool opens straight into an **Advanced Options** dialog rather than a main window — you choose the browsers, profile source, and date range first, then it queries everything in one pass.

2. **Configure the filters.**
   - **Filter by visit date/time:** set to *Load history items from the last 10 days* (12/05/2026 → 19/05/2026) to focus on activity around the project window rather than dragging in every URL ever visited on this VM.
   - **Web Browsers:** ticked both `Internet Explorer 10/11 + Edge` and `Edge (Chromium-based)`.
   - **Load history from:** *Load history from the current running system (All users)*. The "All users" variant tries every profile on the box; sensible default unless you only want one account's data.

<p align="center">
  <img src="https://github.com/user-attachments/assets/51632df1-128a-48ef-9d41-8225dd4124b8" alt="BrowsingHistoryView Advanced Options dialog with Edge (Chromium-based) and Internet Explorer 10/11 + Edge checked, date range 12-19 May 2026, profile source set to all users" width="700"><br>
  <em>Figure 1: BrowsingHistoryView's Advanced Options dialog — browser selection, 10-day window, all-users profile source</em>
</p>

3. **Click OK.** The tool read Edge's SQLite databases in the background and rendered a table showing the accesses resources.

4. **Review the merged history.** **348 items** in the 10-day window.

<p align="center">
  <img src="https://github.com/user-attachments/assets/078a4596-947c-4765-91fb-f396b597494a" alt="BrowsingHistoryView results showing 348 history items including Bing searches for python and github volatility, Python.org pages, Volatility Foundation, GitHub repos, and several file:// entries to local screenshots" width="700"><br>
  <em>Figure 2: BrowsingHistoryView results — 348 items across Edge history, sorted by visit time</em>
</p>



   What jumped out:

   - **Search → click → site pattern.** The first few rows trace a textbook lookup chain: `bing.com/search?qs=AS&pq=python` (01:10:44) → `python.org` (01:10:49) → `python.org/downloads/` (01:13:18). Visible cause-and-effect over a few minutes. The same pattern repeats later for `bing.com/search?q=github%2Fvolatilityfoundation%2Fvolatility3` → `github.com/volatilityfoundation/volatility3` → `volatilityfoundation.org`.
   - **`file:///` entries from local folders.** Several rows like `file:///C:/Users/nyco8/OneDrive/Immagini/Screenshots/Project1` and `file:///C:/Users/nyco8/OneDrive/Desktop/Volatility/`. Edge's history records these because the user dragged folders or screenshots into a browser tab to view them — useful for forensics because it tells you *which local folders* the user opened in their browser, not just websites.
   - **Visit Count column.** Most entries are 1, but some are 2, 3, even 4 — meaning the user navigated to that URL more than once during the window. `volatilityfoundation.org` shows 3 visits; the local `file:///C:/Users/nyco8/OneDrive/Desktop/Volatility` folder shows 4. Repeat visits are forensically meaningful — they tell you what the user *kept coming back to*, not just what they passed through.
   - **The redirect entries.** Rows like `bing.com/ck/a?!&&p=92bb466cfd4b94c5462e95270b3800f3...` aren't pages the user typed or clicked — they're Bing's click tracking redirects. Each one was followed by the real destination a fraction of a second later. Worth knowing they're not "real" visits; they're redirect hops.
   - **MemLabs research.** `github.com/stuxnet999/MemLabs` and its `Lab%201` page — educational memory forensics CTF labs. Matches the Volatility download. The user was probably setting up a memory analysis environment.



> 💡 **What to look for in a real history dataset:**
> - Visits at unusual hours, especially overnight on what should be a personal machine
> - Repeat visits (high Visit Count) to administrative URLs, password reset pages, or webmail login pages
> - `file:///` entries — they show what the user opened locally through their browser
> - Searches that telegraph intent (`how to delete browser history`, `how to disable Defender`, `bypass admin`)
> - Bing `ck/a` and Google `url?` redirect entries — useful for confirming a click happened, but the *real* destination is the next row

> 💡 **`file:///` entries are an underrated artifact.** Edge logs them in `urls` like any other navigation, which means BrowsingHistoryView surfaces them too. Cross-referenced against the file system (or the MFT and Shellbags - see my project -  Forensic analysis of Windows file system and artifacts ), they help confirm which local resources the user actually opened in the browser versus which just had sitting on disk.

---

### Exercise 2 — Recovering Stored Passwords

**🎯 Objective:** Use WebBrowserPassView to enumerate passwords saved in the installed browsers.

**Background:** Many browsers offer to save passwords because it's easier for users to forget them. The trade-off is that the credentials sit on disk, encrypted with a key the OS hands back to anyone running as that user. WebBrowserPassView automates the decryption for every browser it knows about.

**Expected Output:** A table of saved credentials with the originating browser, URL, and form-field context for each.

**Steps:**

1. **Run `WebBrowserPassView.exe`** - the tool (exe) opens straight into its main view and starts scanning installed browsers automatically.


<p align="center">
  <img src="https://github.com/user-attachments/assets/b49e9a36-0d34-47c5-86d9-ec93b3324b19" alt="WebBrowserPassView main window showing two recovered Edge passwords" width="700"><br>
  <em>Figure 3: WebBrowserPassView with two saved Edge credentials recovered</em>
</p>

2. **Review the recovered credentials.** Two entries came back, both from `Chromium-Based Edge`:
   - `https://account.live.com/password/change` — Microsoft account, password change page, with a `RetypePassword` field (created 08/05/2026 at 00:19:23)
   - `https://github.com/login` — GitHub login (created 25/04/2026 at 15:16:20)

   Each row shows URL, browser, username, password, password strength, the names of the form fields the credentials were associated with, and the source file under the user profile.



> 💡 **It still works on current Edge.** I'd half-expected this to come back empty because Chromium browsers added app-bound encryption to login data in 2024 and WebBrowserPassView hasn't been updated in a while. In practice it pulled back both of the saved logins (2 out of 2) on a current Edge build without issues. The result might be different on a profile with more entries or on a different Chromium version.

> 💡 **The username/password column is the headline, but the metadata is just as useful.** `Created Time`, `User Name Field`, and `Password Field` together tell you which saved entry corresponds to which form on which page — useful when a user has multiple accounts for the same service or when you're trying to work out exactly which page they used to log in.

> ⚠️ **Defender flagged the tool on first run.** Microsoft Defender raised a warning about `WebBrowserPassView.exe` the first time I tried to launch it — detected as `HackTool:Win32/PassView`. That's expected and not a false positive in the malware sense; the tool genuinely does what its category name says. I dismissed the warning to let it run, which is fine in a lab on a VM but obviously not the right approach on a production machine.
---

### Exercise 3 — Analysing Browser Cookies

**🎯 Objective:** Open a Chromium browser's `Cookies` database directly in DB Browser for SQLite to look at what resources have been accessed.

**Background:** Cookies are the small key-value pairs websites use to remember you across requests — session tokens, login state, preferences, ad tracking. Chromium browsers store them all in a single SQLite database called `Cookies`, in the user's profile folder. Each row tells you the host that set it, the name, the encrypted value, when it was created, and when it expires.

**Expected Output:** Direct read access to every cookie the browser has stored, queryable with SQL.

**Steps:**

1. **Locate the Cookies file.** On this VM it lived at:
   ```
   C:\Users\nyco8\AppData\Local\Microsoft\Edge\User Data\Default\Network\Cookies
   ```
   The `Network` subfolder threw me at first — older guides put `Cookies` directly under `Default\`, but current Chromium (Chrome and Edge) moved it into `Default\Network\`. If you don't see it in `Default`, check `Network` next.

2. **Copy the file to the working folder.** Edge keeps an exclusive lock on `Cookies` while the browser is running, so a normal Explorer copy refuses with a "file in use" error. I closed Edge first (had to end the lingering background `msedge.exe` processes in Task Manager too ), then copied the file to `Desktop\Project 4\Cookies`.

3. **Open the copy in DB Browser for SQLite.** One small gotcha — the `Cookies` file has no extension, so the file dialog hides it by default. Switching the file type filter to **All files (\*)** at the bottom right of the open dialog made it appear.

4. **Browse the `cookies` table.** The Default Edge profile on this VM had **287 cookies**.


<p align="center">
  <img src="https://github.com/user-attachments/assets/ee0784c9-1a00-4260-9911-a53037f069b2" alt="DB Browser for SQLite showing the Edge Cookies database with 287 rows visible across columns creation_utc, host_key, frame_site, name, value, encrypted_value" width="700"><br>
  <em>Figure 4: Edge's Cookies database loaded in DB Browser — 287 rows, with encrypted values stored as BLOBs</em>
</p>

5. **Review what's there.** A few things stood out from the first 16 rows:
   - Hosts split between Microsoft resources (`.bing.com`, `.msn.com`, `.microsoft.com`, etc ) and developer tooling (`.github.com`, `.gitkraken.com`).
   - Several `.gitkraken.com` cookies named `_ga`, `_gcl_au`, `_uetvid` — Google Analytics, Google Ads conversion linker, and Microsoft UET (Universal Event Tracking) tags. Standard ad/analytics tracking, set even on a developer-focused site.
   - `_EDGE_V` appears under both `.bing.com` and `.msn.com` — Edge's own browser-identifying cookie.
   - `CookieConsent` on `www.gitkraken.com` records the user's cookie banner choice; useful in a privacy investigation because it shows the user actually interacted with the consent dialog.



> 💡 **The values are encrypted.** As expected, every row shows `BLOB` in the `encrypted_value` column and the plain `value` column is empty. The metadata (host, name, timestamps, path, secure/HttpOnly flags) stays in plaintext though, and for most investigative questions — *which* sites set cookies, *when*, and *what they were called* — that's enough.

> 💡 **Chromium timestamps aren't Unix time.** Notice the `creation_utc` values are 17-digit numbers like `13421236984570297`. That's "Webkit time". To convert to a normal datetime in DB Browser's `Execute SQL` tab:
> ```sql
> SELECT datetime((creation_utc / 1000000) - 11644473600, 'unixepoch') AS created,
>        host_key, name
> FROM cookies
> ORDER BY creation_utc DESC;
> ```
> Without that conversion you can sort by recency (newer = larger number) but the raw values look meaningless.

---

### Exercise 4 — Examining Browser Cache

**🎯 Objective:** To load the Chromium cache folder into ChromeCacheView and see what content the browser has held on to.

**Background:** The cache is where browsers stash copies of resources they've already downloaded — images, scripts, stylesheets, sometimes whole pages — so they don't have to fetch them again. Forensically it's useful because it tells you specific URLs the user actually loaded (not just navigated to), and/or it can contain content from pages that no longer exist online. Chromium's cache format is binary and indexed across multiple files, so you don't just open it in a text editor — you point a parser at the whole folder.

**Expected Output:** A list of cached resources with the URL, MIME type, size, and timestamps for each.

**Steps:**

1. **Locate the cache folder.** On this VM:
   ```
   C:\Users\nyco8\AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data
   ```

2. **Run `ChromeCacheView.exe`**  By default it autoloads the Chrome cache, so the title bar initially showed a `Google\Chrome\...` path even on a machine where I'd been using Edge. I've switched it to Edge: `File` > `Select Cache Folder`, then pointed the picker at `AppData\Local\Microsoft\Edge\User Data\Default\Cache\Cache_Data`.



<p align="center">
  <img src="https://github.com/user-attachments/assets/6d7d67b6-9a63-475c-a7e3-77944eecf83b" alt="ChromeCacheView Select Cache Folder dialog with a Browse for Folder picker navigated to the Edge Cache_Data folder" width="700"><br>
  <em>Figure 5: Pointing ChromeCacheView at Edge's <code>Cache_Data</code> folder rather than its default Chrome path</em>
</p>

3. **Load Cache** No locking issue this time — ChromeCacheView read the index live without Edge needing to be closed, presumably because it opens the files read-only. Came back with **3,946 cached items**.


<p align="center">
  <img src="https://github.com/user-attachments/assets/d6503707-0f0c-4db7-a9df-c34a373a1a4f" alt="ChromeCacheView showing 3,946 cached items from Edge with columns for Filename, URL, Content Type, File Size, Last Accessed, Record Created Time, and Server Time" width="700"><br>
  <em>Figure 6: ChromeCacheView with Edge's cache loaded — 3,946 items, sorted by filename</em>
</p>

4. **Review what's there.** A few patterns from the visible rows:
   - **GitHub Copilot Chat traffic** — multiple `application/json` responses from `github.com/github-copilot/chat/implicit-context/NickB-...` across 19/05/2026, file sizes 213–346 bytes. Each one is a request the Copilot extension made while the user worked in a repo. The cache effectively reconstructs the user's Copilot interaction timeline minute by minute.
   - **Bing search response fragments** — three `text/html` entries from `bing.com/recoRS?&IG=...` with different IG (impression GUID) values, sizes ranging from 69 to 878 bytes, spread across 14/05 and 19/05. Each unique IG is a distinct search session.
   - **Edge autofill service calls** — several `application/json` entries from `edge.microsoft.com/autofillservice/core/page/...`. These are Edge phoning home with autofill metadata as the user filled forms; sizes range from 20 bytes to over 9 KB.
   - **Google async tracking** — `google.com/async/hpba?vet=10ahUKEwiJlefc5cW...` came up in the cache despite no obvious Google site visit. Edge embeds Google services in its new tab page and Bing search results, so this gets cached as a side effect.

   Across all that, the columns ChromeCacheView pulls out — `Filename`, `URL`, `Content Type`, `File Size`, `Last Accessed`, `Record Created Time`, `Server Time` — give a sortable, filterable picture of every resource the browser fetched, when it fetched it, and what type it was.



> 💡 **The cache shows what the user *actually loaded*, not just what they navigated to.** History tells you the user visited `github.com`. The cache tells you the page made a GraphQL call to `_graphql` at 11:44:44 returning 730 bytes — that's a much more specific record.

> 💡 **No browser-closing step needed here.** Unlike `Cookies` and `History`, ChromeCacheView reads the cache index without needing exclusive access. Saves a step compared to the SQLite-based exercises.

> 💡 **"ChromeCacheView" is a misleading name.** It reads any Chromium browser's cache since all use the same format. You just have to override the default folder when working with anything other than Chrome.

---

### Exercise 5 — Interpreting Download Records

**🎯 Objective:** Pull the download history out of the browser's `History` database and see what's been saved to disk.

**Background:** Download records are stored in the same `History` SQLite database as the URL history, just in a different table — `downloads`. Each row records the source URL, the local file path, start and end times, total bytes, and a state code. It's a tidy record of every file the user has saved through the browser, including ones they've since deleted.

**Expected Output:** A queryable list of every file downloaded through the browser, with source URL, target path, byte counts, and timestamps in human-readable form.

**Steps:**

1. **Locate the `History` database.** On this VM:
   ```
   C:\Users\nyco8\AppData\Local\Microsoft\Edge\User Data\Default\History
   ```

2. **Copy the file to `Desktop\Project 4`** after closing Edge fully (same routine as Exercise 3 — kill leftover `msedge.exe` processes in Task Manager first ).

3. **Open the copy in DB Browser for SQLite** and selected the `downloads` table from the dropdown on the Browse Data tab. The schema panel on the right showed **24 tables** in total — `downloads`, `downloads_url_chains`, `downloads_slices`, `edge_downloads`, `urls`, `visits`, and others. The `downloads` table holds the main records; the other `downloads_*` tables hold related metadata (redirect chains, file segments, Edge-specific extras).


<p align="center">
  <img src="https://github.com/user-attachments/assets/004456d0-f43c-441f-89d8-f28440b79e2f" alt="DB Browser showing the downloads table in Edge's History database with 22 rows, columns visible include id, guid, target_path, start_time (as raw Webkit timestamps), received_bytes, total_bytes, state, danger_type" width="700"><br>
  <em>Figure 7: <code>downloads</code> table in Edge's History DB — 22 rows, with raw Webkit timestamps in <code>start_time</code></em>
</p>

4. **Notice the timestamps are unreadable.** `start_time` values like `13421636794256747` aren't Unix time — Chromium uses **Webkit time**. Same format as the `creation_utc` in the cookies database.


<p align="center">
  <img src="https://github.com/user-attachments/assets/f6321983-9a6a-44de-9e12-4a6f3856bc4f" alt="Same downloads table scrolled to show the top of the result set, including ChromeSetup.exe, LogParser.msi, WFA.zip, volatility3 archives, and python installer files" width="700"><br>
  <em>Figure 8: Scrolling through the same view — file names visible (ChromeSetup.exe, LogParser.msi, WFA.zip, python-3.14.4, etc.)</em>
</p>

5. **Convert the timestamps with SQL.** Switched to the **Execute SQL** tab and run:
   ```sql
   SELECT
     datetime((start_time / 1000000) - 11644473600, 'unixepoch') AS start_readable,
     datetime((end_time   / 1000000) - 11644473600, 'unixepoch') AS end_readable,
     target_path,
     tab_url,
     total_bytes,
     state,
     mime_type
   FROM downloads
   ORDER BY start_time DESC;
   ```
   Divide by 1,000,000 to convert microseconds to seconds, subtract 11,644,473,600 (the gap between 1601 and 1970) to get Unix time, then `datetime(..., 'unixepoch')` formats it.


<p align="center">
  <img src="https://github.com/user-attachments/assets/53b5b82f-0287-4aed-b394-cfbebe63117b" alt="Execute SQL tab in DB Browser showing the timestamp conversion query and result panel with start_readable, end_readable, target_path, tab_url columns" width="700"><br>
  <em>Figure 9: The conversion query in the Execute SQL tab — 22 rows returned in 127ms</em>
</p>

6. **Review the readable results.**


<p align="center">
  <img src="https://github.com/user-attachments/assets/e953a55f-0287-4067-bb74-a89821416a30" alt="Result of the conversion query showing 22 downloads with readable timestamps, target paths, source URLs and byte counts" width="700"><br>
  <em>Figure 10: Same data with timestamps converted to <code>YYYY-MM-DD HH:MM:SS</code> — much easier to read</em>
</p>

   The download history covers **10–19 May 2026** with 22 entries. Key patterns:

   - **The tooling for this project** — `ChromeSetup.exe` from `google.com/intl/en_uk/` (11.7 MB, 16:42 on 19 May), `WFA.zip` from `mitec.cz/wfa.html` (2.1 MB, 13:57 on 12 May), `LogParser.msi`, the NirSoft tools from `nirsoft.net/utils/`, DB Browser from `sqlitebrowser.org/dl/`, FTK Imager from `go.exterro.com`. Every tool I'd installed for the current and previous projects shows up here with a source URL.
   - **Larger data downloads** — `volatility3-...` , `MemLabs-Lab1.7z` , `Git-2.54.0` , `windows.zip`  and `python-3.14.4` builds.
   - **A duplicate WFA download** — `WFA.zip` followed by `WFA (1).zip` at the same byte count from the same URL. Windows adds the `(1)` suffix when a file with the same name already exists; useful evidence that the user downloaded the same archive twice rather than e.g. moved or renamed it. Possibly downloaded twice by accident, or once to test and once to keep.
   - **The `state` column is `1` across every row** — `1` means the download completed successfully. No interrupted or cancelled downloads in the set. The codes are: `0` = in progress, `1` = complete, `2` = cancelled, `3` = interrupted, `4` = interrupted (resumable).
   - **`danger_type` is `4` on every row** — Edge's classification for "user validated the dangerous download" (i.e. clicked through any SmartScreen warning). Not surprising for things like `.exe` and `.msi` installers, which almost always trigger the prompt.
   - **The `tab_url` column tells us what page the user was on when they clicked the link**, which isn't always the same as where the file came from. For most rows here it's just the download page (e.g. `https://www.nirsoft.net/utils/...`).


> 💡 **Cross-reference with the MFT (if available).** The `end_time` from this table should line up closely with the `$STANDARD_INFORMATION` creation time of the file on disk. A big mismatch is interesting — either timestomping, or the file was moved/renamed after download. The `WFA.zip` / `WFA (1).zip` pair would be a good candidate to verify against an MFT timeline.

> 💡 **`state` and `danger_type` are useful filters during triage.** `state = 1 AND danger_type != 0` gives us the completed downloads the browser flagged as potentially dangerous at some point. 

> 💡 **The `History` DB has 24 columns.** `urls` + `visits` together give you the URL history; `keyword_search_terms` and `edge_keyword_search_terms_source` capture searches; `downloads_url_chains` records the full redirect chain that led to each download (useful when phishing). Worth seing what else lives here before closing the file.

---

## 🎓 Lessons Learned

### Technical insights
- **Browsers store everything in SQLite, and that's the unlock.** Once I realised `History`, `Cookies`, and `Login Data` are just SQLite files I could open in DB Browser, the project went from "use these three GUI tools" to "you can query any field with SQL if the tool's view doesn't show what you want." The dedicated tools (BrowsingHistoryView, WebBrowserPassView, ChromeCacheView) are convenient wrappers; DB Browser is what you reach for when the wrapper doesn't expose what you need but.... you need to learn some SQL.
- **Chromium uses Webkit timestamps everywhere** Caught me out the first time in the cookies database (`creation_utc` like `13421236984570297`) and again in the downloads table (`start_time`). Conversion is `datetime((value / 1000000) - 11644473600, 'unixepoch')` and it's worth memorising because every Chromium-derived database uses it. I used this website for the conversions: https://www.epochconverter.com/webkit. There are other methods but wanted to keep it simple.
- **Cookie values are encrypted, metadata isn't.** Every row in Edge's `cookies` table showed `BLOB` in the `encrypted_value` column and an empty `value` column. A stolen DB on its own doesn't leak session tokens. The plaintext `host_key`, `name`, `creation_utc`, and `expires_utc` are still plenty for most investigative questions ("which sites set cookies, when, and what were they called").
- **The cache shows what was *loaded*, not what was *navigated to*.** Two different data points. History told me the user visited `github.com`; the cache showed me the specific `_graphql` POST that page made at 11:44:44 returning 730 bytes. On a SaaS-heavy machine the cache is often a finer-grained record of user activity than the URL history.
- **Tool names lie.** ChromeCacheView reads any Chromium browser's cache including Edge — it just defaults to Chrome's path and you have to override it. WebBrowserPassView still works on current Edge despite not being updated in a few years and despite Chromium adding app-bound encryption (it pulled back both saved logins on this machine).
- **The `History` SQLite database holds 24 tables, not just history.** Downloads, search terms, redirect chains, favicons, visit segments, content annotations. Worth scrolling the schema panel before closing the file. There's a lot more in there than the name suggests.

### SOC analyst mindset
- **Search → click → site is a cause-and-effect chain you can read off the history table.** The Bing/python.org sequence in Exercise 1 took place over a 3-minute window and could be reconstructed from three rows. Useful info on a real investigation to prove a user found a malicious page (they searched for it and clicked through a result, not someone redirected them).
- **Visit counts matter more than first visits.** A URL visited once is a lookup; a URL visited four times is something the user came back to. Sorting by `visit_count` rather than `visit_time` surfaces what the user actually cared about.
- **Browser artifacts complement file system artifacts, they don't replace them.** The download history shows where files came from and when they arrived. The MFT shows whether they're still on disk and what their NTFS timestamps look like now. Cross-referencing the two catches anything that's been moved, renamed, or tampered with post-download.
- **Absence is information.** WebBrowserPassView returned only two entries because only two passwords were saved. On an unfamiliar machine that's a useful baseline — a heavy browser user with no saved passwords has either disabled the feature deliberately, cleared the data, or is using a third-party password manager. Again, useful to know.
- **`file:///` history entries are easy to miss and worth a second look.** Edge logs them in `urls` like any other navigation. They reveal which local folders the user opened in their browser, often hours before or after the rest of their activity, and they don't show up in any "websites visited" summary.

### Challenges & how to overcome them
- **Generate test data deliberately before starting analysis.** This VM had a useful mix of real activity already — saved passwords, downloads, searches, browsing — but on a fresh machine You'd want to deliberately visit a handful of sites, save a couple of test credentials, and download some files first, so the browser has content worth examining. 
- **Edge's profile was pretty difficult to find.** I checked `AppData\Local\Microsoft\` expecting to see an `Edge` folder; the directory listing in File Explorer showed `GameDVR`, `Windows`, and `WindowsApps` but seemingly no `Edge`. Turned out the folder was there all along — just not visible in the truncated/scrolled view I had open. The fix was to paste the full path directly into File Explorer's address bar. **Lesson:** when a folder you expect to exist isn't visible, don't trust the directory listing — paste the path.
- **The Cookies file moved.** Older guides put it directly at `Default\Cookies`; modern Chromium puts it under `Default\Network\Cookies`. Cost me a couple of minutes the first time I looked for it. 
- **Edge holds files open while it's running.** A simple Explorer copy of `Cookies` or `History` refuses with "in use" — and just closing the Edge windows isn't enough, because background `msedge.exe` processes linger. Task Manager → end them all → copy works. ChromeCacheView opens the cache index read-only.
- **Defender flagged WebBrowserPassView.** Expected (it's detected as `HackTool:Win32/PassView`) and not a false positive in the malware sense. I dismissed the warning to let the tool run, which is acceptable in a lab VM but obviously not how you'd handle it on a production host.
- **The `Cookies` file has no extension.** DB Browser's "Open Database" dialog filters by extension, so the file was invisible until I switched the file type filter at the bottom right of the dialog to **All files (\*)**. Small thing but easy to miss.

### What I would do differently next time
- **Convert Webkit timestamps to readable form on first open** rather than scrolling through 17-digit numbers. I used a website at first but I could only convert 1 no at a time. Then found the snippet. Pasted into the Execute SQL and run. Almost like magic.
- **Try a forensic image rather than the live system.** Everything in this project was done against the live Edge profile on the running VM. Closer to real practice would be to image the disk first and analyse the artifacts out of the image, so the workflow stays valid against a target you can't afford to change. I have another project related to that, plase see bellow.

---

## 🔗 Related Projects

- 🛡️ [Forensic Analysis of Windows File Systems and Artifacts](https://github.com/NickB-26/Forensic-Analysis-of-Windows-File-Systems-and-Artifacts) — companion project parsing MFT, Prefetch, and Shellbags. Browser artifacts and file system artifacts answer different questions about the same machine; together they give you the full picture.
---

## 📚 References

- [NirSoft — BrowsingHistoryView](https://www.nirsoft.net/utils/browsing_history_view.html)
- [NirSoft — WebBrowserPassView](https://www.nirsoft.net/utils/web_browser_password.html)
- [NirSoft — ChromeCacheView](https://www.nirsoft.net/utils/chrome_cache_view.html)
- [DB Browser for SQLite](https://sqlitebrowser.org/)
- [Chromium source — `History` schema](https://source.chromium.org/chromium/chromium/src/+/main:components/history/core/browser/)
- [SANS DFIR — Browser Forensics cheat sheet](https://www.sans.org/posters/windows-forensic-analysis/)
