# Claude Code Quickstart

> 📅 2026-07-27 · Getting Started
> The first post in the Claude Code tutorial series — get up and running with installation and basic usage in no time.

---

This is the first post in the Claude Code tutorial series. More in-depth content is coming.

***

## Claude Code Quickstart

### Who this is for

You already have basic terminal skills and want to genuinely integrate AI into your local development workflow. You want AI to be more than just "writing a few lines of code" — you want it to actually move tasks forward, read your project, modify code, run commands, and verify results.

***

### Installation

```bash
npm install -g @anthropic-ai/claude-code
```

After installing, confirm the version:

```bash
claude --version
```

The first time you use it, you'll need to log in with your Claude account:

```bash
claude
```

Claude Code will guide you through the authorization flow.

***

### Core concept: start in the right directory

Claude Code isn't a chat box detached from project context. It reads the code and file structure of your current directory directly.

**Basic principles:**

* Enter the project directory before launching Claude Code
* Don't just run it in cluttered directories like your Desktop or Downloads
* If you don't have a project yet, create an empty directory first, then let Claude Code start working

***

### Best practice for beginners: the sandwich collaboration rhythm

The working rhythm that suits Claude Code best:

```
Understand → Plan → Execute
```

#### Step 1: Let it understand the project first

```
First, take a look at the overall structure of this project.
Tell me what the main directories are for, and roughly which files the homepage-related code lives in.
Don't change any code yet.
```

#### Step 2: Ask for a plan

```
I want to add a carousel to the homepage.
First give me an implementation plan — tell me which files you plan to modify and how.
Don't execute anything yet.
```

#### Step 3: Execute only after confirmation

```
You can start making changes.
After you're done, run a build check and tell me which files you changed.
```

***

### Building a new project from scratch

When you want Claude Code to help you set up a new project:

```
I'm in an empty directory and want to build a Next.js + TypeScript project from scratch.

First tell me the initialization approach you recommend:
- What scaffold to use
- Which dependencies will be installed
- Which key directories will be created

Don't execute anything yet.

Once I confirm, start building the project.
When it's done, start the project and tell me whether it ran successfully.
```

**Don't mix "set up a project" and "develop features" into a single sentence.**

***

### Common beginner mistakes

| Mistake | Correct approach |
|------|----------|
| Launching on your Desktop or in Downloads | cd into the project directory first |
| Saying "make me a project" in one sentence | Be specific about the stack: Next.js + TypeScript |
| Not saying "don't change code yet" | Analyze first → plan next → execute last |
| Only checking "whether it finished" and ignoring error messages | Always read the build result |

***

### Small tasks that are best for practice

* Style tweaks on a single page
* Refactoring a function
* Writing a set of unit tests
* Modifying a piece of API-call logic
