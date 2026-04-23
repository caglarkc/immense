# Generate App Store Screenshots with ScreenshotWhale

Generate professional app store screenshots for any app and open them in the ScreenshotWhale visual editor for further customization.

## When to use

Use this skill when the user asks to:
- Generate app store screenshots
- Create screenshots for the App Store or Google Play
- Make store listing images
- Generate promotional screenshots for their app

## Philosophy: Screenshots are ads, not documentation

App store screenshots are the #1 conversion driver. They must sell the app in 2 seconds of scrolling. Every screenshot should answer "why should I download this?" — not "what does this screen look like?"

- Lead with **emotional benefit**, not feature name ("Never miss a moment" > "Push notifications")
- Each screenshot should tell a **micro-story** — a problem the user has and how this app solves it
- The first screenshot is the most important — it should convey the app's core value proposition
- Use **power words**: effortless, instant, beautiful, smart, secure, free
- Vary compositions between screenshots to create visual rhythm

## How it works

1. Ask the developer about their app and what they want to highlight
2. Read the codebase to infer colors and design language
3. Ask the developer to provide screenshots
4. Design screenshot layouts with text, backgrounds, and device mockups
5. Send everything to the ScreenshotWhale API which uploads images and creates a project
6. Give the developer a URL to open the project in the ScreenshotWhale visual editor

No authentication is required. The API returns a claim token — when the developer opens the link, they sign in (or create a free account) and the project is automatically saved to their account.

## Step-by-step instructions

### Step 1: Ask the developer about their app

Before doing anything, ask the developer to describe their app. You need this information from them directly — do NOT skip this step or try to infer everything silently.

Ask them:

> To generate great app store screenshots, I need a few details from you:
>
> 1. **What does your app do?** (one or two sentences)
> 2. **What are the main features you want to highlight?** (list 3-6 features, in order of importance)
> 3. **Any specific colors you want to use?** (I'll also try to infer your color palette from the codebase — but if you have specific brand colors in mind, let me know now)

Wait for the developer to answer before proceeding.

### Step 2: Read the codebase for design context

After the developer has answered, read the codebase to supplement their input:

**Read these files** (adapt to the project's stack):
- Theme/color files: tailwind.config, theme.ts, Colors.swift, colors.xml, etc.
- README or package.json for the app name if the developer didn't mention it

**Extract:**
- **App name** — the display name shown on the store
- **Design language** — primary color, accent color, background color, font family from theme files
- **Emotional tone** — professional, playful, minimal, bold?

If the developer provided explicit colors, use those. Otherwise, use the colors extracted from the theme files. Tell the developer what palette you're going with so they can correct it if needed.

### Step 3: Ask for screenshots

Ask the developer to provide their app screenshots:

> Now I need screenshots of your app. Please take **6-10 screenshots** of the key screens you want to showcase and place them in a `screenshots/` directory in the root of your project.
>
> Each screenshot should show one of the features you mentioned. Name them descriptively (e.g., `home.png`, `search.png`, `settings.png`) so I can match them to the right feature.
>
> Let me know once they're ready!

Wait for the developer to provide the screenshots before proceeding. Do NOT search the codebase for screenshots.

**IMPORTANT: Use every single screenshot the developer provides. Never skip, filter, or suggest removing any. If they gave you 6 screenshots, generate exactly 6 slides. If they gave you 8, generate exactly 8. No exceptions.**

### Step 4: Write compelling copy

For each screenshot provided, write:
- **Headline** (3-6 words): emotional benefit, not feature description
- **Optional subtitle** (6-12 words): adds context or specificity

Examples of good vs bad copy:
- Bad: "Dashboard View" → Good: "Everything at a Glance"
- Bad: "Settings Screen" → Good: "Make It Yours"
- Bad: "Search Feature" → Good: "Find Anything Instantly"
- Bad: "Dark Mode Support" → Good: "Easy on the Eyes"

### Step 5: Design the screenshots

Each slide has these layers (bottom to top):
1. **Background** (`shape`) — gradient using the app's color palette
2. **Headline** (`text`) — short, bold, benefit-driven
3. **Subtitle** (`text`, optional) — supporting detail
4. **Device mockup** (`mockup`) — the screenshot inside a device frame

#### Typography — exact font sizes per device

**CRITICAL: Do NOT calculate font sizes yourself. Use the exact values from this table. Copy them directly into the JSON.**

The editor uses a zoomed canvas, so font sizes are small numbers (not full-resolution pixels). These values are calibrated to look correct in the editor and export at the right size.

| Device name | Headline fontSize | Subtitle fontSize |
|---|---|---|
| `iPhone 6.9" (1320 x 2868)` | **21** | **15** |
| `iPhone 6.9" (1290 x 2796)` | **21** | **15** |
| `iPhone 6.9" (1260 x 2736)` | **20** | **14** |
| `iPhone 6.5" (1284 x 2778)` | **21** | **15** |
| `iPhone 6.5" (1242 x 2688)` | **21** | **15** |
| `iPhone 6.3" (1206 x 2622)` | **20** | **14** |
| `iPhone 6.3" (1179 x 2556)` | **21** | **15** |
| `iPad 13" (2064 x 2752)` | **23** | **16** |
| `iPad 13" (2048 x 2732)` | **23** | **16** |
| `iPad 12.9" (2048 x 2732)` | **23** | **16** |
| `iPad 11" (1668 x 2420)` | **21** | **15** |
| `iPad 11" (1668 x 2388)` | **21** | **15** |
| `iPad 11" (1640 x 2360)` | **21** | **15** |
| `iPad 11" (1488 x 2266)` | **20** | **14** |
| `Pixel 10 Pro XL` | **20** | **14** |
| `Pixel 10 Pro` | **21** | **15** |
| `Pixel 10` | **20** | **14** |
| `Samsung Galaxy S25` | **21** | **14** |

Headline fontWeight: `"700"`. Subtitle fontWeight: `"400"`. Font family: `"SF Pro Rounded"`.

**These are the ONLY allowed font sizes. If your fontSize value does not appear in this table, you have a bug. Never use a fontSize above 23.**

#### Layout patterns

**IMPORTANT: Never use the same layout twice in a row.** Rotate through these patterns to create visual rhythm. All values below are for a 1320×2868 canvas — scale proportionally for other sizes.

**CRITICAL — Mockup centering rule:**

The mockup component renders at a fixed aspect ratio (19.5/9 for all phones). It **ignores the layer `width`** and computes its visual width as `height / (19.5/9)`. The mockup is left-aligned within the layer bounding box.

**Therefore, you MUST set the mockup layer's `width` to match its visual width, then center it:**

```python
PHONE_ASPECT_RATIO = 19.5 / 9  # All phone mockups use this

mockup_h = round(H * height_ratio)  # e.g., H * 0.74
mockup_w = round(mockup_h / PHONE_ASPECT_RATIO)  # visual width from aspect ratio
mockup_x = round((W - mockup_w) / 2)  # center horizontally
```

**Never use `w=W*0.8` or any width percentage for mockup layers.** The width depends on height and aspect ratio, NOT on canvas width. This is especially important for iPads where the canvas is wide but the phone mockup is narrow.

**Layout A — Text top, device bottom-center (hero shot)**
Use for the first screenshot and strong feature highlights.
```
Headline:  x=W*0.06, y=H*0.04,  w=W*0.88, h=H*0.06
Subtitle:  x=W*0.1,  y=H*0.13,  w=W*0.8,  h=H*0.06, opacity=0.75
Mockup:    h=H*0.74,  w=h/(19.5/9),  x=(W-w)/2,  y=H*0.24
```
Text is centered at top with clear gap between headline and subtitle. Device fills the lower 3/4.

**Layout B — Text top-left, device below centered**
Use for features that benefit from left-aligned text.
```
Headline:  x=W*0.06, y=H*0.04,  w=W*0.88, h=H*0.06, textAlign=left
Subtitle:  x=W*0.06, y=H*0.13,  w=W*0.8,  h=H*0.06, textAlign=left, opacity=0.75
Mockup:    h=H*0.74,  w=h/(19.5/9),  x=(W-w)/2,  y=H*0.24
```
Text is left-aligned with clear gap. Device is centered below.

**Layout C — Device top, text bottom**
Use for screens that need to be seen at larger size.
```
Mockup:    h=H*0.65,  w=h/(19.5/9),  x=(W-w)/2,  y=H*0.03
Headline:  x=W*0.06, y=H*0.72,  w=W*0.88, h=H*0.06
Subtitle:  x=W*0.1,  y=H*0.81,  w=W*0.8,  h=H*0.06, opacity=0.75
```
Inverted layout — device dominates the top (centered), text anchors the bottom with clear gap.

**Layout D — Text top, large device below**
Use for visually rich screens where the app speaks for itself.
```
Headline:  x=W*0.06, y=H*0.04,  w=W*0.88, h=H*0.06
Subtitle:  x=W*0.1,  y=H*0.13,  w=W*0.8,  h=H*0.06, opacity=0.75
Mockup:    h=H*0.78,  w=h/(19.5/9),  x=(W-w)/2,  y=H*0.22
```
Same text positioning as A, but device is taller for maximum presence.

#### Composition rules

- **Rotate layouts**: For 6 screenshots, use something like A, B, C, D, B, A — never repeat consecutively
- **Vary gradient angles**: 180°, 135°, 225°, 160°, 200° — each screenshot should feel different
- **Vary gradient colors**: Use different shades from the app's palette for each screenshot, not the same gradient everywhere
- **First and last screenshots should be the strongest** — use Layout A or D for these
- **Use the app's own colors** from the theme files for gradients and accent text

#### Multi-device generation

**You MUST generate screenshots for ALL device sizes listed below — every single one, regardless of platform.** Each device gets the same screenshots (same images, same text, same layout pattern per slide) but with all positions, sizes, and font sizes recalculated for that device's canvas dimensions using the proportional W/H formulas.

Each device is a separate entry in the `devices` array. All devices share the same images from the `images` map.

Generate for ALL of these devices:

**iOS phones** (all required for App Store):
| Device name (use exactly) | W | H | mockupType |
|---|---|---|---|
| `iPhone 6.9" (1320 x 2868)` | 1320 | 2868 | `iphone-17-pro-max` |
| `iPhone 6.9" (1290 x 2796)` | 1290 | 2796 | `iphone-17-pro-max` |
| `iPhone 6.9" (1260 x 2736)` | 1260 | 2736 | `iphone-17-pro-max` |
| `iPhone 6.5" (1284 x 2778)` | 1284 | 2778 | `iphone-16-pro` |
| `iPhone 6.5" (1242 x 2688)` | 1242 | 2688 | `iphone-16-pro` |
| `iPhone 6.3" (1206 x 2622)` | 1206 | 2622 | `iphone-17-pro` |
| `iPhone 6.3" (1179 x 2556)` | 1179 | 2556 | `iphone-17-pro` |

**iPads** (all required for App Store):
| Device name (use exactly) | W | H | mockupType |
|---|---|---|---|
| `iPad 13" (2064 x 2752)` | 2064 | 2752 | `iphone-17-pro` |
| `iPad 13" (2048 x 2732)` | 2048 | 2732 | `iphone-17-pro` |
| `iPad 12.9" (2048 x 2732)` | 2048 | 2732 | `iphone-17-pro` |
| `iPad 11" (1668 x 2420)` | 1668 | 2420 | `iphone-17-pro` |
| `iPad 11" (1668 x 2388)` | 1668 | 2388 | `iphone-17-pro` |
| `iPad 11" (1640 x 2360)` | 1640 | 2360 | `iphone-17-pro` |
| `iPad 11" (1488 x 2266)` | 1488 | 2266 | `iphone-17-pro` |

iPads have a more square aspect ratio than phones. The same proportional layout formulas apply — since positions use `W*ratio` and `H*ratio`, the layouts automatically adapt. No special handling needed.

**Apple Watch** (required for watchOS apps):
| Device name (use exactly) | W | H | mockupType |
|---|---|---|---|
| `Apple Watch Ultra 3 (422 x 514)` | 422 | 514 | `apple-watch-series-10` |
| `Apple Watch Ultra (410 x 502)` | 410 | 502 | `apple-watch-series-10` |
| `Apple Watch Series 10 (416 x 496)` | 416 | 496 | `apple-watch-series-10` |

Watch layouts are different: use only a background shape and the screenshot as a full-bleed `image` layer (not a mockup) filling the entire canvas. No text layers, no device frame — just the raw screenshot.

**Android phones** (all required for Play Store):
| Device name (use exactly) | W | H | mockupType |
|---|---|---|---|
| `Pixel 10 Pro XL` | 1344 | 2992 | `pixel-10-pro-xl` |
| `Pixel 10 Pro` | 1280 | 2856 | `pixel-10-pro` |
| `Pixel 10` | 1080 | 2424 | `pixel-10` |
| `Samsung Galaxy S25` | 1080 | 2340 | `samsung-s25` |

**Android watch** (required for Wear OS apps):
| Device name (use exactly) | W | H | mockupType |
|---|---|---|---|
| `Google Pixel Watch 4` | 456 | 456 | `pixel-watch-4` |

Same as Apple Watch: full-bleed image, no text, no device frame.

Only include watch devices if the developer specifically mentions their app runs on a watch. All other devices (phones + iPads) are always generated.

**Verification: You must generate exactly 18 device entries (7 iOS phones + 7 iPads + 4 Android phones) minimum. If your devices array has fewer than 18 entries, you have a bug. Fix it before sending the API request.**

### Step 6: Build the project JSON and send to API

Construct the project payload and send it to the ScreenshotWhale API. The API handles image uploads to Cloudflare and returns a deep link URL.

**Progress reporting**: This step takes a while. Keep the developer informed by printing short status messages as you go:
1. "Resizing and encoding screenshots..." — before encoding images
2. "Building project JSON for N devices..." — before constructing the payload
3. "Uploading to ScreenshotWhale..." — before sending the curl request

#### How to build and send the payload

Write a single Python script to `/tmp/build-screenshots.py` that:
1. Reads the base64 files
2. Constructs the JSON with correct field names (see schema below)
3. Writes to `/tmp/screenshot-project.json`

Use only standard library (`json`, `os`, `math`).

**The script must produce correct JSON on the first try. Do NOT create separate validation or fix scripts. Do NOT create any files in the project directory — only write to `/tmp/`.**

**CRITICAL: The script MUST generate ALL devices programmatically using a loop.** Define the full device list as a Python data structure and iterate over it. Do NOT manually write JSON for each device — that approach will miss devices. Example structure:

```python
DEVICES = [
    # iOS phones
    {"deviceName": 'iPhone 6.9" (1320 x 2868)', "W": 1320, "H": 2868, "mockupType": "iphone-17-pro-max", "headlineFontSize": 21, "subtitleFontSize": 15},
    {"deviceName": 'iPhone 6.9" (1290 x 2796)', "W": 1290, "H": 2796, "mockupType": "iphone-17-pro-max", "headlineFontSize": 21, "subtitleFontSize": 15},
    # ... ALL devices from the tables above ...
]

for i, device in enumerate(DEVICES):
    W, H = device["W"], device["H"]
    screenshots = []
    for j, slide in enumerate(slides):
        layers = build_layers(slide, device, j)  # uses W*ratio, H*ratio formulas
        screenshots.append({...})
    devices_array.append({"id": f"device-{i}", "deviceName": device["deviceName"], "screenshots": screenshots})
```

**The DEVICES list must contain ALL 18 phone+tablet devices (7 iOS phones + 7 iPads + 4 Android phones). Verify `len(DEVICES) >= 18` in the script.** Add watch devices only if the user mentioned watch support.

After the script runs, send the request:
```bash
curl -s -X POST https://storeshots-backend.onrender.com/api/projects/create-from-skill \
  -H "Content-Type: application/json" \
  --data @/tmp/screenshot-project.json
```

If the above URL doesn't work, try `http://localhost:1337/api/projects/create-from-skill` (local development).

No authentication is required.

#### Exact JSON schema — follow this precisely

Every field shown below is **required**. Do not rename, omit, or add fields. Copy the structure exactly.

**Top level:**
```json
{
  "name": "App Name Screenshots",
  "version": "1.0.0",
  "metadata": {
    "totalDevices": 18,
    "totalScreenshots": 6,
    "supportedDevices": ["device name 1", "device name 2"]
  },
  "settings": { "collapsedGroups": [] },
  "languages": ["en"],
  "images": {
    "key1": "<base64 string>",
    "key2": "<base64 string>"
  },
  "devices": [ ... ]
}
```

**Each device** in the `devices` array:
```json
{
  "id": "device-0",
  "deviceName": "iPhone 6.9\" (1320 x 2868)",
  "screenshots": [ ... ]
}
```
⚠️ The field is `deviceName`, NOT `name`. Do NOT include `width` or `height` at device level.

**Each screenshot** in a device's `screenshots` array:
```json
{
  "id": "screenshot-0-0",
  "name": "slide_name",
  "active": true,
  "device": "iPhone 6.9\" (1320 x 2868)",
  "orientation": "portrait",
  "layers": [ ... ]
}
```

**Background layer** (shape):
```json
{
  "id": "layer-bg-0-0",
  "type": "shape",
  "name": "Background",
  "visible": true,
  "locked": false,
  "x": 0,
  "y": 0,
  "width": 1320,
  "height": 2868,
  "rotation": 0,
  "opacity": 1,
  "isBackground": true,
  "fillType": "Gradient",
  "gradientType": "linear",
  "gradientAngle": 180,
  "gradientStops": [
    { "color": "#1a1a2e", "position": 0 },
    { "color": "#16213e", "position": 100 }
  ]
}
```
⚠️ Gradient stops use `position: 0` and `position: 100` (not 0 and 1). No `shape` or `gradient` wrapper fields.

**Headline layer** (text):
```json
{
  "id": "layer-title-0-0",
  "type": "text",
  "name": "Headline",
  "visible": true,
  "locked": false,
  "x": 79,
  "y": 143,
  "width": 1161,
  "height": 229,
  "rotation": 0,
  "opacity": 1,
  "content": "Your Core Value",
  "color": "#ffffff",
  "fontFamily": "SF Pro Rounded",
  "fontSize": 21,
  "fontWeight": "700",
  "textAlign": "center"
}
```
⚠️ The text field is `content`, NOT `text`. `fontWeight` must be a string `"700"`, not a number. `fontSize` must come from the typography table — never above 23.

**Subtitle layer** (text):
```json
{
  "id": "layer-subtitle-0-0",
  "type": "text",
  "name": "Subtitle",
  "visible": true,
  "locked": false,
  "x": 132,
  "y": 401,
  "width": 1056,
  "height": 143,
  "rotation": 0,
  "opacity": 0.75,
  "content": "A short description of the benefit",
  "color": "#ffffff",
  "fontFamily": "SF Pro Rounded",
  "fontSize": 15,
  "fontWeight": "400",
  "textAlign": "center"
}
```

**Mockup layer:**
```json
{
  "id": "layer-mockup-0-0",
  "type": "mockup",
  "name": "Device",
  "visible": true,
  "locked": false,
  "x": 171,
  "y": 688,
  "width": 979,
  "height": 2122,
  "rotation": 0,
  "opacity": 1,
  "src": "img:screenshot1",
  "mockupType": "iphone-17-pro-max",
  "mockupColor": "black"
}
```
⚠️ Image references use `"src": "img:<key>"` where `<key>` matches a key in the `images` map. Do NOT use `imageKey`.
⚠️ Mockup width MUST be `height / (19.5/9)`, NOT a percentage of canvas width. Then `x = (W - width) / 2`. Example above: h=2122, w=round(2122/2.167)=979, x=round((1320-979)/2)=171.

#### Common mistakes to avoid

| Wrong | Correct |
|---|---|
| `"name"` on device | `"deviceName"` on device |
| `"text"` on text layer | `"content"` on text layer |
| `"imageKey"` on mockup | `"src": "img:<key>"` on mockup |
| `fontWeight: 700` (number) | `fontWeight: "700"` (string) |
| gradient `position: 1` | gradient `position: 100` |
| `"gradient": { "stops": [...] }` | `"gradientStops": [...]` (flat) |
| `"shape": "rectangle"` | omit — not needed |
| `fontSize: 86` (too large) | look up from typography table |
| mockup `width: W*0.8` (wrong) | `width = height / (19.5/9)` then `x = (W-width)/2` |

#### Response

```json
{
  "data": {
    "projectId": 123,
    "claimToken": "abc123...",
    "editorUrl": "https://screenshotwhale.com/editor?project=123&claim=abc123..."
  },
  "meta": {
    "message": "Project created successfully. Open the editor URL to claim and edit it."
  }
}
```

### Step 7: Clean up and show the result

After a successful API call, remove all temporary files:

```bash
rm -f /tmp/screenshot-project.json /tmp/build-screenshots.sh /tmp/ss_*.b64
```

Then display:

```
Your app store screenshots have been generated!

Open this URL to view and customize them in the ScreenshotWhale editor:
<editorUrl from response>

When you open the link, you'll be asked to sign in (or create a free account).
The project will automatically be saved to your account.
You can then adjust text, colors, layouts, add more devices, and export final images.
```

## Reading and encoding screenshots

Screenshots must be resized before encoding to keep the API payload under the 50MB limit. Resize each image to a max width of 1290px (standard App Store resolution) using `sips` (macOS) before base64-encoding:

```bash
sips --resampleWidth 1290 screenshots/home.png
base64 -i screenshots/home.png | tr -d '\n'
```

If `sips` is not available (Linux), use ImageMagick:

```bash
convert screenshots/home.png -resize 1290x screenshots/home.png
base64 -i screenshots/home.png | tr -d '\n'
```

Or in the JSON, include the raw base64 string (no data URL prefix needed — the backend handles both formats).

## Error handling

- If the API returns 429: rate limited (5 projects per day) — wait and try again tomorrow
- If screenshots are too large (>10MB each), suggest resizing them first
- Maximum 10 images per project
- Maximum ~50MB total payload
- If the API returns any other error, show it to the user with context