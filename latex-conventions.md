# LaTeX Writing Conventions

These rules govern the prose of every first-party LaTeX document (papers, reports, theses). They target the writing habits agents fall into most often.

## 1. Short Sentences

A sentence carries one idea.

- **No long, compound sentences.** If a sentence stacks several clauses with "which", "while", "whereas" or nested commas, split it.
- **Aim for sentences a reader parses in one pass.** When a sentence runs past about 25 words, it is probably two sentences.

```latex
% Bad
Our scheduler, which builds on prior work that assumed static workloads, adapts the partition at runtime while keeping the overhead low, whereas existing systems must be restarted.

% Good
Our scheduler adapts the partition at runtime. Existing systems must be restarted to do so. Its overhead stays low.
```

## 2. No Triple Lists

Do not write the "X, Y, and Z" pattern.

- **No rhetorical triads.** Phrases such as "fast, scalable, and robust" pad the text and claim more than they show. Keep the one property the paragraph actually supports.
- **Real enumerations go into a list.** When there are genuinely several parallel items, use `itemize` or `enumerate`, or give each item its own sentence.

```latex
% Bad
Our system is efficient, scalable, and easy to deploy.

% Good
Our system reduces tail latency by 40\%.
```

## 3. Define Every Term

A term is defined before it is used.

- **Define on first use.** Every technical term and abbreviation is introduced with a definition or expansion the first time it appears, e.g. "a translation lookaside buffer (TLB)".
- **One term per concept.** After defining a term, use exactly that term; do not switch to a synonym that the reader must guess is the same thing.
- **No undefined jargon.** Do not rely on terms the target reader cannot be assumed to know, even if they are common in a neighbouring field.

## 4. No Dashes

Do not use em dashes (`---`, `—`) or en dashes (`--`, `–`) as punctuation.

- **Restructure the sentence instead.** An aside set off by dashes becomes its own sentence, or a clause joined with a comma or a colon.
- **Ranges are allowed.** An en dash in a numeric range such as `pages 3--7` or `2019--2024` is not punctuation and stays.

```latex
% Bad
The cache --- unlike the buffer --- is shared across cores.

% Good
The cache is shared across cores. The buffer is not.
```

## 5. Paragraphs

A paragraph makes one point.

- **At most 6 sentences per paragraph.** A longer paragraph is split at its natural break.
- **Open with the point.** The first sentence states what the paragraph argues; the rest support it.
- **Separate paragraphs with a blank line** in the source, never with `\\` or `\newline`.
