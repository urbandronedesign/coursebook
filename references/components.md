# Components — the copy-paste catalogue

Every block the format supports, what it is for, and when it is the wrong choice. Keep the set small: a manual with nine kinds of box teaches the reader to skip boxes.

---

## Chapter

```html
<section class="chap" id="ch7">
  <div class="fr-only">
    <span class="chapter-eyebrow">Chapitre 7</span>
    <h2 class="chapter">Titre</h2>
    <p>Pourquoi ce chapitre existe.</p>
    <h3>7.1 Section</h3>
    …
  </div>
  <div class="en-only">
    <span class="chapter-eyebrow">Chapter 7</span>
    <h2 class="chapter">Title</h2>
    <p>Why this chapter exists.</p>
    <h3>7.1 Section</h3>
    …
  </div>
</section>
```

`section.chap` is what the search index and the scroll spy walk — content outside one is invisible to both. `h2.chapter` is the indexed title. Section numbers must be identical across editions; the audit compares them.

The **first paragraph says why the chapter exists**, not what it contains. "This chapter covers X, Y and Z" is a table of contents the reader already has.

## Part divider

```html
<div class="part-divider">
  <div class="fr-only"><h2>Partie III · L'écriture</h2><p>Ce que la partie établit.</p></div>
  <div class="en-only"><h2>Part III · Writing</h2><p>What this part establishes.</p></div>
</div>
```

Sits between sections, not inside one. Parts are for orientation; they carry no content.

---

## Callouts

One border colour, one uppercase label, no icons, no background fill.

```html
<div class="callout c-warn">
  <span class="lbl">Le piège</span>
  <p>…</p>
</div>
```

| Class | For | Label reads like |
|---|---|---|
| `c-warn` | the mistake everyone makes, and its cost | "The trap", "Avoid this" |
| `c-fix` | how to recover once it has gone wrong | "If it already happened" |
| `c-doc` | a verified external fact, with its date or source | "Documented", "As of March 2026" |
| `c-teach` | studio practice, a convention, a house rule | "Studio practice", "House rule" |
| `c-tip` | a genuine aside — use sparingly | *(often no label)* |

Write a **specific label**, not the category name. "The trap" is fine; "The one-wire rule" is better. A page of boxes all labelled WARNING is a page the reader skims.

Rules: at most two callouts per section; never open a section with one; never nest them; never put the load-bearing sentence of a chapter inside one — callouts are read last.

---

## Tables

```html
<div class="tbl-wrap">
  <table>
    <thead><tr>
      <th><span class="fr-only">Colonne</span><span class="en-only">Column</span></th>
      <th>…</th>
    </tr></thead>
    <tbody>
      <tr><td>…</td><td>…</td></tr>
    </tbody>
  </table>
</div>
```

`.tbl-wrap` is required — it is what lets a wide table scroll instead of breaking the page on a phone.

Tables are the workhorse of this format: decision tables, comparisons, symptom indexes, parameter references. Every `tbody tr` is indexed individually, so a symptom index is searchable row by row for free.

Header cells are the one place bilingual `<span>` pairs beat duplicating the table. Row counts must match across editions — pass 4 catches a dropped row.

**Not for**: sequences (use `<ol>`), or anything with one column.

---

## Copyable block

For anything the reader lifts verbatim: a prompt, a command, a snippet, a template.

```html
<div class="pr">
  <div class="pr-hd">
    <span class="who">Étape 4</span><span>Ce que ce bloc produit</span>
    <span class="sp"></span>
    <button class="pr-copy" onclick="prCopy(this)">Copier</button>
  </div>
<pre>Le contenu, tel quel.
Les <span class="var">CHAMPS</span> en accent.</pre>
</div>
```

The `<pre>` starts at column 0 — indentation inside `<pre>` is content and lands in the clipboard.

`prCopy()` copies the `<pre>` **within the same `.pr`**, so the FR block copies French and the EN block copies English. If your prompt content is identical in both languages, still keep one block per edition rather than one shared block: the header text differs, and a shared block breaks the structural parity count.

`.var` marks the fields the reader must replace. Use it consistently: readers learn the colour means "your text goes here".

---

## Exercise / stage

The unit that makes a manual a course rather than a reference.

```html
<div class="exercise">
  <div class="hd">Étape 4 · Le geste à faire</div>
  <h4>Ce que vous faites</h4>
  <p>L'action, à l'impératif.</p>
  <h4>Vous pouvez passer à la suite quand</h4>
  <ul><li>Un critère observable.</li></ul>
</div>
```

Two headings, always the same two, always in this order. The second is the one that matters: **an observable criterion, not a quality judgement.**

- good — "a 3 s clip renders in under 10 minutes"; "twenty rows exist and the durations sum to 60 s"; "a stranger can find any reference in ten seconds"
- bad — "when the composition is good"; "when you are happy with it"

Stage numbers are continuous across the whole manual, not per chapter, and the audit checks they match between editions. A third heading ("what to hand in") is worth adding if the stage is assessed.

---

## Figure slot

See `architecture.md` §4 for the mechanism.

```html
<figure class="shot" id="fig-thing">
  <div class="shot-frame">
    <img src="images/thing.png" alt=""
         onload="this.closest('.shot').classList.add('has-image')">
  </div>
  <div class="shot-todo">
    <span class="id">Figure 3</span>
    <p class="what">What the capture must show.</p>
    <span class="file">images/thing.png</span>
  </div>
  <figcaption>What to look at.</figcaption>
</figure>
```

The `.what` text is a brief for whoever takes the screenshot — including you in six months. "The ComfyQ queue with three jobs, one running" beats "the interface".

Captions say **what to look at**, not what the thing is. "The seam is visible along the left edge" beats "Screenshot of the result".

---

## FAQ entry

```html
<div class="faq-group">Un groupe de questions</div>
<details class="faq">
  <summary>La question, dans les mots de l'étudiant.</summary>
  <div class="faq-body"><p>La réponse.</p></div>
</details>
```

Phrase the summary the way a student would ask it, mistakes included — "why does my character change clothes between shots", not "on inter-shot consistency". The FAQ is a search surface: it works by matching the reader's vocabulary, not yours.

Answers stay short and point at the chapter. A long FAQ answer is a chapter section in the wrong place.

`.faq-body` is required — `buildIndex()` reads it, and the print stylesheet forces it visible.

---

## Symptom index

The most-used page in these manuals. A plain table, organised by **what the reader sees**, not by feature.

```html
<div class="tbl-wrap">
  <table>
    <thead><tr><th>Ce que vous voyez</th><th>Cause</th><th>Correction</th><th>Ch.</th></tr></thead>
    <tbody>
      <tr><td>Les vêtements changent de couleur</td>
          <td>La charte ne précise ni la couleur ni la matière</td>
          <td>Réécrivez la signature vestimentaire, puis retouche par instruction</td>
          <td><a href="#ch10">10</a></td></tr>
    </tbody>
  </table>
</div>
```

Each row is indexed separately, so a student searching their symptom lands on the row. Write symptoms in the reader's words; the chapter link is the escape hatch, not the answer.

---

## Glossary

One table, both languages in the same rows.

```html
<tr><td>Terme</td><td>Term</td>
    <td><span class="fr-only">Ce que ça veut dire ici.</span>
        <span class="en-only">What it means here.</span></td></tr>
```

Define terms **as this course uses them**, not as a dictionary would. The value is disambiguation, and the FR/EN pairing lets a French student read English documentation — which is most documentation.

---

## Spec list

For a card of facts: a model, a machine, a hardware requirement.

```html
<dl class="spec">
  <dt>VRAM</dt><dd>24 GB</dd>
  <dt>Licence</dt><dd>Apache 2.0</dd>
</dl>
```

Better than a two-column table when there is one subject and several attributes. Use a table when there are several subjects to compare.

## Two-column comparison

```html
<div class="grid-2">
  <div><h4>Voie A</h4><p>…</p></div>
  <div><h4>Voie B</h4><p>…</p></div>
</div>
```

Only for genuine either/or choices, and only when both sides are short. Collapses to one column on narrow screens, so the visual comparison is lost on a phone — if the comparison is the point, use a table with a row per criterion.

---

## Status and level chips

```html
<span class="lv-tag">A3+</span>          <!-- on a heading whose band is filtered -->
<span class="status">Prerelease</span>   <!-- on a tool that is not stable yet -->
<span class="ch">14</span>              <!-- companion pages: chapter cross-reference -->
```

`.lv-tag` on any heading inside a `data-level` block, so the reader understands why a section is absent at a lower band. The audit compares tag lists between editions.

---

## What not to add

- **Tabs** — they hide content from search and from print.
- **Accordions outside the FAQ** — a chapter that hides its own body is a chapter nobody reads.
- **Syntax highlighting** — needs a library, breaks the self-contained rule, and adds nothing to prompts or file trees.
- **A second accent colour** — the palette has one for a reason.
- **Icons and emoji in body text** — the README can have them; the manual should not.
- **A "next chapter" button** — the sidebar and scroll do this already.
