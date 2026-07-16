# CLOUDIAN SKILLS 
## 1. Overview 

This project contains all the AI Agent skills I use often. These skills help me build projects much faster and speed up my coding process (like the fast, precise coding style of my duelist) 

### Notice!
These skills do not cover how to deploy projects to production servers or set up hosting pipelines. 

This is a personal project to help me work faster. I gathered these skills from different places on GitHub. This project is built upon the outstanding contributions of the open-source community. I would like to express my sincere gratitude and deep respect to the original creators whose work inspired and shaped these skills:

- **Matt Pocock**: https://github.com/mattpocock/skills
- **Addy Osmani**: https://github.com/addyosmani/agent-skills
- **Anthropic (Frontend Design)**: https://github.com/anthropics/skills/blob/main/skills/frontend-design/SKILL.md
- **Ertugrul-dmr (Clean Code)**: https://github.com/ertugrul-dmr/clean-code-skills
## 2. All 23 Sides 

### Interview and Analyze Idea
| Skill | What It Does | Use When |
| :--- | :--- | :--- |
| interview-me | Asks questions one by one to find out what you really want to build. | The task details are not clear, or you want to align on the core requirements. |
| idea-refine | Explores different options to turn a rough idea into a clear proposal. | You have a vague concept and want to make it better before writing a plan. |

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

## 3. How to use 
Most AI code tools (like Codex, Cursor, or Antigravity) can read skills from a folder at your project root. 

Clone this repository and place these skills in your project like this:

```text
your-project/
└── .agents/
    └── skills/
        ├── interview-me/
        │   └── SKILL.md
        ├── idea-refine/
        │   └── SKILL.md
        ├── boy-scout/
        │   └── SKILL.md
        └── ...
```

For other (claude, continues...): Place them in a specific folder like `claude/skills`, `continue/skills`
## 4. Why Agent SKILLS? 

AI Agents can write code very fast, but they can still make mistakes, use old styles, or write messy code. Giving the AI a giant list of rules in one prompt does not work well because it takes up too much memory.
Using structured AI skills is better because:
*   **It saves space:** The AI only reads the specific skill file it needs for the current task (for example, it only reads the security rules when handling user inputs).
*   **It keeps code consistent:** It forces the AI to follow the exact same clean code and folder structures across the entire project.
*   **It makes code clean:** It stops the AI from writing messy code by enforcing short functions, clear names, and helpful comments.
*   **It reduces bugs:** It forces the AI to write tests and check for security issues before saving the work.
*   **Competing at zero cost ($0):** Using these skills wisely helps you level up your AI's capabilities. It gives you the power to build great software and compete with larger projects at absolutely zero cost.