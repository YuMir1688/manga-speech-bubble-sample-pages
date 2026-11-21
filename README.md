# manga-speech-bubble-sample-pages

CC0 synthetic manga, vertical manhwa webtoon, and comic speech bubble test pages (JP/KR/ZH) for OCR, inpainting, and typesetting benchmarks.

Tested with browser-based translation and typesetting pipelines such as [BubbleTranslate](https://bubbletranslate.com/), including dedicated workflows for [vertical manhwa strips](https://bubbletranslate.com/manhwa-translator/) and [multi-panel comics](https://bubbletranslate.com/comic-translator/).

## Page Layout & Pipeline Specifications

| Format | Reading Flow & Text Orientation | Slicing / Canvas Considerations | Cleaning & Inpainting Notes |
| :--- | :--- | :--- | :--- |
| **JP Manga (`tategaki`)** | Right-to-left panels; vertical columns (`writing-mode: vertical-rl`) | Single or double-page spreads; high-DPI monochrome (600–1200 px width) | Screentone halo masking around furigana and overlapping SFX strokes |
| **KR Manhwa / Webtoon** | Top-to-bottom continuous scroll; horizontal Hangul blocks | Tall vertical strips (`800 x 10000+` px) require overlapping window slicing (e.g., 1600 px chunks with 180 px overlap) | Soft anti-aliased gradients and drop shadows behind floating narration boxes |
| **ZH Manhua / Comics** | Mixed horizontal/vertical bubbles; simplified & traditional CJK | Dense multi-clause bubbles requiring adaptive line-wrapping & auto-font fitting | Colored panel backgrounds outside white oval fills require edge-preserving canvas inpainting |

### 1. JP Vertical `tategaki` vs. Horizontal Grouping
- Vertical columns read **right-to-left** inside each bubble; naive left-to-right OCR line clustering scrambles multi-column Japanese dialogue.
- Furigana (ruby) glyphs beside kanji should be filtered or grouped into the parent vertical column before translation so they do not duplicate tokens.

### 2. KR Webtoon Tall-Strip Slicing
- Long vertical webtoon episodes exceed WebGL/Canvas max texture dimensions on many client browsers if processed as a single bitmap.
- Overlapping vertical window slicing prevents speech bubbles that straddle slice boundaries from being clipped or double-detected; duplicate detections inside the overlap band are merged via IoU (`>= 0.45`).

### 3. Screentone Halo Masking
- Binary thresholding alone leaves dark dot artifacts ("halos") around erased Japanese text over screentones or halftone gradients.
- Dilating the glyph mask by `2–4 px` inside the detected bubble interior and blending from the local background ring preserves clean speech bubble interiors without eroding black outlines.

## Sample Bounding-Box JSON Schema

```json
{
  "schema_version": "1.0.0",
  "page_id": "synthetic-jp-manga-001",
  "source_language": "ja",
  "layout": "manga_tategaki",
  "dimensions": { "width": 1200, "height": 1700 },
  "bubbles": [
    {
      "id": "b_01",
      "type": "speech_oval",
      "reading_order": 1,
      "orientation": "vertical_rl",
      "bbox_px": [780, 140, 210, 320],
      "polygon": [[800, 140], [980, 155], [990, 445], [780, 460]],
      "text_mask_dilation_px": 3,
      "screentone_background": false,
      "source_text": "ここが最初のテストページだ！",
      "translated_text": "This is the first test page!"
    },
    {
      "id": "b_02",
      "type": "narration_box",
      "reading_order": 2,
      "orientation": "vertical_rl",
      "bbox_px": [110, 920, 180, 290],
      "polygon": [[110, 920], [290, 920], [290, 1210], [110, 1210]],
      "text_mask_dilation_px": 2,
      "screentone_background": true,
      "source_text": "スクリーントーン上の文字消去テスト",
      "translated_text": "Screentone text cleaning test"
    }
  ]
}
```

## License

- Repository metadata, schema definitions, and documentation are released under the [MIT License](./LICENSE).
- Synthetic sample specifications and bounding-box fixtures are intended for CC0 public benchmarking.
