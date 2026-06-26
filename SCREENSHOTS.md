# 📸 Adding Screenshots

The README displays images from the `screenshots/` folder.

## Available Screenshots

| Filename | Content | Size |
|---|---|---|
| `screenshots/social-preview.png` | Social media banner / OG image | 1200×630 px |
| `screenshots/fullscreen.png` | Full app view (panel + 3D side by side) | 2626×1842 px |
| `screenshots/3d-view.png` | 3D garden view with shade sail and shadow cast | 1280×1128 px |
| `screenshots/panel.png` | Control panel — Garden & Location/Time sections | 680×1582 px |
| `screenshots/panel-posts.png` | Control panel — Posts section | 674×1572 px |
| `screenshots/panel-sails.png` | Control panel — Sails section (2 sails) | 662×1596 px |
| `screenshots/panel-furniture.png` | Control panel — Furniture section | 662×1622 px |
| `screenshots/panel-shadow.png` | Control panel — Shadow area, Daily Analysis & Sun | 660×1122 px |

## How to Update Screenshots

1. Open the app: https://nomiknomik.github.io/shade-sail-simulator/
2. Take a screenshot of the desired view
3. Save as PNG in the `screenshots/` folder
4. Commit and push:

```bash
git add screenshots/
git commit -m "docs: update screenshots"
git push
```

## Screenshot Guidelines

- **3D view**: Use a representative scene with sail, shadow, furniture, and colored posts visible
- **Panel screenshots**: Scroll to the relevant section; capture the full section without cropping
- **Fullscreen**: Capture both panel and 3D view in a single shot at ~1300×900 px or larger
- **Format**: PNG (lossless), no JPEG artifacts
