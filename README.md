# 🎬 Zero-Credit Vox Animation Engine: Complete Setup & Code Guide
**SIDमनु TECH (@sidmanutech)**

Stop paying Runway, Kling, or Midjourney \$30–\$50 every month for short video clips. With this open-source agent workflow, you can generate complete, authentic Vox-style paper-collage documentaries on your Mac or PC for **₹0 in video credits**.

---

## ⚡ The Architecture at a Glance

```
Topic Prompt ("India Gate 1921")
       │
       ├──► 1. Wikipedia / Perplexity (Fact-Checking & Concrete Numbers)
       ├──► 2. Wikimedia Commons Archive Searcher (Public-Domain High-Res Photos)
       ├──► 3. Native Apple Vision Framework (Zero-API Subject Background Lift)
       ├──► 4. Cloned Voice Engine (ElevenLabs or Local VoiceStudio on Apple Silicon MPS)
       └──► 5. HyperFrames + GSAP 12fps Stop-Motion Engine (Paper Boil, Torn Edges, Slams)
```

---

## 🛠️ Step 1: Native Subject Lift (Apple Vision — No Cloud API)

Cloud background removal APIs (Remove.bg, Photoroom) charge \$0.20–\$0.50 per image. On macOS, you can use Apple's built-in **VNGeneratePersonSegmentationRequest** and **VNGenerateForegroundInstanceMaskRequest** completely free and offline.

### Compiling the Swift CLI Tool (`tools/lift`):
Save this as `lift.swift` and compile it with macOS native Swift:

```swift
import Foundation
import Vision
import CoreImage
import CoreImage.CIFilterBuiltins

guard CommandLine.arguments.count > 2 else {
    print("Usage: lift <input_image> <output_png>")
    exit(1)
}

let inputURL = URL(fileURLWithPath: CommandLine.arguments[1])
let outputURL = URL(fileURLWithPath: CommandLine.arguments[2])

guard let ciImage = CIImage(contentsOf: inputURL) else { exit(1) }

let request = VNGenerateForegroundInstanceMaskRequest()
let handler = VNImageRequestHandler(ciImage: ciImage, options: [:])
try? handler.perform([request])

guard let result = request.results?.first else { exit(1) }
let mask = try! result.generateScaledMaskForImage(forInstances: result.allInstances, from: handler)

let filter = CIFilter.blendWithMask()
filter.inputImage = ciImage
filter.backgroundImage = CIImage.empty()
filter.maskImage = CIImage(cvPixelBuffer: mask)

let context = CIContext()
guard let output = filter.outputImage,
      let cgImage = context.createCGImage(output, from: output.extent) else { exit(1) }

let dest = CGImageDestinationCreateWithURL(outputURL as CFURL, "public.png" as CFString, 1, nil)!
CGImageDestinationAddImage(dest, cgImage, nil)
CGImageDestinationFinalize(dest)
print("✅ Subject extracted successfully to: \(outputURL.path)")
```

Compile it once in terminal:
```bash
swiftc -O lift.swift -o lift
chmod +x lift
# Run on any image:
./lift archive_photo.jpg cut_subject.png
```

---

## 📜 Step 2: Public-Domain Archive Search (`photos.mjs`)

Vox videos rely on genuine historical documents, maps, and archival photos. Wikimedia Commons has over 100 million public-domain, CC-0 assets:

```javascript
// tools/photos.mjs
import fs from 'fs';

const query = process.argv[2] || "India Gate 1920";
const url = `https://commons.wikimedia.org/w/api.php?action=query&generator=search&gsrsearch=${encodeURIComponent(query)}&gsrlimit=5&prop=imageinfo&iiprop=url|mime|extmetadata&format=json`;

const res = await fetch(url, { headers: { 'User-Agent': 'VoxEngine/1.0 (sidmanutech)' } });
const data = await res.json();
const pages = data.query?.pages || {};

for (const [id, page] of Object.entries(pages)) {
  const info = page.imageinfo?.[0];
  if (info && info.mime.startsWith('image/')) {
    console.log(`Found: ${page.title} -> ${info.url}`);
  }
}
```

---

## ✂️ Step 3: Stop-Motion Physics & Paper Boil (`collage-kit.js`)

Real paper stop-motion does **not** use smooth linear easing. It uses stepped ease curves (12fps) and continuous micro-rotation ("boil"):

```javascript
// 12fps Stepped Ease Curve
const sm = (easeStr, duration) => {
  const frames = Math.max(2, Math.round(duration * 12));
  return `stepped(${frames})`;
};

// Paper Boil Jitter (Simulates physical stop-motion wiggle)
function boil(durationSec = 40) {
  const tl = gsap.timeline({ repeat: -1 });
  const frameDur = 1 / 12; // 12fps
  for (let t = 0; t < durationSec; t += frameDur) {
    tl.set(".paper-cutout", {
      rotation: () => (Math.random() - 0.5) * 1.8,
      x: () => (Math.random() - 0.5) * 3,
      y: () => (Math.random() - 0.5) * 3,
    }, t);
  }
}
```

### Essential CSS Paper-Cut Sticker Styling:
```css
/* Authentic White Sticker Border & Depth Drop Shadow */
.paper-cutout {
  position: absolute;
  filter: 
    drop-shadow(0 6px 0 #ffffff)
    drop-shadow(0 -6px 0 #ffffff)
    drop-shadow(6px 0 0 #ffffff)
    drop-shadow(-6px 0 0 #ffffff)
    drop-shadow(0 20px 30px rgba(0, 0, 0, 0.45));
  will-change: transform;
}

/* Halftone Texture Overlay */
.halftone-overlay {
  position: absolute;
  inset: 0;
  pointer-events: none;
  background-image: radial-gradient(#111 1px, transparent 1px);
  background-size: 6px 6px;
  opacity: 0.12;
  mix-blend-mode: multiply;
}
```

---

## 🎙️ Step 4: Voiceover Alignment

1. **Script Rule:** Keep numbers written as English words (e.g. `nineteen twenty-one`, `seventy thousand soldiers`).
2. **Timing:** Export timestamped word alignment JSON so stop-motion visual cutouts drop on the exact syllable.

---

## 🚀 Step 5: Render at 60fps via HyperFrames

1. Install HyperFrames CLI:
   ```bash
   npm i -g hyperframes
   ```
2. Check for WCAG contrast and layout perfection:
   ```bash
   hyperframes check .
   ```
3. Export full 1080×1920 60fps MP4:
   ```bash
   hyperframes render . -o vox_animation_skill_reel.mp4 --fps 60
   ```

---

## 📲 Community & Support
Questions or issues setting up on your OS?
- DM on Instagram: [@sidmanutech](https://instagram.com/sidmanutech)
- Follow on YouTube: [@sidmanutech](https://youtube.com/@sidmanutech)
