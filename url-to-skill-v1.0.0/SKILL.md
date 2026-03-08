---
name: url-to-skill
description: >
  Convert any GitHub repository URL or documentation website URL into a structured
  Claude Code skill file. Automatically fetches, analyzes, and packages the content
  into a reusable skill saved to ~/.claude/skills/. Use this skill whenever the user
  wants to turn a URL into a skill, create a skill from a GitHub repo, generate a
  skill from documentation, or says things like "make a skill from this link",
  "convert this repo to a skill", or "I want a skill for [framework/library]".
  Also trigger when users paste a GitHub URL or docs URL and ask to "learn" it,
  "remember" it, or "create context" from it.
---

# URL to Skill

Turn any GitHub repository or documentation website into a structured Claude Code skill file.

## How It Works

This skill follows a 5-step pipeline: detect URL type → fetch content → analyze → generate skill → save.

## Step 1: Detect URL Type

Classify the input URL:

- **GitHub repo**: matches `github.com/{owner}/{repo}` or shorthand `{owner}/{repo}`
- **Documentation site**: everything else (any HTTP/HTTPS URL)

Extract `owner` and `repo` from GitHub URLs by parsing the path. Strip trailing slashes, `.git` suffixes, and extra path segments (like `/tree/main/...`).

## Step 2: Fetch Content

### For GitHub Repositories

Run these commands to gather repo data. Launch independent calls in parallel where possible.

**Metadata + README + file tree (parallel):**

```bash
# Repo metadata (description, language, topics, default branch)
gh api repos/{owner}/{repo} --jq '{description, language, default_branch, stargazers_count, topics}'

# README content (base64-encoded)
gh api repos/{owner}/{repo}/readme --jq '.content' | base64 -d

# Full file tree
gh api repos/{owner}/{repo}/git/trees/{default_branch}?recursive=1 --jq '.tree[] | select(.type=="blob") | .path'
```

**Then identify and fetch key files:**

1. From the file tree, pick 3-5 core source files based on:
   - Entry points: `main.*`, `index.*`, `app.*`, `cli.*`
   - Source directories: `src/`, `lib/`, `pkg/`, `internal/`
   - Config files: `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`
   - The repo's primary language
2. If `CLAUDE.md` or `AGENTS.md` exists, fetch it — it contains project conventions
3. Fetch each file:
   ```bash
   gh api repos/{owner}/{repo}/contents/{path} --jq '.content' | base64 -d
   ```

**Priority order**: README > project config > source entry points > other source files

### For Documentation Websites

1. Use `WebFetch` on the main URL to get the landing page content
2. From the page, identify navigation links pointing to key sections:
   - Getting started / quickstart / installation
   - API reference / reference docs
   - Core concepts / guides / tutorials
3. Fetch 3-5 of the most important pages using `WebFetch`
4. Extract: code examples, API signatures, configuration patterns, best practices

**Priority order**: Getting started > API reference > Guides > Tutorials

## Step 3: Analyze Content

From all fetched content, extract and organize:

| Category | What to look for |
|----------|-----------------|
| **Identity** | Project name, one-line purpose, version, license |
| **Core concepts** | Key terminology, mental models, architecture overview |
| **API surface** | Functions, classes, methods — with signatures and brief descriptions |
| **Patterns** | Common usage patterns with real code examples from the source |
| **Best practices** | Do's and don'ts, anti-patterns, performance tips |
| **Structure** | Typical project layout, file conventions, naming conventions |
| **Dependencies** | Prerequisites, required tools, compatible versions |

Focus on what a developer would need to be productive with this project. Skip marketing content, contributor guidelines, and changelog details.

## Step 4: Generate the Skill File

Write a SKILL.md with YAML frontmatter and structured markdown body.

**Frontmatter template:**

```yaml
---
name: {project-name-lowercase}
description: >
  Provides expertise on {Project Name} — {one-line description}.
  Use this skill when working with {project name}, its APIs, patterns,
  or when the user asks about {key topics}. Also trigger for related
  terms like {aliases, abbreviations, related concepts}.
---
```

The description is the trigger mechanism — make it specific and slightly "pushy" so it activates when relevant. Include concrete keywords and phrases users might say.

**Body structure:**

```markdown
# {Project Name}

{2-3 sentence overview: what it is, what it's for, key differentiator}

## Core Concepts

{Key terminology and mental models, explained concisely}

## API Reference

{Most important APIs with signatures and one-line descriptions}
{Use actual signatures from the source, not placeholders}

### {Category 1}

- `functionName(param: Type): ReturnType` — what it does
- `ClassName` — what it represents
  - `.method(args)` — what the method does

## Common Patterns

{3-5 real-world usage patterns with code examples}
{Use code blocks with language tags}

### {Pattern Name}

```{language}
// Real code example extracted from the source
```

## Best Practices

- **Do**: {positive practice with brief reason}
- **Don't**: {anti-pattern with brief reason}

## Project Structure

```
typical-project/
├── src/          — {what goes here}
├── tests/        — {what goes here}
└── config.file   — {what this configures}
```

## Quick Reference

{Cheat sheet: most common operations as a compact list}

| Task | How |
|------|-----|
| {common task} | `{command or code}` |
```

**Guidelines for the generated content:**
- Aim for 200-500 lines total — enough to be useful, short enough to fit in context
- Every code example must come from the actual source, not invented
- Use the project's real API signatures, types, and naming conventions
- If information is missing or couldn't be fetched, note it briefly and move on
- Organize API reference by logical category, not alphabetically

## Step 5: Save the Skill

1. Create the output directory if needed:
   ```bash
   mkdir -p ~/.claude/skills/{project-name}
   ```

2. Write the skill file to `~/.claude/skills/{project-name}/SKILL.md`

3. Report to the user:
   - File path where the skill was saved
   - Brief summary: what the skill covers, how many APIs/patterns were captured
   - Any content that couldn't be fetched (with reason)
   - Suggest: "You can now use this skill by asking questions about {project name}. To update it later, just run `/url-to-skill {url}` again."

## Error Handling

- If `gh` CLI is not authenticated or available, tell the user to run `gh auth login` first
- If WebFetch fails on a URL, skip it and note what was missed — don't abort the whole process
- If the repo is empty or the docs site has no useful content, explain what happened and suggest alternatives (e.g., "Try pointing to the docs site instead" or "This repo has minimal documentation")
- If a file is too large to fetch via the API (>1MB), skip it and pick a smaller alternative
