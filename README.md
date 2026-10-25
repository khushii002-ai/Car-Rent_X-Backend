# CarRentX Backend

Node.js + Express + SQLite REST API for the CarRentX website
(https://khushii002-ai.github.io/Car-Rent_X/).

Covers: signup/login (JWT), personal info, car listing, availability, booking
requests with server-side pricing and double-booking protection, booking
lookup/cancel, and an admin API to confirm or reject requests.

## Setup (local)

1. **Install Node.js 18+** from https://nodejs.org and check with `node -v`.
2. **Open a terminal in this folder** and install packages:
   ```bash
   npm install
   ```
3. **Create your env file:**
   ```bash
   cp .env.example .env        # Windows: copy .env.example .env
   ```
   Edit `.env`:
   - `JWT_SECRET`: generate one with
     `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"`
   - `ADMIN_EMAIL`: the email you will sign up with to become admin
   - `CORS_ORIGINS`: keep `https://khushii002-ai.github.io` and add any local origin you use
4. **Start the server:**
   ```bash
   npm run dev      # auto-restarts on changes
   # or
   npm start
   ```
   The database file is created at `data/carrentx.db` and the four cars are seeded automatically.
5. **Check it works:** open http://localhost:4000/api/health (should show `{"ok":true}`)
   and http://localhost:4000/api/cars.

## Test with curl

```bash
# Sign up
curl -X POST localhost:4000/api/auth/signup -H "Content-Type: application/json" \
  -d '{"name":"Test User","email":"test@example.com","password":"password123"}'

# Book a car (guest booking)
curl -X POST localhost:4000/api/bookings -H "Content-Type: application/json" -d '{
  "car":"swift","pickupDate":"2026-12-01","returnDate":"2026-12-03",
  "pickupTime":"10:00","returnTime":"10:00","pickupLocation":"Noida",
  "fullName":"Test User","mobile":"9876543210","email":"test@example.com"}'
```

## Connect your website

1. Copy `frontend-snippet/api.js` into your website repo and include it with
   `<script src="api.js"></script>` on login, signup, booking and renter-details pages.
2. In `booking.html`, set each car `<option value>` to its slug:
   `swift`, `city`, `nexon`, `c-class`, then call `CarRentX.book({...})` on submit
   (example in the file).
3. Login/signup forms call `CarRentX.login(email, password)` / `CarRentX.signup(name, email, password)`.
4. Renter details form calls `CarRentX.saveProfile({ phone, licenseNumber, address, city, dob })`.

Your site is on GitHub Pages (HTTPS), so browsers will block calls to an
`http://localhost` API from the live site. Test locally first, then deploy the
backend (below) and change `API_BASE` in `api.js` to the deployed URL.

## Deploy (Render example)

1. Push this folder to a GitHub repo (`.env` and `data/` are git-ignored).
2. On https://render.com create a **Web Service** from the repo.
   - Build command: `npm install`
   - Start command: `npm start`
3. Add environment variables: `JWT_SECRET`, `ADMIN_EMAIL`,
   `CORS_ORIGINS=https://khushii002-ai.github.io`
4. **Persistence:** SQLite is a file. On Render, attach a Disk (e.g. mounted at `/var/data`)
   and set `DB_PATH=/var/data/carrentx.db`. Without it, data is lost on each redeploy.
5. Set `API_BASE` in `api.js` to `https://your-service.onrender.com/api`.

## API reference

| Method | Endpoint | Auth | Purpose |
|---|---|---|---|
| GET | `/api/health` | - | Health check |
| POST | `/api/auth/signup` | - | `{name,email,password}` |
| POST | `/api/auth/login` | - | `{email,password}` returns token |
| GET | `/api/auth/me` | User | Current user |
| PUT | `/api/auth/profile` | User | Personal info |
| GET | `/api/cars` | - | All cars |
| GET | `/api/cars/:slug` | - | One car |
| GET | `/api/cars/available?pickup=&return=` | - | Cars free for dates (`YYYY-MM-DDTHH:MM`) |
| POST | `/api/bookings` | Optional | Create booking request |
| GET | `/api/bookings/mine` | User | My bookings |
| GET | `/api/bookings/lookup?reference=&email=` | - | Guest lookup |
| POST | `/api/bookings/:reference/cancel` | User | Cancel own booking |
| GET | `/api/admin/bookings?status=` | Admin | All bookings |
| PATCH | `/api/admin/bookings/:reference/status` | Admin | `confirmed`/`rejected`/`cancelled`/`completed` |
| PATCH | `/api/admin/cars/:slug` | Admin | Change `dailyRate` or `active` |

Send the token as `Authorization: Bearer <token>`.

## Notes

- Booking status flow: `pending` → `confirmed` or `rejected` (matches the site's
  "not confirmed until the provider verifies" message). Only confirmed bookings block dates.
- Total = days x daily rate, with partial days rounded up (minimum 1 day). Taxes,
  deposit and fuel are not included, same as the site's disclaimer.
- No payments or email/SMS notifications yet. Next steps would be Razorpay and
  Nodemailer/Twilio.
