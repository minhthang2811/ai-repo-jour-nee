---
name: repository-to-html
description: Generates a stunning, single-file HTML explainer page with Three.js 3D animations for any GitHub repository. Analyzes the repo's README, docs, and structure, then produces an interactive visual showcase with a 3D architecture diagram, glassmorphism UI, scroll animations, and dark-themed modern design. Use this skill whenever the user wants to create a visual explainer, showcase, or presentation page for a GitHub repository. Trigger on mentions of "repository-to-html", "repo to html", "repo explainer page", "visualize this repository", "make an HTML page for this repo", "explain this repo visually", "turn this repo into a webpage", or any request involving converting a GitHub repository URL into a beautiful interactive HTML page — even if the user just pastes a GitHub link and says "make it look good" or "explain this project."
---

# Repository to HTML

Transform any GitHub repository into a breathtaking, self-contained HTML explainer page. The output is a single HTML file with Three.js 3D animations, glassmorphism UI, and scroll-driven effects that explains the project like a premium product landing page — not like documentation.

## Workflow

### Step 1: Parse the Repository URL

Extract owner and repository name from the GitHub URL.

Example: `https://github.com/obra/superpowers` → owner=`obra`, repo=`superpowers`

Handle variations: trailing slashes, `.git` suffix, URLs with `/tree/main` paths, etc. Strip them to get the clean `owner/repo`.

### Step 2: Fetch Repository Data

Use `read_url_content` to gather information. Fetch these in parallel where possible:

1. **GitHub API — repo metadata**
   ```
   https://api.github.com/repos/{owner}/{repo}
   ```
   Extract: name, description, stargazers_count, forks_count, language, topics, created_at, updated_at, homepage

2. **README (raw markdown)**
   ```
   https://raw.githubusercontent.com/{owner}/{repo}/main/README.md
   ```
   If 404, try `master` branch. This is the primary information source.

3. **Repository tree (directory structure)**
   ```
   https://api.github.com/repos/{owner}/{repo}/git/trees/main?recursive=1
   ```
   If 404, try `master`. Use this to understand the project architecture — major directories, key files, what languages are used.

4. **Documentation files** — check the tree for these and fetch any that exist:
   - `docs/` or `documentation/` directories
   - `ARCHITECTURE.md`, `DESIGN.md`, `CONTRIBUTING.md`
   - Any root-level `.md` files beyond README

If the GitHub API returns rate-limit errors, fall back to scraping the repo's main page with `read_url_content` on `https://github.com/{owner}/{repo}`.

### Step 3: Analyze and Extract

From the gathered data, produce four analysis blocks. Think deeply about each — the quality of the explainer depends on your understanding.

**Block 1 — What Is This?**
A clear, jargon-light explanation of what the repository is. Lead with the value proposition: what does this give the user? Not "a Python library that implements..." but "a way to...". One paragraph, 3-5 sentences.

**Block 2 — The Problem**
What pain point does this solve? Why does it exist? What was the world like before this tool? Make it relatable — even a non-technical person should understand the frustration this project addresses. Think about the "before and after" narrative.

**Block 3 — How It Works (Architecture)**
This is the most important analysis — it drives the pipeline visualization. Identify **4-8 key components or steps** in the system:
- What are the major modules, services, or processing stages?
- What is the data flow or pipeline?
- How do the pieces connect to each other?
- What are the inputs and outputs?

For each component, write: a short label (2-4 words), a one-sentence description, and an icon emoji that represents it. Order them as a sequential pipeline flow.

**Block 4 — How to Use**
Pull installation steps, basic usage commands, and a quickstart example directly from the README. Include:
- Prerequisites (Node.js, Python version, etc.)
- Installation command(s)
- Basic usage / first command to run
- One concrete example showing input → output

### Step 4: Generate the HTML

Create a single, self-contained HTML file. **Read `references/html-template-guide.md`** for the complete code patterns, CSS design system, and section-by-section template code.

Key technical requirements:
- **Single file** — all CSS in `<style>`, all JS in `<script>`, no external files except CDN
- **CDN dependencies** (loaded in `<head>`):
  - Three.js: `https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js` (hero particles only)
  - GSAP + ScrollTrigger: `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/gsap.min.js` and `https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.2/ScrollTrigger.min.js`
  - Google Fonts: Inter (body) and JetBrains Mono (code)
- **Dark theme** with glassmorphism cards
- **Fully responsive** — works on mobile, tablet, desktop
- **Smooth scroll** navigation with fixed nav bar
- **CSS pipeline timeline** for architecture (not canvas — pure CSS is more readable and performant)
- **Three.js** only for the hero particle background

### Step 5: Save the Output

1. Create a directory named after the repository in the current working directory: `{repo-name}/`
2. Save the HTML file inside as `{repo-name}/index.html`
3. Tell the user the path and suggest: `open {repo-name}/index.html`

---

## Design Principles

The output should feel like a **premium product landing page** — think Apple's product pages, Stripe's documentation, or Linear's marketing site. Not a wiki. Not a README rendered as HTML. A showcase.

### Color System
| Role | Value | Usage |
|------|-------|-------|
| Background | `#0a0a0f` → `#1a1a2e` gradient | Page background |
| Primary accent | `#667eea` → `#764ba2` gradient | Headings, CTAs, glows |
| Secondary accent | `#00d2ff` | Highlights, node connections |
| Tertiary | `#f093fb` | Hover states, secondary glow |
| Text primary | `#ffffff` | Headings, important text |
| Text secondary | `#8892b0` | Body text, descriptions |
| Glass surface | `rgba(255,255,255,0.05)` | Cards, panels |
| Glass border | `rgba(255,255,255,0.1)` | Card borders |

### Typography
- **Font stack**: `'Inter', -apple-system, sans-serif` for body; `'JetBrains Mono', monospace` for code
- **Hero title**: `clamp(2.5rem, 5vw, 4.5rem)`, font-weight 800
- **Section titles**: `clamp(1.8rem, 3vw, 2.8rem)`, font-weight 700
- **Body**: `1.1rem`, line-height 1.7, color `#8892b0`
- **Code**: `0.9rem`, background `rgba(0,0,0,0.3)`, rounded corners

### Animations
- **Scroll-triggered** — sections fade up on scroll (GSAP ScrollTrigger). **Important**: use `gsap.set()` to hide elements, then `ScrollTrigger.create({ once: true })` with `gsap.to()` to reveal them. Do NOT use `gsap.from()` with ScrollTrigger — it sets elements to `opacity: 0` on page load and they may never become visible if the trigger doesn't fire correctly.
- **Three.js hero** — animated particle/star field behind the hero text
- **CSS pipeline** — animated vertical timeline with glowing nodes and flowing particles for architecture
- **Hover effects** — cards lift and glow on hover (CSS transform + box-shadow)
- **Gradient text** — headings use animated gradient fills
- **Smooth scroll** — CSS `scroll-behavior: smooth` with nav anchors

---

## Page Structure

### Navigation
Fixed top nav bar with glassmorphism background. Contains:
- Repo name (left)
- Section links: Overview · Problem · Architecture · Usage (center/right)
- GitHub link icon (far right, links to repo)

### Section 1: Hero (full viewport)
- Three.js particle/star field background (canvas behind content)
- Repository name in large glowing gradient text
- One-line description from the repo metadata
- Stats row: ⭐ {stars} · 🍴 {forks} · 💻 {language}
- Animated "Explore ↓" button scrolling to next section
- Subtle floating particles or grid lines for depth

### Section 2: The Problem
- Section title with gradient underline
- Glassmorphism card containing the problem narrative
- Optional: "Before" vs "After" comparison if the data supports it
- Scroll-triggered fade-in animation
- CSS-animated decorative elements (floating dots, subtle gradients)

### Section 3: How It Works (the star of the show)
- Section title
- **CSS pipeline timeline** showing the architecture:
  - Vertical glowing gradient spine line running through the center
  - Animated particle dot flowing down the spine
  - Color-coded numbered node circles (01, 02, ...) on the spine with pulse animations
  - Glassmorphism cards alternating left and right, connected to nodes by gradient lines
  - Each card has an icon, title, and description
  - Cards lift and glow in their unique color on hover
  - On mobile: collapses to a left-aligned vertical timeline
- This approach is far more readable than a 3D canvas — users can scan the flow instantly without needing to interact

### Section 4: How to Use
- Step-by-step installation with numbered glassmorphism cards
- Dark code blocks with syntax highlighting (use `<pre><code>` with custom CSS)
- Copy-to-clipboard buttons on code blocks
- Example usage in a terminal-style card

### Footer
- Horizontal gradient line separator
- "Built from" + link to the GitHub repository
- Subtle "Generated with ❤️" text
- Current year

---

## Quality Checklist

Before saving the file, verify:

- [ ] Three.js hero scene renders without console errors
- [ ] All four sections are present and populated with real data (no placeholder text)
- [ ] The architecture pipeline has steps matching the actual project components
- [ ] Navigation links scroll to correct sections
- [ ] Page is responsive (test with mobile viewport mentally)
- [ ] Code blocks contain actual commands from the repo, not generic examples
- [ ] Stats (stars, forks, language) are real values from the API
- [ ] All CDN URLs are correct and use HTTPS
- [ ] The file opens and works as a standalone HTML file
- [ ] Text is readable (sufficient contrast, appropriate sizing)
- [ ] GSAP scroll animations use the set/to/once pattern (NOT gsap.from with ScrollTrigger)
- [ ] All section content (cards, steps, descriptions) is visible after scrolling — not stuck at opacity 0

## Error Handling

- If the GitHub API is rate-limited, inform the user and suggest they try again later or provide a GitHub token
- If the README is empty or missing, use the repo description and directory structure to infer the project's purpose
- If the repo has very few files (< 5), simplify the architecture section to a linear flow rather than a complex node graph
- Always produce output even with partial data — a beautiful page with 80% of the info is better than no page
