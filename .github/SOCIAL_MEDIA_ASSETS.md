# Social Media Assets & Open Graph Images

This guide helps you create professional social media cards and Open Graph images for the repository.

---

## 📐 Image Specifications

### Open Graph Image (Facebook, LinkedIn, Discord)
- **Size:** 1200 x 630 pixels
- **Format:** PNG or JPG
- **File size:** Under 1MB
- **Aspect ratio:** 1.91:1
- **File name:** `og-image.png`

### Twitter Card
- **Size:** 1200 x 675 pixels (or 1200 x 628)
- **Format:** PNG or JPG
- **File size:** Under 5MB
- **Aspect ratio:** 16:9
- **File name:** `twitter-card.png`

### GitHub Social Preview
- **Size:** 1280 x 640 pixels
- **Format:** PNG or JPG
- **File size:** Under 1MB
- **Aspect ratio:** 2:1
- **Location:** Repository Settings → Social Preview

---

## 🎨 Design Template

### Recommended Design Elements

**Main Elements:**
```
┌─────────────────────────────────────────────┐
│  🍎 iOS Interview Preparation Guide         │
│                                              │
│  110+ Questions & Answers                   │
│                                              │
│  ✅ Swift Programming                       │
│  ✅ UIKit & SwiftUI                         │
│  ✅ Architecture Patterns                   │
│  ✅ Concurrency & Performance               │
│                                              │
│  github.com/mukundjogi/ios-interview-prep   │
└─────────────────────────────────────────────┘
```

**Color Scheme:**
- Background: `#1C1C1E` (Dark) or `#FFFFFF` (Light)
- Primary: `#007AFF` (iOS Blue)
- Accent: `#FF9500` (iOS Orange)
- Text: `#FFFFFF` (on dark) or `#000000` (on light)
- Code blocks: `#2C2C2E` with `#34C759` accents

**Typography:**
- Heading: SF Pro Display Bold (or similar)
- Body: SF Pro Text Regular
- Code: SF Mono (or Menlo, Monaco)

---

## 🛠️ Creation Tools

### Option 1: Canva (Easiest)
1. Go to [Canva.com](https://www.canva.com)
2. Create custom size: 1200 x 630 px
3. Use template below
4. Download as PNG

**Canva Template Elements:**
- Background: Dark gradient (#1C1C1E to #2C2C2E)
- Large iOS/Swift logo or icon
- Title: "iOS Interview Preparation Guide"
- Subtitle: "110+ Questions & Answers"
- Bullet points with features
- Bottom: GitHub URL

### Option 2: Figma (Professional)
1. Open [Figma](https://www.figma.com)
2. Create frame: 1200 x 630
3. Design with layers
4. Export as PNG @2x

**Figma Design Tips:**
- Use Auto Layout for responsive design
- Create components for reusability
- Export multiple variants (OG, Twitter, GitHub)

### Option 3: Code-Based (Advanced)

Using HTML/CSS + Puppeteer or similar:

```html
<!DOCTYPE html>
<html>
<head>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            width: 1200px;
            height: 630px;
            background: linear-gradient(135deg, #1C1C1E 0%, #2C2C2E 100%);
            color: white;
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 60px;
        }
        .container {
            max-width: 1000px;
        }
        .logo {
            font-size: 72px;
            margin-bottom: 20px;
        }
        h1 {
            font-size: 64px;
            font-weight: 700;
            margin-bottom: 20px;
            color: #007AFF;
        }
        h2 {
            font-size: 36px;
            margin-bottom: 40px;
            color: #FF9500;
        }
        .features {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-bottom: 40px;
        }
        .feature {
            font-size: 24px;
            display: flex;
            align-items: center;
        }
        .feature::before {
            content: "✅";
            margin-right: 12px;
        }
        .url {
            font-size: 28px;
            color: #8E8E93;
            font-family: 'SF Mono', Monaco, monospace;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="logo">🍎</div>
        <h1>iOS Interview Preparation</h1>
        <h2>110+ Questions & Answers</h2>
        <div class="features">
            <div class="feature">Swift Programming</div>
            <div class="feature">UIKit & SwiftUI</div>
            <div class="feature">Architecture Patterns</div>
            <div class="feature">Concurrency & Performance</div>
        </div>
        <div class="url">github.com/mukundjogi/ios-interview-prep</div>
    </div>
</body>
</html>
```

Convert to image using:
```bash
# Using Puppeteer
npm install puppeteer
node generate-og-image.js

# Or using wkhtmltoimage
wkhtmltoimage --width 1200 --height 630 og-template.html og-image.png
```

### Option 4: Design Services

**Free Online Generators:**
- [Meta Tags](https://metatags.io/) - Visual OG image editor
- [Social Image Generator](https://www.bannerbear.com/) - Template-based
- [OG Image Generator](https://og-image.vercel.app/) - Vercel's tool

---

## 📝 Design Copy Suggestions

### Variation 1: Feature-Focused
```
🍎 iOS Interview Preparation

110+ Real Interview Questions
✅ Swift, UIKit, SwiftUI
✅ Architecture Patterns
✅ FAANG Company Questions
✅ Production-Ready Code

github.com/mukundjogi/ios-interview-prep
```

### Variation 2: Outcome-Focused
```
Land Your Dream iOS Job

Complete Interview Preparation Guide
• 110+ Questions from Apple, Google, Meta
• Modern Swift & iOS Development
• Architecture & Best Practices
• Free & Open Source

github.com/mukundjogi/ios-interview-prep
```

### Variation 3: Minimalist
```
iOS Interview Prep
110+ Questions | Swift 5.9+ | Open Source

github.com/mukundjogi/ios-interview-prep
```

---

## 🎯 Platform-Specific Cards

### GitHub Repository Card (1280 x 640)

**Key Elements:**
- Repository name prominently displayed
- "110+ Interview Questions" as main value prop
- Technology icons (Swift, iOS)
- Star/Fork CTAs
- Clean, professional look

### Twitter Card (1200 x 675)

**Key Elements:**
- Twitter-optimized aspect ratio
- Catchy headline
- Quick benefits list
- Handle @mukundjogi
- GitHub URL

### LinkedIn Card (1200 x 627)

**Key Elements:**
- Professional appearance
- Credibility indicators (FAANG companies)
- Value proposition clear
- Author info
- Link to guide

---

## 📂 Asset Organization

```
/assets/
├── og-image.png              # Main Open Graph image (1200x630)
├── twitter-card.png          # Twitter-specific card (1200x675)
├── github-social.png         # GitHub preview (1280x640)
├── logo.png                  # Repository logo
├── favicon.ico               # Website favicon
└── variations/
    ├── og-dark.png          # Dark theme variant
    ├── og-light.png         # Light theme variant
    └── og-animated.gif      # Animated version (if needed)
```

---

## ✅ Implementation Checklist

### 1. Create Images
- [ ] Design OG image (1200 x 630)
- [ ] Design Twitter card (1200 x 675)
- [ ] Design GitHub social preview (1280 x 640)
- [ ] Create repository logo/favicon
- [ ] Optimize file sizes (< 1MB)

### 2. Upload Images
- [ ] Upload to `/assets/` folder in repository
- [ ] Upload GitHub social preview in Settings
- [ ] Verify images are accessible via URLs

### 3. Update Meta Tags
- [ ] Verify `_includes/head-custom.html` has correct image URLs
- [ ] Update image paths in meta tags
- [ ] Test OG tags with debuggers

### 4. Test Previews
- [ ] Facebook Sharing Debugger
- [ ] Twitter Card Validator
- [ ] LinkedIn Post Inspector
- [ ] Discord/Slack preview

---

## 🔍 Testing Your Cards

### Facebook/Meta Debugger
1. Go to [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/)
2. Enter: `https://mukundjogi.github.io/ios-interview-prep/`
3. Click "Scrape Again"
4. Verify image appears correctly

### Twitter Card Validator
1. Go to [Twitter Card Validator](https://cards-dev.twitter.com/validator)
2. Enter: `https://mukundjogi.github.io/ios-interview-prep/`
3. Preview card appearance
4. Fix any warnings

### LinkedIn Post Inspector
1. Go to [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/)
2. Enter URL
3. Verify preview
4. Clear cache if needed

### Generic OG Checker
- [OpenGraph.xyz](https://www.opengraph.xyz/)
- [Meta Tags Preview](https://metatags.io/)

---

## 🎨 Brand Assets

### Color Palette
```css
/* Primary Colors */
--ios-blue: #007AFF;
--ios-orange: #FF9500;
--ios-green: #34C759;

/* Backgrounds */
--dark-bg: #1C1C1E;
--dark-elevated: #2C2C2E;
--light-bg: #FFFFFF;

/* Text */
--text-primary: #FFFFFF;
--text-secondary: #8E8E93;
--text-dark: #000000;
```

### Typography Scale
```css
/* Headings */
--heading-xl: 64px;  /* Main title */
--heading-lg: 48px;  /* Subtitle */
--heading-md: 36px;  /* Section */

/* Body */
--body-lg: 28px;     /* Feature text */
--body-md: 24px;     /* Regular */
--body-sm: 20px;     /* Small */
```

---

## 📱 Social Media Preview Examples

### What They Should Look Like:

**LinkedIn:**
```
┌─────────────────────────────────────┐
│  [OG Image: iOS Interview Prep]     │
│                                      │
│  iOS Interview Preparation Guide -  │
│  110+ Questions & Answers            │
│                                      │
│  Complete iOS interview prep with   │
│  real questions from FAANG...        │
│                                      │
│  MUKUNDJOGI.GITHUB.IO               │
└─────────────────────────────────────┘
```

**Twitter:**
```
┌─────────────────────────────────────┐
│  Your Tweet Text                     │
│                                      │
│  ┌─────────────────────────────┐   │
│  │  [Twitter Card Image]        │   │
│  │                              │   │
│  │  iOS Interview Prep          │   │
│  │  110+ Questions & Answers    │   │
│  └─────────────────────────────┘   │
│                                      │
│  github.com/mukundjogi/ios-...      │
└─────────────────────────────────────┘
```

---

## 🚀 Quick Start Script

Create this file as `generate-social-cards.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Social Card Generator</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; padding: 20px; }
        .card { width: 1200px; height: 630px; background: linear-gradient(135deg, #1C1C1E, #2C2C2E); 
                color: white; padding: 60px; box-sizing: border-box; margin: 20px 0; }
        .logo { font-size: 80px; margin-bottom: 20px; }
        h1 { font-size: 64px; color: #007AFF; margin-bottom: 10px; }
        h2 { font-size: 40px; color: #FF9500; margin-bottom: 30px; }
        .features { font-size: 28px; margin-bottom: 30px; }
        .feature { margin: 10px 0; }
        .feature::before { content: "✅"; margin-right: 10px; }
        .url { font-size: 24px; color: #8E8E93; font-family: monospace; }
        button { padding: 15px 30px; font-size: 18px; margin: 10px; cursor: pointer; }
    </style>
</head>
<body>
    <h1>Social Media Card Generator</h1>
    <button onclick="downloadCard()">Download as Image</button>
    
    <div id="card" class="card">
        <div class="logo">🍎</div>
        <h1>iOS Interview Preparation</h1>
        <h2>110+ Questions & Answers</h2>
        <div class="features">
            <div class="feature">Swift Programming</div>
            <div class="feature">UIKit & SwiftUI</div>
            <div class="feature">Architecture Patterns</div>
            <div class="feature">Concurrency & Performance</div>
        </div>
        <div class="url">github.com/mukundjogi/ios-interview-prep</div>
    </div>

    <script src="https://html2canvas.hertzen.com/dist/html2canvas.min.js"></script>
    <script>
        function downloadCard() {
            html2canvas(document.getElementById('card')).then(canvas => {
                const link = document.createElement('a');
                link.download = 'og-image.png';
                link.href = canvas.toDataURL();
                link.click();
            });
        }
    </script>
</body>
</html>
```

**Usage:**
1. Save as `generate-social-cards.html`
2. Open in browser
3. Click "Download as Image"
4. Use the generated image

---

## 💡 Pro Tips

1. **Keep it simple** - Don't overcrowd the image
2. **High contrast** - Ensure text is readable at small sizes
3. **Brand consistency** - Use iOS/Swift colors and styling
4. **Test everywhere** - Preview on all platforms before launch
5. **A/B test** - Try different designs to see what performs better
6. **Update regularly** - Refresh when adding major features
7. **Include CTAs** - Star/Fork/Visit prompts work well

---

## 📈 Analytics

Track which images perform best:
- Click-through rates from social media
- Star/fork conversions
- Platform-specific performance
- A/B test different designs

---

Need help creating images? Use the tools and templates above, or consider hiring a designer on:
- Fiverr (budget-friendly)
- 99designs (professional)
- Dribbble (find freelancers)

Good luck! 🚀

