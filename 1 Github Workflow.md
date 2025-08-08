# GitHub Workflow Hands-On: Random Quote Generator with CI/CD

## 🎯 Learning Objectives
- Experience complete GitHub workflow (branch → commit → PR → review → merge)
- Set up GitHub Actions for automated CI/CD
- Deploy a live application using GitHub Pages
- Practice code review and collaboration

## ⏱️ Timeline (60 minutes)
- **0:00-0:05**: Setup & Overview
- **0:05-0:15**: Create Repository & Initial Code
- **0:15-0:25**: Feature Branch Workflow
- **0:25-0:35**: Pull Request & Code Review
- **0:35-0:50**: GitHub Actions CI/CD Setup
- **0:50-0:60**: Deployment & Wrap-up

---

## 📋 Prerequisites
- GitHub account
- Git installed locally
- VS Code (or preferred editor)
- Basic HTML/CSS/JavaScript knowledge

---

## 🚀 Step-by-Step Instructions

### Phase 1: Repository Setup (0:00-0:15)

#### 1.1 Create New Repository (2 min)
```bash
# On GitHub.com:
# 1. Click "New repository"
# 2. Name: "quote-generator-[yourname]"
# 3. Description: "Random quote generator with CI/CD"
# 4. ✅ Public
# 5. ✅ Add README
# 6. ✅ Add .gitignore (Node)
# 7. Create repository
```

#### 1.2 Clone & Initial Setup (3 min)
```bash
# Clone your repository
git clone https://github.com/[username]/quote-generator-[yourname].git
cd quote-generator-[yourname]

# Open in VS Code
code .
```

#### 1.3 Create Initial Files (10 min)
Create these files in your repository:

**index.html**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Random Quote Generator</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="container">
        <h1>Random Quote Generator</h1>
        <div class="quote-box">
            <p id="quote" class="quote">Click the button to get a quote!</p>
            <p id="author" class="author"></p>
        </div>
        <button id="new-quote" class="btn">New Quote</button>
    </div>
    <script src="script.js"></script>
</body>
</html>
```

**styles.css**
```css
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: Arial, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
}

.container {
    text-align: center;
    padding: 2rem;
}

h1 {
    color: white;
    margin-bottom: 2rem;
}

.quote-box {
    background: white;
    padding: 2rem;
    border-radius: 10px;
    box-shadow: 0 10px 20px rgba(0,0,0,0.1);
    margin-bottom: 2rem;
    min-height: 150px;
    display: flex;
    flex-direction: column;
    justify-content: center;
}

.quote {
    font-size: 1.2rem;
    color: #333;
    margin-bottom: 1rem;
}

.author {
    font-style: italic;
    color: #666;
}

.btn {
    background: #667eea;
    color: white;
    border: none;
    padding: 12px 30px;
    font-size: 1rem;
    border-radius: 5px;
    cursor: pointer;
    transition: background 0.3s;
}

.btn:hover {
    background: #5a67d8;
}
```

**script.js**
```javascript
const quotes = [
    { text: "The only way to do great work is to love what you do.", author: "Steve Jobs" },
    { text: "Innovation distinguishes between a leader and a follower.", author: "Steve Jobs" },
    { text: "Life is what happens when you're busy making other plans.", author: "John Lennon" }
];

const quoteElement = document.getElementById('quote');
const authorElement = document.getElementById('author');
const newQuoteBtn = document.getElementById('new-quote');

function getRandomQuote() {
    const randomIndex = Math.floor(Math.random() * quotes.length);
    const quote = quotes[randomIndex];
    quoteElement.textContent = quote.text;
    authorElement.textContent = `- ${quote.author}`;
}

newQuoteBtn.addEventListener('click', getRandomQuote);
```

#### 1.4 Initial Commit (2 min)
```bash
git add .
git commit -m "Initial commit: Basic quote generator"
git push origin main
```

---

### Phase 2: Feature Branch Workflow (0:15-0:25)

#### 2.1 Create Feature Branch (2 min)
```bash
# Create and switch to new branch
git checkout -b feature/add-more-quotes

# Verify you're on the new branch
git branch
```

#### 2.2 Add New Feature (6 min)
Update **script.js** to add more quotes:

```javascript
const quotes = [
    { text: "The only way to do great work is to love what you do.", author: "Steve Jobs" },
    { text: "Innovation distinguishes between a leader and a follower.", author: "Steve Jobs" },
    { text: "Life is what happens when you're busy making other plans.", author: "John Lennon" },
    // ADD THESE NEW QUOTES
    { text: "The way to get started is to quit talking and begin doing.", author: "Walt Disney" },
    { text: "Don't let yesterday take up too much of today.", author: "Will Rogers" },
    { text: "You learn more from failure than from success.", author: "Unknown" },
    { text: "It's not whether you get knocked down, it's whether you get up.", author: "Vince Lombardi" },
    { text: "We generate fears while we sit. We overcome them by action.", author: "Dr. Henry Link" }
];

// ADD THIS NEW FEATURE: Quote counter
let quoteCount = 0;

function getRandomQuote() {
    const randomIndex = Math.floor(Math.random() * quotes.length);
    const quote = quotes[randomIndex];
    quoteElement.textContent = quote.text;
    authorElement.textContent = `- ${quote.author}`;
    
    // Update counter
    quoteCount++;
    updateCounter();
}

function updateCounter() {
    // We'll add the counter element in the next commit
    console.log(`Quotes viewed: ${quoteCount}`);
}

newQuoteBtn.addEventListener('click', getRandomQuote);
```

#### 2.3 Commit and Push Feature Branch (2 min)
```bash
git add script.js
git commit -m "feat: Add more quotes and view counter logic"
git push origin feature/add-more-quotes
```

---

### Phase 3: Pull Request & Code Review (0:25-0:35)

#### 3.1 Create Pull Request (3 min)
1. Go to your GitHub repository
2. Click "Compare & pull request" button
3. Fill in PR template:
   ```
   ## Description
   This PR adds more quotes to our generator and implements a view counter.
   
   ## Changes
   - Added 5 new inspirational quotes
   - Added counter logic to track quote views
   
   ## Testing
   - Clicked "New Quote" button multiple times
   - Verified new quotes appear
   - Console shows view count
   ```
4. Create pull request

#### 3.2 Code Review (Pair Activity) (5 min)
**Partner with classmate:**
1. Exchange repository URLs
2. Review each other's PR
3. Add at least one comment:
   - Suggestion for improvement
   - Question about the code
   - Approval with positive feedback

Example review comments:
```
"Great addition! Could we also show the counter on the page instead of just console?"
"Nice quotes selection! Consider adding a 'Share Quote' feature in the future."
```

#### 3.3 Address Review & Merge (2 min)
1. Respond to review comments
2. If approved, merge the PR
3. Delete the feature branch on GitHub

---

### Phase 4: GitHub Actions CI/CD (0:35-0:50)

#### 4.1 Create GitHub Actions Workflow (5 min)
1. On main branch locally:
```bash
git checkout main
git pull origin main
```

2. Create `.github/workflows/deploy.yml`:
```bash
mkdir -p .github/workflows
```

**deploy.yml**
```yaml
name: Deploy to GitHub Pages

# Triggers
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

# Jobs
jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    # Checkout code
    - uses: actions/checkout@v3
    
    # Run basic tests
    - name: Run HTML validation
      run: |
        npm install -g html-validator-cli
        html-validator --file index.html
    
    # Check JavaScript syntax
    - name: Check JavaScript
      run: |
        node -c script.js
        echo "✅ JavaScript syntax is valid"
    
    # Deploy to GitHub Pages
    - name: Deploy to GitHub Pages
      if: github.event_name == 'push' && github.ref == 'refs/heads/main'
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./
```

#### 4.2 Enable GitHub Pages (3 min)
1. Go to Settings → Pages
2. Source: Deploy from a branch
3. Branch: `gh-pages` (will be created by Action)
4. Save

#### 4.3 Commit and Push Workflow (2 min)
```bash
git add .github/workflows/deploy.yml
git commit -m "ci: Add GitHub Actions workflow for deployment"
git push origin main
```

#### 4.4 Monitor CI/CD Pipeline (5 min)
1. Go to Actions tab on GitHub
2. Watch the workflow run
3. Check for green checkmark
4. Once complete, visit: `https://[username].github.io/quote-generator-[yourname]`

---

### Phase 5: Final Enhancement & Deployment (0:50-0:60)

#### 5.1 Quick Enhancement (5 min)
Create new branch for visible counter:
```bash
git checkout -b feature/visible-counter
```

Update **index.html** (add after the button):
```html
<p class="counter">Quotes viewed: <span id="counter">0</span></p>
```

Update **styles.css** (add at end):
```css
.counter {
    color: white;
    margin-top: 1rem;
    font-size: 0.9rem;
}
```

Update **script.js** (modify updateCounter function):
```javascript
function updateCounter() {
    const counterElement = document.getElementById('counter');
    if (counterElement) {
        counterElement.textContent = quoteCount;
    }
}
```

#### 5.2 Fast PR & Merge (3 min)
```bash
git add .
git commit -m "feat: Add visible quote counter"
git push origin feature/visible-counter
```

1. Create PR on GitHub
2. Self-merge (for time)
3. Watch CI/CD redeploy

#### 5.3 Verify Deployment (2 min)
1. Wait for Actions to complete
2. Refresh your GitHub Pages URL
3. Test the counter feature
4. Celebrate! 🎉

---

## 📊 What You've Accomplished

✅ **GitHub Workflow**
- Created repository
- Worked with branches
- Made pull requests
- Performed code review
- Merged changes

✅ **CI/CD Pipeline**
- Set up GitHub Actions
- Automated testing
- Automated deployment
- GitHub Pages hosting

✅ **Collaboration**
- Peer code review
- Issue tracking
- Git workflow

---

## 🏠 Homework Extensions

1. **Add Tests**: Create a `tests.js` file with actual unit tests
2. **Enhance CI**: Add ESLint to the GitHub Actions workflow
3. **New Features**: 
   - Add categories to quotes
   - Implement quote sharing
   - Add animations
4. **Documentation**: Create comprehensive README with badges

---

## 🔗 Resources

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Flow Guide](https://guides.github.com/introduction/flow/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)

---

## 🎯 Success Indicators

By the end of this hour, you should have:
1. ✅ A live website at `https://[username].github.io/quote-generator-[yourname]`
2. ✅ At least 2 merged pull requests
3. ✅ A working CI/CD pipeline
4. ✅ Experience with the complete GitHub workflow
5. ✅ Understanding of modern development practices