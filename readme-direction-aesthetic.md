# Aesthetic: turning "where's this from?" comments into sales

**Reference startup:** [myaesthetic.shop](https://www.myaesthetic.shop/)

## Overview
Aesthetic makes a creator's content shoppable without manual work. It connects to a creator's social accounts, detects the products in each post (from the visuals, caption, audio, tags, and comments), builds a storefront automatically, and runs a sales agent that replies to product-intent comments with a link to the exact item. Creators earn a commission on every attributed sale.

Our direction keeps that core idea but rebuilds it around accuracy and trust: every product match, storefront listing, and reply the system generates is checked against real catalog data before anyone sees it.

**Target Audience:** Small and mid-size fashion, beauty, and lifestyle creators (roughly 5K to 200K followers) who get constant "where is this from?" comments but don't have time to tag products or manage affiliate links.

## 1. Solves One Problem Extremely Well
Creators lose sales every time a follower asks "where's that jacket from?" and the answer comes hours later, never comes, or links to the wrong item. Tagging every product by hand across Instagram, TikTok, and Pinterest is tedious, so most creators don't do it.

We solve one problem: **when a follower asks where something in a post is from, they get the correct product link within seconds, in the creator's voice.** The automatic storefront is a byproduct of solving that problem, not a separate feature set.

## 2. Customer Access
Our customers are two connected groups: creators (who adopt the tool) and their followers (who buy).

- As UVA students, we are surrounded by both groups. Many classmates run small fashion, thrifting, or beauty accounts, and nearly everyone we know has commented "where's this from?" on a post.
- We can reach student creators through campus fashion and style organizations, NSBE, ColorStack, and our own social networks for interviews and beta testing.
- Micro-creators are reachable directly through Instagram and TikTok DMs, and they are the segment least served by existing tools, so they are more willing to try something new.
- We can test the follower side ourselves with a small group of shoppers by measuring whether the links they receive are actually the right items.

## 3. Technical Feasibility
We are familiar with, or can quickly learn, every piece of the stack:

- **Frontend:** Next.js, React, and Tailwind CSS for the storefront and creator dashboard
- **Backend:** Python (FastAPI) for the detection and validation pipeline
- **Database:** PostgreSQL (Supabase) with pgvector for product embeddings
- **AI:** A multimodal LLM API (Claude, Gemini, or GPT) for product detection, matching, and reply drafting
- **Image similarity:** CLIP or SigLIP embeddings to compare cropped items in a post against candidate product photos
- **Product data:** Affiliate network and retailer APIs (for example ShopMy, LTK, Rakuten, or Amazon Product Advertising API) for real prices, stock, and links
- **Social integration:** Instagram Graph API and TikTok APIs for posts, comments, and replies

Our coursework in Machine Learning, Artificial Intelligence, Software Engineering, and LLMs & Software Engineering covers the embedding, model evaluation, and API design work directly. The newest pieces for us are the social platform APIs and affiliate networks, which are well documented.

## 4. Reimagining the Existing Solution
Existing tools (LTK, ShopMy, Linktree storefronts, and Aesthetic itself) still leave creators with real problems: wrong or "similar" items presented as exact matches, replies that sound like a bot, and no visibility into why a match was chosen.

Our version improves this through:

- **Similar items ranked by confidence.** Instead of sending one link and hoping it's right, the follower gets a short ranked list (up to five items) of the best matches, ordered by a confidence score with the most likely match first. Each item shows how confident the match is, so followers can see the closest option and cheaper lookalikes in one place. The score is computed by the validator from measured signals (visual similarity, attribute agreement, and brand clues), not from the model's own opinion, and an item is only called "the exact one" when its score clears a high threshold.
- **Grounded answers to follow-up questions.** Followers often ask "what size is she wearing?" or "is it true to size?" The agent only answers from facts the creator has actually given (their sizes, notes they've written), and otherwise says it will ask the creator.
- **A creator approval mode.** Low-confidence matches go to a quick approve or reject queue in the creator's dashboard instead of being posted automatically. Each decision becomes training data for better matching.
- **Live price and stock.** Links are rechecked before sending, so followers don't click through to sold-out items.
- **Built-in disclosure.** Every reply includes the required affiliate disclosure automatically, so creators stay compliant with FTC rules without thinking about it.

## 5. Essential Use of LLMs / Generative AI
The product cannot exist without a generative multimodal model. It is not a chatbot added to a storefront. The model does the core work:

- **Product detection:** The LLM looks at a post's images or video frames together with the caption, audio transcript, tags, and comments, and produces a structured list of items (category, color, material, distinguishing details, brand clues, and bounding boxes).
- **Search query generation:** For each detected item, the LLM writes targeted catalog search queries ("black linen single-breasted blazer women's relaxed fit").
- **Candidate ranking:** The LLM compares candidate products against the detected item and explains which details match and which don't.
- **Intent detection:** The LLM classifies comments to separate real buying intent ("where's the blazer from? 😍") from compliments ("clean 🔥").
- **Reply generation:** The LLM drafts each reply in the creator's voice, learned from the creator's own past captions and replies.

Rule-based systems and keyword search cannot reliably connect a blurry photo of a jacket and a comment saying "need that" to a specific product page. Generative models are what make this possible.

## 6. LLM Output Validation
No LLM-generated detection, match, storefront listing, or reply reaches a creator or follower without passing automated validation.

The validator checks:

- **Schema:** Every detection and match is valid JSON matching a strict schema (item category from a fixed list, required fields present, bounding boxes inside the image).
- **The product is real:** Every product ID and URL must exist in the retailer or affiliate API response. The model cannot invent a product or a link.
- **Visual agreement:** The cropped item from the post and each product image must meet a minimum embedding similarity score to be shown at all. Items below it are dropped from the list.
- **Confidence score and ranking:** The validator computes each item's confidence from visual similarity, attribute agreement, and brand or caption evidence. The final list must be sorted by that score, and only the top item can be described as "exact" in a reply, and only if its score clears the exact-match threshold.
- **Attribute consistency:** Category and color in the match must agree with the detection (for example, a "black blazer" cannot be matched to a brown coat).
- **Facts in replies:** Any price, brand, or size mentioned in a reply must exactly equal the catalog data or the creator's stated facts. The model can't make up a number.
- **Live availability:** The product must be in stock and the link must resolve when the reply is sent.
- **Policy and safety:** The reply includes the affiliate disclosure, stays within platform length limits, contains no prohibited content, and makes no health or body claims.

If validation fails and cannot be fixed, the output is not shown. The comment is sent to the creator's approval queue instead.

## 7. Automatic Error Correction
When an output fails validation, the specific errors are sent back to the LLM automatically, and it regenerates.

Example error messages fed back to the model:

- `"product_id 88213 not found in catalog results; choose only from the provided candidates"`
- `"reply calls item 2 'the exact one' but its confidence is 0.61 (needs 0.85); describe it as a similar option"`
- `"ranking is out of order: item 3 (confidence 0.82) is above item 1 (0.74); reorder by confidence"`
- `"reply states price $89 but catalog price is $100"`
- `"reply is missing the affiliate disclosure"`

**Example workflow:**

1. A follower comments "where's the blazer from?" on a new post.
2. The LLM classifies the comment as purchase intent and identifies the blazer in the post.
3. The LLM generates search queries and ranks catalog candidates.
4. The validator checks that each product exists, is visually similar enough, matches the attributes, and is in stock, then scores each one and checks that the list is ranked by confidence.
5. If any checks fail, the errors are returned to the LLM, which corrects its candidates, ranking, or wording.
6. The LLM drafts a reply, and the validator checks its facts, disclosure, and length.
7. Only a fully validated reply is posted. After three failed attempts, the comment goes to the creator's approval queue with the errors attached.

Every retry and failure is logged, so we can measure how often the model self-corrects and which checks fail most often.

## 8. Deployment Feasibility
We can build and deploy a working version within the semester.

Deployment infrastructure:

- **Frontend:** Vercel (storefronts at `creator.ourdomain.com`)
- **Backend:** Render or Fly.io (FastAPI service and background workers)
- **Database:** Supabase PostgreSQL with pgvector
- **Background jobs:** A simple queue (Supabase cron or Redis with RQ) for processing new posts and comments
- **AI:** Hosted multimodal LLM API and a hosted embedding model

The initial MVP will include:

1. Creator sign-up and connecting an Instagram professional account (or manually uploading posts during Meta app review).
2. Automatic product detection and matching on new posts, with similar items ranked by confidence.
3. An auto-generated storefront page per creator.
4. Comment monitoring with validated, auto-generated replies, plus the creator approval queue.
5. The full validation and self-correction loop with logging.

**Known risk:** Instagram's API requires app review before we can reply to comments for creators outside our test accounts. During review, we will run the MVP with our own test accounts and a small group of student creators added as testers. We will start with five to ten student creators and track match accuracy and click-through before expanding.

## Conclusion
Aesthetic shows that creators want their content to sell for them. Our direction focuses on doing one thing extremely well: answering "where's this from?" with the right product, quickly and honestly. The generative model does the detection, matching, and writing, and a strict validator with automatic feedback makes sure every link and reply a follower sees is real, accurate, and in stock.
