---
name: rayu-doc-export
description: Generate office documents and PDFs (docx, xlsx, pptx, pdf) programmatically using open-source libraries. Use when a task needs to produce a Word, Excel, PowerPoint, or PDF deliverable from data or content.
---

# Rayu Doc Export

Produce real office files with well-maintained open-source libraries. Pick the library by output format and ecosystem, install it, generate the file, and verify it opens. All guidance below is built on the libraries' own public documentation.

## Library map

| Format | Python (recommended) | JavaScript/Node |
|---|---|---|
| **Word** `.docx` | `python-docx` | `docx` (npm) |
| **Excel** `.xlsx` | `openpyxl` | `exceljs` |
| **PowerPoint** `.pptx` | `python-pptx` | `pptxgenjs` |
| **PDF** | `reportlab` (or `weasyprint` for HTML→PDF) | `pdf-lib` |

Python has the most mature OOXML coverage; use it unless the project is JS-only.

## Word — python-docx

```bash
pip install python-docx   # NOTE: package is "python-docx"; import is "docx"
```

```python
from docx import Document
from docx.shared import Pt, Inches

doc = Document()
doc.add_heading('Quarterly Report', level=1)
doc.add_paragraph('Summary of results for Q1.')
doc.add_paragraph('A bullet item', style='List Bullet')

table = doc.add_table(rows=1, cols=2)
table.style = 'Light Grid Accent 1'
hdr = table.rows[0].cells
hdr[0].text, hdr[1].text = 'Metric', 'Value'
row = table.add_row().cells
row[0].text, row[1].text = 'Revenue', '$1.2M'

doc.add_picture('chart.png', width=Inches(5))
doc.save('report.docx')
```

Key objects: `Document`, `add_heading`, `add_paragraph` (+ `style`), `add_table`/`add_row`, `add_picture`. Runs (`paragraph.add_run('x').bold = True`) control inline formatting.

## Excel — openpyxl

```bash
pip install openpyxl
```

```python
from openpyxl import Workbook
from openpyxl.styles import Font

wb = Workbook()
ws = wb.active
ws.title = 'Sales'
ws.append(['Month', 'Revenue'])          # write a row
ws['A1'].font = Font(bold=True)
ws.append(['Jan', 1200])
ws['B3'] = '=SUM(B2:B2)'                  # formulas are just strings
ws.column_dimensions['A'].width = 16
wb.save('sales.xlsx')
```

Key objects: `Workbook`, `wb.active`/`wb.create_sheet`, cell access (`ws['A1']` or `ws.cell(row, column)`), `ws.append(list)`, styles (`Font`, `PatternFill`, `Alignment`), and `wb.save`. For huge sheets use `write_only=True`.

## PowerPoint — python-pptx

```bash
pip install python-pptx
```

```python
from pptx import Presentation
from pptx.util import Inches, Pt

prs = Presentation()
title_slide = prs.slides.add_slide(prs.slide_layouts[0])  # 0 = Title
title_slide.shapes.title.text = 'Rayu Deck'
title_slide.placeholders[1].text = 'Generated with python-pptx'

bullets = prs.slides.add_slide(prs.slide_layouts[1])      # 1 = Title+Content
bullets.shapes.title.text = 'Agenda'
tf = bullets.placeholders[1].text_frame
tf.text = 'First point'
tf.add_paragraph().text = 'Second point'

prs.save('deck.pptx')
```

Key objects: `Presentation`, `slide_layouts` (indexes into the template's layouts), `shapes.title`/`placeholders`, `text_frame` + `add_paragraph`, and `shapes.add_textbox`/`add_picture` for free placement.

## PDF — reportlab (Python) or pdf-lib (JS)

**reportlab, low-level canvas** (precise placement):

```python
from reportlab.pdfgen import canvas
from reportlab.lib.pagesizes import letter

c = canvas.Canvas('out.pdf', pagesize=letter)
c.drawString(72, 720, 'Hello World')   # origin is bottom-left
c.showPage()                            # finish the page
c.save()
```

**reportlab, Platypus** (flowing documents — paragraphs, tables, page breaks):

```python
from reportlab.platypus import SimpleDocTemplate, Paragraph, Table
from reportlab.lib.styles import getSampleStyleSheet

styles = getSampleStyleSheet()
doc = SimpleDocTemplate('report.pdf')
doc.build([
    Paragraph('Quarterly Report', styles['Title']),
    Paragraph('Summary text.', styles['BodyText']),
    Table([['Metric', 'Value'], ['Revenue', '$1.2M']]),
])
```

**pdf-lib (Node/browser):**

```js
import { PDFDocument, StandardFonts } from 'pdf-lib'

const pdf = await PDFDocument.create()
const page = pdf.addPage([595, 842])           // A4 in points
const font = await pdf.embedFont(StandardFonts.Helvetica)
page.drawText('Hello World', { x: 72, y: 770, size: 18, font })
const bytes = await pdf.save()                  // Uint8Array → write to disk
```

Use Platypus/reportlab when generating from scratch; if you already have HTML/CSS, `weasyprint` (HTML→PDF) is often faster to a polished result.

## Common pitfalls

- **Wrong Word package:** install `python-docx` (not the abandoned `docx`); the import is still `import docx` / `from docx import Document`.
- **reportlab coordinates** start at the bottom-left, and `showPage()` + `save()` are required or the file is empty/corrupt.
- **pptx layout indexes** depend on the template; enumerate `prs.slide_layouts` rather than guessing.
- **openpyxl formulas** are stored as strings and only evaluated when the file is opened in Excel (openpyxl won't compute them).
- **Fonts/images:** embed fonts and reference image paths that exist at generate time; verify the output opens before reporting done.

## Working in the rayu swarm

Use this for any "produce a report/spreadsheet/deck/PDF" deliverable. Match the data to the right library, generate into the project's output path, and confirm the file is valid. These are open-source libraries — no proprietary document engine is required.
