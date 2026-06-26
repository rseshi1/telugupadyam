---
name: testing-telugu-poem-reader
description: Test the Telugu Poem Reader app end-to-end. Use when verifying OCR, TTS, upload, and UI changes.
---

# Testing the Telugu Poem Reader

## Overview
The Telugu Poem Reader is a single-page client-side web app (`index.html`) that uses Tesseract.js for OCR and Web Speech API for TTS. All processing runs in the browser — no backend.

## Prerequisites
- Python 3 (for local HTTP server)
- Chrome browser
- A test image with Telugu text (PNG/JPG)

## Devin Secrets Needed
None — this app is fully client-side with no authentication.

## Setup

1. **Start the HTTP server** from the repo directory:
   ```bash
   cd /home/ubuntu/repos/telugupadyam
   python3 -m http.server 8080 &
   ```

2. **Create a test Telugu image** (if one doesn't exist):
   ```python
   from PIL import Image, ImageDraw, ImageFont
   img = Image.new('RGB', (800, 400), color='white')
   draw = ImageDraw.Draw(img)
   font = ImageFont.truetype('/usr/share/fonts/truetype/noto/NotoSansTelugu-Bold.ttf', 48)
   lines = ['చిన్ని చిన్ని ఆశలు', 'మనసులో పూచే పూలు', 'కలల మబ్బుల నీడలో', 'కనులు తెరిచే వెలుగులు']
   for i, line in enumerate(lines):
       draw.text((50, 50 + i * 80), line, fill='black', font=font)
   img.save('test_telugu_poem.png')
   ```
   Place the image in the server's root directory so it's accessible via fetch.

3. **Open Chrome** at `http://localhost:8080`

## Key Testing Notes

### File Upload
- The `<input type="file">` element has `id="file-input"` (hyphenated, not camelCase).
- To programmatically upload a file via the browser console:
  ```js
  fetch('/test_telugu_poem.png').then(r => r.blob()).then(b => {
    const f = new File([b], 'test.png', {type:'image/png'});
    const dt = new DataTransfer();
    dt.items.add(f);
    document.getElementById('file-input').files = dt.files;
    document.getElementById('file-input').dispatchEvent(new Event('change'));
  });
  ```
- The test image must be in the HTTP server's serving directory for fetch to work.

### OCR
- Tesseract.js downloads the Telugu language model on first use (~5-10 seconds).
- OCR results may have minor character-level variations — this is expected. Check for recognizable Telugu text fragments rather than exact matches.
- Progress updates appear in real-time: "Initializing OCR engine..." then "Recognizing Telugu text... X%".

### TTS (Text-to-Speech)
- **Headless Chrome on Linux has no speech synthesis voices.** The app handles this gracefully:
  - Shows red error: "No speech voices are available. Text-to-speech is not supported in this browser."
  - Disables the Play button.
- To fully test TTS playback (Play/Pause/Stop, speed slider with audio), use Chrome on macOS, Windows, or Android where Telugu voices might be available.
- The speed slider (`id="speed-slider"`) range is 0.5 to 2.0, label updates in real-time (`id="speed-value"`).

### Error Handling
- Uploading an unsupported file type (e.g., .txt) shows an error toast at the bottom: "Unsupported file type. Please upload a PNG, JPG, or PDF."
- The toast auto-dismisses after ~5 seconds.

## Test Checklist
1. Initial UI state: header, drop zone, footer visible; no result/playback controls
2. Image upload + OCR: Telugu text extracted and displayed in textarea
3. Text editing: textarea is editable, text can be selected and modified
4. TTS voice detection: appropriate warning/error shown based on available voices
5. Speed slider: label updates when slider is adjusted
6. Reset flow: "Upload another file" button returns to initial state
7. Error handling: unsupported file types show error toast
8. (If TTS voices available) Play/Pause/Stop controls work correctly
