---
name: blog-github-publish
description: >
  Commit and push blog posts (markdown, MDX, HTML) directly to a GitHub
  repository. Handles file placement, frontmatter validation, git commit
  messages, branch selection, and push. Use whenever the user says "publish
  to GitHub", "push my blog post", "commit to repo", "deploy blog post",
  "save to GitHub", or "push to GitHub". Also triggers when the user finishes
  writing a post and wants it saved or published anywhere.
---

# Blog GitHub Publisher

Commits a finished blog post to a GitHub repository and pushes it to the
remote — no manual git commands needed.

## Prerequisites

To push to GitHub, Claude needs:
1. **A GitHub Personal Access Token (PAT)** with `repo` write scope.
   - Create at: https://github.com/settings/tokens → "Generate new token (classic)" → check `repo`
2. **The target repository URL** (e.g. `https://github.com/username/repo`)
3. **The blog post content** (already written, or just generated)

If any of these are missing, ask the user before proceeding.

---

## Workflow

### Step 1: Gather Info

Confirm with the user (or infer from context):

| Item | Default |
|------|---------|
| Repo URL | Ask if unknown |
| Branch | `main` |
| Post file path | `posts/<slug>.md` or `content/<slug>.mdx` — detect from repo structure |
| Commit message | `feat: add blog post "<title>"` |
| GitHub PAT | Must be provided by user |

### Step 2: Validate Frontmatter

Before committing, ensure the post has valid frontmatter. Required fields:

```yaml
---
title: "Your Post Title"
date: YYYY-MM-DD        # today's date if missing
slug: your-post-slug    # kebab-case, derived from title if missing
description: "..."      # 1-2 sentence meta description
tags: [tag1, tag2]      # at least one tag
---
```

Auto-fill any missing fields where possible. Warn the user if `title` is absent.

### Step 3: Determine File Path

Inspect the cloned repo to find where posts live:

```bash
# Common blog post directories — check which exists:
ls content/posts/ 2>/dev/null || \
ls content/ 2>/dev/null || \
ls posts/ 2>/dev/null || \
ls src/content/ 2>/dev/null || \
ls _posts/ 2>/dev/null
```

Use the discovered directory. If ambiguous, ask the user.

### Step 4: Clone & Configure Git

```bash
# Clone using PAT for authenticated push
git clone https://<PAT>@github.com/<owner>/<repo>.git /tmp/blog-publish-repo
cd /tmp/blog-publish-repo

# Configure git identity
git config user.email "claude-blog-publisher@anthropic.com"
git config user.name "Claude Blog Publisher"

# Checkout target branch
git checkout <branch>
```

### Step 5: Write the File

Save the post content to the correct path:

```bash
# Example: posts/my-new-post.md
cat > /tmp/blog-publish-repo/posts/<slug>.md << 'EOF'
<post content here>
EOF
```

### Step 6: Commit & Push

```bash
cd /tmp/blog-publish-repo
git add posts/<slug>.md
git commit -m "feat: add blog post \"<title>\""
git push origin <branch>
```

If the push succeeds, report the commit hash and the file path to the user.

### Step 7: Confirm & Report

Tell the user:
- ✅ Committed: `<commit hash>`
- 📄 File: `posts/<slug>.md`
- 🌿 Branch: `main`
- 🔗 View on GitHub: `https://github.com/<owner>/<repo>/blob/<branch>/posts/<slug>.md`

---

## Error Handling

| Error | Resolution |
|-------|-----------|
| `Authentication failed` | Ask user to re-check PAT — needs `repo` write scope |
| `branch not found` | List available branches; ask user which to use |
| `file already exists` | Ask: overwrite, rename, or cancel? |
| `merge conflict` | Pull latest, resolve by keeping new post, re-push |
| Repo not found | Verify URL; check PAT has access to that repo |

---

## Security Notes

- **Never log or display the PAT** in output or commit messages.
- Use the PAT only for the clone URL; discard after the session.
- Clean up the local clone after pushing: `rm -rf /tmp/blog-publish-repo`

---

## Example Interaction

**User:** "Push my post to GitHub — here's my PAT: ghp_xxxx"

**Claude:**
1. Clones the repo with the PAT
2. Detects post directory (e.g. `posts/`)
3. Validates/fixes frontmatter
4. Writes file as `posts/my-post-slug.md`
5. Commits with message `feat: add blog post "My Post Title"`
6. Pushes to `main`
7. Reports success with commit hash and GitHub link
8. Deletes local clone
