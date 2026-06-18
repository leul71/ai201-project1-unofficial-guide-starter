# Project 1 Planning: The Unofficial Guide

> Write this document before you write any pipeline code.
> Your spec and architecture diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Update the Retrieval Approach and Chunking Strategy sections if you change your approach during implementation.
> Update this file before starting any stretch features.

---

## Domain

This guide covers student life at the University of Washington Seattle specifically 
the informal, experience-based knowledge that students share with each other but that 
never appears in official university materials. Topics include professor and course 
quality, dining hall opinions, housing experiences, registration strategies, and 
campus survival tips. This knowledge is valuable because it reflects what students 
actually encounter day to day, but it's hard to find because it's scattered across 
Reddit threads, review sites, and word of mouth rather than collected in any one place.

---

## Documents

<!-- List your specific sources: URLs, subreddit names, forum threads, or file descriptions.
     Aim for at least 10 sources that together cover different subtopics or perspectives within your domain. -->

| # | Source | Description | URL or location |
|---|--------|-------------|-----------------|
| 1 | r/udub Reddit | Student tips, freshman advice, and "things I wish I knew" threads | https://www.reddit.com/r/udub/ |
| 2 | r/udub — Registration Tips | Crowdsourced advice on MyPlan, time scheduling, and avoiding bad quarters | https://www.reddit.com/r/udub/search/?q=registration+tips |
| 3 | Rate My Professors — UW Seattle | Student reviews of UW professors including difficulty, grade, and comments | https://www.ratemyprofessors.com/school/1530 |
| 4 | r/udub — "Things I wish I knew" | Specific advice threads from students looking back on their first year | https://www.reddit.com/r/udub/search/?q=things+i+wish+i+knew |
| 5 | r/udub — Housing & Dorms | Student opinions on dorm quality, roommate experiences, and which dorms to avoid | https://www.reddit.com/r/udub/search/?q=dorms+housing |
| 6 | r/udub — Dining | Student takes on which dining halls are worth it, meal plan value, and hidden gems | https://www.reddit.com/r/udub/search/?q=dining+food |
| 7 | r/udub — Off-Campus Housing | Threads about neighborhoods, landlords, and apartment hunting near UW | https://www.reddit.com/r/udub/search/?q=off+campus+housing+apartments |
| 8 | r/udub — CS/Engineering Advice | Major-specific tips on courses, professors, and internship recruiting | https://www.reddit.com/r/udub/search/?q=CSE+computer+science |
| 9 | r/udub — Quarterly Registration | Threads about getting into impacted courses, waitlists, and section strategies | https://www.reddit.com/r/udub/search/?q=registration+waitlist+quarters |
| 10 | Yelp — UW Campus Area Restaurants | Student reviews of food spots near campus, useful for "where should I eat" queries | https://www.yelp.com/search?find_desc=restaurants&find_loc=University+of+Washington%2C+Seattle |

---

## Chunking Strategy

<!-- How will you split documents into chunks?
     State your chunk size (in tokens or characters), overlap size, and explain why those
     numbers fit the structure of your documents.
     A review-heavy corpus warrants different chunking than a long FAQ. -->

**Chunk size: My documents are primarily short-form student reviews and Reddit comments, about 1 to 5 sentences long. Because the useful information in each review is concentrated in 
a single thought rather than spread across paragraphs, I will use a chunk size of 
300–400 characters with an overlap of 50–75 characters.**

**Overlap: Overlap is important here because some Reddit posts contain a key fact at the end of one sentence and supporting detail at the start of the next without overlap, that 
context could be split across two non retrievable chunks.**

**Reasoning: Chunks smaller than this risk losing enough context to be meaningless on their own. Chunks larger than this 
risk merging two unrelated reviews into one embedding, diluting the semantic signal.**

---

## Retrieval Approach

<!-- Which embedding model are you using (e.g., all-MiniLM-L6-v2 via sentence-transformers)?
     How many chunks will you retrieve per query (top-k)?
     If you were deploying this for real users and cost wasn't a constraint, what tradeoffs
     would you weigh in choosing a different embedding model — context length, multilingual
     support, accuracy on domain-specific text, latency? -->

**Embedding model: all MiniLM L6 v2 via sentence transformers. It runs locally 
with no API key or rate limits and performs well on short, opinion-based text like 
Reddit comments and professor reviews.**

**Top-k: This gives the LLM enough context to synthesize an answer across 
multiple reviews without flooding it with loosely related results. If retrieval 
feels too noisy in testing I will drop to k=3; if answers feel incomplete I will 
raise to k=7.**

**Production tradeoff reflection: all MiniLM L6 v2 caps at 256 tokens, which fits 
short reviews well but would truncate longer documents all-mpnet-base-v2 at 512 
tokens would be a better fit for longer-form content. General-purpose models may also 
underperform on slang-heavy student writing, so a model fine-tuned on review data 
would likely improve retrieval quality for this domain. On the latency side, larger 
models produce better embeddings but run slower locally; for a high-traffic app, a 
hosted API like OpenAI's text-embedding-3-small would offer better scalability. 
Multilingual support is not a concern here, but would matter if the guide were 
serving international students writing in their native language.**

---

## Evaluation Plan

<!-- List your 5 test questions with their expected correct answers.
     Questions should be specific enough that you can judge whether the system's response
     is right or wrong. "What are good dining halls?" is too vague.
     "What do students say about wait times at [dining hall name] during lunch?" is testable. -->

| # | Question | Expected answer |
|---|----------|-----------------|
1 | What do students say about getting off the CS major waitlist at UW? | Students should mention that getting off the waitlist is difficult, that direct admission is competitive, and may reference strategies like taking prereqs early or applying to related majors first. |
| 2 | Which dining halls do UW students recommend and which do they avoid? | Students should name specific dining halls (e.g. Cedars, Eight, District Market) with reasons — food quality, wait times, meal plan value, or location. |
| 3 | What tips do UW students give for surviving registration on MyPlan? | Response should include specific advice like adding courses to a plan early, knowing your registration time, using the waitlist strategically, and checking for open sections the first week of the quarter. |
| 4 | What do students say about living in the dorms vs. off-campus housing at UW? | Response should cover tradeoffs like dorm convenience vs. off-campus cost and independence, and may mention specific dorms or neighborhoods like the U-District. |
| 5 | What do UW students wish they had known before their first quarter? | Response should include specific actionable tips — not generic advice — such as which buildings are far apart, how to use office hours, or which resources students underutilize. |

---

## Anticipated Challenges

<!-- What could go wrong? Name at least two specific risks with reasoning.
     Consider: noisy or inconsistent documents, missing source attribution, off-topic
     retrieval, chunks that split key information across boundaries. -->

1.

2.

---

## Architecture

<!-- Draw a diagram of your pipeline showing the five stages:
     Document Ingestion → Chunking → Embedding + Vector Store → Retrieval → Generation
     Label each stage with the tool or library you're using.
     You can use ASCII art, a Mermaid diagram, or embed a sketch as an image.
     You'll use this diagram as context when prompting AI tools to implement each stage. -->

---

## AI Tool Plan

<!-- For each part of the pipeline below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, which requirements)
     - What you expect it to produce
     - How you'll verify the output matches your spec

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Chunking Strategy section and ask it to implement chunk_text()
     with my specified chunk size and overlap" is a plan. -->

**Milestone 3 — Ingestion and chunking:**

**Milestone 4 — Embedding and retrieval:**

**Milestone 5 — Generation and interface:**
