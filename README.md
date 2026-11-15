# Hall Booking System

A full-stack web app for discovering and booking event venues: weddings, corporate events, birthday parties, conferences, and more. The frontend is a React single-page app (Material UI), and the backend is a Django REST API backed by MySQL, with Stripe for card payments.

> **Project status:** work in progress. The UI is largely built, but several parts are not yet wired together end to end. See [Current Status](#current-status) for exactly what works and what doesn't.

---

## Features

- **Authentication pages**: login and sign-up screens with animated MUI forms and a video background
- **Venue discovery**: event category browser (Weddings, Corporate Events, Birthday Parties, Conferences, Private Parties, Charity Events) with per-category pages
- **Home page**: venue search filters (locality, guests, rental cost, venue type) and "Book Now" actions
- **Booking form**: modal dialog capturing name, email, phone, event type, and event date
- **Payments**: Stripe card form that records the payment in the backend
- **Booking history and admin dashboard**: view of bookings and available halls
- **Vendors page**: filterable list of event vendors (catering, photography, etc.) by category and location
- **Photo gallery and About page**
- **ER diagram** of the planned data model in `HallBooking_ER.drawio`

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | React 18, React Router v6, Material UI (MUI v5), Framer Motion, Axios |
| State | React Context (`AuthContext`), Redux (`store.js`), `localStorage` |
| Payments | Stripe (`@stripe/react-stripe-js`, `@stripe/stripe-js`) |
| Backend | Django, Django REST Framework, `django-cors-headers` |
| Database | MySQL (configured in `settings.py`); a sample `db.sqlite3` is also committed |
| Mock / extras | `json-server` (`db.json`), Express + Twilio OTP prototype (`src/Server.js`) |

## Architecture

```
React app (localhost:3000)
   │
   ├── Django REST API (localhost:8000)   ← users, payments  (MySQL)
   ├── json-server     (localhost:5000)   ← mock data from db.json (optional)
   └── Express + Twilio (localhost:5000)  ← OTP prototype, not wired into the UI
```

> `json-server` and the Express OTP prototype both default to port `5000`. Run only one at a time.

## Project Structure

```
.
├── public/                 # Static assets (background videos, payment logos, index.html)
├── src/                    # React frontend
│   ├── App.js              # Routes and protected-route wiring
│   ├── LoginPage.js / SignUpPage.js
│   ├── HomePage.js         # Search filters and venue cards
│   ├── EventCategoriesPage.js / EventDetailsPage.js
│   ├── BookingForm.js / BookingHistory.js
│   ├── Payment.js / CheckoutForm.js   # Stripe checkout
│   ├── AdminDashboard.js / BookingManagement.js
│   ├── VendorPage.js / PhotosPage.js / About.js
│   ├── AuthContext.js / ProtectedRoute.js / store.js
│   └── Server.js           # Express + Twilio OTP prototype
├── website_back/           # Django backend
│   ├── website_back/       # Project settings and root URLs
│   └── myapp/              # Models, serializers, viewsets, routes
├── db.json                 # json-server mock data
├── HallBooking_ER.drawio   # ER diagram
└── package.json
```

## Frontend Routes

| Path | Page | Protected |
|---|---|---|
| `/login`, `/signup` | Authentication | No |
| `/home` | Home with venue search | Yes |
| `/event-categories`, `/event-details/:category` | Event types and venues | Yes |
| `/available-halls`, `/hall-list` | Halls near a location | Yes |
| `/booking-form`, `/booking-history` | Booking flow | Yes |
| `/payment` | Stripe checkout | Yes |
| `/vendors` | Vendor directory | Yes |
| `/product/:id` | Hall detail | Yes |
| `/admin` | Admin dashboard | Yes |
| `/photos`, `/about` | Gallery and About | No |

## Backend API (Django REST Framework)

Base URL: `http://localhost:8000`

| Method | Endpoint | Description |
|---|---|---|
| GET, POST | `/users/` | List or create users |
| GET, PUT, PATCH, DELETE | `/users/{id}/` | Retrieve, update, or delete a user |
| GET, POST | `/payment/` | List or create payment records |
| GET, PUT, PATCH, DELETE | `/payment/{id}/` | Retrieve, update, or delete a payment |
| — | `/admin/` | Django admin site |

**Models:** `User` (email, password) and `PaymentDetail` (user, payment method, amount, timestamp).

## Getting Started

### Prerequisites

- Node.js 18+ and npm
- Python 3.10+
- MySQL 8 (or change `DATABASES` in `settings.py` to SQLite for local testing)
- A [Stripe](https://stripe.com) account (test mode is fine)

### 1. Clone the repo

```bash
git clone https://github.com/karanhub356/App-Dev.git
cd App-Dev
```

### 2. Backend (Django)

There is no `requirements.txt` yet, so install the packages the project imports:

```bash
cd website_back
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install django djangorestframework django-cors-headers mysqlclient
```

Create the MySQL database that `settings.py` expects (`backenddet`):

```sql
CREATE DATABASE backenddet;
```

Then set your own MySQL `USER` and `PASSWORD` in `website_back/website_back/settings.py` under `DATABASES`, and run:

```bash
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver 8000
```

### 3. Frontend (React)

From the repository root:

```bash
npm install
```

Open `src/Payment.js` and replace the `'your-publishable-key'` placeholder with your Stripe **test** publishable key (`pk_test_...`). Then:

```bash
npm start
```

The app runs at `http://localhost:3000`.

### 4. Optional: mock server

```bash
npm run server      # json-server on http://localhost:5000, serving db.json
```

### Available scripts

| Command | What it does |
|---|---|
| `npm start` | Run the React dev server |
| `npm run build` | Production build |
| `npm test` | Run tests |
| `npm run server` | Start json-server on port 5000 |

## Current Status

| Area | Status |
|---|---|
| Login, sign-up, and booking UI | Built |
| Event categories, vendors, gallery, About | Built (static sample data) |
| Django `users` and `payment` API | Working |
| Login | Checks credentials against `/users/` but does not store the logged-in user in `AuthContext`, so protected routes redirect back to `/login` |
| Sign-up | Form only renders the name field (email and password inputs are placeholders) |
| Home page | Uses Redux, but no Redux `<Provider>` wraps the app |
| Bookings | Saved in browser `localStorage`, not the database; there is no `Booking` or `Hall` model (earlier versions were removed in migration `0003`) |
| Booking history | Expects `booking.venue`, but the booking form saves flat fields |
| Admin dashboard | Reads bookings from `localStorage`; any logged-in user can open `/admin` |
| Hall search | Calls `/halls` endpoints that no server currently provides |
| Payments | Stripe key is a placeholder; user ID (`1`) and amount (`1000`) are hard-coded |
| OTP (Twilio) | Prototype in `src/Server.js`, not connected to the UI |

## Security Notes

Fix these before deploying or sharing the project widely:

- **Secrets are committed.** `settings.py` contains the Django `SECRET_KEY` and the MySQL root password. Rotate both, move them to environment variables (for example with `python-decouple` or `django-environ`), and add them to `.gitignore`.
- **Passwords are stored and compared in plain text**, and login works by downloading the full user list from `/users/` and matching in the browser. Use Django's built-in auth with hashed passwords and token or session authentication (for example `djangorestframework-simplejwt`).
- **API endpoints have no permissions.** `IsAuthenticated` is commented out in `views.py`, so anyone can list, edit, or delete users and payments.
- `DEBUG = True` and an empty `ALLOWED_HOSTS` are suitable for development only.
- Remove `db.json` sample credentials, `db.sqlite3`, and `__pycache__` folders from version control.

## Roadmap

- [ ] Call `login()` from `AuthContext` on successful login and persist the session
- [ ] Finish the sign-up form (email, password, confirm password)
- [ ] Add `Hall` and `Booking` models and endpoints, and move bookings out of `localStorage`
- [ ] Hash passwords and add token-based authentication with role-based admin access
- [ ] Real hall search by location, date, capacity, and budget
- [ ] Connect the booking flow to Stripe payments (real amount, user, and booking ID)
- [ ] Availability calendar and double-booking prevention
- [ ] Consolidate state management (pick Context or Redux, not both)
- [ ] Add `requirements.txt`, environment-based config, and automated tests

## Author

**Karan** ([@karanhub356](https://github.com/karanhub356))
