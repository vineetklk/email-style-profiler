---
name: email-style-profiler
description: >-
  Reads your last 50 sent emails, classifies them as internal (@sap.com) or external, extracts your personal writing style for each context, and generates and installs a personalised email-drafter skill that drafts ready-to-copy emails in your voice. Generic enough for any colleague to run and get their own personalised drafting skill. Trigger phrases: "create my email style skill", "analyse my email style", "build my email drafter", "generate email drafting skill", "profile my email style", "create email skill from my emails", "build personalised email skill", "make me an email drafting skill".
allowed-tools: list_emails get_email search_emails render_ui create_skill
metadata:
  author: SAP RISE Enterprise Architect
  version: 1.1.0
  tags: email productivity style communication meta-skill
---

# Email Style Profiler

Analyse the user's sent emails to extract their personal writing style — separately for internal SAP colleagues and external recipients — then generate and install a personalised `email-drafter` skill.

The goal is to produce emails that sound unmistakably human and personal — not AI-generated. The deepest value is in capturing micro-writing patterns: how the person punctuates, whether they ever use bullets, whether they use dashes or colons, sentence rhythm, and vocabulary habits.

---

## Step 1 — Fetch Sent Emails

Call `list_emails` with:
- `folder: "sentitems"`
- `top: 50`

This returns the 50 most recent sent items. Do NOT read full bodies yet.

---

## Step 2 — Classify Emails

For each email, inspect the recipient address(es):
- **Internal**: recipient ends with `@sap.com`
- **External**: any other domain

Split into two groups: `internal_emails` and `external_emails`.

**Balance check**: If either group has fewer than 10 emails, tell the user:
> "I only found [N] [internal/external] sent emails in your last 50. The style profile for that group may be thinner. Shall I proceed anyway, or would you like me to extend the sample?"

Wait for the user's answer if the check fails. Otherwise proceed automatically.

---

## Step 3 — Read Full Email Content

Select up to **20 emails from each group** (40 total), prioritising the most recent.

Read them in **parallel batches of 10**: emit 10 `get_email` calls in a single message, wait for all results, then emit the next 10.

---

## Step 4 — Extract Style Signals

For each group (Internal / External), extract TWO layers of signals:

### Layer A — Surface structure

**Greeting style**
Dominant opening pattern. Examples: "Hi [Name],", "Dear [Name],", "Hello [Name],", "[Name]," or no greeting.

**Sign-off style**
Dominant closing. Examples: "Best regards", "Thanks", "Cheers", "BR". Note whether they include name/title.

**Email length**
Short (1–3 sentences), Medium (1–2 paragraphs), Long (3+ paragraphs). Note the typical range.

---

### Layer B — Micro-writing patterns (CRITICAL for human-sounding output)

This layer is what makes the difference between AI-generated and genuinely personal emails. Extract each signal carefully:

**Bullet points**
Does this person use bullet points at ALL? If yes — how often, and in what situations? If they never use them, note: NEVER USE BULLETS.

**Em-dashes and en-dashes**
Do they use em-dashes (—) or en-dashes (–)? These are strong AI-signature patterns. If the person never uses them, note: NEVER USE DASHES.

**Colons for lists or emphasis**
Do they use colons (e.g. "Three things: …", "Key point: …")? If not, note: AVOID COLONS.

**Semicolons**
Do they use semicolons to join clauses? Most human writers rarely do. Note presence or absence.

**Sentence length and rhythm**
Are sentences short and punchy? Long and flowing? Mixed? Note the dominant pattern.
Example: "Short and direct — typically 8–15 words per sentence, rarely more."

**Paragraph structure**
Do they write in full paragraphs, or single-line breaks between thoughts? Note the pattern.

**Capitalisation habits**
Do they follow strict title/sentence case, or are there casual deviations (e.g. all lowercase in quick internal notes)?

**Number formatting**
Do they write "3 items" or "three items"? Consistent pattern?

**Abbreviations and shorthand**
Do they use FYI, ASAP, BTW, pls, thx, or similar? Note which ones and in which context.

**Filler or connector words they use**
Look for recurring transition phrases — "Just wanted to", "Wanted to follow up", "Let me know", "Happy to jump on a call", "Please find below", "As discussed". Note the actual phrases found.

**Phrases they NEVER use**
AI-signature phrases to check for and confirm absence: "I hope this email finds you well", "Please do not hesitate", "As per my previous email", "Going forward", "Touch base", "Circle back", "Deep dive", "Synergies". If absent from the sample, these go on the NEVER USE list.

**Tone register**
Formal / Semi-formal / Informal — with a one-sentence description of how this shows up. Example: "Semi-formal: professional vocabulary but uses first names and contractions freely."

---

## Step 5 — Present the Style Profile

Show the user a two-part summary:

**Part 1 — Surface structure** using `render_ui` with `hint: "table"`:
Rows: Internal, External
Columns: Context | Greeting | Sign-off | Length | Tone

**Part 2 — Micro-pattern summary** as a short Markdown block per context:
List the key micro-patterns found, including explicit NEVER USE items. For example:

```
**Internal micro-patterns:**
- No bullet points — writes in flowing sentences
- No em-dashes — never used in sample
- Short sentences (8–12 words typical)
- Uses "Let me know" and "Thanks" frequently
- NEVER USE: dashes, bullets, colons as list headers, "I hope this email finds you well"

**External micro-patterns:**
- Occasional short bullet lists (3 items max, only for deliverables)
- No em-dashes
- Medium sentences, one or two paragraphs
- Uses "Please find" and "Kind regards" consistently
- NEVER USE: em-dashes, "Please do not hesitate", "Going forward", "Touch base"
```

Then say:
> "This is your writing profile. The micro-patterns are what will make your emails sound like you rather than AI. Does this look right? Tell me anything that feels off before I generate your skill."

Wait for the user to confirm or correct. Apply any corrections before proceeding.

---

## Step 6 — Generate the Personalised Email Drafter Skill

Once the user confirms, call `create_skill`.

**Replace every placeholder in square brackets with actual extracted values. Do NOT leave any `[...]` text in the generated skill body — the skill must contain real patterns, not instructions to fill in later.**

### skill_name
`email-drafter`

### description
Write a description that includes trigger phrases and mentions the user's name if known:
```
Drafts emails in [USER NAME IF KNOWN, else "your"] personal writing style — calibrated for internal SAP colleagues or external recipients (customers, partners). Produces a clean, copy-ready email draft that sounds human, not AI-generated.

Use this skill when the user says: "draft an email", "write an email to", "compose a message to", "help me write an email", "draft a reply to", "write a quick note to my team", or any request to write or compose an email — internal or external.
```

### body
Write the full body using the template below. Every bracketed section must be replaced with real, specific values from Step 4. The NEVER USE section is critical — it is what prevents AI-sounding output.

---
```
# Email Drafter

Draft emails in the user's personal writing style — adapted for internal (SAP colleagues) or external (customers, partners) communication. The output must sound like the person, not like AI. Follow the style profiles and NEVER USE rules below exactly.

---

## Activation

Trigger on any request to draft, write, compose, or reply to an email:
- "Draft an email to my customer about the renewal delay"
- "Write a quick note to my team about the project status"
- "Help me reply to this message"
- "Compose an email to [name] about [topic]"

---

## Step 1 — Gather Context

If the user's message already contains a recipient and a topic, proceed directly to Step 2.

If either is missing, ask in a single message:
- "Who is this email to? (name or role)"
- "What is the main message or ask?"

Wait for the answer before drafting.

---

## Step 2 — Determine Communication Context

Classify the recipient:
- **Internal**: SAP colleague, team member, or internal stakeholder
- **External**: customer, partner, prospect, or anyone outside SAP

If unclear, ask: "Is this to someone inside SAP or an external contact?"

---

## Step 3 — Draft Using the Style Profile

### Internal Style Profile
- **Greeting**: [ACTUAL INTERNAL GREETING]
- **Sign-off**: [ACTUAL INTERNAL SIGN-OFF]
- **Length**: [ACTUAL INTERNAL LENGTH]
- **Tone**: [ACTUAL INTERNAL TONE]
- **Structure**: [ACTUAL INTERNAL STRUCTURE]
- **Sentence rhythm**: [ACTUAL INTERNAL SENTENCE RHYTHM]
- **Paragraph style**: [ACTUAL INTERNAL PARAGRAPH STYLE]
- **Abbreviations / shorthand used**: [LIST ANY USED, or "None"]
- **Connector phrases to use**: [ACTUAL RECURRING PHRASES — e.g. "Let me know", "As discussed"]

### External Style Profile
- **Greeting**: [ACTUAL EXTERNAL GREETING]
- **Sign-off**: [ACTUAL EXTERNAL SIGN-OFF]
- **Length**: [ACTUAL EXTERNAL LENGTH]
- **Tone**: [ACTUAL EXTERNAL TONE]
- **Structure**: [ACTUAL EXTERNAL STRUCTURE]
- **Sentence rhythm**: [ACTUAL EXTERNAL SENTENCE RHYTHM]
- **Paragraph style**: [ACTUAL EXTERNAL PARAGRAPH STYLE]
- **Connector phrases to use**: [ACTUAL RECURRING PHRASES]

---

## NEVER USE — Internal

These patterns were absent from the user's actual emails. Using them would make the email sound AI-generated. Never include them:

[LIST EACH FORBIDDEN PATTERN ON ITS OWN LINE — e.g.]
- Em-dashes (—) or en-dashes (–)
- Bullet points
- Colons used as list headers ("Key points:")
- "I hope this email finds you well"
- "Please do not hesitate to reach out"
- "Going forward"
- "Touch base" / "Circle back"
- [ANY OTHER PATTERNS NOT FOUND IN THE SAMPLE]

## NEVER USE — External

[SAME STRUCTURE — list what was absent from external emails]
- [PATTERN]
- [PATTERN]
- [PATTERN]

---

## Step 4 — Present the Draft

Show the email in a clean block:

Subject: [Subject line]

[Greeting]

[Body — written following the style profile and NEVER USE rules above]

[Sign-off]

Then say:
> "Here is your draft. Let me know if you would like to adjust the tone, length, or any specific wording."

---

## Step 5 — Refine if Requested

If the user asks for changes, revise immediately and show the updated draft.

Refinement guide:
- "Shorter" → Cut to the essential ask; remove context if implicit
- "More formal" → Remove contractions, use full salutation and sign-off
- "More casual" → Add contractions, shorten sentences, first-name sign-off
- "Add urgency" → Include a clear deadline or consequence
- "Less AI" → Check the NEVER USE list and remove any violations; simplify sentence structure
- "Different subject" → Re-draft with a new subject line only
```
---

### allowed_tools for the generated skill
`list_emails search_emails get_email`

---

## Step 7 — Confirm Installation

After `create_skill` completes, tell the user:

> "Your personalised email drafting skill is installed. Find it in the Extensions panel under Skills.
>
> To use it, start a new conversation and say something like:
> - *'Draft an email to my customer about the project timeline'*
> - *'Write a quick note to my team about next week's standup'*
>
> It will match your style — internal or external — and avoid any AI-signature patterns. After testing, just tell me what to tweak."