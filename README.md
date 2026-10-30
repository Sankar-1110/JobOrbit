# JobOrbit 🚀

JobOrbit is a full-stack job aggregation platform that scrapes live job listings from multiple company career portals, stores them in MongoDB, and provides a searchable dashboard through a React frontend.

## Tech Stack

- **Frontend:** React + Vite
- **Backend:** Node.js + Express
- **Database:** MongoDB Atlas
- **Scraper Platforms:** Greenhouse, Lever, Workday
- **Deployment:** Vercel (frontend), Render (backend), MongoDB Atlas (database)

## Live Deployment

- **Frontend:** https://job-orbit-nine.vercel.app/
- **Backend:** https://joborbit-3hn1.onrender.com/

## Features

- Search jobs by role, location, experience, and category
- Automated background scraping on a scheduled cron job
- Manually trigger scraping through the dashboard or API
- MongoDB-backed job storage for faster search responses
- Pagination for scalable job listing queries
- Dashboard statistics for active jobs, active companies, and scraping status
- Support for multiple scraper platforms through a registry-based backend

## Architecture

```text
                         User
                           |
                           v
                  React Frontend (Vercel)
                           |
                           v
                 Node.js + Express API
                        (Render)
                           |
                           v
                    Scraper Service
                           |
              +------------+------------+
              |            |            |
              v            v            v
          Greenhouse     Lever       Workday
              |            |            |
              +------------+------------+
                           |
                           v
                     MongoDB Atlas
```

## Data Flow

1. A scheduled job runs every six hours.
2. The backend loads active companies from MongoDB.
3. Each company is processed by the scraper registered for its `scraper_type`.
4. Scraped jobs are normalized and upserted into the `jobs` collection.
5. The frontend queries `/api/jobs` and `/api/stats`.
6. Users browse stored job listings without waiting for live scraping on every request.

## Project Structure

```text
JobOrbit/
├── backend/
│   ├── config/
│   ├── controllers/
│   ├── data/
│   ├── models/
│   ├── routes/
│   ├── scheduler/
│   ├── scrapers/
│   ├── scripts/
│   ├── services/
│   ├── utils/
│   ├── package.json
│   └── server.js
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── config/
│   │   ├── data/
│   │   ├── pages/
│   │   ├── services/
│   │   └── styles/
│   ├── package.json
│   └── vite.config.js
└── README.md
```

## Backend

### Tech Stack

- Node.js
- Express.js
- Mongoose
- Axios
- Cheerio
- node-cron

### Current Scraper Support

#### Greenhouse

- Stripe
- Airtable
- Datadog
- Rubrik
- MongoDB
- Figma
- Postman

#### Lever

- Dun & Bradstreet
- Mistral AI

#### Workday

- Workday
- Hitachi Energy

### Core Backend Files

| File | Responsibility |
|---|---|
| `backend/server.js` | Application initialization and server startup |
| `backend/config/db.js` | MongoDB connection |
| `backend/controllers/jobController.js` | Job API logic |
| `backend/routes/jobRoutes.js` | API route definitions |
| `backend/services/scrapeService.js` | Scraper orchestration and database upserts |
| `backend/scheduler/cronJobs.js` | Scheduled scraping |
| `backend/models/Job.js` | Job schema |
| `backend/models/Company.js` | Company schema |
| `backend/data/companies.js` | Initial company list |

### Job Schema

```json
{
  "company": "Airtable",
  "title": "Software Engineer, Product Backend (2-8 YOE)",
  "location": "San Francisco, CA; New York, NY; Remote (Seattle, WA only)",
  "experience": "2-8 years",
  "skills": [],
  "job_link": "https://job-boards.greenhouse.io/airtable/jobs/7845291002",
  "source_job_id": "7845291002",
  "scraper_type": "greenhouse",
  "scraped_at": "2026-03-13T16:58:10.623Z",
  "is_active": true
}
```

### Company Schema

```json
{
  "company_name": "Workday",
  "career_url": "https://workday.wd5.myworkdayjobs.com/en-US/Workday",
  "workday_api_url": "https://workday.wd5.myworkdayjobs.com/wday/cxs/workday/Workday/jobs",
  "scraper_type": "workday",
  "is_active": true,
  "last_scraped_at": "2026-03-13T17:50:46.317Z",
  "last_scrape_status": "success"
}
```

## Frontend

### Tech Stack

- React 18
- Vite 5
- CSS Modules

### Main Screens

- **HomePage:** Landing page and job search entry point
- **DashboardPage:** Job dashboard with filters, statistics, pagination, and a manual scraping trigger

### Core Frontend Files

| File | Responsibility |
|---|---|
| `frontend/src/App.jsx` | Application-level navigation |
| `frontend/src/pages/HomePage.jsx` | Landing page |
| `frontend/src/pages/DashboardPage.jsx` | Job dashboard |
| `frontend/src/components/Navbar.jsx` | Homepage navigation |
| `frontend/src/components/Sidebar.jsx` | Dashboard navigation |
| `frontend/src/components/JobCard.jsx` | Job display card |
| `frontend/src/services/api.js` | Backend API calls |
| `frontend/src/services/jobData.js` | API data normalization for the UI |

## API Documentation

### Base URLs

- **Local:** `http://localhost:5000`
- **Production:** `https://career-crawler-1.onrender.com`

### `GET /api/health`

Checks whether the backend is running.

Example response:

```json
{
  "status": "ok",
  "service": "careercrawler-backend",
  "timestamp": "2026-03-13T16:40:37.259Z"
}
```

### `GET /api/jobs`

Fetches paginated job listings with optional filters.

**Query parameters:**

- `role`
- `location`
- `experience`
- `category`
- `page`
- `limit`

Example request:

```text
/api/jobs?role=backend&location=India&experience=1-3%20years&page=1&limit=20
```

Example response:

```json
{
  "items": [],
  "total": 506,
  "page": 1,
  "limit": 20,
  "totalPages": 26
}
```

### `GET /api/jobs/all`

Returns all active job listings without pagination.

### `GET /api/stats`

Returns dashboard metrics.

Example response:

```json
{
  "activeJobs": 506,
  "activeCompanies": 11,
  "lastScrapedAt": "2026-03-13T17:50:46.317Z",
  "scrapeInProgress": false,
  "lastScrapeCompletedAt": null,
  "nextSchedule": "0 */6 * * *"
}
```

### `GET /api/scrape/status`

Returns the current in-memory scraping status.

### `POST /api/scrape`

Triggers a manual scraping run.

Example response:

```json
{
  "message": "Scrape complete - 11 companies processed",
  "companiesProcessed": 11,
  "jobsUpserted": 506,
  "completedAt": "2026-03-13T17:50:46.317Z",
  "errors": [],
  "resultsCount": 11
}
```

## Local Development

### Prerequisites

- Node.js 18 or later
- npm
- MongoDB Atlas cluster

### 1. Clone the Repository

```bash
git clone https://github.com/Sankar-1110/JobOrbit.git
cd JobOrbit
```

### 2. Backend Setup

Navigate to the backend directory:

```bash
cd backend
npm install
```

Create a `backend/.env` file:

```env
PORT=5000
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/j oborbit?retryWrites=true&w=majority
SCRAPE_CRON=0 */6 * * *
SCRAPE_CONCURRENCY=5
SCRAPE_DELAY_MS=2000
ENABLE_SCHEDULER=true
```

**Important:** Replace the MongoDB connection string with your actual connection string. Use `joborbit` as the database name if desired, and ensure the URI contains no accidental spaces.

Seed the company list:

```bash
npm run seed:companies
```

Start the backend:

```bash
npm run dev
```

### 3. Frontend Setup

Open a new terminal and navigate to the frontend directory:

```bash
cd frontend
npm install
```

Create a `frontend/.env` file:

```env
VITE_USE_MOCK_DATA=false
VITE_API_BASE_URL=http://localhost:5000
```

Start the frontend:

```bash
npm run dev
```

### 4. Open the Application

- **Frontend:** http://localhost:5173
- **Backend:** http://localhost:5000

## Deployment Guide

### MongoDB Atlas

1. Create a MongoDB Atlas cluster.
2. Create a database user.
3. Allow network access for your Render service.
4. Copy the MongoDB connection string.
5. Configure the database name as `joborbit` if you want to use the new name.

Example environment variable:

```env
MONGODB_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/j oborbit?retryWrites=true&w=majority
```

Replace the example with a valid connection string and remove any spaces from the database path.

**Note:** Changing the database name in the connection string may cause the application to use a different database. Existing job and company records will not automatically move to it.

### Backend Deployment on Render

Create a new Web Service from the GitHub repository.

**Recommended settings:**

- Root Directory: `backend`
- Build Command: `npm install`
- Start Command: `npm start`

Configure the following environment variables:

```env
PORT=10000
MONGODB_URI=<your_mongodb_atlas_uri>
SCRAPE_CRON=0 */6 * * *
SCRAPE_CONCURRENCY=5
SCRAPE_DELAY_MS=2000
ENABLE_SCHEDULER=true
```

**Notes:**

- Render automatically provides the `PORT` environment variable. Your backend should read `process.env.PORT`.
- Scheduled scraping only works while the backend service is running.
- If your hosting plan sleeps while idle, scheduled jobs may not execute as expected.

After deployment, verify:

- Health check: `https://career-crawler-1.onrender.com/api/health`
- Statistics: `https://career-crawler-1.onrender.com/api/stats`

### Frontend Deployment on Vercel

Create a Vercel project using the same GitHub repository.

**Recommended settings:**

- Root Directory: `frontend`
- Framework Preset: `Vite`
- Build Command: `npm run build`
- Output Directory: `dist`

Configure these environment variables:

```env
VITE_USE_MOCK_DATA=false
VITE_API_BASE_URL=https://career-crawler-1.onrender.com
```

After deployment:

- **Frontend:** https://career-crawler-6tga.vercel.app/

You can optionally configure a custom domain containing `joborbit` later.

## Production Checklist

Before considering the deployment complete, verify:

- [ ] Rotate database credentials if they were ever exposed.
- [ ] Set `VITE_API_BASE_URL` to the correct backend URL.
- [ ] Confirm `MONGODB_URI` is configured correctly on Render.
- [ ] Confirm CORS allows requests from the frontend.
- [ ] Seed companies in production at least once.
- [ ] Verify `/api/health`.
- [ ] Verify `/api/stats`.
- [ ] Verify `/api/jobs`.
- [ ] Verify `/api/scrape`.

## Known Limitations

- `lastScrapeCompletedAt` is stored in memory and resets when the backend restarts.
- Saved jobs are currently UI-only and are not persisted to the database.
- Location normalization can be improved for some Workday values.
- The custom dropdown UI needs further refinement.
- Google and Amazon custom scrapers are not implemented yet.

## Future Improvements

- Persist scraping history.
- Add Google and Amazon scrapers.
- Add more India-focused multinational companies.
- Improve location and experience normalization.
- Persist saved jobs for individual users.
- Add resume parsing and skill-gap analysis.

## Security

Never commit real credentials or secrets to:

- `backend/.env`
- `frontend/.env`
- `.env.example`

If a MongoDB password or API key has been exposed, rotate it immediately in the provider dashboard and update the deployment environment variables.

---

**JobOrbit — Discover opportunities. Navigate your career.**
