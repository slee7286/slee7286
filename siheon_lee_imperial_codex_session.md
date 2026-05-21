You are an engineering agent working in a React (TypeScript) monorepo for the hackeurope2026 SLT therapy web app. Context: I just pulled UI changes from GitHub on main. Before pulling, I had implemented: A practice session UI wired to the therapy/game engine (useTherapyEngine / GameTab / practice view). A Bing image multiple‑choice UI for "picture_description" therapy blocks (image choices, clickable, one correct answer). After the merge, both of these features stopped working (broken navigation / components / imports / state wiring). I want you to: Inspect the current codebase structure. Re‑wire the practice session and Bing image multiple‑choice UI so they work again with the new UI layout that exists on GitHub. Keep the new UI layout and components from main intact; only integrate the practice functionality into them. Do NOT invent a new UI framework; work within the current React/TS app. FIRST: ASK CLARIFYING QUESTIONS Before writing code, ask me: What branch should we treat as the source of truth for UI? main (pulled from GitHub) or practiceadd or another branch? Where is the current main UI shell now (after pulling)? For example: frontend/src/App.tsx frontend/src/components/Layout.tsx Which component currently defines navigation / tabs (e.g. Admin vs Patient vs Practice)? Which files currently contain my earlier practice/game work? Likely: frontend/src/components/GameTab.tsx frontend/src/hooks/useTherapyEngine.ts src/routes/imageSearch.ts Confirm the exact paths and whether they still exist after the pull. How is the therapy engine exported now? Path and name of the hook/provider to control the practice session, e.g.: frontend/src/hooks/useTherapyEngine.ts Its API: state and actions object. How is the Bing image search endpoint supposed to be exposed? Path: e.g. src/routes/imageSearch.ts (Express route), or Next.js API route, etc. Confirm that we still want /api/image-search (or tell me the desired route). After I answer, proceed with integration. GOALS Practice session UI working again There must be a clear way in the current UI (tabs, navigation, etc.) to open a Practice / Game screen that: Uses the therapy engine (useTherapyEngine or TherapyEngineProvider) to: Load a therapy plan JSON. Start the session. Present current task (block type, prompt). Accept text / voice / click answers. Show feedback and progress. Works with the new layout and styling from main. Bing multiple‑choice picture UI working again For "picture_description" blocks: Call the Bing image search endpoint using the prompt/topic. Render multiple clickable images. Only one is considered the “correct” answer. Clicking an image sends an appropriate value into actions.submitText(...) so the engine can score it. This must be integrated into the new UI, not as a detached old screen. Do not break existing Check‑in / Admin UI The new main branch added or changed several components: frontend/src/App.tsx frontend/src/components/Layout.tsx frontend/src/components/AdminVoiceSettings.tsx frontend/src/components/SessionHistory.tsx etc. You must preserve those and integrate the practice session as a new route/tab/section, not replace them. IMPLEMENTATION REQUIREMENTS A. Re‑wire the Practice / Game screen into the new UI Locate the main layout / routing component after the pull Probably: frontend/src/App.tsx and frontend/src/components/Layout.tsx. Add a new navigation item or route for the Practice session For example: A “Practice” or “Game” tab/button in whatever navigation Layout is now using. It should render a component, e.g. PracticeSessionPage or GameTab. Reuse or re‑create the practice UI component If frontend/src/components/GameTab.tsx already exists: Update it to conform to the new layout (import the same CSS, layout wrappers, etc.). Make sure it imports useTherapyEngine from the right place. If this component was lost/renamed after pull: Re‑create GameTab / PracticeSession using the existing hook and engine, but following the current style of other components. Hook up the therapy engine In the Practice/Game component: Use something like: ts const { state, actions } = useTherapyEngine(); On mount: Load a therapy plan (from wherever the app now exposes it—context, API call, or a placeholder). Call actions.loadPlan(plan) then actions.start() when ready. Render the UI using the existing state machine: Show: Block / item progress, Task info, Prompt (or TTS/voice as previously implemented). Provide: Text input as fallback, Submit / Next buttons. Ensure compatibility with new TTS/STT / layout The new UI likely enhanced TTS (ElevenLabs) and STT hooks. Reuse the same TTS/STT hooks the Check‑in section uses (do not create new ones). Ensure the practice screen shares consistent styling and layout with other patient‑facing components. B. Restore the Bing image multiple‑choice behaviour Confirm or re‑create the image search backend route Locate src/routes/imageSearch.ts (or similar). Verify it is registered in the Express server (e.g. in src/server.ts). Confirm it calls Bing Image Search using process.env.BING_IMAGE_API_KEY and returns an array of simplified results: { url, thumbnailUrl, title, width, height }. Re‑connect the frontend to the backend route In the Practice/Game component (or a child component for picture tasks): When state.currentTask.blockType === "picture_description": Use useEffect to: Build a search query from promptText + topic/difficulty. Call the image search API (e.g. via fetch('/api/image-search?query=...&count=4') or a helper). Store the images in local state. Render multiple clickable images For each image result, render something like: tsx <button onClick={() => handleImageChoice(i)} disabled={status !== 'awaitingAnswer'}> <img src={image.thumbnailUrl || image.url} alt={Option ${i + 1}} /> </button> When clicked: Compute a canonical answer string (e.g. "choice_0", "choice_1", etc., or the image URL). Call actions.submitText(answerString). Ensure the therapy plan expects that answer If the plan’s items[].answer is something like "choice_0": Use the same mapping locally so the correct click leads to that answer. If the plan expects the actual prompt or image description: Provide a small mapping/adapter between clicked index and the expected string (document it in code comments). Combine with other UI features For non‑picture_description tasks, show the existing input / TTS / STT UI as before. For picture_description, hide the normal text input and show only the image choices and feedback. CODE OUTPUT EXPECTATIONS Once the clarifying questions are answered and you’ve inspected the repo, your response should: Summarize: Which components are now used as entry points (App.tsx, Layout.tsx). Where the practice/game UI will live (file path and component name). Where the Bing image route is defined and what its URL is. Provide concrete code updates in separate sections: Updated main UI / navigation file (e.g. frontend/src/App.tsx or frontend/src/components/Layout.tsx) showing: How the Practice/Game tab/route is added. Updated or new Practice/Game component file (e.g. frontend/src/components/GameTab.tsx) showing: Hook into useTherapyEngine. UI logic for block types. Picture_description handling with image choices. Updated src/server.ts (or equivalent) showing that: The Bing imageSearch route is registered. Any paths/names reflect the current structure. If needed, a small frontend/src/api/imageSearchClient.ts helper (or similar) that wraps the fetch call. Keep styling minimal and consistent with new UI; do not introduce extra libraries. DO NOT Do not delete or radically change the new components from main (AdminVoiceSettings, SessionHistory, etc.). Do not re‑implement the therapy engine reducer or state machine—reuse the existing one. Do not implement direct Bing API calls in the browser if the app is already using a backend route. Do not introduce new design patterns; follow the style of existing components. Begin by asking the clarifying questions, then explain your plan, then show the code.
No tasks in progress


I want the text to move a tiny bit up from the current position in the practice session part

I’ll make a minimal CSS tweak in practice mode to nudge the text slightly upward, then run a quick frontend build check.

Adjusted. I moved the practice-session text slightly up by changing:

frontend/src/styles/theme.css
.session-flow--practice .speech-orb-caption { top: 50%; } → top: 46%;
Build check passed (npm run build).



You are updating the app’s behavior related to practice sessions and Admin voice settings.

There are three main changes:

1. Add “Number of Questions” control in Admin voice settings
Goal

In Admin voice settings, add an option to control the number of questions asked in each practice session.
Requirements

Add a new configurable field in Admin voice settings, something like:

Label: Number of practice questions
Type: numeric input (or slider/dropdown), with:
Minimum value: 4
Maximum value: 50
Default: choose a reasonable default (for example 10), but ensure it respects [4, 50].
Validation:

Do not allow values less than 4 or greater than 50.
If the admin enters an invalid value, either:
Clamp it to the nearest valid value, or
Show a validation error and prevent saving until corrected.
Persistence:

Store this value along with the other Admin voice settings (same mechanism currently used for other admin settings).
Ensure that the practice session logic can read this value when creating a new practice session.
Usage:

When starting a practice session, the system should use this configured number as the target number of questions for that session.
2. Question distribution, with Picture Description as majority (except when 4)
Goal

There are different types of practice questions (for example: Picture Description, and other types).
When the number of questions is not exactly 4, Picture Description questions should be the majority of the questions.
When the number of questions is exactly 4, there should be one of each type of question (no majority type in that case).
Requirements

Identify the existing question types in the practice session (for example):

Picture Description
Other categories (e.g. multiple choice, short answer, etc., use whatever exists in the codebase).
Logic for numQuestions = 4:

Ensure the practice session generator creates exactly one question of each type (assuming there are 4 types).
If there are more than 4 types, adjust as appropriate but follow the intent: when set to 4, produce a balanced set, not skewed toward one type.
The key rule: numQuestions = 4 should give a balanced set (e.g., 1 of each type for the 4 main types).
Logic for numQuestions > 4:

The total number of questions is the admin-selected value.
Ensure Picture Description questions make up the majority of that total.
For example:
At least more than half of the questions should be Picture Description.
The rest can be distributed among other types as they are currently, or in a simple proportional way.
Example:
If numQuestions = 10:
Picture Description: at least 6
Other types: 4 total
Implement simple logic to:
First assign a minimum number of non-picture-description questions (at least one of each other type if that’s desired).
Fill the remainder of slots with Picture Description questions so that Picture Description remains the majority.
Make sure the question generator uses the new admin-configured number (between 4 and 50) and respects the distribution rules above.

3. Improve Picture Description: Unsplash call should be “topic + word”
Current issue

Picture Description questions are not very accurate.
The images being fetched from Unsplash may not match closely enough to the combination of the topic and the singular concept/word.
Goal

When generating a Picture Description question, the Unsplash API call should use a query string built from:
The topic, plus
The singular concept/word for the question.
Example:
Topic: cooking
Word/concept: frying pan
Unsplash query should be: "cooking frying pan"
Requirements

Locate the code that:

Generates Picture Description questions.
Calls the Unsplash API to fetch an image.
Identify the variables representing:

The current topic of the practice session or question.
The specific word or concept used for that Picture Description question.
Build the Unsplash query string as:

"<topic> <word>" (with a space between them), for example:
"cooking frying pan"
"travel airplane"
"music guitar"
Update the Unsplash API request so that:

Instead of using only the topic or only the word, it uses the combined string.
For example (pseudo-code):
js
const query = ${topic} ${word};
// then use query in the Unsplash API call
Keep all existing Unsplash integration behavior the same:

Same endpoint.
Same authentication.
Same handling of the response.
Only change the query parameter.
General constraints
Do not change:

The existing voice selection behavior in Admin voice settings (besides adding the number-of-questions option).
The JSON structure and data flow between check-in and practice sessions.
Any back-end contracts, unless absolutely necessary, and if so, keep them backward compatible.
Only add/modify:

The admin setting for number of practice questions (with min 4, max 50).
The question generation logic to:
Use the admin-configured number of questions.
Produce one of each type when the number is 4.
Make Picture Description questions the majority when the number is greater than 4.
The Unsplash query used for Picture Description questions so it is "topic word".
Summary

Add a “Number of practice questions” setting (min 4, max 50) in Admin voice settings and use it to determine the number of questions in the practice session.
When the number is 4, generate a balanced set with one of each question type.
When the number is greater than 4, ensure Picture Description questions are the majority.
For Picture Description questions, change the Unsplash API call so that the query equals the topic + the singular concept word, like "cooking frying pan".

Show more

I’m going to trace where admin settings are stored, where practice question counts/types are generated, and where Unsplash queries are built, then I’ll implement the three changes end-to-end and run build/tests.

I’ve mapped the code paths. Next I’m implementing the new Admin question-count setting with validation/persistence wiring, then backend session+plan generation support, then the Unsplash topic + word query update.

I’ve finished the frontend wiring. I’m now making the backend accept and persist the optional practice-question count per session, then enforcing count/distribution in plan generation and updating Unsplash query composition.

Context automatically compacted
I’m implementing the three requested changes end-to-end and will first verify the current repo state, fix any broken integration points, then run full build/test validation before I report results. I’m starting by checking modified files and compile blockers.

I confirmed your requested files are modified and on main. Next I’m reviewing the exact edits in the session, admin settings, and picture-image provider paths to fix compile issues and confirm behavior wiring.

I found at least two likely compile blockers in the current patch set: duplicate variable names in src/routes/session.ts and a duplicate combined identifier in src/services/pictureImageProvider.ts. I’m fixing those first, then I’ll run build/tests.

I’ve patched both compile blockers. Next I’m running backend/frontend validation scripts to catch any remaining integration errors in the new question-count and image-query flow.

Build and tests are passing after fixes. I’m doing a final diff check on the touched files to confirm the new admin control, question distribution rules, and topic + word Unsplash query are all wired exactly as requested.

I’ve confirmed the requested behavior is wired and validated in build/tests. I’m collecting exact file references for each change so you can review quickly.

I also ran a quick sanity check for leftover Bing references in the active image flow to ensure the updated picture query path is using the Unsplash-based provider with topic+word input.

Implemented. All three requested changes are now wired and validated.

What changed

Admin setting: Number of practice questions (4..50)
Added to frontend/src/components/AdminVoiceSettings.tsx:24 and UI input at frontend/src/components/AdminVoiceSettings.tsx:236.
Validation blocks invalid values and shows inline error (frontend/src/components/AdminVoiceSettings.tsx:169).
Persisted with other admin settings in localStorage via frontend/src/App.tsx:18, frontend/src/App.tsx:46, frontend/src/App.tsx:97.
Passed into session start/demo-skip via frontend/src/App.tsx:133 and frontend/src/App.tsx:140.
Practice generation uses configured count + distribution rules
Frontend API now sends optional practiceQuestionCount:
frontend/src/api/sessionClient.ts:72
frontend/src/api/sessionClient.ts:131
Hook forwards it:
frontend/src/hooks/useSession.ts:96
frontend/src/hooks/useSession.ts:128
Backend accepts/clamps it:
src/routes/session.ts:21, src/routes/session.ts:41, src/routes/session.ts:70
Stored on session state and passed to plan generator:
src/types.ts:113
src/engine/conversationEngine.ts:235, src/engine/conversationEngine.ts:274, src/engine/conversationEngine.ts:359
Enforced distribution in generator:
src/engine/sessionPlanGenerator.ts:121 (buildTargetCounts)
src/engine/sessionPlanGenerator.ts:150 (rebalanceTherapyBlocks)
numQuestions === 4 => 1 each type
numQuestions > 4 => picture_description strictly majority
Picture Description Unsplash query now uses topic + word
Frontend passes both target word and topic:
frontend/src/components/GameTab.tsx:141 (fetchPictureChoices(item.answer, block.topic))
frontend/src/api/sessionClient.ts:192 (includes topic query param)
Backend route receives topic:
src/routes/pictureImages.ts:10, src/routes/pictureImages.ts:16
Unsplash query built from combined concept:
src/services/pictureImageProvider.ts:194
Query variants now use combinedQuery (e.g. "cooking frying pan" and "cooking frying pan photo").
Extra fixes made while integrating

Fixed duplicate identifier compile errors in:
src/routes/session.ts
src/services/pictureImageProvider.ts
Validation run

npm run build (backend): pass
npm run build (frontend): pass
npm run test (frontend): pass (9/9 tests)
If you want, I can now commit these changes with a single message focused on the 3 features.



You are a git/code integration agent. Merge my feature branch into main safely, preserving all features added on BOTH branches since they diverged.

Inputs:
- Feature branch: questions
- Target branch: main
- Remote: origin

Requirements:
1. Fetch latest refs and work from a clean repo state.
2. Identify the merge base between questions and main.
3. Enumerate commits unique to each side since the merge base.
4. Merge questions into main (do not rebase unless required).
5. If conflicts occur, resolve them so that:
   - feature behavior from questions is preserved,
   - independent updates already on main are also preserved,
   - no regression is introduced in shared files.
6. Explicitly verify “both sides intact” by:
   - listing changed files from each side since merge base,
   - checking that each side’s key feature files/logic are present post-merge.
7. Run all available validation commands (build/test/lint/typecheck). Use project scripts if present.
8. Ensure secrets/env files remain untracked/ignored (.env, .env.*, etc.) and report if anything sensitive is tracked.
9. Provide a final report with:
   - merge base SHA,
   - commits unique to each branch pre-merge,
   - conflicts encountered and exact resolutions,
   - validation results,
   - final branch status (ahead/behind),
   - exact push command.

Constraints:
- Never force-push.
- Never delete or overwrite unrelated user changes.
- Do not modify remote history.
- If blocked, stop and explain the minimal decision needed.

Proceed now.

Show more

I’ll execute this as a full merge operation now: first I’ll clean the working tree safely, fetch latest origin, compute merge base and unique commits, then merge questions into main and run full validation.

Fetch completed. Next I’m computing the merge base and the exact commit sets unique to questions and main before merging.

I’ve captured merge-base, unique commits, and per-side changed files. I’m now switching to main, updating it to origin/main, and merging origin/questions into it.

Merge produced conflicts in frontend/src/api/sessionClient.ts, src/routes/session.ts, and src/types.ts. I’m resolving them to preserve both branches’ behavior, then I’ll run full validation.

I’m resolving src/routes/session.ts by keeping both sets of behavior: the question-count start/demo-skip flow from questions and the practice-summary/history endpoints from main.

Conflicts are resolved and staged. I’m now verifying both-side file deltas after resolution, then running backend/frontend build and tests before completing the merge commit.

Validation passed. I’m now doing the “both sides intact” verification by checking key logic from questions and key logic from main in the merged result before committing.

Merge commit is complete and working tree is clean. I’m collecting the final merge report details (commit sets, conflict resolutions, validation, secret tracking, and push command).

Merge completed successfully on main with both branches preserved.

Merge Base

dacd97d86e962643dbd35ad9731679e49ee0a256
Pre-merge Unique Commits

origin/questions since merge base:
e8edb28 Added everything
origin/main since merge base:
2789b5a Store session history
Changed Files Since Merge Base (Both Sides Intact Check Input)

origin/questions side:
frontend/src/App.tsx
frontend/src/api/sessionClient.ts
frontend/src/components/AdminVoiceSettings.tsx
frontend/src/components/Layout.tsx
frontend/src/hooks/useSession.ts
frontend/src/hooks/useTextToSpeech.ts
frontend/src/styles/theme.css
src/engine/conversationEngine.ts
src/engine/sessionPlanGenerator.ts
src/routes/pictureImages.ts
src/routes/session.ts
src/services/pictureImageProvider.ts
src/types.ts
origin/main side:
.gitignore
frontend/src/App.tsx
frontend/src/api/patientData.ts
frontend/src/api/sessionClient.ts
frontend/src/components/GameTab.tsx
frontend/src/components/SessionHistory.tsx
src/routes/session.ts
src/store/practiceSummaryStore.ts
src/types.ts
Conflicts Encountered and Exact Resolutions

frontend/src/api/sessionClient.ts
Conflict resolved by preserving both:
questions behavior: practiceQuestionCount support in startSession/startDemoSkipSession.
main behavior: error parsing helper plus practice summary/history API functions.
src/routes/session.ts
Conflict resolved by preserving both:
questions behavior: request-body parsing/clamping of practiceQuestionCount for /start and /demo-skip.
main behavior: /practice-summary and /history endpoints plus practiceSummaryStore flow.
Also retained /message and /plan behavior from both.
Merge commit created: 101293c22fde45cb813d510d7e1ca46816d9e6de.
Both Sides Intact Verification (Post-merge)

questions key logic present:
Admin question-count setting in frontend/src/components/AdminVoiceSettings.tsx
Persist + pass-through in frontend/src/App.tsx
Plan count and distribution logic in src/engine/sessionPlanGenerator.ts
Topic+word image query path in frontend/src/components/GameTab.tsx, src/routes/pictureImages.ts, src/services/pictureImageProvider.ts
main key logic present:
Practice summary store in src/store/practiceSummaryStore.ts
Summary/history routes in src/routes/session.ts
Frontend history and summary API methods in frontend/src/components/SessionHistory.tsx and frontend/src/api/sessionClient.ts
Validation Results

npm run build (repo root): passed
npm run build (frontend): passed
npm run test (frontend): passed (9/9)
Secrets / Env Tracking Check

git ls-files .env .env.*: no tracked env files.
.env patterns are ignored via .gitignore (.env.*, *.env, *.env.local).
No matching hardcoded key patterns found in tracked files from grep check.
Final Branch Status

Current branch: main
Sync vs origin/main: ahead 2, behind 0
Exact Push Command

git push origin main


Summarize what languages, frameworks, platforms, cloud services, databases, APIs, or other technologies this project uses 


I’ll quickly scan the repo manifests and key server/frontend entry files, then give you a concise stack summary by category (languages, frameworks, APIs, storage, platform).

Languages

TypeScript (frontend + backend primary)
JavaScript (legacy/aux code under src/game-logic)
CSS (custom styling)
Frontend

React 18 (react, react-dom)
Vite 5 (build/dev tooling)
Browser Web APIs (fetch, MediaRecorder audio capture flow)
Backend

Node.js + Express 4
dotenv for environment config
cors middleware
uuid for session/summary IDs
AI / External APIs / Cloud Services

Anthropic Claude via @anthropic-ai/sdk (check-in agent, plan generation, semantic answer evaluation)
ElevenLabs TTS API (/api/tts)
Google Cloud Speech-to-Text REST API (/api/stt)
Unsplash Search API (picture description images)
Wikipedia/Wikimedia image lookup fallback
Bing Image Search route still exists in code (/api/image-search) as an additional/legacy path
Data / Storage

No SQL/NoSQL database currently
In-memory session store (Map) for active sessions
File-backed JSON store for practice summaries (data/practice-session-summaries.json)
Frontend persistence via localStorage (voice settings, captions, question count)
Testing / Quality

Vitest
Testing Library (@testing-library/react, jest-dom)
TypeScript compiler checks (tsc) for type safety
Deployment Shape

Split app: React SPA frontend + Express API backend
Typical local dev: Vite frontend + Node/Express backend on separate ports with /api proxy setup


Can you add a section on missing technologies this project uses such as Google Cloud Speech-to-Text REST API and refine the whole thing based on the project? No code should be changed.

## Inspiration

Having seen firsthand the challenges of stroke recovery, a member of our team knows all too well the shortcomings of today’s computer aided speech and language therapy. Existing apps have become a cornerstone of care, giving patients the independence to practise in their own time. However, they rarely adapt to what each person actually needs to say and understand day to day. With modern AI, therapy can be tailored to the individual, with exercises that reflect their lives in content and delivery in a familiar voice.

## What it does

Speech-Therapy.ai uses agentic AI to create custom lessons for patients, and Elevenlabs' extensive library of regional accents and voice cloning capabilities to provide lessons in an accent the patient is familiar with. In turn, helping patients to recovery in their own time and to their fullest potential.

## How we built it

We built Speech-Therapy.ai as a TypeScript React web app, using Claude’s agentic capabilities to generate and adapt therapy exercises based on conversations with the user. We integrated the ElevenLabs API for high quality text-to-speech so prompts can be delivered clearly, and in a consistent and natural voice, and we used the Unsplash API to pull relevant imagery to support naming and comprehension tasks. Together, these tools let us create a personalised, multimodal therapy experience that adapts in real time.

## Challenges we ran into

Our biggest challenges throughout the project were difficulties that arose from four engineers working on a small codebase at the same time. We resolved (pardon the pun) these issues by clearly communicating what we were working on and coordinating with one another to organise branch merge order. In instances of conflicts we used coding agents to clean up the mess.

We took significant time to formulate prompts that evoked the desired behavior and tone from Claude agents.

## Accomplishments that we're proud of

We are particularly proud of including voice cloning in the project, due to time constraints this at a time seemed in doubt. More generally, we are proud of a product that has the potential to assist people in reclaiming their lives after tragedy.

## What we learned

We were all astonished by how much we achieved in the time frame. We’ll take it as a lesson in what’s possible when we stay focused and work well in a team. From talking to the very impressive builders around us at this incredible event we have learnt that by believing in an idea and persevering with it you can take something from a sketch to an MVP, to eventually a business.

## What's next for Speech-Therapy.ai

With more refinement and guardrails we will have an app ready to test with real patients. We will build relationships and in turn gain the trust of speech and language therapists in Ireland and the UK, from which they will feel confident in recommending our product. We will also contact researchers in stroke rehabilitation to run rigorous academic evaluations and build a strong evidence base. That clinical backing will give patients, clinicians, and healthcare providers the confidence to adopt our application, helping us to earn trust quickly and scale globally.

Show more

Here’s a refined version you can use, with an explicit section for the missing technologies.

Inspiration
Having seen stroke recovery up close, one of our team members experienced firsthand where current computer-aided speech and language therapy tools fall short. Existing apps are valuable for independent practice, but they often do not adapt to what each person actually needs to say and understand in daily life.
We built Speech-Therapy.ai to use modern AI for truly personalized therapy: exercises tailored to the individual’s goals, delivered in a familiar, natural voice.

What It Does
Speech-Therapy.ai runs an AI-guided check-in conversation, then generates a personalized speech and language practice session.
It supports multimodal therapy tasks (including picture-based prompts), voice interaction, and adaptive difficulty, with prompts spoken in regionally appropriate voices.

How We Built It
We built Speech-Therapy.ai as a TypeScript + React web app with a TypeScript/Express backend.

Claude powers the check-in agent, plan generation, and semantic answer evaluation.
ElevenLabs provides high-quality text-to-speech with configurable voice options.
Google Cloud Speech-to-Text REST API powers speech transcription for user responses.
Unsplash (with Wikipedia fallback) provides image options for picture-description tasks.
A therapy engine manages session flow, scoring, and feedback across question types.
Session summaries/history are stored in a lightweight local JSON store for MVP iteration.
This architecture lets us deliver a personalized, voice-first, and visually supported therapy experience.

Technologies Used (Full Stack)
Languages: TypeScript, JavaScript, CSS
Frontend: React, Vite, Vitest, Testing Library
Backend: Node.js, Express
AI/LLM: Anthropic Claude (agent/check-in, plan generation, answer evaluation)
Voice: ElevenLabs Text-to-Speech API
Speech Recognition: Google Cloud Speech-to-Text REST API
Images: Unsplash API, Wikipedia/Wikimedia fallback
Storage: In-memory session store + file-based JSON persistence for practice history
Dev/runtime tooling: dotenv, cors, uuid
Additional/Missing Technologies (Often Overlooked)
In addition to Claude, ElevenLabs, and Unsplash, the project also relies on:

Google Cloud Speech-to-Text REST API for transcribing user speech.
Wikipedia/Wikimedia image fallback when primary image retrieval is limited.
Backend semantic answer evaluation endpoint (Claude-based) to accept near-equivalent responses.
Practice session history persistence via server-side JSON storage for session analytics and review.
Challenges We Ran Into
The biggest challenge was parallel development by four engineers in a fast-moving codebase. We handled this with tighter branch discipline, clearer ownership, and explicit merge sequencing.
We also spent significant time iterating prompts to achieve the right therapeutic tone, structure, and tool-calling behavior from Claude.

Accomplishments We’re Proud Of
We’re especially proud that we integrated personalized voice delivery under tight hackathon timelines.
More broadly, we built an MVP with real potential to help stroke survivors practice communication in a way that feels personal, practical, and dignified.

What We Learned
We learned how much can be built quickly when the team is aligned and execution is disciplined.
We also learned that prompt design and integration quality matter as much as model choice when building reliable AI-assisted health tools.

What’s Next for Speech-Therapy.ai
Our next steps are to strengthen clinical guardrails, run structured pilot testing, and collaborate with SLTs and stroke rehabilitation researchers in Ireland and the UK.
The goal is to build a rigorous evidence base and a clinically trusted product that can scale responsibly to patients and providers globally.



* How does your project fit into your selected track?
Suggested length: 1-2 sentences per challenge.
Paid.ai: Agentic AI Track

Paid.ai & Track Context
Traditional SaaS pricing was built for a world of static dashboards and per-seat licences, but AI agents don't sit in seats. They work autonomously, consume resources dynamically, and deliver value that varies wildly from task to task. Having raised a $20mn seed fund led by Sequoia Capital and EQT Ventures, Paid.ai was founded to build the financial infrastructure that helps AI companies price, bill, and prove the value of their agents.

Track Goal: Build an agent that autonomously completes a meaningful task and proves its value.

How to Get Started
Read Agentic Framework Documentation: LangChain / LangGraph
Understand Agent Economics: Lightspeed on Paid
Research Multi-Agent Systems: CrewAI · AutoGen
Test Agentic Workflows with Claude: Anthropic Agent Docs
Experiment with Agentic Cost Tracking: LangSmith · Arize Phoenix
Project Directions
You can choose any domain - we just want to see an agent that works and proves its value!

Case Studies & Inspiration
Lindy: AI agents that automate entire workflows from email triage to meeting scheduling to CRM updates
Artisan: AI sales agent that handles outbound prospecting end-to-end
Helicone: Open-source LLM observability platform that tracks every request, latency, cost, and user and making agent economics visible
Sample Ideas
Invoice Agent: Receives supplier invoices, extracts data, cross-references against purchase orders, flags discrepancies — tracking time saved vs. manual processing
Talent Scout: Takes a job description, searches candidate sources, scores profiles, drafts outreach — with a clear cost-per-hire estimate
This track is currently selected for your team.

Show more

Autonomous meaningful task: Speech-Therapy.ai uses an agentic check-in flow where Claude independently gathers patient context (mood, interests, difficulty), decides when enough information is collected, and triggers generation of a personalized therapy plan with multimodal exercises. The agent then drives practice delivery (voice + image-based tasks) with minimal clinician intervention.

Proving value: We quantify value per session through measurable outputs such as completion status, question count, accuracy, and persisted session summaries/history, showing concrete therapeutic progress rather than just chat interactions. This creates a clear path to agent economics (cost per completed session and outcomes per session), which is exactly aligned with the Paid.ai track focus.




Set up Agent sandbox to continue
Set up




Default permissions

5.3-Codex
High


Work locally
