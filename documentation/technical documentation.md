Okay, here is the revised Technical Documentation, aligned with the simplified MVP scope we've defined for SmartCV Builder.

**SmartCV Builder – Technical Documentation (MVP Revision)**

## 1. Overview

SmartCV Builder is a web application that allows users to upload their CV, select a pre-defined design template, and use a chat interface powered by OpenAI models to iteratively edit CV content and customize the chosen template. The system supports real-time preview and PDF export. The MVP focuses on a single active CV per user.

**For Phase 1 of the MVP, authentication is handled via Firebase Anonymous Login. No email/password signup or login is required. User management is based on anonymous sessions. Email/password authentication may be added in a later phase.**

---

## 2. System Architecture

```
┌──────────────┐      ┌────────────┐      ┌──────────────┐
│   Frontend   │ <--> │  API Layer │ <--> │  Firebase    │
│ (Next.js)    │      │(Cloud Fn.) │      │  Services    │
└──────────────┘      └────────────┘      └──────────────┘
       │                   │                   │
       ▼                   ▼                   ▼
┌────────────┐       ┌──────────┐        ┌──────────┐
│ OpenAI GPT │       │ Firestore│        │ Storage  │
│            │       │ (JSON)   │        │ (Files)  │
└────────────┘       └──────────┘        └──────────┘
                           │
                           ▼
                  PDF Generation
               (Puppeteer / html2pdf.js)
```
*(Note: OpenAI Vision API removed from this MVP diagram)*

---

## 3. Frontend (Next.js)

*   **Framework**: React via Next.js
*   **Styling**: Tailwind CSS
*   **UI Components**: shadcn/ui, react-pdf
*   **Modules**:
    *   Auth: **Firebase Anonymous Login** (Phase 1)
    *   Upload: File picker + Firebase Storage for original CV.
    *   Template Selection: UI to choose from 3-5 pre-defined system templates.
    *   Chat: Direct HTTP communication with LLM Coordinator for immediate updates.
    *   Preview: Live HTML CV renderer, applying selected template and user customizations.
    *   *(Deferred for MVP)* Job-Match: Specialized chat mode.

---

## 4. Backend (Firebase Cloud Functions)

*   **Runtime**: Node.js 18 (or latest stable)
*   **Functions**:
    1.  **CV Parsing**
        *   Handles DOCX (`mammoth.js`), PDF (`pdf-parse`) uploads.
        *   Extracts structured `cvJson`. If parsing is incomplete, this function might simply store the raw text or a partially parsed structure, relying on the LLM to help the user complete it via chat.
    2.  **LLM Coordinator**
        *   Manages chat interactions for content edits and template customizations.
        *   Calls OpenAI GPT-4 (with function calling) based on user prompts and selected mode (Content-Edit, Layout-Design for pre-defined templates).
        *   Updates Firestore with the latest `cvJson` and `userCustomizations` for the selected template.
        *   Facilitates chat-based CV content input if initial parsing is insufficient.
    3.  **PDF Generation**
        *   Takes `cvJson` and the selected base template identifier + `userCustomizations`.
        *   Server-side renders the HTML CV.
        *   Uses Puppeteer or html2pdf.js to export to PDF.
        *   Returns a signed URL for download or directly streams the PDF.

---

## 5. Data Storage (Firestore & Firebase Storage)

### Firestore Schema (MVP - Single CV per User)

```
users/{userId}:
  profile: { createdAt } // For anonymous users, no email is stored
  activeCV: { // Represents the single CV the user is working on
    cvJson: object        // The structured content of the CV
    selectedTemplateId: string // Identifier for the chosen pre-defined template
    userCustomizations: object // User-specific overrides for the selected template (e.g., colors, fonts)
    chatHistory: array    // History of chat interactions for this CV
  }
```

*Note: For anonymous users, the profile only contains metadata such as creation date. If/when email/password auth is added, the schema can be extended.*

### Initial State and Creation

The `activeCV` document is created automatically when a new anonymous user session is created, with the following default state:

```json
{
  "cvJson": {
    "header": {
      "fullName": "",
      "email": "",
      "phone": "",
      "summary": ""
    },
    "experience": [],
    "education": [],
    "skills": [],
    "languages": [],
    "projects": []
  },
  "selectedTemplateId": "modern_compact", // Default template
  "userCustomizations": {
    "colors": {
      "primary": "#007ACC",
      "text": "#333333"
    },
    "fonts": {
      "headingFont": "Arial, sans-serif",
      "bodyFont": "Georgia, serif"
    },
    "layoutPreferences": {
      "spacingScale": "normal"
    }
  },
  "chatHistory": []
}
```

This initialization is handled by a Firebase Auth trigger (Cloud Function) that runs on user creation. The default state ensures that:
1. Users can start editing their CV immediately after session creation
2. The application always has a valid state to work with
3. The default template and its customizations are consistently applied
4. The chat history is ready to record interactions

### Firebase Storage

```
cv_uploads/{userId}/original.{docx|pdf} // Stores the original uploaded CV file
// Potentially: generated_pdfs/{userId}/cv.pdf (if temporary server-side storage is chosen before signed URL)
```
Files are accessible only by the authenticated (anonymous) owner.

---

## 6. Data Models

### CV JSON (`cvJson`)

```ts
interface CVJson {
  header: {
    fullName?: string; // Optional to allow chat-based entry
    email?: string;
    phone?: string;
    summary?: string;
  };
  experience: Array<{
    title?: string;
    company?: string;
    location?: string;
    startDate?: string;
    endDate?: string;
    description?: string | string[];
  }>;
  education: Array<{
    degree?: string;
    institution?: string;
    location?: string;
    startDate?: string;
    endDate?: string;
  }>;
  skills?: string[];
  languages?: string[];
  projects?: Array<{
    name?: string;
    description?: string;
  }>;
  // Potentially a rawText field if parsing fails and user needs to work from it via chat
  rawText?: string;
}
```
*Note: Fields are more optional to support progressive creation/correction via chat.*

### Pre-defined Template Structure (Conceptual - managed by the system)

Each pre-defined template (e.g., `templateId: "modern_compact"`, `templateId: "classic_detailed"`) will have its own base HTML/CSS structure. The system knows how to render `cvJson` into these templates.

### User Customizations (`userCustomizations`)

This object stores user-applied modifications to the *selected* pre-defined template.
```ts
interface UserCustomizations {
  colors?: {
    primary?: string;
    secondary?: string;
    background?: string;
    text?: string;
  };
  fonts?: {
    headingFont?: string;
    bodyFont?: string;
  };
  layoutPreferences?: { // High-level adjustments
    spacingScale?: "compact" | "normal" | "spacious"; // Example
    // Section order might be complex for MVP with pre-defined templates, might be deferred
  };
}
```

---
## 7. API Endpoints

All endpoints require a Firebase ID token in the `Authorization: Bearer <FIREBASE_ID_TOKEN>` header and operate on the user's single `activeCV` stored in Firestore unless otherwise specified. For Phase 1, this token is obtained via Firebase Anonymous Login.

**Common Error Response Format:**
All error responses will attempt to follow this format with appropriate HTTP status codes (e.g., 400, 401, 403, 404, 500):

```json
{
  "errorCode": "ERROR_CODE_STRING",
  "message": "A descriptive error message for the client or developer."
}
```
---

### 1. Upload CV

*   **Endpoint:** `/api/cv/upload`
*   **Method:** `POST`
*   **Description:** Uploads a CV file (DOCX or PDF). The backend parses it, extracts structured content into `cvJson`, and stores it in the user's `activeCV`.
*   **Request Headers:**
    *   `Authorization: Bearer <FIREBASE_ID_TOKEN>`
    *   `Content-Type: multipart/form-data`
*   **Request Body:**
    *   `file`: The CV file being uploaded.
*   **Successful Response (200 OK):**
    Returns the parsed `cvJson` and a success message.
    ```json
    {
      "message": "CV uploaded and parsed successfully.",
      "cvJson": {
        "header": {
          "fullName": "John Doe",
          "email": "john.doe@example.com",
          "summary": "Experienced software developer..."
        },
        "experience": [
          {
            "title": "Senior Developer",
            "company": "Tech Solutions Inc.",
            "description": "Developed and maintained web applications."
          }
        ],
        // ... other cvJson fields
        "rawText": "Full extracted text if parsing was partial..." // Only if applicable
      }
    }
    ```
*   **Error Response Example (400 Bad Request - Invalid File Type):**
    ```json
    {
      "errorCode": "INVALID_FILE_TYPE",
      "message": "Uploaded file is not a supported type (DOCX, PDF)."
    }
    ```

---

### 2. Select Design Template

*   **Endpoint:** `/api/cv/select-template`
*   **Method:** `POST`
*   **Description:** Allows the user to select one of the pre-defined design templates for their CV. This updates the `selectedTemplateId` and resets `userCustomizations` to the defaults for that template in the `activeCV`.
*   **Request Headers:**
    *   `Authorization: Bearer <FIREBASE_ID_TOKEN>`
    *   `Content-Type: application/json`
*   **Request Body:**
    ```json
    {
      "templateId": "modern_compact" // Identifier of the chosen pre-defined template
    }
    ```
*   **Successful Response (200 OK):**
    Returns the updated `selectedTemplateId` and the default `userCustomizations` for the chosen template.
    ```json
    {
      "message": "Template selected successfully.",
      "selectedTemplateId": "modern_compact",
      "userCustomizations": { // Default customizations for 'modern_compact'
        "colors": {
          "primary": "#007ACC",
          "text": "#333333"
        },
        "fonts": {
          "headingFont": "Arial, sans-serif",
          "bodyFont": "Georgia, serif"
        },
        "layoutPreferences": {
          "spacingScale": "normal"
        }
      }
    }
    ```
*   **Error Response Example (404 Not Found - Invalid Template ID):**
    ```json
    {
      "errorCode": "TEMPLATE_NOT_FOUND",
      "message": "The specified templateId does not exist."
    }
    ```

---

### 3. Chat Interaction (LLM Edits)

*   **Endpoint:** `/api/chat`
*   **Method:** `POST`
*   **Description:** Sends a user's message to the LLM coordinator for processing and returns the immediate result. The LLM can edit CV content (`cvJson`) or customize the selected design template (`userCustomizations`) based on the `mode` and prompt. The response includes both the updated data and an explanation of the changes made.
*   **Request Headers:**
    *   `Authorization: Bearer <FIREBASE_ID_TOKEN>`
    *   `Content-Type: application/json`
*   **Request Body:**
    ```json
    {
      "mode": "Content-Edit", // or "Layout-Design"
      "userPrompt": "Change my job title for Tech Solutions Inc. to Lead Developer.",
      // The backend will retrieve the current cvJson, selectedTemplateId, and userCustomizations
      // from Firestore based on the authenticated user.
      // Alternatively, for optimistic updates or specific contexts, client could send them:
      // "currentCvJson": { ... },
      // "selectedTemplateId": "modern_compact",
      // "currentUserCustomizations": { ... }
    }
    ```
*   **Successful Response (200 OK):**
    Returns the updated data (`cvJson` or `userCustomizations`) and an explanation from the LLM.
    ```json
    {
      "message": "Edits applied successfully.",
      "updatedData": { // Contains either updatedCvJson or updatedUserCustomizations
        // Example for Content-Edit:
        "cvJson": {
          "header": { /* ... */ },
          "experience": [
            {
              "title": "Lead Developer", // Changed field
              "company": "Tech Solutions Inc.",
              "description": "Developed and maintained web applications."
            }
          ],
          // ...
        }
        // Example for Layout-Design:
        // "userCustomizations": {
        //   "colors": { "primary": "#FF0000" }, ...
        // }
      },
      "llmExplanation": "I've updated your job title at Tech Solutions Inc. to Lead Developer."
    }
    ```
*   **Error Response Example (503 Service Unavailable - LLM Error):**
    ```json
    {
      "errorCode": "LLM_PROCESSING_ERROR",
      "message": "There was an issue communicating with the AI assistant. Please try again."
    }
    ```

---

### 4. Get CV HTML Preview

*   **Endpoint:** `/api/cv/preview`
*   **Method:** `GET`
*   **Description:** Retrieves the current state of the user's CV rendered as HTML, based on their `cvJson`, `selectedTemplateId`, and `userCustomizations`.
*   **Request Headers:**
    *   `Authorization: Bearer <FIREBASE_ID_TOKEN>`
*   **Request Body:** None.
*   **Successful Response (200 OK):**
    *   `Content-Type: text/html`
    *   The response body is the HTML string of the CV.
    ```html
    <!DOCTYPE html>
    <html>
    <head><title>CV Preview</title><!-- styles based on template and customizations --></head>
    <body>
      <!-- CV content rendered here -->
      <div class="header"><h1>John Doe</h1>...</div>
      ...
    </body>
    </html>
    ```
*   **Error Response Example (404 Not Found - No Active CV):**
    ```json
    // Assuming JSON error even if success is HTML
    {
      "errorCode": "NO_ACTIVE_CV",
      "message": "No active CV found for this user to preview."
    }
    ```

---

### 5. Export CV to PDF

*   **Endpoint:** `/api/cv/export`
*   **Method:** `POST`
*   **Description:** Generates a PDF version of the user's current CV and provides a way to download it.
*   **Request Headers:**
    *   `Authorization: Bearer <FIREBASE_ID_TOKEN>`
    *   `Content-Type: application/json`
*   **Request Body:** (Optional, could be empty if all data is taken from `activeCV`)
    ```json
    {
        // Potentially allow passing specific export options in the future,
        // but for MVP, it will use the current activeCV state.
    }
    ```
*   **Successful Response (200 OK):**
    Returns a JSON object containing a signed URL for downloading the PDF.
    ```json
    {
      "message": "PDF generated successfully.",
      "downloadUrl": "https://storage.googleapis.com/your-bucket/user-id/cv_export.pdf?Expires=..."
    }
    ```    *(Alternatively, the server could stream the PDF directly with `Content-Type: application/pdf`)*
*   **Error Response Example (500 Internal Server Error - PDF Generation Failed):**
    ```json
    {
      "errorCode": "PDF_GENERATION_FAILED",
      "message": "Could not generate the PDF. Please try again later."
    }
    ```

---

## 8. LLM Integration

### Prompt Modes (Simplified for MVP)

*   **Content Edit**: `editContent(currentCvJson, userInstructions)`
    *   Focuses on modifying text, adding/removing sections from `cvJson`.
    *   Can also be used to structure content provided by the user via chat if initial parsing failed or for manual entry.
*   **Layout Design**: `customizeTemplate(selectedTemplateId, currentUserCustomizations, userInstructions)`
    *   Focuses on applying user's desired changes (colors, fonts, basic spacing) to the `userCustomizations` object for the `selectedTemplateId`.

### Flow

1.  Client sends: `{ mode, userPrompt, currentCvJson_if_relevant, selectedTemplateId_if_relevant, currentUserCustomizations_if_relevant }`.
2.  Backend (LLM Coordinator Cloud Function) builds system/user prompt tailored to the mode.
3.  Calls OpenAI GPT-4, potentially using function calling to get structured data for `cvJson` updates or `userCustomizations` updates.
4.  Parses LLM response and updates `activeCV` (specifically `cvJson` or `userCustomizations`) in Firestore.
5.  Returns updates to the frontend for live preview.

---

## 9. PDF Export Process

1.  Frontend requests PDF export.
2.  Backend Cloud Function retrieves the `activeCV` (containing `cvJson`, `selectedTemplateId`, `userCustomizations`).
3.  Server renders the HTML using the appropriate pre-defined template, applying the `userCustomizations` to the `cvJson`.
4.  Puppeteer (or html2pdf.js if client-side export is reconsidered, though server-side is more robust) generates the PDF.
5.  Signed URL is returned to the client for download, or PDF is streamed.

---

## 10. Error Handling & Monitoring

*   Typed error responses returned: `{ errorCode, message }`.
*   Logging via Cloud Logging.
*   Frontend error tracking (e.g., Sentry, optional for deep MVP but good practice).
*   Specific handling for CV parsing failures (e.g., storing raw text, guiding user to chat-based input).

---

## 11. Authentication & Security

*   **Phase 1:** Firebase Auth (anonymous login) secures all API endpoints.
*   Backend Cloud Functions verify Firebase ID tokens.
*   Firestore security rules will restrict access to `users/{userId}/activeCV` to the authenticated (anonymous) user.

---

## 12. Deployment

*   Environment: Production focus for MVP.
*   Deployment: Firebase CLI (manual deployment is acceptable for MVP).
*   No CI/CD or advanced infrastructure-as-code for initial MVP.

---

## 13. Known Limitations (MVP)

*   **Single CV per user.**
*   **No user-uploaded design templates (PDF/image/HTML files); users select from system-provided templates.**
*   Limited number of pre-defined templates (3-5).
*   LLM-based layout customization is limited to modifying properties of the selected pre-defined template.
*   No OCR for image-based PDFs within an uploaded CV file.
*   No internationalization (i18n) / localization.
*   Accessibility (a11y) support will be minimal/best-effort for MVP.
*   No admin role-based access.
*   No dedicated metrics dashboard in the application for MVP.
*   **User sessions are anonymous for Phase 1. If a user clears their browser data or logs out, their data may be lost.**