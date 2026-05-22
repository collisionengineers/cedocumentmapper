# INTRO

This is a transcript of the initial developers chat with Claude, where he worked on many iterations of the project. There may be some relevant or useful insights into the thinking, issues, and requirements here.

The transcript is just a copypaste and so it is not normalized with USER: and CLAUDE: responses. You will need to infer from context and possibly normalize the document. 

# TRANSCRIPT

# INITIAL ATTACHMENT PROMPT

I want you to continue an existing project called “CE Document Mapper”.

Context:
This is a lightweight Windows desktop app (Python/Tkinter, packaged with PyInstaller) used to ingest instruction documents and map data from them into structured fields. The app is already quite far along and should be treated as an existing product, not a greenfield rewrite.

Primary goal:
Continue improving the importer / mapper while preserving the current UX and behavior unless I explicitly ask for changes.

Current project state:
- The latest working build is v40.
- It is a Python desktop app with drag-and-drop import.
- It stores settings and providers.json in Documents\CE Document Mapper.
- It has minimal and expanded views.
- It is intended to be compiled into a portable .exe with PyInstaller.
- It is being used to map insurance / vehicle-related documents.

Important UX constraints:
- Keep the UI clean and simple.
- Do not add tabs or major new UI concepts unless absolutely necessary.
- Do not change existing working behavior unless required.
- Success popups were intentionally removed; only show alerts for errors.
- Minimal view is for day-to-day use.
- Expanded view is for setup/configuration.

Current supported concepts:
1. Provider presets
- Saved in providers.json
- Each provider preset has mapping rules
- There is also an “Engineer Report” tickbox saved per preset

2. Mapping methods
Current simplified mapping methods are:
- Single Label
- Two Labels
- Fixed Position
- Manual Input

3. Fields currently in the app
Please preserve this field order:
1. Work Provider
2. VRM
3. Vehicle Model
4. Claimant Name
5. Reference
6. Incident Date
7. Instruction Date
8. Inspection Date
9. Inspection Address
10. Accident Circumstances
11. VAT Status
12. Mileage
13. Mileage Unit

Mileage and Mileage Unit must remain the last two fields, with Mileage Unit last.
JSON export must follow the same order.

4. Export options
Current export options are:
- Export JSON
- Export Images

Notes:
- “Export DOC” was removed.
- “Export JSON String” was renamed to “Export JSON”.
- Success alerts after export were removed intentionally.
- Export filenames use:
  - [Work Provider]_[VRM] if Work Provider is populated
  - [VRM] only if Work Provider is blank

5. Batch mode
- Single file import = normal UI
- Multi-file import = batch mode
- In batch mode:
  - left/right arrow buttons appear below the export options
  - export buttons are prefixed with “Batch ”
- UI must revert to normal single-file mode if a single file is imported afterward

6. Engineer Report logic
This is a major feature already added and must be preserved:
- Some provider presets are marked as “Engineer Report”
- An Engineer Report is a secondary/companion document that overwrites only mapped fields from the current data
- Blank/unmapped fields in the Engineer Report preset must not overwrite existing values

Rules:
- one instruction + one engineer report = allowed
- more than one instruction + one engineer report = error:
  “You cannot process multiple documents with an engineer's report”
- one instruction + more than one engineer report = error:
  “You cannot process more than one engineer's report at once”
- more than one document without engineer report = standard batch mode
- engineer report without any other document = error:
  “You must process an instruction before an engineer's report”

7. Import support
Currently intended imports include:
- PDF
- DOCX
- DOC
- EML
- MSG (best effort, Outlook-based on Windows)

Current known importer improvement requests:
These are the next main tasks I want worked on.

Task A: Improve .DOC ingestion
I have attached a .DOC file called:
- “Inspection Request - Audit Report.DOC”

Problem:
- The header and footer content of this .DOC file are currently not being included in the app’s Source Text Preview / importer output.
- I want the importer to include header/footer text from legacy .DOC files if at all possible.

Task B: Improve PDF table rendering / extraction
I have attached a PDF called:
- “EVA Report.pdf”

Problem:
- Some table text is rendering as strings like:
  /uni004D/uni0072/uni0020/uni004A/uni0075
- Important table content is being missed / mangled
- I want the importer improved so tables in PDFs are rendered/extracted more usefully where possible

Please inspect those uploaded files and focus specifically on:
- getting header/footer content from the .DOC into the extracted text
- improving table text extraction for the PDF
- preserving as much usable text as possible for mapping
- not breaking the existing importer behavior for other file types

Important normalization logic already in place:
1. VRM normalization
- This happens at mapping time, not in the Source Text Preview
- Preview must remain raw
- Detected VRM field is normalized (spaces removed, uppercased)
- Exports use normalized VRM
- Preview highlighting for VRM must still work even if the source contains spaces

2. Inspection Address normalization
- Happens at mapping time, not in the Source Text Preview
- Detected Inspection Address is forced into exactly 6 lines:
  Address Name
  Address Address
  Address Town
  Address City
  Address County
  Address Postcode
- Commas are converted to line breaks first
- If more than 6 lines:
  - keep lines 1–4
  - line 5 absorbs overflow
  - line 6 stays the final original line (postcode)
- If 2–6 lines:
  - final original line is moved to line 6
  - earlier lines stay above
  - blanks fill the gap
- If single-line address:
  - normally keep it on line 1
  - if “Force Postcode” is ticked in the preset, detect a UK postcode at the end of the line and move it to line 6, leaving the rest on line 1

UK postcode logic already discussed:
- Force Postcode should detect a postcode at end-of-line using proper UK postcode patterns
- It must work with:
  A9 9AA
  A9A 9AA
  A99 9AA
  AA9 9AA
  AA9A 9AA
  AA99 9AA
- It should also work if the postcode has no space
- It should then normalize to standard spaced form (e.g. B634AB -> B63 4AB)

Other relevant UI state:
- Minimal view height should currently be 720 px
- Expanded view height should currently be 800 px
- Work Provider is not colour coded, but other fields use consistent colour coding across:
  - detected fields
  - source text preview highlights
  - provider mapping labels
- Text boxes themselves should not be colour coded

Current packaging/build notes:
- App icon support was added using ce_document_mapper.ico
- Build command is along the lines of:
  pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico --add-data "ce_document_mapper.ico;." app.py
- Pillow may be needed for image extraction from PDFs

How I want you to work:
- First, understand the current architecture and preserve working features
- Then focus on the importer improvements for the two attached files
- Be careful not to regress existing mapping, batch mode, engineer report logic, normalization, or UI behavior
- If you propose code changes, keep them targeted and incremental
- If you create a new version, clearly state what changed

Attached files to inspect:
- Inspection Request - Audit Report.DOC
- EVA Report.pdf

Please start by:
1. Summarising your understanding of the current app
2. Explaining the most likely technical cause of:
   - .DOC header/footer text being missed
   - PDF table text showing as /uniXXXX sequences
3. Recommending the cleanest targeted improvements
4. Then implementing the next build


# FULL CHAT

I want you to take over development work on a project.Don't make any changes yet, just look at everything and attempt to understand it all.I pasted a handover prompt from the last model that worked on this.6 MayClaude responded: I'll take a look at everything and build up an understanding of the project before making any changes.I'll take a look at everything and build up an understanding of the project before making any changes.Ran 2 commandsRan 2 commandsA 3170-line single file. Let me work through it in chunks to understand the architecture.Viewed 18 filesViewed 18 filesI now have a comprehensive understanding. Let me also check where the two attached files referenced in the handover prompt actually are — the prompt mentions them but I should verify.Look for the .DOC and PDF files mentioned in the handoverLook for the .DOC and PDF files mentioned in the handoverConfirmed — the two test files referenced in the handover prompt (Inspection Request - Audit Report.DOC and EVA Report.pdf) were not actually uploaded with this handover. Only the project source is here.
Here's my understanding of where things stand. I haven't changed anything yet.
What I've absorbed
Architecture. Single-file Python desktop app, app.py, ~3,170 lines. Two top-level concerns:

ExtractionEngine — handles file ingestion, provider detection, field extraction, image extraction. Pure logic, no UI.
App — Tkinter UI: drag-and-drop, three-panel layout (values left / preview middle / mapping right), batch mode, exports.

Free-floating helpers below the engine (normalise_inspection_address_value, post_process_extracted_value, normalize_vrm_value, build_rjs_docx, the various DOC↔DOCX converters). The README mentions a v41 importer pass and the handover says current build is v40 — I'll trust the handover and treat current state as v40.
Fields and order. The DEFAULT_FIELDS tuple at the top is the canonical order, matches the handover exactly, with Mileage/Mileage Unit last. JSON export iterates FIELD_KEYS so the order is preserved.
Mapping methods. Four current methods (single_label, two_labels, fixed_position, manual_input) plus a LEGACY_METHOD_TO_DISPLAY_CODE map that quietly migrates older provider files. The legacy fallback methods (vrm_fallback, address_fallback, etc.) are still in extract_by_rule, but extract_fields short-circuits when config_value is blank — so presets have full control and fallbacks only fire if a legacy preset still references them by method name. Worth knowing before any importer changes.
Engineer Report. Implemented exactly as the handover describes — is_engineer_report_provider flag on the provider, then the four error-message branches in load_files (lines 2614-2674) covering one-instruction-many-engineer, no-instruction, multiple-engineer, etc. combine_instruction_and_engineer_report does the merge: only non-blank fields from the engineer session overwrite. Don't touch this.
Normalization. VRM stripped/uppercased only at mapping time and at export prep. Inspection Address forced to 6 lines via normalise_inspection_address_value — single-line + force_postcode triggers the end-of-line UK postcode detection (the postcode_end_re pattern at line 1436 covers all six UK formats). Preview text stays raw.
Importers — current state of the two problem areas.
For DOC, the order in extract_text (lines 774-791) is: Microsoft Word → LibreOffice → antiword. Word automation in _extract_doc_text_via_word does loop sections and pull headers/footers (lines 528-549), so on a Word-equipped machine that path should work. LibreOffice path converts to DOCX then calls _extract_docx_text, which does call _extract_docx_header_footer_lines. The antiword path strips headers/footers entirely. So the likely cause of missing header/footer text is the path actually being taken at runtime — antiword on a non-Word machine, or a DOC with headers stored in a section that section-iteration misses (e.g. linkToPrevious not resolving, or first-page-only headers when Section.PageSetup.DifferentFirstPageHeaderFooter is true and only Headers(1) was checked — but the code does loop all three header types 1/2/3, so that should be covered).
For PDF, extract_text tries PyMuPDF first, falls back to pypdf. The /uniXXXX mangling is only the pypdf branch — _decode_pypdf_unicode_sequences already handles it. PyMuPDF doesn't emit /uniXXXX. So if a user is seeing those sequences, either (a) PyMuPDF isn't installed at runtime, or (b) PyMuPDF returned empty text and the code fell through to pypdf. The if text.strip(): return guard at line 745 means PyMuPDF returning whitespace-only text triggers fallback. For tables specifically, page.get_text("text", sort=True) is the simplest extraction mode — it can scramble multi-column table cells. Better options exist: get_text("blocks"), get_text("dict"), or PyMuPDF's find_tables() API (PyMuPDF ≥ 1.23).
Likely targeted fixes (not implementing yet, just naming them):
For DOC: keep current order, but make the antiword path a last resort with a status note when it's used so the user knows headers/footers may be incomplete. Also worth checking: the Word automation path returns content via doc.Content.Text, which is the main story only — textboxes/shapes in headers may not be in Section.Headers(...).Range.Text either. Could iterate StoryRanges to be thorough.
For PDF: switch the PyMuPDF call to a smarter extraction. Concrete plan would be: try find_tables() first to render tables as pipe-separated rows, then get_text("blocks") for the rest, falling back to current get_text("text", sort=True) only if those fail. Also keep the pypdf /uniXXXX decoder as a safety net but strengthen it — currently it only handles 4-hex-digit /uniXXXX; some encoders emit /glyphNNN or /uniXXXX_uniXXXX ligature sequences.
Things I'll be careful about.
The two test files (Inspection Request - Audit Report.DOC and EVA Report.pdf) referenced in the handover aren't actually here — only the source files were uploaded. That matters because the right fix for the table problem depends on what the PDF actually looks like (whether PyMuPDF's find_tables() recognises them, whether the text is encoded weirdly, whether they're vector/image tables). Without the PDF I'd be guessing.
Same for the DOC — without it I can't tell whether the headers are stored in a way Word automation already captures (in which case the user's machine likely fell back to antiword) or whether there's a structural issue requiring code changes.
Want to upload those two files? Once I can poke at them I can give you a concrete diagnosis and a targeted patch rather than speculative work.You said: So the last version was V41, but the chat with the model that was working on the project got corrupted.pdfInspection Request - Audit Report.DOCDOCSo the last version was V41, but the chat with the model that was working on the project got corrupted. The version I uploaded is V41. The  changes between V40 to V41 were targeted importer improvements for .DOC files to try to capture headers and footers, as well as PDF importer improvements to try to improve table rendering.6 MayClaude responded: Got it — so V41 already contains the importer improvements, and the question is whether they actually work on these two files.Got it — so V41 already contains the importer improvements, and the question is whether they actually work on these two files. Let me verify by running the importers against the real files.Ran 3 commandsRan 3 commandsNow let me test the current PDF importer against the EVA report and see what comes out:Ran 3 commands, created a fileRan 3 commands, created a fileInteresting — the PDF has actually got meaningful content. PyMuPDF is working, no /uniXXXX sequences in this output. But look closely at how it handled the table:

Reg No: JR07CVR — present ✓
Vehicle: BMW X5 XDRIVE40D M SPORT AUTO — present ✓
Vin No: WBAKS620300P88661 — present ✓
Damage: Moderate Collision/Impact RH Rear — present, on the same line as Incident: 31/03/2026 and the tyre cells

But the table cell layout is somewhat scrambled — the multi-column header rows got interleaved by the sort=True reading order, e.g. Audio: Not Seen ended up on the same line as Manf'd: 2016 and Spare Tyre. That's the "tables not rendering well" issue. Also the last 2000 chars show the text was cut — it ends mid-sentence in wheel/t. Let me check why:Check actual end of extracted text vs the stdout flush issueCheck actual end of extracted text vs the stdout flush issueOK, the full text is extracted (6953 chars) — the truncation in my first test was just stdout chunking. The footer/header content (Our Ref, Continued . . . . ., Page 2/3, Claim No: EHR99287) is being captured. So the EVA PDF is actually working pretty well overall.
The remaining issue is purely the table cell scrambling in the vehicle details box. Now let me see what find_tables() does with it:Created a file, ran a commandCreated a file, ran a commandAha, this is illuminating. PyMuPDF's find_tables() detects the vehicle-details box as two overlapping tables (table 1 has the "outer" frame and table 2 has the "inner" cells). Neither one alone gives a clean reconstruction — the cells are split across the two tables and key columns like "Reg No:" labels appear in one table while values appear in another.
So the simple-minded approach of "just use find_tables()" wouldn't work here — it would actually produce worse output than what get_text("text", sort=True) already gives us.
Let me think about this more carefully. What we need for the mapping rules to work isn't pretty tables — it's that key/value pairs like Reg No: JR07CVR end up adjacent in the text. Let me look at get_text("blocks") since blocks preserve spatial grouping:Created a file, ran a commandCreated a file, ran a commandNow this is much cleaner. Each label/value pair is its own block, e.g. "Reg No: JR07CVR", "Vin No: WBAKS620300P88661", "Audio: Not Seen", "Damage: Moderate Collision/Impact RH Rear". If we read the page in block order (sorted by y-then-x), we get clean key-value pairs that the existing label-based mapping rules can readily consume.
This is the right approach. Now let me also verify the DOC. The existing engine on this Linux box won't have Word — let me see how the fallbacks behave:Check what DOC readers are availableCheck what DOC readers are availableLibreOffice is available — that's the same path most non-Word users would hit. Let me run the DOC importer:Created a file, ran a commandCreated a file, ran a commandExcellent — the LibreOffice path works and the headers and footers ARE being extracted:

Header: "_OUR REF: _JMW/CB", "Information classification: Client Confidential" (top)
Footer: "850 Ibis Court, Centre Park | Warrington | WA1 1RL" and the full registered-office boilerplate (bottom)

So the V41 DOC fix actually works on this Linux LibreOffice path. Good. The path goes: _extract_doc_text_via_word (fails — no Word) → _extract_doc_text_via_soffice (succeeds, converts to DOCX) → _extract_docx_text (which calls _extract_docx_header_footer_lines). That whole chain works.
Now let me look at one specific DOC concern. The handover says headers/footers were missing. The current LibreOffice path does work. But let me double-check what happens on Word's path — i.e. does the _extract_doc_text_via_word function correctly capture them? I can't run Word here, but I can read the code carefully:Reread the Word DOC path closelyReread the Word DOC path closelyThat code looks correct. It loops every section, every header type (primary/first-page/even), and the same for footers. One subtle thing: Section.Headers(1).Range.Text — Word's StoryRanges for headers/footers include text in textboxes inside them only if you also walk Range.ShapeRange or use wdHeaderStoryRange. If the original DOC put the firm's address in a text frame inside the header (which is common in legacy templates), Range.Text won't pick it up. But that's an edge case; for now the DOC chain is working correctly on the test file via LibreOffice.
Now let me actually test whether the field extraction is working well on our PDF — that's the real measure. Let me try mapping the EVA report against an "EVA"-like provider:Created a file, ran a commandCreated a file, ran a commandThis is the real problem. PyMuPDF's get_text("text", sort=True) returns rows that horizontally interleave columns, producing lines like:
Reg No: JR07CVR          Registered: May 2016       Type: Estate                          Trans: Automatic
The current extract_after_label regex grabs everything to end-of-line, so:

Reg No: → matches the whole line, then normalize_vrm_value strips spaces → JR07CVRREGISTERED:MAY2016TYPE:ESTATETRANS:AUTOMATIC
Vehicle: → grabs the whole line BMW X5 XDRIVE40D M SPORT AUTO Colour: GREY Speedo: Miles
Incident: → grabs 31/03/2026 L/H/F Tyre Not Kno L/H/R Tyre Not Kno
Our Ref: → grabs 00033481/JAKEREnfield (because address column "Enfield" is at the same y-coordinate)

The fix is to use block-mode extraction for PDFs. Each block is a spatial unit, so Reg No: JR07CVR would be its own block, and extract_after_label("Reg No") would correctly return just JR07CVR.
Let me prototype the block-mode extraction and verify the mapping works correctly with it:Created a file, ran a commandCreated a file, ran a commandNow each cell is its own block. Reg No: JR07CVR is on its own line, Incident: 31/03/2026 would be on its own line, Our Ref: 00033481/JAKER on its own line. Let me run the mapping test against this:Created a file, ran a commandCreated a file, ran a commandLook at that. Every field clean and correct:

vrm: 'JR07CVR' (was JR07CVRREGISTERED:MAY2016TYPE:ESTATETRANS:AUTOMATIC)
vehicle_model: 'BMW X5 XDRIVE40D M SPORT AUTO' (was contaminated with Colour: GREY Speedo: Miles)
reference: '00033481/JAKER' (was 00033481/JAKEREnfield)
incident_date: '31/03/2026' (was 31/03/2026 L/H/F Tyre Not Kno L/H/R Tyre Not Kno)
mileage_unit: 'Miles' ✓

This is the right fix.
Let me also confirm no regression on the DOC, since our DOC path doesn't touch PDF code anyway:Confirm DOC still worksConfirm DOC still worksGood — DOC still working. Let me also verify with the existing RJS provider (the only saved one) that PDF block mode doesn't break anything weird. Let me also test whether the line-numbering in the preview still makes sense when blocks introduce more blank lines (since the existing Fixed Position mapping method depends on stable line numbers):Compare line counts old vs newCompare line counts old vs newOK so block mode is fewer characters (no internal padding spaces) but more lines (because each block is separated by a blank line). The Fixed Position mapping method uses absolute line numbers, so anyone who has saved a Fixed Position rule for an existing PDF provider would have their numbering shifted. That's a regression risk worth flagging.
Looking at the existing config file (providers.json) and the default config — only RJS is in there, and RJS uses no fixed_position rules at all. So no current users are affected, but it's worth noting in the changelog. Anyone who later builds a Fixed Position mapping for a PDF provider would do so against the new line numbers, so it'd be self-consistent.
Now let me write up everything clearly so you can decide where to go from here.
Summary of what I tested
On the EVA PDF:

Current V41 PyMuPDF path (get_text("text", sort=True)) successfully decodes everything — there are no /uniXXXX sequences in the output and the headers/footers are present. The /uniXXXX problem was only ever the pypdf fallback path, and the V41 changes correctly prefer PyMuPDF first.
However, the visible textual output has the table cells horizontally interleaved, so when label-based mapping rules try to extract a value, they grab adjacent columns too. This is the root cause of the "table content mangled" complaint, and it directly breaks field extraction in serious ways (VRM, vehicle model, reference, incident date all corrupted).
I tried find_tables() — on this PDF it detects two overlapping tables and produces worse output than get_text("text"). Not the right fix.
get_text("blocks", sort=True) is the right fix. Each label/value pair becomes its own block (Reg No: JR07CVR is isolated). I monkey-patched the engine to use blocks and re-ran the mapping — every field came out clean. This is a small, low-risk change: about 15 lines in _extract_pdf_text_via_pymupdf.

On the audit-report DOC:

Headers and footers are being captured correctly through the LibreOffice path (_OUR REF, Information classification, the full registered-office footer block all appear). The V41 _extract_doc_text_via_word path looks structurally correct too — I can't run Word on Linux to verify, but the section-loop and header-type-loop logic is right. If users on a Word machine were still seeing missed headers in V40, it was probably because V40 never did this loop at all.
I don't see anything that needs fixing on the DOC side based on this test file. Worth confirming the user is actually still seeing missing headers/footers on their end before I touch this code.

Recommendation
Make a single targeted change for V42: switch _extract_pdf_text_via_pymupdf from get_text("text", sort=True) to get_text("blocks", sort=True) with a small per-block normalisation (each block becomes a group of lines, blocks separated by blank lines). That's it. Don't touch the DOC path; it's already working on this test file, and any further improvement there would be speculative.
Two small notes to flag in the changelog:

PDF Fixed Position line numbers will shift relative to V41 (due to blank-line-between-blocks). Existing RJS provider doesn't use Fixed Position so no real-world impact, but worth mentioning.
The _decode_pypdf_unicode_sequences helper stays as a safety net for the pypdf fallback path.

Want me to go ahead and implement this as V42?You said: I fully agree.I fully agree.One thing to note before continuing - the previous model would consistently output the "providers.json" file along with the application. I'm not fully sure if this is required when I compile the app into an .exe? Regardless, the providers.json present doesn't really represent real world user mapping data. Another thing to note is that since the app is still in development we really don't need to worry about messing up existing mapping, all we need to keep in tact is a way to convert existing preferences when the user opens a new version of the application (I think you mentioned that this was already in place).6 MayClaude responded: Quick answers on both before I implement V42:Quick answers on both before I implement V42:
On providers.json shipping alongside the app. It's not required for the .exe build at all. The --add-data flag in your PyInstaller build command only includes ce_document_mapper.ico, not providers.json. The app creates its own copy at Documents\CE Document Mapper\providers.json on first run via load_or_create_config (line 206-211) using the embedded DEFAULT_CONFIG constant. So the previous model was probably handing you providers.json just so you could see/edit the working state, not because the app needs it. I'll do the same — show you the file so you can inspect it, but it's not part of the executable's runtime requirements.
On legacy mapping conversion. Yes, that's already in place — normalize_provider_config (line 217) plus LEGACY_METHOD_TO_DISPLAY_CODE (line 84) handle the migration when an older providers.json is loaded. Old method names get translated to the four current ones, legacy field_labels get folded into field_rules, removed flags are dropped. So if you don't need to preserve the current providers.json content (which makes sense — it's just the default RJS preset), I have full flexibility here. The V42 change to PDF extraction doesn't affect the providers schema anyway, so this conversation layer isn't even exercised by this change.
Implementing V42 now.Viewed a file, ran a commandViewed a file, ran a commandNow the targeted edit. I'll replace the PyMuPDF extraction with a block-mode version. Each text block becomes its own group of lines; blocks are separated by blank lines so each spatially-isolated label/value pair sits on its own. Image blocks (block_type == 1) are skipped. The /uniXXXX decode in the pypdf fallback stays untouched.Switch PyMuPDF extraction to block mode for cleaner table cell isolationSwitch PyMuPDF extraction to block mode for cleaner table cell isolationNow run the same end-to-end mapping test against the patched file (no monkey-patching this time — the code itself is updated):Created a file, ran a commandCreated a file, ran a commandBoth pass. Quick syntax sanity-check on the patched file:Ran a command, viewed a fileRan a command, viewed a fileNow I'll also append a V42 note to the README, then move both updated files to the outputs directory:Ran a command, read a fileRan a command, read a fileV42 is built. Summary of what changed:
Single targeted edit in _extract_pdf_text_via_pymupdf (around line 716). Switched from get_text("text", sort=True) to get_text("blocks", sort=True). Each PyMuPDF text block becomes its own group of lines, blocks are separated by blank lines, image blocks (block_type == 1) are skipped, and there's a per-page fallback to the old text mode if a page produces no blocks (so nothing breaks on unusual PDFs).
Verified end-to-end on the EVA report — all seven mapped fields come out correct (vrm: 'JR07CVR', vehicle_model: 'BMW X5 XDRIVE40D M SPORT AUTO', incident_date: '31/03/2026', etc., where V41 produced contaminated values). DOC test still passes — header (_OUR REF: _JMW/CB) and footer (Performance Car Hire Limited) both present in the audit report extraction, unchanged from V41.
Nothing else was touched. The pypdf fallback with its /uniXXXX decoder is still there as a safety net. DOCX/DOC/EML/MSG paths are untouched. Provider schema, mapping methods, engineer-report logic, normalisation helpers, UI, batch mode, exports — all unchanged.
One thing worth flagging in the changelog and to users: Fixed Position mapping rules use absolute preview line numbers. Block mode introduces more blank lines than the old text mode (~194 lines vs ~130 on the EVA report), so any existing Fixed Position rule targeting a PDF would need its line numbers updated. The default RJS provider doesn't use Fixed Position, so out of the box no one is affected.AppPY DownloadReadmeDocument · MD DownloadDownload allYou said: Are you able to always include "requirement.1778062504650_requirements.txt7 linestxtAre you able to always include "requirement.txt" when providing a new version to download? I'm assuming the attached version is still good enough?6 MayClaude responded: Yes, I'll always include requirements.Yes, I'll always include requirements.txt going forward.
I checked the attached requirements.txt against everything V42 actually imports — it's still good as-is, no changes needed. The block-mode switch uses page.get_text("blocks", sort=True), which has been part of PyMuPDF since well before 1.24.0, so the existing PyMuPDF>=1.24.0 pin already covers it.
Here are all three files for V42:Ran a command, read a fileRan a command, read a fileAll three V42 files ready. I'll keep requirements.txt in every future drop, and I'll only modify it if a change actually pulls in a new dependency or needs a version bump — otherwise I'll just pass through the existing one so it stays in sync with whatever you're shipping.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: That's way better.That's way better.
We now need to add in two special rules.
Firstly: the "Mileage" field needs to accept any numbers until the first non-numeric character (exclude commas just in case they are mid-number).
For example, if the source text is "Speedo: 28,487 Miles", we want users to be able to use the Single Label Method and have "Speedo:" as the regex. The application should scan "28487" and stop at the "M" in miles (trimming white space before it). The detected data for "Mileage" will read "28487".
The reason we need this rather than requesting users use the Two Labels Method is because the mileage unit can vary, therefore there is no fixed/predictable second label.
Second special rule: we need a special rule for the "Mileage Unit" field which operates on the same principle as the "VAT Status" field. With the "VAT Status" field, users simply identify the "Yes" scenario and, if met, the detected data for the field simply says "Yes" - if the "Yes" scenario which the user identified is not met, the detected data for the field simply says "No" - and thirdly, if the user doesn't specify anything in the provider settings for the "VAT Status" field, it is left blank in the detected data. We need to follow this approach with "Mileage Unit". Users should be asked to identify the "Miles" scenario and, if this isn't met, the detected data for the field becomes "Km" instead. If the user leaves the "Mileage Unit" field blank in the provider settings, then the detected data will always be blank. In summary, the only accepted states for the detected data for the "Mileage Unit" field are "Miles", "Km", and empty, and the user is asked to identify the "Miles" scenario.Show more6 MayClaude responded: Two clarifying questions before I write any code, because the spec has a few choices buried in it:Two clarifying questions before I write any code, because the spec has a few choices buried in it:
Question 1 — Mileage parsing scope. "Take numbers until the first non-numeric character, ignoring commas" — does that apply to the value after the user's label match, or to the whole line/block? Two interpretations:
(a) Find Speedo:, then in whatever text the existing single_label extraction returns, walk left-to-right and grab the first run of digits-and-commas, stop at the first non-numeric. So Speedo:    28,487 Miles\nNext line stuff → 28487. Leading non-numerics like spaces are skipped over to find the start of the number.
(b) The number must be the immediate first content after the label, leading whitespace only. If anything else comes between the label and the digits, no match.
I think you mean (a) based on your example, but want to confirm — particularly whether the parser should skip leading non-digit chars to find the number, or whether the number has to be right at the start of the post-label text.
Question 2 — Mileage Unit, "Miles scenario" semantics. For VAT Status the "Yes scenario" matching is a bit forgiving — it tries the rule's extracted value, then falls back to keyword presence anywhere in the text, then tries a couple of regexes. For Mileage Unit, do you want me to:
(a) Mirror VAT Status exactly — same multi-tier matching (regex extraction → keyword presence → label tokens scanned anywhere in document), just with "Miles" as the positive outcome and "Km" as the negative; or
(b) Keep it simpler — only "does the user's configured pattern match anywhere in the document text"? Yes → Miles, no → Km, blank config → blank.
VAT Status's tiered logic was probably built up over time to handle messy real documents. Mirroring it gets you consistency; the simpler version is easier to reason about. I lean toward (a) — mirror VAT Status for consistency, but it's your call.
Also one small thing worth flagging: there's a third value the field could legitimately hold today — Mileage Unit is currently a free-text single-label field, so existing presets might have a config that produces neither "Miles" nor "Km". The new rule will change the output for those cases. Since the app is still in development that's fine, but worth being explicit: any saved provider with a non-blank Mileage Unit config will, after this change, only ever produce "Miles" or "Km" — never the raw extracted value. Confirming that's what you want.You said: Question 1 - AQuestion 1 - AQuestion 2 - To be honest I didn't know that there was a fallback in place for the VAT status. Fortunately, this hasn't effected anything so far since the mapping the user's have created is precise enough to either match or not. We don't really want any fallbacks, we want full user control6 MayClaude responded: Got it — precise matching, no fallbacks.Got it — precise matching, no fallbacks. So:

Mileage: scan post-label text; skip leading non-digits/commas; collect digits and strip commas; stop at first other character. "   28,487 Miles" → 28487.
Mileage Unit: only match the user's configured tokens exactly as configured against the document. Match → Miles, no match → Km, blank config → blank. No regexes, no synonym lists, no fallback methods.

You also raised the VAT Status fallbacks. Want me to strip those too while I'm in there — for consistency and "full user control" — or leave VAT Status as-is for now? My recommendation: strip them. The current resolve_vat_status (line 1268) tries the rule, then checks every config token against the lowercased document, then re-scans with the start label of a two_labels config. That's a lot of fallback behaviour that you didn't know was there. Tightening it to just "does the configured rule produce a hit" matches your stated principle and means there's only one matching approach to reason about across both Yes/No fields.
Quick design question for both Mileage Unit and (if you agree) VAT Status: when the method is single_label or two_labels, do you want the match to mean "the rule produces any extracted value" (so any text after Speedo: flips it to Miles) or "the rule produces a value containing the configured tokens" (so Speedo: config has to be paired with tokens)?
I think the natural reading is the former — same as how a normal Single Label rule works today. User configures Speedo: as the label, document contains Speedo: 28,487 Miles, rule extracts 28,487 Miles, that's a hit → Miles. If the document doesn't have Speedo: at all, the rule extracts nothing, no hit → Km. That keeps the mental model consistent with every other field. Confirm?You said: For "Mileage Unit" users are going to put in however miles is expressed in their document mapping presets.For "Mileage Unit" users are going to put in however miles is expressed in their document mapping presets. So this will likely be "miles" "mi". They will not be putting in anything else at all. The idea is that if miles doesn't exist, it must be km.6 MayClaude responded: Got it — much simpler than I was making it.Got it — much simpler than I was making it. Mileage Unit is essentially a "look for these literal tokens anywhere in the document" check:

Config blank → output blank
Any configured token (e.g. miles, mi) found in the document text (case-insensitive) → output Miles
Otherwise → output Km

The mapping method dropdown for Mileage Unit becomes irrelevant — it's just a list of tokens to check for. I'll either ignore the method choice for this field or hide the dropdown. I think hiding it is cleaner since exposing a dropdown that does nothing is confusing. Same approach as VAT Status currently has (its method dropdown is shown but resolve_vat_status mostly ignores it).
Actually — since you want full user control and precise matching, let me ask: should I do the same for VAT Status while I'm at it? Right now its UI shows a Method dropdown for VAT Status that mostly does nothing. If I tighten VAT Status to "any configured token found in document → Yes, none found → No, blank → blank" that mirrors Mileage Unit perfectly. Both Yes/No fields would work the same way: a comma-separated list of tokens, presence check, no method involvement.
Three things to confirm before I implement:

Mileage: scan post-label text, skip leading non-digit chars, collect digits-and-commas, strip commas, stop at first other character. "   28,487 Miles" → 28487. Returns blank if no digits found.
Mileage Unit: presence check only. Comma-separated tokens (e.g. miles, mi) checked case-insensitively against full document text. Hit → Miles, no hit → Km, blank config → blank. Hide the method dropdown for this field.
VAT Status (optional, for consistency): same simplification. Presence check on configured tokens. Hit → Yes, no hit → No, blank config → blank. Hide method dropdown. Strip the fallback chains in resolve_vat_status.

Confirm 1 and 2, and let me know yes/no on 3.You said: Yes, I suppose the dropdowns can be removed for bothYes, I suppose the dropdowns can be removed for both6 MayClaude responded: Implementing now.Implementing now. Plan:

Mileage: post-process the value extracted by the user's chosen mapping method — skip leading non-digits, collect digits-and-commas, strip commas, stop at first other character. Plug into post_process_extracted_value so it works regardless of method. Empty string if no digits.
Mileage Unit: rewrite as presence-check-only. Comma-separated tokens, case-insensitive search across full document text. Hit → Miles, no hit → Km, blank → blank. Hide the method dropdown for this field.
VAT Status: same model. Replace resolve_vat_status with presence-check-only. Hide method dropdown.
Schema migration: when an old providers.json is loaded, if mileage_unit or vat_status has method: "two_labels" and config like "foo || bar", take just the start label as the token list. Keep them on single_label going forward (since the method is now hidden). Helps anyone who already saved a preset.

Start by confirming where to splice the changes:Viewed 2 files, ran a commandViewed 2 files, ran a commandNow look at post_process_extracted_value and normalize_provider_config:Viewed 2 filesViewed 2 filesNow update_method_ui and where the rules row is built:Update_method_uiUpdate_method_uiNow look at where the rules table builds the Method dropdown so I can hide it for VAT and Mileage Unit:Look at how each row is builtLook at how each row is builtI have enough context. Now let me also check the save flow to make sure VAT/Mileage Unit save the right method:Save_provider_mappingSave_provider_mappingRight — and load_provider_into_editor:Load_provider_into_editorLoad_provider_into_editorGood. I have everything I need. Let me make the changes.
Edit 1: Define a PRESENCE_CHECK_FIELDS set near the top, and a constant set for "fields where the method dropdown is hidden". Add helpers for the new logic.Add a constant for fields that are hidden-method presence-check Yes/No styleAdd a constant for fields that are hidden-method presence-check Yes/No styleNote: I changed vat_status and mileage_unit defaults to single_label — they now always save as single_label (since the dropdown is hidden, the saved method is just for forward consistency). The existing default config has them as manual_input which would still work since extract_fields short-circuits on blank configs and the presence-check function ignores method. But for new presets the default will be single_label.
Edit 2: Replace resolve_vat_status with a generic resolve_presence_check method, and add the mileage number extraction helper:Replace resolve_vat_status with a generic presence-check method, and add a mileage post-processorReplace resolve_vat_status with a generic presence-check method, and add a mileage post-processorEdit 3: Update extract_fields to dispatch presence-check fields generically:Generic presence-check dispatch in extract_fieldsGeneric presence-check dispatch in extract_fieldsEdit 4: Add mileage normalization helper and wire it into post_process_extracted_value:Add normalize_mileage_value helper and wire it into post_processAdd normalize_mileage_value helper and wire it into post_processWait — there's a subtle issue. prepare_export_values in the App class (around line 2766) does post-processing for VRM and inspection_address but not for mileage. Let me check that and also confirm export behaviour:Viewed 2 files, ran a commandViewed 2 files, ran a commandThis is a re-normalization step that runs on whatever the user manually edited the field to before export. The user could edit the Mileage field to type 12,345 miles themselves, and the export should clean it. Let me extend this to mileage too:Also re-normalize mileage at export timeAlso re-normalize mileage at export timeEdit 5: Hide the Method dropdown for VAT Status and Mileage Unit. The cleanest way: in the row-build loop, skip creating the combo for these fields and place a label-style placeholder instead (or just hide the combo). Hiding gives back nice horizontal space. Also need to hide the two_frame / show only the single entry, and update update_method_ui to never switch them.
Looking at the row-build: column 1 is the Method combo, column 2 is the config frame. If I combo.grid_remove() for these fields, column 2 stretches because of the column weights. That should look fine. Let me make the edit:Hide method dropdown for presence-check fields and lock them to single_label UIHide method dropdown for presence-check fields and lock them to single_label UIEdit 6: Make update_method_ui always show the single-entry box for presence-check fields, regardless of the saved method. This is defensive — protects against legacy presets that have e.g. two_labels saved for vat_status:Lock presence-check fields to single-entry UI in update_method_uiLock presence-check fields to single-entry UI in update_method_uiEdit 7: save_provider_mapping and load_provider_into_editor need to handle presence-check fields cleanly. On save: always write single_label and the raw config. On load: collapse any old two_labels config ("foo || bar") back to a single token list.Save_provider_mapping treats presence-check fields as plain single-entry textSave_provider_mapping treats presence-check fields as plain single-entry textNow load_provider_into_editor — add legacy migration for presence-check fields. If an old preset has two_labels config like "foo || bar" for vat_status, we want to load just "foo" (the start label was the positive scenario in old configs).
Actually wait — re-reading the old resolve_vat_status: in two_labels mode with "foo || bar", it was checking all comma-separated tokens of the whole config string against the document, AND if the start label (e.g. "foo") matched anywhere in the document. So legacy behaviour was: "foo" being present → Yes. So I should migrate "foo || bar" → "foo" when loading, taking the start label as the new token.Load_provider_into_editor migrates legacy two_labels configs for presence-check fieldsLoad_provider_into_editor migrates legacy two_labels configs for presence-check fieldsAll edits done. Let me verify the file compiles and run tests:Syntax checkSyntax checkNow let me write a test that exercises both rules thoroughly:Created a file, ran a commandCreated a file, ran a commandThe one "failure" is actually correct behaviour and my test was wrong. The doc has "not VAT registered" and the user's positive token is "VAT registered" — that IS a substring match, so the engine correctly returns Yes. My test expected No, but in fact the user is responsible for picking tokens that uniquely identify the positive scenario. If their document has "not VAT registered" they should instead configure something like "isn't VAT registered" for the negative scenario... wait no, that's backwards.
Actually this is exactly the kind of thing the user should be aware of. With pure substring matching, configuring "VAT registered" will match in both "VAT registered" and "not VAT registered". That's the trade-off of "full user control / no fallback magic". The user can mitigate by picking tokens that are uniquely positive (e.g. configure ": Yes" or ": Vatable" based on what their docs use).
Let me update the test to reflect that this is the intended behaviour, and note the tradeoff:Edited a file, ran a commandEdited a file, ran a commandAll passing. Now run the V42 regression tests too — the EVA PDF and audit DOC paths shouldn't have changed:Run V42 regression testsRun V42 regression testsWait — mileage_unit is now 'Miles' rather than blank! The V42 test EVA provider config had mileage_unit: {"method": "single_label", "config": "Speedo"}. Under the old logic that was a label-extraction rule (returning the post-label text). Under V43 it's now a presence-check: is "Speedo" present in the EVA PDF text? Yes, so → Miles.
That's actually arguably correct behaviour for a presence-check field. It demonstrates that the breaking change applies. For the EVA report, the user would now want to configure mileage_unit as miles (the literal token) — which gives Miles as expected.
Let me also do a smoke test on the EVA PDF using both new fields configured properly:Created a file, ran a commandCreated a file, ran a commandEverything works end-to-end. Now let me also confirm the UI changes don't break by importing and instantiating the App class:Smoke test module importsSmoke test module importsAll clean. Now let me also try to actually instantiate the App class with a virtual display, to catch any UI-construction bug I might have introduced:Ran 2 commands, created a fileRan 2 commands, created a fileEverything works:

App builds without errors with the hidden dropdowns
Legacy two_labels configs for VAT Status and Mileage Unit are correctly migrated to single_label with just the start label
After the user saves, the data on disk is in the new clean form

Now update the README and copy outputs:Ran 2 commands, read a fileRan 2 commands, read a fileV43 is built. Summary:
Mileage — added normalize_mileage_value and wired it into post_process_extracted_value. The user's mapping rule (Single Label, Two Labels, whatever) still runs first; whatever it returns then gets the digit-extraction pass: skip leading non-digits, collect digits, strip commas, stop at first non-digit/non-comma. Also runs at export time via prepare_export_values so manual edits get cleaned up too.
Mileage Unit and VAT Status — both now use a new generic resolve_presence_check method. Comma-separated tokens, case-insensitive substring match anywhere in the document. Hit → positive value (Miles/Yes), no hit → negative value (Km/No), blank config → blank. The old resolve_vat_status with its regex fallbacks and synonym lists is gone.
UI — Method dropdown hidden for both fields. Defensive logic in update_method_ui always shows them as single-entry. save_provider_mapping always writes them as single_label. load_provider_into_editor migrates legacy two_labels configs ("foo || bar") by taking just the start label.
Architecture note — I introduced a PRESENCE_CHECK_FIELDS constant near the top of the file. If you ever need a third Yes/No-style field, you just add it to that dict with its own positive/negative pair and the rest of the machinery picks it up automatically.
Verified end-to-end with Xvfb — App actually builds, legacy provider with old two_labels configs loads correctly, save round-trips to clean form. EVA PDF (V42 regression) and audit DOC (V41 regression) still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Just a small change for now and then I'm going to run extensive tests - regarding the "Engineer Report" import which replaces/overwrites whichever fields are m…Just a small change for now and then I'm going to run extensive tests - regarding the "Engineer Report" import which replaces/overwrites whichever fields are mapped in the provider setup, can we add something to the UI to indicate to the user when something has been replaced? My thought are to highlight the replaced fields in red in the detected data panel. This should be a permanent highlight rather than a quick flash (and when a fresh document is dragged in, the highlight disappears).We need to make sure this highlighting also works if a normal document and engineer report are dragged in at the same time (they should already be processed in the correct order).6 MayClaude responded: Before I implement, let me check a few things about the current Engineer Report flow so I do this right.Before I implement, let me check a few things about the current Engineer Report flow so I do this right.Viewed 2 files, ran a commandViewed 2 files, ran a commandOK, now I have enough context. Quick design plan:
What needs to happen. When an Engineer Report overwrites a field, that field's value entry widget (left panel, "Detected Fields") should get a permanent red highlight until a fresh document is dragged in.
Design choices.

Where to put the red. The labels in column 0 of the values panel already use the per-field pastel highlight colors. Recoloring those would lose the color-coding the user is used to. Better: outline the entry/text widget itself in red — change bd to 2 and highlightbackground/highlightthickness so the field's input box gets a red border. That visually surrounds just the affected fields without touching the existing labels or text contents.
State tracking. Need a self.engineer_overridden_fields: set[str] on the App. Set it when an engineer report applies (in both apply_engineer_report_session and combine_instruction_and_engineer_report flow). Clear it when a new document is dragged in (which is the start of load_files, before sessions are built — that way it's cleared regardless of whether the new drop triggers an engineer-report flow or a normal flow).
Which fields count as "overwritten". The merge logic only writes a value if engineer_session.values.get(key) is non-blank. So a field is "overwritten" when (a) the engineer session had a non-blank value for it, and (b) that value is different from the instruction's existing value. If the engineer report happens to produce the exact same value as the instruction, arguably that's not "replacement" — but pragmatically the user can't tell from looking. I'll go with: if the engineer report would have written to the field (non-blank value present), mark it as overridden. That matches the user's mental model of "the engineer report touched this field".
Actually, on second thought — if the value didn't actually change, highlighting it red is misleading. Let me only mark it red when the value changed. That's the "replaced" semantics from your message.
Batch mode. The simultaneous drop case (combine_instruction_and_engineer_report) is already handled in the same place — it merges into a single session shown immediately. So I just need to compute the override set there too. Multi-file batch mode without an engineer report doesn't apply (no overrides happen). And when batch mode does happen with an engineer report, the existing logic forces it into the "single merged session" path, never the multi-session batch path — which the existing error messages confirm.
Storage on session. When a normal document is shown via show_session (batch nav), I need to know whether overrides happened. But batch mode + engineer report can't coexist, and the merged session is single-file. So engineer_overridden_fields can be a simple App-level set, no per-session storage needed.

One edge case worth flagging. The flow at apply_engineer_report_session is the "drop instruction first, then drop engineer report after" path. The simultaneous-drop path goes through combine_instruction_and_engineer_report and then through the same for key in FIELD_KEYS: self.set_field_value(...) loop. Both need the override set computed. I'll factor out a small helper.
Implementing now.Viewed 3 files, ran a commandViewed 3 files, ran a commandImportant note: notice that apply_engineer_report_session is the path for "drop instruction first, then drop engineer second". The single-drop engineer-after path goes through load_files → ends at apply_engineer_report_session (line 2714 area). But load_files clears the override set at the top of load_files — which would wipe the existing instruction's overrides... wait, the instruction was loaded earlier in a separate load_files call. So when the engineer report is dropped second, the instruction has already been displayed (no overrides yet — fresh load). Then load_files runs again, and at the top I'd want to clear the override set... but I'm about to set new overrides in apply_engineer_report_session.
So the cleanup should happen before sessions are processed but after it's known there's a normal new document. Better plan: clear the override set at the very top of load_files, then apply_engineer_report_session and combine_instruction_and_engineer_report set it. Since load_files is followed immediately by overrides being set if relevant, this works. The "fresh document = clear" rule maps cleanly to "any new drag-drop = clear, then engineer flows re-set as needed."
Now the implementation:
Edit 1: Add the override set to __init__, plus storage of the entry widgets so we can change their borders.Track engineer-overridden fields and the value entry widgetsTrack engineer-overridden fields and the value entry widgetsEdit 2: Register the entry widgets in build_values_panel. Use tk.Entry and tk.Text highlightbackground/highlightcolor/highlightthickness to draw a colored ring around the widget.Track entries in value_entry_widgets so override highlighting can find themTrack entries in value_entry_widgets so override highlighting can find themEdit 3: Add the apply/clear methods for the override highlight, and a small helper to compute which fields actually got replaced.Find a good spot to insert the new helpersFind a good spot to insert the new helpersI'll add a constant for the red color and then insert the helper methods just above clear_field_highlights:Add the override color as a module-level constantAdd the override color as a module-level constantNow the helper methods. I'll put them right above clear_field_highlights:Viewed a file, edited a fileViewed a file, edited a fileEdit 4: Hook clear_engineer_override_highlights into the load_files entry point. Plus into clear_form so that anywhere the form gets cleared, the borders are cleared too. Then call apply_engineer_override_highlights at the end of both engineer-merge paths.Clear override state when the form is cleared (covers all reset paths)Clear override state when the form is cleared (covers all reset paths)Now: looking at load_files, the early "no sessions could be loaded" branch at line 2690 doesn't go through clear_form. And the "drop engineer report after instruction" path (where len(clean_paths) == 1 and len(engineer_sessions) == 1 and not instruction_sessions) doesn't go through clear_form either — it hits apply_engineer_report_session directly which preserves the existing form state. So I should NOT clear the override set at the top of load_files blindly — because if someone drops an engineer report onto an instruction, I want the apply path to be the one that sets the new override state. The right move: clear the override set whenever we're about to display a fresh primary document (the three "single session" / "merged session" / "batch" cases that all go through clear_form), and have apply_engineer_report_session set/append.
Let me check this more carefully by re-reading load_files:Load_files full bodyLoad_files full bodyThe logic structure:

Engineer-only drop (after a previous instruction): goes to apply_engineer_report_session directly. Doesn't call clear_form. The override set should be set here.
Combined drop (instruction + engineer at once): calls clear_form then merges. The clear_form would clear overrides, then the merge sets new ones. Good.
Single normal drop: clear_form then load. Overrides cleared, none set. Good.
Multi-file batch drop (no engineer report): clear_form then show first session. Overrides cleared, none set. Good.

So I just need to:

Make apply_engineer_report_session compute and set engineer_overridden_fields, then call apply_engineer_override_highlights. (It does NOT clear first, because we want to track overrides cumulatively if some were already there from previous combined drops? Actually no — if user dropped instruction → then engineer → then engineer again, that second engineer is rejected by existing rules. So it's a fresh "engineer being applied right now" event. Set, don't append.)
Make combine_instruction_and_engineer_report flow set the override set after merging values.

Let me also handle one subtle point: in apply_engineer_report_session, the merged_values set is computed after values were already in the form. So I want to compute _compute_engineer_overrides(self.collect_values(), session.values) before applying changes. The current code does merged_values = self.collect_values() first which is the base — let me preserve that base for the override computation.
Edit 5: Apply override logic to both engineer flows.Compute and apply override highlight in apply_engineer_report_sessionCompute and apply override highlight in apply_engineer_report_sessionNow the combined-drop path. Looking at the flow: combine_instruction_and_engineer_report returns a merged session, which is then displayed. I need to compute the overrides during the combine (where we have both instruction and engineer values) and pass them through. Cleanest way: compute in load_files between the two function calls, and apply the highlight at the end of the merged-display block.Apply override highlight in the combined-drop pathApply override highlight in the combined-drop pathNow compile-check and run all the existing tests, plus a new test for the highlight behaviour:Ran 2 commands, created a fileRan 2 commands, created a fileAll passing — including the inspection_address case (Text widget, which uses the same highlight-ring API as Entry).
Run the previous V42/V43 tests too to confirm no regressions:Re-run V42/V43 tests for regression checkRe-run V42/V43 tests for regression checkNo regressions. Now update README and finalise:Ran a command, read a fileRan a command, read a fileV44 is built. Summary:
What changed. Added a permanent red border around any field on the left panel whose value was replaced by an Engineer Report. The highlight stays until a new document is dragged in.
Implementation. Three pieces:

New state on the App: engineer_overridden_fields (a set of field keys) and value_entry_widgets (dict mapping field key → the entry/text widget).
Three new helper methods: _compute_engineer_overrides (figures out which fields actually changed), apply_engineer_override_highlights / clear_engineer_override_highlights / _set_value_widget_override_border (manage the borders).
Both engineer-report flows now compute and apply the override set: apply_engineer_report_session (drop engineer after instruction) and the combined-drop branch in load_files (drop both at once). clear_form clears the override state, so any fresh document drop wipes the highlights.

Subtle decisions.

"Replaced" means the value actually changed. If the engineer report produces the same value the instruction already had, no highlight — that wasn't really a replacement.
Border drawn via Tk's highlightthickness/highlightbackground ring rather than recoloring the field label — keeps your existing pastel field-color coding intact.
Color is a single module-level constant (ENGINEER_OVERRIDE_BORDER_COLOR = "#d62828") at the top of the file, easy to tweak if you want a different shade of red.

Verified. 18 highlight-related tests pass, including: instruction-only load shows no highlights; an engineer report that changes some fields and leaves others identical highlights only the actually-changed ones; clearing the form removes all highlights; the simultaneous-drop path produces the same highlight set as the sequential-drop path. V42 PDF regression and V43 mileage/presence-check tests still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Final little tweak - can we prevent JSON export if the Work Provider field in the detected fields panel is empty?Final little tweak - can we prevent JSON export if the Work Provider field in the detected fields panel is empty? This should be per single import and batch import. The idea here is if a user imports multiple documents, one of which is just images, then the user clicks Export JSON, an "UnknownVRM.JSON" JSON should not be created for the document that contained only the images (or any document that couldn't be identified, for that matter).6 MayClaude responded: Quick clarification before I implement, because there's a couple of ways this could behave and they have different ergonomics:Quick clarification before I implement, because there's a couple of ways this could behave and they have different ergonomics:
Single import case — straightforward. If Work Provider is empty when user clicks Export JSON, do nothing useful. Question is just: silent no-op + status message, or an error popup? Given the codebase deliberately removed success popups but keeps error popups, I'd say a small error popup ("Cannot export JSON: Work Provider is empty.") plus a status bar message is the right fit. Confirm?
Batch import case — this is the more interesting one. User has 5 docs, 2 have unknown providers, 3 are mapped. They click Export JSON. Should it:
(a) Skip the unmapped ones, export the mapped ones. Status message says "Exported JSON for 3 of 5 files (2 skipped: no provider)." No popup. Most useful, matches the spirit of your request — the unmapped ones just don't get a stray UnknownVRM.json, but the mapped ones still export.
(b) Refuse the whole batch if any are unmapped. Error popup, nothing exported. Stricter but means one bad doc blocks the whole batch.
I'd go with (a). It directly addresses your "don't create UnknownVRM.json for the image-only doc" complaint while still letting the user export the rest of a batch in one click. The skipped ones are listed in the status so the user knows.
Two more small things while I'm in there:

Should this also apply to Export Images? Same logic — Export Images also uses export_base_name, which falls back to UnknownVRM when Work Provider is blank. The unmapped doc would currently produce UnknownVRM_img_1_1.png etc. Probably the same rule should apply: silent skip in batch, error popup in single mode. Worth doing now since it's the same change. Confirm?
What about a doc where Work Provider is set but VRM is empty? Currently that exports as WorkProvider_UnknownVRM.json. Your message specifically targets the "doc that couldn't be identified" case (= no provider matched), not the "provider matched but VRM extraction failed" case. I'll leave that one alone — Work Provider being empty is the gate.

So my read is: gate Export JSON (and Export Images, if you agree) on Work Provider being non-empty. Single mode → error popup if empty. Batch mode → silently skip the unmapped docs, export the rest, status message lists the skipped count. Confirm those two points and I'll implement.You said: No status messages for this scenario, this is a productivity tool and this is expected behaviour, so don't inform the userNo status messages for this scenario, this is a productivity tool and this is expected behaviour, so don't inform the user6 MayClaude responded: Got it — no status messages, no popups, just silently skip.Got it — no status messages, no popups, just silently skip. Single import with empty Work Provider becomes a no-op too (no popup either, given productivity-tool context).
Implementing now.Ran a command, viewed a fileRan a command, viewed a filePlan:

export_json_string (single mode) — early return if work_provider is empty. Silent. Don't set status. Don't show popup.
export_doc (single mode for Images) — early return if work_provider is empty and export_images is True. Same silent behaviour. Don't gate export_doc itself since DOC export was removed anyway, but the function still runs. Looking at the code, on_export_images_click calls export_doc(export_doc=False, export_images=True), so I need to gate when export_images=True.
batch_export — skip individual sessions whose work_provider is empty. The status message at the end already shows the count of files actually processed, which is a behavioural side-effect of the skip but not something I'm adding for this purpose. You said no status messages "for this scenario" — I take that to mean don't add new ones. The existing batch status message ("Completed JSON export for N files") already runs and would just show a smaller N. That seems fine.

Actually wait — re-reading your message: "No status messages for this scenario, this is a productivity tool and this is expected behaviour, so don't inform the user." I think the intent is "the user shouldn't see anything special when this skip happens — it's just normal expected behaviour." The existing batch status message says "Completed JSON export for 3 files" regardless of whether it was 3-of-3 or 3-of-5. That's fine — it's just reporting what was exported, not announcing anything skipped.
One thing to be careful about: a session's work_provider can be in two places. The session was built at import time with values["work_provider"] populated only if a provider was detected (extract_fields line 1362-ish). So checking session.values.get("work_provider", "").strip() is the right gate. For single mode, the equivalent is self.collect_values().get("work_provider", "").strip() (which is what would be on screen in case the user manually edited it).
Let me also think about the edge case where the user manually types a Work Provider into the field. That should count as "identified" — we shouldn't gate on whether a saved provider matched, just whether the field has any text. That matches what collect_values() returns.
For batch mode, the session.values was set at import time. If the user navigates to that file in the batch and types a Work Provider, then navigates away, does that get saved back? Yes — save_current_batch_state() (line 2027) does exactly this. And batch_export calls self.save_current_batch_state() at the top before iterating. So session.values["work_provider"] will reflect any manual edits the user made while viewing that session. Good.
Implementing:Gate single-mode JSON export on Work Provider being non-emptyGate single-mode JSON export on Work Provider being non-emptyNow the single-mode Image export. Let me look at export_doc again — it's the function that handles both DOC and Image exports in single mode:Gate single-mode Image export on Work Provider being non-emptyGate single-mode Image export on Work Provider being non-emptyNow the batch export — skip individual sessions whose Work Provider is empty:Edited a file, ran a commandEdited a file, ran a commandNow write tests. The tricky bit is that the export functions actually write files to disk; I'll redirect app.OUTPUT_DIR to a temp directory and inspect what gets written.Created a file, ran a commandCreated a file, ran a commandAll passing — including the silent-status assertions. Run the V44/V43/V42 regression tests to confirm no breakage:Regression check across all prior test suitesRegression check across all prior test suitesAll four prior test suites pass. Update README and finalise:Ran a command, read a fileRan a command, read a fileV45 is built. Summary:
Three small gates added:

export_json_string (single-mode JSON) — silent return if Work Provider blank.
export_doc (single-mode Images, since DOC export is gone) — silent return if Work Provider blank.
batch_export — per-session check inside the loop; sessions with blank Work Provider are silently skipped, others export normally.

Silent means truly silent. No popup, no status bar update, no log entry. Verified by setting a sentinel status string before calling the gated functions and confirming it's unchanged afterwards.
Why all three places. Single-mode JSON and single-mode Images both fall back to UnknownVRM.json / UnknownVRM_img_*.png filenames via safe_filename(vrm or "UnknownVRM") — the gate prevents either from happening. In batch mode, the iteration already runs through every session, so the gate is just a continue inside the existing loop.
Verified. 9 new tests cover: single-mode silent no-op for both JSON and Images; batch mode skipping individual unidentified sessions while exporting the rest; all-unidentified batch producing zero files; sentinel-status preservation. V42, V43, V44 regression suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: This is just a hypothetical - don't make any code changes yet.This is just a hypothetical - don't make any code changes yet. Is there any real chance of getting .msg imports in a good place? We tried to implement them before but currently dragging one into the application causes the application to freeze indefinitely. .EML import seems to work.The reason I ask is some users are on classic outlook which doesnt direct support EML saving. Classic outlook can save to txt though6 MayClaude responded: Short answer: yes, totally fixable, and the fix is small.Short answer: yes, totally fixable, and the fix is small. The freeze is a specific known issue, not a fundamental capability problem.
What's almost certainly happening. Looking at the code, .msg goes through _extract_msg_text_via_outlook which uses COM automation: DispatchEx("Outlook.Application") then session.OpenSharedItem(...). That call can hang indefinitely in real-world conditions:

Outlook isn't running and a profile prompt is waiting offscreen for a click
Outlook is running but in a modal state (a dialog open, a sync in progress, an Add-in misbehaving)
The .msg file was saved by a different Outlook version and triggers a "convert" prompt
Group policy / security warning ("a program is trying to access Outlook") that the user has to click

OpenSharedItem is a synchronous COM call. The Tk main loop is blocked in the extract_text chain during import. Result: frozen UI, no recovery, and on top of that there's no timeout, no thread isolation, and no fallback path.
Why this is fixable. Two independent angles:

Pure-Python .msg parser (no Outlook at all). A .msg file is a Microsoft Compound File Binary Format container — well-documented, can be read offline. The extract-msg library (BSD-licensed, pure Python plus compressed-rtf and olefile) does exactly this. Drop it into requirements.txt, add an import branch, and 90%+ of .msg files Just Work — including ones from Classic Outlook, New Outlook, and Outlook 365. No COM, no profile prompts, no freezes. This would be the primary path.
Outlook COM as a fallback only. Keep it for edge cases (encrypted/IRM-protected messages, weird embedded structures), but: (a) move it to a worker thread so the UI never freezes, (b) add a timeout (~10s) that aborts cleanly, (c) detect Outlook isn't responding and fail fast. With a pure-Python primary path, COM-required cases become rare enough that the fallback's flakiness is acceptable.

In practice you'd probably get away with just step 1 and could drop the COM path entirely. extract-msg is well-maintained, handles attachments, headers, plain-text body, and HTML body, and is the de facto choice for this exact problem.
On the Classic Outlook → .txt workaround you mentioned. Worth flagging that classic Outlook can also "Save As" → .msg directly (File → Save As → Outlook Message Format .msg). So users don't actually have to fall back to .txt — they can just save .msg, which after this fix would import cleanly. The .txt route is a fallback if for some reason saving .msg isn't available, but it carries less context (often just the body, no headers, sometimes no recipient list depending on settings) and is brittle to parse. .msg via extract-msg is strictly better.
Cost. Small. Probably 30-50 lines of new extraction code that mirrors the existing _extract_eml_text shape (Subject/From/To/Date headers + body), plus removing or threading the COM fallback. One new dependency (extract-msg), which is pure Python so no PyInstaller fuss. Risk is low — .eml working confirms the harness around the message-extraction step is fine; the freeze is purely the COM call.
So when you do want this implemented, it's a clean win and not a research project.You said: If we can implement msg import natively that'd be perfect, let's give that a go.If we can implement msg import natively that'd be perfect, let's give that a go.6 MayClaude responded: Plan: drop in extract-msg, add a primary pure-Python .Plan: drop in extract-msg, add a primary pure-Python .msg reader that mirrors the existing .eml shape, and rip out the COM/Outlook fallback entirely (since it's the freeze source and extract-msg covers Classic Outlook, New Outlook, and 365). I'll handle a few annoying corner cases the library has — RTF-only bodies (some Classic Outlook messages store only RTF), CRLF normalisation, attachment lists.
First let me see what extract-msg actually gives us:Ran 2 commandsRan 2 commandsGood — all the attributes I need exist. Let me check pinning behaviour for requirements.txt:Check extract-msg dependenciesCheck extract-msg dependenciesOK — pulls in a few transitive deps. All pure-Python except none I see compile to native code. Pinning approach: extract-msg>=0.45.0 matches the current 0.x lineage and avoids being too tight.
Let me also do a quick sanity check that I don't have a .msg file to test with locally — I don't, but I can synthesise a minimal test or rely on extract-msg's own behaviour:Inspect Message constructorInspect Message constructorNo way to synthesise here without a real .msg sample. I'll rely on logic-only testing of the new code's behaviour against a mocked extract_msg.Message, which is enough to verify the assembly logic.
Now look at the current .msg and .eml paths so I can mirror the shape:Viewed 3 filesViewed 3 filesNow I have enough. Plan:

Add extract_msg to the optional-imports block at the top of the file (mirror the fitz pattern — set to None if missing).
Replace the entire _extract_msg_text_via_outlook method with a new _extract_msg_text method that uses extract_msg.
Update the .msg dispatch in extract_text to call the new method, and emit a clearer status note.
Add extract-msg to requirements.txt.

Edits:Add optional extract_msg import alongside fitzAdd optional extract_msg import alongside fitzNow replace the Outlook-based .msg extractor with a pure-Python one. The new method needs to:

Open the .msg via extract_msg.Message(path)
Pull Subject, From (using sender), To, Cc, Date — header order matches .eml
Get the body. Preference order: plain body → if blank, fall back to htmlBody (strip tags) → if still blank, RTF body de-encapsulated (extract-msg handles compressed RTF, but the result is RTF code; for our purposes plain/HTML is virtually always present, and falling back to "no body" is fine)
Append a list of attachment filenames at the end (matches the spirit of preserving information)
Normalise CRLF, collapse excessive blank lines, return string
Replace _extract_msg_text_via_outlook with pure-Python _extract_msg_text using extract-msgReplace _extract_msg_text_via_outlook with pure-Python _extract_msg_text using extract-msgNow update the .msg dispatch in extract_text:Update .msg dispatch to use the new pure-Python extractorUpdate .msg dispatch to use the new pure-Python extractorSanity-check syntax then update requirements.txt:Ran 3 commandsRan 3 commandsNow write tests. Without a real .msg file, I'll mock extract_msg.Message with a fake class that simulates each interesting scenario: typical message, HTML-only body, RTF-only body, attachments, missing attributes, exceptions during attribute access. This covers the new code's behavioral surface without needing a binary test fixture.Created a file, ran a commandCreated a file, ran a commandAll 21 tests pass. Verify all prior test suites still pass:Regression check across all prior test suitesRegression check across all prior test suitesNo regressions. Update README and finalise outputs:Ran a command, read a fileRan a command, read a fileV46 is built. Summary:
What changed. Replaced the entire Outlook-COM .msg path with a pure-Python reader based on extract_msg. The old freeze-on-import bug is gone because there's no Outlook process, no profile prompt, no modal dialog to wait on — the .msg compound binary file is parsed directly.
Output shape preserved. I deliberately kept the output mirroring the .eml path (Subject: / From: / To: / Cc: / Date: then body), so any existing provider mapping rules work identically against .msg and .eml. Cc is added (.eml already supported it implicitly via the headers loop, but it was missing from the old .msg reader); only emitted when present.
Body fallback chain. Plain text body first — that's what 99% of .msg files have. Falls back to htmlBody (stripped via the existing _strip_html_tags), then to rtfBody (rare — added a small _strip_rtf_markup helper for it). Real-world Classic Outlook saves with both plain and HTML bodies, so the fallbacks rarely fire.
Attachments listed at the end. Filename only, informational. Handy for context when reviewing the source preview but doesn't pollute the searchable label-mappable text.
Defensive against attribute exceptions. extract_msg occasionally throws when accessing certain attributes on weirdly-encoded messages — wrapped every getattr in a try/except so one bad attribute doesn't break the whole import. Verified by mocking a Message whose to raises.
Dependencies. Added extract-msg>=0.45.0 to requirements.txt. Pure Python, plays fine with PyInstaller. pywin32 stays in requirements (still used by the Word DOC path on Windows machines that have it, even though .msg no longer needs it).
Tests. 21 new tests covering: typical message, HTML-only body, RTF-only body, attachments, defensive attr-access, empty message, missing-library error, and end-to-end dispatch through extract_text. All prior test suites (V42–V45) still pass.
Caveat I'd flag. I haven't run this against a real .msg file because none was uploaded — the tests use a mock extract_msg.Message. The library itself is well-established (extract-msg is the de facto choice for this), so the core read should be solid, but worth doing a manual smoke test on a couple of real messages from your users when you get a chance: one Classic Outlook .msg and one New Outlook / 365 .msg ideally.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: I've attached the msg I import and here I am pasting how it appears in our application:29680 - Mebrahtom Debesay  - KP22LRL  .msgmsgI've attached the msg I import and here I am pasting how it appears in our application:Subject: 29680 - Mebrahtom Debesay  - KP22LRL  
From: Info <info@fairwaylegal.co.uk>
To: Engineers <engineers@collisionengineers.co.uk>
Date: 2026-05-06 12:11:02+01:00
b' \r\n \r\n \r\n v\\:* {behavior:url(#default#VML);}\r\no\\:* {behavior:url(#default#VML);}\r\nw\\:* {behavior:url(#default#VML);}\r\n.shape {behavior:url(#default#VML);}\r\n \r\n \r\n \r\n \r\n \r\n \r\n \r\n \r\n \r\n \r\n New INSTRUCTIONS: 
\r\n \r\n &nbsp; 
\r\n Date:\r\n 06 May 2026 
\r\n Our Ref: 29680-01 
\r\n &nbsp; 
\r\n Our Insured: Name:\r\n Mr Mebrahtom Debesay&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\r\n 
\r\n Address:\r\n 53 Regan Crescent &nbsp;&nbsp; Birmingham&nbsp; B23 5NN 
\r\n Contact number:\r\n ,,&nbsp; 07751577762 
\r\n National Number : \r\n SZ380025B 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n Clients Own :\r\n 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n Accident Date:\r\n 05 May 2026 &nbsp; Time:\r\n 09:25 
\r\n &nbsp; 
\r\n Vehicle Registration Number:\r\n KP22LRL &nbsp;\r\n 
\r\n Make/Model:\r\n Toyota COROLLA DESIGN HEV CVT &nbsp;\r\n Damage: 
\r\n &nbsp; 
\r\n Accident Location:\r\n Water Orton Lane 
\r\n &nbsp; 
\r\n Circumstance:\r\n Our Client Was Driving In Their Lane When They Saw Another Vehicle Attempting To Overtake A Stationary Bus And And Move Into Their Path. Our Client Slowed Down And Came To A Stop\r\n To Avoid A Collision. However, Tp Continued Driving In Our Client\'s Lane And Hit Our Client\'s Vehicle Head On. Dashcam Footage Available. 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n Third Party Name \r\n 
\r\n Third Party Reg: BG06UYX Ford KA COLLECTION Red 
\r\n &nbsp; 
\r\n Third PartyInsurance :Acromas Insurance Co Ltd* *\r\n 
\r\n Third Party Ref:\r\n 1003668708 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n Inspection Location:\r\n 
\r\n &nbsp; 
\r\n Somstar Recovery &amp; Storage&nbsp; Land Of Rea Street &amp; Moseley Street&nbsp; Birmingham&nbsp; B5 6JX 07462530375 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n &nbsp; 
\r\n Should you have any query relating to the attached please kindly respond to \r\n Info@fairwaylegal.co.uk 
\r\n &nbsp; 
\r\n Kind Regards, 
\r\n Emily Doyle 
\r\n Senior Claims &amp; Credit Hire Specialist 
\r\n &nbsp; 
\r\n 
\r\n Fairway Legal Expenses Ltd
\r\n13 Bury &amp; Rochdale Old Road BuryBL9-7TY
\r\n Tel:0161 765 3390 :&nbsp; 0845 260 3000
\r\nFax: 0161 797 7990
\r\nCompany number 5143223 
\r\nAuthorised and regulated by the FCA.
\r\n [www.fairwaylegal.co.uk](https://www.fairwaylegal.co.uk) 
\r\n &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
\r\n Unless otherwise specified, this email and its contents should be treated as \x91Confidential\x92 
\r\n This email and any files sent with it are intended only for the named recipient. If\r\n
\r\nyou are not the named recipient please telephone/email
\r\nthe sender immediately. You should not disclose the content or 
\r\ntake/retain/distribute any copies. 
\r\n &nbsp; 
\r\n \r\n \r\n \r\n'
Attachments: image001.pngShow more6 MayClaude responded: Got it — clear regression.Got it — clear regression. Two things wrong with that output:

The body is being inserted as b'…' (Python repr() of a bytes object). That means extract_msg returned bytes for .htmlBody, and _safe_attr did str(...) on it which produces the b'…' literal instead of decoding the bytes. Easy fix in _safe_attr.
Even after we decode it properly, the body is still raw HTML with VML/Office boilerplate, &nbsp;, smart-quote bytes (\x91/\x92), and so on. Our _strip_html_tags handles <br> and </p> but not the rest. The output you see is what you'd get if HTML stripping never ran — and it didn't, because the bytes-shaped value made it past the "do I have plain body" check via _safe_attr returning the b'…' repr (which is truthy and non-empty), so the HTML fallback never engaged.

Both bugs have the same root: bytes leaking through. Plan:

Make _safe_attr decode bytes properly (try utf-8, fall back to cp1252), strip, return string.
Strengthen _strip_html_tags to handle a few more cases that show up in real Outlook HTML: HTML entities (&nbsp;, &amp;, &#39;, etc.), VML/XML namespace blocks, <style> and <script> block contents, smart-quote bytes.
Add a check in body selection so that if body looks like HTML/VML markup (starts with < after stripping whitespace, or contains <o:p> / <v:), treat it as HTML even though it came in via body. Some Outlook saves do this.

Let me actually verify what the .msg file contains first by running it through extract_msg directly:Probe the real .msg with extract_msgProbe the real .msg with extract_msgNow I can confirm exactly what's going on:

body is None for this message — there's no plain text body at all. So the code correctly fell through to htmlBody.
htmlBody is bytes, and _safe_attr did str(bytes_value) which produces a b'…' repr string — definitely not what we want. Then _strip_html_tags got that repr string and the \r\n literals plus the b' prefix prevented its regex from matching the actual <br/> patterns.
Subject contains a trailing \x00 (null terminator). Need to strip those.
Attachment names also have trailing \x00 for the same reason.
Smart quotes like \x91 and \x92 (cp1252 left/right single quote) need to be handled — they're showing up as \x91 in the output because somewhere encoding fell back to ascii/utf-8 with errors=ignore, leaving the raw byte unprintable.

So the real fixes:

Decode bytes properly when extracting attribute values. Outlook HTML bodies are typically utf-8 or cp1252.
Strip nulls from string attribute values.
Make _strip_html_tags actually handle Outlook HTML — entities, <style>/<script> blocks, VML namespaces, the lot.
Treat the message as HTML when body is None/empty AND htmlBody exists (we already do this, but the bytes bug masked it).

Let me look at the current code again to confirm where to splice:Look at _safe_attr and _strip_html_tags currentlyLook at _safe_attr and _strip_html_tags currentlyNow the edits.
Edit 1: Strengthen _strip_html_tags. This is shared between .eml and .msg, so the improvements help both — the existing .eml path was already working but probably doing a sloppier job than it could on Outlook HTML.Strengthen _strip_html_tags to handle Outlook HTML, VML, scripts, styles, entitiesStrengthen _strip_html_tags to handle Outlook HTML, VML, scripts, styles, entitiesEdit 2: Fix _safe_attr to decode bytes properly and strip nulls. Also change the body selection to detect when a "plain" body is actually HTML (because some tools store HTML in body).Viewed 2 filesViewed 2 filesNow rewrite the inner _safe_attr to decode bytes, plus the body selection logic to detect HTML in body. I'll replace the whole block:Make _safe_attr decode bytes and strip nulls; detect HTML even in 'body'; clean attachment namesMake _safe_attr decode bytes and strip nulls; detect HTML even in 'body'; clean attachment namesCompile check, then run against the real .msg:Ran 2 commands, created a fileRan 2 commands, created a fileThat's a clean, mappable output now. All the previous artefacts are gone:

b'…' byte-repr wrapper: gone
Literal \r\n strings throughout: gone
VML/Office namespace boilerplate (v\:* {behavior:url(...);} etc.): gone (style block stripped)
&nbsp; everywhere: gone (entities decoded)
&amp;: decoded to &
Smart-quote bytes \x91/\x92: decoded to proper apostrophes ('Confidential')
Null bytes (after subject and attachment names): stripped
Attachment list: image001.png, image002.png (and a second one we didn't even know was there)

Now provider mapping rules work cleanly. For example, Vehicle Registration Number: extracts KP22LRL, Our Ref: extracts 29680-01, Accident Date: extracts 05 May 2026, etc.
Let me run the full test suite to make sure I haven't regressed anything:Full regression sweepFull regression sweepAll previous suites still pass. Let me also write a focused test that pins this specific real-world fix so it doesn't regress later:Created a file, ran a commandCreated a file, ran a commandUpdate README and finalise:Ran a command, read a fileRan a command, read a fileV47 is built — a hotfix on V46.
Three bugs, one fundamental cause: bytes leaking through unhandled.

_safe_attr was calling str() on whatever extract_msg returned. For htmlBody that's bytes, and str(b'<html>...') produces the literal string "b'<html>...'" — including the b', the quotes, and \r\n rendered as \\r\\n. That faux-string then looked truthy and non-empty, so the code thought it had a plain text body and never invoked the HTML stripper. Replaced with a proper _coerce helper: decodes bytes (utf-8 → cp1252 → latin-1), strips null bytes, returns clean string.
The HTML stripper was three regexes and didn't handle entities, <style> blocks, VML namespaces, or smart-quote bytes — all of which Outlook HTML emits liberally. Rewrote it to drop <style>/<script> blocks wholesale, decode HTML entities via html.unescape, replace cp1252 smart-quote bytes with their ASCII equivalents, and handle more block-level tags (<p>, <div>, <tr>, <li>, <h1-6>).
Some Outlook saves store HTML directly in the plain body property. Added a quick _looks_like_html sniff (<html, <body, <o:p, <v: near the start) to route those through the stripper too.

Real-world output now: instead of the b'\r\n…&nbsp;&nbsp;…' mess, you get clean labelled lines that map cleanly. The Mebrahtom Debesay sample produces output where Vehicle Registration Number: KP22LRL, Our Ref: 29680-01, Accident Date: 05 May 2026, Make/Model: Toyota COROLLA DESIGN HEV CVT, Inspection Location: Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham B5 6JX 07462530375 etc. all appear as clean adjacent label/value pairs ready for Single Label rules.
Bonus. Picked up a second attachment (image002.png) that wasn't in the screenshot of the original bad output — confirms the attachment loop also benefits from the bytes/null-byte cleanup.
Side benefit for .eml. The improved _strip_html_tags is used by both .eml and .msg paths, so HTML-only .eml imports also got better.
Tests. 30 new assertions in test_v47.py against the real Mebrahtom file, pinned so this regression can't sneak back in. All previous V42–V46 suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Another hypothetical - don't do anything yet.Another hypothetical - don't do anything yet.Is there any way at all to deal with inspection address in this attached .msg?So it appears in our application on a single line like this:"Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham B5 6JX 07462530375"Even our force postcode approach doesn't work because the line ends in a mobile number.My thoughts are that this is a lost cause, and the best scenario is capturing it on one line and asking the user to manually split it up over lines.6 MayClaude responded: Mostly your read is right — fully automated splitting of that line is a lost cause, but there's a useful middle ground worth considering before you give up ent…Mostly your read is right — fully automated splitting of that line is a lost cause, but there's a useful middle ground worth considering before you give up entirely.
Why full automation can't work here. Look at what we actually have:
Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham B5 6JX 07462530375
There are no commas, no consistent capitalisation cues, no "Address Line 1 / Address Line 2" labels, and the postcode is mid-line because of the trailing phone number. To split this into the canonical 6 lines (Name / Address / Town / City / County / Postcode) you'd need to know things that aren't in the text:

Where the business name ends and the street begins (Storage Land Of Rea Street — is "Storage" the last word of the business name, or part of "Storage Land"? Actually the latter, but a model would have to know that)
Whether Land Of Rea Street & Moseley Street is one street name or two (it's a junction reference — both streets, joined)
That Birmingham is a city, not a street name (could be either — there's a Birmingham Road)
That B5 6JX is a UK postcode (this part actually is detectable)

A regex can never get this consistently right. An LLM might, but you've not got one in the loop and adding one is a much bigger architectural change.
What can be improved without giving up entirely. Three things:

Force Postcode should still work even when the postcode isn't at end-of-line. Currently force_postcode only checks \b(postcode)\b\s*$ — postcode followed by end-of-line. Easy fix: scan the whole line for a postcode, and if found, split the line at that point: everything before becomes line 1, the postcode becomes line 6, and everything after the postcode (the phone number, in this case) gets dropped or moved to the body. That single change makes this exact sample produce:

   Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham
   
   
   
   
   B5 6JX
The postcode lands in the right slot, the phone number is gone. Lines 2-5 are blank and the user manually splits the long line 1 across them. That's the "user splits it manually" outcome you described, but with the postcode automatically extracted into line 6 — which is the field RJS export needs in a fixed position.

Optionally drop trailing phone numbers before any further processing, even when Force Postcode isn't ticked. UK mobile/landline numbers have recognisable shapes (07xxx, 01xxx etc.) and removing them is low-risk because they're never legitimately part of an inspection address. Worth doing as a small standalone normalisation step.
Inline editing is already supported. The Inspection Address widget is a 6-line tk.Text (line 2271). The user can already click into it and reflow. The current "asking the user to split it up" workflow is just typing into that widget. So the user-facing UX you're imagining basically already exists.

My recommended approach for when you do want this. Implement #1 only. It's a small, contained change to normalise_inspection_address_value — extend the postcode regex from "end-of-line" to "anywhere in line, with content possibly trailing", and when a postcode is found mid-line in single-line mode, split there and toss the trailing chunk. Still respects the force_postcode_for_inspection_address flag, so existing presets without it ticked are unaffected. The user still has to break up "Somstar Recovery & Storage / Land Of Rea Street & Moseley Street / Birmingham" across lines 1-3 themselves, but the postcode and phone number are handled automatically. Concretely about 10 lines of changed code.
If you want even more, #2 is also worth it — strip trailing phone numbers before any other processing, gated on a new "Strip Trailing Phone" provider checkbox. But it's diminishing returns and adds another preset toggle to explain.
Beyond that, you're into LLM territory and it's not worth it for a productivity tool. Your instinct about asking the user to split it manually is correct — just make the postcode part automatic.You said: Fully agree - let's implement that change to force postcode and leave it there.Fully agree - let's implement that change to force postcode and leave it there.6 MayClaude responded: Plan: extend the Force Postcode logic so when the input is a single line (which is our case here — the .Plan: extend the Force Postcode logic so when the input is a single line (which is our case here — the .msg body collapsed Inspection Location: content into one line) and a postcode is found anywhere in that line, we split there: text before becomes line 1, postcode goes to line 6, anything after the postcode is dropped.
Two things to be careful of:

Don't change behaviour when Force Postcode is OFF. This is gated on the existing checkbox.
Don't change the multi-line branch. If the address already arrived with line breaks (e.g. from a different provider with comma-separated Address: 53 Regan Crescent, Birmingham, B23 5NN), the existing logic already handles it correctly. The change is only for the single-line sub-branch.

Let me re-read the existing function to splice precisely:Ran a command, viewed a fileRan a command, viewed a fileI see exactly what to change. The single-line branch first tries postcode_end_re (postcode at end of line). If not found and force_postcode is on, currently it just gives up and keeps the whole line as body. I'll fall through to postcode_anywhere_re — if that hits, split: pre-postcode → body, postcode → line 6, post-postcode → dropped.
One subtle thing worth checking against the user's exact sample:
Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham B5 6JX 07462530375
After the re.sub(r"\s*,\s*", "\n", text) (line 1674), this stays one line because there are no commas. Then it falls into the len(raw_lines) == 1 branch. Currently postcode_end_re.search fails (postcode isn't at end). With my change, postcode_anywhere_re would match B5 6JX and we'd split: body = Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham, postcode = B5 6JX, trailing = 07462530375 (dropped).
Edit:Edited a file, ran a commandEdited a file, ran a commandNow write tests covering the user's scenario plus edge cases:Created a file, ran a commandCreated a file, ran a commandAll 16 cases pass — the user's exact sample, original end-of-line behaviour, all six UK postcode formats found mid-line, compact-postcode normalisation, edge cases, and force-off being untouched.
Run the full regression sweep:Full regression sweep across V42-V47Full regression sweep across V42-V47All prior suites still pass. Update the README and copy outputs:Ran a command, read a fileRan a command, read a fileV48 is built.
The change. When Force Postcode is ON and the input is a single line, we now do two-stage matching: first try the existing "postcode at end-of-line" pattern, and if that fails fall through to "postcode anywhere in line". On a hit, we split there: pre-postcode → body, postcode → line 6, post-postcode → dropped. About 12 lines of new code in normalise_inspection_address_value.
On the user's exact sample (Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham B5 6JX 07462530375), the result is now:
Line 1: Somstar Recovery & Storage Land Of Rea Street & Moseley Street Birmingham
Line 2: 
Line 3: 
Line 4: 
Line 5: 
Line 6: B5 6JX
The user clicks into the inspection address widget and manually breaks line 1 across lines 1-3 (or however they want it laid out). The postcode and phone number are handled automatically, which is the part that's actually mechanical.
What's preserved. Force Postcode OFF behaves identically to before. End-of-line postcode behaviour is identical. Multi-line and comma-separated inputs are untouched (the new logic only fires inside the single-line branch). The compact-postcode normalisation (B634AB → B63 4AB) still works mid-line too.
Tests. 16 new assertions including the literal Mebrahtom sample, all six UK postcode shapes mid-line, compact-postcode handling, force-off non-interference, multi-line non-interference. All V42–V47 suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Is it also possible for the "Export Images" option to work on .Images .docxdocxIs it also possible for the "Export Images" option to work on .docx files, or is this not plausible? It currently works perfectly for PDFs6 MayClaude responded: Interesting — the code already supports DOCX image extraction.Ran a command, viewed a fileRan a command, viewed a fileInteresting — the code already supports DOCX image extraction. There's a .docx branch (line 1625) that calls extract_from_docx(), which reads the zip and pulls everything from word/media/. So the question becomes: why isn't it working in practice?
Let me actually try it on the uploaded file:Ran 2 commandsRan 2 commandsSix JPEGs in word/media/, exactly where the existing code looks. So the existing extraction logic should work. Let me run it:Confirm DOCX image extraction actually works on this fileConfirm DOCX image extraction actually works on this fileThe engine extracts all 6 images correctly. So the engine code is fine. The bug must be elsewhere — most likely the export path is gated such that the user never reaches extract_images_to_desktop for DOCX files. Let me check:Ran a command, viewed a fileRan a command, viewed a fileNo gating that excludes DOCX from image export. So the path should work end-to-end. Let me reconcile: you said it doesn't work. There are two possibilities:

The user-facing path actually works and you just hadn't tried with a DOCX containing extractable images, OR
There's a subtle bug I'm missing.

Let me trace the full path with the actual UI to be sure:Created a file, ran a commandCreated a file, ran a commandIt works. All 6 JPEGs export cleanly to the Desktop with the expected TEST_AB12CDE_img_N.jpeg naming scheme. The pipeline is end-to-end functional, no code changes needed.
So the answer to your question is: it already works, no implementation needed. The DOCX image extraction has been there since the original V40 code (line 1605-1611 in extract_images_to_desktop), reading directly from the DOCX zip's word/media/ folder.
A couple of things worth noting on what might have caused you to see it not working:

The V45 gate. As of V45, Export Images silently no-ops if Work Provider is empty. If you tested by dragging in a DOCX that didn't match any provider and didn't manually type a Work Provider, nothing would happen — and given the deliberately silent behaviour, no error message would tell you why. That's by design but it's the most likely explanation.
Empty DOCX text. Notice current text length: 0 in my run. The "Images_.docx" file only contains images (no text), so provider detection has nothing to match against. extract_text succeeded but returned an empty string, which means provider detection returned Unknown / Unmapped, which means Work Provider stayed blank, which means V45's gate kicked in. In my test I worked around it by manually typing TEST and AB12CDE into the fields. A user would have to do the same.

So the question becomes more interesting: if a user drops in an image-only DOCX, they currently have to manually type in Work Provider and VRM before Export Images works. Is that what you want, or do you want some special-case handling here? Three options:
(a) Leave as-is. "Document doesn't have provider info → user must identify it" is the deliberate behaviour we agreed for V45 to prevent stray UnknownVRM.json files. Same logic applies to Images. User types Work Provider and VRM, hits Export Images, all six images land on the Desktop with the right names.
(b) Loosen the V45 gate for Image export specifically. Allow Export Images to fire even with blank Work Provider, falling back to UnknownVRM_img_N.jpeg. This re-introduces the noise we just spent V45 removing.
(c) Different gate for Image vs JSON. Image export only requires VRM (not Work Provider), since the filename is mostly about uniquely identifying the vehicle.
I'd suggest (a) — it's consistent with what we just established. But if you wanted me to do something here, (c) is the most defensible compromise.
Want me to do anything, or should I close this out as "already works, no changes needed"?You said: Image export should require nothing - do not skip image export for documents with no detected work providerImage export should require nothing - do not skip image export for documents with no detected work provider6 MayClaude responded: Got it.Got it. Image export should always run regardless of Work Provider being empty.
Quick clarification before I go: there are two ways the "always run" behaviour could fall back when Work Provider is blank, and they differ in what filenames you get:
(a) Use UnknownVRM fallback — the same filename pattern that existed before V45. Files would be UnknownVRM_img_1.jpeg, UnknownVRM_img_2.jpeg, etc. for an image-only DOCX with no metadata at all. If Work Provider is set but VRM isn't, you'd get the existing WorkProvider_UnknownVRM_img_*.jpeg.
(b) Use the source filename when Work Provider is blank — so Images_img_1.jpeg, Images_img_2.jpeg. Probably more useful for users since they can tell which doc the images came from. If multiple files in a batch, each one gets its own prefix.
I'd recommend (b) — it gives you a useful filename that ties images back to their source, and matches the spirit of "image export should always work" because the resulting files are immediately useful, not generically named. Source-filename prefixing also handles batch mode well: drop 5 image-only DOCX files, get 5 cleanly-namespaced sets of images on the Desktop.
Should I do (a) — keep the current UnknownVRM fallback and just remove the V45 gate for Images — or (b) — fall back to the source filename when Work Provider is empty?You said: BB6 MayClaude responded: Plan:Plan:

Remove the V45 Work Provider gate from Image export only — both the single-mode export_doc(..., export_images=True) path and the batch-mode loop in batch_export.
Change export_base_name to fall back to the source filename's stem (e.g. Images_.docx → Images_) when Work Provider is blank.
Make sure JSON export still respects the V45 gate (Work Provider required).

Two design points worth flagging:

export_base_name is shared between JSON and Images. Right now it returns UnknownVRM when both Work Provider and VRM are blank. If I change that fallback unconditionally, JSON export would also start writing files with source-filename basenames when Work Provider is empty — but JSON export is gated upstream by V45, so this never actually happens. Still, to keep the boundaries clean, I'll pass the source path into export_base_name so it can compute the fallback only when needed.
Batch mode for Images. Each session can have its own source path, so each image set gets its own prefix derived from its own filename. This naturally handles your "drop 5 image-only DOCX files" scenario.
Ran a command, viewed a fileRan a command, viewed a fileOne thing to think about: export_outputs_for_values is called with export_images=True for all sources, but base_name is computed once per call. In batch mode each session calls export_outputs_for_values separately, so each session gets its own base name — good. But in single mode the call passes a list of sources (source_file=self.current_source_files or [self.current_file]) and loops through them on line 3090-3094 sharing the same base_name. That comes into play when an instruction + engineer report were combined: current_source_files contains both files. For Images, that means images from both docs end up sharing one base_name prefix. That's existing behaviour and not what this change is about — leaving it alone.
For source-filename-based naming when Work Provider is blank, the right anchor is the primary source file (first in the list). I'll thread the source path into export_base_name and use the stem of the first source.
Edit 1: Update export_base_name to take an optional source-path fallback:Export_base_name falls back to source filename stem when Work Provider is emptyExport_base_name falls back to source filename stem when Work Provider is emptyEdit 2: Pass the primary source path to export_base_name from export_outputs_for_values:Forward primary source path into export_base_nameForward primary source path into export_base_nameEdit 3: Remove the V45 Work Provider gate from export_doc only when exporting images. JSON export still goes through export_json_string which has its own gate. The DOC export was removed long ago, so this export_doc function is effectively the Images path:Remove the Work Provider gate for Image export in single modeRemove the Work Provider gate for Image export in single modeEdit 4: Remove the gate from batch_export only when exporting images:Viewed a file, edited a file, ran a commandViewed a file, edited a file, ran a commandNow write tests covering the new behaviour and confirming JSON export is still gated:Created a file, ran a commandCreated a file, ran a commandAll 12 V49 cases pass. Note one nuance to highlight: when there's no provider but the user has typed a VRM into the form (case 4 of the unit checks), the source-stem still wins over the VRM. That's deliberate — it's consistent ("no provider → use source filename, period") and avoids the confusing case where you'd get filenames like AB12CDE_img_1.jpeg with no indication of which doc they came from.
Run the full regression sweep:Full regression sweep V42-V48Full regression sweep V42-V48V45 has one failure — that's correct and expected. The V45 test asserted that single-mode Images export with blank Work Provider would silently no-op. We've just deliberately changed that — now Images export does run regardless. The V45 assumption is no longer true. I need to update the V45 test to reflect the new V49 behaviour:Viewed 2 files, ran a commandViewed 2 files, ran a commandUpdate the V45 test — the JSON gate is still in effect, but the Images gate is gone:Update V45 silent-gating test to match V49 behaviourUpdate V45 silent-gating test to match V49 behaviourAlso need to update the V45 test that says "single Images export with empty work_provider produces no files" — that's no longer true. But that test fixture's source path is /tmp/fake.pdf which doesn't exist on disk, so image extraction will silently fail to find images and produce no files anyway. Let me verify that's actually still the case:Re-run V45Re-run V45V45 now passes. The earlier "single Images export with empty work_provider produces no files" assertion happens to still pass because the fake /tmp/fake.pdf doesn't exist as a real file, so extract_images_to_desktop adds an "image extraction failed" note and produces nothing — but that's coincidental rather than because of any gate. The assertion is misleading in name now (it implies gating, which is gone). Let me update it to reflect what's actually being tested:Find the misleading V45 image assertionFind the misleading V45 image assertionReplace the misleading Images assertion in V45 — Images export is no longer gated:Edited a file, ran a commandEdited a file, ran a commandFinal regression sweep:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV49 is built. Summary:
The functional change. Removed the Work Provider gate from Image export in both single mode and batch mode. JSON export still requires Work Provider — that gate stays.
The naming change. When Work Provider is empty, export_base_name now falls back to the source file's stem (e.g. Images_.docx → Images_). When Work Provider is set, behaviour is unchanged (Provider_VRM_*). Source filenames are still run through safe_filename so awkward chars get sanitised.
Three small edits:

export_base_name(values, source_path=...) — added the optional source_path parameter and the new fallback rule.
export_outputs_for_values — picks the primary source path (first if a list, e.g. instruction + engineer report combined) and threads it through.
export_doc and batch_export — the V45 Work Provider gate is now scoped to non-Image exports only.

Real-world result on your test file. Dragging in Images_.docx with no provider config matching, no manual edits, just a click of Export Images, produces all 6 JPEGs on the Desktop named Images__img_1.jpeg through Images__img_6.jpeg (the trailing underscore comes from your source file actually being named Images_). If you'd manually typed in a provider and VRM first, you'd get Provider_VRM_img_*.jpeg as before.
Tests. 12 new V49 cases including the actual Images_.docx end-to-end, batch mode mixing identified and unidentified sessions, awkward filenames going through safe_filename, and export_base_name unit checks for every fallback combination. V45's now-obsolete "Images export gated" assertion was updated to reflect the new behaviour. All other prior suites (V42-V48) still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Last little change - the Engineer Report overwrite feature, can the red highlight appear regardless of whether the value changed?Last little change - the Engineer Report overwrite feature, can the red highlight appear regardless of whether the value changed? If the value was detected and entered but didn't change, we still want the red highlight just to confirm that the mapping method is effective.6 MayClaude responded: Got it — the rule changes from "highlight when changed" to "highlight when touched" (i.Got it — the rule changes from "highlight when changed" to "highlight when touched" (i.e. the engineer report produced a non-blank value for that field, regardless of whether it differed from the existing value).
This is one small change to _compute_engineer_overrides:Ran 2 commands, viewed a file, edited a fileRan 2 commands, viewed a file, edited a fileNow V44's tests will fail because they were written specifically against the "only-changed-fields" rule. Let me update them to match the new behaviour:Confirm V44 fails as expectedConfirm V44 fails as expectedAs expected — the failures are exactly the assertions that reflect the old "only-changed" rule. I'll update those:Ran a command, viewed a fileRan a command, viewed a fileThe fixture has the engineer session producing non-blank values for: work_provider, vrm, vehicle_model, claimant_name, incident_date, mileage, mileage_unit — and blank for reference (only reference is blank in the engineer values). work_provider isn't in NON_PROVIDER_FIELDS so it's excluded.
So the new expected set is: {vrm, vehicle_model, claimant_name, incident_date, mileage, mileage_unit}. Only reference should be unhighlighted (because the engineer report had nothing for it).Edited 2 files, viewed a file, ran a commandEdited 2 files, viewed a file, ran a commandFull regression sweep:Full regression sweep V42-V49Full regression sweep V42-V49All clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV50 is done — tiny change.
The functional change. _compute_engineer_overrides now adds a field to the override set whenever the engineer report's value for that field is non-blank, regardless of whether it differs from the existing value. Was: "non-blank AND different". Now: "non-blank".
Six-line edit in the helper, plus a docstring rewrite explaining the new semantic. The base_values parameter is now unused but kept in the signature so neither call site has to change — it's just del'd at the top of the function for explicitness.
Tests updated. V44's three "highlight only when changed" assertions had to be flipped to "highlight on touch", with reference (the only field the engineer report fixture leaves blank) being the one expected NOT to highlight. All other suites (V42, V43, V43_ui, V45, V46, V47, V48, V49) still pass unchanged.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Analyse my settings file.providers.jsonjsonAnalyse my settings file.Look at the two providers that begin with FW.Can we modify the auto-detect behaviour to require all the key terms to be present? Currently all documents that resemble the FW format are being detected as FW (Garage), when we need documents without "Inspection Location:" to be detected as FW (Solicitor)6 MayClaude responded: Let me look at the file: I see what's happening.Let me look at the file:Look at uploaded providers.jsonLook at uploaded providers.jsonI see what's happening. Let me also look at the truncated section to make sure I have the full picture:Check the truncated middle sectionCheck the truncated middle sectionNow I have full context. Let me also re-read the current detection logic to confirm what's wrong:Viewed 2 files, ran a commandViewed 2 files, ran a commandNow I fully understand. Here's the analysis:
What's happening with FW (Solicitor) vs FW (Garage)
Both FW providers are designed for documents from Fairway Legal:

FW (Solicitor) has one detect phrase: fairwaylegal
FW (Garage) has two detect phrases: fairwaylegal AND Inspection Location:

The current detection treats phrases as an OR: any phrase matching counts as a hit, with score weighted by phrase length and number of matches.
For a Solicitor-style document (which contains fairwaylegal but NOT Inspection Location:):

FW (Solicitor): 1 match, score = len("fairwaylegal") = 12
FW (Garage): 1 match (fairwaylegal), score = 12
Tied on score and match_count. The tiebreaker is longest_match, again equal. So whichever was checked first wins.

For a Garage-style document (which contains both):

FW (Solicitor): 1 match, score = 12
FW (Garage): 2 matches, score = 12 + len("Inspection Location:") = 12 + 20 = 32 → wins ✓

So Garage docs work fine, but Solicitor docs are basically a coin flip — and the way the providers are ordered in the config (FW (Garage) listed before FW (Solicitor)), Garage comes out on top of the tied case.
The fix you're asking for
You want all of a provider's detect phrases to be required. If FW (Garage) requires both fairwaylegal and Inspection Location:, then a Solicitor doc that only has fairwaylegal will fail FW (Garage) entirely and only match FW (Solicitor).
This is actually the more intuitive semantic — most users would expect listing multiple phrases means "this is a fingerprint set, all required" rather than "any of these counts."
What the change would look like
In detect_provider, replace the "any phrase matching counts" loop with "all phrases must match, otherwise the provider is skipped entirely". Concrete change:
pythonfor phrase in phrases:
    if matched:
        match_count += 1
        ...
    else:
        # As of vXX, every detect phrase must be present.
        match_count = 0
        break

if match_count != len(phrases):  # all required
    continue
Then match_count always equals len(phrases), and the score is just the sum of phrase lengths — the existing tiebreaker logic still picks the more-phrases-required provider when both match (so FW (Garage) wins on a Garage doc, FW (Solicitor) on a Solicitor doc).
One thing to flag
This is a breaking semantic change. Looking through the rest of your providers: every single one has only one detect phrase, so they're all unaffected. But it's worth being explicit that any future provider with multiple phrases will require ALL of them, not ANY. The old behaviour was OR-of-phrases.
Quick sanity check before I implement: my reading is the FW (Garage) provider name implies "this is a Fairway document where the inspection happens at a physical garage location" — so it logically requires the Inspection Location: field — and FW (Solicitor) is the catch-all "regular Fairway document with no garage location, image-based assessment instead". Is that right?
If yes, the rule "all phrases must match" is exactly the right fix. Want me to implement?You said: Yes do it.Yes do it.6 MayClaude responded: Now write tests covering the FW scenario and adjacent edge cases: All 10 cases pass.Edited a file, ran a commandEdited a file, ran a commandNow write tests covering the FW scenario and adjacent edge cases:Created a file, ran a commandCreated a file, ran a commandAll 10 cases pass. The FW scenario works correctly regardless of order in the providers list, single-phrase providers are unchanged, and edge cases (no phrases, unrelated docs) still behave correctly.
Run the full regression sweep:Full regression sweep V42-V50Full regression sweep V42-V50All clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV51 done.
The change. detect_provider now uses AND-of-phrases instead of OR-of-phrases. The matching loop short-circuits with all_matched = False and break the moment any required phrase is missing, then the outer if not all_matched: continue skips that provider entirely. About 6 lines changed.
On the FW scenario, end-to-end:

Solicitor doc (has fairwaylegal, doesn't have Inspection Location:) → FW (Garage) is filtered out, FW (Solicitor) matches → detected as FW (Solicitor) ✓
Garage doc (has both) → both providers match the AND check, FW (Garage) wins on the existing tiebreakers (more phrases matched, higher total score) → detected as FW (Garage) ✓
Order in the providers list no longer affects the outcome (verified by reversing the order in tests).

Single-phrase providers (RJS, AX, EVA, CNX, LAW, OAK, PCH, QCL, QDOS, SBL — every other provider in your settings) behave identically to before. AND of one phrase is the same as OR of one phrase.
Tests. 10 new V51 cases including order-independence, multi-word phrases, and the empty-phrases edge case. All V42-V50 prior suites pass unchanged.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: When using the "Two Labels" mapping approach, one of the labels cannot simply be "-" and nothing else, when a user enters this into the provider setup panel an…When using the "Two Labels" mapping approach, one of the labels cannot simply be "-" and nothing else, when a user enters this into the provider setup panel and clicks save, then tries to use the mapping, the "-" has disappeared.7 MayClaude responded: Quick check — let me trace what's happening: Found it.Quick check — let me trace what's happening:Viewed 2 files, ran a commandViewed 2 files, ran a commandFound it. clean_value strips - from both ends of the string (line 1127: value.strip(" :-\n")). When the user types - as a label, parse_two_label_config calls self.clean_value("-") on it, strip(" :-\n") eats the only character, returns "" — and the label silently vanishes.
This is the same clean_value that's also used to clean extracted values, where stripping a trailing : or - is genuinely useful (Address: 123 High St → 123 High St). But applying that same logic to user-typed labels in the provider editor is wrong: it loses information the user explicitly entered.
The right fix is to use a different cleaning function for labels — one that preserves leading/trailing punctuation. Let me also check where else parse_two_label_config runs clean_value and what other paths might silently eat user input:Find all clean_value usesFind all clean_value usesMost of those uses are extracting values from documents or cleaning up matched text — those should keep stripping :- because that's the right behaviour for matched text (Reference: ABC-123 → label is Reference, value should be ABC-123 not : ABC-123).
The wrong uses are specifically when handling user-typed labels. There are two:

parse_two_label_config (line 1417 and 1418) — runs on user-typed two-labels config. The bug.
extract_after_label also calls self.clean_value(label) on the user's Single Label config (around line 1191/1246). Same bug class — if a user types - as a single label, clean_value would erase it. But this is a much weirder edge case for single-label since you'd be looking for "the value after a literal hyphen."

Let me focus narrowly on the reported bug — the two-labels case — but also handle the single-label case while I'm in there since it's the same class of bug.
Plan: introduce a new _clean_label (or similar) that preserves labels as the user typed them, only normalising whitespace. Use it in parse_two_label_config and the label-cleaning sites for extract_after_label config tokens. Leave clean_value alone for extracted-value cleanup.
Actually let me think about this more carefully. There are subtler scenarios:

User types Address: as a label. They expect Address: to be matched. clean_value strips the trailing : to give just Address. Currently this happens to work because extract_after_label tolerates trailing : in the document text via its :? regex.
User types Address as a label. clean_value returns Address. Works.
User types - as a label (the reported bug). clean_value returns "". Broken.
User types Address as a label. clean_value strips leading/trailing whitespace. Works.
User types Address- as a label. clean_value strips the -. Works for finding Address-: value, fails for finding Address-value.

The key point: stripping :, -, and whitespace from the outside of a label was a "normalisation" that made the matching more lenient. But for users typing a label like -, that normalisation removes the entire intent of the label.
Cleanest fix: in label-cleaning paths, strip whitespace only — never characters that the user might be intentionally trying to use as the label itself. That preserves the existing leniency for trailing : (which is handled by extract_after_label's :? pattern anyway) without erasing standalone single-character labels.Look at extract_after_label to see exactly how labels are cleaned thereLook at extract_after_label to see exactly how labels are cleaned thereOK so extract_after_label and friends always clean both the user-typed label AND the document text the same way before comparing. If I change label-cleaning to be stricter (don't strip :/-), I also need to change document-line cleaning during comparison to match.
Wait, that's not quite right. Look at line 1201: norm_line.rstrip(":") == norm_label. The code already explicitly handles trailing : on document lines by rstripping it — so we don't need clean_value to strip : from labels for matching to work. The current behaviour is double-handling.
Best fix: make clean_value on user-typed labels strip only whitespace, not punctuation. The : handling already exists in match logic. The result:

- typed by user → preserved as -
Address: typed by user → preserved as Address: (matches against document line Address:)
Address typed by user → preserved as Address (matches against Address or Address: via existing :? logic)

Actually, let me look more carefully. If user typed Address: and we don't strip :, then norm_label = "address:". Line 1203 checks norm_line.startswith(norm_label + ":") → looking for address::. That'd break documents that use Address: as the label.
So removing : stripping from labels would break the very common case where users type Address:. The fix needs to be more surgical: strip whitespace and : from labels (since those have established semantics in the document), but not - (which has no special meaning as a delimiter).
Simplest correct fix: change clean_value from strip(" :-\n") to strip(" :\n") — no longer strip hyphens. Then - typed by the user is preserved.
Let me think about what could break. clean_value is also used on extracted document text. If a document has Reference: ABC-123 and the matcher returns ABC-123, clean_value strips whitespace to ABC-123. Was the trailing-hyphen stripping ever doing useful work? If a document has Reference- ABC and returns ABC... no, clean_value strips from BOTH ends. Could it strip a useful trailing hyphen from an extracted value?
Looking at the code, clean_value on extracted matches from regex (.+?) is mostly cleaning up whitespace. The trailing - strip would only fire on documents where extracted values end with - — extremely unusual. Most likely the - was added to strip defensively to handle labels with trailing dashes, not to clean values.
So changing strip(" :-\n") → strip(" :\n"):

Fixes the reported bug ✓
Existing label-cleaning of Address: → Address still works (: still stripped) ✓
Extracted values that legitimately end in - (rare) would now keep the -. Probably better behaviour.
Labels with trailing hyphens like Address- would keep the -. Probably better behaviour.

That's a one-character fix. Let me do it.Edited a file, ran a commandEdited a file, ran a commandNow write tests. I want to verify three things: (1) the reported bug is fixed, (2) the common Address: → Address cleaning still works, (3) extraction still produces correct results in the existing scenarios:Created a file, ran a commandCreated a file, ran a commandAll passing — including the round-trip through the editor save/load cycle, which is exactly the scenario the user reported. Run the full regression sweep:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV52 done — single-character fix.
The bug. clean_value was calling strip(" :-\n"), which strips hyphens from both ends. A label of "-" got both characters stripped (it's the same character on both ends), leaving "". This happened during parse_two_label_config, which runs on every save and load — so the label was being silently erased on save and confirmed-gone on load.
The fix. Changed strip(" :-\n") to strip(" :\n"). Hyphens are now treated as ordinary label characters. Surrounding whitespace and leading/trailing colons still get stripped, so the very common Address: → Address label-cleaning behaviour is unaffected.
Why this is safe. The trailing-colon stripping is what makes Address: and Address interchangeable as labels, and that was the only "useful" punctuation-stripping clean_value was doing. The hyphen stripping was probably defensive, never load-bearing — it ran on extracted document values too, but documents whose values legitimately end in - (or labels with trailing dashes) are now preserved correctly. If anything that's a better behaviour for those edge cases too.
Tests. 19 V52 cases including the round-trip through the App's save/load cycle (the exact user-reported scenario), plus regression checks confirming Address:-style labels still get cleaned. All V42-V51 prior suites still pass unchanged.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Unfortunately, that hasn't solved my specific issue.Unfortunately, that hasn't solved my specific issue. It turns out that having one of the labels as simply "-" in a complex document means you accidentally capture more than intended as "-" is a frequent character.Here is the source text:"Mobile:
07936853974
Vehicle:
Toyota Prius - BR19 SRXYour instructions to prepare this report is for the benefit of the Court as per your obligation under the"I've cut it off beyond that point as it's irrelevant. I need a mapping approach for capturing the VRM, none of the current approaches fit. I don't think a specialised approach like our postcode approach will work since we deal with private plates as well.Show more7 MayClaude responded: Let me look at this carefully before suggesting an approach.Let me look at this carefully before suggesting an approach.
The text:
Mobile:
07936853974
Vehicle:
Toyota Prius - BR19 SRX
A few approaches I can imagine, ranked by how likely they are to work cleanly:
Approach 1 — Take the last word of the value after a label. User configures Single Label Vehicle:. Existing behaviour: returns Toyota Prius - BR19 SRX. New "Last Word" variant: returns BR19 SRX (the last whitespace-separated token, possibly two tokens if VRMs typically have a space). Issue: VRMs aren't always at the end. Some documents might have Vehicle: Toyota Prius - BR19 SRX (red) or Vehicle: BR19 SRX Toyota Prius. Plus "last word" isn't really meaningful when private plates can be 1 or MR1 or SOM3 ONE.
Approach 2 — Take the value after the last hyphen on the line. User configures something like Vehicle: ... - meaning "after Vehicle:, find the last -, take what's after it". Returns BR19 SRX. Works for this exact format. Fragile if a document ever has Toyota Prius - red - BR19 SRX (still works, last hyphen wins) or Vehicle: BR19-SRX (no hyphen-with-spaces, fails).
Approach 3 — User specifies start label AND a delimiter character. Effectively a generalised Two Labels where the second "label" is a delimiter (last-occurrence). User config: Vehicle: || -. Different from current Two Labels semantic (which captures between). This would be a new mapping method "Last Delimiter": find start label, then find the last occurrence of the delimiter on the same line/block, return what's after.
Approach 4 — Smarter VRM heuristic post-processing. Same as the postcode approach: User configures Vehicle: and there's a VRM-shape regex that finds the VRM-shaped token in whatever was extracted. Issue: as you said, private plates break this. UK private plates can be 1-7 characters, almost any letter combination. There's no robust regex that says "this is a UK VRM" without false positives or false negatives.
Approach 5 — Last whitespace-separated token (or last N tokens). Vehicle: returns the value, then take the last 1-2 words. For this exact document, BR19 SRX is the last 2 words. For Vehicle: BR19SRX (compact), it's the last 1 word. Could allow user to specify N. Fragile to documents where the VRM isn't last.

Honestly, looking at your specific problem, I think Approach 3 (a new "Last Delimiter" method) is the cleanest fit. Here's why:

It generalises beyond just -. Same method handles Vehicle: Make Model | VRM (delimiter |), or Vehicle: Make Model -- VRM (delimiter --), or any future weird format.
It's predictable: user states exactly what they want as the delimiter, no heuristics.
The "last occurrence" semantic is what makes it robust to documents where the make/model contains the same delimiter character. Toyota Prius - BR19 SRX — last - wins. Even Mercedes Benz E-Class - BR19 SRX — still last - wins.

The new method semantics:

User selects mapping method: Last Delimiter (new dropdown option)
User config: <label> || <delimiter> (reuses the existing two-field UI)
Behaviour: find the line/block containing the label, find the last occurrence of the delimiter on that line/block (after the label), return the trimmed text after it.

For your example with config Vehicle: || -:

Line Toyota Prius - BR19 SRX — last - is the only -, returns BR19 SRX
Line Mercedes E-Class - BR19 SRX — last - is the second one (after E-Class), returns BR19 SRX
Line BR19 SRX (no delimiter) — falls back to: return the whole value? Or empty? Probably empty, so the user knows the mapping didn't fit and can edit manually.

One thing this DOESN'T handle as well: the value is on the line after Vehicle: (which is the case in your sample — Vehicle: is on its own line, the value is on the next line). The existing label extraction handles that case, so I'd want the new method to behave the same way: find Vehicle:, look at the line/block containing the value, find last delimiter there.
A few clarifying questions before I implement:

Naming. "Last Delimiter" feels OK to me but a bit jargony. Other options: "After Last Character", "Tail After", "Keep After Last". Any preference?
Multi-character delimiters. Should the user be able to type -- or | as delimiters, or restrict to a single character? I'd say allow any string — same as labels.
Block vs line scope. Existing single_label captures everything until end-of-block (since V42 PDF mode). Do you want "last delimiter" to scan across the whole post-label block, or only the immediate line containing the value? The block approach is more permissive (handles wrapped content); the line approach is stricter.
No-delimiter fallback. If the user configured Vehicle: || - but a particular document has Vehicle: BR19 SRX with no -, what should happen? Empty (forces manual edit)? Or fall back to returning the whole post-label value? I'd lean toward empty — consistent with how Two Labels behaves when the end label is missing.

What do you think?You said: Can we implement a "Fixed Position + Label" method?Can we implement a "Fixed Position + Label" method? It will do exactly what it sounds like - look for a specific line, then take anything after the user specified label (behaves like a normal single label)7 MayClaude responded: Got it.Got it. Let me make sure I have the spec right before I implement:
Fixed Position + Label.

User configures two values: a line number and a label.
The method goes to that exact line in the source preview, then within that line (and only that line) does a single-label match for the configured label.
Returns whatever follows the label on that line.

For your sample:
1: Mobile:
2: 07936853974
3: Vehicle:
4: Toyota Prius - BR19 SRX
Config: line 4, label -. Result: BR19 SRX. The user pinpoints the exact line so the - ambiguity goes away.
Two design points to confirm:

The line number is the line in the source preview. Same as the existing Fixed Position method (which uses lines[start - 1:end] indexed from 1). Same numbering rules — including the V42 caveat about block-mode introducing blank lines, which means line numbers shift when block mode kicks in. The user works against what they actually see in the preview, which is correct.
Label-on-line semantic. Within that single line, look for the label and return the text after it. If the label appears, return the trimmed text after; if the line doesn't contain the label, return empty (forces manual edit). I won't fall through to other lines — that would defeat the point of pinpointing.

Two implementation questions:

UI: which two-field shape? The existing Two Labels method uses two side-by-side text entries. The existing Fixed Position uses a single small entry for the line number. I'd reuse the Two Labels UI shape: first field "Line" (small, numeric), second field "Label" (wider). Same layout, different labels above the fields. Sound good?
Storage format. The existing fixed_position stores "3" as config. Two Labels stores "start || end". For consistency I'd store this as "4 || -" (line, then ||, then label) — same delimiter, lets parse_two_label_config parse it. The new method code is fixed_position_label or similar.
Method label. The dropdown has "Single Label", "Two Labels", "Fixed Position", "Manual Input". I'll add "Fixed Position + Label". Sound good?

Confirm those points and I'll implement.You said: Confirm on all those.Confirm on all those.7 MayClaude responded: Good.Viewed 2 files, ran a commandViewed 2 files, ran a commandGood. Now look at the UI side — how the rules row builds the config widgets:Viewed 2 filesViewed 2 filesGood. So the two_labels branch already shows a two-field frame. The new method will reuse the exact same two_frame (reusing existing widgets), since the storage format "4 || -" parses through parse_two_label_config cleanly. UI-wise: when method is fixed_position_label, just show the two-frame.
Now look at save and load to make sure they handle the new method:Look at save_provider_mappingLook at save_provider_mappingGood — save_provider_mapping just needs the fixed_position_label method to follow the same || join as two_labels. Now load:Look at load_provider_into_editorLook at load_provider_into_editorGood — needs a new branch for fixed_position_label that splits on || like two_labels does.
Now I have everything. Implementing:
Edit 1: Add the new method to METHOD_CHOICES.Add Fixed Position + Label to method choicesAdd Fixed Position + Label to method choicesEdit 2: Add the extractor method on the engine, and dispatch in extract_by_rule.Edited 2 filesEdited 2 filesEdit 3: UI — show the two-field frame for the new method too. Update update_method_ui:Show two-field frame for fixed_position_label in update_method_uiShow two-field frame for fixed_position_label in update_method_uiEdit 4: save_provider_mapping joins both fields with ||:Save_provider_mapping joins fixed_position_label same way as two_labelsSave_provider_mapping joins fixed_position_label same way as two_labelsEdit 5: load_provider_into_editor splits same way:Load_provider_into_editor splits fixed_position_label config into two fieldsLoad_provider_into_editor splits fixed_position_label config into two fieldsOne thing that concerns me: parse_two_label_config runs both halves through clean_value, which strips colons. The line number 4 won't be touched, so that's fine. The label - will be preserved (since V52). Good.
Compile check:Syntax checkSyntax checkNow write tests covering the user's exact scenario plus edge cases:Created a file, ran a commandCreated a file, ran a commandOne failure — let me trace it. Config "2 || :", line 2 is "Line two: ABC", label :. parse_two_label_config runs both halves through clean_value, which strips colons. So the label : becomes "" → returns empty.
This is the same class of issue as V52 — clean_value is stripping a character the user intentionally typed. For labels, single-character punctuation labels are valid.
Two ways to fix:

Apply V52-style logic to colon stripping too: change clean_value to never strip colons either, only whitespace. But that would break the existing extraction pipeline that depends on Address: cleaning to Address for matching.
In parse_two_label_config, only call clean_value if the result wouldn't be empty — preserve the raw input when stripping would erase it.

Actually, option 2 is hacky. Let me think again.
The real fix: when we use parse_two_label_config for the new method, the second half is exactly the user's literal label and should not be cleaned at all (other than trailing whitespace from typing). The colon-stripping is "useful" only for the legacy two_labels extraction-time matching — and even there, V52 reasoning applies: the existing extract_between_labels and friends already tolerate trailing colons in document text via their own logic.
Let me check whether V52's logic generalises. We had strip(" :-\n") in V51, V52 removed -, leaving strip(" :\n"). Now this case shows the : strip is also a problem for user-typed labels. Same fix: don't strip those characters either.
But: removing : from clean_value is more impactful than removing - was. clean_value is called in many places (30+ uses), some of them on extracted document text where stripping a trailing : might be load-bearing. Let me audit.
Looking at the uses again:

Lines 378, 417, 421, 435, 522: cleaning DOCX/EML text lines — would not strip : from values that genuinely contain trailing colons. Probably fine; users would expect Some Title: to remain Some Title: if they're looking at it raw.
Line 1137 (raw_lines): just splits and cleans lines — same as above.
Line 1178, 1191, 1197, 1204, 1217, 1246, 1261, 1275, 1299, 1306, 1361, 1371, 1376, 1410: inside extract_after_label and friends — cleaning both labels and document text. These DO benefit from the : strip because they normalise label vs document for comparison.

So the : strip IS load-bearing in the matching path. I shouldn't remove it from clean_value.
Best fix: introduce a separate cleaning function specifically for user-typed labels in the new context. Or: handle the strip separately inside parse_two_label_config to preserve the user's literal intent for the label half.
Actually, even simpler: parse_two_label_config is used for user input parsing, not extraction. The existing two_labels usage (line 1487-1488 area) parses the config and feeds the labels into extract_between_labels, which itself runs more cleaning during matching. So if parse_two_label_config did NOT strip :/-, the existing two_labels behaviour would still work because the matching code does its own normalisation.
Let me make parse_two_label_config use a label-preserving strip:Re-look at parse_two_label_configRe-look at parse_two_label_configWait — but the existing two_labels logic relies on parse_two_label_config cleaning Address: → Address so the label can be compared after the matcher rstrips colons from the document line. Let me trace this more carefully.
Look at extract_between_labels:Look at extract_between_labels to see if it does its own normalisationLook at extract_between_labels to see if it does its own normalisationextract_between_labels regex is re.escape(start_label) + r"\s*:?" — it explicitly handles trailing optional colon AFTER the label. So the start label Address with :? matches both Address and Address: in the document. But what if the user typed Address: as the label? Then re.escape("Address:") → Address: and the regex becomes Address:\s*:? — looks for Address: followed by another optional :. That fails on Address: 123 because after Address: there's a space, then 123 — the :? is OK (matches nothing), then captures 123. So it actually still works.
Let me verify: would removing : stripping from parse_two_label_config break the existing two_labels behaviour? The pattern re.escape("Address:")\s*:?\s*(.*?)\s*(?=...) against Address: 123 End:

Match Address: literally. ✓
\s* matches the space. ✓
:? matches nothing. ✓
\s* matches nothing. ✓
(.*?) lazy-matches. ✓

Yes, it works either way. OK — parse_two_label_config can safely stop stripping :/-.
Better approach: do a label-preserving strip directly in parse_two_label_config:Edited a file, ran a commandEdited a file, ran a commandAll passing. Now run full regression sweep to confirm I haven't broken existing two_labels behaviour:Full regression sweep V42-V52Full regression sweep V42-V52One V52 test fails — and looking at it, the test was actually wrong. It asserted that parse_two_label_config("Address: || -") returns ("Address", "-") (with the : stripped). But under the new V53 semantics, the : is preserved → ("Address:", "-"). The new behaviour is correct (literal preservation) and the V52 test assumption was based on the old behaviour.
Let me update that V52 assertion:Edited a file, ran a commandEdited a file, ran a commandFinal regression sweep:Final regression sweepFinal regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV53 done. Summary:
The new method. Fixed Position + Label — fifth dropdown option. User specifies a 1-based line number and a label; extractor goes to that line, finds the label, returns text after it. If the label isn't found on that line, returns empty.
Implementation pieces:

METHOD_CHOICES gains ("fixed_position_label", "Fixed Position + Label").
New extract_by_fixed_position_label engine method — parses config via parse_two_label_config, looks up the line, does case-insensitive .find() for the label, returns trimmed text after.
extract_by_rule dispatches to the new method.
update_method_ui shows the same two-field frame as two_labels for this method.
save_provider_mapping and load_provider_into_editor handle the new method by going through the same ||-joined config path as two_labels.

Bonus fix in parse_two_label_config. It was running both halves through clean_value, which stripped leading/trailing colons. That made parse_two_label_config(":  || -") give ("", "-") — losing the user's literal : label. Now it only strips surrounding whitespace. The downstream matching code (extract_between_labels) does its own colon handling via \s*:? in the regex, so the existing Two Labels method works identically.
On your specific scenario. Configuring VRM with method Fixed Position + Label, line 4, label - against:
Line 1: Mobile:
Line 2: 07936853974
Line 3: Vehicle:
Line 4: Toyota Prius - BR19 SRX
returns BR19 SRX, which then goes through the existing VRM normalisation to BR19SRX for export. Confirmed end-to-end.
Tests. 21 new V53 cases covering the motivating scenario, edge cases (out-of-range, missing label, malformed config), case-insensitive matching, dispatch through extract_by_rule, save/load round-trip, and end-to-end via extract_fields. V42-V52 prior suites still pass (one V52 test had to be updated to reflect the new literal-preservation in parse_two_label_config — a documentation issue, not a behaviour regression).AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Can anything be done about these examples which I've attached?pdfpdfpdfpdfpdfpdfCan anything be done about these examples which I've attached? I'm fairly sure these are scanned documents, not true native PDFs, but I may be wrong?If this is going to be a waste of time we don't need to explore this.What I've noticed is that, using Chrome as a PDF viewer, I am able to highlight all the text in the document?7 MayClaude responded: Quick check on the actual nature of these PDFs before I commit to anything: Confirmed — every one of these PDFs has zero text content.Quick check on the actual nature of these PDFs before I commit to anything:Probe each PDF to determine if it's truly scanned or has a text layerProbe each PDF to determine if it's truly scanned or has a text layerConfirmed — every one of these PDFs has zero text content. They're each a single image-of-a-page (3507×2480, A4 at 300 DPI, PNG or JPEG depending on the source). PyMuPDF and pypdf both come back empty. Your importer is correctly extracting nothing because there's nothing in the PDF as PDF data to extract.
On the Chrome highlighting thing. Chrome has built-in OCR — it runs OCR on the rendered page when you select text. So you're not seeing real text from the PDF, you're seeing Chrome doing live OCR on the rendered image. The PDF data itself doesn't contain any of those characters; Chrome is recognising them visually. Other PDF viewers like Adobe Reader or Edge in pre-OCR mode would not let you select text from these.
So the question is: can the app do OCR on its end?
Yes, it absolutely can — and it's not a huge change. But there's a real trade-off here:
Option A: Add OCR via Tesseract. Tesseract is the industry-standard open source OCR engine. There's a Python wrapper (pytesseract) that's pure-Python on top of a Tesseract binary you'd need to bundle with the .exe.

What you get: image-only PDFs become readable. Field extraction would work on these documents using your existing Single Label / Two Labels / Fixed Position rules. The text Tesseract produces from a clean 300 DPI scan is usually 95-99% accurate — for these typewritten-looking letters with high contrast and standard fonts, probably 99%+. It would correctly read Vehicle Reg: SK112 ZYX, BMW 520D etc.
Cost #1: bundling. Tesseract is a ~30 MB binary that needs to ship alongside your .exe. PyInstaller can include it as a data file but the install jumps from a ~30 MB .exe to a ~60-80 MB .exe, and you'd need to source the Tesseract Windows binaries (UB Mannheim builds are the standard).
Cost #2: speed. OCR is slow compared to text extraction — figure 1-3 seconds per page on a typical machine for a 300 DPI image. Single-doc imports become noticeably slower; 10-doc batches go from "instant" to "several seconds." Mitigations: only OCR when text extraction returns empty (so existing PDFs aren't slowed at all), show a status bar indicator while OCR runs.
Cost #3: never 100% accurate. Even on clean scans, Tesseract occasionally misreads. O vs 0, I vs 1, 8 vs B, etc. For VRMs specifically that's painful — OW75 EHD could come out as 0W75 EHD. Users would need to verify after OCR-based imports. Doable but worth flagging.

Option B: Tell the user to OCR upstream and import the result. Adobe Acrobat, Microsoft Lens, Apple Preview, and many free tools can OCR a PDF and produce a text-layered PDF. The user runs it through one of those, then drops the result into your app. No app changes. Slower workflow per-doc but the app stays simple and fast for everyone else.
Option C: Just accept these documents need manual data entry. If the volume of image-only PDFs is small relative to the volume of native-text ones, the cost of Option A's complexity might not be worth it.
My honest read: Option A is realistic and worth doing if image-only PDFs are a recurring problem. The "only OCR when text extraction returns empty" guard means existing PDFs aren't penalised at all — you keep the V42 block-mode performance for the 95% case, and OCR is reserved as a fallback for these scanned ones. The 60-80 MB .exe size isn't unreasonable for a desktop tool. The accuracy concern is real but mitigated by users glancing at the extracted fields before exporting (which they should be doing anyway).
Three things I'd want to confirm before implementing:

Volume. Roughly how often do you receive image-only PDFs like these? Once a month, once a day, once an hour? That tells me whether OCR is worth the bundling cost.
Tesseract bundling acceptable? Adding Tesseract means your installer/distribution gets bigger and more complex. If you just hand .exe files around, this is fine. If you go through any kind of corporate IT approval, the larger .exe is an extra step.
OK with "OCR when text empty" being silent fallback? Or do you want a checkbox "Use OCR" that the user has to opt into? I'd default to silent fallback (consistent with the rest of the app's productivity-tool ethos) but happy to do explicit if you'd rather see when OCR is happening.

If you say "yes, do it, silent fallback" I'll implement Option A. If you say "let's not bother," that's a perfectly reasonable answer too — these are tricky documents and the app does its core job well on the 95% of normal PDFs.You said: It's not a common problem, it's an edge case, and I am open to exploring implementing it, but my greatest fear is that a lot of documents we have currently map…providers.jsonjsonIt's not a common problem, it's an edge case, and I am open to exploring implementing it, but my greatest fear is that a lot of documents we have currently mapped accurately and reliably also contain a few images (branding, logos, signatures), and I'm worried that changing how the importer works at all might mess all this up?7 MayClaude responded: Your concern is well-founded but the fix as I'm proposing would NOT trigger on those documents.Your concern is well-founded but the fix as I'm proposing would NOT trigger on those documents.
Let me re-confirm the mechanic. The importer does two things in sequence today:

extract_text — pulls text from the PDF via PyMuPDF block mode
Provider matching against that text

The "fallback to OCR" plan is to add a single if check at the end of step 1: if the extracted text is empty (or near-empty), and only then, run OCR on the page images. A document that has ANY real text — even one paragraph — never enters the OCR branch. So:

A PDF with text + a logo image: text extraction returns the text, OCR never runs, importer behaves identically to today. ✓
A PDF with text + multiple inline photos and a signature scan: same — text extraction wins, OCR never runs. ✓
A PDF that's literally just a photographed page (the MP_*.pdf samples): text extraction returns empty → OCR fires → returns text.

The trigger is "did we get any meaningful text from this PDF, yes or no" — which is a single boolean, not a quality assessment. It's not "did the text look complete enough" or "did some characters look weird" — anything else would risk false-triggering on partial-but-correct extractions.
A second protective step worth adding: define "empty" strictly. Not len(text) == 0 but something like len(text.strip()) < 20 — to handle a document that has only e.g. a header "Page 1" somehow leak through but is otherwise scanned. But not so loose that a short-but-valid email body triggers OCR. I'd pick a low threshold (20 chars or fewer = treat as scanned) and we can tune it.
Also worth knowing: this kind of "OCR fallback when no text" is a well-trodden pattern. Adobe Acrobat does it. Mozilla's PDF.js does it (when running in Chrome). It's not exotic.
Risks I'm being honest about:

Tesseract isn't installed: if the user is on a machine without Tesseract, the OCR call fails. Right thing to do: silently swallow the failure and return empty (same as today), with a friendly status bar message ("This PDF appears to be scanned. Install Tesseract for OCR support."). The app stays functional for the existing 95%.
Speed when OCR triggers: 1-3 seconds per page added when it fires. The user can tell because the status bar would show "Running OCR on scanned PDF..." — same friction as anyone running OCR anywhere else. Doesn't apply when OCR doesn't fire.
A document with mostly images and a tiny bit of incidental text: imagine a PDF with a single page containing one printed line "Confidential" and otherwise scanned content. Text extraction returns "Confidential" — passes the threshold check — OCR is never run — the user gets a useless extraction. This is a genuine edge case but I think it's actually fine: that document was already broken in the current importer, and not running OCR doesn't make it worse than today. Whereas if I made the threshold more aggressive (always-OCR-if-images-exist), I'd risk slowing down legitimate documents.

So the trade-off lands: strict empty-or-near-empty trigger → zero risk to existing documents, OCR only fires on the literally-image-only edge case. That's the right shape.
If you want extra peace of mind, I can also add a per-provider override: a checkbox in the provider editor "Skip OCR fallback for this provider". So if you ever discover a specific provider where OCR is misbehaving, you can disable it for that provider only. But honestly I don't think you'd need it — the trigger is so strict.
Just to confirm before I implement: should I look at your updated providers.json first to see if anything's changed in there that I should be aware of, then proceed with implementation? Or would you like more discussion first?You said: Are they all single image (the documents I uploaded)?Are they all single image (the documents I uploaded)? If every single one is single image then I'm open to adding a rule -> If text content is completely empty (not close, totally empty), and the document contains a single image -> then OCR7 MayClaude responded: Every one of them: 1 page, 1 image, 0 text characters.Confirm every uploaded sample is a single page with a single image and zero textConfirm every uploaded sample is a single page with a single image and zero textEvery one of them: 1 page, 1 image, 0 text characters. The pattern holds perfectly.
Your proposed rule is a strong one. Let me restate it precisely so we're aligned:

Trigger OCR fallback only when the PDF contains exactly zero text characters AND every page contains exactly one image (no more, no fewer).

This is even stricter than my earlier "low threshold" suggestion. Some implications worth being explicit about before I commit:
Pros (compared to my earlier "near empty" trigger).

Zero risk of false-triggering on a normal text PDF that happens to have logos/signatures (those have text → 0 chars condition fails immediately).
Zero risk of triggering on a multi-page PDF where some pages are scans and others are real text (the per-page text check would fail).
Easy to reason about: a user looking at a doc and asking "would this trigger OCR?" can answer with two clicks (does any text highlight, does it look like a single photographed page).

Cons / edge cases worth being explicit about.

If someone scans a multi-page document where they accidentally embedded a second small image per page (e.g. a photograph of a signature glued onto a scan), the rule wouldn't fire because images != 1. Probably fine — these are unusual.
If a scanner produces a PDF where each page has two image objects (e.g. background + foreground), the rule wouldn't fire. Some flatbed scanners do this. I can mitigate by being slightly looser: "exactly zero text AND at least one image per page, no upper bound." Looser but still strict — a real text PDF with logos won't trigger because text isn't zero.

Quick question on which interpretation you want:
(a) Exactly one image per page (your literal proposal). Strictest. Maps cleanly to your samples.
(b) At least one image per page (slightly looser). Same protection against text PDFs (the zero-text gate handles those), but tolerates multi-image scanned PDFs.
Both are safe in practice — either guarantees no behaviour change on your existing text-bearing documents. (b) is just slightly more useful for unusual scanner outputs. I'd recommend (b) — same protection for the cases you care about, slightly more capable on the edge cases. But (a) is also fine if you'd rather be conservative.
Also: I noticed you uploaded a new providers.json but I didn't read it yet because you didn't reference it. Want me to look at it first, or is it just here for reference and we should proceed with OCR?You said: The thing is, we sometimes get documents like the 2 attached which are images of a vehicle and the client has just decided to send them as a document rather th…__Images.docxdocxpdfThe thing is, we sometimes get documents like the 2 attached which are images of a vehicle and the client has just decided to send them as a document rather than separate files. We don't want the importer to trigger an OCR for these because that's a huge waste of efficiency7 MayClaude responded: Good news: this photos PDF wouldn't trigger OCR under your proposed rule anyway.Inspect the photos PDFInspect the photos PDFGood news: this photos PDF wouldn't trigger OCR under your proposed rule anyway. It has ~1900 characters of real text already (the captions like AudaImage MAR15131-10.jpg and the page headers Our Ref: / Your Ref: EHR96041 / Page 1). Zero-text gate already filters it out.
But your concern points at the more general case: a client could send a "photos document" where the captions/headers are baked into the image rather than typed text. Those would have text_chars=0 and trigger OCR. Then we waste 30+ seconds running OCR on photos that won't yield anything mappable anyway.
So your refinement to my proposal — zero text AND single image per page — is a tighter guard. Let me think through whether that's really enough.
A "single image per page" means: one image object on each page. Photo dump documents (like the one you uploaded) have 5-7 images per page. Scanned letters (like the MP samples) have exactly 1 image per page. Solid distinguisher.
But what about edge cases I should sanity-check:

A scanned letter that the scanner saved as multiple image strips per page (some scanners stripe their output)?
A scanned letter where the scanner added a separator image (page divider, watermark) on top?
A photo dump with one big collage image per page (rather than 6 separate images)?

Let me see how strict you actually want this. (a) "all pages must have exactly 1 image" or (b) "every page has exactly 1 image, AND that's the only content type the page contains" (i.e. no other vector graphics, drawings, etc.)?
I think (a) is the right level of strict. We want to catch the common case of "scanned-to-PDF letter" reliably, and we already have the zero-text guard upstream. Adding more conditions on top of "1 image per page" would be over-engineering.
Final proposed rule, locked down:

OCR fires if and only if (1) the document has zero text characters across all pages, AND (2) every page contains exactly one image. Any deviation = no OCR.

This is provably safe for your concern:

Photo dump PDFs (multiple images per page, with or without text captions): condition 2 fails → no OCR ✓
Photo dump PDFs where every page is a single big collage image, but there's caption text: condition 1 fails → no OCR ✓
Photo dump DOCX (the __Images.docx): the text extraction path for DOCX is separate from PDFs. I'd only add OCR to the PDF path, not DOCX. DOCX images can be exported via the existing image export feature without OCR being relevant ✓
Normal text PDF with logos: condition 1 fails → no OCR ✓
Scanned letter (the MP samples): both conditions hit → OCR fires ✓

Three things I want to confirm before implementing:

PDF only, not DOCX. OCR will be added to the PDF text-extraction path only. DOCX, DOC, EML, MSG paths are untouched. Confirm?
Tesseract bundling. PyInstaller can bundle tesseract.exe and its tessdata (the trained models for English) as data files. That adds ~30-40 MB to the .exe. Alternative: detect Tesseract on the user's PATH at runtime and only use it if present (no bundling), with a friendly status message if it's missing. The bundled approach "just works" out of the box; the unbundled approach keeps the .exe small and only the users who actually receive scanned PDFs need to install Tesseract themselves. Which would you prefer?
Status feedback. When OCR runs, do we want any indication in the status bar (e.g. "Running OCR on scanned PDF…"), or fully silent? OCR takes 1-3 seconds per page so silence on a multi-page scan would feel like a hang. I'd lean toward a brief status message during the operation. Confirm?

You also uploaded a fresh providers.json which I haven't analysed. Do you want me to check it for anything specific or is it just here for reference while we discuss?You said: PDF only, bundle everything (we want a completely independent portable app), fully silent (this is a productivity tool).PDF only, bundle everything (we want a completely independent portable app), fully silent (this is a productivity tool).7 MayClaude responded: Got it.Got it. Plan:

Add pytesseract to requirements.txt.
Add a Tesseract detection helper that finds the bundled tesseract.exe when running from PyInstaller.
In _extract_pdf_text_via_pymupdf, after collecting block-mode text, if total text is empty AND every page contains exactly one image, run OCR on each page's rendered image.
PyInstaller spec needs --add-data "tesseract;tesseract" (or the equivalent .spec modification). I'll document this in the README so when you build the .exe you know what to bundle.

Important note: I can't actually install/bundle Tesseract on this Linux box and ship it to your Windows machine. What I'll do:

Write the code so it works whether Tesseract is bundled, installed system-wide, or not present at all.
Test the OCR logic on this Linux box (where I can install tesseract-ocr via apt) to verify the code path works end-to-end.
Give you clear instructions for bundling on the Windows build machine.

Let me first verify Tesseract works at all on these PDFs:Ran 2 commandsRan 2 commandsOCR quality is excellent on these — Vehicle Reg: FG21 DGV, Toyota Prius (Private Hire), Our Ref: RA6458842, Our Client: Mr Mohamed Patel, Date of Accident: 23/04/2026 — all crisp. Even the harder one with the letterhead: OW75 EHD came through correctly. Per-page timing 2.8-3.2s on this Linux box at 300 DPI; will be similar on Windows.
Implement the changes:Add pytesseract optional import alongside fitz/extract_msgAdd pytesseract optional import alongside fitz/extract_msgNow add the Tesseract bundling helper. The best practice with PyInstaller is to bundle tesseract.exe and the tessdata folder, then set pytesseract.pytesseract.tesseract_cmd to the bundled path at startup. PyInstaller exposes sys._MEIPASS for the bundle root.Ran a command, viewed a fileRan a command, viewed a fileAlready a resource_path helper that knows about PyInstaller. Add a Tesseract configuration helper next to it:Add configure_bundled_tesseract helperAdd configure_bundled_tesseract helperWire it into app startup so it runs once at launch. Find the main entry point:Ran 2 commands, viewed 2 files, edited a fileRan 2 commands, viewed 2 files, edited a fileNow the core change: the OCR fallback inside _extract_pdf_text_via_pymupdf. The logic will be:

Run the existing block-mode extraction.
If the resulting total text is exactly empty AND every page has exactly one image → run OCR.
OCR fires per page: render at 300 DPI, run pytesseract, collect text.
If pytesseract is unavailable, return empty (same as today).
Wrap OCR in try/except so any failure quietly returns the empty-text result rather than crashing the import.
Look at current PyMuPDF extractionLook at current PyMuPDF extractionOK so currently this method opens doc, iterates pages, closes doc. To do the OCR check + OCR fallback I need to know per-page image counts. The cleanest approach is to refactor: do the text extraction first, hold the result. If empty, open the doc again to check image counts and run OCR. Or — keep the doc open, gather image counts in the same loop.
I'll do the latter — gather image counts during the same iteration, then decide after the loop. That keeps it efficient and the document open.Add OCR fallback when entire PDF has zero text and every page has exactly one imageAdd OCR fallback when entire PDF has zero text and every page has exactly one imageI'm using io.BytesIO, let me make sure io is imported:Check io is importedCheck io is importedNot imported. Let me check how other places in the file handle bytes-to-PIL — there might already be an io import I missed:Ran a command, edited a fileRan a command, edited a fileCompile check:Syntax checkSyntax checkNow end-to-end test on the actual files:Created a file, ran a commandCreated a file, ran a commandEvery single one of the user's expected fields was correctly OCR'd from the scanned PDFs, including the tricky JAV 8R private plate, all the Mr Ali Ahmed Qurban-style claimant names, all the RA6458832-style refs. Photo-dump PDF correctly skipped (0.02s, no OCR). EVA report unchanged.
Per-page OCR time: 1.7-1.8s on this Linux box. Should be similar on Windows (Tesseract is CPU-bound, not platform-specific).
Run the full regression sweep:Full regression sweep V42-V53Full regression sweep V42-V53All clean. Now requirements.txt and README — and crucially, the README needs to explain the Windows bundling step since I can't actually do that part for you:Ran 3 commands, read a fileRan 3 commands, read a fileV54 is built. Summary:
The functional change. OCR fallback in PDF text extraction. Fires only when the entire document has zero text characters AND every page has exactly one image. Photo-dump PDFs (multiple images per page) and normal text PDFs (with logos or signatures) are completely unaffected.
Implementation.

pytesseract and PIL.Image added as optional imports (alongside fitz and extract_msg).
New configure_bundled_tesseract() helper points pytesseract at a Tesseract binary bundled in tesseract/tesseract.exe next to the .exe (or in PyInstaller's _MEIPASS at runtime). Sets TESSDATA_PREFIX for the trained-data folder. Called once at startup in main().
_extract_pdf_text_via_pymupdf now tracks per-page image counts during the existing block-mode pass. After the loop, if combined == "" AND every page had exactly 1 image AND pytesseract+PIL are available, it loops the pages again to render at 300 DPI and OCR them. Failures are caught silently and fall back to empty text.
A note "Read PDF using OCR fallback (no text layer)." appears in the import notes when OCR fires (visible to user via existing status flow but not popup-y).

Tested. All six of your scanned PDFs produce clean OCR output with the right VRMs, refs, names, addresses extracted. The photo-dump PDF is correctly skipped (0.02s extraction). The EVA report (text PDF with logos and a vehicle table) is unchanged. All 12 prior test suites pass.
The bundling caveat I want to be very clear about. I implemented and tested the OCR logic on Linux. The code is portable, but the actual .exe build needs Tesseract bundled. The README now includes step-by-step Windows bundling instructions:

Install Tesseract from the UB Mannheim Windows builds.
Copy tesseract.exe, its DLLs, and tessdata/eng.traineddata into a tesseract/ folder next to app.py.
Add --add-data "tesseract;tesseract" to your PyInstaller command.

The .exe goes from ~30 MB to ~60-80 MB, ships everywhere, no separate Tesseract install needed by users. If you ever build the .exe without bundling Tesseract, the OCR fallback silently does nothing — same behaviour as before V54 — so users who never get scanned PDFs see zero impact.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Am I still compiling by running:Am I still compiling by running:pip install -r requirements.txtthenpyinstaller --noconsole --onefile --icon=ce_document_mapper.ico --add-data "ce_document_mapper.ico;." --collect-all PIL app.py7 MayClaude responded: Almost — your old command will still produce a working .Almost — your old command will still produce a working .exe, but it won't include Tesseract, so the OCR fallback won't fire on scanned PDFs (it'll silently no-op, behaving exactly like the pre-V54 build).
To actually get OCR working in the .exe, you need to bundle the tesseract/ folder you set up next to app.py. The new full command:
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
Two pieces:

One-time setup in your project folder before building. Next to app.py, create a tesseract/ folder and populate it from your Windows Tesseract install:

   project/
     app.py
     ce_document_mapper.ico
     tesseract/
       tesseract.exe
       tessdata/
         eng.traineddata
       (plus any DLLs from the Tesseract install — copy the whole bin folder is easiest)
You can install Tesseract from UB Mannheim's Windows builds (free, the standard distribution). It defaults to C:\Program Files\Tesseract-OCR\. From there, copy tesseract.exe, every .dll next to it, and the tessdata/eng.traineddata file into your project's tesseract/ folder. You don't need any other language data files; deleting them keeps the bundle smaller.

pip install -r requirements.txt unchanged — the new pytesseract line gets installed automatically.

After this, your .exe jumps from ~30 MB to roughly 60-80 MB (Tesseract + DLLs + eng.traineddata is the bulk of it). One bundle, runs anywhere, no separate Tesseract install needed by users.
If you'd rather skip the Tesseract bundling for now and ship V54 without OCR working, your original command is fine as-is — V54 will still ship all the other improvements, the OCR fallback just won't fire. Whenever you want to add OCR later, the only change is the extra --add-data line plus populating the tesseract/ folder. The Python code itself doesn't need to change.You said: There is a huge amount of files in my C:\Program Files\Tesseract-OCR folder, what is needed to be copied into the tesseract subfolder next to app.Directory of C:\Program Files\Tesseract-OCR

07/05/2026  10:30    <DIR>          .
07/05/2026  10:30    <DIR>          ..
11/11/2024  08:25            17,823 ambiguous_words.1.html
11/11/2024  08:25            59,528 ambiguous_words.exe
11/11/2024  08:25            19,529 classifier_tester.1.hpastedThere is a huge amount of files in my C:\Program Files\Tesseract-OCR folder, what is needed to be copied into the tesseract subfolder next to app.py?7 MayClaude responded: Right, let me filter the noise.Right, let me filter the noise. The vast majority of those files are training/utility binaries you don't need at runtime — only the things that tesseract.exe actively loads or links against.
Here's what to copy:
1. The main binary (1 file):

tesseract.exe

2. All lib*.dll and zlib1.dll files (52 DLLs). Tesseract dynamically links to a chunk of these at runtime. Determining exactly which ones is fiddly and version-dependent — easier and safer to just copy them all. They total around 60 MB and are the bulk of the bundle either way.
The easiest copy command to run from C:\Program Files\Tesseract-OCR (Command Prompt, with your project folder as the destination):
copy tesseract.exe "C:\path\to\your\project\tesseract\"
copy *.dll "C:\path\to\your\project\tesseract\"
3. The English language data (1 file):

tessdata\eng.traineddata

The tessdata folder contains many language models — you only need English. Create the tessdata subfolder and copy just eng.traineddata into it:
mkdir "C:\path\to\your\project\tesseract\tessdata"
copy "tessdata\eng.traineddata" "C:\path\to\your\project\tesseract\tessdata\"
What you should NOT copy (saves ~10 MB and removes clutter):

All the .html files (manual pages — runtime doesn't read these)
All the training/utility .exe files (ambiguous_words.exe, classifier_tester.exe, cntraining.exe, combine_lang_model.exe, combine_tessdata.exe, dawg2wordlist.exe, lstmeval.exe, lstmtraining.exe, merge_unicharsets.exe, mftraining.exe, set_unicharset_properties.exe, shapeclustering.exe, text2image.exe, unicharset_extractor.exe, winpath.exe, wordlist2dawg.exe) — these are command-line tools used to train new models, not to run OCR
tesseract-uninstall.exe
The doc/ folder
The osd.traineddata and any non-English language files in tessdata/

Final structure should look like:
project\
  app.py
  ce_document_mapper.ico
  tesseract\
    tesseract.exe
    libtesseract-5.dll
    libleptonica-6.dll
    ... (all the other lib*.dll files)
    zlib1.dll
    tessdata\
      eng.traineddata
Then your build command (unchanged from what I gave you):
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
Quick sanity check after build: drop one of those scanned MP_PDF letters into the .exe. If OCR is working you'll see the VRM and other fields auto-populate after a 1-3 second pause. If they don't appear, OCR isn't engaging — most likely cause is a missing DLL, in which case copy any DLLs you skipped.You said: I don't care too much about clutter, can I copy it all?I don't care too much about clutter, can I copy it all?7 MayClaude responded: Yes, absolutely.Yes, absolutely. Copy everything. The build will work identically — you'll just include about 7-8 MB of files that won't be used at runtime (HTML manual pages, training utilities, the uninstaller, the doc folder). The bundled .exe will be roughly 7-8 MB larger than the optimised version, which is nothing to worry about.
Two practical notes:
On the tessdata/ folder. If you don't care about clutter you can copy the whole thing — but it contains a tessdata/configs/ subfolder and possibly other language .traineddata files (osd.traineddata is the only other one you might encounter; full installs sometimes have more). Each unused language file is ~10-15 MB, so if you've got several of those it adds up faster than the rest. Up to you whether that crosses your "I don't care" threshold. The English eng.traineddata is the only one Tesseract will actually load given how the app is configured.
On the simplest copy command. Easiest way is just to copy the entire Tesseract install folder wholesale. From Command Prompt:
xcopy "C:\Program Files\Tesseract-OCR" "C:\path\to\your\project\tesseract" /E /I
That gets you everything — DLLs, training tools, HTML docs, all language data — in one go. Then run the same PyInstaller command:
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
The result will work, just slightly chunkier than the minimal bundle. Final .exe probably lands in the 80-100 MB range rather than 60-80 MB.You said: Let's add another mapping method for edge cases.Let's add another mapping method for edge cases. Let's add a method named "Single Label +/-". This mapping method will have 2 fields in the UI, the first field is a regex for a label, and the second field is a + or - number of lines. The purpose of this is for OCR edge cases when there's no consistent fixed position and all other methods are inadequate, but the OCR quality is good, therefore you can find a label and add a + or - line count to look X number of lines above or below the line the regex identified. The output will be whatever is on that line.
Let's do a UI tweak while we're at it. Let's remove the big explanation of the mapping methods completely "Single Label = one label, then..." etc.7 MayClaude responded: Two clarifying questions before I dive in:Two clarifying questions before I dive in:
Q1 — How should the second field validate? The user types -2 to look 2 lines above, +1 for 1 below, 0 for the same line. Should I accept:
(a) Strict: only -N, +N, 0 (with optional whitespace). 2 without a sign is rejected.
(b) Lenient: bare 2 treated as +2. Most flexible, but 2 looks like a Fixed Position config, which could confuse users.
I'd recommend (a) strict — the +/- sign makes the offset's intent obvious and matches the method's name "Single Label +/-". A bare number with no sign would silently behave like +, which is fine in practice but reads as a typo.
Q2 — What does the first field actually contain? You called it "a regex for a label," but the existing Single Label takes a literal string (e.g. Vehicle Reg:), not a regex. Two options:
(a) It's a literal string just like Single Label. Simpler, consistent with other methods. Internally I do a case-insensitive substring match per line.
(b) It's an actual Python regex. More powerful — user can write Vehicle\s*Reg(?:istration)?:? to match variants. But: regexes need escaping (Our Ref. would match more than intended because of the .), and a malformed regex shouldn't crash the import.
Either is implementable. (a) matches user expectations from every other method, (b) is more powerful for OCR edge cases (which is the whole point of this method). My instinct is (b) regex since the use case is messy OCR output where labels might appear with extra whitespace, optional colons, or slight character variations — exactly what regex is good at. I'd validate the regex at extraction time and silently return empty if it's malformed (no crash, no popup).
Confirm those two and the third behaviour question:
Q3 — Out-of-bounds offset. If the label is found on line 5 and the offset is -10, line -5 doesn't exist. What should happen? Same as I'd handle other "out of range" cases — return empty. Same with +N past the end of the document. Confirm?
Also — on the UI tweak: the explanation block to remove, are you referring to the help text under the rules table that explains each method? I'll find and remove it. Want me to leave the column header row ("Field / Method / Configuration") intact, or strip the column headers too? My read is you want to keep the column headers and remove only the longer paragraph-style explanation of what each method does. Confirm.You said: Q1) strict + or - only.Q1) strict + or - only.Q2) I'm not really sure, is there any material difference? I think literal string sounds right.Q3) Go as far as possible, so if -10 on line 5, use line 1Keep the column header row just remove the help text explaining each method.7 MayClaude responded: On Q2 ("any material difference?On Q2 ("any material difference?") — yes, but a small one. With literal string (option a), the user types Vehicle Reg: and we look for that exact text (case-insensitive). With regex (option b), the user types Vehicle\s*Reg:? and we'd match flexible variants. For your specific OCR-edge-case use, literal is genuinely fine — OCR output from a clean scan is consistent enough that a literal match works. Regex would only matter if OCR was producing wildly different versions of the same label across documents, which it generally doesn't.
Going with literal string — same semantics as Single Label, just with the offset twist. Easier mental model for the user, no edge cases around malformed regex.
On Q3 (clamp to bounds): I'll clamp both ways — -10 from line 5 → line 1, +99 from last page → last line. Always returns a line, only returns empty if the label itself isn't found.
Implementing:Find spots to mirrorFind spots to mirrorMost of the wiring will mirror fixed_position_label exactly — same two-field UI, same ||-joined storage. Edits in turn.
Edit 1: Add to METHOD_CHOICES.Add Single Label +/- method choiceAdd Single Label +/- method choiceEdit 2: Add the engine method and dispatch:Viewed a file, edited a fileViewed a file, edited a fileEdit 3: Dispatch in extract_by_rule:Dispatch single_label_offsetDispatch single_label_offsetEdit 4: UI — show two-field frame for the new method:Show two-field frame for single_label_offsetShow two-field frame for single_label_offsetEdit 5: load handler:Load_provider_into_editor handles single_label_offsetLoad_provider_into_editor handles single_label_offsetEdit 6: save handler:Save_provider_mapping handles single_label_offsetSave_provider_mapping handles single_label_offsetNow find and remove the help/explanation text. Let me find it:Ran a command, viewed a fileRan a command, viewed a fileRemove that help-text Label and bump row:Remove the method-explanation help text blockRemove the method-explanation help text blockCompile check:Syntax checkSyntax checkNow write tests:Created a file, ran a commandCreated a file, ran a commandAll 28 cases pass. Run the full regression sweep:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV55 done.
The new method: Single Label +/-. Sixth dropdown option. User specifies a literal label (case-insensitive) and a signed-integer offset (+N, -N, +0/-0). Extractor finds the first line containing the label, then returns the line offset lines above (negative) or below (positive). Out-of-bounds offsets clamp to document edges. Strict requirement on the +/- sign — bare 2 is rejected since it would silently behave like +2 and obscure the user's intent.
Implementation mirrors fixed_position_label's wiring throughout — same two-field UI, same ||-joined storage format. The new engine method extract_by_single_label_offset does the work; extract_by_rule dispatches to it; update_method_ui, save_provider_mapping, and load_provider_into_editor all extend their existing ("two_labels", "fixed_position_label") lists to include the new method code.
UI tweak. Removed the four-method explanation paragraph that sat above the rules table. Column headers (Field / Method / Config) stay. With six mapping methods now, the paragraph was getting unwieldy.
Tests. 28 V55 cases covering positive/negative/zero offsets, bounds clamping, strict signed-integer validation, missing label, case-insensitive label match, first-occurrence anchoring, dispatch through extract_by_rule, save/load round-trip, and verification that the help-text block is gone from the actual built UI. All V42-V54 prior suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: We need to separate the preset names from the Work Provider field.We need to separate the preset names from the Work Provider field. The reason for this is because some work providers have multiple document formats and so users are listing them in the Saved Provider list like "FW (Garage)" "FW (Solicitor)", but our output JSON needs a strict format for the work provider field so the JSON gets read correctly by our management software that we are importing it into. The simplest solution is separate the "preset names" from the Work Provider field itself by adding a Work Provider as a field that only accepts manual input (no mapping methods). This should be at the very top in the UI (just below the column header row). We should also reorder the UI on the right panel to match the detected data panel on the left as it causes unnecessary confusion when the fields are listed in a different order.7 MayClaude responded: Before I implement, let me work through this to make sure I have the full picture, because there are a few moving parts:Before I implement, let me work through this to make sure I have the full picture, because there are a few moving parts:
Understanding the change. Today, the Work Provider field on the left (Detected Fields) panel is auto-filled with the preset name whenever a provider is detected. So if you have presets FW (Garage) and FW (Solicitor), the JSON export ends up with "work_provider": "FW (Garage)" — but your management software needs just "FW".
You want to break that link. Each preset should have its own user-typeable Work Provider value (e.g. both FW (Garage) and FW (Solicitor) would have Work Provider = FW), independent of the preset name. The preset name is then purely for organising in the dropdown.
On the UI side, you want the Work Provider field added to the rules table on the right (just below the column headers) as a manual-input-only row — no Method dropdown, just a single text entry. Currently, work_provider is intentionally excluded from the NON_PROVIDER_FIELDS list so it never appears in the rules table. The change inverts that: it gets a row, but a special one with no method choice.
Plus the right-panel reorder. Match the left panel's order: Work Provider, VRM, Vehicle Model, Claimant Name, Reference, Incident Date, Instruction Date, Inspection Date, Inspection Address, Accident Circumstances, VAT Status, Mileage, Mileage Unit. Right now the right panel uses a different order (let me check what that is — I think it differs because Work Provider was excluded from the rules table entirely).
A few subtle decisions:

Storage in providers.json. A new work_provider value alongside field_rules. Either as a top-level key on the provider dict (e.g. "work_provider": "FW") or inside field_rules like the other fields. I'd put it inside field_rules for consistency, with method manual_input and config = the user's text. That way the existing extract_fields flow already handles it (it has a manual_input branch that returns config.strip()). Plus the schema migration logic for legacy field_rules already runs in normalize_provider_config.
Migration for existing presets. Anyone who's already been using the app has presets where work_provider was implicit (= preset name). The first time they open such a preset after this change, the work_provider field rule will be missing or blank. Two options:
(a) Default to the preset name on first load. Preserves existing behaviour for old presets — they'd see "FW (Garage)" prefilled as the Work Provider, can edit if they want.
(b) Default to blank, force user to fill it in. Cleaner conceptually but breaks existing presets silently — until the user notices their JSON now has empty work_provider.
I'd recommend (a) — falls back to the preset name when no explicit work_provider is set in the rule, then once the user edits and saves, that explicit value is written and the preset-name fallback never fires again for that preset.
What happens at detection time? Currently, when a document matches a provider, extract_fields populates work_provider with the preset name. With this change, it should populate it with the preset's work_provider field rule (defaulting to the preset name if the rule's config is blank, per #2). The detected field on the left panel then shows the user's chosen work_provider value, not the preset name.
Work Provider colour coding. Currently Work Provider uses #ffffff (white, no colour) on the left panel — explicitly excluded from the colour map (your handover said "Work Provider is not colour coded"). I'll keep that consistent on the right panel too — Work Provider's row label gets white background.
Detected provider name vs work provider value. The left panel shows two pieces: a "Detected: FW (Garage)" status above the fields, and the Work Provider field within them. The first should still show the preset name (so users know which preset matched). The second is now user-defined.

Three questions before I implement:
Q1: Confirm the migration: blank work_provider config → fall back to preset name (so existing behaviour is preserved). Right?
Q2: For the new Work Provider row in the rules table — should it have any "colour" treatment? Right now the field's column-0 label uses white (#ffffff per existing rule). Same on the right? Just confirming.
Q3: When a user creates a brand new preset, should the Work Provider field default to empty in the editor (forcing them to type something) or default to the preset name they're typing into the "Provider Name" field? My instinct: default to empty. Saves a corner case where a user creates a preset called "FW (Garage)" and assumes the Work Provider was auto-filled but actually it was empty when saved.You said: Q1) Force user to fill.Q1) Force user to fill.
Q2) Give it a white label for consistency, but it doesn't need any highlighting in the source text panel.
Q3) Force user to fill.One other question - is it possible for me to put my up to date providers.json in the app folder at the time of compiling so that, if providers.json is missing from the documents folder when the app launches (every time), it creates a new one that's the same as the one it was compiled with?7 MayClaude responded: Got it on all three.Got it on all three. Default-to-blank means I'll need to handle the case where a user opens an old preset that doesn't have a work_provider value yet — left panel just shows blank for that field until they edit the preset and add one. They'll notice, which is the point.
On the providers.json bundling question: yes, absolutely possible. Currently the app falls back to a hardcoded DEFAULT_CONFIG Python dictionary (with the RJS preset) when no providers.json exists in Documents. I can change that to:

Look for a bundled providers.json next to the .exe (in the PyInstaller bundle, accessible via the existing resource_path helper).
If found, copy it to Documents\CE Document Mapper\providers.json on first run.
If not found, fall back to the existing hardcoded DEFAULT_CONFIG.

Build flow becomes: drop your up-to-date providers.json next to app.py before building, add --add-data "providers.json;." to the PyInstaller command, and the .exe will seed new installs with that exact file. Users who already have a providers.json get nothing changed (the seed only runs when no file exists in Documents).
I'll bundle this with V56 along with the Work Provider change. Implementing now.Ran a command, viewed a fileRan a command, viewed a fileLook at the right-panel rules-table ordering — that's ordered_fields at line ~2870:Ran a command, viewed a fileRan a command, viewed a fileConfirmed: right panel currently does VRM through Inspection Address, then Mileage / Mileage Unit, then Accident Circumstances / VAT Status. Left panel has the canonical DEFAULT_FIELDS order with Accident Circumstances and VAT Status BEFORE Mileage. So I need to reorder the right panel to match the left panel.
Now let me look at extract_fields to see where work_provider is set:Look at extract_fields work_provider handlingLook at extract_fields work_provider handlingNow look at normalize_provider_config:Normalize_provider_configNormalize_provider_configOK good. I have enough context. Let me now implement.
The architectural choice. I'll keep NON_PROVIDER_FIELDS as-is (excluding work_provider) for code paths like extraction that already work that way, and instead use a separate concept for the rules-table UI: RULES_TABLE_FIELDS which includes work_provider. That way I minimise blast radius — the existing extraction loop (for field_name in NON_PROVIDER_FIELDS) stays untouched, and the new field gets handled with its own special path.
Let me sketch the plan:

Add work_provider to the rules table UI (just below the column headers), with a single-entry input only — no method dropdown.
Storage: put it in field_rules["work_provider"] = {"method": "manual_input", "config": "FW"}. Reuses existing schema, no new top-level keys.
extract_fields reads field_rules["work_provider"]["config"] if present, otherwise leaves work_provider blank (Q1: force user to fill).
save_provider_mapping writes the new field rule.
load_provider_into_editor reads it into a new mapping_config_vars["work_provider"] StringVar.
Reorder the right-panel ordered_fields list to match DEFAULT_FIELDS.
Bundling: have load_or_create_config look for a bundled providers.json first via resource_path.

Implementation:
Edit 1: Update extract_fields to read work_provider from the rule, default to blank:Extract_fields: work_provider comes from the rule's config, not the preset nameExtract_fields: work_provider comes from the rule's config, not the preset nameEdit 2: Make sure the work_provider field rule is preserved through normalize_provider_config. Currently it loops NON_PROVIDER_FIELDS (excluding work_provider), so the rule passes through unchanged, but I should ensure a default exists when missing:Ensure work_provider field rule defaults exist after normalisationEnsure work_provider field rule defaults exist after normalisationEdit 3: Add a StringVar for work_provider mapping in the App:Viewed a file, edited a fileViewed a file, edited a fileEdit 4: Add the work_provider row in the rules table (right panel) — and reorder the rest to match DEFAULT_FIELDS. The work_provider row gets a single entry box, no dropdown:Viewed a file, edited a fileViewed a file, edited a fileEdit 5: save_provider_mapping writes the work_provider rule:Viewed 2 files, ran a commandViewed 2 files, ran a commandAdding the work_provider rule to the saved field_rules dict, and the provider_name fallback shouldn't pull from the editor's work_provider entry anymore (it would now just be the user-provided value, which doesn't make a good default for a new preset's name). I'll keep the fallback as a one-time convenience for users who type in Work Provider but forget to type the Provider Name:Save_provider_mapping writes work_provider rule and uses editor field as fallback nameSave_provider_mapping writes work_provider rule and uses editor field as fallback nameEdit 6: load_provider_into_editor reads the work_provider rule into the entry:Viewed a file, edited a fileViewed a file, edited a fileSame for clear_provider_editor:Clear_provider_editor also clears work_providerClear_provider_editor also clears work_providerCaps mistake on False. Let me retry:Viewed a file, edited a fileViewed a file, edited a fileEdit 7: The bundled providers.json seeding. Update load_or_create_config:Viewed a file, edited a fileViewed a file, edited a fileWait — there's a flow bug. After seeding from the bundle, the code falls through to data = json.loads(path.read_text(...)) which re-reads the file we just wrote. That's actually fine and gets us through normalisation, but it's a tiny waste. Let me restructure:Tidy up the bundled-seed flow so it always normalises through the same pathTidy up the bundled-seed flow so it always normalises through the same pathCompile:Syntax checkSyntax checkNow let me think about something I may have overlooked. The user asked for the rules-table reorder to "match the detected data panel on the left" — including Mileage at the end. But the left-panel order has Mileage and Mileage Unit last. I reordered the right panel correctly to match. Good.
But there's another implication of putting Work Provider in the rules table: the field's old left-panel highlight color (#ffffff) is unchanged, but for the right panel I just used #ffffff directly in my new code. The existing right-panel code uses self.field_highlight_colors.get(key, "#ffffff") which would also resolve to #ffffff for work_provider. Consistent.
Tests:Created a file, ran a commandCreated a file, ran a commandBug: rules_frame is a ttk.Frame which doesn't expose the background cget option. Use the standard tk.Label with no bg, or bg="":Edited a file, ran a commandEdited a file, ran a commandAll V56 tests pass. Run the full regression sweep:Full regression sweep V42-V55Full regression sweep V42-V55All clean. Update the README and finalise. The README needs a careful update because the build instructions now have one more --add-data line:Ran a command, read a fileRan a command, read a fileV56 done. Two changes shipped together.
Change 1: Work Provider separated from preset name.
The Work Provider value is now a field on the preset (manual-input rule), not the preset name. So the user can have multiple presets like FW (Garage) and FW (Solicitor) whose JSON output both correctly read "work_provider": "FW" for their downstream management software.
UI-wise: a new Work Provider row sits at the top of the rules table. Manual-input only — no method dropdown, just a text entry. The right panel is now ordered to match the left (Work Provider, VRM, Vehicle Model, ..., Mileage Unit). Mileage and Mileage Unit moved to the bottom on the right panel to align with the left.
Behaviour for existing presets: any preset without an explicit work_provider rule gets a default blank one injected on load. A document matching such a preset will show blank in Work Provider — forcing the user to open the preset and fill it in. JSON export remains gated on Work Provider being non-empty (per V45), so this surfaces silently as "JSON didn't export, why?" when the user notices and goes to fix it. That's intentional.
Change 2: Bundled providers.json seeding.
load_or_create_config now checks for a bundled providers.json (via resource_path) on first launch when the user has no file in Documents. If found, that file is used to seed Documents. If not found, the existing hardcoded DEFAULT_CONFIG is used. Subsequent launches load whatever's in Documents — never overwrites.
Updated build command (one extra --add-data line for providers.json):
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "providers.json;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
Just drop your up-to-date providers.json next to app.py before building. Users who already have a providers.json see no change.
Tests. 14 V56 cases covering: extract_fields uses the work_provider rule (not preset name) for both FW presets, legacy presets get a default blank rule via normalize_provider_config, the rules table renders all 14 expected rows in the correct order, work_provider load/save round-trip works, bundled providers.json seeds the Documents folder on first launch, subsequent launches don't re-seed, missing bundled file falls back to DEFAULT_CONFIG. All V42-V55 prior suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: These words docs appear to have square symbols in the source text viewer at the start of each line.ALS 01.DOCDOCALS 02.DOCDOCThese words docs appear to have square symbols in the source text viewer at the start of each line.When I export a JSON with one of these symbols in the detected field, it shows in the JSON as \u0007What can be done? I don't want to sacrifice the import performance of any other documents7 MayClaude responded: Quick check first — let me look at the files and confirm what's going on: Interesting — when I run it through LibreOffice on Linux, the \x07 characters aren't …Quick check first — let me look at the files and confirm what's going on:Ran 3 commands, created a fileRan 3 commands, created a fileInteresting — when I run it through LibreOffice on Linux, the \x07 characters aren't there. So they must be coming through the Microsoft Word extraction path on the user's machine. Let me confirm by reading the source code:Viewed 3 filesViewed 3 filesFound it. Two diagnostic findings:

Yes, this is a Word automation issue. Look at line 599: there's already a partial fix for \x07 — unique_story_lines replaces \r\x07 with \n. The \x07 (BEL character) is what Word's COM API emits as a table cell terminator in Range.Text. Each cell ends with \x07, and the row ends with \r\x07\x07 or similar. The existing fix only handles the \r\x07 combo (in headers/footers), but doc.Content.Text at line 655 just does .replace("\r", "\n") — leaving \x07 characters untouched in the main body content.
The downstream effect. When the user maps a field that pulls from a table cell (e.g. an Address: rule grabbing the address line), the trailing \x07 from the cell terminator ends up in the value. It's invisible in the preview (Tk's text widget renders it as a tofu square), gets exported as \u0007 in JSON.

The fix is targeted: strip \x07 from the document content in the Word automation path. No other path is affected — LibreOffice converts to DOCX which doesn't have these markers, antiword strips them, DOCX direct doesn't expose them, etc. Performance impact: zero — it's a single regex on the already-extracted string in the one path that triggers them.
Even better: I should also clean_value-strip them defensively (so if any other path ever produces them they get cleaned). And unique_story_lines already strips \r\x07 but not lone \x07 — let me extend that too.
The right fix is a small character-cleanup applied to the Word extraction's output. Let me decide on the cleanest place:

Option A: In _extract_doc_text_via_word only — clean \x07 from content, headers, footers. Targeted, no impact elsewhere.
Option B: A shared post-processor that removes \x00-\x08, \x0b-\x0c, \x0e-\x1f (control characters except \t, \n, \r) from any extracted text. More defensive, applies everywhere — but it's a stronger change with potential to surprise.

Given your constraint ("don't sacrifice import performance of any other documents"), Option A is the right shape. It's a string operation on already-in-memory text in the one path that's affected. Cost: unmeasurable.
Implementing now:Strip Word cell-terminator BEL bytes from the main contentStrip Word cell-terminator BEL bytes from the main contentNow the bigger one — doc.Content.Text itself. Currently:
pythoncontent = (doc.Content.Text or "").replace("\r", "\n")
This leaves \x07 intact. Strip them too:Strip BEL bytes from main document contentStrip BEL bytes from main document contentNow think about whether any other paths could leak \x07. The user reports seeing them, so something is producing them in their actual flow. Let me also check that the DOCX path (which fires after LibreOffice converts a .DOC to .DOCX, or when the user imports a .DOCX directly) isn't producing them. The python-docx library's paragraph.text and cell.text properties already filter to plain text, so they should be clean. But let me also add a small belt-and-braces cleanup in clean_value — that helper runs on every extracted line everywhere, so it would catch any leak:
Wait, but the task says don't sacrifice performance. clean_value runs on every line, every match, dozens of times per document. Adding a regex re.sub(r"[\x00-\x08\x0b\x0c\x0e-\x1f]", "", value) would cost something measurable on large documents.
Let me think about how often clean_value is called and whether the cost matters in practice. It's used during extraction matching, not during preview rendering. On a typical document the call count is ~50-200 lines × a handful of fields = maybe 500 calls. A simple .replace() is faster than a regex; let me test both:Measure cleanup costMeasure cleanup costOn a 500-character line, even the regex costs ~3μs per call — completely unmeasurable in practice (500 calls = 1.5ms total). The user wouldn't notice.
But I'm going to skip the clean_value change anyway. The fix is already targeted at the source in the Word path. Adding belt-and-braces in clean_value would be defensive coding for a problem that's already solved at the source. Better to keep clean_value simple. If a future codepath leaks \x07, we'll fix it there.
Compile and test:Syntax checkSyntax checkTests. Since Word automation only runs on Windows, I can't run the actual COM extraction here. But I can:

Unit-test the unique_story_lines helper directly.
Test that clean_value still works as before.
Mock doc.Content.Text to verify the main content path strips \x07.
Created a file, ran a commandCreated a file, ran a commandExcellent — all 14 cases pass, including the end-to-end test that simulates the user's actual scenario (BEL-tainted content → field extraction → JSON encoding) and confirms \u0007 no longer appears in the JSON.
Run the full regression sweep:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV57 done.
Diagnosis. Word's COM API uses \x07 (BEL control character) as a table-cell terminator in Range.Text. The existing code already handled the \r\x07 row-ending combo for headers/footers via the unique_story_lines helper, but the main body content (doc.Content.Text at line 655) was just doing .replace("\r", "\n") — leaving lone \x07 characters in the body and inside cells. Any mapping rule pulling from a table cell ended up with a trailing BEL in the value, which Tk renders as a tofu square in the preview and JSON encodes as \u0007.
Fix. Two lines of code, in the Word path only:

Main content: (doc.Content.Text or "").replace("\r\x07", "\n").replace("\x07", "") before the existing \r → \n step.
unique_story_lines (used for headers/footers): same treatment so any leftover lone \x07 outside the \r\x07 pattern also gets stripped.

Why this won't affect anything else. The change is in _extract_doc_text_via_word only — the path that runs when Microsoft Word is available on the user's machine. The LibreOffice fallback, antiword, DOCX-direct, PDF (block mode + OCR), EML, and MSG paths are all untouched. Zero performance cost — it's a single str.replace on already-in-memory text.
Tests. 14 V57 cases including: main-content \x07 stripping, header/footer \x07 stripping, ALS-style real-world content, end-to-end JSON encoding (verifying \u0007 no longer appears in the JSON output). All V42-V56 prior suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Another quick update, can the offset in "Single Label +/-" ignore blank lines?Another quick update, can the offset in "Single Label +/-" ignore blank lines?7 MayClaude responded: Quick clarification before I implement, because there are two reasonable interpretations:Quick clarification before I implement, because there are two reasonable interpretations:
(a) Skip blank lines in the count. The offset only counts non-blank lines. So with config Vehicle Reg: || +1:
Vehicle Reg: AB12CDE     ← anchor
                          ← blank, not counted
                          ← blank, not counted  
12 High Street           ← +1 (first non-blank below)
Returns 12 High Street.
(b) Anchor on label, then return the next non-blank line at-or-near offset. Same as (a) in practice — I think this is what you mean.
I think you mean (a): blank lines don't consume offset count, only non-blank lines do. So +1 means "the first non-blank line below the anchor", +2 means "the second non-blank line below", etc. For negative offsets, same — -1 is the first non-blank line above. The anchor line itself (+0 / -0) returns the label line, blank or not.
Quick edge case to confirm: what if the anchor line is itself blank? Can't happen — labels don't match blank lines, so the anchor line always has content. So that's a non-issue.
One more consideration: bounds clamping with skipped blanks. Today, +99 from any line clamps to the last line of the document. With blank-skipping, +99 would clamp to the last non-blank line. Sound right?
Confirm:

Blanks ignored in both directions (positive and negative offsets).
Out-of-bounds clamps to the last/first non-blank line.
You said: Confirm on bothConfirm on both7 MayClaude responded: Replace the simple offset-and-clamp with blank-skipping logic: Now run the existing V55 tests — most of them should still pass because they use blank-free inpu…Viewed 2 filesViewed 2 filesReplace the simple offset-and-clamp with blank-skipping logic:Edited a file, ran a commandEdited a file, ran a commandNow run the existing V55 tests — most of them should still pass because they use blank-free input. The ones that test bounds clamping need a closer look: "second || -10" against "first\nsecond\nthird\n" should still return "first" because there are no blanks involved.Run V55 testsRun V55 testsV55 fully passes (the existing assertions don't involve blank lines, so all the same answers come out). Now write V58 specifically for blank-skipping behaviour:Created a file, ran a commandCreated a file, ran a commandAll 16 cases pass — including whitespace-only lines being treated as blank (handy for OCR'd docs that produce indented blank-looking lines), bounds clamping to last/first non-blank line, and +0/-0 still returning the anchor itself.
Full regression sweep:Full regression sweep V42-V57Full regression sweep V42-V57All clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV58 done.
The change. extract_by_single_label_offset now skips blank (and whitespace-only) lines when counting offsets. Walks the document one line at a time in the offset's direction; only non-blank lines decrement the remaining-steps counter. Returns whatever the last non-blank line we landed on was (which clamps cleanly to last/first non-blank line if we run out of document).
Implementation. A small step-walker in place of the previous arithmetic. About 12 lines of code in extract_by_single_label_offset. No change to UI, save/load, storage format, or any other method.
Edge cases handled correctly:

Whitespace-only lines (spaces/tabs only) count as blank — using line.strip() rather than just line == "".
+0 and -0 still return the anchor line itself (early-return shortcut).
Bounds clamp to the last non-blank line in the requested direction, not the document's literal last line.
If there are no non-blank lines in the offset's direction at all, we clamp to the anchor.
Label-not-found still returns empty (no change).

Tests. 16 V58 cases covering positive/negative blank-skipping, whitespace-only lines, +0/-0, bounds clamping with trailing/leading blanks, and a behaviour-preservation check confirming that documents with no blank lines produce identical results to V55. All V42-V57 prior suites still pass — including the V55 cases for the same method, since none of them involved blank lines.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: The attached pdf causes our application to freeze, my assumption is because it's trying to OCR it for some reason, but that's pure speculation.pdfThe attached pdf causes our application to freeze, my assumption is because it's trying to OCR it for some reason, but that's pure speculation.The PDF is a 26 page document containing just images7 MayClaude responded: Let me check what's actually happening before assuming: Confirmed your hypothesis.Let me check what's actually happening before assuming:Diagnose the freezing PDFDiagnose the freezing PDFConfirmed your hypothesis. This is exactly the OCR trigger condition we built in V54. 26 pages, zero text, exactly one image per page → all conditions met → OCR fires on all 26 pages. At 1.7-3.2s per page that's roughly 60-90 seconds of OCR before the app responds again.
The app isn't truly "frozen" — it's just running OCR synchronously on the main thread (no background processing). From the user's perspective for ~90 seconds, identical to a freeze.
Now to think through the right fix carefully — this is a meaningful design question.
The V54 trigger we agreed on was: "zero text AND every page has exactly one image". The intent was "scanned letter" — typically 1-2 pages, occasionally 3-5. We didn't consider photo-dump documents that happen to be 1-image-per-page (because each page is one big collage photo or one camera-shot per page).
Three ways to fix this, ranked by my preference:
(a) Page count cap. OCR fires only if total pages ≤ N (probably 5). A genuine scanned letter is almost never longer than a few pages. A photo dump where each page is a single image is the photo dump case, not a scanned letter. Easy to reason about, easy to explain.
(b) Async OCR with cancel button. Move OCR to a background thread, show a progress indicator with a cancel button. The app stays responsive, the user sees what's happening, can stop it if they realise it's wrong. Bigger code change but better UX overall.
(c) Both. Cap to ≤5 pages AND make the existing OCR async with progress. Belt and braces.
I'd recommend (a). Reasoning:

Your real scanned letters were 1 page. Even allowing 5 covers the realistic envelope.
Async with a cancel button is more code and more failure modes (threading + Tk quirks). For a "productivity tool" that should mostly Just Work, a smarter heuristic that doesn't trigger inappropriately is better than a fancier UI to handle the inappropriate trigger.
A 26-page photo dump with 1 image per page is genuinely a photo-dump — not something the user wants OCR'd. They want it skipped. A page cap correctly skips it.

One subtlety: where to set the cap. Looking at scanned letters in the wild:

1 page: instruction letter (most common)
2 pages: longer instruction + addendum
3-4 pages: multi-section instruction
5+ pages: getting into "this is something else" territory

I'd cap at 5 pages. Wider than your typical use, narrow enough to exclude photo dumps. If you ever have a legitimate 6-page scanned letter, the cap can be raised — but I'd prefer to start conservative.
Two small follow-up questions:
Q1: Cap at 5? Or somewhere else (3? 10?)?
Q2: When OCR is skipped because of the page-count cap, should the user see a note in the status bar — something like "Skipped OCR (26 pages exceeds limit)" — or fully silent (consistent with the current "OCR fires silently" behaviour)? My instinct: silent. The user already gets empty text if no provider matches, which is the same outcome as today. No new noise.
Confirm those and I'll implement.You said: I'm thinking cap to 2, there is only one single OCR that spanned 2 pages, every other OCR scenario was 1 page.I'm thinking cap to 2, there is only one single OCR that spanned 2 pages, every other OCR scenario was 1 page. Image PDF was mostly always be 3+ pages, so 2 can be the cut off where logic switches7 MayClaude responded: Cap at 2 it is.Cap at 2 it is. That's a tighter fit — the gap between "scanned letter" (1-2 pages) and "photo dump" (3+ pages) is genuinely the right dividing line for your data. And keeping it silent (no status message) per the V54 ethos.
Implementing:Ran a command, viewed a file, edited a fileRan a command, viewed a file, edited a fileNow define OCR_PAGE_LIMIT near the top with the other constants:Ran a command, viewed a file, edited a fileRan a command, viewed a file, edited a fileCompile:Syntax checkSyntax checkNow write tests covering:

The freezing PDF (26 pages, one image each) is skipped by the cap
1-page scanned letter still triggers OCR
2-page scanned letter (your single example) still triggers OCR
3-page document with one image per page does NOT trigger OCR (the new boundary case)
Existing V54 tests still pass
Ran 2 commandsRan 2 commandsGood — all four fake PDFs match the trigger pattern (zero text, one image per page). Now the test:Created a file, ran a commandCreated a file, ran a commandThe 26-page real PDF correctly skips OCR (0.18s — no waste). The MP scanned letters still trigger correctly. The cap is in place.
The two failures are on my synthetic fake PDFs — likely those are blank white pages so OCR returns empty. That's an OCR-output thing not a trigger thing. Let me verify:Verify fake PDFs OCR to emptyVerify fake PDFs OCR to emptyConfirmed — the OCR fires on the fake PDFs (because they pass all conditions including the page cap), but the white-square images contain nothing to recognise so OCR returns empty, the if ocr_pages and not ocr_failed: guard fails, and no note is added. The trigger logic works; my test fixtures are just too synthetic.
Let me fix the test fixtures by drawing actual text on the images, so OCR has something to find:Ran 2 commandsRan 2 commandsAll 11 cases pass. The boundary cases work correctly: 1 page → OCR fires, 2 pages → OCR fires, 3 pages → no OCR. The real 26-page freezing PDF takes 0.18s now instead of locking up the app for 60-90 seconds.
Full regression sweep:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV59 done.
Diagnosis. Your hypothesis was correct. The PDF has 26 pages, zero text characters, and exactly one image per page — perfectly matching the V54 OCR trigger. The app was running OCR on all 26 pages synchronously, taking ~60-90 seconds during which the main thread is blocked, indistinguishable from a freeze.
Fix. Added a third condition to the OCR trigger: the document must be at most OCR_PAGE_LIMIT pages. New trigger condition is now:

Zero text characters AND every page has exactly one image AND total pages ≤ 2

Implementation is a single OCR_PAGE_LIMIT = 2 module-level constant and one extra clause in the trigger expression (and 0 < len(per_page_image_counts) <= OCR_PAGE_LIMIT). About 3 lines of new code.
Verified end-to-end on the actual PDF. Your 26-page photo dump now imports in 0.18 seconds (down from ~60-90 seconds of frozen UI). The V54 scanned letters still OCR correctly.
Boundary tested: 1 page → OCR fires; 2 pages → OCR fires (at the limit); 3 pages → OCR skipped. Tuning is one constant if you ever decide a 3-page scanned letter is realistic.
Tests. 11 V59 cases including the actual freezing PDF (now 0.18s, no OCR), boundary tests at 1/2/3/5 pages, and V54 regression checks confirming MP scanned letters still trigger OCR. All V42-V58 prior suites pass unchanged.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Quick query, do all users need to manually install tesseract ocr on their pcs to use this tool, or is it portable inside our app after compiling?Quick query, do all users need to manually install tesseract ocr on their pcs to use this tool, or is it portable inside our app after compiling?7 MayClaude responded: Good question to ask before deployment — this is exactly the kind of thing that bites people once they hand the .Good question to ask before deployment — this is exactly the kind of thing that bites people once they hand the .exe to someone else. Let me trace through the flow concretely.
Short answer: portable, no install needed by users — IF you bundle it correctly when you build the .exe.
Here's the flow:

You (on the build machine) install Tesseract once via the UB Mannheim installer. This is just to get the binaries onto your machine.
You copy the Tesseract folder into a tesseract/ subfolder next to app.py in your project.
You run PyInstaller with --add-data "tesseract;tesseract". PyInstaller bundles that folder into the .exe.
You ship the .exe to users.
Users double-click the .exe. At runtime, PyInstaller extracts the bundled tesseract/ folder to a temp directory (sys._MEIPASS). The app's configure_bundled_tesseract() startup function points pytesseract at that temp path.
Users get full OCR functionality. They never had to install anything.

The catch — and this is genuinely important. The check is: did you use this build command:
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "providers.json;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
If you used your older command (without the --add-data "tesseract;tesseract" line), the .exe doesn't include Tesseract, and OCR will silently no-op when users hit a scanned PDF — exactly like the pre-V54 behaviour. No crash, no error, just "scanned PDF imports as empty."
How to confirm OCR is actually bundled in your .exe. Two ways:

The size check. A bundled .exe should be roughly 80-100 MB. A non-bundled one is roughly 30 MB. If yours is small, Tesseract isn't in there.
The functional check. Drop one of the MP_PDF_*.pdf scanned letters into the running .exe. If you see VRM/refs/etc fields auto-populate after a 1-3 second pause, OCR is working. If the source preview comes up empty and no fields populate, Tesseract isn't bundled.

So the answer is: fully portable inside the .exe, zero user install — provided you bundle Tesseract during your build. Once that's done, you can hand the .exe to anyone (no admin rights, no Tesseract install, nothing) and OCR will work for them.You said: I've ran a number of tests and I've realised that leaving inspection address empty causes an import error in our management software when trying to import the …AMS_MX16SYJ.json15 linesjsonAX_SG75CEU.json15 linesjsonI've ran a number of tests and I've realised that leaving inspection address empty causes an import error in our management software when trying to import the JSON. Look at the 2 JSONs I have attached, one will import, and the other causes an error.My thoughts are to add a rule on export to force 6 lines in the inspection address even if there's no data?7 MayClaude responded: Diagnosis is correct — the AMS one (with "") imports cleanly because it's a single empty string, but the AX one fails because of the "\n\n\n\n\n\n" (six newlin…Diagnosis is correct — the AMS one (with "") imports cleanly because it's a single empty string, but the AX one fails because of the "\n\n\n\n\n\n" (six newlines, seven lines all empty). Wait, that's the wrong way around. Let me re-read.
You said one imports and one errors. Looking at the values:

AMS: "Inspection Address": "" — completely empty string
AX: "Inspection Address": "\n\n\n\n\n\n" — six newlines = seven lines, all blank

Without knowing which one imports, my guess is the AMS one (empty string) is the one that errors, because your management software expects the 6-line format and a bare empty string isn't that. The AX one ("\n\n\n\n\n\n") gives 7 lines which is one too many but maybe truncated to 6 fields... or maybe one of the seven gets dropped as an empty trailing line.
Either way, your proposed solution sounds right but I want to nail down the exact format. Quick question: when an inspection address IS populated (provider matched, address extracted), what does the JSON value look like? Is it currently like:
"Inspection Address": "53 Regan Crescent\nBirmingham\n\n\n\nB23 5NN"
— always 6 lines exactly, separated by 5 newlines, with empty middle lines as needed? Let me check the existing normaliser to confirm:Ran a command, viewed a fileRan a command, viewed a fileOK so I now understand the full picture:

The function already produces a 6-line normalised address when there's content. The output is "line1\nline2\nline3\nline4\nline5\npostcode" — always 6 elements separated by 5 newlines.
But it returns "" (empty string) when the input is empty or whitespace-only — see line 1982-1983 and line 1998. That's the bug for export purposes: if a provider doesn't auto-extract an address and the user doesn't manually fill it, the JSON gets "" instead of "\n\n\n\n\n" (5 newlines = 6 empty lines).

So which JSON imports cleanly? Looking at the AX one: "\n\n\n\n\n\n" is 6 newlines = 7 lines. The AMS one: "" is just an empty string.
If your management software needs exactly 6 lines, the AX one is wrong too (it's 7 lines). And the AMS one is also wrong (zero lines).
So I need to know: which one imports successfully? If it's the AX one (7 lines), then the management software is being lenient and stripping trailing empties down to 6. If it's the AMS one (empty string), then the management software is OK with just an empty string but not with newlines.
Most likely: your management software wants exactly 5 newlines separating 6 fields (5 empty + 1 with postcode, or 6 empty if no data) — and "\n\n\n\n\n\n" (7 lines) is being parsed and the trailing empty gets dropped, leaving 6 fields.
The fix I'll implement is: always export a 6-line normalised address, even when empty. That handles both cases (the AMS empty string AND the AX 7-line string both become "\n\n\n\n\n" = 6 empty lines).
Let me code this up. The cleanest place is in the export-time normalisation — I want to be careful not to change the behaviour of the function when called during extraction (where empty means "we have no data, leave the field truly empty in the UI"). The post-process step that normalises for export is the right place.Ran a command, viewed a fileRan a command, viewed a fileTwo relevant call sites:

Line 1899: Called during extraction (in extract_fields) — this is the one that fills the UI. For UI purposes, an empty address should stay empty (no point showing 6 blank lines in the entry widget).
Line 3386: Called during export prep (prepare_export_values) — this is the one that builds the JSON export. For export, we always want 6 lines.

Cleanest fix: change normalise_inspection_address_value to always return 6 lines (so empty → "\n\n\n\n\n"), but the UI display of the address widget needs to keep showing it as empty when blank — let me trace what the entry widget does with this value.Ran a command, viewed a fileRan a command, viewed a fileGood — get_field_value for inspection_address uses .rstrip() to remove trailing whitespace. So if we set the widget value to "\n\n\n\n\n" (6 empty lines), reading it back gives "" — the trailing newlines get stripped. That's a problem if we want the export to be 6 lines.
Two paths possible:
Path A: Change normalise_inspection_address_value to always return 6 lines. Set the address widget to a 6-line value when blank → user sees 6 blank lines in the entry widget. JSON export gets 6 lines. But .rstrip() in get_field_value would then erase them, so we'd need to remove the rstrip too. Bigger change.
Path B: Leave the in-memory/widget representation as "" for empty, but normalise specifically at JSON-export time so the JSON gets "\n\n\n\n\n". Surgical change in prepare_export_values (or just in the JSON-build step). UI is unchanged.
Path B is cleaner. The user still sees a blank address widget when it's blank. The JSON output is always 6-line normalised. No UI changes, no risk of breaking other places that consume inspection_address.
Cleanest implementation: ensure normalise_inspection_address_value always returns 6 lines (including the empty-input case), and make sure it's called during export prep only on values that haven't already been normalised.
Wait — actually, let me think again. prepare_export_values already calls post_process_extracted_value("inspection_address", ...) which calls normalise_inspection_address_value. So if I just change normalise_inspection_address_value to always return 6 lines (instead of returning "" for empty input), the export-time call already does the right thing. The question is: does anything else call this function on extracted values that we don't want to be 6-line?
Look at the two call sites:

Line 1899 (in extract_fields): extracted = post_process_extracted_value(...) — this populates the UI. If it returns "\n\n\n\n\n" for an empty extraction, the address widget would show 6 blank lines. That's actually... fine? The user sees they need to fill it in (or that there's no detected address). But it might also be visually weird — they'd see a tall blank widget.

Actually, the concrete problem: if extraction returns "" (no address found in document), normalise_inspection_address_value("") would now return "\n\n\n\n\n", and that goes to the UI — showing as 6 blank lines. If they then export, they get the 6 blank lines in JSON. ✓ Good.
But: if they then type an address, the widget contains their text, get_field_value rstrips trailing newlines, and the export still goes through normalise_inspection_address_value which 6-line-normalises it. ✓ Good.
The only weird thing is the UI appearance: a blank address would show as a 6-line tall blank widget rather than empty. That's actually consistent with what the widget already does when there's a populated address (which is also 6 lines).
Let me verify by looking at how the widget is built:Ran a command, viewed a fileRan a command, viewed a fileThe widget is a 6-line-tall tk.Text (line 2810: height=6). So whether the value is empty or "\n\n\n\n\n", the widget physically displays 6 lines worth of space. The visual difference is that an empty value shows the cursor at the top with no lines visible, and a "\n\n\n\n\n" value shows the cursor at the top with the user able to see 5 line-break positions below. In practice indistinguishable.
OK going with Path A: change normalise_inspection_address_value to always return a 6-line value (including the empty case). Reason it's cleaner: the contract becomes "this function always returns a 6-line normalised address" — no empty-string special case for downstream code to worry about.
But there's still the rstrip issue in get_field_value. If I set the widget to "\n\n\n\n\n", reading back via get_field_value rstrips to "". Then export prep calls normalise_inspection_address_value("") which now returns "\n\n\n\n\n". So the round-trip works correctly. ✓
Wait, let me also check if a populated value going through this path would lose data. Address is "53 Regan Crescent\n\n\n\n\nB23 5NN" (6 lines, lines 2-5 blank). User reads via get_field_value → rstrip removes nothing because the last line has content (B23 5NN). Good. What if user has "53 Regan Crescent\nB23 5NN\n\n\n\n\n" (a malformed populated value with trailing blanks)? rstrip gives "53 Regan Crescent\nB23 5NN" — 2 lines. Then normalise_inspection_address_value on a 2-line input takes line 2 as postcode → "53 Regan Crescent\n\n\n\n\nB23 5NN". Good.
Alright, the change is minimal and safe. Let me implement:Edited a file, ran a commandEdited a file, ran a commandTests:Created a file, ran a commandCreated a file, ran a commandAll passing — including the round-trip from widget-empty through export. The user's two failing scenarios both produce a clean 6-line output now.
Full regression sweep:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV60 done.
Diagnosis. Looking at the two JSONs you sent:

AMS_MX16SYJ.json: "Inspection Address": "" (empty)
AX_SG75CEU.json: "Inspection Address": "\n\n\n\n\n\n" (6 newlines = 7 lines, all blank)

The AX one imports because your management software treats it as 7 fields and probably trims trailing empties down to the 6 it expects. The AMS one fails because it's a bare empty string that doesn't match any line count at all.
Fix. normalise_inspection_address_value now always returns the canonical 6-line shape — including for empty input. Empty/whitespace input now returns "\n\n\n\n\n" (5 newlines = 6 empty fields) instead of "". Two-line change at the top of the function.
Side benefit: The AX-style 7-line value also gets renormalised. Once it goes through the export pipeline, it becomes 6 lines just like the AMS one. So both your example scenarios produce identical, schema-compliant output.
No UI changes. The Inspection Address widget is already a 6-line-tall text box, so a 6-line empty value displays identically to how a true-empty value used to. The widget read-back path still rstrips trailing whitespace (so "really empty in the UI" reads as "" for internal purposes), and then the export pass re-normalises to the 6-line shape — the round-trip is clean.
Tests. 14 V60 cases including the literal AMS and AX scenarios, the empty/None/whitespace inputs all producing 6 lines, populated addresses still being 6-line normalised, V48 force-postcode behaviour preserved, and an end-to-end UI round-trip through prepare_export_values. All V42-V59 prior suites pass unchanged.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: One final thing - we need to make sure dates are all formatted in DD/MM/YYYY in the JSON exportOne final thing - we need to make sure dates are all formatted in DD/MM/YYYY in the JSON export7 MayClaude responded: Let me look at the actual dates that come through to understand what needs converting: Look at the dates from the AMS sample:Let me look at the actual dates that come through to understand what needs converting:Find the date fields and current handlingFind the date fields and current handlingLook at the dates from the AMS sample:

Incident Date: 23/04/2026 ✓ already DD/MM/YYYY
Instruction Date: 27th April 2026 ✗ long form
Inspection Date: 07/05/2026 ✓ already DD/MM/YYYY

And the AX sample:

Incident Date: 21 Apr 2026 ✗ short month name
Instruction Date: 05 May 2026 ✗ short month name
Inspection Date: 08/05/2026 ✓ already DD/MM/YYYY

So we need to convert formats like "27th April 2026" and "21 Apr 2026" into "27/04/2026" and "21/04/2026". A few formats to think about and decide whether to support:
Formats I'd want to handle (from the samples and other UK-typical):

DD/MM/YYYY — already correct, pass through
DD/MM/YY — UK-style with 2-digit year, expand to 4-digit
DD-MM-YYYY — alternative separator, normalise to /
DD MMM YYYY (e.g. 21 Apr 2026) — short month name
DD MMMM YYYY (e.g. 27 April 2026) — full month name
DDth MMMM YYYY (e.g. 27th April 2026) — ordinal suffix
MMMM DD, YYYY (e.g. April 27, 2026) — US-ish but possible
YYYY-MM-DD (ISO) — possible from some systems

Formats I'd probably NOT auto-convert because of ambiguity:

DD/MM or DD MMM (no year) — too sparse
MM/DD/YYYY — looks identical to DD/MM/YYYY for days ≤ 12, ambiguous

Two design questions before I implement:
Q1: Should the date normalisation happen at extraction time (so the user sees DD/MM/YYYY in the UI) or at export time (so the UI shows whatever was extracted, but the JSON always exports DD/MM/YYYY)?
I'd recommend export time. Reasoning:

It mirrors the pattern we just established with V60 inspection address (preserve UI shape, normalise on export).
If the user manually types something into a date field, they probably want to see it in the format they typed.
On export, the JSON always meets the schema regardless of user input or extraction format.

Q2: When a date can't be parsed (gibberish input, unsupported format), what should happen? Two options:

(a) Leave it as-is in the JSON — user sees the unparsed value and notices the import error.
(b) Empty it out — silent loss of data but JSON is always valid.

I'd recommend (a): leave unparseable values as-is. Same philosophy as the rest of the app — never silently throw away user data, the user catches the issue at import-time and fixes the source.
Confirm those two and I'll implement.You said: Q1) Export timeQ1) Export time
Q2) Leave as is7 MayClaude responded: Plan:Ran a command, viewed a fileRan a command, viewed a filePlan:

Add a top-level normalise_date_value function that takes a string and tries each known format, returning DD/MM/YYYY on success or the original string on failure.
Add it to post_process_extracted_value for *_date fields (so it composes with the existing per-field normalisers).
Wire it into prepare_export_values for the three date fields.

Let me think carefully about the parsing strategy. I'll use a list of strptime format strings and try each in order. The tricky part is the ordinal suffix (27th, 1st, 2nd, 3rd) — strptime doesn't handle those. I'll strip ordinal suffixes before parsing.
Format list, in priority order (most specific first):

%d/%m/%Y — already correct, fast pass-through
%d/%m/%y — 2-digit year
%d-%m-%Y
%d-%m-%y
%d %B %Y — 27 April 2026
%d %b %Y — 21 Apr 2026
%B %d %Y — April 27 2026
%b %d %Y — Apr 21 2026
%Y-%m-%d — ISO

For %B and %b, Python's strptime is locale-sensitive but defaults to English month names which is what we want.
Let me code:Viewed a file, edited a fileViewed a file, edited a fileNow the question — do I want the date normalisation to fire only at export time, or also during extraction? The user said export time. So I should NOT call post_process_extracted_value for date fields during extraction.
Let me check the extraction call site:Look at extraction call siteLook at extraction call siteHmm — so post_process_extracted_value is currently called during extraction. If I add the date branch there, dates would be normalised at extraction time too — which contradicts the user's "export time only" preference.
Two options:
Option A: Add a for_export flag to post_process_extracted_value. Default False, so extraction-time calls skip date normalisation. Export-time call passes True.
Option B: Keep post_process_extracted_value simple (just normalise everything), and leave the design as "normalisation can happen multiple times along the pipeline; it's idempotent." Then dates would be normalised at extraction AND at export, but since DD/MM/YYYY → DD/MM/YYYY is idempotent, there's no harm. The UI would just show DD/MM/YYYY for all cases.
Option C: Don't use post_process_extracted_value for dates at all. Add a separate prepare_export_values step specifically for dates.
The user explicitly said "export time" — my read is they want the UI to preserve whatever the document said, and only normalise at the JSON output. That points to Option A or C.
Option C is cleanest because it keeps the extraction path untouched and adds the date normalisation only at the export-prep step. Let me back out my edit to post_process_extracted_value and do it as a separate step in prepare_export_values:Revert post_process_extracted_value to not date-normalise during extractionRevert post_process_extracted_value to not date-normalise during extractionNow wire normalise_date_value into prepare_export_values:Add date normalisation to prepare_export_valuesAdd date normalisation to prepare_export_valuesCompile check:Syntax checkSyntax checkTests covering the user's specific scenarios plus a comprehensive matrix of formats:Created a file, ran a commandCreated a file, ran a commandAll 41 V61 cases pass — including the literal AMS and AX scenarios. Run the full regression:Full regression sweepFull regression sweepAll clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV61 done.
The change. A new normalise_date_value helper that converts a date string into DD/MM/YYYY. Wired into prepare_export_values for the three date fields (incident_date, instruction_date, inspection_date). UI is untouched — the Detected Fields panel still shows whatever the document said. Only the JSON export canonicalises.
Implementation. A list of strptime format strings tried in order. Pre-cleans the input (strips ordinal suffixes like 27th, normalises commas and whitespace). On parse success returns DD/MM/YYYY; on failure returns the original string unchanged so the user spots the issue at management-software import time.
Recognised formats:

DD/MM/YYYY (canonical, passes through)
DD/MM/YY (2-digit year)
DD-MM-YYYY, DD-MM-YY
DD MMMM YYYY (e.g. 27 April 2026)
DD MMM YYYY (e.g. 21 Apr 2026)
MMMM DD YYYY and MMM DD YYYY (month-first variants)
YYYY-MM-DD (ISO)
All of the above with ordinal suffixes (1st, 2nd, 3rd, 27th, case-insensitive)

Verified end-to-end on your two scenarios:

AMS: "Instruction Date": "27th April 2026" → "27/04/2026" ✓
AX: "Incident Date": "21 Apr 2026" → "21/04/2026" ✓
AX: "Instruction Date": "05 May 2026" → "05/05/2026" ✓
All others (already canonical) pass through unchanged.

Tests. 41 V61 cases covering: every supported format, ordinal suffixes (lowercase and uppercase), whitespace/comma tolerance, month-first variants, ISO format, empty/None/whitespace inputs, unparseable inputs preserved as-is, end-to-end through prepare_export_values, and the literal AMS/AX export scenarios. All V42-V60 prior suites still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: First roll out and it's going well, but one issue we've discovered is that users with Onedrive installed have trouble finding the exports because they are on t…First roll out and it's going well, but one issue we've discovered is that users with Onedrive installed have trouble finding the exports because they are on their "real" desktop, not the cloud/Onedrive one. What can we do about this?8 MayClaude responded: Good catch.Good catch. Let me trace what we're actually doing today before suggesting fixes:Ran a command, viewed a fileRan a command, viewed a fileOK so the issue is clear. We do Path.home() / "Desktop" — which on Windows gives C:\Users\<user>\Desktop. But on a OneDrive-managed machine, the actual desktop the user sees is C:\Users\<user>\OneDrive\Desktop (or C:\Users\<user>\OneDrive - <Org>\Desktop for work accounts). Files we drop in C:\Users\<user>\Desktop end up on the legacy desktop folder which is invisible to the user.
This is a well-known Windows/OneDrive landmine. The robust fix on Windows is to ask Windows itself where the user's Desktop currently is, via the Shell API. Specifically, the KNOWNFOLDERID for Desktop (FOLDERID_Desktop) — Windows answers with whatever the actual current desktop path is, taking OneDrive redirection into account.
Two ways to call this from Python:
(a) Via ctypes calling SHGetKnownFolderPath — built-in, no extra dependency. About 25 lines of glue.
(b) Via pywin32's shell.SHGetFolderPath — already a dependency on Windows builds (in requirements.txt). Older API but works.
I'd go with (a) — SHGetKnownFolderPath is the modern API (introduced in Vista, still recommended) and using ctypes avoids a hard dependency on any pywin32 module.
Plan:

On Windows, ask the Shell for the real Desktop path via SHGetKnownFolderPath(FOLDERID_Desktop).
If that fails, fall back to the existing Path.home() / "Desktop" logic (legacy, but works on machines without OneDrive redirection).
On non-Windows, use the existing fallback (Linux/Mac don't have this issue).

Three things to confirm before I implement:
Q1: Should I make this silent (just use the right folder) or show the user the resolved path somewhere (e.g. in a status message after each export, or a "Show in Folder" button)? My instinct: silent — the goal is "exports land where the user expects them." Adding UI to "explain" the fix introduces friction. The status bar already says "Exported JSON to Desktop" — that's enough.
Q2: When the user has OneDrive but is offline, the OneDrive Desktop folder still exists locally — it just won't sync until they're online. So no special-casing needed for that. But: on machines with OneDrive paused or with sync conflicts, the folder still exists and is writable — files end up syncing later. I'll trust Windows to give us the right folder either way.
Q3: Anything else that uses Path.home() directly that might also be affected? Let me check:Ran a command, viewed a fileRan a command, viewed a fileget_documents_dir has the exact same bug. The user's providers.json lives at Documents\CE Document Mapper\providers.json — which is C:\Users\<user>\Documents\CE Document Mapper\ on a normal machine, but on a OneDrive-redirected machine the user's actual Documents folder is C:\Users\<user>\OneDrive\Documents.
This may or may not have caused them confusion — providers.json works as long as the same path is used both times (the app writes it and reads it from the same place), so this hasn't surfaced as an issue. But for consistency I should fix both.
Also one consideration: if a user's been running V60 for a few days, they'll already have a providers.json in C:\Users\<user>\Documents\CE Document Mapper\providers.json. If V62 changes to use C:\Users\<user>\OneDrive\Documents\CE Document Mapper\providers.json, the user suddenly has no providers (a clean re-seed from the bundled file). Their custom providers vanish. Bad.
So for Documents, I need a migration path: if the new Shell-resolved Documents folder differs from the legacy one AND the legacy providers.json exists AND the new one doesn't, copy the legacy one across. One-time auto-migration.
For Desktop, no migration needed — outputs are transient, they're not data the user expects to persist.
Confirm Q1 (silent), and let me know if you want the Documents migration as I described or if you'd prefer something else.You said: Silent.Silent.We basically want the application to know where the visual Desktop is (the one which you see when no windows are open) and to use that, however we can make that happen.We should probably apply the same change to the Documents folder for consistency.8 MayClaude responded: Two persistent files in the user's Documents folder: providers.Find places that depend on Documents pathFind places that depend on Documents pathTwo persistent files in the user's Documents folder: providers.json and app_settings.json. Both need to migrate from the legacy path to the Shell-resolved path on first launch of V62 if a user already has them.
Plan:

Add a Windows-specific helper that resolves the real known-folder path via SHGetKnownFolderPath for both Desktop and Documents.
Update get_desktop_dir() and get_documents_dir() to use it on Windows, falling back to the existing Path.home() logic on other platforms or if the Shell call fails.
Add a one-time migration: if there's a legacy providers.json (or app_settings.json) in the legacy Documents location AND the new Documents location doesn't have one, copy the legacy file across.

Let me implement.Add SHGetKnownFolderPath helper and use it for both Desktop and DocumentsAdd SHGetKnownFolderPath helper and use it for both Desktop and DocumentsNow wire the migration helper into the startup, just before APP_DATA_DIR.mkdir(...):Run migration before creating the new app data dirRun migration before creating the new app data dirCompile:Syntax checkSyntax checkNow tests. I can't run actual Windows Shell calls on Linux, but I can:

Verify non-Windows behaviour (the helper returns None, fall-through to Path.home() / "Desktop")
Mock the Shell helper to simulate a OneDrive-redirected machine and verify the right path is used
Test the migration: legacy file present, new path empty → file copied; new path has its own file → legacy untouched; legacy path missing → no-op
Created a file, ran a commandCreated a file, ran a commandAll 14 V62 cases pass. Run the full regression sweep:Full regression sweep V42-V61Full regression sweep V42-V61All clean. Update README and finalise:Ran a command, read a fileRan a command, read a fileV62 done.
The fix. A new _windows_known_folder_path(guid) helper that calls Windows' SHGetKnownFolderPath Shell API via ctypes. On a OneDrive-redirected machine this returns the actual OneDrive\Desktop (or OneDrive\Documents) path; on a non-redirected machine it returns the standard C:\Users\<user>\Desktop. Both get_desktop_dir() and get_documents_dir() now ask the Shell first; the existing Path.home()-based fallback only fires on non-Windows or if the Shell call fails for any reason.
Why ctypes not pywin32. pywin32 is already a Windows-only dependency for extract-msg etc., but using ctypes here means this single file doesn't pull in another optional import. Stays compact and self-contained.
The migration. Users who already have V60/V61 installed have customised presets in the legacy Documents\CE Document Mapper\providers.json location. After V62 their app would look in the new OneDrive-redirected location, find nothing, and silently re-seed from the bundled file — losing custom presets. To prevent this, V62 includes a one-time migration: on first launch, if the legacy location has files and the new location is empty, the legacy files are copied across (providers.json and app_settings.json). Best-effort, silent on success, silent on failure (in which case the user gets the bundled seed which is acceptable).
Edge cases handled:

Both legacy and new paths exist → migration is no-op (legacy left alone, new used as-is).
Legacy doesn't exist → no-op (fresh install).
Legacy and new are the same path (no OneDrive) → no-op (no copy onto self).
Non-Windows → no-op.

Tests. 14 V62 cases covering Shell-resolution mocks, fallback to home-relative on Shell failure, all migration scenarios. All V42-V61 prior suites pass unchanged.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: We need to add a new mapping method that is specific to just the "Instruction Date" field.We need to add a new mapping method that is specific to just the "Instruction Date" field. This mapping method we can name "Email Date".
The purpose of this method is to extract the date from the heading of .msg files which presents the date in a predictable format. There can sometimes be inconsistencies with the line number in .msg imports, so we shouldn't rely on that. My thoughts for the logic behind the "Email date" method can be to follow the "Single Label" approach, but then look specifically for YYYY-MM-DD.
For example, line 4 of this msg file looks like this:
"Date: 2026-05-05 16:57:31+01:00"
The user should be able to choose the "Email Date" method, input "Date:" into the setup, and the function should arrive at "05/05/2026" (dd/mm/yyyy) as the detected data.8 MayClaude responded: Quick clarifications before I implement, because this is a slightly unusual method:Quick clarifications before I implement, because this is a slightly unusual method:
Q1: Should this method be available in the dropdown only when configuring instruction_date, or visible everywhere but only meaningful for instruction_date?
The existing methods are universal (any field can use any of them). The cleanest pattern would be to keep it universal — Email Date appears in every field's dropdown, but only instruction_date is the realistic use case. This is consistent with how Fixed Position is technically usable on any field but pragmatically used for a few. Less code, less special-casing.
The alternative (filter the dropdown per-field) would be more "correct" UX but introduces a new pattern that doesn't apply elsewhere. I'd recommend keeping it universal.
Q2: The config takes a label (Date:). After matching the label, the method searches the rest of that line for a YYYY-MM-DD pattern. Two scenarios to think about:
(a) Label is on the same line as the date (your example):
Date: 2026-05-05 16:57:31+01:00
Match Date:, look for YYYY-MM-DD on rest of line → 2026-05-05 → format as 05/05/2026.
(b) Label on its own line, date on next line — like our other email-style content:
Date:
2026-05-05 16:57:31+01:00
Should this also work?
I'd recommend only case (a) — same line. Reasoning: this is .msg header territory which always has Date: <value> on a single line, by RFC standard. Supporting case (b) would mean the method fires on Date: labels in document body content (which appear on their own line in instruction letters), causing accidental matches. Case (a) only is the safer trigger.
Q3: What if multiple YYYY-MM-DD patterns appear on the same line after the label? E.g., Date: 2026-05-05 to 2026-05-10. I'd recommend first occurrence wins — same logic as other Single Label-style methods. Predictable, easy to reason about.
Q4: What if the label is found but no YYYY-MM-DD is on that line? Return empty (the rule didn't match), forcing the user to either fix the rule or fill in manually. Same philosophy as Fixed Position + Label and Single Label +/-.
Q5: Storage format. Since this method only takes one value (the label), it should reuse the Single Label UI shape — just one text entry, no second field. Storage as {"method": "email_date", "config": "Date:"}.
Confirm Q1-Q5 and I'll implement.You said: Q1) Okay let's keep it universal for simplicity.Q1) Okay let's keep it universal for simplicity.Q2) AQ3) Yes, first occurance winsQ4) Yes, leave emptyQ5) Yes, reuse Single Label UI shape8 MayClaude responded: Implementing.Implementing.
Edit 1: Add to METHOD_CHOICES.Add Email Date method choiceAdd Email Date method choiceEdit 2: Add the engine method and dispatch. Place it right after extract_by_single_label_offset:Ran a command, viewed a fileRan a command, viewed a fileNow insert extract_by_email_date and the dispatch line:Edited 2 filesEdited 2 filesEdit 3: UI — update_method_ui shows the single-entry frame (not the two-field one) for email_date. Let me check the current logic:Ran a command, viewed a fileRan a command, viewed a fileThe current logic is correct as-is for email_date — it falls into the else branch (single entry shown). No change needed. Email Date uses Single Label's UI shape.
Edit 4: save_provider_mapping — email_date has just one config field, save it as a single string (no ||):Look at save_provider_mapping field-loopLook at save_provider_mapping field-loopGood — email_date falls into the else branch which uses the single config var. No change needed.
Edit 5: load_provider_into_editor — same story:Look at load_provider_into_editorLook at load_provider_into_editorGood — email_date falls into the else branch which sets the single config var. No change needed.
One more thing to consider: the prepare_export_values step calls normalise_date_value on all date fields. Email Date already returns DD/MM/YYYY directly from the engine method. Running normalise_date_value on that would be a no-op (DD/MM/YYYY is the canonical format), so no harm done. Idempotent.
Compile:Syntax checkSyntax checkTests:Created a file, ran a commandCreated a file, ran a commandAll 22 V63 cases pass — including your literal sample (Date: 2026-05-05 16:57:31+01:00 → 05/05/2026), invalid date rejection, the deliberate same-line-only behaviour (label alone on line + date on next does NOT match), and end-to-end through extract_fields + JSON export.
Run the full regression sweep:Full regression sweepFull regression sweepOnly failure is the V59 timing assertion ("< 2s") — that's flaky timing on a busy test runner, not a behaviour regression. Let me confirm:Re-run V59 alone to confirm timing flakeRe-run V59 alone to confirm timing flakeConfirmed flaky — V59 passes when run on its own. The earlier sweep was running multiple test suites in parallel (or close to it) and the disk/CPU contention pushed the 26-page extraction over the 2-second threshold. Behaviour-wise everything is correct.
Update README and finalise:Ran a command, read a fileRan a command, read a fileV63 done.
The new method. Email Date — seventh dropdown option. User specifies a label (e.g. Date:). Engine finds first line containing the label, scans the portion of that line after the label for YYYY-MM-DD, returns DD/MM/YYYY. Same-line only — no fall-through to next line.
Implementation pieces:

METHOD_CHOICES gains ("email_date", "Email Date").
New extract_by_email_date engine method: case-insensitive label search, regex for \b(\d{4}-\d{1,2}-\d{1,2})\b, validates via datetime.strptime("%Y-%m-%d") so invalid dates (month 13, Feb 31) are rejected, returns DD/MM/YYYY via strftime.
extract_by_rule dispatches to the new method.
UI/save/load: no changes needed — Email Date falls into the existing else branches for single-config-string methods. The save format is just "Date:" (no ||).

On your literal example. Date: 2026-05-05 16:57:31+01:00 with config Date: returns 05/05/2026. Tested end-to-end through extract_fields and the JSON export pipeline.
Edge cases handled:

Multiple Date: lines: first with a valid date wins; if first has no date the method continues scanning.
Multiple dates on a single line: first occurrence wins.
Invalid dates rejected via strptime.
Label not found → empty.
Label found but no date on the same line → empty (same-line only, deliberate).

Tests. 22 V63 cases covering the literal .msg header sample, label case-insensitivity, single-digit month/day handling, invalid-date rejection, same-line-only behaviour (no fall-through), multi-occurrence skip-and-continue, dispatch through extract_by_rule, save/load round-trip, and end-to-end through extract_fields + prepare_export_values. All V42-V62 prior suites pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: I seem to be unable to download all?I seem to be unable to download all?8 MayClaude responded: Let me re-present the files so the download links refresh: Files are still there.Let me re-present the files so the download links refresh:Confirm files are still in outputsConfirm files are still in outputsFiles are still there. Let me re-present them — sometimes the download buttons need refreshing:Presented 3 filesPresented 3 filesThree files re-presented:

app.py (~180 KB) — V63 source with all changes V42 through V63
README.md (~41 KB) — full changelog
requirements.txt — Python dependencies

If clicking the file names individually doesn't trigger a download, a few things to try:

Click each file name one at a time, then on the page that opens, look for a download icon (usually a downward arrow) at the top-right of the file preview.
Try a different browser — sometimes corporate/work browsers block downloads from chat interfaces. Edge, Chrome, and Firefox all generally work; Safari can sometimes be restrictive.
Right-click the file name and try "Save link as…" if the regular click doesn't work.
Disable any download blockers / popup blockers that might be active on the page.

If none of those work, let me know what you're seeing (does clicking do nothing, does it show a preview but no download button, does it error?) and I'll try a different approach — for example, I can dump the file contents inline as text so you can copy-paste them into local files manually if it comes to that.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: For some reason some of our documents that we need to map have dates formatted with a gap between the day of the month and the suffix, like this:For some reason some of our documents that we need to map have dates formatted with a gap between the day of the month and the suffix, like this:"9 th April 2026"Can we adapt our date normalisation method to account for this without effecting any of the current mappings? We don't want this new logic to break any of the existing normalisation for date formats8 MayClaude responded: Quick check before I implement — let me look at the current ordinal-stripping regex: Good — the V61 ordinal-stripping happens at the normaliser level, not the …Quick check before I implement — let me look at the current ordinal-stripping regex:Find current ordinal regexFind current ordinal regexGood — the V61 ordinal-stripping happens at the normaliser level, not the extraction level. Let me trace exactly what's happening. The V61 regex was r"(\d+)(st|nd|rd|th)\b" — requires the digit and suffix to be adjacent. So "9 th April 2026" (with the space between 9 and th) wouldn't match — the th survives, then strptime rejects the resulting "9 th April 2026".
Let me first verify this is the issue:Confirm the bugConfirm the bugConfirmed. All three space-separated ordinal cases fail — the original string is returned unchanged. Now I need to think about how to fix this without breaking anything.
The current regex is (\d+)(st|nd|rd|th)\b. I want to also match (\d+)\s+(st|nd|rd|th)\b. I could change (st|nd|rd|th) to \s*(st|nd|rd|th), but that has a risk:
Risk to think about: a value like "31 st" currently doesn't match the regex (because st isn't followed by another non-word boundary if there's no space). Wait, \b is a word boundary, so "31 st" with the new \s* regex would match — replacing 31 st with 31. That's fine.
But what about "9 the next day"? The regex would try to match \d+\s*(st|nd|rd|th)\b. The 9 the next day has 9 then  the — does \s*(th) match the leading whitespace and then the th of the? Let me check: 9 (digit), then \s* matches   (one space), then th — but \b after th needs a word boundary. In 9 the next day, the position after th is e which is a word character, so \b fails there. Good — false match avoided.
What about "5 thousand"? 5 matches \d+,   matches \s*, th matches, but then \b needs a word boundary — after th is o which is a word character, so \b fails. Good.
One more risk: what if the user types something deliberately containing 9 th as part of a non-date value (in a different field that this normaliser doesn't apply to)? It only applies to *_date fields via prepare_export_values, so this can't leak into other fields. Safe.
The fix is a one-character change in the regex: add \s* between the digits and the suffix. Let me also think about \s+ vs \s* — the existing case (27th, no space) needs to keep working, so it has to be \s* (zero-or-more), not \s+ (one-or-more).
Implementing:Ran 2 commands, edited a fileRan 2 commands, edited a fileThe user's exact case now works (9 th April 2026 → 09/04/2026). All V61 cases still pass. Both potential false-positive cases (5 thousand, 9 the next) are correctly left as-is.
Now write proper tests and run the full regression:Created a file, ran a commandCreated a file, ran a commandAll 27 V64 cases pass. Run the full regression sweep — V61 in particular is the one I want to confirm hasn't broken:Full regression sweepFull regression sweepAll clean. V61 in particular passes — every existing format still works, only the new spaced-ordinal case is the new addition. Update README and finalise:Update README and copy V64 outputsUpdate README and copy V64 outputsHmm, transient I/O error. Let me retry:Retry copy and verifyRetry copy and verifyBoth files now match. Present them:Presented 3 filesPresented 3 filesV64 done. One-character fix to the ordinal-stripping regex:
Before: r"(\d+)(st|nd|rd|th)\b" — required the digit and suffix to be adjacent.
After: r"(\d+)\s*(st|nd|rd|th)\b" — allows zero or more whitespace characters between them.
Why it's safe: The \b word boundary after the suffix prevents false matches. 5 thousand doesn't match because after th is the word-character o (no boundary). 9 the next day doesn't match for the same reason. 2 names doesn't match because after n is a. The regex also only runs inside normalise_date_value, which only fires on the three *_date fields at JSON export time — so even an unlikely false-positive could only affect dates, never other content.
Verified end-to-end on your case. 9 th April 2026 now becomes 09/04/2026 in the JSON export. Same for 1 st January 2026, 21 ST April 2026, 27 th April 2026, and even pathological cases like 9   th April 2026 (multiple spaces) and 9\tth April 2026 (tab).
Tests. 27 V64 cases covering the new spaced-ordinal forms, V61 regression cases (joined ordinals, all the original formats), false-positive guards (5 thousand, 9 the next, etc.), and an end-to-end run through prepare_export_values. All 22 prior test suites (V42-V63) still pass.AppPY DownloadReadmeDocument · MD DownloadRequirementsTXT DownloadDownload allYou said: Is it okay for the portable .Is it okay for the portable .exe to live in the "CE Document Mapper" folder in "Documents" of the users with OneDrive? I think this is already the case for one user and doesnt seem to be causing issues.11 MayClaude responded: Short answer: yes, it's fine in practice — but it's worth knowing the tradeoffs so you can decide deliberately.Short answer: yes, it's fine in practice — but it's worth knowing the tradeoffs so you can decide deliberately.
Let me think through what actually happens when the .exe lives inside the folder it also reads/writes:
What goes well.
The .exe is read-only at runtime — PyInstaller extracts its bundled files to a temp directory (sys._MEIPASS) and runs from there, not from the .exe location. The Documents\CE Document Mapper\ folder is only ever touched for providers.json and app_settings.json (read/write) — both of which are separate from the .exe file. So the .exe and the config files coexist happily, no file locking, no conflicts.
The fact that one user is already running it from there and it works is a meaningful signal — those are exactly the failure modes that would show up immediately.
What's worth knowing.
OneDrive syncs the entire folder, including the .exe. There are some practical implications:

Sync time on the first save. An ~80-100 MB .exe will take a few seconds to upload to OneDrive's servers the first time. After that, it's a single file sitting in OneDrive — barely a network cost.
Sync conflicts if the user opens the app on multiple machines. If the user runs the .exe on their work PC and you push an updated .exe later, OneDrive will sync the new version to their other machines too. That's actually a feature in your case — when you ship V65, anyone with OneDrive on multiple machines gets the update everywhere automatically. The user just needs to make sure the .exe isn't running on the other machine at sync time (because Windows locks running .exe files, and OneDrive can't overwrite a locked file — it'll usually queue and retry).
Quota cost. ~80-100 MB consumed from each user's OneDrive quota. Trivial for most users (1 TB plans), worth a thought if someone's near a 5 GB free-tier limit.
Antivirus suspicion. Sometimes corporate AV is more wary of .exe files in user-writable locations than in C:\Program Files\. This is the most likely source of any actual trouble, but it's the AV's behaviour not OneDrive's — and your existing user clearly isn't hitting it.
First-run seeding. The bundled providers.json gets copied to Documents\CE Document Mapper\providers.json on first launch. If the .exe is also there, both files end up sitting next to each other. The app reads providers.json by path (Documents\CE Document Mapper\providers.json) — same file, no confusion.

What would actually be problematic.
The only scenarios that genuinely cause trouble are:

Putting the .exe in C:\Program Files\ (requires admin install and special handling).
Two users sharing the same .exe on a network drive (file locking issues).
Running the .exe from a OneDrive folder while OneDrive is trying to download an updated version (locks).

None of those apply to your setup.
My recommendation. Don't change anything. The current arrangement (user copies the .exe into Documents\CE Document Mapper\) is fine. If you want one small bit of insurance, you could tell users to pin a shortcut to the Start menu / Desktop so they don't accidentally try to "delete" the .exe thinking it's clutter — but even that's optional.