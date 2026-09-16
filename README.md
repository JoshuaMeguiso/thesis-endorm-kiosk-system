# EnDorm Kiosk System

EnDorm is a thesis project for a dormitory payment kiosk. It allows tenants to view their dormitory information, check their statement of account, pay bills, and review their payment history.

This project is a first working version. The code is intentionally simple and still depends on some hardware and services outside this repository.

## What the system can do

### Tenant kiosk features

- Log in with a username and password.
- Log in by tapping a registered RFID card.
- View tenant and room details.
- View the current balance.
- View an unpaid statement of account.
- Pay a statement using cash collected by the kiosk.
- Print or send statement and payment information to the local printer service.
- View previous payment transactions.
- Change the tenant password.
- Register an RFID card for a tenant.
- Log out of the kiosk.

### Server features

The Express server provides API routes for:

- Users and login
- Tenants
- Rooms
- Monthly transactions or statements
- Payment history
- Notification tokens

When a tenant is created, the server also creates a basic user account using the tenant ID. The initial password in the current code is `12345` and should be changed after the first login.

The server calculates monthly charges from the room rate, water charge, meter readings, and the number of people in the room. If a bill is overdue, the current code can add a `200` penalty during room access validation.

## Project structure

```text
endorm-client/     React kiosk interface
endorm-server/     Express and MongoDB API
```

The client uses React Context to keep authentication, tenant, transaction, and payment-history data available to the pages.

## Technologies used

- React 18 and Create React App
- React Router
- Node.js and Express
- MongoDB with Mongoose
- JSON Web Tokens
- bcrypt for password hashing
- Firebase Admin for notification-related code
- Semaphore API for SMS notifications
- A local hardware service for RFID, cash, and printing

## Requirements

Install these before running the project:

- Node.js and npm
- MongoDB, either locally or through MongoDB Atlas
- The local kiosk hardware service, if RFID, cash, or printing will be tested

## Configuration

### Server environment variables

Create a file named `.env` inside `endorm-server/`:

```env
PORT=5000
MONGO_URI=your_mongodb_connection_string
SECRET=your_jwt_secret
SMSAPIKEY=your_semaphore_api_key
```

`SMSAPIKEY` is only needed for the room-access SMS feature. The server currently expects the other values when it starts.

The server also initializes Firebase Admin using `endorm-server/config/privateKey.json`. Do not publish a real Firebase private key in a repository. Use a new development key or move the credentials to environment variables before deploying this project.

### Client configuration

The client currently has this proxy in `endorm-client/package.json`:

```json
"proxy": "https://endorm-server.onrender.com"
```

This means requests such as `/user/login` are sent to the deployed Render server while running the React app. Change the proxy to your local server URL if you want the client to use a locally running API.

Some hardware requests are currently hard-coded to:

```text
http://127.0.0.1:8000
```

The local hardware service is expected to provide:

- `GET /uid` to read an RFID card
- `GET /credits` to read inserted cash
- `POST /send_string` to send data to the printer or kiosk device

Without that service, normal username/password login may work, but RFID login, card registration, cash payment, and printing will not work correctly.

## Running the project

Open two terminal windows.

### 1. Start the server

```bash
cd endorm-server
npm install
npm start
```

The server connects to MongoDB and listens on the port set in `.env`.

### 2. Start the client

```bash
cd endorm-client
npm install
npm start
```

The React application opens at:

```text
http://localhost:3000
```

## Typical tenant workflow

1. A tenant is created through the server or an administration workflow.
2. The system creates a tenant record, updates room occupancy, and creates a user account.
3. The tenant logs in with their tenant ID and password, or taps a registered RFID card.
4. The tenant opens **Profile** to view their information and balance.
5. The tenant opens **Statement** to view the current bill.
6. The tenant selects payment and inserts cash into the kiosk.
7. The system updates the transaction and payment history when the minimum payment amount is reached.
8. The kiosk sends formatted information to the local printer service.
9. The tenant can view completed payments under **Transaction**.

## Main API routes

The API is mounted with these route prefixes:

| Prefix         | Purpose                                                                   |
| -------------- | ------------------------------------------------------------------------- |
| `/user`        | Login, signup, password changes, RFID registration, and room access login |
| `/tenant`      | Tenant records                                                            |
| `/room`        | Room records and occupancy                                                |
| `/transaction` | Statements, bill calculations, and payment updates                        |
| `/payment`     | Payment history                                                           |
| `/token`       | Notification token storage                                                |

The server currently does not include a separate authentication middleware for protecting every API route. JWTs are created during login, but the client mainly keeps the returned user data in browser local storage. This is acceptable for an early thesis prototype, but it should be improved before production use.

## Available scripts

### Client

From `endorm-client/`:

```bash
npm start       # Start the React development server
npm test        # Run the Create React App test runner
npm run build   # Create a production build
```

### Server

From `endorm-server/`:

```bash
npm start       # Start the Express server
npm test        # Placeholder command; no server tests are configured yet
```

## Current limitations

- Hardware integration requires a separate local service on port `8000`.
- The client is configured to use a deployed API by default.
- Server tests have not been added yet.
- Authentication and authorization are still basic.
- The initial tenant password is hard-coded as `12345` when a tenant is created.
- Firebase and SMS integrations require valid external credentials.
- Some older controller functions and comments still need cleanup.

## Future improvements

- Move all secrets and private credentials out of tracked files.
- Add authentication middleware and validate JWTs on protected routes.
- Add admin pages for managing tenants, rooms, and monthly readings.
- Add automated client and server tests.
- Improve error handling when MongoDB, the hardware service, or external APIs are unavailable.
- Replace hard-coded hardware URLs and initial passwords with configuration values.

## License

No license has been defined for this thesis project yet.
