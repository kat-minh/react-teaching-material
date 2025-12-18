# REACTJS LESSON PLANS - COMPLETE INDEX

## 📚 Overview
This directory contains **17 detailed, instructor-ready lesson plans** for teaching ReactJS from beginner to production-ready level. Each lesson follows a strict 10-section format designed for instructors with zero teaching experience.

**Target Audience (Students):** Completed MERN backend basics, weak in React/Frontend API handling.  
**Target Audience (Instructors):** Know JavaScript/Web basics, may lack deep React expertise or teaching experience.  
**Total Duration:** 17 sessions × 2 hours = 34 hours (8-9 weeks @ 2 sessions/week)

---

## � Quick Navigation by Chapter

### Chapter 1: Foundation & Setup (Sessions 00-01)
- [Session 00: React Foundation & Mindset](session-00-react-foundation.md) - Why React exists, Mental models
- [Session 01: Vite + TypeScript Setup](session-01-setup-ts.md) - Project scaffolding, Tailwind v4

### Chapter 2: React Core (Sessions 02-05)
- [Session 02: Props & State](session-02-props-state.md) - Component composition, useState
- [Session 03: Router & Layouts](session-03-router-layout.md) - Client-side routing, Nested routes
- [Session 04: Protected Routes](session-04-protected-useEffect.md) - Route guards, useEffect
- [Session 05: Zustand & Auth](session-05-zustand-auth.md) - Global state, Token management

### Chapter 3: UI & Forms (Sessions 06-08)
- [Session 06: Tailwind & shadcn/ui](session-06-ui-practical.md) - Modern UI components
- [Session 07: Axios & Interceptors](session-07-axios-interceptors.md) - API client, Refresh token
- [Session 08: Forms (RHF + Zod)](session-08-forms-rhf.md) - Validation, Error mapping

### Chapter 4: Data Management (Sessions 09-10)
- [Session 09: TanStack Query (Fetch)](session-09-react-query.md) - Server state, Caching
- [Session 10: Mutations & Auth Flow](session-10-mutations.md) - useMutation, Invalidation

### Chapter 5: Project Sprint (Sessions 11-13)
- [Session 11: Project Setup](session-11-project-setup.md) - Architecture, Login/Register
- [Session 12: Profile Flows](session-12-profile-flows.md) - CRUD operations
- [Session 13: Media & Refresh Token](session-13-media-refresh.md) - File upload, Token refresh

### Chapter 6: Polish & Deploy (Sessions 14-16)
- [Session 14: UI Polish & Debug](session-14-polish-debug.md) - DevTools, Breakpoints
- [Session 15: Integration Testing](session-15-integration-testing.md) - E2E checklist, Code audit
- [Session 16: Deployment](session-16-deployment.md) - Vercel, Production config

---

## �🗺️ Curriculum Structure

### **PHASE 0: FOUNDATION (Session 00)**
Mental models and "why React" before diving into code.

| Session | Title | Key Topics | Duration |
|:--------|:------|:-----------|:---------|
| [00](session-00-react-foundation.md) | **React Foundation & Mindset** | Why React exists, Mental models, Component=Function, JSX, Hooks concept | 90-120min |

---

### **PHASE 1: FUNDAMENTALS (Sessions 01-05)**
Building blocks of modern React development.

| Session | Title | Key Topics | Duration |
|:--------|:------|:-----------|:---------|
| [01](session-01-setup-ts.md) | **Vite + TypeScript Setup** | Project scaffolding, Absolute imports, Tailwind v4 | 2h |
| [02](session-02-props-state.md) | **Props & State Fundamentals** | Component composition, useState, Conditional rendering | 2h |
| [03](session-03-router-layout.md) | **React Router & Layouts** | Client-side routing, Nested routes, useEffect rules | 2h |
| [04](session-04-protected-useEffect.md) | **Protected Routes & useEffect** | Route guards, Auth checking, useEffect cleanup | 2h |
| [05](session-05-zustand-auth.md) | **Zustand & Auth Store** | Global state, Persistence, Token management | 2h |

---

### **PHASE 2: PRODUCTION PATTERNS (Sessions 06-10)**
Professional-grade tooling and patterns.

| Session | Title | Key Topics | Duration |
|:--------|:------|:-----------|:---------|
| [06](session-06-ui-practical.md) | **Tailwind v4 & shadcn/ui** | Modern UI components, Toast notifications, Dialogs | 2h |
| [07](session-07-axios-interceptors.md) | **Axios Layer & Interceptors** | Centralized API client, Request/Response interceptors, Refresh token flow | 2h |
| [08](session-08-forms-rhf.md) | **Forms (RHF + Zod)** | React Hook Form, Schema validation, Error mapping | 2h |
| [09](session-09-react-query.md) | **TanStack Query (Fetch)** | Server state management, Caching, Loading states | 2h |
| [10](session-10-mutations.md) | **Mutations & Auth Flow** | useMutation, Invalidation, Complete auth lifecycle | 2h |

---

### **PHASE 3: PROJECT SPRINT (Sessions 11-16)**
Building a production-ready Shopping Cart application.

| Session | Title | Key Topics | Duration |
|:--------|:------|:-----------|:---------|
| [11](session-11-project-setup.md) | **Project Setup & Auth Core** | Architecture, Folder structure, Login/Register implementation | 2h |
| [12](session-12-profile-flows.md) | **Profile Flows (CRUD)** | Get/Update/Delete user data, Query invalidation | 2h |
| [13](session-13-media-refresh.md) | **Media Upload & Refresh Token** | FormData, File previews, Silent token refresh, Loop prevention | 2h |
| [14](session-14-polish-debug.md) | **UI Polish & Debug Workshop** | Skeletons, Error states, DevTools mastery, Breakpoints | 2h |
| [15](session-15-integration-testing.md) | **Integration Testing & Audit** | Manual E2E checklist, Code quality rubric, Peer review | 2h |
| [16](session-16-deployment.md) | **Deployment & Graduation** | Vercel deployment, Environment variables, SPA routing, CV tips | 2h |

---

## 🎯 Learning Outcomes

By the end of this curriculum, students will be able to:

✅ **Build** a full-stack React application with TypeScript  
✅ **Implement** robust authentication with auto-refresh tokens  
✅ **Manage** server state (React Query) and client state (Zustand) effectively  
✅ **Handle** forms with validation (RHF + Zod)  
✅ **Debug** production issues using Chrome DevTools  
✅ **Deploy** to production (Vercel) with proper environment configuration  
✅ **Write** clean, maintainable code following industry best practices  

---

## 📖 Lesson Plan Format

Each lesson plan follows this 10-section structure:

1. **Session Overview** - Goals, duration, outcomes
2. **Instructor Opening Script** - Exact words to say, motivation
3. **Mental Model & Conceptual Foundation** - Analogies, "Why this exists"
4. **Live Coding – Step by Step** - Detailed code with inline comments
5. **Common Student Mistakes & Debugging** - Symptoms, causes, fixes
6. **Instructor Questions & Expected Answers** - Socratic method prompts
7. **In-Class Mini Task** - Hands-on practice
8. **Homework / Extension Task** - Reinforcement activities
9. **Checkpoint & Evaluation** - Observable success signals
10. **Teaching Notes** - Time management, red flags, emphasis points

---

## 🛠️ Tech Stack

- **Core:** React 19, TypeScript, Vite
- **Routing:** React Router v6
- **State Management:** 
  - Zustand (Client/Global state)
  - TanStack Query (Server state)
- **Forms:** React Hook Form + Zod
- **UI:** Tailwind CSS v4 + shadcn/ui
- **HTTP:** Axios (Interceptor pattern)
- **Notifications:** Sonner (Toast)

---

## 🎓 Pedagogical Principles

### Pain-Driven Development
Every concept is introduced by first showing the "pain" (manual approach) before the "cure" (modern solution).

### Guardrails
Students are given strict rules (e.g., "Never fetch in components", "Always use env vars") to prevent common pitfalls.

### Production Rule of Thumb
Code samples follow production-grade patterns, not "tutorial shortcuts".

### Checkpoints
Each session has mid-session and end-session checkpoints to ensure no student is left behind.

---

## 📋 Quick Reference

### Session Dependencies
- **Session 00:** Standalone (mental models)
- **Sessions 01-05:** Sequential (each builds on previous)
- **Sessions 06-10:** Can be taught in flexible order (but 07→09→10 recommended)
- **Sessions 11-16:** Strictly sequential (project-based)

### Difficulty Progression
- **Foundation:** Session 00 (concepts only)
- **Beginner:** Sessions 01-03
- **Intermediate:** Sessions 04-07
- **Advanced:** Sessions 08-13
- **Professional:** Sessions 14-16

### Critical Sessions (Must Not Skip)
- Session 00 (Foundation) - Mental models prevent confusion later
- Session 07 (Axios Interceptors) - Foundation for all API work
- Session 09 (React Query) - Modern data fetching paradigm
- Session 10 (Mutations) - Completes the data flow cycle
- Session 13 (Refresh Token) - Production-critical security

---

## 🚀 Getting Started (For Instructors)

1. **Start with Session 00** - Build mental models before code.
2. **Read Session 01** to understand the format and tone.
3. **Review the Tech Stack** - Ensure you can run a basic Vite + React + TS project.
4. **Prepare Backend** - Students need a live API. Use the provided backend or deploy your own.
5. **Follow the Script** - Lesson plans are designed to be read verbatim if needed.
6. **Adapt as Needed** - Adjust timing based on class pace, but maintain core concepts.

---

## 📞 Support & Feedback

For questions, clarifications, or suggestions:
- Review the **Teaching Notes** section in each lesson plan
- Check **Common Student Mistakes** for troubleshooting
- Refer to **Anti-Patterns** sections for what NOT to teach

---

## 📄 License & Usage

These lesson plans are designed for educational use. Instructors are free to:
- Use verbatim in classroom settings
- Adapt timing and examples to local context
- Translate to other languages

Please maintain attribution and do not remove pedagogical scaffolding (checkpoints, anti-patterns, etc.) as they are critical for student success.

---

**Last Updated:** 2025-12-18  
**Version:** 1.1 (Complete Curriculum + Session 0)  
**Total Lesson Plans:** 17
