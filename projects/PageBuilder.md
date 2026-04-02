# PageBuilder Creator | Lightweight Drag & Drop Website Builder

## 🚀 Business Overview

PageBuilder Creator is a modern website builder designed as a lightweight alternative to heavyweight solutions like Elementor. It delivers an intuitive drag-and-drop interface, enabling anyone to create professional websites without deep coding expertise. Built as a commercial client project — production deployment outside personal portfolio.

**Key Value Propositions:**
- **Lightweight & Fast:** Streamlined architecture eliminates the bloat common in traditional page builders.
- **Real-Time Interactivity:** Instant visual feedback with live multi-device responsiveness preview.
- **Platform-Independent:** No lock-in — exports clean HTML, CSS, and React components.

---

## 🏗️ Engineering Challenges & Solutions

### 1. High-Performance Drag & Drop at Scale

**Challenge:** Managing hundreds of draggable elements on a single canvas without performance degradation.

**Solution:** Virtual rendering with viewport culling — only visible elements render to the DOM. A shadow state maintains the complete element hierarchy. Drag operations update lightweight index references instead of re-rendering the entire tree, maintaining 60 FPS performance.

---

### 2. Responsive Design Consistency Across Breakpoints

**Challenge:** Editing responsive properties on one breakpoint breaks the layout on others.

**Solution:** Cascading breakpoint engine where element properties cascade intelligently from parent breakpoints. Breakpoint-specific overrides are isolated, preventing cascading conflicts while maintaining true responsive control.

---

### 3. Canvas-to-Code Synchronisation

**Challenge:** Visual canvas state can diverge from exported HTML/CSS/React code.

**Solution:** Single source of truth using a normalised element schema. Every visual change is represented in the schema first — export is a pure transformation of that schema, not a separate codepath. This ensures exports always match the editor.

---

### 4. File Storage & Asset Management

**Challenge:** Connecting the visual builder to persistent file storage without breaking the lightweight architecture.

**Solution:** Backend integration with database-backed file storage. Files UI built as a first-class feature — upload, browse, and attach assets directly within the builder. Currently extending responsiveness and performance across all breakpoints.

---

## 🛠️ Technology Stack

| Layer | Technologies |
|---|---|
| Frontend | Next.js · TypeScript · TailwindCSS |
| Drag & Drop | Custom performance-optimised implementation |
| Backend | FastAPI · PostgreSQL · file storage |
| Export | HTML · CSS · React components |

---

## 📊 Status

Commercial project — client commission. Production deployment active. Currently extending: file storage in DB, Files UI, responsiveness, performance — connecting all page-building features end-to-end.
