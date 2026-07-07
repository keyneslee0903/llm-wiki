---
source_url: https://mixroute.ai/zh-hant/blog/hermes-agent-multi-agent-setup/
ingested: 2026-07-08
sha256: cb5db978731ed929
---

Hermes Agent Multi-Agent Setup Guide | MixRoute
跳至主要內容
QR Code
掃描以分享
查看所有書籤
One
Hermes
agent doing your research, your analysis, and your reporting at the same time does all three badly. Its context fills with noise. It confuses what to find with what to think about with what to tell you. The work gets worse with every job you add, not because the model got dumber, but because no single agent can hold three jobs cleanly at once. The intermediate fix is a Hermes Agent multi-agent setup, and that is what this guide builds.
It splits the work across three focused profiles sharing one knowledge base. It runs on a schedule, gets deeper every night, and delivers a five bullet brief every morning.
The short version.
A Hermes Agent multi-agent setup splits research across three isolated profiles instead of overloading one. A Scout finds signals, an Analyst synthesizes them into a shared Obsidian wiki, and a Briefer delivers a daily brief. Each profile has its own model, memory, and schedule, coordinated through one shared knowledge base.
TL;DR.
One Hermes agent doing search, analysis, and reporting at once does all three badly. A Hermes Agent multi-agent setup splits the work across three focused profiles (Scout, Analyst, Briefer) that share one Obsidian knowledge base, producing compounding results for roughly $20 to $27 a month. This guide covers the full build: profiles, SOUL.md files, cron jobs, the wakeAgent cost gate, the shared wiki, and an optional NotebookLM connection.
If this is your first time building a Hermes bot, start with the beginner guide first, get one agent running, then come back. Everything below assumes you already have a working agent and a model gateway wired.
This is for you if you already run one Hermes agent and want it to do more without babysitting it. Founders tracking competitors. Creators who need daily research in one niche. Operators watching several markets for clients. Anyone still spending thirty minutes a day reading newsletters and checking the same five tabs.
Why a Hermes Agent multi-agent setup beats one profile
In Hermes, each separate agent is a profile, and a profile is fully its own thing: its own personality, its own model, its own memory, its own skills, its own schedule. The beginner setup gives you one profile and one good model. That carries you a long way. It stops working the moment you give that single profile a second real job.
The reason has a name: context. Everything a profile is working on sits in one window. Pile three jobs in there and they bleed into each other. By day three the agent is reading this morning’s task through a fog of last week’s research.
Three problems follow, and they compound.
The context fills with noise. Today’s brief has nothing to do with the forty sources the agent pulled last Tuesday, but they are all still in the window, still costing tokens, still pulling attention.
The skills pile up. By week two one profile has skills for web search, paper parsing, Telegram formatting, competitor diffing, and a dozen more. Tool Search helps, but you are still asking one identity to be a researcher, an analyst, and a secretary.
The judgment blurs. The profile that should cast a wide net for signals is also deciding what matters and writing the tight summary. Those are opposite muscles. Wide and loose for finding. Narrow and strict for reporting. One profile flips between them badly.
Split the work and each problem melts away. The finder only finds. The analyst only analyzes. The reporter only reports. Each stays clean because each stays small. That is the whole idea behind a Hermes Agent multi-agent setup: one job per agent, one shared memory between them.
The three roles: Scout, Analyst, Briefer
The department in a Hermes Agent multi-agent setup is three profiles. The names tell you the jobs.
Profile
Job
Writes to
Model tier
Scout
Finds signals, no analysis
the inbox folder only
cheap, fast
Analyst
Turns findings into linked, confidence tagged notes
the shared wiki
strong reasoning
Briefer
Reads the wiki, sends the morning brief
Telegram
cheap, fast
Scout finds signals. It checks your sources on a schedule and drops what it finds into an inbox folder as plain files. No analysis, no opinion, no summary. Raw signal in, raw signal saved. Scout is busy and a little dumb on purpose, which is exactly what you want from a finder.
Analyst makes sense of things. It picks up whatever Scout dropped in the inbox, runs real synthesis, and writes proper notes into the shared wiki: what is true, what is likely, what contradicts what. This is the one profile where model quality genuinely matters, because the Briefer’s morning output is built on what the Analyst wrote overnight.
Briefer tells you what matters. Every morning it reads what the Analyst wrote, checks it against what you are working on this week, and sends five bullets. Most important first. One finding, why it matters, what to do about it. No research, no analysis. It reads and it reports.
How to create multiple Hermes Agent profiles
The fastest path is the Hermes Desktop app and its Profile Builder, a five step wizard: Identity, Model, Skills, MCPs, Review. Run it three times, once per profile.
hermes dashboard → Profiles → Build
If you prefer the terminal, the same thing is three lines:
hermes profile create scout
hermes profile create analyst
hermes profile create briefer
Both paths land in the same place. Desktop is friendlier the first time. The CLI is faster once you know what you want.
This is also where wiring one model gateway on day one pays off. You are about to create three agents, and two of them want different models. With a single gateway key already in place, you open no new accounts and generate no new keys. Every profile reaches every model through the one key you have, and switching a profile from one model to another is a one word edit.
Writing a SOUL.md for each profile
The SOUL.md is read at the start of every single turn, so a long one gets re-read forever and quietly burns tokens. Keep each profile’s soul tight, around fifty lines. The difference at this stage is that each soul is ruthless about one job and openly forbids the other two.
Scout’s soul. Terse. Finds and saves, nothing else.
# Soul
You are a research scout. Your job is to find signals.
You do not analyze. You do not summarize.
You find relevant information and you save it.

## Voice
Terse. File names and one line descriptions only.

## Operations
Search the sources listed in your cron jobs.
For each finding, save the full text as a markdown file
to ~/research/inbox/ with the format:
YYYY-MM-DD-source-keyword.md
Put the source URL on the first line.

## Restrictions
Never analyze or synthesize what you find.
Never write more than 3 lines of your own text per file.
Never touch files written by another profile.
A cheap, fast model fits Scout’s high volume, low reasoning work. One note on X: a plain cheap model cannot search X on its own. If watching X matters, either add the xurl skill or run Scout on a Grok model, which has X search built in. Web, arXiv, and RSS work with any model.
Analyst’s soul. Precise. Synthesizes, tags confidence, flags conflicts.
# Soul
You are a research analyst. Your job is to synthesize.
You turn raw findings into structured knowledge.
You verify claims. You flag contradictions.

## Voice
Precise. Evidence based. Tag every claim with a
confidence level: [verified] [likely] [unverified] [conflicting].

## Operations
Process files from ~/research/inbox/.
For each batch:
1. Synthesize across the sources (through NotebookLM if connected,
   otherwise run it directly with /goal).
2. Write structured notes into ~/obsidian-wiki/ using the LLM Wiki skill.
3. Tag each entry with a confidence level.
4. Flag anything that contradicts an existing entry.
5. Move processed files to ~/research/processed/.

## Restrictions
Never present an unverified claim as a fact.
Never write to the wiki without a source.
Never delete a wiki entry. Update it or flag it.
This is the profile to spend on. A strong reasoning model writes the knowledge the whole system stands on, so weak reasoning here weakens everything downstream.
Briefer’s soul. Short. Reads and reports.
# Soul
You are a briefing officer. Your job is a short,
prioritized, actionable morning brief.
You do not research. You do not analyze.
You read what the Analyst wrote and tell me what matters today.

## Voice
5 bullets, maximum. Each bullet: one finding,
why it matters to me, a suggested action.

## Operations
1. Read the wiki entries from the last 24 hours.
2. Cross reference with my current projects.
3. Send a 5 bullet brief to Telegram, most important first.
4. End with total token spend this week.

## Restrictions
Never go past 5 bullets.
Never repeat yesterday's items unless the status changed.
A cheap model handles the Briefer’s one short daily job well.
How to schedule Hermes Agent profiles with cron jobs
A cron job is a task on a timer. This is what turns three idle profiles into a department that works while you sleep. Write them by hand, or tell each profile in plain language and let Hermes build the job, the folders, and the config.
Scout runs often, because finding is cheap and you want fresh signal:
/cron add "every 3h" \
  --prompt "Search the web for posts about [your niche keywords]
  from the last 3 hours. Save each finding as a markdown file
  to ~/research/inbox/. Put the source URL on the first line." \
  --deliver telegram
Analyst runs once a day, and only works if Scout left it something:
/cron add "every day 10am" \
  --script check-inbox.py \
  --prompt "Process every file in ~/research/inbox/. Synthesize them.
  Write structured notes to the wiki. Tag confidence levels.
  Flag contradictions. Move processed files to ~/research/processed/." \
  --deliver telegram
Briefer runs once a day, before you have had coffee:
/cron add "every day 8am" \
  --prompt "Read the wiki entries from the last 24 hours. Cross
  reference with my current projects. Send a 5 bullet prioritized
  brief to Telegram, most important first. End with token spend." \
  --deliver telegram
All three profiles deliver to the same Telegram bot. One bot, three agents, one chat.
What is a wakeAgent gate, and why it keeps the cost near zero
Most of the time there is nothing new to process. The inbox is empty. If a profile wakes on schedule, looks at an empty folder, and goes back to sleep, you still paid for it to wake and look. Across several profiles and daily runs, that adds up.
A wakeAgent gate is a tiny script that runs before the agent and decides whether waking it is worth it. Empty inbox, the agent never wakes and you pay nothing. Files waiting, the agent and its model come online only then.
Save this as
~/.hermes/scripts/check-inbox.py
:
#!/usr/bin/env python3
import os, json

inbox = os.path.expanduser("~/research/inbox")
files = [f for f in os.listdir(inbox) if f.endswith('.md')] if os.path.exists(inbox) else []

if files:
    print(json.dumps({"wakeAgent": True}))
    print(f"{len(files)} new files in inbox:")
    for f in files:
        print(f"  {f}")
else:
    print(json.dumps({"wakeAgent": False}))
The
--script check-inbox.py
flag on the Analyst’s cron runs this first. The same pattern works for any monitoring job. Watching competitor pages? Hash each page and only wake the agent when the hash changes. This single flag is why a system that appears to run all day costs about the same as a few coffees a month.
Using Obsidian and the LLM Wiki as a shared knowledge base
The three profiles share exactly one thing: a folder. Obsidian stores everything as plain markdown files in one vault and draws the links between notes as a visible web. That web is the shared memory of the department.
Hermes ships with a bundled LLM Wiki skill, built on Andrej Karpathy’s wiki pattern. It files notes, keeps cross references linked, and flags when a new note contradicts an old one. You turn it on per profile.
A vault structure that keeps everyone in their lane:
~/obsidian-wiki/
├── inbox/            raw findings from Scout
├── sources/          processed source pages
├── synthesis/        the Analyst's structured notes
├── briefs/           archived morning briefs
├── entities/         people, companies, products
└── contradictions/   flagged conflicts
Point every profile at the same folder in the dashboard config:
WIKI_PATH = ~/obsidian-wiki
OBSIDIAN_VAULT_PATH = ~/obsidian-wiki
On first run into an empty vault, the wiki skill asks for your domain and builds a small schema file, a tag taxonomy for your niche, then uses it to file everything consistently.
Keep the ownership strict. Scout writes only to inbox. Analyst reads inbox and writes to sources and synthesis. Briefer only reads synthesis. When each profile owns its folders, you debug the whole system by opening a folder and looking. When everyone writes everywhere, you cannot.
How to connect Hermes Agent to NotebookLM
The Analyst can synthesize through its own model, which is already good. NotebookLM makes it deeper by ingesting the whole pile of sources and drawing connections across all of them at once, rather than reasoning over what fits in one context window.
Connect it as an MCP server. The tool is notebooklm-mcp-cli:
pip install notebooklm-mcp-cli
nlm login
Then add it in the dashboard:
hermes dashboard → MCP → Add Server
Name: notebooklm
Transport: stdio
Command: nlm
Arguments: mcp serve
Save → Test Connection
Assign it to the Analyst profile only, since it is the only profile that needs it.
One honest caveat. NotebookLM has no official public API for the consumer product, so this community tool drives it through browser automation. It works well, but if Google changes something internal, the tool can break until it is updated. Plan for it.
Give the Analyst a fallback, written into its soul: if NotebookLM is not reachable, run synthesis directly with
/goal
. NotebookLM makes the Analyst deeper. The fallback makes it reliable.
How the three profiles coordinate
Coordination in a Hermes Agent multi-agent setup is just files plus the wakeAgent gate. A straight pipeline like this needs no project board.
Scout, every 3 hours:    searches, drops files into the inbox, pings Telegram
Analyst, every day 10am: gate checks the inbox, synthesizes, writes the wiki,
                         moves processed files out
Briefer, every day 8am:  reads the wiki, sends the 5 bullet brief
Scout never reads. Briefer never writes to the inbox. Each profile touches its own folders, so the handoffs cannot collide. The inbox is the only meeting point, and it is a folder you can open and inspect.
Hermes does have a heavier coordination tool, a kanban board with a dispatcher, useful once you have agents whose work is not a straight line. For three profiles in a pipeline, files are simpler, cheaper, and easier to debug. Add the board the day you outgrow the line.
What a Hermes Agent multi-agent setup costs to run
Running a Hermes Agent multi-agent setup costs roughly $20 to $27 a month depending on which models you put in each profile. For contrast, a part time research assistant runs far more and does not work at 3am. The setup stays cheap because of three things: cheap models on the two light profiles, one strong model only where it earns its keep, and wakeAgent gates so nothing runs on an empty inbox.
The part that quietly makes this affordable is how you power the three models. Two profiles want a cheap, fast model. One wants a strong reasoning model. Instead of opening a separate account with each provider, run all three through one AI gateway and you get a single key for every model.
This is where
MixRoute
fits the build. It is an AI API gateway: one OpenAI-compatible API that routes across 200+ models from every major provider (Claude, GPT, Gemini, DeepSeek, and more). One key covers all three profiles, it charges no markup over the providers’ own prices, and it falls over to another provider automatically if one has a bad night, so a single outage does not take your whole department down. Switching a profile’s model is a one word edit, not a migration, because you never wired the provider directly.
A few habits keep the bill small. Start with three or four Scout crons, not ten, because every Scout run feeds the inbox and every inbox file is work for the Analyst. Watch the spend with
/usage
, and the Briefer ends every brief with the week’s total anyway. Cost scales with how much you ask for, so keep Scout focused and keep the gate in place.
What to expect: day 1, week 2, month 1
A Hermes Agent multi-agent setup is cold on the first day and impressive by the end of the month. That curve is the point.
Day 1. Three profiles created, three short souls written, a handful of crons running. The vault is empty. The first brief is broad and generic. Useful, not impressive.
Week 2. Scout has pulled fifty to a hundred sources. The Analyst has written thirty or forty wiki entries. The graph shows its first cross references. The briefs start naming your actual projects, and one morning the Analyst flags something you would never have searched for. That is the moment it earns its keep.
Month 1. Two hundred plus entries, linked and tagged. Contradictions tracked in their own folder. You have told Scout to stop finding certain things, so the noise is down. The brief reads like it was written by someone who knows your work, because in a real sense it now does. The system starts handing you things you did not ask for. That is the compounding.
Common mistakes to avoid
Most mistakes in a Hermes Agent multi-agent setup come down to one instinct: overloading a single profile instead of splitting the work. The temptation is always to add one more job to a profile that already exists. Resist it. New job, new profile.
Skipping the wakeAgent gate. Without it, every profile bills you to look at an empty folder. With it, idle is free.
Writing a fat SOUL.md for each role. Each one is read every turn. Keep each tight, single job, the other jobs forbidden.
Letting every profile write everywhere in the vault. Give each profile its own folders and keep the lanes strict, or you lose the thing that makes this debuggable.
Reaching for the kanban board too early. A three step pipeline does not need a dispatcher.
Trusting the confidence tags blindly. The Analyst tags claims using its own judgment. It is not independently fact checking the world. Treat the brief as a sharp starting point and check anything important before acting on it.
Frequently asked questions
Can you run multiple agents in Hermes Agent?
Yes. Each agent is a profile, and a profile is fully isolated: its own SOUL.md, model, memory, skills, and schedule. You can run several on one machine, and they share nothing unless you point them at a common folder.
How many profiles do you need for a research setup?
Three covers a full research pipeline. Scout finds signals, Analyst synthesizes them into a shared wiki, and Briefer delivers the morning brief. Each does one job, which keeps each one’s context clean.
How much does a Hermes Agent multi-agent setup cost?
Roughly $20 to $27 a month depending on the models you choose. Two profiles run on a cheap model and one on a stronger model, and wakeAgent gates keep idle runs free. Running all three models through one gateway keeps it to a single bill.
What is a wakeAgent gate?
A small script that runs before a scheduled profile and checks whether there is anything to do. If the inbox is empty the agent never wakes, so monitoring costs nothing until there is real work.
Do you need NotebookLM for this setup?
No. The Analyst synthesizes through its own model by default. NotebookLM adds deeper cross source synthesis when connected, but it relies on a browser automation wrapper, so always keep a direct
/goal
fallback in the Analyst’s soul.
Which model should each profile use?
Scout and Briefer do light, high volume work, so a cheap, fast model fits. The Analyst does the heavy synthesis, so give it a strong reasoning model. Pick the current names from your gateway rather than pinning a model that will be renamed.
Do the three profiles share memory?
Not by default. They share only the Obsidian vault they all point to. Scout writes to the inbox, Analyst writes the synthesis, Briefer reads it. That single shared folder is the entire coordination layer.
The bottom line
A Hermes Agent multi-agent setup is the jump from one overloaded generalist to a small team of specialists that share one memory. Scout watches your world on a schedule, Analyst turns findings into linked, confidence tagged knowledge overnight, and Briefer hands you five bullets every morning that already know what you care about. One job per agent, one shared memory between them, and the work compounds.
All three profiles need models, and the cleanest way to power them is one gateway instead of a pile of provider accounts. MixRoute gives you a single OpenAI-compatible key for 200+ models, no markup, automatic failover, and a one word model swap. Five minutes from signup to first call. Start building on MixRoute.
分享
書籤
查看全部
搜尋文章
文章分類
AI
熱門文章
01
How to Fix 429 Rate Limit Errors When Using AI APIs
02
Hermes Agent Custom Endpoint: Run Any Model Through One API
03
OpenRouter Alternatives in 2026: 5 Options Worth Considering
04
How to Pay for AI APIs With Crypto (USDT and USDC) in 2026
05
DeepSeek vs ChatGPT: Which AI Model Should You Actually Use? (2026)
訂閱更新
加入 24,000+ 位讀者，每月深入探討科技架構的未來趨勢。
立即訂閱
訂閱其他電子郵件