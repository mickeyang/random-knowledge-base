> From: https://gemini.google.com/app/87559db8cbccd633?utm_source=app_launcher&utm_medium=owned&utm_campaign=base_all&hl=en_GB

# you asked

message time: 2026-07-10 20:59:40

I need you to analyse this markdown file and coding a google apps script, let's start with a simple email first, only include today + overdue.

---

# gemini response

Here is the complete Phase 1 setup for your Google Apps Script. This script scans all your task lists, pulls any tasks inside your **"1-Today"** list, identifies **overdue** items across your whole account, formats them into a clean HTML email, and delivers it straight to your inbox.

---

## 🛠️ Step 1: Create the Script & Enable Tasks API

Because Google Tasks is an advanced service, you have to explicitly turn it on in your script settings.

1. Go to [script.google.com](https://script.google.com/) and click **New Project**.
2. Rename your project to something like `Daily Task Reminder`.
3. In the left sidebar, look for **Services** and click the **`+` (Add a service)** icon.
4. Scroll down, select **Tasks API**, and click **Add**. *(This gives the script permission to talk to your Google Tasks app).*

---

## 💻 Step 2: Paste the Code

Clear out any code in the default `Code.gs` file and paste the following script:

```javascript
function sendDailyAgenda() {
  const todayTasks = [];
  const overdueTasks = [];
  
  // Get current date string in local timezone (YYYY-MM-DD)
  const timeZone = Session.getScriptTimeZone();
  const todayStr = Utilities.formatDate(new Date(), timeZone, "yyyy-MM-dd");
  
  // Fetch all task lists
  let taskLists;
  try {
    taskLists = Tasks.Tasklists.list().items;
  } catch (e) {
    Logger.log("Error fetching task lists: " + e.message);
    return;
  }
  
  if (!taskLists || taskLists.length === 0) {
    Logger.log("No task lists found.");
    return;
  }
  
  // Use a Set to track processed task IDs and completely avoid duplicates
  const processedTaskIds = new Set();
  
  taskLists.forEach(list => {
    let tasksResult;
    try {
      // Pull only uncompleted/active tasks
      tasksResult = Tasks.Tasks.list(list.id, {
        showCompleted: false,
        showHidden: false
      });
    } catch (e) {
      Logger.log("Error fetching tasks for list " + list.title + ": " + e.message);
      return;
    }
    
    const tasks = tasksResult.items;
    if (!tasks || tasks.length === 0) return;
    
    tasks.forEach(task => {
      if (processedTaskIds.has(task.id)) return;
      
      let isToday = false;
      let isOverdue = false;
      
      // Match against your "1-Today" list or anything containing "Today"
      const isTodayList = list.title.toLowerCase().includes("today");
      
      if (task.due) {
        // Google Tasks due date is returned as midnight UTC string: "YYYY-MM-DDT00:00:00.000Z"
        const taskDueStr = task.due.substring(0, 10);
        
        if (taskDueStr === todayStr) {
          isToday = true;
        } else if (taskDueStr < todayStr) {
          isOverdue = true;
        }
      }
      
      // If a task is in your "1-Today" execution list, treat it as a Today task (even without a due date)
      if (isTodayList && !isOverdue) {
        isToday = true;
      }
      
      if (isToday) {
        todayTasks.push(task.title);
        processedTaskIds.add(task.id);
      } else if (isOverdue) {
        const dateLabel = task.due ? ` (Due: ${task.due.substring(5, 10)})` : '';
        overdueTasks.push(`${task.title} <span style="color:#ba1a1a; font-size:12px;">${dateLabel}</span>`);
        processedTaskIds.add(task.id);
      }
    });
  });
  
  // Only trigger the email if you actually have things to clear out
  if (todayTasks.length > 0 || overdueTasks.length > 0) {
    sendEmailNotification(todayTasks, overdueTasks);
  } else {
    Logger.log("No tasks scheduled for today and zero overdue tasks. Enjoy your open schedule!");
  }
}

function sendEmailNotification(todayTasks, overdueTasks) {
  // Automatically detects and uses your own account email
  const emailAddress = Session.getActiveUser().getEmail(); 
  const subject = "☀️ Your Daily Agenda Summary";
  
  let htmlBody = `
    <div style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; max-width: 500px; margin: 0 auto; padding: 24px; border: 1px solid #e0e2e6; border-radius: 12px; background-color: #ffffff;">
      <h2 style="color: #1a73e8; margin-top: 0; margin-bottom: 8px; font-weight: 600;">Good Morning! ☀️</h2>
      <p style="color: #5f6368; font-size: 14px; margin-bottom: 24px;">Here is your clean agenda update for today.</p>
  `;
  
  // Overdue Section
  if (overdueTasks.length > 0) {
    htmlBody += `
      <div style="margin-bottom: 24px; padding: 16px; background-color: #fce8e6; border-left: 4px solid #d93025; border-radius: 4px;">
        <h3 style="color: #b31412; margin-top: 0; margin-bottom: 12px; font-size: 16px; font-weight: 600;">⚠️ Overdue Attention Needed</h3>
        <ul style="padding-left: 20px; margin: 0; color: #202124; font-size: 14px; line-height: 1.5;">
    `;
    overdueTasks.forEach(task => {
      htmlBody += `<li style="margin-bottom: 6px;">${task}</li>`;
    });
    htmlBody += `</ul></div>`;
  }
  
  // Today's Focus Section
  htmlBody += `
    <div style="padding: 16px; background-color: #e6f4ea; border-left: 4px solid #137333; border-radius: 4px;">
      <h3 style="color: #137333; margin-top: 0; margin-bottom: 12px; font-size: 16px; font-weight: 600;">🎯 Today's Target Focus</h3>
  `;
  
  if (todayTasks.length > 0) {
    htmlBody += `<ul style="padding-left: 20px; margin: 0; color: #202124; font-size: 14px; line-height: 1.5;">`;
    todayTasks.forEach(task => {
      htmlBody += `<li style="margin-bottom: 6px;">${task}</li>`;
    });
    htmlBody += `</ul>`;
  } else {
    htmlBody += `<p style="color: #137333; font-style: italic; margin: 0; font-size: 14px;">Your 1-Today list is clear right now!</p>`;
  }
  
  htmlBody += `
    </div>
    <hr style="border: 0; border-top: 1px solid #e0e2e6; margin: 24px 0;">
    <p style="color: #9aa0a6; font-size: 11px; text-align: center; margin: 0;">Automated smoothly via Google Apps Script</p>
    </div>
  `;
  
  GmailApp.sendEmail(emailAddress, subject, "Please enable HTML views to see your daily agenda dashboard.", { htmlBody: htmlBody });
}
```

---

## 🧪 Step 3: Run a Test Match

1. Click the **Save** floppy-disk icon at the top toolbar.
2. Ensure `sendDailyAgenda` is selected in the run drop-down menu, then click **Run**.
3. **Authorization Request:** A window will pop up asking for permissions because the script acts on your behalf to read your tasks and send emails. 
   * Click *Review Permissions*, choose your account, click *Advanced* (at the bottom), and then click *Go to Daily Task Reminder (unsafe)* to grant access.
4. Check your Gmail inbox—you should see a stylized summary arriving instantly!

---

## ⏰ Step 4: Automate It for 6:00 AM Daily

Now that the script works perfectly, you can tell Google to run it while you're asleep.

1. On the left sidebar of the Apps Script dashboard, click the **Triggers** icon (it looks like an alarm clock ⏰).
2. Click the blue **Add Trigger** button in the bottom right corner.
3. Configure these exact settings:
   * **Choose which function to run:** `sendDailyAgenda`
   * **Select event source:** `Time-driven`
   * **Select type of time based trigger:** `Day timer`
   * **Select time of day:** `5 AM to 6 AM`
4. Click **Save**.

Your custom email dashboard is now fully functional. Every morning, Google will dynamically crawl your records and deliver an isolated, actionable roadmap before you start your day.