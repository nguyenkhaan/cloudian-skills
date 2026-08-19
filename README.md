# CLOUDIAN SKILLS  
## 1. Overview 

This project contains some necessary things for AI Agents. It includes: Skill markdown files, rule markdown files and AGENT.md based on programming languages. These skills help me build projects much faster and speed up my coding process (like the fast, precise coding style of my duelist) 

### Notice!
Agent skills do not cover how to deploy projects to production servers or set up hosting pipelines. The AGENT.md only contains comprehensive coding language programming styles. It don't cover agent-style for different AI Agent Harness: Codex, Gemini, Claude... 

This is a personal project to help me work faster. I gathered these things from different places on GitHub. This project is built upon the outstanding contributions of the open-source community. I would like to express my sincere gratitude and deep respect to the original creators whose work inspired and shaped these development agentic kit. 

- **Matt Pocock**: https://github.com/mattpocock/skills
- **Addy Osmani**: https://github.com/addyosmani/agent-skills
- **Anthropic (Frontend Design)**: https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md
- **Ertugrul-dmr (Clean Code)**: https://github.com/ertugrul-dmr/clean-code-skills
- **xganets (AGENT.md file)**: https://github.com/srose69/x.AGENTS.md
- **superpowers (Agentic Developer Kit)**: https://github.com/obra/superpowers

## 2. All 23 Sides 

### Interview and Analyze Idea
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| interview-me | Asks questions one by one to find out what you really want to build. | The task details are not clear, or you want to align on the core requirements. |
| idea-refine | Explores different options to turn a rough idea into a clear proposal. | You have a vague concept and want to make it better before writing a plan. |
| spec-driven-development | Write a PRD covering objectives, commands, structure, code style, testing, and boundaries before any code | Starting a new project, feature, or significant change. |

### Delve into Idea
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| grill-me | Asks hard questions about your plan to find mistakes and choose the best path. | You want to double-check your design before writing code. |
| grill-with-docs | Checks your plan against project documents to make sure it follows project rules. | You want to make sure your new idea fits the current project standards or **existing documents**. |

### Planning
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| planning-and-task-breakdown | Breaks a big task down into a step-by-step TODO list. | At the start of a task, so you have a clear plan to follow. |

### Document Writing
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| prd | Writes a document that describes the project goals, features, and requirements. | Before starting a big new feature or project. |
| documentation-and-adrs | Writes down major design choices and rules so the team knows why they were made. | Making a choice that changes how the project is built. |

### Writing Code
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| incremental-implementation | Writes and test code step-by-step instead of doing everything at once. | Making changes that touch multiple folders or files. |
| source-driven-development | Reads official documentation of libraries to write correct, modern code. | Using new libraries, writing configurations, or calling external APIs. |
| test-driven-development | Writes tests first, then writes code to make the tests pass. | Writing algorithms or helpers that must work perfectly. |

### Frontend
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| frontend-ui-engineering | Builds user interfaces that are easy to use, work on mobile, and follow accessibility rules. | Creating or updating web pages and UI components. |
| frontend-design | Applies colors, gradients, dark mode, and animations to make pages look premium. | Styling UI components or layouts to look beautiful. |
| improve | Shadcn skills. It allow AI to know and use **shadcn** components for design | Applying **shadcn components** to the UI, reach the consistency in design. |

### Folder Structure & Clean Code
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| improve-codebase-architecture | Finds messy code or bad folder layouts and shows you how to structure them better. | Folder structure (files) are too connected, hard to test, or confusing to look at. |
| boy-scout | Cleans up small things (like bad names or unused code) in the files you are editing. | Every time you edit code files, and you want AI makes them be clean again. |
| clean-functions | Makes functions short, single-purpose, and limits parameters to 3. | Writing new functions or fixing long, messy code. |
| clean-names | Uses clear and descriptive names for variables, functions, and files. | Naming new items or fixing confusing names in code. |
| clean-comments | Deletes old comments and avoids comments that just repeat what the code does. | Writing comments or cleaning up old code comments, messy comments... |
| clean-general | Checks your code against clean code rules (like no duplicate code and no magic numbers). | You want to check if recently written code is clean and neat, or the folder strucuture is current good or bad? |

### Code Improvements
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| impeccable | Produces high-quality, well-reasoned, and reliable judge and solutions. | tasks, critical code changes, or when breaking problems, especially to judge the UI/UX design. |
| code-simplification | Makes complicated code simple and easy to read without changing what it does. | The code works but is too complex to understand easily. |
| code-review-and-quality | Reviews code changes for bugs, style, and security issues before saving. | Before completing a task or submitting your code. |

### Performance
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| performance-optimization | Finds and fixes slow code, heavy loops, or slow database calls. | The application is slow or API calls take too long. |

### Security
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| security-hardening | Secures code against hacks, validates user input, and keeps credentials safe. | Handling file uploads, database inputs, or user login data. |

### Git & Version Control 
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| git-workflow-and-versioning | Manages clean git commits, branch merges, version tagging, and changelogs. | Making commits, resolving conflicts, or releasing new software versions. |

## 3. Agent Rules 
This project contains ready-to-use agent coding rules inspired by well-known books on software design. I chose some commonly used skills from Github: 
[](https://github.com/ciembor/agent-rules-books). Thank you for your contributions 

Each rule set is released in three versions, based on your usage purpose: 
- `full`: The canonical full reference source 
- `mini`: The recommended version for most real tasks use 
- `nano`: The fallback for very tight context budget 

**14 rule sets:**

| name | usage purpose |
| --- | --- | 


## 4. AGENT.md 
AGENT.md play as a README.md file for coding agent. A dedicated, predictable place to provide context, instructions to help them working on your projects. 

In this project, I aggregate different AGENT.md files, divided by comprehensive code in any languages. 

This is designed to product code that is maintainable, secure and reliable. It is written in the style of Linux kernel coding style: consice, direct, and upcompromising about quality. Every line of code must have a reason. Every error must have messages. Every failure must be loud, immediate, and obvious. 

I have those AGENT.md file **with the help of**: [](https://github.com/srose69/x.AGENTS.md). Thank you for your contribution. 

## 5. Agent CLI plugins 
Help Agent CLI: codex, claude, gemini can detect the skills set in the development kit. 

## 6. How to use 
Clone this project to your personal project 

### Setup Skills 
Most AI code tools (like Codex, Cursor, or Antigravity) can read skills from a folder at your project root. 

Place these skills in your root project like this. 

```text
your-project/
    └── skills/
        ├── interview-me/
        │   └── SKILL.md
        ├── idea-refine/
        │   └── SKILL.md
        ├── boy-scout/
        │   └── SKILL.md
        └── ...
```

### Setup rules 
For other (claude, continues...): Place them in a specific folder like `claude/skills`, `continue/skills`

Place rules in your root project like this: 
```text
your-project/
    └── rules/
        ├── interview-me/
        │   └── SKILL.md
        ├── idea-refine/
        │   └── SKILL.md
        ├── boy-scout/
        │   └── SKILL.md
        └── ...
```
### AGENT.md 
Choose the most appropriate AGENT rule for your project's languages. Place it in the root project, rename it to `AGENT.md`