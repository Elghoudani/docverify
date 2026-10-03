# docverify

**Finds the edit in a payslip, shows where it is, and explains it to an analyst.**

[Model on Hugging Face](https://huggingface.co/ghostsas001/docverify) ·
[YOLO field detector](https://huggingface.co/ghostsas001/docverify-yolo-fields)

![A rewritten amount the image model misses — the sums don't](https://huggingface.co/ghostsas001/docverify/resolve/main/figures/ex_doc01788.jpg)

Editing a payslip takes two minutes with a free PDF editor. docverify checks every page with
**five independent kinds of evidence** — what the document is, where its fields are, which pixels
were edited, whether the sums still hold, and what the PDF file itself reveals — and returns one
risk level with the reasons in plain French or English.

All data is **synthetic** (generated French payslips, fake people). No real document was used.

## How one document is checked

Each check catches what the others miss. Fusion keeps them honest: any strong signal makes the
risk high, weak signals are only noted, and an unreadable value is "unknown" — never "fraud".

![How one document is checked](https://huggingface.co/ghostsas001/docverify/resolve/main/figures/pipeline.png)

| | Check | What it does |
|---|---|---|
| 1 · Router | **SigLIP** | Frozen vision-language model + a linear probe: payslip, RIB or invoice. 100% on the test documents (zero-shot: 68%). |
| 2 · Fields | **YOLO** | Finds the ten fields — net pay, gross, base salary, IBAN, SIRET… — so an edit is pinned to *net pay*, not to pixels. mAP50-95 0.975. |
| 3 · Pixels | **TamperNet** | ConvNeXt on the image, SRM noise residuals, and the JPEG 8×8 grid: copy-move, splice and re-typed digits leave traces there. |
| 4 · Sums | **Seven payslip rules** | Hours × rate = base, base + extras = gross, gross − contributions = net… on OCR, or exactly on a PDF's text layer. |
| 5 · File | **PDF forensics** | An old amount still hiding under a white box, a different font, a second revision, an online editor as producer. |
| 6 · Words | **Granite note** | A local IBM Granite model writes the analyst's note — it chooses facts and wording, but every figure comes from templates. |

## Results

| | |
|---|---|
| Forgeries caught — image **+ arithmetic** | **95%** at **1.7%** false alarms |
| Edited area found (pixel F1) | **81%** |
| Tamper model ROC-AUC | **0.902** |
| Arithmetic false alarms on genuine payslips | **0%** |
| Edited PDFs caught by a strong file signal | **100%**, 0% on genuine PDFs |
| Field detection (YOLO, mAP50-95) | **0.975** |
| Document type (SigLIP) | **100%** |
| Numbers invented by the analyst note | **0**, by construction |
| Speed on 4 CPU threads | **6.7 s / page** |

Test set: 793 payslips never seen in training (390 genuine).

![Image only vs image plus arithmetic](https://huggingface.co/ghostsas001/docverify/resolve/main/figures/03_image_vs_combined.png)

![ROC and scores](https://huggingface.co/ghostsas001/docverify/resolve/main/figures/04_roc_and_scores.png)

![Pixel localisation](https://huggingface.co/ghostsas001/docverify/resolve/main/figures/05_pixel_localisation.png)

<table><tr>
<td><img src="https://huggingface.co/ghostsas001/docverify/resolve/main/figures/02_v1_vs_v2.png" alt="v1 vs v2"></td>
<td><img src="https://huggingface.co/ghostsas001/docverify/resolve/main/figures/07_maths_by_field.png" alt="Arithmetic by field"></td>
</tr><tr>
<td><img src="https://huggingface.co/ghostsas001/docverify/resolve/main/figures/08_pdf_forensics.png" alt="PDF forensics"></td>
<td><img src="https://huggingface.co/ghostsas001/docverify/resolve/main/figures/10_risk_levels.png" alt="Risk levels"></td>
</tr><tr>
<td><img src="https://huggingface.co/ghostsas001/docverify/resolve/main/figures/09_yolo_siglip.png" alt="YOLO and SigLIP"></td>
<td><img src="https://huggingface.co/ghostsas001/docverify/resolve/main/figures/15_throughput.png" alt="Throughput"></td>
</tr></table>

## Beyond payslips

The same checks apply to tax notices, bank statements, invoices, repair quotes, medical bills and
rental files — in banking, insurance and beyond.

## Limits

- Every number is measured on synthetic payslips, not on real documents.
- A forger who edits every amount consistently escapes the arithmetic; the image model is then the only check.
- The PDF forgeries were made by us, so the 100% is optimistic.

## License

The tamper model is Apache-2.0. The YOLO field detector is derived from Ultralytics YOLO and stays
under AGPL-3.0 in [its own repository](https://huggingface.co/ghostsas001/docverify-yolo-fields).
