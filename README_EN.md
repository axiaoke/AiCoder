# 🤖 aicoder: AI-Native Multi-Agent Development Framework

> [English] | [简体中文](README.md) | [繁體中文](README_ZH.md) | [日本語](README_JP.md) | [한국어](README_KR.md)

> **Authorized for release by the creator: Sting**

`aicoder` is a set of collaborative development skills specifically designed for modern AI coding tools like **Claude Code**, **Cursor**, and **Windsurf**. By simulating the division of labor in a professional software team, it transforms complex development processes into predictable, high-quality "gate-based" iterations, helping developers (especially beginners) build projects with professional engineering standards from scratch.

---

## 🌟 Core Concept: Multi-Agent Collaboration

`aicoder` is more than just code generation; it establishes a complete **digital team architecture**. Through clear folder isolation and role definitions, the AI can seamlessly switch between roles based on the current context:

- **🛡️ Workflow Guardian (CLAUDE.md)**: Responsible for global routing, Gate checks (Gate 1-4), and iteration wrap-ups. Ensures the project doesn't bypass stages and that knowledge accumulation is centered.
- **🎨 Product Specialist (product.md)**: Based on IDEO Design Thinking. Responsible for scenario reconstruction, requirement divergence and convergence, outputting rigorous PRDs, and establishing visual specifications.
- **📐 Architectural Designer (tech.md)**: A stability-first decision-maker. Responsible for tech stack selection, database modeling, API design, and TRD creation.
- **👨‍💻 Full-Stack Implementer (developer.md)**: A craftsman with independent judgment. Responsible for code implementation, unit self-checks, integration testing, and deviation reviews.

---

## 🚀 Key Features

### 1. The Gate System (Gate-Based Iterations)
Projects are broken down into four distinct gates to ensure every step is documented and verified:
- **Gate 1**: PRD Confirmed (Define "What to build").
- **Gate 2**: TRD Confirmed (Define "How to build").
- **Gate 3**: Development, Integration, and Acceptance Complete (Ensure "Built correctly").
- **Gate 4**: Iteration Wrap-up and Knowledge Accumulation (Ensure "What remains").

### 2. Standardized Documentation System
Every iteration produces standardized deliverables such as `sprint.md` (task list), `decisions.md` (architectural decision records), and `backlog.md` (legacy issues). This perfectly solves the "hallucination" and "forgetting" problems associated with long-running AI sessions.

### 3. Global Standard Integration
Built-in **code quality red lines** for frontend (Vue 3/TS/Vite) and backend (NestJS/MySQL). It automatically applies responsive checklists and security reviews, ensuring output is production-ready.

---

## 🛠️ How to Get Started

1. **Workspace Initialization**:
   - Load this project into your AI coding tool.
   - Say: "Initialize new project [Project Name]".
   - The Workflow Guardian will automatically set up the standard `product/`, `tech/`, and `developer/` directory structure for you.

2. **Role Injection**:
   - The `CLAUDE.md` in each workspace contains corresponding role instructions.
   - When you enter a folder, the AI will automatically load the corresponding mental model.

---

## 📂 Project Structure Sample

```text
aicoder/
├── roles/               # Core role instructions (product, tech, developer, coordinator)
├── templates/           # Document templates and coding standard samples
├── projects/            # Shared knowledge base across sessions (project.md, decisions, etc.)
├── product/             # Product design workspace
├── tech/                # Technical design workspace
└── developer/           # Code implementation and testing workspace
```

---

## 📄 Copyright and Acknowledgments

The entire set of processes, documentation standards, and role positioning in this project was developed by the author **Sting** through extensive practice. It is now released publicly under the author's authorization. Developers are welcome to learn, cite, and jointly improve development standards in the AI era.

---

*"Making AI not just a tool, but your professional team."*
