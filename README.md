# Bharti AXA Life × AU Bank — Training Assessment

Employee-facing assessment tool: register → take a randomized MCQ test on a
chosen topic → get a certificate PDF if you score ≥ the pass percentage, with
every attempt logged to a Google Sheet for audit.

## What's in this folder

- `sheet-template/BhartiAXA-AU-Assessment-Template.xlsx` — starter Google Sheet (Topics, a sample question tab, CertificateSettings, Audit). Upload this and convert it to a native Google Sheet — this is where all content lives.
- `apps-script/Code.gs` — the backend. Bind it to the Google Sheet above via Extensions > Apps Script.
- `webapp/index.html` — the employee-facing page. Fully self-contained; host it anywhere static (GitHub Pages, Netlify, an intranet server, Google Sites embed).

## Setup (one-time)

1. **Sheet**: Upload `BhartiAXA-AU-Assessment-Template.xlsx` to Google Drive, open it, and let Drive convert it to Google Sheets (or File → Import → Insert new sheet(s)). Read the `Instructions` tab inside it.
2. **Backend**: In the Sheet, go to **Extensions → Apps Script**, delete the placeholder code, and paste in all of `apps-script/Code.gs`. Save.
3. **Deploy**: Click **Deploy → New deployment → Web app**. Set "Execute as" to **Me**, and "Who has access" to **Anyone**. Deploy, and copy the resulting web app URL (it authorizes the script's Google account the first time — approve the permission prompt for that account).
4. **Connect the frontend**: Open `webapp/index.html`, find `CONFIG.API_URL` near the top of the `<script>` block, and paste the deployment URL in.
5. **Host**: Put `webapp/index.html` wherever employees will reach it (a static host or your intranet). Nothing else needs to be uploaded alongside it — it's a single file.

Whenever you edit `Code.gs` later, you need to **Deploy → Manage deployments → Edit → New version** for changes to go live (saving alone isn't enough).

## Day-to-day: adding/editing a topic

1. Add a row to the `Topics` tab: topic name, `Active` = `Y`, optional custom pass % (blank defaults to 70%).
2. Create a tab whose name **exactly matches** that topic name, with columns `Question | Option1 | Option2 | Option3 | Option4 | Option5 | CorrectOption (1-5)`.
3. That's it — the dropdown on the employee page reads the `Topics` tab live, so a new row appears immediately without touching any code.

To retire a topic, set `Active` to `N` rather than deleting its tab (keeps historical audit rows meaningful).

## Certificate signatory

The `CertificateSettings` tab (row 2 only) controls what appears on every certificate's
signature block: `Signatory Name`, `Signatory Title` (e.g. "Training Coordinator" or "Head of
Training" — free text, shows exactly as typed), and `Signature Image URL`. If you leave the
image URL blank, the signatory's name is printed instead of an image signature. The image
must be a direct, publicly reachable link (a scanned PNG signature on a transparent background
works well) that allows cross-origin loading — Google Drive's default sharing links usually do
**not** allow this reliably; a host like GitHub, Imgur, or a Google Sites-published image tends
to work better. If the image fails to load when generating a PDF, the page falls back to the
printed name automatically.

## How grading works (security note)

The webpage never receives correct answers — it only gets questions and their option text.
When an employee submits, their selected answers are sent to the Apps Script backend, which
re-reads the sheet and grades server-side. This means opening browser dev tools during the test
never reveals which option is correct.

## Audit trail

Every submitted attempt — pass or fail, including retakes — is appended as a row to the `Audit`
tab: timestamp, employee details, topic, score, percentage, result, attempt number, and
certificate ID (blank if failed). Do not edit this tab by hand.

## Certified employees register

The `Certified Employees` tab is a separate, shorter list containing **only passing attempts**
— one row per successful certification: Employee Code, Name, Email, Branch Code, Topic, Date of
Certification, Score %, and Certificate ID. This is the tab to use when you need a clean list of
"who's certified on what" without wading through the full pass/fail audit log. It's also written
automatically by the backend — don't edit it by hand.

## WhatsApp sharing

On mobile browsers that support the Web Share API with files (most current Android/iOS Chrome
and Safari), tapping "Share on WhatsApp" opens the native share sheet with the certificate PDF
attached — the employee picks WhatsApp there. On browsers without that support (most desktop
browsers), the button instead downloads the PDF and the employee attaches it manually in
WhatsApp Web. There is no way to auto-deliver the PDF into someone's WhatsApp without a paid
WhatsApp Business API integration, which was intentionally left out of this build per your
sign-off.

## Known limitations to keep in mind

- Employee Code / Branch Code are free-text — not validated against an HR master list.
- No timer, no negative marking, unlimited retakes with no cooldown.
- Every question in a topic's tab is shown each attempt (not a random subset); only question and option order are shuffled.
