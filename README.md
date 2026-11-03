# AniSidd Safety Network

A redesigned responsive safety web app based on `AniSiddProject.html`, with a C++ REST backend, SQLite storage, real SMS OTP verification through Twilio Verify, trusted contacts, location recording, an SOS event log, and a map showing nearby police facilities with ⭐ markers.

## Project layout

- `public/index.html` — app markup and pages
- `public/styles.css` — responsive styling and animations
- `public/app.js` — browser logic, geolocation, contacts import where supported, map, and API calls
- `backend/main.cpp` — C++ REST API, SQLite persistence, Twilio Verify and SMS integration
- `backend/CMakeLists.txt` — CMake build

## Important safety and privacy limits

- The app asks the browser for location permission. A normal web page cannot force permission or guarantee continuous/background location access after the page is closed. Use HTTPS in production.
- Contact import uses the browser Contact Picker API only where supported and only after the user taps Import contacts. Manual contact entry is available everywhere.
- The map tiles and nearby police station listings use OpenStreetMap/Overpass data. A Google Maps link opens the current location. Station data may be incomplete or delayed; verify emergency information independently.
- **SOS is not a police dispatch integration.** The backend records the event and attempts SMS alerts to the user's saved contacts. It does not automatically alert a police station or emergency dispatch service. For immediate danger in India, call **112**. A real police dispatch integration requires explicit approval and an official integration with the relevant authority.
- No sound is played by the SOS UI; vibration is attempted with the browser Vibration API. Browser/device support varies.
- Location/contact collection should be disclosed and consented to. Store production secrets outside source control, use HTTPS, configure data retention/deletion, rate-limit OTP and SOS endpoints, and have a security review before real-world deployment.

## Configure real 5-digit OTP

1. Create a Twilio account and a Twilio Verify Service.
2. In the Verify Service settings, set the code length to **5 digits**. The API uses Twilio Verify so the server never generates or displays a pretend OTP.
3. Add the service SID, account SID, and auth token as environment variables using `.env.example` as a reference. Do not commit actual credentials. Twilio trial accounts may restrict recipient numbers and countries; comply with provider rules and local law.
4. Set `TWILIO_FROM_NUMBER` to an SMS-enabled Twilio number for sending SOS texts to trusted contacts.

If provider credentials are absent, OTP endpoints return an explicit configuration error; there is no hard-coded bypass code.

## Build backend (Linux/macOS with CMake, C++17, SQLite and libcurl)

```bash
cd backend
cmake -S . -B build
cmake --build build -j
```

The build fetches `cpp-httplib` and `nlohmann/json` via CMake FetchContent, so the first build needs internet access. Install the SQLite and libcurl development packages before building (for Debian/Ubuntu: `sudo apt install build-essential cmake libsqlite3-dev libcurl4-openssl-dev`).

Export configuration in the same terminal, then run the executable:

```bash
export TWILIO_ACCOUNT_SID='AC...'
export TWILIO_AUTH_TOKEN='...'
export TWILIO_VERIFY_SERVICE_SID='VA...'
export TWILIO_FROM_NUMBER='+1...'
export CORS_ORIGIN='http://localhost:5500'
./build/anisidd-api
```

For local development, open a second terminal at the project root:

```bash
python3 -m http.server 5500 --directory public
```

Open `http://localhost:5500`. The frontend defaults to the API at `http://localhost:8080`. Change `window.API_BASE` in `public/index.html` for a deployed API URL. Browser geolocation generally requires HTTPS or localhost.

## API routes

- `GET /api/health`
- `POST /api/auth/start` — request SMS verification; checks whether the phone is registered
- `POST /api/auth/verify` — verify code and create a session
- `POST /api/auth/logout`
- `GET/POST /api/profile`
- `GET/POST /api/contacts`, `DELETE /api/contacts/:id`
- `POST /api/location`
- `POST /api/sos`
- `GET /api/history`
- `GET /api/admin/overview` — admin-only aggregate of users, contacts, latest locations, and SOS events (requires `X-Admin-Key` matching `ADMIN_API_KEY`)

## Before production

This is a starter implementation, not a certified emergency-response service. Add HTTPS and a reverse proxy, request throttling and abuse detection, session expiration/rotation, CSRF protections if using cookies, stronger database access controls, encrypted backups, audit logs, retention/deletion controls, provider delivery receipts, and security testing. Do not expose the C++ API directly to the public internet without these controls.
