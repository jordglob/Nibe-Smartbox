# GitHub Desktop Upload Guide - Nibe Smartbox v1.0

## 📋 Step-by-Step Instructions

### Step 1: Open GitHub Desktop

1. Launch **GitHub Desktop** application
2. If this is your first time, sign in with your GitHub account

### Step 2: Add This Repository

**Option A: If you see "Add an Existing Repository"**
1. Click **"Add an Existing Repository from your hard drive"**
2. Click **"Choose..."** button
3. Navigate to: `C:\Users\jordg\Documents\GitHub\Nibe Smartbox`
4. Click **"Select Folder"**

**Option B: From the menu**
1. Click **File** → **Add local repository...**
2. Click **"Choose..."** button
3. Navigate to: `C:\Users\jordg\Documents\GitHub\Nibe Smartbox`
4. Click **"Select Folder"**

**If you see "This directory does not appear to be a Git repository":**
1. Click **"create a repository"** link
2. GitHub Desktop will initialize git for you

### Step 3: Review Changes

You should see all your files listed in the left panel:
- ✅ smartbox.yaml
- ✅ README.md
- ✅ BUILD_GUIDE.md
- ✅ CHANGELOG.md
- ✅ LICENSE
- ✅ .gitignore

All files should have a **green +** icon (new files)

### Step 4: Create Initial Commit

1. In the bottom-left corner, you'll see:
   - **Summary** field (required)
   - **Description** field (optional)

2. Enter commit message:
   ```
   Summary: Release v1.0.0 - Nibe Smartbox
   
   Description:
   Commercial-grade smart controller for Nibe F1245 heat pumps
   - Autonomous spot price optimization
   - UDP Gateway (NibeGW) for Home Assistant
   - Web Server v3 for local monitoring
   - Verified working on LilyGo T-CAN485 v1.1
   ```

3. Click the blue **"Commit to main"** button

### Step 5: Publish to GitHub

1. After committing, you'll see a button: **"Publish repository"**
2. Click **"Publish repository"**

3. A dialog will appear:
   - **Name**: `nibe-smartbox` (or your preferred name)
   - **Description**: `Commercial-grade smart controller for Nibe heat pumps with autonomous electricity spot price optimization`
   - **Keep this code private**: ☐ Uncheck (make it public)
   - **Organization**: None (unless you want to publish to an org)

4. Click **"Publish repository"**

### Step 6: Verify Upload

1. GitHub Desktop will upload all files (may take 10-30 seconds)
2. When complete, you'll see: **"Last fetched just now"** at the top
3. Click **"View on GitHub"** button to see your repository online

### Step 7: Create Release Tag (Optional but Recommended)

**In GitHub Desktop:**
1. Click **Repository** → **Create tag...**
2. Tag name: `v1.0.0`
3. Description: `Release v1.0.0 - Initial production release`
4. Click **"Create tag"**
5. Click **Repository** → **Push** (or click "Push origin" button)

**On GitHub.com:**
1. Go to your repository: `https://github.com/jordglob/nibe-smartbox`
2. Click **"Releases"** (right side)
3. Click **"Draft a new release"**
4. Choose tag: `v1.0.0`
5. Release title: `v1.0.0 - Nibe Smartbox Initial Release`
6. Description: Copy from CHANGELOG.md
7. Click **"Publish release"**

---

## ✅ Success Checklist

After upload, verify on GitHub.com:

- [ ] All 6 files are visible
- [ ] README.md displays nicely with badges
- [ ] LICENSE shows MIT
- [ ] Repository has a description
- [ ] Topics/tags added (optional): `esphome`, `nibe`, `heat-pump`, `smart-home`, `esp32`

---

## 🎨 Optional: Make it Look Professional

### Add Topics (Tags)

1. On your GitHub repository page
2. Click the ⚙️ gear icon next to "About"
3. Add topics:
   - `esphome`
   - `nibe`
   - `heat-pump`
   - `smart-home`
   - `esp32`
   - `home-automation`
   - `spot-price`
   - `energy-optimization`

### Add Repository Description

1. Click the ⚙️ gear icon next to "About"
2. Description: `Commercial-grade smart controller for Nibe heat pumps with autonomous electricity spot price optimization`
3. Website: (leave blank or add your website)
4. Check ✅ **"Releases"**
5. Check ✅ **"Packages"** (if applicable)
6. Click **"Save changes"**

---

## 🔧 Troubleshooting

### Problem: "This directory does not appear to be a Git repository"

**Solution:**
1. Click **"create a repository"** in the dialog
2. GitHub Desktop will initialize git automatically
3. Continue with Step 3

### Problem: Can't find the folder

**Solution:**
1. Copy this path: `C:\Users\jordg\Documents\GitHub\Nibe Smartbox`
2. In the folder selection dialog, paste it in the address bar
3. Press Enter

### Problem: Files not showing up

**Solution:**
1. Make sure you're in the correct folder
2. Check that .gitignore isn't excluding files
3. Refresh GitHub Desktop (Ctrl+R)

### Problem: "Failed to publish repository"

**Solution:**
1. Check your internet connection
2. Verify you're signed in to GitHub Desktop
3. Try again - sometimes it's just a temporary network issue

---

## 📞 Need Help?

- **GitHub Desktop Docs**: https://docs.github.com/en/desktop
- **GitHub Support**: https://support.github.com/

---

**Once uploaded, your repository will be live at:**
`https://github.com/jordglob/nibe-smartbox`

**Share it with the community! 🎉**
