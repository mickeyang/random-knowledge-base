> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0OfMEYqiIhk

# you asked

message time: 2026-07-10 22:40:16

I am using quite a few AI tools now, including OpenAI chatgpt, Google Gemini, Tecent Yuanbao, and Zhipu Z-AI. I found it's hard to navigate. sometimes I forgot where I asked or searched question A and B. help me to design a system and flow to make the most use of all of them.

---

# yuanbao response

This is a common "AI sprawl" problem — each model has different strengths, but your brain shouldn't be the index. Here's a practical system to tame it, without adding overhead that defeats the purpose.

## 1. First: Pick a "Home Base" (Single Source of Truth)

Don't try to keep all four as equals. Pick **one** as your default capture/logging tool, the others as specialists.

- **Recommended home base**: ChatGPT (Projects + Memory) *or* a Notion/Obsidian note called `AI Workspace`. Reason: ChatGPT's "Projects" lets you organize by topic and persist context; if you prefer non-AI-owned, use a note app.
- Everything starts here: your question, your draft, your thinking. Then you *branch out* if needed.

## 2. Routing Logic — When to Use Which

Don't ask "which is better." Ask "what is this task."

| Task Type | Best Tool | Why |
|---|---|---|
| General reasoning, coding, writing, longest memory | **ChatGPT (OpenAI)** | Most consistent, Projects/Memory, best ecosystem |
| Huge docs, video, Deep Research, 1M+ context | **Gemini** | Context window + native file parsing |
| Chinese-language tasks, WeChat/China ecosystem, local refs | **Yuanbao (腾讯元宝)** | Hunyuan optimized for CN, integrated with WeChat |
| Chinese academic, GLM model family, local compliance, open-weight curious | **Zhipu Z-AI (智谱清言/GLM)** | Strong on Chinese reasoning + long docs, good API story |
| Need fresh web data | Gemini / Yuanbao (both search well in different regions) |
| Creative cross-pollination / hard decision | Run same prompt across 2, compare |

**Simple rule of thumb**: ChatGPT = daily driver. Gemini = big files / research. Yuanbao = Chinese context. Zhipu = Chinese academic / backup.

## 3. The Tracking System (Low-Friction)

You said you forget where you asked what. Two levels, pick whichever fits:

### Light: Naming Convention
In each AI, start every chat with a tag in the title:
- `WRK – Q3 strategy`
- `CN – 合同草案`
- `CODE – python scraper`

After a session, if it produced something valuable → copy the link + 1-line summary into a central list.

### Heavy (recommended if you use AI professionally): A 1-page "AI Log"
A Notion/Obsidian sheet, 3 columns:

| Date | Question/Topic | Where (model) | Link | Outcome/Takeaway |
|---|---|---|---|---|

Takes 30 seconds per session. The payoff: you can Ctrl+F "contract" and see all 3 places you discussed it.

## 4. Workflow: The "Branch & Merge" Pattern

Instead of re-asking the same thing 4 times blindly:

1. **Draft in home base** (ChatGPT Project). Get 80% there.
2. **Export the prompt** → paste into Gemini if you need it to read 200 pages of PDFs, or into Yuanbao if it's CN-law related.
3. **Bring the best chunk back** to home base. That's your single source of truth.
4. For critical decisions → run the same prompt in 2 models, compare (ChatGPT vs Gemini often enough).

## 5. Reduce Friction: Little Tricks

- **Browser bookmarks bar**: pin each AI, name them `AI-C`, `AI-G`, `AI-Y`, `AI-Z`.
- **Prompt snippet tool** (Text Blaze / Raycast / Espanso): save your "standard prompt template" so you can fire it into any model without retyping.
- **ChatGPT Projects**: create one per recurring domain (e.g., "Work", "CN Stuff", "Coding"). Move chats there. This alone solves 50% of your "where did I ask this?" problem.
- **Yuanbao & Zhipu**: both have WeChat mini programs — useful for on-the-go CN queries; just remember they're ephemeral, so export important stuff.

## 6. What NOT to Do

- Don't try to use all 4 equally — you'll burn mental energy deciding.
- Don't rely on each AI's own memory across tools — they don't talk. Your Notion/Obsidian is the only cross-model memory.
- Don't over-log. If the log becomes a chore, drop to just the naming convention + bookmark favorites.

---

Want me to draw you a **visual flowchart** of this (home base → branch → merge), or tailor the routing table to your actual use cases (e.g., are you using these for coding / writing / Chinese biz / research)?

---

# you asked

message time: 2026-07-10 23:27:55

what's the usage of openai chatgpt feature `add to project sources`

---

# yuanbao response

"Add to project sources" is a ChatGPT **Projects** feature — it's basically turning a chat output (or an external link) into **persistent, reusable context** for every future chat inside that project. Here's what it actually does and why it matters for your setup.

## What "Project Sources" Are (vs. Uploading a File)

Inside a ChatGPT Project, you have three ways to feed it context:

| Method | Scope | Use for |
|---|---|---|
| **Upload file** | Attached to one chat | One-off doc you need *now* |
| **Project instructions** | All chats in project | Behavior rules ("write like X", "always output JSON") |
| **Project Sources** | All chats in project, **reusable** | Background that *every* chat in the project should know |

Sources are the "desk drawer" — things the project keeps handy permanently: reference docs, style guides, a glossary, a previous summary you liked.

## What You Can Add as a Source

1. **Files / pasted text** — stable docs you want the project to keep (upload once, all chats see it)
2. **App links (Google Drive files/folders, Slack channels)** — "living" sources; OpenAI calls this **Living Sources** (Feb 2026 update). Instead of downloading a GDocs and uploading, just paste the link — ChatGPT can pull from it inside the project [citation:1].
3. **A ChatGPT response itself** — this is the button you're asking about. If a chat produces a great summary / decision doc / draft, click the message menu → **"Save to project" / "Add to project sources"**. That output becomes a source *other future chats in the project can reference* [citation:2].

## Why It Matters for Your "AI Sprawl" Problem

Remember I suggested ChatGPT as your **home base**? Project Sources is *why*.

Example flow:
- You ask Gemini to digest a 200-page PDF → get a solid summary back.
- Paste that summary into a ChatGPT Project chat → click **"Add to project sources"**.
- Now *every* new chat in that project (coding, drafting, brainstorming) inherits that summary. You never re-upload, never re-explain.

Or:
- You have a recurring task (say, Chinese contract review via Yuanbao). Take Yuanbao's output, bring the key points into a ChatGPT Project, save as source. Now ChatGPT knows your contract style without you re-pasting each time.

## Common Mistakes to Avoid

- **Pasting the same material into 5 separate chats** instead of saving once as a source. That's exactly what Sources is for [citation:2].
- **Confusing Project Instructions vs Sources**: Instructions = "how to behave" (tone, format, constraints). Sources = "what to know" (docs, facts, prior outputs). Both live at project level but do different jobs.
- **Leaving great outputs buried in one chat** — if a response becomes "working knowledge," promote it to Sources immediately, or you'll re-generate it next week and waste tokens + time.

---

In short: **"Add to project sources" turns a one-time output (or external doc/link) into project-level memory.** For your multi-AI system, it's the glue — stuff you get from Gemini/Yuanbao/Zhipu gets distilled and parked here, and ChatGPT becomes the only place you need to re-open.

Want a concrete example of how to structure *one* Project for your cross-AI workflow (e.g. a "Research" project with Sources from Gemini, a "CN Work" project with Sources from Yuanbao)?