**Product Requirements Document (PRD)**

**Product Name:** SmartCV Builder
**Author:** Magdy
**Last Updated:** 2025-05-17

## **1. Product Overview**

SmartCV Builder is a web application that allows users to upload their CVs. Users select from pre-defined design templates and then use a chat interface powered by a large language model (LLM) to iteratively edit CV content and customize the design of the chosen template. Users can preview changes in real-time and export the final CV as a polished PDF document. The MVP will focus on a single active CV per user.

---

## **2. Goals**

*   Enable users to generate CVs using structured data extracted from their uploads.
*   Allow users to interact with an LLM via chat to edit CV content and customize the layout/style of selected pre-defined templates.
*   Support uploading existing CVs (PDF, DOCX).
*   Provide a selection of pre-defined CV design templates (HTML/CSS based).
*   Provide real-time preview of edits.
*   Support PDF export of the final CV.
*   Offer a chat-based method for manual CV content entry or correction if automated parsing is insufficient.
*   **For Phase 1 of the MVP, user authentication will use Firebase Anonymous Login.**

---

## **3. User Roles**

*   **User**: Can access the app via anonymous login, upload their CV, select a design template, interact with the chat assistant to edit content and customize the template, preview and export their CV.

---

## **4. Features**

### 4.0. System Feedback and Error Handling

*   If CV parsing fails or is incomplete, the user is notified with clear guidance to re-upload or use the chat interface to manually enter/correct CV content.
*   If LLM response is unhelpful or unclear, the system prompts the user to clarify their request.
*   Prompt assistant responses are always accompanied by a transparency note summarizing the change.

### 4.1. User Management

*   **Phase 1:** Users are authenticated via Firebase Anonymous Login. No email/password signup or login is required.
*   Users will work on a single active CV for the MVP.
*   A new `activeCV` document is automatically created in Firestore when a user session is created, with default template and empty CV structure.

### 4.2. CV Upload

*   Accepts DOCX or PDF files.
*   Extracts structured content into JSON format using tools like `mammoth.js` or `pdf-parse`.

### 4.3. Template Selection and Customization

*   Users can select from a list of 3-5 pre-defined CV design templates (HTML/CSS based).
*   The LLM chat assistant can be used to modify aspects of the selected template (e.g., colors, fonts, basic spacing adjustments).

### 4.4. Chat Assistant

*   Powered by OpenAI GPT API.
*   Functions:
    *   Content edits (based on uploaded CV or chat input).
    *   Layout/style changes (customizing the selected pre-defined template).
    *   Explain what changes were made.
*   Chat interface built using React and shadcn/ui.

### 4.5. Live Preview

*   Real-time HTML rendering of CV using a component-based system, styled according to the selected and customized template.
*   Updates on every significant edit.

### 4.7. Export

*   Convert HTML to PDF using Puppeteer or html2pdf.js.

---

## **5. Technical Architecture**

### 5.1. LLM Prompt Modes

To maintain clarity and flexibility in how the chat assistant functions, the system will support distinct prompt modes:

*   **Content-Edit Mode**: Modify or rewrite CV text, respond to user changes, or structure content provided by the user via chat.
*   **Layout-Design Mode**: Modify visual and structural layout aspects of the *chosen pre-defined template* based on user instructions.
*   *(Deferred for MVP)* **Job-Match Mode**: Analyze CV content against a provided job description and offer improvement suggestions.
*   These modes may be toggled explicitly via the UI or inferred from user queries using intent classification.

### 5.2. Frontend

*   **Framework:** Next.js
*   **Styling:** Tailwind CSS
*   **UI Components:** shadcn/ui, react-pdf
*   **Features:** Chat interface, CV preview, file upload, authentication UI (anonymous login for Phase 1).

### 5.3. Backend

*   **Hosting & Services:** Firebase (Auth [Anonymous for Phase 1], Firestore, Storage, Cloud Functions)
*   **LLM:** OpenAI GPT-4 (with function calling)
*   **CV Parsing:** `mammoth.js`, `pdf-parse`
*   **Preview Rendering:** HTML/CSS rendered in DOM or iframe, based on selected template and `cvJson`.
*   **PDF Export:** Puppeteer or html2pdf.js

---

## **6. MVP Development Roadmap**

### **Phase 1: MVP Core (Weeks 1–5)**

*   **Week 1:**
    *   Set up Firebase project (Auth [Anonymous for Phase 1], Firestore, Storage).
    *   Bootstrap Next.js frontend with Tailwind CSS.
    *   Set up routing and basic UI shell for a single CV editing experience.
*   **Week 2:**
    *   Implement file upload for CV (DOCX, PDF).
    *   Parse CV content into JSON (`mammoth.js`/`pdf-parse`).
    *   Implement user authentication (anonymous login, single active CV per user).
*   **Week 3:**
    *   Implement selection of 3-5 pre-defined HTML/CSS design templates.
    *   Build initial chat interface and integrate OpenAI GPT API.
    *   Create JSON-to-HTML CV rendering engine that applies the selected template.
*   **Week 4:**
    *   Handle chat-driven content edits (including manual input if parsing fails).
    *   Handle basic chat-driven style edits on the selected template (e.g., primary color, font family for body/headings).
    *   Bind live CV rendering to `cvJson` and template customization updates.
*   **Week 5:**
    *   Enable real-time preview updates on each significant chat edit.
    *   Add PDF export functionality.
    *   Polish UI and perform core feature testing.

### **Phase 2: Post-MVP Enhancements (Example Weeks 6–9)**

*   **Weeks 6-7: Multi-CV & Enhanced Customization**
    *   Implement dashboard/system for managing multiple CVs per user.
    *   Expand LLM capabilities for more detailed customization of selected templates (e.g., fine-grained spacing, section reordering if feasible with chosen rendering approach).
    *   Introduce more pre-defined templates.
    *   Refine HTML/CSS rendering for greater accuracy.
    *   Optimize PDF export rendering.
*   **Weeks 8-9: Polish, UX & Initial Job Match Exploration**
    *   Enhance UI with helper tips, onboarding tooltips, and user feedback messages.
    *   Perform usability testing and address friction points.
    *   Final bug fixes and performance optimization.
    *   Begin development of "Job-Match Mode" (as defined in original PRD Phase 4).

---

## **7. Constraints and Considerations**

*   All uploaded files (CVs) and parsed content will be securely stored in Firebase Storage and Firestore, with access rules scoped to the authenticated user for their active CV.
*   Parsing accuracy may vary depending on file clarity and structure; chat-based input is the fallback.
*   The MVP will focus on a single active CV per user.
*   MVP will offer a curated list of pre-defined templates; users will not upload their own design files (e.g., images/PDFs for design inference, or arbitrary HTML files) in this phase.
*   English-only support for MVP.
*   No third-party integrations required for MVP.
*   No local LLM inference; use OpenAI APIs.
*   Must ensure user data privacy and security.
*   **For Phase 1, user sessions are anonymous. Email/password login may be added in a future phase.**

---

## **8. Future Enhancements**

*   Dashboard for managing multiple CVs (first post-MVP enhancement).
*   Job-Match Mode.
*   Ability to upload and analyze custom design templates (e.g., using Vision API, or robust parsing of user-provided HTML/CSS templates).
*   Multi-language CV support.
*   Support collaboration (comments, recruiter access).
*   Integrations (LinkedIn, GitHub).
*   Version control with detailed diffs.
*   Scroll-to-view or highlight edited sections in the preview.

---

## **9. Success Metrics**

*   Time from upload/initial input to first PDF export < 7 minutes for common cases.
*   High completion rate for users attempting to edit and export a CV.
*   Real-time preview latency < 1.5s.
*   High user satisfaction in LLM chat suggestions for content and basic style modifications.

---