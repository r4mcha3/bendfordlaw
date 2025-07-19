# Benford's Law Analyzer Web App

## Overview
[This web application](https://r4mcha3.github.io/benfordlaw/) analyzes text files (`.txt`) or images (`.png`, `.jpg`) to check if the distribution of first digits (1-9) follows **Benford's Law**, which can indicate data authenticity or potential manipulation. It supports three modes:
- **Text Analysis**: Uses Optical Character Recognition (OCR) via Tesseract.js to extract numbers from images (e.g., receipts, invoices) or processes text files directly.
- **Raw Image Analysis**: Extracts Discrete Cosine Transform (DCT) coefficients from JPEG images to detect synthetic images (e.g., GAN-generated). Client-side analysis is limited; server-side processing is recommended.
- **Pixel Value Analysis (Experimental)**: Analyzes the first digits of pixel RGB values (0–255). This mode is experimental, as pixel values may not reliably follow Benford’s Law.

The app provides a live bar chart (via Chart.js), a progress bar, a results table, and a statistical analysis using a chi-squared test to detect anomalies.

## What is Benford's Law?
Benford's Law states that in many naturally occurring datasets (e.g., financial records, DCT coefficients), the leading digits follow a non-uniform distribution:
- 1: 30.1%
- 2: 17.6%
- 3: 12.5%
- 4: 9.7%
- 5: 7.9%
- 6: 6.7%
- 7: 5.8%
- 8: 5.1%
- 9: 4.6%

This applies to datasets spanning multiple orders of magnitude but not artificial data (e.g., sequential numbers). Deviations can indicate manipulation.

## App Features
- **File Upload**: Upload `.txt` files or images (`.png`, `.jpg`).
- **Analysis Modes**:
  - Text: Extracts numbers via OCR (images) or directly (text files).
  - Raw Image: Analyzes DCT coefficients from JPEGs.
  - Pixel Values: Analyzes RGB values (experimental).
- **OCR Enhancements**: Preprocesses images with OpenCV.js.
- **Live Tally**: Updates chart with throttled rendering.
- **Progress Bar**: Shows progress (0-100% for text, 0-50% for OCR/DCT/pixel, 50-100% for counting).
- **Results Table**: Displays digit counts and percentages.
- **Statistical Analysis**: Chi-squared test (p-value < 0.05 flags anomalies).
- **Dataset Validation**: Checks Benford compliance.
- **User Guidance**: Help section links to manipulation advice.
- **Responsive Design**: Stable across browsers.

## What to Do If You Suspect Image Manipulation
If the app flags a “Potential Anomaly” (p-value < 0.05), follow these steps:

1. **Verify the Input**:
   - **Text**: Ensure high-contrast, typed images. Check OCR text in console (`OCR extracted text: ...`).
   - **Raw**: Ensure JPEG format. Use server-side API.
   - **Pixel**: Note that pixel analysis is experimental; deviations may reflect image content.
2. **Compare with Original Data**:
   - Analyze original datasets or other image copies.
   - Look for tampering signs (e.g., pixel artifacts).
3. **Test Multiple Samples**:
   - Analyze other files to confirm patterns.
4. **Investigate Context**:
   - Ensure Benford-compliant data. Unsuitable data may cause false positives.
5. **Further Analysis**:
   - Use forensic tools (e.g., FotoForensics).
   - Consult experts for critical data.
6. **Report Findings**:
   - Document results (e.g., screenshot chart).
   - Note preliminary nature of results.

## Getting Started
### Prerequisites
- Modern browser (Chrome, Firefox, Edge).
- Server-side DCT analysis requires Python, Flask, `scipy`.

### Installation
1. Clone or download:
   - `index.html`
   - `styles.css`
   - `script.js`
   - (Optional) `favicon.ico`
2. Place files in the same directory.
3. Open `index.html` in a browser.

### Server-Side Setup (Optional, for Raw Image Analysis)
Set up a Flask API for DCT analysis:
```python
from flask import Flask, request
from PIL import Image
import scipy.fftpack
import numpy as np
import io

app = Flask(__name__)

@app.route('/analyze_dct', methods=['POST'])
def analyze_dct():
    file = request.files['image']
    img = Image.open(file).convert('L')
    img_array = np.array(img)
    dct = scipy.fftpack.dct(scipy.fftpack.dct(img_array.T, norm='ortho').T, norm='ortho')
    coefficients = dct.flatten()
    numbers = [abs(c) for c in coefficients if abs(c) > 0]
    return {'numbers': numbers[:1000]}
if __name__ == '__main__':
    app.run(debug=True)
```
Install dependencies:
```bash
pip install flask pillow scipy numpy
```
Update `script.js` to call the API:
```javascript
function extractDCTCoefficients(file, callback) {
    const formData = new FormData();
    formData.append('image', file);
    fetch('http://localhost:5000/analyze_dct', {
        method: 'POST',
        body: formData
    })
    .then(response => response.json())
    .then(data => callback(data.numbers))
    .catch(err => {
        console.error('Server-side DCT failed:', err);
        callback([]);
    });
}
```

### Usage
1. **Select Analysis Mode**:
   - Text, Raw Image, or Pixel Values.
2. **Upload a File**:
   - Text: Numbers separated by spaces.
   - Images: Clear text (OCR), JPEG (DCT), any image (pixel).
3. **Monitor Progress**:
   - Text: 0-100%.
   - Images: 0-50% for OCR/DCT/pixel, 50-100% for counting.
4. **Review Results**:
   - Check table and analysis (p-value ≥ 0.05 for consistency).
5. **Check Help**:
   - Refer to help section.

### Testing
- **Text File**: Upload `.txt` with numbers.
- **Image (Text)**: Use a receipt image (e.g., `draw.jpg`). Check OCR text.
- **Image (Raw)**: Upload JPEG with server-side API.
- **Image (Pixel)**: Test any image; note experimental results.
- **Edge Cases**: Test empty files, non-numeric images.
- **Browser Stability**: Verify in Chrome, Firefox, Edge.

## Limitations
- **OCR Accuracy**: Struggles with handwritten or low-resolution images.
- **Client-Side DCT**: Limited to JPEGs and small images.
- **Pixel Analysis**: Experimental; RGB values (0–255) may not follow Benford’s Law.
- **Statistical Test**: Requires ≥100 numbers.
- **File Types**: Only `.txt`, `.png`, `.jpg` supported.

## Future Improvements
- Add PDF support.
- Default server-side DCT.
- Allow OCR language selection.
- Add image preview.
- Support two-digit Benford analysis.

## License
MIT License.

## Acknowledgments
- Built with [Chart.js](https://www.chartjs.org/), [Tesseract.js](https://github.com/naptha/tesseract.js), [OpenCV.js](https://opencv.org/), [jStat](https://jstat.github.io/), [jpeg-js](https://github.com/eugeneware/jpeg-js).
- Inspired by Benford’s Law in fraud detection and image forensics.

For support, open an issue or contact the developer.
