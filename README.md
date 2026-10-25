# Ibex Jira Releases Web App

A lightweight, secure, responsive, and themeable Google Apps Script web app for viewing and updating Ibex Jira release versions.

- **Google Sites Portal:** [Ibex PPM Delivery Hub - Releases](https://sites.google.com/ibex-ai.com/ibex-ppm-delivery-hub/ibex-jira-releases)
- **Live Web App:** [Open Web App](https://script.google.com/a/macros/ibex-ai.com/s/AKfycbzmoYfAQQEVFAvfXqv1ASx7DWo3__scTuBPh5k3FLMrRuRiQr8RQWOY6MVGto-fjvgj/exec)
- **GitHub Repository:** https://github.com/Lavi-ibex/gapps_script_jira_releases
- **Google Apps Script IDE:** [Project Editor](https://script.google.com/d/1WzW04E-Dbtj4ICURpLU07W2crfuZ3REn5l8SIEgWaHSn4UxQfL8CbAFX/edit)
- **Script ID:** `1WzW04E-Dbtj4ICURpLU07W2crfuZ3REn5l8SIEgWaHSn4UxQfL8CbAFX`

---

## 1. Features & Architecture

- **Direct Jira Hyperlinks:** Each release name is a clickable direct link (`https://ibex-ai.atlassian.net/projects/IBX/versions/{id}`) with an external link indicator that opens the release page in Jira in a new tab for instant viewing and editing.
- **Dark / Light Mode Support:** Includes a dedicated theme toggle button in the header with high-contrast color palettes for both modes. User preference is saved in `localStorage` and defaults automatically to the user's OS preference.
- **Pure Client-Side Rendering:** Uses `HtmlService.createHtmlOutputFromFile('Index')` with asynchronous `google.script.run` data fetching. This completely prevents template compilation syntax errors.
- **Secure Backend Proxy:** Jira authentication credentials (`JIRA_USER_EMAIL` and `JIRA_API_TOKEN`) reside exclusively inside Google Apps Script `Script Properties`. They are **never** exposed in client-side HTML/JavaScript or to site visitors.
- **Hourly Caching:** Google Apps Script `CacheService` caches Jira releases data for 1 hour (3600 seconds) to ensure near-instant site loading times. Reloading the browser reloads the dashboard but uses this normal cached-data path whenever a valid cache entry exists.
- **On-Demand Refresh:** The dashboard **Refresh** button bypasses the cache and fetches current release data from Jira immediately.
- **Optional Google Sites Embedding:** Configured with `XFrameOptionsMode.ALLOWALL` so the web app can also be embedded in a Google Site when needed.
- **Release Selection:** Shows non-archived, unreleased versions with a planned release date, plus non-archived versions released during calendar year 2026. Released versions are newest first; active versions are ordered by their planned date.
- **Collapsible Descriptions:** Use the header **Show all descriptions** control to expand every available description at once, or hide them all again. Each release card also retains its own **Show description** control and a compact **Edit** action when expanded.
- **Domain-Restricted Description Editing:** Signed-in `ibex-ai.com` users can edit a release description. The update is sent from Apps Script to Jira with the server-side service account, then the one-hour release cache is cleared so refreshed results show the saved value.
- **Domain-Restricted Release-Date Editing:** Signed-in `ibex-ai.com` users can select the small **Edit** action beside a release date, choose a valid replacement date, and save it. A date is required and cannot be cleared. Apps Script updates Jira through the server-side service account, clears the cache, and refreshes the dashboard so the release is shown in its newly sorted position.

---

## 2. Development & Deployment Workflow (clasp)

This repository is connected directly to Google Apps Script via `.clasp.json`.

### Syncing Code with Clasp
- **Push local changes to Google Apps Script:**
  ```bash
  npx @google/clasp push
  ```
- **Pull remote changes from Google Apps Script:**
  ```bash
  npx @google/clasp pull
  ```

### Script Properties Setup (One-Time Setup)
1. In the Apps Script project editor, click **Project Settings** (gear icon ⚙️).
2. Under **Script Properties**, ensure the following properties are configured:
   - `JIRA_BASE_URL`: `https://ibex-ai.atlassian.net`
   - `JIRA_PROJECT_KEY`: `IBX`
   - `JIRA_USER_EMAIL`: `your.email@ibex-ai.com`
   - `JIRA_API_TOKEN`: Your Atlassian API token (generated from [id.atlassian.com](https://id.atlassian.com/manage-profile/security/api-tokens))

*(Note: The Jira account identified by `JIRA_USER_EMAIL` must have permission to manage versions in the configured Jira project. Keep this account and its API token in Script Properties only.)*

### Deploying a New Version
When code changes in `Code.gs` or `Index.html` are pushed, update the active Web App deployment:

- **Via clasp CLI:**
  ```bash
  npx @google/clasp deploy --description "Description of updates"
  ```
- **Via Apps Script Web Editor:**
  1. Open the [Project Editor](https://script.google.com/d/1WzW04E-Dbtj4ICURpLU07W2crfuZ3REn5l8SIEgWaHSn4UxQfL8CbAFX/edit).
  2. Click **Deploy > Manage deployments** (top right).
  3. Select the active Web App deployment from the list on the left.
  4. Click the **Pencil (Edit)** icon at the top right of the modal.
  5. Under **Version**, choose **New version**.
  6. Click **Deploy**.

*(Note: The deployment URL remains constant across version updates. The direct web app serves the new version after deployment is updated. If the app is embedded elsewhere, refresh that host page after deploying.)*

### Dashboard Interactive Editing
- **Show/Hide Descriptions:** Select **Show all descriptions** to expand all cards, or **Hide all descriptions** to collapse them.
- **Edit Description:** Expand a card's description, click **Edit**, modify the text, and click **Save description**.
- **Edit Release Date:** Click the **Edit** action in the release date badge, choose a valid date, and click **Save date**.
*(Editing operations are restricted to signed-in users in the `ibex-ai.com` domain.)*

---

## 3. Customization & Maintenance

- **Changing the Jira Project:** Update `JIRA_PROJECT_KEY` in Script Properties (e.g. to another project key).
- **Default Theme:** To change the default fallback theme from Light to Dark, change `<html lang="en" data-theme="light">` to `data-theme="dark"` in `Index.html`.
- **Release Filtering:** The visibility rules are implemented in `processRawVersions` in `Code.gs`; update `test_payload_structure.py` alongside an intentional change to these rules.
- **Description Updates:** `apiUpdateReleaseDescription(versionId, description)` validates the signed-in domain, validates input size, updates Jira through the server-side proxy, and clears the cached release payload.
