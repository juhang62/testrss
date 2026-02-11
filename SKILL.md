---
name: xiaohongshu-post
description: Research Japanese universities, find campus photos from Wikipedia, write Xiaohongshu posts in Chinese, and save to Google Sheet. User can specify university names or selection criteria.
argument-hint: [university names or selection criteria]
allowed-tools: Bash, Read, Write, Edit, WebSearch, WebFetch, Grep, Glob
---

# Xiaohongshu Post Generator — Japanese Universities

Generate Xiaohongshu (Little Red Book) posts for Japanese universities and write the data to a Google Sheet.

**User input:** $ARGUMENTS

## Step 1: Determine University List

- If the user provides **specific university names**, use those.
- If the user provides **selection criteria** (e.g., "top 5 private universities", "universities in Kansai region", "best art universities"), research and select universities matching the criteria.
- If no input is given, ask the user what universities or criteria they want.
- For each university, use **WebSearch** to gather:
  - Full name in Japanese, English, and Chinese
  - Official website URL (top-level domain, e.g., `https://www.u-tokyo.ac.jp`)
  - Key facts: QS/THE rankings, notable alumni, campus highlights, city/location advantages, unique programs, student life perks

## Step 2: Find Campus Photos from Wikipedia

For each university:

1. Query the **Wikipedia API** to list images on the university's English Wikipedia page:
   ```
   https://en.wikipedia.org/w/api.php?action=query&titles={WIKI_TITLE}&prop=images&format=json&imlimit=50
   ```
2. Filter results: keep `.jpg`/`.png` photos of **campus buildings, scenery, landmarks, and student life**. Exclude logos, icons, flags, seals, emblems, maps, and individual portrait headshots.
3. If fewer than 5 campus photos are found, also search **Wikimedia Commons**:
   ```
   https://commons.wikimedia.org/w/api.php?action=query&list=search&srsearch={University Name} campus&srnamespace=6&srlimit=20&format=json
   ```
4. Also try the **Japanese Wikipedia** page if the English page has too few images.
5. Resolve selected filenames to full URLs via the **imageinfo** API:
   ```
   https://en.wikipedia.org/w/api.php?action=query&titles={FILE_TITLES_PIPE_SEPARATED}&prop=imageinfo&iiprop=url&format=json
   ```
   Batch up to 50 titles per request.
6. Select **5 photos per university**. All URLs must be full `https://upload.wikimedia.org/...` paths.

## Step 3: Write Xiaohongshu Posts

For each university, write a Chinese-language post following Xiaohongshu style:

- **Title line**: Catchy, with emojis, `标题｜副标题` format (e.g., "🏫 日本No.1学府｜东京大学，梦开始的地方")
- **Body**: 200-400 Chinese characters
  - Short paragraphs, each starting with an emoji bullet
  - Personal, engaging tone — write as if recommending to a friend
  - Reference what the reader will see in the photos (campus buildings, scenery, student activities)
  - Include concrete facts: rankings, number of Nobel laureates, famous alumni, unique features
  - Mention lifestyle perks: food, city life, transport, cost of living, seasonal attractions
- **Hashtags**: End with 5-8 hashtags like `#大学名 #日本留学 #城市名`

## Step 4: Write to Google Sheet

1. Use Python `gspread` with `gspread.service_account()` (service account is pre-configured).
2. Open the spreadsheet **"xiaohongshu_post"**.
3. Read row 1 to get existing column headers.
4. Find the last row with data and **append below it** — never overwrite existing rows.
5. Write data matching column headers:
   - `university` — university name in Chinese + English, e.g., "东京大学 (The University of Tokyo)"
   - `url` — official website URL
   - `content` — the Xiaohongshu post text
   - `photo1` through `photo5` — Wikimedia Commons image URLs
   - Leave other columns (e.g., `posted`) empty
6. Print a summary showing each university name and photo count.

## Important Rules

- All Wikipedia/Commons API calls must include `User-Agent` header.
- Batch API calls where possible to minimize requests.
- Verify all image URLs resolve before writing.
- Never duplicate a university that already exists in the sheet — check existing rows first.
- The Google Sheet already exists and is shared with the service account.
