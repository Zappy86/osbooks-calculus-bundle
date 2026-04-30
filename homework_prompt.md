Generate a complete, self-contained LaTeX document for my weekly homework assignment.
The section number and exercises are specified in my message.

Use ONLY this repository as source of truth:
- `@Zappy86/osbooks-calculus-bundle/files/modules`

Do NOT scrape openstax.org pages.
Do NOT use LibreTexts or any mirror.
Do NOT infer missing text.

---

## Inputs I will provide each week

I will not use explicit variable names. Parse my message directly to extract:
- `section`: a number in format `X.Y` appearing in my message. I may provide multiple sections with exercises for each one, in which case modify the latex accordingly with another title and another enumerate environment.
- `exercises`: all exercise numbers or ranges in my message
- `section_start`: the number of the first exercise in the section, provided explicitly as e.g. "starts at 313" or "section start: 313". Use this as the offset to map CNXML exercise order to page numbers — do not infer this from the exercise list.

For example:
- `4.7: 313, 317, 319, Odd 321--333, starts at 313` → `section: 4.7`, `exercises: 313, 317, 319, Odd 321--333`, `section_start: 313`
- `Section 4.7 odd 313-333, section start 313` → `section: 4.7`, `exercises: Odd 313--333`, `section_start: 313`

Do not ask for clarification if the section, exercises, and section start can be clearly parsed from my message.

---

## Repository navigation algorithm (strict)

The chapter-to-subcollection mapping is fixed. Use this exact table to resolve section `X.Y` (If you see the list of modules in the GitHub repo you will notice an extra one at the beginning of each chapter, these are unnumbered introductions and have been omitted from the table. Simply use the table and all will be well):

| Chapter | Title | Subcollection modules (in order = sections .1, .2, ...) |
|---------|-------|----------------------------------------------------------|
| 1 | Parametric Equations and Polar Coordinates | m53834, m53850, m53852, m53840, m53846 |
| 2 | Vectors in Space | m53900, m53897, m53902, m53903, m53870, m53874, m53875 |
| 3 | Vector-Valued Functions | m53913, m53916, m53919, m53930 |
| 4 | Differentiation of Functions of Several Variables | m53946, m53933, m53934, m53937, m53938, m53940, m53942, m53943 |
| 5 | Multiple Integration | m53949, m53963, m53966, m53965, m53967, m53971, m53970 |
| 6 | Vector Calculus | m53989, m54012, m53987, m53982, m53986, m54004, m54009, m54001 |
| 7 | Second-Order Differential Equations | m54040, m54047, m54044, m54046 |

To resolve section `X.Y`:
1. Look up chapter `X` in the table above.
2. Take the `Y`th module ID from that chapter's list.
3. Load `modules/m#####/index.cnxml`.
4. Extract the exact section title from that file (do not guess).

If `X` or `Y` is out of range, STOP with a failure report.

---

## Exercise extraction algorithm (strict)

1. In `modules/m#####/index.cnxml`, find node(s) with `class="section-exercises"`.
2. Only within that subtree, collect exercises in appearance order.
3. Ignore `<exercise>` elements outside `class="section-exercises"` (examples/activities are not assigned exercises).
4. Number exercises using `section_start` as the offset: first exercise in `section-exercises` = `section_start`, second = `section_start + 1`, etc.
5. Resolve the final exercise set from the `exercises` input by evaluating each sub-expression and taking the union, deduplicated and sorted.
6. Include exactly those exercises, no more, no less.
7. For each group of consecutive assigned exercises that share the same CNXML instruction `<para>` immediately preceding them in `section-exercises`, emit a `\item[]\textit{...}` line before the first exercise in that group. Use the exact text of that `<para>` but scope it to only the assigned exercises from the group (e.g. "For exercises 51 and 54, find the work done." becomes two separate lines if the exercises require different instructions). Do **not** emit an instruction line whose text does not accurately describe the exercises that follow it — give each logically distinct exercise or sub-group its own instruction line.
8. Convert math markup faithfully into LaTeX.
9. For exercises containing figures/images:
   - resolve asset paths from the module/repo
   - output raw GitHub URLs
   - insert figures using `\webimage{figN}{IMAGE_URL}` format from template comments

If any requested exercise text, math, or figure URL is missing/ambiguous, STOP with a failure report.

---

## Hard failure rule

If ANY required item cannot be confidently retrieved, do NOT generate LaTeX.
Return only:

### Retrieval Failure
- Requested section: `X.Y`
- Requested exercises: `[exercises]`
- Failed items:
  - `<item type>`: `<section title/exercise/image>`
    - Number (if applicable): `<N>`
    - Module ID: `<m#####>`
    - File(s) checked: `<exact repo paths>`
    - Reason: detailed explanation

No partial output.

---

## Validation checklist (must pass before writing file)

1. Section `X.Y` resolved via lookup table to module id.
2. Section title extracted from `modules/m#####/index.cnxml`.
3. Exercise numbers emitted exactly match requested set.
4. Every included image URL is fully resolved to a raw GitHub asset URL.
5. No bracket placeholders remain in final LaTeX.
6. Resolved exercise set exactly matches what was specified in `exercises` input — no missing, no extra.

If any check fails, trigger **Retrieval Failure** instead of writing the file.

---

## LaTeX template (strict)

Do not change the preamble, title formatting, macro definitions, or list formatting. Only fill placeholders and insert exercises.

```
% Compile with: pdflatex --shell-escape homework.tex (run it twice).

\documentclass[12pt, twoside]{article}
\usepackage{amsmath}
\usepackage{geometry}
\usepackage{enumitem}
\usepackage{graphicx}

\geometry{margin=1in}

\makeatletter
\renewcommand{\maketitle}{%
  \begin{center}
    {\LARGE\textbf{\@title}}\\[0.4em]
  \end{center}
  \vspace{-0.5em}
}
\makeatother

\newcommand{\webimage}[2]{%
  \immediate\write18{curl -L -o #1.jpg "#2"}%
  \includegraphics[width=0.45\textwidth]{#1.jpg}%
}

\title{Homework: Section [X.X] --- [Section Title]\\[0.3em]
{\large\textit{Calculus Volume 3} (OpenStax)}}

\begin{document}

\maketitle

\noindent\textbf{Instructions:} Complete the following exercises from Section~[X.X] of
\textit{Calculus Volume 3} by OpenStax (Strang \& Herman). [Odd / All] exercises [N--N].\\[0.3em]
\noindent\textit{Source: OpenStax repository source,} \texttt{Zappy86/osbooks-calculus-bundle}\textit{, Section~[X.X].}

\bigskip

\begin{enumerate}[leftmargin=*, itemsep=1.5em]

% [INSERT EXERCISES HERE]
% - Before each group of exercises sharing the same source instruction para, emit:
%     \item[]\textit{For exercise(s) N (and M), <instruction text>.}
%   Scope the instruction to only the assigned exercises in that group. If exercises
%   in the same source group require different descriptions, give each its own line.
% - Use \item[\textbf{N.}] for each exercise
% - Typeset all math in LaTeX
% - For exercises with figures, use:
%   \begin{center}
%     \webimage{figN}{IMAGE_URL}
%   \end{center}

\end{enumerate}

\end{document}
```

---

## Output contract (strict)

If validation passes:

1. Write the complete LaTeX document to `homework.tex` in the repository root using the template above.
2. Output a short confirmation line: `Generated homework.tex for Section X.Y ([exercises]).`
3. Remind me: `Compile with: pdflatex --shell-escape homework.tex (run it twice).`

Do not attempt to run pdflatex, simply remind me that I should.

---

## Additional guardrails

- Never renumber exercises outside section-exercises counting.
- Never merge/split exercises.
- Never fabricate missing text.
- Keep section title exactly as in module source.
