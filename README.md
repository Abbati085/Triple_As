# Triple_As
Our AI system detects corrosion in metal images, marking affected areas, estimating severity, and suggesting maintenance. An LLM interface creates reports, and a Roboflow model shows live detections with confidence scores. Tests distinguished corroded from unaffected stainless steel, supporting early inspection and preventive maintenance.
CorroLLM — Image-Based Corrosion Screening and Automated Reporting
CorroLLM is a Google Colab notebook for exploratory, image-based corrosion screening. Upload a photograph of a metal surface, select its material, and generate an annotated image plus editable Word and PDF inspection summaries.
> **Important:** The current notebook uses OpenCV contours and hand-written heuristics. It does **not** run a trained corrosion detection model or an LLM-based interpretation pipeline. Its labels, confidence values, affected-area estimates, root-cause suggestions, and cost estimates are preliminary outputs, **not validated measurements or engineering diagnoses**. Do not use them alone to make safety-critical maintenance decisions.
Quick start
Download or clone this repository.
Open `CorroLLM_Explained_From_User_Code.ipynb` in Google Colab. You can upload the `.ipynb` file using File → Upload notebook.
Select Runtime → Run all, or run the code cells sequentially from top to bottom. The first code cell installs dependencies automatically (internet access is required).
When prompted in the final interactive cell, upload one image showing the surface to inspect.
Enter a material number from `1` to `5`, then enter an optional inspection location.
Review the annotated image and printed results. The notebook generates and attempts to download a `.docx` and a `.pdf` report.
No API key is required for the default workflow. Use Google Colab rather than a plain local Jupyter installation unless you adapt the Colab-specific upload/download code and `/content/` paths.
Supported material selections
Input	Material
`1`	Carbon steel
`2`	Stainless steel (304/316)
`3`	Aluminum alloy
`4`	Galvanized steel
`5`	Copper/brass
An invalid selection defaults to carbon steel. Material selection provides context for the generated narrative; it does not configure a material-specific image recognition model.
How the notebook works
```text
Upload surface image
       ↓
Load image with OpenCV; resize if necessary
       ↓
Grayscale conversion → Canny edge detection → external contours
       ↓
Filter candidate contours and assign heuristic labels/severity
       ↓
Draw color-coded bounding boxes and legend
       ↓
Generate template-based summary, possible causes and recommendations
       ↓
Export DOCX + PDF; initiate Colab downloads
```
The notebook is organized into these sections:
Notebook section	Purpose
Block 0 — Environment Setup	Installs dependencies and initializes the report directory
Block 1 — Image Acquisition	Explains image upload and image loading
Block 2 — Corrosion Analysis Engine	Defines `CorrosionAnalyzer`, candidate detection, severity heuristics and overlays
Block 3 — Automated Reporting Layer	Defines `ReportGenerator` for Word/PDF output
Block 4 — Knowledge Base and Reasoning Logic	Defines preset recommendations and template-based root-cause summaries
Block 5 — Main End-to-End Analysis Function	Defines `analyze_and_generate_report()`
Block 6 — Interactive Colab Run Cell	Uploads an image, collects material/location and downloads reports
The markdown headings and code-cell numbering differ slightly; execute the actual code cells in order.
Dependencies
The notebook installs:
```bash
pip install opencv-python-headless pillow numpy python-docx reportlab anthropic
```
It also imports `google.colab` for upload/download controls and IPython for displaying images. `anthropic` is installed and an optional Claude client can be initialized in `CorrosionAnalyzer`, but the supplied end-to-end function does not pass an API key or call Claude. Thus, Anthropic is not required for the analysis actually performed.
Inputs
Image: a surface photograph readable by OpenCV. Use a clear, well-lit image with the region of interest visible; the notebook does not currently enforce a specific file extension.
Material: one of the five menu choices above.
Location: optional free-text metadata included in the report.
The code resizes images whose longest dimension exceeds 2048 pixels. It processes the first uploaded file if multiple files are selected.
Outputs
The notebook creates `/content/reports/` and writes:
```text
/content/reports/
├── overlay.jpg
├── corrosion_report_YYYYMMDD_HHMMSS.docx
└── corrosion_report_YYYYMMDD_HHMMSS.pdf
```
`overlay.jpg` is overwritten by each run; reports have timestamp-based names. Both report formats contain an executive summary, candidate findings, suggested root causes, severity-linked recommendations and preventive measures. The annotated image is displayed in Colab but is not automatically embedded in the reports.
The notebook prints candidate corrosion types, severity categories (`low`, `moderate`, `high`, `critical`), heuristic confidence scores, and estimated affected-area percentages. The overlay uses green, yellow, orange and red for these severity categories, respectively.
Understanding the analysis results
The detector converts the image to grayscale, applies Canny edge detection (`50`, `150` thresholds), and searches for external contours. It considers up to eight large contours, discards contours smaller than 0.8% of the image area, and returns at most five detections in the result list.
Classification uses contour area and grayscale intensity. Some corrosion-type labels are randomly selected among `crevice`, `galvanic` and `stress_cracking` when earlier rules do not match. Consequently, identical inputs can yield different labels. The numeric `confidence` is a hand-crafted formula, not a calibrated probability.
If no qualifying contour is found, the code deliberately inserts a fallback detection with a nominal 20% affected area and 0.72 confidence. This means a reported detection does not establish that corrosion exists. Area percentages are contour-based approximations, not verified surface-loss measurements; the reported total may also include candidates omitted from the displayed five.
Root-cause explanations, priorities, cost ranges and timelines are predefined templates. They are not derived from chemical analysis, ultrasonic thickness measurements, electrochemical testing, or a verified failure investigation.
Troubleshooting
Problem	What to check
`ModuleNotFoundError: google.colab`	Open the notebook in Google Colab, or adapt the Colab-specific code for local Jupyter.
`NameError` for an analyzer or report function	Run all earlier code cells in order.
`Could not load image`	Upload a valid, supported image; check for corruption or an unsupported format.
No download prompt	Look in the Colab file browser under `/content/reports/` and download the files manually.
Implausible labels or percentages	These are heuristic screening outputs. Confirm observations independently; consider lighting, reflections, scale and surface condition.
Package installation fails	Check Colab internet connectivity, restart the runtime if needed, and rerun the notebook.
Limitations and responsible use
Not trained or validated on a labeled corrosion image dataset.
Cannot reliably distinguish corrosion mechanisms from appearance alone.
No measured depth of pitting, wall-thickness loss, corrosion rate, or remaining service life.
No image-scale calibration or ground-truth segmentation.
Detection is sensitive to illumination, texture, coatings and non-corrosion edges.
Fallback detections and random type assignment can produce false or inconsistent findings.
Recommendations and budget ranges are generic placeholders, not site-specific engineering estimates.
Intended use: demonstrations, prototyping, education, and preliminary visual review. For operational asset-integrity decisions, involve a qualified corrosion/inspection professional and appropriate validated inspection methods.
Suggested future improvements
Add a labeled dataset, expert-verified annotations and reproducible evaluation metrics (precision, recall, F1, IoU).
Replace random labels and unconditional fallback detections with validated classification and a genuine no-corrosion/uncertain outcome.
Add corrosion-specific segmentation, calibrated area measurements and controlled imaging guidance.
Integrate an LLM only where useful, clearly distinguishing model-generated explanations from measured evidence.
Add unit tests, pinned dependencies, reproducible examples, and an explicit software license.
Repository layout
```text
.
├── README.md
└── CorroLLM_Explained_From_User_Code.ipynb
```
License and citation
No software license or formal publication citation is specified in the supplied notebook. Add a `LICENSE` file and your preferred citation information before inviting external reuse. Without a license, public visibility on GitHub does not automatically grant broad permission to reuse or redistribute the code.
