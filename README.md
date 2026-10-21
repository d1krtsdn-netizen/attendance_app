# Office Attendance – iPhone app setup

**What you get**
- An iPhone home-screen app (PWA) that works like your Excel tracker: same statuses, same calendar, same formulas.
- Your 380 logged days from the spreadsheet (Apr 2025 – Oct 2026), ready to import.
- An iPhone Shortcut that runs at 17:00 on weekdays:
  - **Already logged** → does nothing.
  - **Within 0.5 mile of the office or home** → logs it for you and sends a confirmation.
  - **Anywhere else** → sends a reminder to log it yourself.

Files:

| File | Purpose |
|---|---|
| `index.html`, `manifest.webmanifest`, `sw.js`, `icon-*.png` | The app |
| `supabase/01_schema.sql` | Database tables, security rules, bank holidays, auto-log function |
| `supabase/02_import_history.sql` | Your Excel history |

---

## Step 1 – Supabase (about 5 minutes)

1. Create a **new** free project at supabase.com. Keep it separate from your marketplace app.
2. **SQL Editor → New query** → paste all of `01_schema.sql` → **Run**.
3. **Authentication → Sign In / Providers → Email**: you can turn off *Confirm email* so you can sign straight in.
4. **Project Settings → API**: copy the **Project URL** and the **anon public** key.

## Step 2 – Host the app

1. Open `index.html` and paste the URL and anon key into `SUPABASE_URL` and `SUPABASE_ANON`, near the top of the `<script>` section.
2. Upload all the app files to a GitHub repo and turn on GitHub Pages, or deploy to Vercel. Use the same method as your expenses app.
   - A public repo is fine here. The anon key is meant to be public, and Row Level Security means only your own login can read or write your data. **Never** commit the `service_role` key.
3. On your iPhone, open the site in **Safari** → **Share** → **Add to Home Screen**.
4. Open the app from the home screen → **Create account** → **Sign in**.

## Step 3 – Import your Excel history

1. Open `02_import_history.sql` and replace `YOUR_EMAIL` with the email you signed up with.
2. Paste it into the SQL Editor and select **Run**. Re-open the app and check **Summary**: each month should match the % in your workbook.

## Step 4 – Set your locations

In the app, go to **Settings**.

1. While you're at the office, tap **Set office to my current location**. While you're at home, tap **Set home to my current location**.
2. You can also type in the coordinates. To get them, long-press a spot in Apple Maps or Google Maps.
3. Set the radius. The default is 0.5 mile.
4. Tap **Save**.

## Step 5 – Build the Shortcut (the 17:00 reminder and auto-log)

Open the **Shortcuts** app → **Shortcuts** tab → **+** and name it **Log Attendance**. Add these actions in order:

1. **Get Current Location**
2. **Get Contents of URL**
   - URL: the *Endpoint URL* from the app's Settings screen (`https://YOUR-PROJECT.supabase.co/rest/v1/rpc/auto_log`)
   - Method: **POST**
   - Headers: `apikey` = *anon key*, `Content-Type` = `application/json`
   - Request Body: **JSON**
     - `p_token` (Text) = *your secret token from Settings*
     - `p_lat` (Number) = **Current Location → Latitude** (tap the variable and choose *Latitude*)
     - `p_lon` (Number) = **Current Location → Longitude**
3. **Get Dictionary Value** → key `notify` from *Contents of URL*
4. **If** *Dictionary Value* **is** `yes`
   - **Get Dictionary Value** → key `message` from *Contents of URL*
   - **Show Notification** → *Dictionary Value*
5. **End If**

Run it once by hand. Allow location access when asked, and check that you get a sensible message.

Then go to **Automation** → **+** → **Time of Day**:
- Time: **17:00**. Change this whenever you like; this is the "configurable" part.
- Repeat: **Weekly** → Mon–Fri
- Choose **Run Immediately** and turn off *Notify When Run*
- Action: **Run Shortcut → Log Attendance**

### Recommended extra: log the office when you arrive
At 17:00 you might be on the train home, so you'd be near neither location. To cover that, add a second automation:
- **Automation → + → Arrive** → choose your office address → **Run Immediately** → **Run Shortcut → Log Attendance**.

When you arrive, today is logged as Office. The 17:00 run then sees it's already logged and stays quiet.

---

## How it matches the spreadsheet
- **Working days** = days that are `--`, Office, Home, WFA or OOO. Weekends and bank holidays are excluded, as in your "Is Eligible Day" column.
- **Attendance %** = (Office + WFA) ÷ (Working days − OOO) × 100.
- Weekends and UK bank holidays are filled in automatically. **Settings → Refresh UK bank holidays** pulls future years from gov.uk.
- Days logged automatically show ⚡ in the Month view. You can change any day by hand.

## Good to know
- The bank holiday list in your workbook is missing **Mon 28 Dec 2026**, the Boxing Day substitute day. The app already includes it.
- iOS shows the Shortcut notification as coming from *Shortcuts*. Tapping it opens Shortcuts, not this app.
- If you think the token has been shared, use **Settings → Regenerate**, then paste the new token into the Shortcut.
