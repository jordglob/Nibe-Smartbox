# VS Code Git Integration Guide

**How to commit and push changes to GitHub directly from Visual Studio Code**

---

## 🎯 Overview

VS Code has built-in Git support that makes it easy to commit and push changes without using GitHub Desktop or command line. This guide shows you how to use it.

---

## 📋 Step-by-Step Instructions

### Step 1: Open Source Control Panel

**Method 1: Click the icon**
- Look at the left sidebar in VS Code
- Click the **Source Control** icon (looks like a branching diagram)
- Keyboard shortcut: `Ctrl+Shift+G`

**Method 2: From menu**
- Click **View** → **Source Control**

### Step 2: Review Your Changes

In the Source Control panel, you'll see:
- **Changes** section - Lists all modified files
- Each file shows what type of change:
  - **M** = Modified (yellow)
  - **U** = Untracked/New (green)
  - **D** = Deleted (red)

**Example for v1.2:**
```
Changes (4)
  M smartbox.yaml
  M CHANGELOG.md
  M BUILD_GUIDE.md
  M README.md
```

### Step 3: Stage Changes

**Option A: Stage All Files**
1. Hover over **"Changes"** heading
2. Click the **+** (plus) icon that appears
3. All files move to **"Staged Changes"** section

**Option B: Stage Individual Files**
1. Hover over a specific file
2. Click the **+** icon next to that file
3. Only that file is staged

**What is staging?**
- Staging = selecting which changes to include in your commit
- You can stage some files and leave others for later

### Step 4: Write Commit Message

At the top of the Source Control panel:

1. **Message box** - Click where it says "Message (Ctrl+Enter to commit)"
2. Type your commit message:
   ```
   v1.2.0 - Fix sensor architecture + Standalone prep
   
   - Commented out homeassistant platform sensors (circular dependency)
   - Increased buffer to 16KB (13.6KB response verified)
   - Added Standalone Visualization to roadmap
   - Updated architecture documentation
   ```

**Tips for good commit messages:**
- First line: Short summary (50 chars or less)
- Blank line
- Detailed description (what and why)

### Step 5: Commit Changes

**Method 1: Keyboard shortcut**
- Press `Ctrl+Enter` while in the message box

**Method 2: Click button**
- Click the **✓ Commit** button above the message box

**Method 3: Dropdown menu**
- Click the **▼** arrow next to Commit
- Select **"Commit"** or **"Commit & Push"**

### Step 6: Push to GitHub

After committing, you need to push to GitHub:

**Method 1: Click the sync button**
- Look for **"Sync Changes"** button (cloud icon with arrows)
- Click it to push your commits

**Method 2: Use the menu**
- Click the **...** (three dots) at the top of Source Control panel
- Select **"Push"**

**Method 3: Status bar**
- Look at the bottom-left of VS Code
- Click the **↑** (upload) icon with a number
- This shows how many commits need to be pushed

### Step 7: Verify on GitHub

1. Open your browser
2. Go to: `https://github.com/jordglob/Nibe-Smartbox`
3. Refresh the page
4. You should see your new commit at the top
5. Check that all files are updated

---

## 🔧 Common Operations

### View File Changes (Diff)

1. In Source Control panel, click on a modified file
2. VS Code opens a **side-by-side diff view**:
   - Left side: Old version (red highlights = removed)
   - Right side: New version (green highlights = added)

### Discard Changes

**To discard changes in a file:**
1. Hover over the file in Changes
2. Click the **↶** (curved arrow) icon
3. Confirm "Discard changes"

⚠️ **Warning**: This permanently deletes your changes!

### View Commit History

1. Click the **...** menu in Source Control
2. Select **"View History"** (requires Git Graph extension)
3. Or use: `git log` in terminal

### Create a Branch

1. Click the branch name in bottom-left (currently "main")
2. Select **"Create new branch..."**
3. Enter branch name (e.g., `feature/husdata-visualization`)
4. Press Enter

### Pull Latest Changes

If someone else pushed changes:
1. Click the **...** menu
2. Select **"Pull"**
3. Or click the **↓** (download) icon in status bar

---

## 🎨 VS Code Git Extensions (Optional)

### Recommended Extensions

**1. GitLens** (Highly recommended)
- Shows who changed each line of code
- Inline blame annotations
- Rich commit history
- Install: Search "GitLens" in Extensions (Ctrl+Shift+X)

**2. Git Graph**
- Visual commit history
- Branch visualization
- Install: Search "Git Graph" in Extensions

**3. Git History**
- View file history
- Compare versions
- Install: Search "Git History" in Extensions

### Installing Extensions

1. Click Extensions icon (left sidebar) or press `Ctrl+Shift+X`
2. Search for extension name
3. Click **"Install"**
4. Reload VS Code if prompted

---

## 🚀 Quick Reference

### Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Open Source Control | `Ctrl+Shift+G` |
| Commit | `Ctrl+Enter` |
| Open Terminal | `` Ctrl+` `` |
| Command Palette | `Ctrl+Shift+P` |

### Git Commands in VS Code Terminal

```bash
# View status
git status

# View commit history
git log --oneline

# View current branch
git branch

# Pull latest changes
git pull

# Push commits
git push

# Create and switch to new branch
git checkout -b feature/my-feature
```

---

## 🔧 Troubleshooting

### Problem: "Please tell me who you are"

**Solution**: Configure Git identity
1. Open Terminal in VS Code (`` Ctrl+` ``)
2. Run these commands:
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

### Problem: Can't see Source Control panel

**Solution**:
1. Press `Ctrl+Shift+G`
2. Or: View → Source Control

### Problem: "No repository found"

**Solution**:
1. Make sure you're in the correct folder
2. Check if `.git` folder exists
3. If not, initialize: `git init` in terminal

### Problem: Push fails with authentication error

**Solution**:
1. VS Code will prompt for GitHub credentials
2. Use your GitHub username and **Personal Access Token** (not password)
3. Create token at: https://github.com/settings/tokens

### Problem: Merge conflicts

**Solution**:
1. VS Code highlights conflicts in files
2. Click **"Accept Current Change"** or **"Accept Incoming Change"**
3. Or manually edit the file
4. Stage the resolved file
5. Commit

---

## 📊 Comparison: VS Code vs GitHub Desktop

| Feature | VS Code | GitHub Desktop |
|---------|---------|----------------|
| **Integrated** | ✅ Built into editor | ❌ Separate app |
| **File editing** | ✅ Immediate | ❌ Switch apps |
| **Diff view** | ✅ Side-by-side | ✅ Side-by-side |
| **Commit** | ✅ Fast | ✅ Easy |
| **Visual** | ⚠️ Less visual | ✅ Very visual |
| **Beginner-friendly** | ⚠️ Moderate | ✅ Very easy |

**Recommendation**: 
- **Beginners**: Use GitHub Desktop (easier to understand)
- **Developers**: Use VS Code (faster workflow)
- **Both**: You can use both! They work together perfectly

---

## 🎓 Best Practices

### Commit Messages

**Good:**
```
v1.2.0 - Fix sensor architecture

- Commented out homeassistant platform sensors
- Added Standalone roadmap
- Updated documentation
```

**Bad:**
```
fixed stuff
```

### Commit Frequency

- ✅ Commit after each logical change
- ✅ Commit working code (compiles successfully)
- ❌ Don't commit broken code
- ❌ Don't commit sensitive data (passwords, API keys)

### Before Pushing

**Checklist:**
- [ ] Code compiles successfully
- [ ] Tested on device (if applicable)
- [ ] Documentation updated
- [ ] No sensitive data in files
- [ ] Commit message is clear

---

## 📞 Need Help?

- **VS Code Git Docs**: https://code.visualstudio.com/docs/sourcecontrol/overview
- **Git Basics**: https://git-scm.com/book/en/v2/Getting-Started-Git-Basics
- **GitHub Docs**: https://docs.github.com/

---

## 🎯 Quick Workflow Example

1. **Open Source Control** (`Ctrl+Shift+G`)
2. **Review changes** (modified files listed)
3. **Stage all** (click + next to "Changes")
4. **Write message**:
   ```
   v1.3.0 - Timing-safe spot price controller
   
   - 15-minute price granularity
   - Control Mode "The Brain" (3 modes)
   - 17 bug fixes for MODBUS timing safety
   - Reorganized docs/ folder structure
   ```
5. **Commit** (`Ctrl+Enter`)
6. **Push** (click "Sync Changes" button)
7. **Verify** on GitHub.com

**Done! Your release is live! 🎉**

---

**Last Updated**: 2026-03-17  
**Version**: 1.3
