# Handoff Notes

Last session: 2026-06-27. Working through a translation cleanup pass on the
LGP-21 Subroutine Handbook.

## Where we left off

Mid-way through de-stilting `chapters/matrix-operations.tex`. The original
prose throughout that chapter reads like a 1960s German-to-English technical
translation ("utilizes the Floating-Point Interpretive System 1 to replace the
elements of...", "Prior to executing the calling sequence, an explicit exit
must be performed", etc.). We've been rewriting it paragraph-by-paragraph in
plain English, with you saying y/n on each draft.

`chapters/matrix-operations.tex` is now fully de-stilted:
- D1-11.0 Matrix Inversion 1
- D1-12.0 Matrix-Vector Multiplication 1
- D1-13.0 Matrix Multiplication 1
- D1-14.0 Matrix Addition and Subtraction 1
- D1-15.0 Matrix Transposition 1
- H1-10.0 Complex Operations Interpretive System

Still to do:
- **`chapters/floating-point.tex` -- the big one.** Same author voice as the
  matrix chapter but turned up to 11 -- "simulated pseudo-accumulator,"
  "architectural baseline footprint of the interpreter core loop," "operands
  and addresses can be modified dynamically without exiting the interpretive
  system environment," etc. ~1400 lines total. Going to be a multi-session
  job. Expect lots of the same patterns: paragraphs that say the same thing
  three ways, bullet lists that the table below restates, post-table prose
  that reads each row of the table back.

## House style we settled on during the cleanup

When you come back, ask me to follow these:

- **Subroutine name lead-ins** in PURPOSE blocks: bare `\texttt{X-Y.Z}` followed
  by the verb. Not "The program X-Y.Z calculates...". The subsection heading
  already names the routine.
- **Cell vs. location**: use "location" throughout (matches the existing
  `at location $\alpha+2$` phrasing). The original German was "Zelle".
- **Memory vs. temporary storage** in STORAGE REQUIREMENTS sections:
  - "Memory" = permanent program/data allocation (was previously "Program
    storage", "Storage cells used", etc.)
  - "Temporary storage" = scratch sectors (was previously "Buffer usage")
  - Word counts in parens after the track/sector figure, e.g.
    `3 tracks, 48 sectors (240 words)`. 64 sectors per track.
- **Scaling factor notation**: `$q=n$` everywhere. Not `@n`, not `$@n$`. The
  one exception is the explanatory paragraph in `chapters/introduction.tex`
  that names the alternative notations.
- **Degree symbol**: literal `°` (U+00B0), not `\textdegree`. The font in
  this document doesn't map `\textdegree` cleanly and renders it as `ř`.
- **Tables that need to stay under their `\paragraph{}` heading**: use
  `\begin{center}...\end{center}` rather than `\begin{table}[h!]...\end{table}`.
  The `table` environment is a float and LaTeX will hoist it past the
  heading. (This pattern bit us in arcsin/arccos and the log-functions
  exponential tables.)
- **Cross-references to other routines**: spell out the name, then point at
  the section. Not bare `\ref{H1-11.0}` -- that renders as just a number and
  the surrounding sentence reads as nonsense. Instead:
  `Floating-Point Interpreter 1 (Section~\ref{H1-11.0})`.
- **Tables with description columns**: prefer
  `\begin{tabularx}{\textwidth}{cccX}` over `\begin{tabular}{ccc p{6cm}}`.
  The `X` column auto-fits the remaining textwidth and avoids overfull
  warnings.
- **LLM-generated footnotes** ("Architectural Note: ...", "structural
  transcription discrepancies", "execution syntax mapping") -- those are
  almost always vacuous or wrong; rewrite in translator voice or just drop.

## Standard rewrite patterns we kept reusing

For every routine in the matrix chapter, the same prose blocks needed the
same patterns:

1. **PURPOSE first paragraph**: drop "The program utilizes the Floating-Point
   Interpretive System 1 (H1-11.0)" prefix and just fold it into the verb
   ("`X-Y.Z` does foo using the Floating-Point Interpretive System 1").
2. **PURPOSE second paragraph** (the "operates as a subroutine invoked via
   a calling sequence" bit): swap for "X is called as a subroutine. Exit the
   Interpretive System (with an `E` instruction) before the call. The calling
   sequence supplies ... . On return, ... ."
3. **INPUT block**: drop the "stored sequentially in row-major order (i.e.,
   the first element of the second row immediately follows ...)" explanation
   (everyone knows what row-major means by the second time). Keep dimensions,
   format requirement, and total-memory constraint.
4. **CALLING SEQUENCE intro**: compress the numbered list of "$X$: starting
   address in decimal track/sector notation" entries (which the table below
   restates anyway) into a single sentence: "The calling sequence supplies,
   in order: ..., ..., and ... . All addresses are in decimal track/sector
   notation."
5. **Prose after the table**: the routines all repeat each table row in
   prose. Replace with one sentence about the `E` instruction caveat and a
   pointer to Section~\ref{subsec:standard-call2}.
6. **OUTPUT**: "is written to consecutive locations starting at X, N elements
   total."
7. **NOTES / REMARK**: "The subroutine exits the Interpretive System on its
   own before returning, so control comes back to the main program in native
   (fixed-point) mode." (Standardized this exact wording across the chapter.)

## TODOs you mentioned for later

- **Verify the floating-point internal number format bit count.** The prose
  says "sign bit followed by 24 bits of precision" for the mantissa and
  "sign bit and 5 bits of precision" for the exponent (1+24+1+5 = 31 bits),
  but the example bit patterns in the diagrams (e.g. `01111100000000000000000000000100`)
  are 32 bits long. Could be a padding/spacer bit between fields, or could
  be a transcription error in either the prose or the diagram. Pinned down
  before we get into Internal Number Format / Internal Registers prose.


- **Consistency pass on "two distribution formats"** in the OPERATING DETAILS
  boilerplate. We left it alone in this pass because it's used consistently
  across all programs, but the wording could be tightened.
- Continue reading top-to-bottom for "personal infelicities" -- you said you'd
  do that yourself.
- Test the actual programs at some point so we can verify the alphanumeric
  keyword table (`alphanumeric-output.tex`) -- there's a known suspicious
  `A` vs `Q` in there that needs LGP-21 hardware to resolve.
- Watch for more leftover damage from the `@N` -> `q=N` automated sweep.
  Two known patterns:
  - `$X$@N$` adjacent math spans collapsing into `$X$q=N$$` (display math).
  - `at $@N$` inside a math span producing nested `$...$q=N$...$`.
  - Already swept twice; should be clean, but flag anything that looks off.

## Build / verify

```
tectonic -k lgp-21-subroutine-handbook.tex
```

The PDF is at the repo root and is committed. Build is currently clean of
errors; only cosmetic underfull/overfull `\hbox` warnings remain (all
< 30pt, mostly < 10pt). No undefined citations or references.

## Picking up next session

When you come back, say something like "let's continue de-stilting the
matrix chapter, starting with H1-10.0" and I'll read it and start showing
you draft rewrites paragraph by paragraph.
