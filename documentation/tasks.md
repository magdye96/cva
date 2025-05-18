# SmartCV Builder MVP – Step-by-Step To-Do List

## Phase 1: Project Setup & Core Infrastructure

- [ ] **Initialize Repository & Tooling**
  - [ ] Set up Git repository and .gitignore.
  - [ ] Configure Prettier, ESLint, and basic commit hooks.
- [ ] **Set Up Firebase Project**
  - [ ] Create Firebase project.
  - [ ] Enable Auth (Anonymous for Phase 1), Firestore, and Storage.
  - [ ] Configure security rules for single-CV-per-user model.
- [ ] **Bootstrap Next.js Frontend**
  - [ ] Create Next.js app with TypeScript.
  - [ ] Integrate Tailwind CSS and shadcn/ui.
  - [ ] Set up base folder structure (pages, components, hooks, utils).
- [ ] **Configure Environment Variables**
  - [ ] Add .env.example for required keys (Firebase, OpenAI, etc.).
  - [ ] Ensure .env is gitignored.
- [ ] **Set Up Firebase Cloud Functions**
  - [ ] Initialize functions project (Node.js 18+).
  - [ ] Configure deployment scripts.

---

## Phase 2: Authentication & User Management

- [ ] **Implement Anonymous Auth UI**
  - [ ] On app load, sign in user with Firebase Anonymous Login.
  - [ ] Show minimal onboarding or welcome screen (no email/password form).
  - [ ] Add error handling and feedback for auth failures.
- [ ] **Create User Profile & activeCV Document**
  - [ ] On anonymous session creation, trigger Cloud Function to create Firestore user profile and default activeCV document.
  - [ ] Ensure default template and empty cvJson are set.
- [ ] **Implement Auth State Handling in Frontend**
  - [ ] Redirect unauthenticated users to onboarding/welcome screen.
  - [ ] Redirect new users to CV upload if no activeCV exists.

---

## Phase 3: CV Upload & Parsing

- [ ] **Build CV Upload Page**
  - [ ] Create FileUploadDropzone component (shadcn/ui Card, Button, Input).
  - [ ] Add file type/size validation (DOCX, PDF, max 5MB).
- [ ] **Implement File Upload to Firebase Storage**
  - [ ] Store uploaded file at cv_uploads/{userId}/original.{docx|pdf}.
- [ ] **Create CV Parsing Cloud Function**
  - [ ] Use mammoth.js for DOCX, pdf-parse for PDF.
  - [ ] Extract structured content into cvJson (or rawText if parsing fails).
  - [ ] Store cvJson in activeCV document.
- [ ] **Handle Parsing Feedback**
  - [ ] Show progress/loading spinner during upload/parsing.
  - [ ] Display error/info alerts for parsing failures or unsupported files.
  - [ ] Allow fallback to chat-based manual entry if parsing fails.

---

## Phase 4: Template Selection & Customization

- [ ] **Design Pre-defined Templates (3–5 HTML/CSS)**
  - [ ] Create base HTML/CSS for each template (e.g., modern_compact, classic_detailed, creative_minimalist).
  - [ ] Store template metadata (id, name, thumbnail, default customizations).
- [ ] **Build Template Selection UI**
  - [ ] Implement TemplateCard component (shadcn/ui Card, Button, AspectRatio).
  - [ ] Allow user to select template; highlight selected.
- [ ] **Implement Template Selection API**
  - [ ] Create /api/cv/select-template endpoint.
  - [ ] Update selectedTemplateId and reset userCustomizations in activeCV.
- [ ] **Sync Template Selection with Preview**
  - [ ] Update CVPreviewPanel to re-render on template change.
  - [ ] Show notification toast on template selection.

---

## Phase 5: Main CV Editor & Live Preview

- [ ] **Build Main Editor Page Layout**
  - [ ] Implement multi-panel layout (controls, preview, chat) using shadcn/ui ResizablePanel.
  - [ ] Ensure responsive design for tablet/mobile.
- [ ] **Develop CVPreviewPanel**
  - [ ] Render cvJson using selected template and userCustomizations.
  - [ ] Show placeholder if cvJson is empty.
- [ ] **Implement Real-time Preview Updates**
  - [ ] Re-render preview on any cvJson or customization change.
  - [ ] Ensure updates are performant and visually accurate.
- [ ] **Add System Feedback Components**
  - [ ] Integrate NotificationToast, LoadingSpinner, and error modals.

---

## Phase 6: Chat Assistant Integration

- [ ] **Build ChatInterface Component**
  - [ ] Display chat history, user/AI avatars, input box, send button, thinking indicator.
- [ ] **Implement Chat API Endpoint (/api/chat)**
  - [ ] Route user prompts to LLM Coordinator Cloud Function.
  - [ ] Support Content-Edit and Layout-Design modes.
- [ ] **Integrate OpenAI GPT-4 API**
  - [ ] Enable function calling for structured cvJson/userCustomizations edits.
  - [ ] Return LLM explanations (transparency notes) with each response.
- [ ] **Sync Chat with Firestore**
  - [ ] Store chat history in activeCV.
  - [ ] Update cvJson/userCustomizations based on LLM output.
- [ ] **Handle Chat-Driven Edits**
  - [ ] Apply content edits to cvJson.
  - [ ] Apply style/layout edits to userCustomizations.
  - [ ] Show system feedback for each change.
- [ ] **Support Manual CV Entry via Chat**
  - [ ] If parsing fails, guide user to build CV from scratch using chat.

---

## Phase 7: Actions & Export

- [ ] **Implement Undo/Redo Actions**
  - [ ] Add undo/redo buttons to Actions panel.
  - [ ] Track and update cvJson/userCustomizations history in Firestore.
  - [ ] Show notification on action.
- [ ] **Build PDF Export API Endpoint (/api/cv/export)**
  - [ ] Render HTML using selected template and customizations.
  - [ ] Generate PDF with Puppeteer or html2pdf.js.
  - [ ] Return signed download URL.
- [ ] **Integrate Export to PDF in UI**
  - [ ] Add export button to Actions panel.
  - [ ] Show loading state and notification on completion/error.

---

## Phase 8: Polish, Testing & Accessibility

- [ ] **UI Polish & Responsive Testing**
  - [ ] Refine spacing, colors, and typography for all breakpoints.
  - [ ] Ensure consistent use of shadcn/ui components.
- [ ] **Accessibility Improvements**
  - [ ] Ensure keyboard navigation, ARIA attributes, and color contrast.
- [ ] **Error Handling & User Feedback**
  - [ ] Add global error modals and inline alerts for all failure cases.
- [ ] **Core Feature Testing**
  - [ ] Manual and automated tests for upload, parsing, chat, preview, export.
  - [ ] Fix bugs and edge cases.

---

## Phase 9: Deployment & Monitoring

- [ ] **Prepare for Production Deployment**
  - [ ] Set up Firebase Hosting for frontend.
  - [ ] Deploy Cloud Functions and Firestore rules.
  - [ ] Configure environment variables for prod.
- [ ] **Monitor & Log Errors**
  - [ ] Enable Cloud Logging for backend.
  - [ ] Integrate frontend error tracking (e.g., Sentry, optional).
- [ ] **Final Review & Launch**
  - [ ] Review all requirements and polish.
  - [ ] Launch MVP to production.

---

**Note:** Email/password authentication may be added in a later phase. 