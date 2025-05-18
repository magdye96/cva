Okay, this is an excellent technical foundation! As an expert UI/UX designer, I'll create page-by-page design briefs and text-based UI wireframes for the SmartCV Builder MVP. We'll heavily lean on **shadcn/ui** components built on **Tailwind CSS**, as specified, to ensure a modern, accessible, and developer-friendly implementation.

## Design System & Reusable Components

Before diving into pages, let's define some core principles and reusable components beyond the base shadcn/ui elements.

**Design System Philosophy:**

*   **Simplicity & Focus:** MVP means clear, uncluttered interfaces. Guide the user through one primary task at a time.
*   **Feedback & Transparency:** Always let the user know what the system is doing (loading, saving, AI processing) and what the AI changed.
*   **Consistency:** Use shadcn/ui components consistently for predictable interactions.
*   **Accessibility (a11y):** Leverage shadcn/ui's built-in accessibility. Use semantic HTML. Ensure good color contrast.

**Key Reusable Custom Components (built using shadcn/ui primitives):**

1.  **`AppHeader`**:
    *   **Contains:** Logo, User Avatar (with dropdown for "Logout").
    *   **shadcn/ui:** `Avatar`, `DropdownMenu`.
2.  **`FileUploadDropzone`**:
    *   **Contains:** Area for drag-and-drop, file input button, text instructions (e.g., "Drag & Drop DOCX or PDF here, or click to select").
    *   **shadcn/ui:** `Card`, `Button`, `Input type="file"` (styled).
3.  **`CVPreviewPanel`**:
    *   **Contains:** Iframe or div rendering the HTML of the CV. Placeholder text if no CV is loaded/parsed yet.
    *   **shadcn/ui:** `AspectRatio` (optional, to maintain CV shape), `ScrollArea`.
4.  **`ChatInterface`**:
    *   **Contains:** Message display area (scrollable), text input for user prompt, send button, AI "thinking" indicator.
    *   **shadcn/ui:** `ScrollArea`, `Textarea`, `Button`, `Avatar` (for user/AI messages), `Skeleton` (for thinking indicator).
5.  **`TemplateCard`**:
    *   **Contains:** Small visual thumbnail/representation of a template, template name, select button.
    *   **shadcn/ui:** `Card`, `Button`, `AspectRatio` (for thumbnail).
6.  **`SectionCard`**:
    *   A styled `Card` component to group related controls or information.
    *   **shadcn/ui:** `Card`, `CardHeader`, `CardTitle`, `CardDescription`, `CardContent`, `CardFooter`.
7.  **`LoadingSpinner / FullScreenLoader`**:
    *   For indicating background processes (e.g., parsing, PDF generation).
    *   **shadcn/ui:** `Loader2` icon (animated).
8.  **`NotificationToast`**:
    *   For non-blocking feedback (e.g., "CV Saved", "Action Undone").
    *   **shadcn/ui:** `Toast`.

---

## Page-by-Page Design Briefs & UI Wireframes

---

### 1. Landing / Onboarding Page (Anonymous Auth)

**Objective:** Allow users to start using the app immediately via Firebase Anonymous Login. Briefly introduce the product. No email/password form is shown for Phase 1.

**Key Elements:**

*   Product Name/Logo (`SmartCV Builder`)
*   Brief tagline (e.g., "Build your perfect CV with AI assistance.")
*   [Button: Get Started] (triggers Firebase Anonymous Login)
*   (Optional) Short note: "No account or signup required for MVP."

**UI Wireframe (Text-based):**

```
[AppHeader - Minimal: Logo only or Logo + "SmartCV Builder"]

---------------------------------------------------------------------
|                                                                   |
|                     SMARTCV BUILDER                               |
|         Build your perfect CV with AI assistance.                 |
|                                                                   |
|   +-----------------------------------------------------------+   |
|   |                                                           |   |
|   |   [Button: Get Started]                                   |   | (shadcn/ui: Button)
|   |   (No signup or login required for MVP)                   |   |
|   |   Your data is saved for this session.                    |   |
|   +-----------------------------------------------------------+   |
|                                                                   |
---------------------------------------------------------------------

[Footer: © 2025 SmartCV Builder, Privacy Policy, Terms of Service]
```

**User Flow:**

1.  User lands on the page.
2.  Clicks "Get Started" (triggers Firebase Anonymous Login).
3.  On success, user is redirected to CV Upload Page (if no `activeCV`) or Main CV Editor Page (if `activeCV` exists).
4.  On error, display error message (e.g., using shadcn/ui `Alert` or inline text).

**Responsive:** Form elements stack vertically on smaller screens.

**Accessibility:** Ensure proper labeling for button, focus management.

---

### 2. CV Upload Page (Initial State / No Active CV)

**Objective:** Guide the user to upload their existing CV to start the process. This page is shown after login if no `activeCV` is found for the user.

**Key Elements:**

*   Clear instruction to upload.
*   `FileUploadDropzone`.
*   Information about supported file types (DOCX, PDF).
*   (Optional) Link to "Start from scratch with chat" (if initial parsing is insufficient is a primary path).

**UI Wireframe (Text-based):**

```
[AppHeader - Logo, User Avatar (Dropdown: Logout)]

---------------------------------------------------------------------
|                                                                   |
|   [SectionCard: Upload Your CV]                                   |
|   | Title: Get Started with Your CV                             |
|   | Description: Upload your existing CV in DOCX or PDF format. |
|   |              We'll parse it to get you started quickly.     |
|   |                                                             |
|   |   +-----------------------------------------------------+   |
|   |   | [FileUploadDropzone]                                |   |
|   |   |   Icon: Upload Cloud                                |   |
|   |   |   Text: Drag & drop your DOCX or PDF file here,     |   |
|   |   |         or [Button: Click to select file]           |   |
|   |   |   Supported formats: DOCX, PDF. Max 5MB.            |   |
|   |   +-----------------------------------------------------+   |
|   |                                                             |
|   |   (Conditional: Displayed during/after upload attempt)      |
|   |   [Progress Bar: Uploading... 75%] (shadcn/ui: Progress)  |
|   |   [Alert: Error - Invalid file type. Please upload DOCX/PDF] | (shadcn/ui: Alert)
|   |   [Alert: Info - Parsing failed. Try re-upload or use chat to build.] |
|   |                                                             |
|   |   (Optional - if offering a path to bypass upload for manual chat entry)
|   |   Or, [Button: Start by describing your CV content via Chat ->] |
|   +-----------------------------------------------------------+   |
|                                                                   |
---------------------------------------------------------------------
```

**User Flow:**

1.  User drags a file or clicks to select.
2.  Frontend validates file type and size.
3.  File is uploaded. Progress is shown.
4.  Backend parses. `LoadingSpinner` might overlay the dropzone or be shown globally.
5.  **On Success:**
    *   `cvJson` is received.
    *   User is redirected to the Main CV Editor Page.
    *   A `NotificationToast` confirms "CV uploaded and parsed successfully!"
6.  **On Parsing Failure/Partial Success:**
    *   User sees an `Alert` message (e.g., "We had trouble parsing all details. You can correct this using the chat assistant.").
    *   `rawText` might be populated in `cvJson`.
    *   User is redirected to the Main CV Editor Page to use chat for corrections.
7.  **On Upload Error (e.g., wrong type):**
    *   An `Alert` message is shown. User stays on the upload page.

**Responsive:** `FileUploadDropzone` takes full width.

**Accessibility:** Ensure keyboard accessibility for file input.

---

### 3. Main CV Editor Page

**Objective:** The core workspace where users select templates, chat with the AI to edit content and customize the design, see a live preview, and export to PDF.

**Layout:** A multi-panel layout is ideal.
*   **Left Panel (Collapsible on Mobile):** Controls (Template Selection, Action Buttons).
*   **Center Panel:** `CVPreviewPanel`.
*   **Right Panel (Collapsible/Drawer on Mobile):** `ChatInterface`.

**UI Wireframe (Text-based - Desktop View):**

```
[AppHeader - Logo, User Avatar (Dropdown: Logout)]

----------------------------------------------------------------------------------------------------------------------
| [Left Control Panel (shadcn/ui: ResizablePanel)] | [Center Preview Panel (ResizablePanel)] | [Right Chat Panel (ResizablePanel)] |
|--------------------------------------------------|-----------------------------------------|-------------------------------------|
| [SectionCard: Design Template]                   | [CVPreviewPanel]                        | [ChatInterface]                     |
| | Title: Select Template                       | |                                         | | [ScrollArea: Message History]     |
| | [TemplateCard: "Modern Compact"] Thumbnail    | |   [Renders HTML of current CV state]    | |   [AI Avatar] Initial greeting    |
| | [TemplateCard: "Classic Detailed"] Thumbnail  | |                                         | |   [User Avatar] My prompt...      |
| | [TemplateCard: "Creative Minimalist"] Thumbnail| |   Scrollable if content is long.        | |   [AI Avatar] Okay, I've made...  |
| | (Selected template is highlighted)             | |                                         | |     Transparency note: "Changed  |
|                                                  | |   (Placeholder if cvJson is empty:    | |      X to Y in experience."       |
| [SectionCard: Customization Mode]                | |    "Your CV preview will appear here.   | |   [Skeleton: AI is thinking...] |
| | Title: AI Assistant Mode                     | |    Start by uploading or describing    | |                                     |
| | [Select: Mode]                                 | |    your CV content via chat.")         | | [Textarea: Your message to AI...] |
| |   [Option: Content Edit (Default)]             | |                                         | | [Button: Send] [Button: Mic?]   |
| |   [Option: Layout & Design]                    | |                                         | |                                     |
|                                                  | |                                         | |                                     |
| [SectionCard: Actions]                           | |                                         | |                                     |
| | Title: Document Actions                      | |                                         | |                                     |
| | [Button: Undo Last Action] (Disabled if none)  | |                                         | |                                     |
| | [Button: Redo Last Action] (Disabled if none)  | |                                         | |                                     |
| | [Button: Export to PDF]                        | |                                         | |                                     |
| |                                                | |                                         | |                                     |
| (Small print: "All changes saved automatically") | |                                         | |                                     |
----------------------------------------------------------------------------------------------------------------------
```

**Key Interactions & States:**

1.  **Initial Load:**
    *   If `activeCV.cvJson` is populated, `CVPreviewPanel` renders it using `selectedTemplateId` and `userCustomizations`.
    *   If `activeCV.cvJson` is empty (e.g., user chose "start with chat" or parsing failed significantly), `CVPreviewPanel` shows a placeholder. Chat prompts user to describe their CV.
2.  **Template Selection:**
    *   User clicks a `TemplateCard`.
    *   `selectedTemplateId` updates.
    *   `userCustomizations` resets to defaults for that template.
    *   `CVPreviewPanel` re-renders.
    *   A `NotificationToast` confirms "Template '[Template Name]' selected."
3.  **Chat Interaction:**
    *   User selects "Content Edit" or "Layout & Design" mode.
    *   User types prompt in `ChatInterface` and sends.
    *   AI "thinking" indicator appears.
    *   Backend processes, updates `cvJson` or `userCustomizations` in Firestore.
    *   Frontend receives updates.
    *   `CVPreviewPanel` re-renders to show changes.
    *   AI response, including the `llmExplanation` (transparency note), appears in chat.
    *   (PRD: "Scroll-to-view or highlight edited sections" in preview is a great UX touch if feasible).
4.  **Undo/Redo:**
    *   User clicks "Undo" or "Redo".
    *   API call is made.
    *   `cvJson` and `userCustomizations` are updated from server.
    *   `CVPreviewPanel` re-renders.
    *   `NotificationToast`: "Action undone/redone."
5.  **Export to PDF:**
    *   User clicks "Export to PDF".
    *   `FullScreenLoader` or button loading state appears.
    *   Backend generates PDF.
    *   Download is initiated (via signed URL).
    *   `NotificationToast`: "PDF generated. Download starting..."
    *   On error: `NotificationToast` or `Alert`: "PDF Generation Failed."
6.  **System Feedback:**
    *   `LoadingSpinner` for ongoing operations (LLM response, PDF generation).
    *   `NotificationToast` for confirmations.
    *   Clear error messages within chat or as `Alerts` if an operation fails.

**Responsive Considerations:**

*   **Tablet:** Panels might stack, or chat becomes a slide-over panel.
*   **Mobile:**
    *   Control Panel (Templates, Actions) could be in a hamburger menu or a bottom navigation bar.
    *   `CVPreviewPanel` takes most of the screen.
    *   `ChatInterface` could be a `Sheet` (bottom drawer, via shadcn/ui) that slides up.
    *   Mode switcher might be icons or a compact select in the chat input area.

**Accessibility:**

*   Ensure all interactive elements are keyboard navigable.
*   ARIA attributes for dynamic regions (preview, chat).
*   Sufficient contrast for text and UI elements, especially with template customizations.

---

### 4. Global Elements / Modals

**Objective:** Handle global actions or information display.

1.  **Logout Confirmation (Modal):**
    *   Triggered from `AppHeader` User Avatar dropdown.
    *   **shadcn/ui:** `AlertDialog`
    *   **Content:** "Are you sure you want to log out?" [Button: Cancel] [Button: Log Out (destructive variant)]
2.  **Error/Info Modals (General Purpose):**
    *   For critical errors or important info not suitable for a toast.
    *   **shadcn/ui:** `AlertDialog` or `Dialog`
    *   **Content:** [Icon: Error/Info], Title, Descriptive Message, [Button: OK/Close]

---

This set of design briefs provides a solid starting point for building the UI/UX of SmartCV Builder's MVP. The emphasis on **shadcn/ui** will accelerate development while maintaining a high-quality, modern feel. Remember to iterate based on user feedback once the MVP is testable!

**Note:** Email/password authentication may be added in a later phase. For Phase 1, all users are anonymous and no signup/login form is shown.