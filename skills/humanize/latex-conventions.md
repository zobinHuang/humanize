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

## 6. Captions

A caption is a title, not a description.

- **State what is shown in a short phrase.** A figure, table or list caption names its content and nothing more.
- **Do not explain the structure.** No walkthrough of panels, bars, lines, rows, columns, colours or markers, and no definitions of the quantities shown.
- **Move what the reader needs into the body.** Definitions, footnote markers and caveats go in the text that references the figure or table, or in axis labels and column headers.

```latex
% Bad
\caption{Cost per clip. Panel (a) compares layouts on a log scale. The dark bar is our system. Rows marked with an asterisk are estimated.}

% Good
\caption{Cost per clip.}
```

## 7. Headings

A heading tells the reader what the section is about.

- **Name the topic in plain words.** A section or paragraph heading states its subject concretely, so a reader skimming the outline knows what each part covers.
- **No cryptic or clever headings.** Phrases such as "Two caches, two shared parts" or "One more thing" make the reader decode the heading before reading the section. Write the subject instead.

```latex
% Bad
\section{Two caches, two shared parts}

% Good
\section{Sharing the L2 and the last-level cache across cores}
```

## 8. Support Claims with Evidence

A claim is supported by data, citations or reasoning the reader can check.

- **No preaching.** Do not tell the reader what to think or feel: no "clearly", "obviously", "it is crucial to understand", "we must", "one should recognize".
- **Show the evidence.** Replace an insistence with the measurement, citation or derivation behind it, and let the reader draw the conclusion.

```latex
% Bad
Clearly, static partitioning is a fundamentally flawed approach that the community must abandon.

% Good
Static partitioning leaves 38\% of the cache idle on the mixed workload (Figure~4).
```

## 9. No Contrastive Framing

Do not write the "not X, but Y" pattern.

- **State Y directly.** Constructions such as "X is not A, but B", "this is not about A; it is about B" or "rather than A, it does B" set up a claim nobody made in order to knock it down. Say what the thing is.

```latex
% Bad
The bottleneck is not computation, but memory bandwidth.

% Good
The bottleneck is memory bandwidth: the kernel reaches 92\% of peak DRAM throughput and 18\% of peak FLOPs.
```

## 10. No Number Slogans

Do not compress a claim into a juxtaposition of numbers.

- **No slogan fragments.** Phrases such as "2 lines of code, 40\% improvement" or "10\texttimes{} less memory, same accuracy" set two numbers side by side and leave the reader to infer the message. They do not say what changed or what each number measures.
- **Write the claim as a full sentence.** Give it a subject and a verb, so the reader knows what changed and which metric each number belongs to.
- **State the message instead of implying it.** If the point is that a small change yields a large gain, write that sentence and back each half with its number.

```latex
% Bad
Two lines of code, 40\% faster.

% Good
The change adds a prefetch hint to the inner loop of the attention kernel.
It reduces the kernel's runtime by 40\%.
```

## 11. Logical Order

Text follows the order of the argument. A reader meets each idea after the ideas it depends on.

- **Premise before conclusion.** State the problem before the solution. Report an observation before what it implies.
- **Each sentence follows from the one before.** The link is either explicit, such as "because" or "as a result", or obvious from the content. A sentence that could move elsewhere in the paragraph without loss is in the wrong place or does not belong.
- **One order per section.** The paragraphs of a section follow a single ordering, such as cause to effect or general to specific. Keep it to the end of the section.
- **No afterthoughts.** When a point belongs earlier, move it there instead of appending it. Do not return to a point the text has already closed.

```latex
% Bad
Our scheduler partitions the cache at runtime. Static partitioning wastes capacity. It also handles workload phases. Existing systems use static partitioning.

% Good
Existing systems partition the cache statically. Static partitioning leaves capacity idle when the workload changes phase. Our scheduler repartitions the cache at runtime to follow these phases.
```
