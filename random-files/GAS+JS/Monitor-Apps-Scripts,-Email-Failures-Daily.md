> From: https://gemini.google.com/app/b6203138050249cb?utm_source=app_launcher&utm_medium=owned&utm_campaign=base_all&hl=en_GB

# you asked

message time: 2026-07-20 15:03:36

a way to monitor all my scripts deployed to google apps script platform, so that i can know their latest running datetime to detect any failed ones. if possible, develop a JavaScript program to send an email to me everyday

---

# gemini response

Creating a monitoring script within Google Apps Script (GAS) is a fantastic way to keep tabs on your automations. We can achieve this by utilizing the **Google Apps Script API**, which allows a script to query the execution history of all other scripts tied to your account. 

Here is a complete guide and the JavaScript code to build a daily monitoring system that checks for failed executions and emails you a summary.

### **Phase 1: The Code**

This script fetches your recent script executions, filters them for errors (like `FAILED` or `TIMED_OUT`), and generates an HTML email report.

```javascript
function sendDailyScriptReport() {
  // Fetch the 50 most recent executions across your account
  const url = 'https://script.googleapis.com/v1/processes?pageSize=50';
  const options = {
    method: 'get',
    headers: {
      Authorization: 'Bearer ' + ScriptApp.getOAuthToken()
    },
    muteHttpExceptions: true
  };

  const response = UrlFetchApp.fetch(url, options);
  const data = JSON.parse(response.getContentText());

  // Handle API errors gracefully
  if (data.error) {
    console.error('Error fetching processes:', data.error.message);
    MailApp.sendEmail(
      Session.getActiveUser().getEmail(),
      'Error in Apps Script Monitor',
      'Could not fetch script processes. Error: ' + data.error.message
    );
    return;
  }

  const processes = data.processes || [];
  const emailAddress = Session.getActiveUser().getEmail();
  let hasErrors = false;
  
  let emailBody = '<div style="font-family: Arial, sans-serif;">';
  emailBody += '<h2>Google Apps Script - Daily Execution Report</h2>';

  // Filter out successful or currently running scripts to isolate failures
  const failedProcesses = processes.filter(p => 
    p.processStatus !== 'COMPLETED' && 
    p.processStatus !== 'RUNNING'
  );

  if (failedProcesses.length > 0) {
    hasErrors = true;
    emailBody += '<h3 style="color: #d93025;">⚠️ Failed or Timed Out Executions:</h3>';
    emailBody += '<table border="1" cellpadding="8" style="border-collapse: collapse; width: 100%;">';
    emailBody += '<tr style="background-color: #f1f3f4; text-align: left;"><th>Script Name</th><th>Function</th><th>Status</th><th>Start Time</th></tr>';
    
    failedProcesses.forEach(p => {
      const scriptName = p.projectName || 'Unknown Script';
      const functionName = p.functionName || 'Unknown Function';
      const time = new Date(p.startTime).toLocaleString();
      const status = p.processStatus;
      
      emailBody += `<tr><td>${scriptName}</td><td>${functionName}</td><td style="color: #d93025;"><b>${status}</b></td><td>${time}</td></tr>`;
    });
    emailBody += '</table>';
  } else {
    emailBody += '<p style="color: #188038; font-size: 16px;">✅ All recent script executions completed successfully. No failures detected!</p>';
  }
  
  emailBody += '</div>';

  // Send the email digest
  MailApp.sendEmail({
    to: emailAddress,
    subject: hasErrors ? '⚠️ Apps Script Alert: Failed Executions Detected' : '✅ Apps Script Daily Report',
    htmlBody: emailBody
  });
}
```

---

### **Phase 2: Setup and Authorization**

Because this script uses the Apps Script API to look at *other* scripts, you need to explicitly grant it permission via the manifest file.

**Step 1: Create the Project**
1. Go to [script.google.com](https://script.google.com/) and click **New Project**.
2. Name it something like "Apps Script Monitor".
3. Paste the code above into the `Code.gs` file and save.

**Step 2: Update the Manifest (Crucial)**
1. On the left-hand sidebar, click the **Project Settings** (gear icon).
2. Check the box that says **"Show 'appsscript.json' manifest file in editor"**.
3. Go back to the **Editor** (`<>` icon). You will now see `appsscript.json` on the left.
4. Click on it and add the `oauthScopes` array so your file looks like this:

```json
{
  "timeZone": "Asia/Hong_Kong",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": [
    "https://www.googleapis.com/auth/script.processes",
    "https://www.googleapis.com/auth/script.send_mail"
  ]
}
```
*(Make sure to save the file after pasting).*

**Step 3: Run Manually Once**
1. Go back to `Code.gs`.
2. Select `sendDailyScriptReport` from the dropdown at the top and click **Run**.
3. Google will ask you to review permissions. It will warn you that the app isn't verified (because you just wrote it). Click **Advanced** -> **Go to Apps Script Monitor (unsafe)** and grant the permissions.
4. Check your inbox—you should have your first report!

---

### **Phase 3: Automate the Daily Email**

To receive this email every day, you need to set up a Time-Driven Trigger.

1. In your script editor, click on the **Triggers** icon on the left sidebar (it looks like an alarm clock).
2. Click **+ Add Trigger** in the bottom right corner.
3. Configure the settings as follows:
   * **Choose which function to run:** `sendDailyScriptReport`
   * **Choose which deployment should run:** `Head`
   * **Select event source:** `Time-driven`
   * **Select type of time based trigger:** `Day timer`
   * **Select time of day:** Choose the time window you prefer (e.g., `8am to 9am`).
4. Click **Save**.

Your monitoring system is now fully deployed. It will wake up once a day, check the last 50 executions across all your Google Apps Scripts, and send you a clean dashboard right to your email.

---

# you asked

message time: 2026-07-20 15:44:08

I followed your steps, but got this error message. explain and fix

"""
Exception: Specified permissions are not sufficient to call UrlFetchApp.fetch. Required permissions: https://www.googleapis.com/auth/script.external_request
sendDailyScriptReport@ Code.gs:12
"""

---

# gemini response

This error happened because my previous instructions for the `appsscript.json` file were missing a crucial permission. I apologize for that!

When we manually set permissions in the manifest file, Google Apps Script stops automatically detecting them. Because our script uses `UrlFetchApp` to talk to the Google Apps Script API, it needs explicit permission to make external web requests.

Here is exactly how to fix it.

### **The Fix**

You need to add the `script.external_request` scope to your `appsscript.json` file.

**Step 1: Update the Manifest File**
1. Go back to your script editor and open the `appsscript.json` file from the left sidebar.
2. Replace the entire contents with this updated version:

```json
{
  "timeZone": "Asia/Hong_Kong",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": [
    "https://www.googleapis.com/auth/script.processes",
    "https://www.googleapis.com/auth/script.send_mail",
    "https://www.googleapis.com/auth/script.external_request"
  ]
}
```

**Step 2: Save and Re-Authorize**
1. Save the `appsscript.json` file.
2. Go back to your `Code.gs` file.
3. Select the `sendDailyScriptReport` function and click **Run**.
4. Because you added a new permission, Google will ask you to **Review Permissions** again. Go through the prompts (Advanced -> Go to Apps Script Monitor) and click **Allow**. 

The script will now have the authority to fetch your execution history and should send the email to your inbox immediately.

---

# you asked

message time: 2026-07-20 15:46:32

a new error came up, explain and fix

"""
Error fetching processes: Apps Script API has not been used in project 157913461787 before or it is disabled. Enable it by visiting https://console.developers.google.com/apis/api/script.googleapis.com/overview?project=157913461787 then retry. If you enabled this API recently, wait a few minutes for the action to propagate to our systems and retry.
"""

---

# gemini response

This error occurs because Google Cloud projects don't have the **Apps Script API** turned on by default. 

Even though you are writing code *inside* Google Apps Script, the code is trying to talk to Google's backend servers as a developer platform, which requires you to flip a switch in the Google Cloud Console to grant access.

Here is how to fix it step-by-step:

### **The Fix**

**Step 1: Link your Script to a Standard Google Cloud Project**
By default, your script uses a hidden, basic Google Cloud project. To enable APIs, you need to switch it to a standard project.

1. Open your Apps Script monitor project.
2. Click on the **Project Settings** (gear icon) on the left sidebar.
3. Look for the **Google Cloud Platform (GCP) Project** section.
4. If it says "Change project", you need to create a project in the [Google Cloud Console](https://console.cloud.google.com/), grab its **Project Number**, paste it here, and click **Set project**. 

*(Note: If you are using a workspace/school account, a project might have been auto-generated, but you still need to enable the API for it).*

**Step 2: Enable the Apps Script API**
1. Click the specific link provided in your error message, or go directly to the [Google API Library](https://console.developers.google.com/apis/api/script.googleapis.com/overview).
2. Look at the top left of the Google Cloud console to ensure the correct project is selected (matching the project number from your error).
3. Click the blue **Enable** button.

**Step 3: Enable the API in your Google Account Settings**
There is one extra toggle Google requires for security:
1. Go to your [Google Apps Script Settings](https://script.google.com/home/usersettings).
2. Look for **Google Apps Script API** at the bottom of the page.
3. Change the toggle from **Off** to **On**.

---

### **Test It Again**
Once those toggles are turned on, wait about 1–2 minutes for Google's systems to update. Go back to your script editor, click **Run**, and it should successfully fetch your script history without any errors!

---

# you asked

message time: 2026-07-20 16:01:03

I tried to run the code, but got error message .
"""
monitor-all-automations has not completed the Google verification process. The app is currently being tested and can only be accessed by developer-approved testers.  Error 403: access_denied
"""

---

# gemini response

This error happens because the Google Cloud project you created defaults to a **"Testing"** publishing status. Google tightly restricts who can authorize "Testing" apps to prevent malicious software, meaning it won't even let *you* (the creator) log into your own script until your account is explicitly listed as a trusted tester.

Here is how to add yourself as a tester so you can bypass this screen and run your monitor.

### **The Fix**

**Step 1: Go to the OAuth Consent Screen**
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Make sure your project is selected in the top-left dropdown.
3. Click the **Menu icon** (three horizontal lines) in the top-left corner.
4. Navigate to **APIs & Services** > **OAuth consent screen**.

**Step 2: Add Yourself as a Test User**
1. Scroll down the page until you find the **Test users** section.
2. Click the **+ ADD USERS** button.
3. Type in your own Google email address (the one you are using to write the Apps Script).
4. Click **Save**.

---

### **Test It Again**
Go back to your Google Apps Script editor, click **Run**, and try to authorize it again. 

This time, instead of the `Error 403: access_denied` screen, you will see an "Advanced" option. Click **Advanced**, then click **Go to Apps Script Monitor (unsafe)**, and you will finally be allowed to grant the permissions.

---

# you asked

message time: 2026-07-20 16:05:39

I could not find the Test users section.

---

# gemini response

If you can't see the **Test users** section, it means your OAuth Consent Screen is currently set to an **"Internal"** user type. 

For Google Workspace (work or school) accounts, "Internal" means only people inside your exact organization can use the app. Because it is already restricted to your organization, Google hides the "Test users" box—but it also blocks you if your Apps Script environment isn't perfectly synced with the Google Cloud project.

Here is the easiest way to fix this by changing the app to **External** so you can add yourself as a tester.

### **The Fix**

**Step 1: Change User Type to External**
1. Open the [Google Cloud Console OAuth consent screen](https://console.cloud.google.com/apis/credentials/consent).
2. Near the top of the page, look for the **User type** setting. It likely says "Internal".
3. Click the **MAKE EXTERNAL** button right below it.

**Step 2: Add Your Email**
1. Once you switch it to External, the page will refresh or take you to an edit screen.
2. Scroll down, and the **Test users** section will now be visible at the bottom.
3. Click **+ ADD USERS**, type in your Gmail/Workspace email, and click **Save**.

---

### **Alternative Fix (The "Internal" Route)**
If you prefer to keep it "Internal" (or if the button is greyed out by your organization's admin), the 403 error is usually happening because you are logged into multiple Google accounts in the same browser session. 

To bypass it without changing settings:
1. Copy the link of your Google Apps Script editor.
2. Open a new **Incognito / Private Window** in your browser.
3. Log into **only** the single Google account that owns the script.
4. Run the script and authorize it—it should pass through seamlessly.

---

# you asked

message time: 2026-08-09 11:05:17

why other Google Apps Script programs did not need `oauthScopes` to call googleapis? why only this need?

---

# gemini response

The difference comes down to **how Apps Script handles built-in helper libraries versus raw HTTP requests**.

### **1. Built-In Services (Automatic Scopes)**

When you write standard Apps Script code, you usually use Google's built-in wrapper classes:

* `SpreadsheetApp.getActiveSpreadsheet()`
* `GmailApp.sendEmail()`
* `DriveApp.getFolderById()`

Before your script runs, **Apps Script scans your source code**. It sees `SpreadsheetApp` and automatically injects the Google Sheets OAuth scope behind the scenes. You don't need to configure `appsscript.json` because Google automatically infers what your script needs.

---

### **2. Why This Specific Script Needs Explicit `oauthScopes`**

Our monitoring script does something special:

1. **It calls a REST API via `UrlFetchApp.fetch()`:**
   Instead of using a built-in helper class like `GmailApp`, we hit a raw URL (`[https://script.googleapis.com/](https://script.googleapis.com/)...`). 

2. **Apps Script Code Scanner is blind to raw URLs:**
   When Apps Script scans `UrlFetchApp.fetch(url)`, it only sees "a web request to some web address". It cannot inspect the string URL or predict what Google API permissions that endpoint will demand.

3. **Generating the Bearer Token:**
   When the code runs `ScriptApp.getOAuthToken()`, Google generates an access token. However, Google will **only** bundle permissions into that token that were explicitly authorized by the user. Because Apps Script didn't auto-detect the Apps Script API scope, the token ends up missing the required authorization unless you manually declare `oauthScopes` in `appsscript.json`.

---

### **Summary**

| Method | Code Example | How Permissions Are Set |
| :--- | :--- | :--- |
| **Built-in Google Service** | `SpreadsheetApp.openById(...)` | **Automatic:** Apps Script scans the code and requests permissions automatically. |
| **Raw REST API Call** | `UrlFetchApp.fetch(...)` | **Manual:** You must explicitly list `oauthScopes` in `appsscript.json` so the OAuth token includes the right access. |

---

# you asked

message time: 2026-08-09 11:07:43

for this program, why cannot I use built-in google service

---

# gemini response

That is the ultimate "why are we doing this the hard way?" question—and the answer comes down to a deliberate limitation built into Google Apps Script.

### The Problem: There is No Built-in `AppsScriptApp` Service

Google provides built-in services for almost every product in its ecosystem—`SpreadsheetApp`, `GmailApp`, `DriveApp`, `CalendarApp`, and `DocumentApp`. 

However, **Google never created an `AppsScriptApp` or `ExecutionApp` service** that allows a script to inspect other scripts running in your account. 

### Why Google Designed It This Way

1. **Security Isolation:** By design, Google Apps Scripts run in isolated containers. Script A is not supposed to know what Script B is doing unless you explicitly grant administrative API access.
2. **Standardization:** Rather than maintaining a separate built-in service inside Apps Script, Google built the unified **Google Apps Script REST API** in the Google Cloud Console. This API is meant to be used by any programming language (Python, Node.js, Java, or Apps Script itself).

### Why We Have to Use `UrlFetchApp`

Because there is no built-in `AppsScriptApp.getExecutions()` method, we are forced to access Google's backend API directly over HTTPS using `UrlFetchApp.fetch()`. 

When you use `UrlFetchApp` to talk to Google's own APIs:
* Apps Script doesn't automatically know *which* Google service you are targeting.
* It can't auto-detect the required permissions.
* You have to manually add `oauthScopes` to the `appsscript.json` manifest so Google knows to give your script permission to read your execution history.

In short: **We have to use `UrlFetchApp` and manual scopes because Google didn't give us a native built-in service for script monitoring!**

---

# you asked

message time: 2026-08-09 11:21:19

use a infograhic to explain REST API to me

---

# gemini response

Here is an infographic that explains how REST APIs work. 

**How to Read This Infographic:**

* **The Communication Flow:** The central diagram shows how your client (the Apps Script) talks to the server (Google’s servers) over the internet. You send a "Request" (using `UrlFetchApp`) and they send back a "Response" containing data or status codes (like 200 OK).
* **HTTP Methods:** This is how you tell the API what you want to do. For our monitoring script, we are using the **GET** method because we are "reading" your script’s execution history. We are not creating (POST), updating (PUT), or deleting (DELETE) any scripts.
* **Resources & URIs:** Every asset on the API is treated as a "resource" with a unique URL. For our script, the specific resource URL we are hitting is: `[https://script.googleapis.com/v1/processes](https://script.googleapis.com/v1/processes)`.
* **Statelessness:** The diagram highlights that each request must contain everything necessary to understand it. That’s why we have to manually add the Bearer token—it’s the script's ID card, and the server doesn't "remember" who you are from one second to the next.

For this specific monitor, your script is acting as the **Client**, and the **Server** is the unified Google Apps Script API endpoint.

---
Powered by [AI Exporter](https://saveai.net)