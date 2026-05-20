# CE Document Mapper v16

## What changed
- added consistent field colour-coding across:
  - extracted values on the left
  - source text preview in the middle
  - provider mapping controls on the right
- added **Accident Circumstances** as a new field
- added **Use Current Date** beside **Inspection Date** in Provider Setup
- removed the extra preview notice about fixed-position line numbers

## Mapping methods
Each field can use one of these methods:
- **Single Label**
- **Two Labels**
- **Fixed Position**
- **Manual Input**

## Notes
- legacy `.DOC` import still tries `antiword` first, then Microsoft Word, then LibreOffice
- `.DOCX` textbox text is included in the source preview
- exports still go to the Desktop
- settings and mappings are stored in `Documents\CE Document Mapper`


## New in v20

- Added **Export JSON String** to export all fields as a JSON file to the Desktop and copy the JSON string to the clipboard.


New in v25:
- Added VAT Status field.
- VAT Status supports Yes/No logic: map the Yes condition; if matched => Yes, otherwise No.


Address handling update (v35)
- Inspection Address is normalized into a fixed 6-line structure in detected data and JSON export.
- In Provider Setup, the Inspection Address row now includes a Force Postcode option for single-line address strings.
- The Detected Fields panel shows Inspection Address in a 6-line text box.


Importer improvements in v41:
- PDF text extraction now prefers PyMuPDF for better table/text rendering and falls back to pypdf with /uniXXXX decoding.
- DOCX extraction now includes header and footer text where available.
- DOC extraction now prefers Microsoft Word or LibreOffice before antiword so headers/footers are more likely to be included.


## V42 — PDF table extraction

- Switched the PyMuPDF reader from `get_text("text", sort=True)` to
  `get_text("blocks", sort=True)`. PDFs whose data sits inside multi-column
  table-style boxes (e.g. engineer reports with Vehicle / Reg No / Vin No
  / Damage cells laid out in a grid) now keep each cell on its own line
  instead of merging neighbouring columns onto the same line.
- This fixes mapping rules silently grabbing content from adjacent
  columns. For example, on a typical engineer-report layout, the VRM
  field used to come out as `JR07CVRREGISTERED:MAY2016TYPE:ESTATETRANS:AUTOMATIC`
  because `Registered: May 2016`, `Type: Estate` and `Trans: Automatic`
  were on the same line as `Reg No: JR07CVR`. Each is now its own block.
- If a page produces no blocks, the reader falls back to the original
  text-mode extraction so nothing is lost on unusual PDFs.
- The `/uniXXXX` decoder remains in the pypdf fallback path.
- Note: the **Fixed Position** mapping method uses absolute preview line
  numbers, and block mode introduces blank lines between blocks. Existing
  Fixed Position rules that target a PDF will likely need their line
  numbers updated. No saved providers in the default config use Fixed
  Position, so most users are unaffected.
- DOC, DOCX, EML and MSG paths are unchanged.


## V43 — Mileage and Mileage Unit special rules

### Mileage: numeric extraction

The Mileage field now post-processes whatever the user's mapping rule
returns to extract just the numeric portion. Leading non-digit
characters are skipped; once digits start, digits are collected and
commas inside the number are stripped; the first non-digit, non-comma
character ends the number.

This means a Single Label rule with just `Speedo:` against source text
`Speedo: 28,487 Miles` returns Mileage = `28487`. The user no longer
has to use Two Labels to strip the trailing unit, which matters because
the trailing unit varies by document (Miles, miles, mi, km, …).

Behaviour is the same regardless of mapping method — the digit-only
post-processing applies to whatever the configured method returns.

### Mileage Unit: presence-check rule (Miles / Km / blank)

The Mileage Unit field now follows a fixed Yes/No-style rule the same
way VAT Status does. The user enters comma-separated tokens describing
how "miles" appears in their documents (e.g. `miles, mi`):

- If any configured token is found anywhere in the source text → `Miles`
- Otherwise → `Km`
- If the config is left blank → blank

The Method dropdown is hidden for Mileage Unit since the method choice
is irrelevant — only the configured tokens matter. The same applies
to VAT Status now: its dropdown is also hidden.

### VAT Status: simplified

Previously VAT Status had a multi-tier matching strategy with regex
fallbacks and synonym lists. This has been replaced with the same
simple presence check used by Mileage Unit:

- If any configured token is found anywhere in the source text → `Yes`
- Otherwise → `No`
- If the config is left blank → blank

The user is responsible for picking tokens that uniquely identify the
positive scenario in their documents (e.g. picking `VAT registered`
when documents may also contain `not VAT registered` is on the user
to disambiguate). No regex magic, no synonym fallback — full user
control.

### Legacy preset migration

Old presets that used Two Labels for VAT Status or Mileage Unit
(`"foo || bar"` config) are migrated automatically when loaded: just
the start label is kept as the new presence-check token. The first
time the user opens such a preset and saves it, the file is rewritten
in the new single-label form.


## V44 — Engineer Report override highlighting

When an Engineer Report overwrites one or more fields, those fields now
get a permanent red border around their value entry on the left panel
(Detected Fields). The highlight is purely visual — the underlying
value is still editable.

Rules:
- A field is highlighted only when the Engineer Report actually
  *changed* its value. If the engineer's value is identical to what
  was already there, no highlight (it wasn't really replaced).
- Blank fields in the Engineer Report don't overwrite anything, so
  they don't trigger a highlight either.
- The highlight stays on screen until a fresh document is dragged
  in (single instruction, single engineer report, batch, or combined
  drop) — at which point all override highlights are cleared and only
  the new drop's overrides are shown (if any).
- Both engineer-report flows are covered: dropping the engineer report
  *after* an instruction has already loaded, and dropping an instruction
  + engineer report at the same time.


## V45 — Export gated on Work Provider

JSON export and Image export now silently no-op when the Work Provider
field is empty. Applies to both single-import and batch-import:

- **Single import**: clicking Export JSON or Export Images with an
  empty Work Provider does nothing — no file written, no popup, no
  status change.
- **Batch import**: each session is checked individually. Sessions
  with a populated Work Provider are exported as normal. Sessions
  with an empty Work Provider are silently skipped. The remaining
  files still export correctly.

This prevents unidentified documents (e.g. an image-only PDF that
matched no provider) from producing stray ``UnknownVRM.json`` or
``UnknownVRM_img_*.png`` files on the Desktop.

The behaviour is intentionally silent — it's expected, productivity-
oriented behaviour, not an error condition.


## V46 — Native .msg import via extract_msg

Replaced the old Outlook COM-automation .msg path entirely with a
pure-Python reader based on the ``extract_msg`` library.

### Why

The previous .msg path used Outlook automation. In real-world use this
froze the application indefinitely whenever Outlook had a profile
prompt, modal dialog, or background sync. There was no timeout and no
recovery. Behaviour was different on Classic Outlook vs New Outlook
vs 365.

### What changed

- `_extract_msg_text_via_outlook` is gone.
- A new `_extract_msg_text` reads the .msg's compound-file structure
  directly, with no COM, no Outlook process, and no UI dialogs.
- Output shape mirrors the existing `.eml` path so provider mapping
  rules behave identically against either format:

  ```
  Subject: ...
  From: ...
  To: ...
  Cc: ...        (only when present)
  Date: ...

  <body>

  Attachments: a.pdf, b.docx     (only when present)
  ```

- Body preference: plain text → HTML (stripped) → RTF (stripped, very
  rare fallback).
- Attachment filenames are listed informationally at the end.

### Implications for users

- Works on any Outlook version: Classic, New, 365, web. Saves go
  through `File → Save As → .msg` as normal.
- The previous .txt-export workaround for Classic Outlook is no
  longer needed — the .msg file itself imports cleanly.
- No Outlook needs to be running on the user's machine for .msg
  import to work.
- New dependency: ``extract-msg>=0.45.0`` (pure Python, ships fine
  via PyInstaller).
- ``pywin32`` is no longer used for .msg (still listed in
  requirements as it was used by the old DOC path on Windows).


## V47 — .msg HTML body decoding (hotfix)

V46 added native .msg import. Real-world testing revealed the body
came through as ``b'<raw HTML with literal \r\n>'`` instead of being
decoded and stripped to plain text. Three root causes, all fixed:

1. **Bytes-typed HTML body.** ``extract_msg`` returns
   ``htmlBody`` as ``bytes``. The code was calling ``str()`` on it,
   which produced the ``b'…'`` ``repr`` literal and tricked the
   "do I have a plain body" check into thinking we had real text.
   Now there's a proper coerce step that decodes bytes (utf-8 →
   cp1252 → latin-1) and strips trailing null bytes.

2. **HTML stripper too thin.** ``_strip_html_tags`` previously
   handled only ``<br>`` and ``</p>``. Outlook HTML is full of
   ``<style>`` blocks, VML/Office namespace elements, comments,
   non-breaking spaces, and HTML entities. The stripper now drops
   ``<style>`` / ``<script>`` blocks wholesale, decodes both named
   and numeric HTML entities, and translates Windows-1252 smart
   quote bytes that occasionally leak through.

3. **``body`` sometimes contains HTML.** Some Outlook saves put
   HTML markup directly into the plain ``body`` property. We now
   detect that (looking for ``<html``, ``<body``, ``<o:p``, ``<v:``,
   etc. at the start) and strip it just like ``htmlBody``.

This also incidentally improves the ``.eml`` HTML-body fallback,
which uses the same ``_strip_html_tags`` helper.

### Verified

Pinned by a regression test (``test_v47.py``) using a real Outlook
``.msg`` from a UK firm: instructions email with HTML-only body, two
embedded images, VML/Office namespaces, ``&nbsp;`` and ``&amp;``
entities, cp1252 smart-quote bytes, and null-terminated subject and
attachment names. After the fix all those artefacts are gone and
labelled values like ``Vehicle Registration Number: KP22LRL`` come
through clean.


## V48 — Force Postcode handles trailing content

The Force Postcode option on Inspection Address now also handles the
common case where a single-line address ends with a phone number after
the postcode, e.g.

    Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham B5 6JX 07462530375

Previously Force Postcode only matched when the postcode was at the
very end of the line (trailing whitespace allowed but nothing else).
With this trailing phone number it would silently fall back to leaving
the whole line as line 1 — postcode never extracted.

Now: if the end-of-line postcode pattern fails, we look for a postcode
*anywhere* in the single-line address. When found, the line is split
there. Everything before the postcode becomes the address body
(line 1), the postcode goes to line 6, and any trailing content (the
phone number) is dropped. The user still has to manually break up
line 1 across lines 2-5 for clean RJS export — that's a deliberate
limitation; fully automated splitting of a free-form business
address into Name / Address / Town / City / County is a lost cause.

### Behaviour summary

- Force Postcode OFF: unchanged. The whole line stays on line 1
  and the postcode isn't extracted.
- Force Postcode ON, postcode at end-of-line: unchanged.
- Force Postcode ON, postcode mid-line with trailing content
  (phone number, etc.): NEW. Line splits at the postcode; trailing
  content is dropped.
- Multi-line addresses: unchanged. Continue to work the same way.


## V49 — Image export always runs

Image export no longer requires a Work Provider. It runs whether or
not the document has been identified, and uses the source document's
filename as the base name when there's no Work Provider.

### Filename rules

- Work Provider set, VRM set: ``Provider_VRM_img_N.<ext>`` (unchanged)
- Work Provider set, VRM blank: ``Provider_UnknownVRM_img_N.<ext>`` (unchanged)
- Work Provider blank: ``SourceStem_img_N.<ext>`` — the source document's
  filename (without extension) becomes the prefix. So dragging in
  ``Images_.docx`` with no provider produces ``Images__img_1.jpeg``,
  ``Images__img_2.jpeg`` and so on.
- Source filename is run through ``safe_filename`` so spaces become
  underscores and unsafe characters are dropped.

### What's still gated

- **JSON export** still requires Work Provider (single mode no-ops
  silently; batch mode silently skips unidentified sessions). That
  rule is unchanged — it's specifically about preventing stray
  ``UnknownVRM.json`` files for unidentified docs.

### Batch mode

In a mixed batch — some docs identified, some not — Image export
processes every doc, each with its own appropriate base name.
Identified docs use ``Provider_VRM_*`` and unidentified docs use
their source filename. JSON export still skips the unidentified ones.

### Underlying capability

This wasn't actually a new capability addition — DOCX image
extraction has been working correctly under the hood since the
original V40 code. The reason users couldn't get it to fire on
image-only DOCX files was the V45 Work Provider gate, which we
added to prevent ``UnknownVRM.json`` outputs but unintentionally
also blocked Image export. V49 untangles those: JSON export stays
gated, Image export doesn't.


## V50 — Engineer Report highlight on touch, not just on change

The Engineer Report red-border highlight on the Detected Fields panel
now fires whenever the engineer report produced a non-blank value
for a field, regardless of whether that value differed from what was
already there. Previously it only highlighted fields whose values
*changed*.

The highlight now serves a slightly different purpose: it confirms
which engineer-report mapping rules fired at all, including ones
that produced a value matching the instruction's value. Useful for
verifying that a newly written engineer-report preset is doing what
the user thinks it's doing.

### Behaviour summary

- Engineer report extracts a non-blank value for a field → red
  border on that field's value entry, regardless of whether the
  value changed.
- Engineer report extracts a blank / missing value → no border (the
  field wasn't touched).
- Highlight clears when a fresh document is dragged in (unchanged).
- Both engineer-report flows are still covered: drop-engineer-after-
  instruction and combined-drop.


## V51 — Provider auto-detection requires all phrases (AND, not OR)

Provider detection now requires **every** phrase in
``detect_phrases`` to be present in the document. Previously any
single phrase would match the provider; that meant a provider with
two phrases didn't reliably win over one of its phrases shared with
a different provider.

### The motivating case

Two providers shared a base phrase but one needed an additional
fingerprint to identify a specific document shape:

```
FW (Garage):    ["fairwaylegal", "Inspection Location:"]
FW (Solicitor): ["fairwaylegal"]
```

A "Solicitor" document (which only has ``fairwaylegal``) was being
detected as ``FW (Garage)`` when it appeared first in the providers
list, because the OR-of-phrases rule scored both providers equally
on the single match. With AND-of-phrases:

- Solicitor doc → only ``FW (Solicitor)`` matches; Garage is
  filtered out because ``Inspection Location:`` is absent.
- Garage doc → both providers match, but Garage wins on the
  multi-phrase tiebreaker (more phrases matched, longer total).

### Behaviour change

This is a deliberate breaking change to the detection semantic.
Any existing provider with multiple ``detect_phrases`` will now
require **all** of them to match. Providers with a single phrase
behave identically to before. Providers with no phrases continue
to be skipped.

In the user's current settings, only the two FW providers have
multiple phrases, so this change only affects FW detection.


## V52 — Hyphen labels preserved

A user-typed label of just ``-`` (or any label ending or starting
with a hyphen) was being silently erased when saving a provider
preset. The cause was ``clean_value`` calling
``strip(" :-\n")``, which stripped hyphens from both ends of the
string. For a single-character label ``-`` that left an empty
string. After save/load the user saw their label gone.

Fix: ``clean_value`` no longer strips hyphens. It still strips
surrounding whitespace and leading/trailing colons (which is what
makes ``Address:`` and ``Address`` interchangeable as labels), but
hyphens are now treated as ordinary characters.

Effect:

- Two Labels rule with ``Section || -`` saves and loads as-is.
- Single Label rule with ``-`` preserved.
- Labels with trailing or leading hyphens (``Acc-Date-``) preserved.
- Trailing-colon behaviour for ``Address:`` is unchanged — that
  was always handled by stripping ``:`` and continues to work.


## V53 — Fixed Position + Label mapping method

Added a fifth mapping method, **Fixed Position + Label**.

### How it works

The user enters two values: a 1-based line number (matching the
source preview) and a label. The extractor goes to that exact
line and returns the text after the configured label on that
line. If the line doesn't contain the label, the rule returns
empty (forcing a manual edit), so a fragile rule never produces
silently-wrong output.

### When to use it

Useful when a label like ``-`` would be too ambiguous to use as
a Single Label across the whole document, but is unambiguous once
narrowed to a specific line. Real-world example: an engineer
report with the VRM as the trailing token after a hyphen on a
specific line:

```
Mobile:
07936853974
Vehicle:
Toyota Prius - BR19 SRX
```

Configured as line ``4`` and label ``-``, the rule returns
``BR19 SRX``.

### UI

The dropdown gains a fifth option after the existing four. The
config UI shows two text fields side-by-side (the same shape as
Two Labels): line number on the left, label on the right.

### Storage format

Saved as ``"<line> || <label>"`` in ``providers.json``, sharing
the same ``||`` delimiter and parser as Two Labels.

### Side improvement: literal label preservation

While implementing this, the ``parse_two_label_config`` helper
was changed to preserve user-typed labels exactly (only stripping
surrounding whitespace). Previously it ran labels through
``clean_value``, which stripped leading/trailing colons. That
meant a user typing ``:`` or a colon-terminated label would lose
those characters at parse time. The downstream matching code
already does its own colon handling, so this is purely a
preservation improvement and doesn't change matching semantics
for the existing Two Labels method.


## V54 — OCR fallback for image-only scanned PDFs

Some clients send instructions as a single-image scanned PDF — the
PDF has no text layer at all, just a photographed/scanned image of
the page. Existing imports of these returned empty text and no
fields could be mapped.

V54 adds an OCR fallback that fires automatically and silently when:

- The PDF has **exactly zero** text characters across all pages, AND
- Every page contains **exactly one** image.

Both conditions must hold. This is a deliberately strict trigger:
- A photo-dump PDF (multiple images per page, with or without text
  captions) never triggers OCR — its images are a different kind of
  attachment, not a scanned letter.
- A normal text PDF with logos, watermarks, or signature images
  never triggers OCR — text extraction returns characters, the gate
  closes.
- A multi-page scanned letter where every page is a single image
  triggers OCR.

When OCR fires, every page is rendered at 300 DPI and run through
Tesseract. A status note records that OCR was used; otherwise the
flow is identical to a normal PDF import. Mapping rules, field
extraction, exports, and Engineer Report logic all work against the
OCR'd text without any further changes.

### Bundling Tesseract for the .exe (Windows build instructions)

The app expects a Tesseract binary alongside the .exe. ``pytesseract``
is installed via pip but it does NOT ship the OCR engine itself —
the engine is a separate binary that needs to be bundled.

Steps for the Windows build machine:

1. Download a Tesseract Windows installer from the UB Mannheim
   builds (https://github.com/UB-Mannheim/tesseract/wiki). Pick a
   recent stable release — tested against 5.x.

2. Run the installer. By default it lands in
   ``C:\Program Files\Tesseract-OCR\``. Note this folder.

3. Copy the relevant files into a ``tesseract`` subfolder next to
   ``app.py`` in the project:

   ```
   project/
     app.py
     ce_document_mapper.ico
     tesseract/
       tesseract.exe
       tessdata/
         eng.traineddata
       (any DLLs that ship with tesseract.exe — copy them too)
   ```

   Only the English trained-data file (``eng.traineddata``) is
   needed; you can delete the others to keep the bundle smaller.

4. Build with PyInstaller, adding the tesseract folder as a data
   inclusion:

   ```
   pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
       --add-data "ce_document_mapper.ico;." ^
       --add-data "tesseract;tesseract" ^
       app.py
   ```

The .exe will jump from ~30 MB to ~60-80 MB depending on the
Tesseract version. Bundle once, ships everywhere.

### What happens if Tesseract is missing

If the .exe is run without Tesseract bundled, or on a machine where
Tesseract isn't on PATH, the OCR fallback simply doesn't fire — the
import returns empty text the way it did before V54. No crash, no
error popup, no degradation for users who never receive scanned
PDFs.

### Performance

Per-page OCR adds roughly 1-3 seconds. A 1-page scanned letter is
imported in under 2 seconds; a 5-page scanned bundle in 5-8 seconds.
Existing text PDFs are unaffected — their import time is unchanged
because the OCR check fails immediately on any text being present.


## V55 — Single Label +/- mapping method

Added a sixth mapping method, **Single Label +/-**, plus a small UI tweak.

### How it works

The user enters two values: a literal label (case-insensitive, same as
Single Label) and a signed-integer offset like ``-2`` or ``+1``. The
extractor finds the first line in the document containing that label,
then returns the line ``offset`` lines above or below it. The +/- sign
is required — bare ``2`` is rejected, since the strictness makes the
direction unambiguous.

### When to use it

Designed for OCR'd documents where a value sits on a predictable line
near a recognisable label, but where Fixed Position alone won't work
because OCR-derived line numbering varies between scans of the same
form. For example, "the line right below 'Inspection Address:'":

```
Inspection Address:
12 High Street, Birmingham B12 9XY
```

Configured as ``Inspection Address: || +1`` returns the address line.

### Edge cases

- Out-of-bounds offsets clamp to document edges. ``-10`` from line 5
  returns line 1; ``+99`` from the last line returns the last line.
- Label not found anywhere in the document → empty (forces manual
  edit, no silent silly fallback).
- Multiple occurrences of the label → the first one anchors the offset.
- Empty / malformed offset → empty.

### UI tweak: removed method explanation block

The paragraph of text under the rules table that explained each
mapping method ("Single Label = one label, then take the value...";
"Two Labels = take everything between..."; etc.) has been removed.
The column headers (``Field`` / ``Method`` / ``Config``) remain,
keeping the table visually anchored. The rationale: with six mapping
methods now, the explanation paragraph would have grown to twice the
width and stopped being a quick reference. Users can refer to this
README or the dropdown's clear method names instead.

### Storage

Saved as ``"<label> || <offset>"`` in ``providers.json`` — same
``||`` convention as Two Labels and Fixed Position + Label, parsed
through the same helper.


## V56 — Work Provider separated from preset name

The Work Provider field is now independent of the preset name.

### Why

Some real-world providers send multiple document formats and need
multiple presets — e.g. ``FW (Garage)`` and ``FW (Solicitor)`` both
serve documents from "Fairway Legal" but with different layouts. Up
to now the Work Provider value in the JSON export was always equal
to the preset name, so the JSON ended up with
``"work_provider": "FW (Garage)"`` rather than the canonical
``"work_provider": "FW"`` that downstream management software
expects.

### What changed

- A new **Work Provider** row at the top of the rules table on the
  right panel. Manual-input only — no method dropdown, just a text
  entry. The user types whatever the canonical Work Provider value
  should be for that preset (e.g. ``FW``), independently of the
  preset name (which can be any descriptive label, e.g.
  ``FW (Garage)``).
- When a document is detected, the Detected Fields panel and the
  JSON export now show the Work Provider value from the preset's
  rule, not the preset name.
- Storage: stored as a normal field rule
  ``{"method": "manual_input", "config": "FW"}`` inside
  ``field_rules`` — same shape as every other rule.
- Right-panel rules table is now ordered to match the left-panel
  Detected Fields order: Work Provider, VRM, Vehicle Model,
  Claimant Name, Reference, Incident Date, Instruction Date,
  Inspection Date, Inspection Address, Accident Circumstances,
  VAT Status, Mileage, Mileage Unit. (Previously Mileage and
  Mileage Unit appeared before Accident Circumstances and VAT
  Status on the right.)

### Behaviour for existing presets

Existing presets that don't have an explicit ``work_provider`` rule
get a default blank rule injected by ``normalize_provider_config``.
On import, a document matching that preset will show **blank** in
the Work Provider field — forcing the user to open the preset and
fill it in, which is the intentional V56 behaviour. Downstream JSON
exports are also gated on Work Provider being non-empty (per V45),
so a forgotten Work Provider on an existing preset will silently
prevent JSON export until the user updates the preset.

## V56 — Bundled providers.json seeding

The .exe can now ship with a default ``providers.json`` baked in.

### How it works

On first launch, when ``Documents\CE Document Mapper\providers.json``
doesn't yet exist:

1. The app looks for a ``providers.json`` bundled alongside the
   .exe (or next to ``app.py`` in development) via the same
   ``resource_path`` helper used for the icon.
2. If found, the bundled file is copied to the user's Documents
   location and used as the starting set of presets.
3. If not found (or unreadable), the app falls back to the existing
   hardcoded ``DEFAULT_CONFIG`` (which contains the default RJS
   preset).

Subsequent launches load whatever's in Documents — the seeding only
runs when no file exists. Users who already have a providers.json
keep theirs intact.

### Updated build command

To bundle ``providers.json`` into the .exe, drop the file next to
``app.py`` and add an ``--add-data`` line:

```
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "providers.json;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
```

The bundled file is read-only at runtime; users edit the copy in
their Documents folder.


## V57 — Strip Word cell-terminator BEL bytes from .DOC imports

Fixed a bug where some legacy ``.DOC`` files imported via the
Microsoft Word automation path produced visible "tofu square"
characters at the start of lines in the source preview, and
exported as ``\u0007`` in the JSON output.

### Root cause

Word's COM API uses ``\x07`` (the BEL control character) as a
table-cell terminator inside ``Range.Text``. Each cell in a Word
table is delimited by ``\x07`` on the way out via COM. The
existing extraction code already handled the ``\r\x07`` sequence
that ends a row in headers/footers, but lone ``\x07`` cell
terminators in the main document body and inside table cells
were being passed through untouched.

When a label-based mapping rule pulled a value from a table cell
(common with ``Name``, ``Address``, ``Date`` style instructions),
the trailing ``\x07`` ended up in the extracted value. It rendered
as a tofu square in the preview (Tk doesn't have a glyph for the
BEL control character) and JSON-serialised as ``\u0007``.

### Fix

``\x07`` is now stripped from the main content (``doc.Content.Text``)
and from header/footer text before any further processing, in the
``_extract_doc_text_via_word`` path only. No other extraction path
is touched — LibreOffice, antiword, DOCX-direct, PDF, EML, MSG all
remain identical to V56.

Performance impact: zero. The strip is a single ``str.replace``
call on the already-extracted string.


## V58 — Single Label +/- skips blank lines

The Single Label +/- mapping method now ignores blank lines when
counting the offset. ``+1`` returns the first non-blank line below
the anchor; ``+2`` the second; etc. Negative offsets behave the
same way going up.

### Why

OCR'd documents and some legacy formats produce stray blank lines
between rows that would otherwise throw off a strict +/- offset.
For example::

    Customer Information
    <blank>
    Mr Tester
    12 High Street

Previously ``Customer Information || +1`` returned the blank line.
Now it returns ``Mr Tester`` — the user can write the rule in terms
of "the first non-blank line after the label" without having to
count OCR-introduced blanks.

### Behaviour summary

- Positive and negative offsets both skip blank lines.
- Whitespace-only lines (lines containing only spaces or tabs)
  count as blank.
- ``+0`` / ``-0`` still return the anchor (label) line itself.
- Out-of-bounds offsets clamp to the last/first **non-blank** line
  in the requested direction. If there are no non-blank lines in
  that direction, the rule returns the anchor line.
- Label not found in the document → empty (unchanged).


## V59 — OCR fallback page-count cap

Fixed an issue where a 26-page photo-dump PDF (where each page is a
single full-page photograph) caused the app to freeze for ~60-90
seconds while OCR ran on every page synchronously.

### Root cause

The V54 OCR fallback fires on PDFs with zero text and exactly one
image per page — intentionally narrow, designed to catch scanned
letters. But photo-dump documents (a bundle of camera photos, one
per page) often satisfy both conditions: the PDF wraps each photo
on its own page, with no extra text. With 20+ pages, OCR runs for
30-90 seconds blocking the main thread.

### Fix

OCR now requires a third condition: the document must be at most
``OCR_PAGE_LIMIT`` pages (default: 2). The threshold was chosen
based on the observed envelope: real scanned-letter instructions
in the wild are 1-2 pages; image-dump documents start at 3+ pages.
``OCR_PAGE_LIMIT = 2`` is the dividing line.

### Behaviour

- 1-page scanned letter → OCR fires (V54 behaviour preserved)
- 2-page scanned letter → OCR fires (V54 behaviour preserved)
- 3+ page document with no text and one image per page → OCR is
  skipped, document imports near-instantly with no text
- All other PDFs → unchanged (OCR was never going to fire on them)

### Tuning

The limit lives at module top as ``OCR_PAGE_LIMIT = 2``. If a
legitimate scanned letter ever exceeds 2 pages, the constant can be
raised in one place; no other code changes needed.


## V60 — Inspection Address always exports as 6 lines

Fixed a JSON-import error in the user's downstream management
software when the inspection address was empty.

### Symptom

The management software's JSON importer requires
``Inspection Address`` to always be a 6-line value (5 newlines
separating 6 fields, where empty fields are still represented as
empty strings between the newlines). Any other shape — bare
``""``, 7 lines (``"\n\n\n\n\n\n"``), 4 lines, etc. — failed to
import.

### Cause

``normalise_inspection_address_value`` returned a bare empty
string when the input had no content (no provider rule, blank
extracted value, or whitespace-only). On JSON export this became
``"Inspection Address": ""`` — failing the import schema.

### Fix

``normalise_inspection_address_value`` now always returns the
canonical 6-line shape:

- Empty input → ``"\n\n\n\n\n"`` (5 newlines, 6 empty fields).
- Whitespace-only input → same.
- Populated input → unchanged behaviour: 6-line normalised
  address with body lines 1-5, postcode in line 6.

The export pipeline already calls this normaliser as part of
``prepare_export_values``, so JSON exports automatically carry
the 6-line shape regardless of whether the address was extracted,
manually entered, or left blank.

### UI behaviour

The Detected Fields panel's Inspection Address widget is a
6-line-tall ``Text`` box, so an empty 6-line value displays the
same way it always did (a 6-line-tall blank widget). The user
sees no change.

### Round-trip

Reading the widget back via ``get_field_value`` continues to
strip trailing whitespace (so "really empty" still reads as
``""``). The next export pass re-normalises to the 6-line shape.


## V61 — Dates always exported as DD/MM/YYYY

JSON exports now canonicalise the three date fields
(``Incident Date``, ``Instruction Date``, ``Inspection Date``) to
``DD/MM/YYYY``, regardless of the format the source document used.

### Why

Exports were leaking through the document's original date format —
``"27th April 2026"``, ``"21 Apr 2026"``, etc — when the downstream
management software's import schema expects all dates in
``DD/MM/YYYY``. Manual fixing per record was tedious.

### Behaviour

The Detected Fields panel **preserves** whatever shape the document
used (so the user can see what was actually written and verify
extraction is correct). Only the JSON export pipeline canonicalises.
A user typing ``27th April 2026`` into a date field sees that exact
string in the UI and gets ``27/04/2026`` in the exported JSON.

### Recognised input formats

In priority order:

- ``DD/MM/YYYY`` — already canonical, passes through unchanged
- ``DD/MM/YY`` — 2-digit year (Python expands per ``strptime`` defaults)
- ``DD-MM-YYYY`` and ``DD-MM-YY`` — hyphen separator
- ``DD MMMM YYYY`` — ``27 April 2026``
- ``DD MMM YYYY`` — ``21 Apr 2026``
- ``MMMM DD YYYY`` and ``MMM DD YYYY`` — month-first
- ``YYYY-MM-DD`` — ISO

Ordinal suffixes (``1st``, ``2nd``, ``3rd``, ``27th``) are stripped
before parsing, so ``27th April 2026`` is recognised and converted.
Whitespace, commas, and case in the ordinal suffix are tolerated.

### Unparseable values

Values that don't match any recognised format are left **unchanged**
in the JSON. The user spots the issue at management-software import
time and fixes the source document or the mapping rule. This avoids
silently losing data for an exotic format that we don't support.

Examples of values left as-is:
- ``"not a date"``
- ``"2026"`` (year alone, ambiguous)
- ``"27/04"`` (no year)
- ``"31/02/2026"`` (Feb 31, invalid)
- ``"Date: 23/04/2026"`` (extra prefix — could be solved by
  refining the mapping rule rather than the normaliser)


## V62 — OneDrive-aware Desktop and Documents resolution

Fixed the issue where users with OneDrive installed couldn't find
their exported JSON or images. Files were landing on the legacy
``C:\Users\<user>\Desktop`` folder, which on a OneDrive-redirected
machine is empty and invisible — the user's *real* Desktop is
``C:\Users\<user>\OneDrive\Desktop``.

### Cause

``get_desktop_dir()`` and ``get_documents_dir()`` were resolving
paths via ``Path.home() / "Desktop"`` and ``Path.home() / "Documents"``.
On Windows, ``Path.home()`` returns the legacy user-profile folder,
not the OneDrive-redirected one. Files written there go to the
hidden legacy folder, not where the user expects.

### Fix

On Windows, both functions now ask the Shell for the *real* current
folder via ``SHGetKnownFolderPath`` (using ``KNOWNFOLDERID`` GUIDs
``FOLDERID_Desktop`` and ``FOLDERID_Documents``). The Shell answers
with whatever path Windows itself considers the current Desktop /
Documents folder — taking OneDrive redirection (and any other folder
redirection) into account.

The implementation uses ``ctypes`` rather than pywin32 to avoid a
hard dependency on a specific Windows-only module just for this. On
non-Windows platforms or if the Shell call fails, both functions
fall back to the existing home-relative path.

### Migration

Users who'd been running V60/V61 already have a customised
``providers.json`` (and possibly ``app_settings.json``) sitting in
the legacy ``C:\Users\<user>\Documents\CE Document Mapper\`` location.
After the V62 upgrade, those files would silently be invisible to
the app — which would create a fresh ``providers.json`` from the
bundled seed in the new ``OneDrive\Documents\CE Document Mapper\``
location. Custom presets would appear to vanish.

V62 includes a one-time migration: on first launch, if the legacy
location has files and the new location is empty, the legacy files
are copied into the new location. The legacy files are left in
place (a manual delete by the user later). If the new location
already has its own ``providers.json``, the legacy one is *not*
copied — the new wins.

The migration is best-effort: if the copy fails for any reason
(permissions, locked files), the user gets the bundled seed
instead, no error.

### Behaviour summary

- OneDrive-redirected user, fresh install: Desktop/Documents
  resolve to OneDrive-redirected paths, files land where the user
  expects, providers.json seeded from bundle.
- OneDrive-redirected user upgrading from V60/V61: legacy
  ``providers.json`` migrated into the new location, all custom
  presets preserved.
- Non-OneDrive user, fresh install: identical behaviour to before
  (Shell still returns the standard path).
- Non-OneDrive user upgrading: legacy and new paths happen to be
  the same; migration is a no-op.
- Non-Windows: helper returns ``None``, original home-relative
  fallback used (no behavioural change).


## V63 — Email Date mapping method

Added a seventh mapping method, **Email Date**, designed for the
``Date:`` header line in ``.msg`` email files.

### How it works

User specifies a label (e.g. ``Date:``). The extractor finds the
first line containing that label and scans the portion of the line
*after* the label for the first ``YYYY-MM-DD`` pattern. On finding
one, it returns the date formatted as ``DD/MM/YYYY``.

Example: ``Date: 2026-05-05 16:57:31+01:00`` → ``05/05/2026``.

### Why it's needed

``.msg`` files have line numbering inconsistencies between
extractions, so Fixed Position is unreliable. The body content of
``.msg`` files is also unstructured prose, so other label-based
methods can grab unhelpful surrounding text. Email Date is purpose-
built: it knows the value it's looking for is an ISO date and
returns just that part, properly formatted.

### Same-line only

The method only looks at the line containing the label — it does
NOT fall through to the next line. This is deliberate: ``Date:``
labels also appear in instruction-letter bodies on their own line,
where the next-line content is the address book or some other
body content. We don't want to accidentally match document body
content as if it were an email header.

### Edge cases

- Invalid dates (e.g. month 13, day 31 in February) are rejected
  via ``strptime`` validation; the method returns empty.
- Multiple label occurrences: the first one with a valid date wins.
  If the first occurrence has no date, the method continues
  scanning later occurrences.
- First date on the label line wins if there are multiple.
- Label not found at all → empty.

### Storage

Saved as ``"<label>"`` in ``providers.json`` (no ``||`` delimiter
needed since there's only one config field). Reuses the Single
Label UI shape — one text entry, no second field.

### When to use

Primarily for ``Instruction Date`` when the source is a ``.msg``
file. Although the method is universally available in every
field's dropdown for consistency, ``.msg`` instruction emails are
the realistic use case.


## V64 — Spaced ordinal suffixes in date normalisation

Some real-world documents emit dates with whitespace between the day
and the ordinal suffix — e.g. ``9 th April 2026`` rather than the
more usual ``9th April 2026``. V61's normaliser only stripped the
suffix when it was directly attached to the digits, so these dates
fell through to the "unparseable" path and shipped to the JSON
exactly as written.

### Fix

The ordinal-stripping regex now allows zero or more whitespace
characters between the digits and the suffix. ``9 th April 2026``,
``1 st January 2026``, ``21 ST April 2026``, etc. all become valid.

### Why this is safe

The ``\b`` word boundary after the suffix prevents accidental
matches against words that happen to start with the same letters:

- ``5 thousand`` → no match (after ``th`` is the word-character
  ``o``, the boundary fails).
- ``9 the next day`` → no match (same reason).
- ``2 names`` → no match (``n`` is followed by ``a``).

The regex is also only applied inside the date normaliser, which
runs only on the three ``*_date`` fields during JSON export — so
even if a false positive somehow slipped through the boundary
check, it could only affect those fields, never any other content.

### Verified behaviour

- ``9 th April 2026`` → ``09/04/2026``
- ``1 st January 2026`` → ``01/01/2026``
- ``21 ST April 2026`` (uppercase suffix) → ``21/04/2026``
- ``27 th April 2026`` (multi-digit day) → ``27/04/2026``
- ``9   th April 2026`` (multiple spaces) → ``09/04/2026``
- ``9\tth April 2026`` (tab) → ``09/04/2026``

All V61 formats — joined ordinal (``27th``), short and long month
names, ISO, hyphen-separated, etc. — continue to work unchanged.
