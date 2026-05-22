# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

**Run the app:**
```
python app.py
```

**Install dependencies:**
```
pip install -r requirements.txt
```

**Build the Windows .exe:**
```
pyinstaller --noconsole --onefile --icon=ce_document_mapper.ico ^
    --add-data "ce_document_mapper.ico;." ^
    --add-data "providers.json;." ^
    --add-data "tesseract;tesseract" ^
    --collect-all PIL ^
    app.py
```
Or use the existing `app.spec` file: `pyinstaller app.spec`

There are no automated tests. Verification is done by running the app and testing manually.

## Architecture

The entire application lives in a single file: `app.py` (~4250 lines). There is no package structure.

### Two main classes

**`ExtractionEngine`** (line 407) — all document I/O and data extraction:
- `extract_text()` — dispatches to format-specific extractors (PDF via PyMuPDF→pypdf fallback, DOCX, DOC via Word COM→LibreOffice→antiword, EML, MSG via extract_msg). OCR fallback fires on PDFs with zero text and ≤`OCR_PAGE_LIMIT` (2) pages via pytesseract.
- `detect_provider()` — matches document text against each provider's `detect_phrases`. **All** phrases must be present (AND logic) for a provider to match.
- `extract_fields()` — applies per-field mapping rules from the matched provider config.
- `extract_by_rule()` — dispatches to method-specific extractors (see mapping methods below).

**`App`** (line 2693) — the Tkinter GUI:
- Three-panel layout: left (Detected Fields), middle (Source Preview), right (Provider Setup/mapping editor)
- Delegates all extraction logic to `self.engine` (an `ExtractionEngine` instance)
- Handles single-file and batch-file drag-and-drop via tkinterdnd2
- Supports an "Engineer Report" overlay where a second document's extracted values overwrite the instruction's values

### Key data structures

**`DEFAULT_FIELDS`** (line 67) — the 13 fixed field keys and display labels. Field order here controls display order everywhere.

**`providers.json`** — stored in `Documents\CE Document Mapper\` (Shell-resolved to handle OneDrive redirection). Contains a list of provider preset objects, each with:
- `name` — display name
- `detect_phrases` — list of strings that must ALL appear in the document
- `field_rules` — dict of field key → `{"method": "<code>", "config": "<string>"}`
- `use_current_date_for_inspection_date`, `force_postcode_for_inspection_address`, `engineer_report` — booleans

**`DocumentSession`** (line 396) — holds extracted text, matched provider, field values, and notes for one imported document.

### Mapping methods

Seven methods, stored as string codes in `field_rules[field]["method"]`:

| Code | UI Name | Config format |
|------|---------|---------------|
| `single_label` | Single Label | comma-separated label variants |
| `two_labels` | Two Labels | `start_label \|\| end_label` |
| `fixed_position` | Fixed Position | 1-based line number |
| `fixed_position_label` | Fixed Position + Label | `line_number \|\| label` |
| `single_label_offset` | Single Label +/- | `label \|\| +N` or `label \|\| -N` (skips blank lines) |
| `email_date` | Email Date | label; extracts first `YYYY-MM-DD` on that line → `DD/MM/YYYY` |
| `manual_input` | Manual Input | literal value |

Two fields bypass the method dropdown entirely — they use a token presence-check instead:
- `vat_status` → `Yes` / `No` / blank
- `mileage_unit` → `Miles` / `Km` / blank

### Export pipeline

- **JSON export** (`export_json_string`): serialises all field values to Desktop. Gated on `work_provider` being non-empty. Date fields are canonicalised to `DD/MM/YYYY` by `normalise_date_value()`. Inspection Address always exports as 6 newline-separated lines.
- **DOCX export** (`build_rjs_docx`): produces an RJS-format Word document. Only available for the RJS provider.
- **Image export** (`extract_images_to_desktop`): extracts embedded images from the source document. Not gated on work provider.

### Storage and paths

- `APP_DATA_DIR` = Shell-resolved `Documents\CE Document Mapper\` — uses `SHGetKnownFolderPath` via ctypes to handle OneDrive redirection (V62).
- `OUTPUT_DIR` = Shell-resolved Desktop.
- A one-time migration at startup copies files from the legacy `Path.home()/Documents` location to the Shell-resolved location.
- A bundled `providers.json` next to the `.exe` seeds first-run users; subsequent launches use whatever is in Documents.

### Tesseract OCR

The bundled `tesseract/` folder (next to `app.py`) must contain `tesseract.exe` and `tessdata/eng.traineddata`. `configure_bundled_tesseract()` is called at startup to point `pytesseract` at this binary. OCR fires only when a PDF has zero text characters AND exactly one image per page AND ≤2 pages.
