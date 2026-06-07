# EduAlign AI

EduAlign AI is a **submission and review platform** for **asbestos abatement training providers** seeking WorkSafeBC approval for **Level 1: Foundational Awareness** programs.

The platform uses AI-powered curriculum analysis to streamline the approval process by:
- Automatically mapping course materials to WorkSafeBC competency frameworks
- Generating pre-filled curriculum mapping templates
- Enabling structured review workflows for approval officers
- Reducing back-and-forth between reviewers and applicants

---

## Tech Stack

| Layer | Technologies |
|-------|---|
| **Frontend** | React 19, Vite, React Router v7, Firebase SDK, HTML/CSS |
| **Backend** | Node.js, Express, Firebase Admin SDK, Groq API, ExcelJS, pdf-parse, Multer |
| **Infrastructure** | Firebase (Firestore, Storage, Authentication) |

---

## Project Structure

```
EduAlign/
├── backend/
│   ├── config/
│   │   └── constants.js           # Configuration constants
│   ├── controllers/
│   │   ├── authController.js      # Auth handlers
│   │   ├── applicationController.js # Application submission & analysis
│   │   └── reviewerController.js  # Review workflow handlers
│   ├── middleware/
│   │   └── auth.js                # Firebase authentication middleware
│   ├── routes/
│   │   ├── auth.js                # Auth endpoints
│   │   ├── applications.js        # Applicant API routes
│   │   └── reviewer.js            # Reviewer API routes
│   ├── utils/
│   │   ├── firebase.js            # Firebase Admin SDK setup
│   │   ├── groq.js                # Groq AI integration
│   │   ├── pdf.js                 # PDF parsing utilities
│   │   ├── excel.js               # Excel template generation
│   │   └── manageRole.js          # Role management CLI
│   ├── templates/
│   │   └── curriculum-mapping.xlsx # WorkSafeBC template
│   ├── uploads/                   # Temporary file storage
│   ├── server.js                  # Express app entry point
│   ├── package.json
│   └── .env                       # Environment variables (git-ignored)
│
└─�� frontend/
    ├── src/
    │   ├── config/
    │   │   └── constants.js       # Frontend configuration
    │   ├── firebase/
    │   │   └── firebase.js        # Firebase client config
    │   ├── pages/
    │   │   ├── Landing.jsx        # Public landing page
    │   │   ├── Login.jsx          # Login page
    │   │   ├── Signup.jsx         # Registration page
    │   │   ├── ApplicantIndex.jsx # Applicant dashboard
    │   │   ├── ReviewerIndex.jsx  # Reviewer dashboard
    │   │   ├── Application.jsx    # New application form
    │   │   ├── ApplicationRevise.jsx # Revision/resubmission form
    │   │   └── NotFound.jsx       # 404 page
    │   ├── styles/
    │   │   └── *.css              # Component stylesheets
    │   ├── App.jsx                # Main router
    │   ├── main.jsx               # React entry point
    │   ├── index.css              # Global styles
    │   └── App.css                # App styles
    ├── public/                    # Static assets
    ├── index.html
    ├── vite.config.js
    ├── eslint.config.js
    ├── package.json
    └── README.md
```

---

## Configuration

### **Backend** (`backend/config/constants.js`)

| Setting | Value | Notes |
|---------|-------|-------|
| `MAX_FILE_SIZE_MB` | 10 | Maximum file size per upload |
| `MAX_CURRICULUM_FILES` | 10 | Maximum curriculum PDFs (1-10) |
| `APPLICATION_PACKAGE_FILES` | 3 | Required package files (exactly 3) |
| `MAX_INPUT_CHARACTERS` | 10,000 | AI input limit (increase to 50k-100k for production) |
| `MAX_RESPONSE_TOKENS` | 5,000 | AI response limit |

### **Frontend** (`frontend/src/config/constants.js`)

| Setting | Default | Notes |
|---------|---------|-------|
| `API_BASE_URL` | `http://localhost:3000` | Backend API URL |
| `MAX_FILE_SIZE_MB` | 10 | Maximum file size |
| `MAX_CURRICULUM_FILES` | 10 | Maximum curriculum PDFs |
| `REQUIRED_PACKAGE_FILES` | 3 | Required package files |

### **Environment Variables** (`backend/.env`)

```env
# Firebase Admin SDK
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_CLIENT_EMAIL=your-service-account@project.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
FIREBASE_STORAGE_BUCKET=your-project.appspot.com

# Groq API (free tier available)
GROQ_API_KEY=gsk_...

# Server Configuration
PORT=3000
```

**Note:** Frontend uses no `.env` file. Configure the API URL via `VITE_API_BASE_URL` environment variable if needed.

---

## Getting Started

### **Prerequisites**

- **Node.js** v16 or higher
- **Firebase project** with Firestore, Storage, and Authentication enabled
- **Groq API key** (free tier available at [console.groq.com](https://console.groq.com))

### **Installation**

#### **1. Clone repository and install dependencies**

```bash
git clone https://github.com/StewartRogers/EduAlign.git
cd EduAlign

# Backend setup
cd backend
npm install

# Frontend setup (in new terminal from root)
cd frontend
npm install
```

#### **2. Configure backend environment**

Create `backend/.env` with your Firebase and Groq credentials (see Configuration section above).

#### **3. Start development servers**

```bash
# Terminal 1: Backend (runs on http://localhost:3000)
cd backend
npm run dev        # Uses nodemon for auto-reload
# or
npm start          # Standard start

# Terminal 2: Frontend (runs on http://localhost:5173)
cd frontend
npm run dev
```

#### **4. Set up first reviewer (optional)**

```bash
cd backend
node utils/manageRole.js user@example.com
```

---

## Application Workflow

### **1. Submit Application**
- Applicant signs up and logs in
- Uploads 1-10 curriculum PDFs + 3 required package files:
  - Provider Form
  - Course Outline
  - Administration Document

### **2. AI Analysis**
- Groq AI (`llama-3.3-70b-versatile`) analyzes curriculum
- Maps content to 26 Level 1 competencies
- Identifies where/how each competency is taught and assessed

### **3. Generate Excel Template**
- Pre-filled WorkSafeBC mapping template generated
- Mappings included with AI analysis
- Missing competencies marked as "Not covered"

### **4. Review & Feedback**
- Reviewer receives application with generated Excel
- Updates status: **Unreviewed** → **Incomplete** (if revisions needed) → **Approved**

### **5. Revision** (if needed)
- Applicant uploads revised curriculum
- Creates new version (preserves complete history)
- AI re-analyzes and generates updated template

---

## API Endpoints

### **Authentication**
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/set-role` | Set user role (default: applicant, can grant: reviewer) |

### **Applicant** (requires authentication)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/applications/analyze` | Preview analysis without saving |
| `POST` | `/api/applications/submit` | Submit new application |
| `POST` | `/api/applications/revise/:id` | Create revision of existing application |
| `GET` | `/api/applications/my-applications` | List user's applications |
| `GET` | `/api/applications/my-applications/:id` | Get application details |

### **Reviewer** (requires reviewer role)
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/reviewer/applications` | List all applications with stats |
| `GET` | `/api/reviewer/applications/:id` | Get application details |
| `PATCH` | `/api/reviewer/applications/:id/status` | Update application status |

### **Utility**
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/health` | Health check endpoint |

---

## Key Features

### **AI-Powered Analysis**
- **Model:** Groq `llama-3.3-70b-versatile`
- **Input Limit:** 10,000 characters (configurable; 50k-100k for production)
- **Output:** JSON with competency mappings and missing criteria
- **Temperature:** 0.0 (deterministic results)

### **Role-Based Access Control**
- **Roles:** `applicant` (default), `reviewer`
- **Authentication:** Firebase custom claims
- **Middleware:** `requireAuth`, `requireReviewerRole`

### **File Management**
- **Storage:** Firebase Storage with signed URLs
- **Path Structure:** `applications/{applicationId}/{timestamp}_{filename}`
- **URL Validity:** 1 day (regenerated as needed)
- **Limits:** 10MB per file, 15MB request body, 60 requests/hour per IP

### **Version Control & Audit Trail**
- Complete version history preserved for each application
- All files retained for audit purposes
- New revisions created without deleting previous versions
- Timestamps track submission and analysis dates

---

## Database Schema

```javascript
{
  applicationId: "APP20250403ABCD",           // Human-readable ID
  userId: "firebase-uid",                     // Firebase user ID
  providerName: "John Doe",
  organizationName: "ABC Training Inc",
  email: "provider@example.com",
  status: "Unreviewed" | "Incomplete" | "Approved",
  submittedDate: Date,
  currentVersion: 2,
  versions: [
    {
      version: 1,
      analyzedAt: Date,
      curriculumFiles: [
        {
          name: "filename.pdf",
          storagePath: "applications/APP.../timestamp_filename.pdf",
          uploadedAt: Date
        }
      ],
      applicationPackageFiles: [
        { name: "Provider Form", ... },
        { name: "Course Outline", ... },
        { name: "Admin Document", ... }
      ],
      excelFile: {
        name: "mapping_v1.xlsx",
        storagePath: "applications/APP.../timestamp_mapping.xlsx"
      },
      mappings: [
        {
          competency: "1.1 Health Effects",
          taught: "Chapter 2, pages 15-20",
          assessed: "Quiz 1, Question 3"
        }
      ],
      missingCriteria: ["1.5 Exposure Routes", ...]
    }
  ],
  createdAt: Timestamp,
  updatedAt: Timestamp
}
```

---

## Development Commands

### **Backend**
```bash
# Development with auto-reload
npm run dev

# Production start
npm start

# Set reviewer role for a user
node utils/manageRole.js user@example.com
```

### **Frontend**
```bash
# Development server with HMR
npm run dev

# Production build
npm build

# Preview production build
npm run preview

# ESLint check
npm run lint
```

---

## Production Deployment Checklist

- [ ] Upgrade Groq API to paid plan
- [ ] Increase `MAX_INPUT_CHARACTERS` to 50,000–100,000
- [ ] Increase `MAX_RESPONSE_TOKENS` to 25,000–50,000
- [ ] Configure CORS for production domain
- [ ] Set `VITE_API_BASE_URL` to production API URL
- [ ] Enable Firebase security rules
- [ ] Set up Firebase backups
- [ ] Configure rate limiting appropriately
- [ ] Enable HTTPS
- [ ] Set up monitoring and error tracking

---

## Troubleshooting

### **Frontend can't reach backend**
- Ensure backend is running on `http://localhost:3000`
- Check `frontend/src/config/constants.js` for correct `API_BASE_URL`
- Verify CORS is enabled in `backend/server.js`

### **Firebase authentication failing**
- Verify Firebase credentials in `backend/.env`
- Check Firebase project has Authentication enabled
- Ensure Firebase SDK is initialized in `frontend/src/firebase/firebase.js`

### **Groq API errors**
- Verify `GROQ_API_KEY` is valid at [console.groq.com](https://console.groq.com)
- Check request size doesn't exceed `MAX_INPUT_CHARACTERS`
- Verify response size doesn't exceed `MAX_RESPONSE_TOKENS`

---

## Contributing

This project is a student-sponsored initiative developed for WorkSafeBC. Please refer to the LICENSE for terms.

---

## License

Copyright (c) 2025 WorkSafeBC. All rights reserved.

This project is a student-sponsored initiative developed for WorkSafeBC.  
Unauthorized copying, modification, or distribution is prohibited.

**Authors:** Clinton Nguyen, Kevin Ung, Shawn Lee, Tommy Tang

**Acknowledgments:** WorkSafeBC, Groq, Firebase
