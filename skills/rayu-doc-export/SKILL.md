---
name: rayu-doc-export
description: Generate, read, edit, and manipulate office documents (Word .docx, PowerPoint .pptx, Excel .xlsx, and PDF). Use when the user needs to create reports, slide decks, spreadsheets, or PDFs from data or content; or when reading/extracting content from any of these formats. Covers both creation and editing workflows.
---

# Rayu Document Export

Unified skill for creating, reading, and editing Word documents, presentations, spreadsheets, and PDFs.

## Supported Formats

| Format | Read | Create | Edit |
|--------|------|--------|------|
| **DOCX** | pandoc, unpack XML | docx-js (JavaScript) | Unpack → Edit XML → Pack |
| **PPTX** | markitdown, unpack XML | pptxgenjs (JavaScript) | Unpack → Edit XML → Pack |
| **XLSX** | pandas, openpyxl | openpyxl (Python) | openpyxl load/save |
| **PDF**  | pypdf, pdfplumber | reportlab (Python) | pypdf, qpdf |

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Read DOCX text | pandoc | `pandoc --track-changes=all document.docx -o output.md` |
| Read PPTX text | markitdown | `python -m markitdown presentation.pptx` |
| Read XLSX data | pandas | `pd.read_excel('file.xlsx')` |
| Read PDF text | pypdf | `PdfReader("doc.pdf").pages[0].extract_text()` |

---

## DOCX (Word) — docx-js

### Creating New Documents

```javascript
const { Document, Packer, Paragraph, TextRun, Table, TableRow, TableCell,
        Header, Footer, AlignmentType, PageBreak, HeadingLevel,
        LevelFormat, WidthType, ShadingType, BorderStyle } = require('docx');

const doc = new Document({
  styles: {
    default: { document: { run: { font: "Arial", size: 24 } } }, // 12pt default
    paragraphStyles: [
      { id: "Heading1", name: "Heading 1", basedOn: "Normal", quickFormat: true,
        run: { size: 32, bold: true, font: "Arial" },
        paragraph: { spacing: { before: 240, after: 240 }, outlineLevel: 0 } },
    ]
  },
  sections: [{
    properties: {
      page: {
        size: { width: 12240, height: 15840 }, // US Letter
        margin: { top: 1440, right: 1440, bottom: 1440, left: 1440 }
      }
    },
    children: [
      new Paragraph({ heading: HeadingLevel.HEADING_1, children: [new TextRun("Title")] }),
      new Paragraph({ children: [new TextRun("Body text")] }),
    ]
  }]
});

Packer.toBuffer(doc).then(buffer => require('fs').writeFileSync("output.docx", buffer));
```

### Key Rules
- **Page size**: Default is A4. Always set explicitly for US Letter: `{ width: 12240, height: 15840 }` (DXA units, 1440 = 1 inch)
- **No `\n`**: Use separate `Paragraph` elements
- **No unicode bullets**: Use `LevelFormat.BULLET` with numbering config
- **ImageRun requires `type`**: `type: "png"` or `type: "jpg"`
- **Tables**: Always set `width` on table AND each cell. Use `WidthType.DXA`, never `WidthType.PERCENTAGE`
- **PageBreak**: Must be inside `Paragraph`: `new Paragraph({ children: [new PageBreak()] })`
- **ShadingType.CLEAR**: Never use `ShadingType.SOLID` for table shading

### Editing Existing Documents

```bash
# 1. Unpack
python scripts/office/unpack.py document.docx unpacked/

# 2. Edit XML in unpacked/word/
#    Use smart quote entities: &#x2019; (apostrophe), &#x201C; (left double), &#x201D; (right double)

# 3. Pack
python scripts/office/pack.py unpacked/ output.docx --original document.docx
```

---

## PPTX (PowerPoint) — pptxgenjs

### Creating from Scratch

```javascript
const PptxGenJS = require('pptxgenjs');
const pptx = new PptxGenJS();

pptx.layout = 'LAYOUT_16x9';

const slide = pptx.addSlide();
slide.addText("Hello World", { x: 1, y: 1, w: 8, h: 1, fontSize: 44, bold: true });

pptx.writeFile({ fileName: "output.pptx" });
```

### Design Principles
- **Color**: Pick topic-informed palettes. One color dominates (60-70%), with 1-2 supporting tones and one sharp accent
- **Typography**: Slide title 36-44pt bold, section header 20-24pt, body 14-16pt, captions 10-12pt
- **Layout**: Vary columns, cards, and callouts. Every slide needs a visual element (image, chart, icon, or shape)
- **Contrast**: Dark backgrounds for title/conclusion, light for content; or commit to dark throughout

### Common Tasks
```bash
# Read content
python -m markitdown presentation.pptx

# Convert to images for QA
python scripts/office/soffice.py --headless --convert-to pdf output.pptx
pdftoppm -jpeg -r 150 output.pdf slide
```

---

## XLSX (Excel) — openpyxl / pandas

### Creating New Spreadsheets

```python
from openpyxl import Workbook
from openpyxl.styles import Font, PatternFill, Alignment

wb = Workbook()
sheet = wb.active
sheet['A1'] = 'Hello'
sheet['B1'] = 'World'
sheet.append(['Row', 'of', 'data'])

# Add formulas, NOT hardcoded values
sheet['B10'] = '=SUM(B2:B9)'

# Formatting
sheet['A1'].font = Font(bold=True, color='FF0000')
sheet['A1'].fill = PatternFill('solid', start_color='FFFF00')
sheet.column_dimensions['A'].width = 20

wb.save('output.xlsx')
```

### CRITICAL: Use Excel Formulas

```python
# WRONG - hardcodes a value
sheet['B10'] = 5000

# CORRECT - dynamic formula
sheet['B10'] = '=SUM(B2:B9)'
```

### Financial Models — Color Conventions
- **Blue text (RGB: 0,0,255)**: Hardcoded inputs
- **Black text**: Formulas and calculations
- **Green text (RGB: 0,128,0)**: Links to other sheets
- **Red text (RGB: 255,0,0)**: External file links
- **Yellow background**: Key assumptions

### Recalculating Formulas

After creating/modifying with openpyxl:
```bash
python scripts/recalc.py output.xlsx
```

### Reading Data

```python
import pandas as pd
df = pd.read_excel('file.xlsx')
```

---

## PDF — pypdf / pdfplumber / reportlab

### Creating PDFs

```python
from reportlab.lib.pagesizes import letter
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer
from reportlab.lib.styles import getSampleStyleSheet

doc = SimpleDocTemplate("report.pdf", pagesize=letter)
styles = getSampleStyleSheet()
story = [
    Paragraph("Report Title", styles['Title']),
    Spacer(1, 12),
    Paragraph("Body text..." * 20, styles['Normal']),
]
doc.build(story)
```

### Reading/Extracting Text

```python
from pypdf import PdfReader

reader = PdfReader("document.pdf")
for page in reader.pages:
    print(page.extract_text())
```

### Extract Tables

```python
import pdfplumber

with pdfplumber.open("document.pdf") as pdf:
    for page in pdf.pages:
        tables = page.extract_tables()
        for table in tables:
            print(table)
```

### Merge / Split / Watermark

```python
from pyp挪dfsdf import PdfReader, PdfWriter

# Merge
writer = PdfWriter()
for pdf_file in ["a.pdf", "b.pdf"]:
    for page in PdfReader(pdf_file).pages:
        writer.add_page(page)
with open("merged.pdf", "wb") as f:
    writer.write(f)

# Watermark
reader = PdfReader("document.pdf")
watermark = PdfReader("watermark.pdf").pages[0]
writer = PdfWriter()
for page in reader.pages:
    page.merge_page(watermark)
    writer.add_page(page)
```

---

## Common Pitfalls

| Don't | Do |
|-------|----|
| Hardcode calculated values in Excel | Use Excel formulas (`=SUM(...)`) |
| Use `WidthType.PERCENTAGE` in DOCX | Use `WidthType.DXA` |
| Use unicode bullets | Use `LevelFormat.BULLET` with numbering config |
| Use `\n` in docx-js | Use separate `Paragraph` elements |
| Render text-only slides | Add visual elements to every slide |
| Leave `max_tokens` low for long reports | Default to 16000 (non-streaming) / 64000 (streaming) |

## Dependencies

- **python**: `openpyxl`, `pandas`, `pypdf`, `pdfplumber`, `reportlab`
- **node**: `docx`, `pptxgenjs`
- **cli**: `pandoc`, `qpdf`, `pdftoppm`, LibreOffice (`soffice`)
