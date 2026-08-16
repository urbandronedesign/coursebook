# Voice — how these manuals are written

The reader is competent, in a hurry, and slightly stuck. Write for that person.

---

## The register

Address the reader as **vous** / *you*. Present tense. Imperative for actions. Short declarative sentences carrying one fact each.

Say the thing:

> Ne montrez pas la bouche.
> Do not show the mouth.

Not:

> It is generally advisable to consider avoiding showing the mouth where possible.

**Never**: "in today's fast-moving world", "unlock the power of", "let's dive in", "it's important to note that", exclamation marks, emoji in body text, rhetorical questions as headings, "simply" or "just" in front of something that is not simple.

**Do**: name the cost of an error in time or money. "If a clip takes 9 minutes, 20 shots at 3 takes each is 9 hours of compute" teaches more than "rendering can be slow".

## Numbers over adjectives

Every vague quality claim can be replaced by a number, a duration, or a count — and should be.

| Instead of | Write |
|---|---|
| a lot of VRAM | 24 GB |
| takes a while | 9 minutes on a 4070; 40 on a laptop |
| a few references | 3 to 5 |
| keep it short | under 60 words |
| recent hardware | NVIDIA, compute capability 7.5+, driver 570+ |

If you do not know the number, say what you do know and mark it as unverified — do not paper over it with an adjective.

## Name the trap

For each stage, the highest-value paragraph is the one naming the mistake everyone makes and what it costs. These come from watching students, and they are the part of a manual that cannot be got from documentation.

> Vous allez être tenté, au plan 14, d'ajouter `more dramatic` parce que cette image-là manque de force. **Ne le faites pas** : vous venez de créer deux films.

Predict the mistake, explain the mechanism, give the rule. In that order — a rule without a mechanism gets broken as soon as it is inconvenient.

## Say why, once

A rule with a reason survives; a rule without one is negotiated away the first time it costs five minutes. But give the reason **once**, where the rule is set, and refer back afterwards. Re-justifying in every chapter is what makes documentation unreadable.

## Prefer the honest constraint to the false promise

When a tool cannot do something, say so and give the workaround as craft rather than as a hack:

> Un personnage dont on voit la bouche parler exige une synchronisation que la génération vidéo tient mal. Si vous voulez de la parole, montrez autre chose pendant : le dos, les mains, le lieu. **C'est aussi une contrainte de cinéma classique, et souvent plus fort.**

That last clause is the difference between a limitation and a technique. Find it whenever you document a limit.

And when a later page contradicts an earlier rule — because a tool exists that changes the calculus — **go back and qualify the rule with a pointer**. Two pages disagreeing silently is worse than either being wrong.

## Verify before you write

Non-negotiable, and the fastest way to lose a reader is to get one checkable fact wrong.

- Version numbers, licence names, licence **territory restrictions**, hardware floors, context lengths: check the primary source. A repository's `LICENSE` beats a blog post.
- Every external URL: check it with an actual HTTP request before committing. All of them, every time you add a batch.
- Prefer linking the **upstream project** over a tutorial. A repository outlives a video.
- Date anything that will move: "as of August 2026".
- If you cannot verify it, write the shape of the fact and mark it unknown. An honest gap is fine; a confident invention is not.

Never state a figure you have not seen. In a technical manual, one wrong VRAM number costs the reader's trust in every other number.

## Credit the tools and their authors

By name, with a link, at the point of use and in the footer. Non-negotiable when you are shipping someone else's software to students — and it teaches the practice you want them to have.

## Bilingual writing

- Write both editions **in the same session**, chapter by chapter. Translating a finished manual produces two documents that drift.
- Translate the **meaning**, not the words. The FR and EN paragraphs may differ in length and in metaphor. They must agree on every number, identifier, section number and table row — nothing else.
- Both editions carry the same identifiers, in English. `shot_sheet.csv` is `shot_sheet.csv` in the French edition too.
- Prompts and commands stay in English in both editions — that is how the tools work — but the surrounding explanation is in the reader's language.
- Type real punctuation: `«  »` with thin spaces, `'` apostrophes, `·` separators, `—` em dashes. Do not use `&nbsp;` where a space belongs.
- The glossary is where you pay off the whole bilingual effort: it lets a French student read English documentation, which is most documentation.

## Titles and labels

- Chapter titles: a noun phrase naming the work. "Découpage & feuille de plans" — not "Chapter 9: Understanding the Breakdown Process".
- Section headings: numbered, `9.3`, so they can be cross-referenced in speech and in a crit.
- Callout labels: specific to the content, not the category. "The one-wire rule" over "WARNING".
- Captions: what to look at, not what it is.

## The preamble does four things

1. What the reader will be able to **do** at the end.
2. The three ways to read the manual (front to back / by stage / by question).
3. What they hand in.
4. The perishability policy — including *"where the manual and your installation disagree, believe the software."*

Nothing else. A preamble that explains the field is a chapter pretending to be a preamble.

## Reviewing a draft

Read every paragraph and ask:

- Is there a checkable number here that is currently an adjective?
- Would a student stuck at this stage find this by searching their own words?
- Is the criterion at the end of this exercise observable?
- Does this sentence survive the next tool release? If not, is it in the chapter where perishable things live?
- Is the trap named?
- Am I explaining something the reader can see for themselves, instead of the thing they cannot?
