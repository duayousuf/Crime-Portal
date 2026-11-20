# Crime Portal

A full-stack **Crime Monitoring Portal** built with the MERN stack. Citizens of Karachi can register complaints about crimes (even anonymously), confirm and discuss each other's reports, follow the status of their complaints, and explore live crime statistics on charts and a map. Every number, chart and card on the site comes from the database.

## Features

**Complaints**
- Register a complaint with title, description, region, area, category and an optional image
- Edit and delete your own complaints
- **Anonymous reporting:** your name and photo are hidden from other users (hidden on the server, not only on screen)
- **Complaint status:** Pending, Under Review, Resolved (changed by an admin)
- **Confirm incident ("I also saw this"):** other users can confirm a report, and 3 confirmations mark it as *Verified by community*
- Like, dislike and comment (the comment owner or the post owner can delete a comment)
- Search by title, description, area or name, filter by region, category and status, and sort by newest, most liked, most commented or most confirmed

**Live data (no dummy content)**
- Home page: district-wise bars, recent crimes, category counts, community numbers, area safety scores and most discussed crimes
- Crime Reports page: pie chart (by district), line chart (by category), complaint status progress, area safety scores and live counts
- **Area Safety Score:** a score out of 100 for each district from the complaints of the last 30 days (serious crimes lower it more, resolved complaints lower it less)
- **Crime Map:** Karachi map (Leaflet + OpenStreetMap) with a pin for each area, filters by category, status and time, and a list of the most reported areas

**Users**
- Register and log in (JWT authentication, passwords hashed with bcrypt)
- **Notifications:** bell icon and a notifications page for likes, comments, confirmations and status changes on your complaints
- Profile page with your photo, name, email, your own complaints and the likes/comments you received (name and photo can be edited)
- **Emergency numbers page** (police, ambulances, fire, rescue) with call and copy buttons, and a floating emergency button on every page
- Contact form saved to the database, with a "Your Messages" list showing which messages were seen
- Newsletter subscription (emails are saved to the database)

**Admin**
- Change the status of any complaint
- Read, mark as read/unread and delete contact messages

**Design**
- Responsive UI with Tailwind CSS, scroll animations, count-up numbers, hover effects and a mouse-follow spotlight

## Tech Stack

| Part | Technologies |
|---|---|
| Frontend | React 18, Vite, React Router, Tailwind CSS, Axios, Chart.js (react-chartjs-2), Leaflet (react-leaflet), React Toastify, React Icons |
| Backend | Node.js, Express 5, Mongoose, JSON Web Token, bcryptjs, Multer, CORS, dotenv |
| Database | MongoDB (local or MongoDB Atlas) |

## Project Structure

```
Portal/
├── Backend/
│   ├── index.js              # Express app and routes setup
│   ├── db.js                 # MongoDB connection
│   ├── config.js             # JWT secret
│   ├── middleware/           # authMiddleWare (login), adminOnly
│   ├── models/               # User, Post, Contact, Subscriber, Notification
│   ├── Routes/               # auth, post, contact, newsletter, notification
│   ├── utils/                # notify (create notifications), areas (Karachi areas)
│   └── uploads/              # Uploaded images (post images, profile images)
└── Frontend/
    ├── index.html
    ├── tailwind.config.js
    └── src/
        ├── components/       # Navbar, Footer, PageHero, AuthLayout, Motion, NotificationBell,
        │                     # EmergencyButton, SafetyScores, status, areas, useStats
        ├── pages/            # Home, About, ContactUs, CrimeReport, CrimeResponse, CrimeMap,
        │                     # Emergency, Notifications, Profile, AdminMessages,
        │                     # Login, Register, Forgot_password
        └── images/
```

## Getting Started (Local)

### Requirements
- Node.js 18 or newer
- MongoDB running locally (or a MongoDB Atlas connection string)

### 1. Backend

```bash
cd Backend
npm install
```

Create a `.env` file (copy `.env.example`) and fill it:

```env
PORT=4000
MONGO_URI=mongodb://localhost:27017/crime-portal
JWT_SECRET=write_a_long_random_text_here
CLIENT_URL=http://localhost:5173
```

Run it:

```bash
npm run dev      # development (auto restart with nodemon)
npm start        # production
```

The API runs on `http://localhost:4000`.

### 2. Frontend

```bash
cd Frontend
npm install
```

Create a `.env` file (copy `.env.example`):

```env
VITE_API_URL=http://localhost:4000
```

Run it:

```bash
npm run dev
```

The app runs on `http://localhost:5173`.

### 3. Make yourself an admin (optional)

Register a normal account first. Then, in MongoDB Compass, open the `users` collection, edit your user document and add a field:

```
isAdmin: true     (type: Boolean)
```

Log out and log in again. Now you can change the status of complaints on the Crime Response page, and a **Contact Messages (Admin)** button appears on your Profile page (page URL: `/admin_messages`). Nobody can become an admin from the website itself.

## API Overview

All routes need the header `auth-token: <JWT>`, except register, login, add_contact and newsletter subscribe.

**Auth**

| Method | Route | Description |
|---|---|---|
| POST | `/api/auth/register` | Register (multipart form with profile image) |
| POST | `/api/auth/login` | Login, returns a token |
| GET | `/api/auth/getuser` | Logged in user details |
| PUT | `/api/auth/update_profile` | Update name and photo |

**Complaints** (`/api/posts`)

| Method | Route | Description |
|---|---|---|
| GET | `/get_all_post` | All complaints (newest first, anonymous names hidden) |
| GET | `/my_posts` | Complaints of the logged in user |
| POST | `/add_post` | Add a complaint (multipart form) |
| PUT | `/edit_post/:id` | Edit own complaint |
| DELETE | `/delete_post/:id` | Delete own complaint |
| PUT | `/Add_Like/:id`, `/Add_disLike/:id` | Like or dislike |
| PUT | `/confirm/:id` | Confirm or un-confirm an incident |
| POST | `/add_comment/:id` | Add a comment |
| DELETE | `/delete_comment/:postId/:commentId` | Delete a comment |
| PUT | `/set_status/:id` | Change status (admin only) |
| GET | `/stats` | Counts for charts, latest and most discussed posts |
| GET | `/safety` | Safety score of each district (last 30 days) |
| GET | `/map` | Complaints grouped by area (filters: category, status, days) |

**Other**

| Method | Route | Description |
|---|---|---|
| GET | `/api/notifications` | My notifications and unread count |
| GET | `/api/notifications/unread_count` | Only the unread count |
| PUT | `/api/notifications/read_all`, `/read/:id` | Mark as read |
| DELETE | `/api/notifications/:id` | Delete a notification |
| POST | `/api/contact/add_contact` | Send a contact message |
| GET | `/api/contact/my_messages` | Messages sent by the logged in user |
| GET | `/api/contact/all` | All messages (admin only) |
| PUT | `/api/contact/mark_read/:id` | Toggle read status (admin only) |
| DELETE | `/api/contact/delete/:id` | Delete a message (admin only) |
| POST | `/api/newsletter/subscribe` | Subscribe an email |

## How the Area Safety Score Works

Every district starts at 100. Each complaint of the last 30 days lowers the score: murder, target killing, bomb blast and gang rape take 5 points, robbery 3 points and snatching 2 points. A resolved complaint counts only half. The total points are multiplied by 4 and subtracted from 100. A score of 80 or more is *Safe*, 50 to 79 is *Moderate* and below 50 is *High Risk*. The numbers can be changed at the top of the safety route in `Backend/Routes/post.js`.

## Deployment

1. **Database:** create a free cluster on MongoDB Atlas, create a database user, allow network access, and copy the connection string.
2. **Backend** (for example on Render): root directory `Backend`, build command `npm install`, start command `npm start`. Add the environment variables `MONGO_URI`, `JWT_SECRET` and `CLIENT_URL` (your frontend link).
3. **Frontend** (for example on Vercel or Netlify): root directory `Frontend`, build command `npm run build`, output directory `dist`. Add the environment variable `VITE_API_URL` (your backend link, without a trailing slash). `vercel.json` and `public/_redirects` are included so page refresh works with React Router.

## Known Limitations

- **Uploaded images** are stored on the server disk. Most free hosting platforms delete these files when the server restarts. For a permanent solution, move image storage to a service like Cloudinary.
- **Forgot Password** is only a UI. There is no email sending or reset API yet.
- **Newsletter** saves emails to the database but does not send any emails.
- **Map pins** are placed at the approximate centre of each area, not at the exact place of a complaint (this protects the privacy of the person reporting). Only complaints that have an area selected appear on the map.
- **Notifications** refresh every 30 seconds (polling), not instantly.
- **Emergency numbers** were collected from public sources and can change. Please verify them before depending on them.
- The interface is English only.
- Privacy Policy and Terms links in the footer are placeholders.
- The auth token is stored in `localStorage`.

## Author

Made by **Hisra Khan** as a university assignment.