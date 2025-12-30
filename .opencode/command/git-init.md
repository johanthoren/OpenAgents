---
description: Initialize git repository with GitHub remote
model: anthropic/claude-4-5-haiku  # Cheaper model works well for this mechanical task
---

# Git Repository Initializer

Initialize a new git repository with a GitHub remote using sensible defaults.

## Instructions

This command initializes a local git repository, creates a private GitHub repository, and pushes an initial commit with a README and .gitignore.

### 1. **Check if Already in Git Repository**

First, verify we're not already in a git repository:

!`git rev-parse --git-dir 2>/dev/null && echo "Already in git repository" || echo "Not in git repository"`

If already in a git repository, exit with error:

```
Already in a git repository. Cannot initialize here.
Use 'git status' to see the current repository status.
```

### 2. **Determine Repository Name**

Get the repository name from arguments or use current directory name:

Argument provided: $ARGUMENTS

If argument provided, use it as repository name.
If no argument, use current directory name:

!`basename $(pwd)`

Store the repository name for later use.

### 3. **Check if GitHub Repository Exists**

Check if a repository with this name already exists on GitHub:

!`gh repo view [repo-name] 2>&1 || echo "Repository not found"`

If repository exists, exit with error:

```
GitHub repository '[repo-name]' already exists.
Choose a different name or use the existing repository.
```

### 4. **Initialize Local Git Repository**

Initialize the local git repository:

!`git init`

### 5. **Create GitHub Repository**

Create a private GitHub repository:

!`gh repo create [repo-name] --private --source=. --remote=origin`

This creates a private repository and sets it as the origin remote.

### 6. **Create Initial Files**

Create a basic README.md:

```bash
cat > README.md << 'EOF'
# [repo-name]

[Add description here]
EOF
```

Create a basic .gitignore with common patterns:

```bash
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
vendor/

# Environment files
.env
.env.local

# OS files
.DS_Store
Thumbs.db

# IDE files
.vscode/
.idea/
*.swp
*.swo
EOF
```

### 7. **Initial Commit and Push**

Add all files, commit, and push to GitHub:

!`git add README.md .gitignore`

!`git commit -m "Initial commit"`

!`git push -u origin main`

### 8. **Success Message**

```
✅ Repository initialized successfully!

Repository: [repo-name]
GitHub URL: https://github.com/[username]/[repo-name]

Next steps:
- Add files to your repository
- Update README.md with project description
- Customize .gitignore for your project type
```

### Error Handling

- If `gh` not installed: "GitHub CLI required. Install from: https://cli.github.com"
- If not authenticated: "Run 'gh auth login' to authenticate with GitHub"
- If git not installed: "Git required. Install from: https://git-scm.com"
