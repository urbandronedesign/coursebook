# Content model — how a course manual is organised

Format decisions are reversible. These are not: change the spine once chapters exist and every cross-reference, every deliverable and every band tag moves with it.

---

## One artefact, carried through

The strongest organising principle available: **the whole manual produces one thing**, and every chapter is a stage of producing it.

- AI_GEN_NARRATIVES → one 60-second film, 20 shots × 3 s, in five blocks.
- 3DVIZ → one interior, taken to three quality levels.

Name the artefact and its **exact scale** in the preamble. Scale is not a detail — "20 shots of 3 seconds" makes every downstream chapter concrete: how many prompts, how long the render queue, what fits in a term. "Make a short film" produces a manual full of hedges.

The alternative — organising by topic — produces a reference, not a course. References are useful; they are not what students need first.

## Chapter order = the real order of production

Order chapters the way the work actually happens, including the unglamorous parts:

```
understand → gather material → write → lock decisions →
produce images → produce motion → sound → assemble → deliver → law & credits
```

Two consequences worth stating in the manual:

**The return path.** A fault found downstream was made upstream. Bad motion at stage 6 is usually a bad breakdown at stage 3. Put this in the pipeline diagram and refer to it whenever a chapter's failure mode is really an earlier chapter's.

**The highest-return check.** In most pipelines there is one cheap step that catches the most expensive class of error — an animatic, a grey-box render, a dry run. Identify it and make it a gate, not a suggestion.

## What they hand in

Deliverables drive the chapter order, so decide them at the same time.

| | Deliverable | Why it exists |
|---|---|---|
| **A** | The production folder — references, breakdown, bibles, log | Makes the method chapters **directly assessed** instead of preparation for something else |
| **B** | The intermediate artefact — storyboard, approved frames, each with its exact parameters | Catches drift before the expensive stage |
| **C** | The finished thing | The obvious one |
| **D** | *(top levels only)* A reusable asset left to the studio — a documented workflow, a trained model with its dataset, a technique sheet | Turns a student into a contributor, and gives the band system something real to require |

If a chapter's work appears in no deliverable, students will skip it and they are not wrong to.

## Bands: one manual, several years

See `architecture.md` §3 for the mechanism. The content decisions:

- Tag with the **lowest** band that needs it, so bands are cumulative and nothing is unlearned.
- **Two bands maximum.**
- What changes between bands is **format, technical depth, and how much argument is required** — not the pipeline. A first-year does the whole course, smaller.
- Put a table in the preamble showing what each year does: format, scale, which sections apply, what is required. Students and colleagues both read it first.

Example of the shape:

| Year | Format | Required depth |
|---|---|---|
| A1·A2 | 30 s · 10 shots | core spine only |
| A3 | 60 s · 20 shots | + practitioner sections |
| A4·A5 | 60 s + reusable asset | + research sections, transmission |

## Exercises

One per stage, numbered continuously across the manual. Two headings, always the same:

**What you do** — the action, imperative, specific.
**Ready to move on when** — an *observable* criterion.

The second is where courses usually fail. "When the composition works" cannot be checked by the student, so it does not gate anything. "When a 3 s clip renders in under 10 minutes on the machine you will actually use" can, and it prevents a whole class of disaster.

Good gates measure: a count, a duration, a file existing, a stranger succeeding at something.

## Reference sections

Three, at the back, and they carry more traffic than most chapters.

**Symptom → Cause → Fix → Chapter.** Organised by what the reader *sees*, in their words. This is the page students actually use, because a manual is consulted in failure. Aim for 30+ rows; each is separately searchable.

**FAQ**, phrased as students ask, mistakes included. Group by theme. Answers short, pointing at chapters.

**Bilingual glossary.** Terms as *this course* uses them. The FR/EN pairing is what lets a French student read English documentation.

## Handling what will be false next year

Every technical course has facts with a six-month half-life: versions, model names, prices, VRAM figures, menu paths.

**Concentrate them.** One chapter, clearly dated, plus a companion page that is expected to change. Everywhere else, write about roles and methods that outlive names: "a text model", "the segmentation step", "the upscaling pass".

State the policy in the preamble, in the manual's own voice:

> This field turns over every couple of months. The model names quoted here will be stale before you graduate; the method will not. Chapter 3 and the companion page deliberately concentrate everything perishable, so the rest of the manual stays true. **Where the manual and your installation disagree, believe the software.**

That last sentence saves more support time than any other line in the document.

Then add the durable skill that replaces the perishable facts: **how to judge a new tool.** A protocol — one fixed test case, one fixed seed, judge on what costs you dearly, time it — outlives every name in the manual and is the most transferable thing in it.

## Front and back: what students may see

If teachers run infrastructure students never touch — a server, a render farm, an admin panel — the manual documents **only what students can reach**. Everything else goes in a maintainer guide that is not linked from the manual.

This is not secrecy; it is that a student who reads about the server tries to install the server. Keep two documents: the manual, and `CAPTURE-GUIDE.md` / `MAINTENANCE.md` with the front/back boundary written out as a table.

## Legal and ethical chapter

Any course producing media with generative tools needs one, and it belongs at the end where it will be read as part of finishing rather than as a preamble to be skipped.

Cover: licences of the models and assets actually used (with a folder to store the licence texts, because pages change); **people** — consent for likeness, voice, and for training on either, which is a different act from being photographed; attribution and credits; the transparency obligations in your jurisdiction, dated; territory restrictions in model licences, which are easy to miss and can invalidate a whole project.

Ship the consent form as a template file rather than describing it. A form that exists gets used.

## Length

For reference, AI_GEN_NARRATIVES: 18 chapters, ~68,000 words across both languages, 595 KB, 24 sections, 93 tables, 8 diagrams, 34 exercises, 64 FAQ entries.

That is a full-year course for five cohorts. A one-week workshop is 4–6 chapters and perhaps 12,000 words — the same skeleton, the same audit, no bands, one companion page. Do not pad a workshop to look like a course; a short manual that is entirely used beats a long one that is skimmed.
