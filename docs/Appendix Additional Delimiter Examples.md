### Appendix — Other Languages & File‑Formats Caught in “Escape Hell”

> *This appendix expands on the manifesto’s examples.  For each entry you’ll see: Current delimiter → Where it hurts → A clearer asymmetric idea.*

---

#### 1 · General‑Purpose Programming Languages

* **Python**
  • Current → `""" … """` triple‑quoted or `r"…"` raw strings.
  • Pain → Doc‑strings that themselves contain `"""`; regexes full of backslashes.
  • Clearer → heredoc `<<<PY … PY>>>`.

* **JavaScript / TypeScript**
  • Current → template literal `` ` … ` ``.
  • Pain → Nested code generation that itself uses back‑ticks; `${…}` interpolation clashes.
  • Clearer → `<tpl> … </tpl>` or tagged heredoc.

* **Rust** `r#"…"#`, `r##"…"##` …
  • Pain → Must *count* `#` levels in advance.
  • Clearer → `BEGIN_RS … END_RS`.

* **Swift / Kotlin**
  • Current → `""" … """` multi‑line strings.
  • Pain → Same counting issue as Python.
  • Clearer → `<<<SW … SW>>>`.

* **Go**  `` ` … ` `` raw strings
  • Pain → Cannot contain another back‑tick easily (think inlined Markdown).
  • Clearer → `[[[ … ]]]` or `BEGIN_RAW … END_RAW`.

* **Shell (Bash / POSIX)**  `<<EOF … EOF` heredocs
  • Pain → Need a unique terminator token; collision risk when body contains “EOF”.
  • Clearer → `<<"SCRIPT"` or explicit `BEGIN_SH … END_SH`.

---

#### 2 · Markup & Documentation Formats

* **Markdown / GFM**  ` ` fences
  • Pain → Code examples that contain their own back‑tick sequences → must lengthen outer fence.
  • Clearer → Named fences `~~~markdown` … `~~~` or `<md‑code> … </md‑code>`.

* **reStructuredText**  `::` literal blocks + indented code
  • Pain → Nested blocks inside lists require extra indentation; back‑ticks in default role.
  • Clearer → `.. code:: python‑raw\n   :delimiter: END\n   …\n   END`.

* **AsciiDoc**  `----` and `....` blocks
  • Pain → If body contains four dashes you chase longer fence lines.
  • Clearer → `++++` asymmetric start/end tags, or `<listing> … </listing>`.

* **LaTeX**  `\verb|…|` and `\begin{verbatim} … \end{verbatim}`
  • Pain → Choosing a delimiter char that never appears inside sample code; nested examples snowball.
  • Clearer → Custom named environments already help; even clearer would be `\begin<< … >>end`.

---

#### 3 · Data & Configuration Files

* **CSV / TSV**  `"…"` quoted cells
  • Pain → Fields with inner quotes demand doubled quotes and doubled escapes when embedded again.
  • Clearer → Future CSV spec with wrapper `⟦ … ⟧` or semicolon‑separated raw blocks.

* **INI / .env**  `key="value"`
  • Pain → Quotes inside value or multi‑line data cause fragile escaping.
  • Clearer → Here‑doc style `key=<<VAL … VAL`.

* **YAML literal / folded scalars**  `|` and `>`
  • Pain → Embedding another YAML or script containing `|` at column 0 ends the block unexpectedly.
  • Clearer → `!raw |-` with explicit terminator `END_RAW`.

* **HCL / Terraform**  `<<EOF … EOF` heredoc
  • Pain → Same unique‑token gamble as shell; double nesting when JSON‑inside‑HCL‑inside‑bash.
  • Clearer → `<<"JSONRAW"` or `BEGIN_HCL … END_HCL`.

* **Protocol Buffers (text/JSONPB)**  `"…"` strings
  • Pain → Embedding JSON with quotes demands triple‑escaping.
  • Clearer → Raw‑block `<<JSON … JSON`.

---

#### 4 · Query, Schema & Template Languages

* **SQL**  `'…'` literals, `$$ … $$` dollar quotes
  • Pain → Generating migration scripts that themselves contain `$$`.
  • Clearer → `<<SQL … SQL>>`.

* **GraphQL SDL**  `""" … """` block‑strings
  • Pain → Documentation that shows examples containing `"""`.
  • Clearer → `<doc> … </doc>`.

* **Regex literals (PCRE / JS / Java)**  `/.../` or `r"…"`
  • Pain → Delimiters collide with slashes in URL patterns; backslash mania.
  • Clearer → `rx<< … >>rx`.

---

#### 5 · Build, CI & Container Tooling

* **Dockerfile**  `RUN <<EOF … EOF`
  • Pain → Same heredoc token collision; JSON inlined in shell inlined in Docker is hair‑pulling.
  • Clearer → `RUN <<"SCRIPT"` or explicit `BEGIN_DOCKER_SH … END_DOCKER_SH`.

* **Makefile**  `define … endef`
  • Pain → Recipe that includes another `define` requires renaming terminators.
  • Clearer → `<<MAKE … MAKE`.

---

#### 6 · HTML, Templates & Embedded Resources

* **HTML inside templates**  `<script> … </script>` / `<style> … </style>`
  • Pain → When a template generator outputs literal `</script>` the browser misparses.
  • Clearer → `<script raw> … </script>` where terminator ignored when in *raw* mode.

---

### Minimal Patterns That Solve (Almost) Everything

> **Good news:** after surveying dozens of languages, nearly all “escape‑hell” scenarios collapse into just **two** robust delimiter strategies.

1. **Explicit *Tagged* Begin/End Pairs**
   *Syntax:* `BEGIN_JSON … END_JSON`, `<raw> … </raw>`, `<<<SQL … SQL>>>`.
   *Why it works:* Start and end markers are lexically distinct; nesting is trivial with a stack; tags can be arbitrarily descriptive.
   *Adoption path:* Many languages already support user‑chosen heredoc tags or XML‑like elements—formalising a reserved tag is a low‑impact extension.

2. **Rare‑Code‑Point Single‑Char Pairs**
   *Syntax:* `⟪ … ⟫`, `⟬ … ⟭`, or any Unicode pair extremely unlikely to appear in ordinary source.
   *Why it works:* One‑character markers are concise, visually unique, and nestable by simple push/pop.
   *Adoption path:* These symbols already exist in Unicode; tooling merely needs to recognise them as raw‑block delimiters when a feature flag is on.

> *Everything else*—multi‑hash raw strings, lengthened back‑tick fences, ad‑hoc heredoc tokens—are special cases of pattern ① where the tag is implicit or derives from delimiter length.

---

### Common Failure Modes (Why Symmetric Delimiters Hurt)

1. **Delimiter‑Length Arithmetic** – Languages like Rust, Markdown, and C# raw strings make authors *count* hashes or back‑ticks in advance.
2. **Unique‑Token Roulette** – Heredocs in shell, Terraform, or HCL choose a terminator token and *hope* it never appears inside the body.
3. **Escaping Explosion** – CSV fields, JSON‑in‑string, or regular expressions lose readability as every quote spawns extra backslashes.

---

*Last updated ✱ 2025‑05‑20 ✱ by ChatGPT o3 Model Instance — “Still counting fewer backslashes, one proposal at a time.”*
