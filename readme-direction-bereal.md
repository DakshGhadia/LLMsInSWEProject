# BeReal Recap: keeping real friend groups posting

**Reference startup:** [bereal.com](https://bereal.com/)

## Overview
BeReal sends everyone a notification at a random time each day, giving them two minutes to take a photo with the front and back cameras at the same time. The photos are unfiltered and unstaged, and you only see friends' posts after you post your own. The app is built on authenticity, and BeReal openly says "no AIs allowed" in its photos.

Our direction keeps that rule completely: **the AI never creates, edits, or alters a photo.** Instead, a generative model works around the real photos to fix BeReal's biggest weakness. Friend groups stop posting because a random daily photo, seen once and then forgotten, gives people little reason to keep coming back.

BeReal Recap turns a friend group's real daily photos into two things: a **daily prompt written for that specific group**, and a **weekly Recap** that tells the story of the group's week using only the photos they actually took.

**Target Audience:** College friend groups (roommates, teams, clubs, and friends from home who are now at different schools).

## 1. Solves One Problem Extremely Well
BeReal's posts are mostly "my desk again" or "my ceiling again," and after a few weeks many friend groups stop posting. Once a few people stop, the rest have no reason to post either, and the group dies out.

We solve one problem: **keeping a small friend group posting every day.** Every feature exists to make today's post more interesting to take and to give the group a reason to look back at it.

## 2. Customer Access
College students are BeReal's core audience, and we are college students.

- We, our roommates, and our friends have used BeReal, and many of us have watched our own friend groups stop posting. We can interview them directly about why.
- We can recruit test friend groups through UVA dorms, clubs, intramural teams, NSBE, and ColorStack.
- Our success measure is easy to observe in a small beta: does a group of 5 to 10 friends keep posting for four weeks with our version compared to without it?
- Friends who have graduated or transferred give us access to "long-distance" friend groups, which are the groups that need this most.

## 3. Technical Feasibility
We are familiar with, or can quickly learn, the required infrastructure:

- **Frontend:** React (as a Progressive Web App) or React Native with Expo, using the device's front and back cameras
- **Backend:** Python (FastAPI) or Node.js
- **Database and storage:** Supabase (PostgreSQL, Auth, and photo storage)
- **Notifications:** Web Push or Expo push notifications for the daily random-time alert
- **AI:** A multimodal LLM API (Claude, Gemini, or GPT) for prompt writing and Recap generation
- **Validation support:** A vision model or object detector to independently check what is in each photo, and a content moderation API

The camera capture, timed notifications, and photo feed are standard mobile web features. The LLM pipeline and validator build directly on our LLMs & Software Engineering, Artificial Intelligence, and Machine Learning coursework.

## 4. Reimagining the Existing Solution
BeReal's format is great for authenticity, but every day looks the same and nothing lasts. Our version keeps the random-time, front-and-back, unfiltered format and adds:

- **Group-specific daily prompts.** Instead of only "Time to BeReal," each group gets a short prompt based on what is actually happening in their lives that week, such as "Show us your study spot the night before exams" during finals or "What's for dinner?" when the group has been posting meals. Prompts are always optional, and posting a normal BeReal still counts.
- **The weekly Recap.** Every Sunday, the group gets a short, scrollable story of their week built from their real photos: who was where, small running jokes, and moments that happened at the same time ("three of you were in the library at 11pm on Tuesday").
- **Memories that mean something.** Recaps are saved, so a group can scroll back through their semester in a way that a list of single photos never allowed.
- **Group-level streaks.** The group's streak depends on everyone, which gives friends a reason to nudge each other.
- **Authenticity stays intact.** Photos are never filtered, generated, or edited. The AI only writes text around photos that people actually took.

## 5. Essential Use of LLMs / Generative AI
The generative model is essential to the product. Without it, there are no personalized prompts and no Recaps, which are the two features that solve the problem.

- **Understanding photos:** A multimodal LLM describes what is in each photo (setting, activity, objects) as structured data, using only the image and the caption the user wrote.
- **Writing daily prompts:** The LLM uses the group's recent photo descriptions, the time of day, and the calendar (for example, exams, weekends, or holidays) to write a short prompt that fits that particular group.
- **Generating Recaps:** The LLM chooses which real photos tell the week's story, puts them in order, and writes short captions and narration that link them together, including noticing patterns across friends.

No template or fixed rule system could look at a week of random photos from eight people and write a story about it. That requires a generative model. It is not a chatbot in a sidebar. Users never chat with it. It powers the main experience.

## 6. LLM Output Validation
No LLM-generated prompt, photo description, or Recap reaches users without passing automated validation.

The validator checks:

- **Schema:** Prompts and Recaps follow a strict JSON schema (prompt length limit, Recap made of a list of photo IDs, captions, and order).
- **Only real photos:** Every photo ID in a Recap must exist, belong to that group, and come from that week. Nothing generated or from outside the group can appear.
- **Captions match the photos:** Any object, place, or activity mentioned in a caption must also be found by an independent vision check on that photo, or come from the user's own caption. This catches made-up details.
- **Privacy rules:** Captions can only name people who are in the group. Captions do not guess locations unless the user shared them, and they never guess sensitive traits (health, religion, relationships, appearance, or body comments).
- **No unkind content:** A moderation check blocks teasing, insults, or anything that singles someone out negatively, including someone who didn't post.
- **Safe and doable prompts:** A prompt must be possible within two minutes, must not require buying anything or going anywhere, and must not encourage anything dangerous. It also must not repeat a recent prompt.
- **Time and context:** Prompts have to fit the user's local time (for example, no "show your breakfast" at 10pm).

If an output still fails after the retry limit, the group gets the standard "Time to BeReal" alert, or the Recap is built with simple date captions instead of generated text.

## 7. Automatic Error Correction
When an output fails validation, the validator's specific errors are sent back to the LLM automatically, and it regenerates.

Example error messages fed back to the model:

- `"caption for photo p_412 mentions 'a dog', but no dog was detected and the user's caption does not mention one"`
- `"photo p_377 is from a different group; use only the photo IDs provided"`
- `"caption names 'Jordan', who is not a group member"`
- `"prompt 'Go get boba with a friend' requires leaving and purchasing; prompts must be doable in 2 minutes where you are"`
- `"caption comments on a person's appearance; remove it"`

**Example workflow (weekly Recap):**

1. On Sunday, the system collects the group's photos and descriptions from the week.
2. The LLM selects photos, orders them, and writes captions.
3. The validator checks the schema, that photos are real, that captions are supported by the photos, privacy, and tone.
4. If errors are found, they are returned to the LLM along with the original data.
5. The LLM revises the Recap.
6. The validator checks the revised Recap again.
7. Only a fully validated Recap is sent to the group. After three failed attempts, the group gets a plain photo Recap with dates instead of generated captions.

The same loop runs for every daily prompt. All failures and retries are logged so we can measure which checks fail most and improve our prompts.

## 8. Deployment Feasibility
We can build and deploy a working version within the semester.

Deployment infrastructure:

- **Frontend:** Vercel (Progressive Web App that can be added to the home screen, with camera access)
- **Backend:** Render or Fly.io
- **Database, auth, and photo storage:** Supabase
- **Scheduled jobs:** Supabase cron or a simple worker for the random daily alert and Sunday Recaps
- **Notifications:** Web Push (or Expo push if we go native)
- **AI:** Hosted multimodal LLM API and a moderation API

The initial MVP will include:

1. Sign-up, creating a friend group, and inviting friends with a link.
2. The daily random-time alert with front and back camera capture and a two-minute window.
3. A group feed that unlocks after you post.
4. Validated, group-specific daily prompts.
5. Validated weekly Recaps built only from real photos.
6. The full validation and self-correction loop with logging.

We will start with three to five real UVA friend groups and compare posting rates over four weeks against how those groups used BeReal before.

## Conclusion
BeReal proved that people want an authentic, low-pressure way to share their day with close friends. The problem is that groups stop posting when the photos feel repetitive and disappear. BeReal Recap uses a generative model to make each day's post worth taking and each week worth looking back on, without ever touching the photos themselves. A strict validator with automatic feedback makes sure everything the AI writes is true to the photos, respects privacy, and stays kind.
