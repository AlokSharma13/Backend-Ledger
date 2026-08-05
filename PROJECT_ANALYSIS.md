# Backend Ledger Project Analysis

## Project Overview

### What type of project is this?
This is a **Node.js backend API project** built with **Express**. It is not a full-stack app; there is no frontend in this repository.

### Technologies used
- `Node.js`
- `Express`
- `MongoDB` via `Mongoose`
- `JWT` authentication with `jsonwebtoken`
- `bcryptjs` for password hashing
- `cookie-parser` for cookie handling
- `dotenv` for environment variables
- `nodemailer` for email notifications

### Main purpose
The project is a **ledger service** for user accounts and transactions:
- register and login users
- create accounts
- transfer funds between accounts
- track ledger entries
- protect routes with authentication
- send email notifications

---

## How the application works from start to finish

### Startup flow
1. `server.js` runs when you start the app.
2. It loads environment variables with `dotenv`.
3. It imports `src/app.js`.
4. It imports `src/config/db.js` and connects to MongoDB.
5. It starts the Express server on port `3000`.
6. Incoming HTTP requests go through `src/app.js`.
7. Routes dispatch requests to controllers.
8. Controllers perform logic and use models to read/write MongoDB.
9. Responses are sent back to the client.

### Full execution flow
```
Node process starts
↓
server.js loads
↓
dotenv config loads environment variables
↓
connectToDB() connects to MongoDB
↓
app is created from src/app.js
↓
Express middleware registered
↓
Route handlers registered
↓
HTTP request arrives
↓
Route matching selects router
↓
Middleware (auth, cookies, JSON parsing) runs
↓
Controller handles request
↓
Controller uses models and services
↓
Database queries/updates happen
↓
Email service may send notifications
↓
Response is returned
```

---

## Folder structure

### root
- `package.json`
  - stores dependencies and scripts
- `server.js`
  - application entry point

### src/
This is the application source code.

### src/config/
- `db.js`
- Purpose: database connection logic
- Used by: `server.js`
- Depends on: environment variables

### src/controllers/
- `auth.controller.js`
- `account.controller.js`
- `transaction.controller.js`
- Purpose: request handling logic
- Used by: route definitions
- Depends on: models, services

### src/middleware/
- `auth.middleware.js`
- Purpose: protect routes with JWT and blacklist checks
- Used by: account and transaction routes

### src/models/
- `user.model.js`
- `account.model.js`
- `ledger.model.js`
- `transaction.model.js`
- `blackList.model.js`
- Purpose: define MongoDB schemas and data logic
- Used by: controllers and middleware

### src/routes/
- `auth.routes.js`
- `account.routes.js`
- `transaction.routes.js`
- Purpose: define API endpoints
- Used by: `src/app.js`

### src/services/
- `email.service.js`
- Purpose: send emails via Gmail OAuth2
- Used by: auth and transaction controllers

---

## File-by-file explanation

### `package.json`
- Purpose: declares dependencies and scripts
- When executed: by npm/yarn, not at runtime
- Imports: none
- If missing: project cannot install dependencies or run via npm scripts

Key lines:
- `"dev": "npx nodemon server.js"` : development command automatically restarts on changes
- `"start": "node server.js"` : production/start command

---

### `server.js`
- Purpose: entry point for the server
- When executed: when `npm start` or `npm run dev` runs
- Imports:
  - `dotenv` → loads `.env`
  - `./src/app` → Express app
  - `./src/config/db` → DB connector
- Execution flow:
  - `require("dotenv").config()` reads `.env` into `process.env`
  - `const app = require("./src/app")` loads Express setup
  - `connectToDB()` connects MongoDB
  - `app.listen(3000)` starts server
- If missing: app would not start

Summary:
- `server.js` boots the app
- It connects the database first, then listens for requests

Interview Questions:
- beginner: What does `require("dotenv").config()` do?
- intermediate: Why do we separate `server.js` and `src/app.js`?
- advanced: What could happen if the DB connection is delayed after `app.listen()`?

---

### `src/app.js`
- Purpose: configure Express middleware and route mounting
- When executed: imported by `server.js`
- Imports:
  - `express`
  - `cookie-parser`
  - route modules
- Execution flow:
  - creates Express app
  - registers `express.json()` to parse JSON bodies
  - registers `cookieParser()` to parse cookies
  - defines root GET `/`
  - mounts auth, account, and transaction routers
- If missing: server would have no routes or middleware

Important imports:
- `express`: framework for HTTP routes
- `cookie-parser`: turns cookie header into `req.cookies`
- routers: provides API endpoints for auth, accounts, transactions

Summary:
- `src/app.js` is the main routing and middleware setup file

Interview Questions:
- beginner: What does `app.use(express.json())` do?
- intermediate: Why is cookie parser applied globally?
- advanced: How would you add rate limiting middleware here?

---

### `src/config/db.js`
- Purpose: connect to MongoDB
- When executed: by `server.js`
- Imports:
  - `mongoose`
- Function:
  - `connectToDB()` calls `mongoose.connect(process.env.MONGO_URI)`
  - logs success or failure
- If missing: app could not connect to DB

Important note:
- No retry logic is implemented
- app exits on failure

Summary:
- This file isolates DB connection logic from server startup

Interview Questions:
- beginner: What does `mongoose.connect()` return?
- intermediate: Why use `process.env.MONGO_URI` instead of hard-coded URI?
- advanced: How would you add connection event handlers?

---

### `src/routes/auth.routes.js`
- Purpose: auth-related endpoint definitions
- When executed: when app imports router
- Imports:
  - `express`
  - `../controllers/auth.controller`
- Routes:
  - `POST /api/auth/register` → `userRegisterController`
  - `POST /api/auth/login` → `userLoginController`
  - `POST /api/auth/logout` → `userLogoutController`
- If missing: auth endpoints would not exist

Summary:
- Defines auth API contract and sends requests to controller functions

Interview Questions:
- beginner: What is `express.Router()` used for?
- intermediate: How does router route paths get mounted in `app.js`?
- advanced: What if we need versioned API routes like `/api/v1/auth`?

---

### `src/routes/account.routes.js`
- Purpose: account-related endpoint definitions
- Imports:
  - `express`
  - `../middleware/auth.middleware`
  - `../controllers/account.controller`
- Routes:
  - `POST /api/accounts/` protected by `authMiddleware`
  - `GET /api/accounts/` protected by `authMiddleware`
  - `GET /api/accounts/balance/:accountId` protected by `authMiddleware`
- If missing: account endpoints unavailable

Summary:
- Protects account routes and connects them to account controller logic

Interview Questions:
- beginner: Why is `authMiddleware` used here?
- intermediate: What is the route parameter `:accountId` used for?
- advanced: How could you add authorization to block non-owners from viewing balance?

---

### `src/routes/transaction.routes.js`
- Purpose: transaction API endpoints
- Imports:
  - `express`
  - `authMiddleware`
  - `../controllers/transaction.controller`
- Routes:
  - `POST /api/transactions/` protected by `authMiddleware`
  - `POST /api/transactions/system/initial-funds` protected by `authSystemUserMiddleware`
- If missing: transaction endpoints unavailable

Summary:
- Defines two transaction routes, one normal and one system-only

Interview Questions:
- beginner: What does `/system/initial-funds` mean?
- intermediate: Why use a separate middleware for system users?
- advanced: How would you design access control for additional transaction types?

---

## Controllers

### `src/controllers/auth.controller.js`
This file contains three controllers:
- `userRegisterController`
- `userLoginController`
- `userLogoutController`

#### `userRegisterController`
Execution flow:
1. reads `email`, `password`, `name` from `req.body`
2. checks if a user with that email exists
3. if exists → returns 422
4. otherwise creates user
5. signs JWT token for user
6. sets cookie `token`
7. returns user info + token
8. sends registration email

Important lines:
- `userModel.findOne({ email })` checks duplicate email
- `userModel.create({ email, password, name })` saves user
- `jwt.sign({ userId: user._id }, process.env.JWT_SECRET, { expiresIn: "3d" })`
- `res.cookie("token", token)` stores token in browser cookie

If missing: users cannot register

#### `userLoginController`
Execution flow:
1. reads `email`, `password`
2. fetches user with `password` field included
3. returns 401 if user not found
4. compares password using `comparePassword()`
5. returns 401 if invalid
6. signs JWT token
7. sets cookie
8. returns user info + token

Important:
- `.select("+password")` is required because password is excluded by default
- `user.comparePassword(password)` uses bcrypt compare

#### `userLogoutController`
Execution flow:
1. reads token from cookie or Authorization header
2. if no token → respond success
3. stores token in blacklist collection
4. clears cookie
5. returns success

Important:
- logout does not revoke JWT on the server except by blacklisting
- blacklisting uses `tokenBlackListModel.create({ token })`

Summary:
- Handles registration, authentication, logout, and email sending
- Connects with `user.model.js`, JWT, and token blacklist

Interview Questions:
- beginner: Why do we send a JWT after login?
- intermediate: Why is password excluded by default in the user schema?
- advanced: What are the pros and cons of token blacklisting for logout?

---

### `src/controllers/account.controller.js`
Functions:
- `createAccountController`
- `getUserAccountsController`
- `getAccountBalanceController`

#### `createAccountController`
- reads authenticated user from `req.user`
- creates an account linked to `user._id`
- responds with account object

#### `getUserAccountsController`
- queries accounts where `user` equals authenticated user
- returns list of accounts

#### `getAccountBalanceController`
- reads `accountId` from route params
- fetches account owned by authenticated user
- returns 404 if no account
- calls `account.getBalance()` to compute balance
- returns balance

Summary:
- Simple CRUD-style account operations
- Depends on `req.user` from auth middleware
- Uses `account.model.js` methods

Interview Questions:
- beginner: What does `accountModel.create()` do?
- intermediate: Why is `req.user._id` used instead of `req.user`?
- advanced: How might you cache account balances to reduce aggregation cost?

---

### `src/controllers/transaction.controller.js`
This file has two operations:
- `createTransaction`
- `createInitialFundsTransaction`

#### `createTransaction`
This is the most complex logic in the app.

Steps:
1. validate request fields
2. fetch from/to accounts
3. verify accounts exist
4. check idempotency key
5. verify account statuses are `ACTIVE`
6. compute sender balance using ledger aggregation
7. begin MongoDB transaction
8. create transaction document with status `PENDING`
9. create debit ledger entry
10. wait 15 seconds
11. create credit ledger entry
12. update transaction to `COMPLETED`
13. commit session
14. send transaction email
15. return result

Important concepts:
- `idempotencyKey`: prevents duplicate processing
- `mongoose.startSession()` and `session.startTransaction()`: ensure atomic operations
- ledger entries record `DEBIT` and `CREDIT`
- `await (() => { return new Promise((resolve) => setTimeout(resolve, 15 * 1000)); })()` introduces a 15 second delay
- if any failure occurs, the catch block returns a pending error

Weakness:
- the catch block does not rollback the transaction explicitly
- no `await session.abortTransaction()` on failure
- no `session.endSession()` in the error path
- the 15 second delay is unusual and likely for demonstration/test only

#### `createInitialFundsTransaction`
Flow:
1. validate required fields
2. verify destination account exists
3. find a system user account for the authenticated system user
4. start MongoDB transaction
5. create transaction with `PENDING`
6. create debit and credit ledger entries
7. mark transaction `COMPLETED`
8. save and commit
9. return result

Important:
- `authSystemUserMiddleware` is required to ensure only a system user can call this route

Summary:
- Transaction controller is the core financial logic
- Uses ledger and transaction models to maintain account balances
- Depends on MongoDB transactions for atomicity

Interview Questions:
- beginner: What is idempotency and why is it important?
- intermediate: Why use MongoDB transactions here?
- advanced: What are the risks of a 15-second artificial delay inside a DB transaction?

---

## Middleware

### `src/middleware/auth.middleware.js`
Contains:
- `authMiddleware`
- `authSystemUserMiddleware`

#### `authMiddleware`
Flow:
1. read token from cookie or `Authorization` header
2. reject if missing
3. reject if blacklisted
4. verify token with JWT secret
5. fetch user from DB by decoded `userId`
6. attach `user` to `req.user`
7. call `next()`

Explanation:
- protects routes by checking authentication
- if verification fails, returns 401
- uses `jwt.verify(token, process.env.JWT_SECRET)`

#### `authSystemUserMiddleware`
Flow:
1. same token handling and blacklist check
2. verify token
3. fetch user and include `systemUser` field
4. if `user.systemUser` is not true, return 403
5. attach user and `next()`

Why this exists:
- to protect system-level transaction route only for special accounts

Summary:
- middleware centralizes authentication and authorization
- controllers rely on `req.user` being available after it runs

Interview Questions:
- beginner: Why is authorization done in middleware instead of controllers?
- intermediate: What is the difference between 401 and 403?
- advanced: How could you improve this middleware to support token refresh?

---

## Models

### `src/models/user.model.js`
Defines user schema and password hashing logic.

Fields:
- `email`
- `name`
- `password`
- `systemUser`

Important:
- `password.select = false` hides it by default
- `systemUser` is immutable and hidden by default
- `timestamps: true` adds `createdAt` and `updatedAt`

Hooks:
- `pre("save", async function () { ... })`
  - if password changed, hashes it with bcrypt
- `methods.comparePassword`
  - compares plain text password with hashed password

Why needed:
- stores users securely
- password hashing prevents storing raw passwords
- model methods encapsulate logic

If missing:
- auth would not work
- password comparison and hashing would be duplicated elsewhere

Interview Questions:
- beginner: Why should passwords be hashed?
- intermediate: What does `this.isModified("password")` do?
- advanced: Why use `select: false` for password and `immutable: true` for systemUser?

---

### `src/models/account.model.js`
Defines account schema and balance computation.

Fields:
- `user`: reference to `user`
- `status`: `ACTIVE`, `FROZEN`, `CLOSED`
- `currency`: default `INR`

Indexes:
- `user`
- compound `{ user: 1, status: 1 }`

Method:
- `getBalance()`
  - aggregates ledger entries for this account
  - computes credit minus debit

Details:
- if no ledger entries, returns 0
- uses `ledgerModel.aggregate`
- balance is derived rather than stored

Why needed:
- account ownership and status
- balance logic centralizes ledger aggregation

Interview Questions:
- beginner: What is a reference field in MongoDB?
- intermediate: Why compute balance with aggregation instead of storing it?
- advanced: What are the trade-offs of balance derivation vs stored balance?

---

### `src/models/ledger.model.js`
Defines ledger entry schema.

Fields:
- `account` reference
- `amount`
- `transaction` reference
- `type`: `CREDIT` or `DEBIT`

Key design:
- ledger entries are immutable after creation
- many pre-hooks throw error on update or delete

Why needed:
- records the exact credits/debits for an account
- immutability protects financial audit trails

Interview Questions:
- beginner: What does `immutable: true` mean?
- intermediate: Why prevent update/delete on ledgers?
- advanced: What kind of audit requirement does this enforce?

---

### `src/models/transaction.model.js`
Defines transaction schema.

Fields:
- `fromAccount`
- `toAccount`
- `status`
- `amount`
- `idempotencyKey`

Important:
- `idempotencyKey` is unique
- transaction status can be `PENDING`, `COMPLETED`, `FAILED`, `REVERSED`

Why needed:
- tracks transfer state
- enables safe retries

Interview Questions:
- beginner: Why is `idempotencyKey` used?
- intermediate: What does transaction status represent?
- advanced: When would `REVERSED` be used and how would you implement reversal?

---

### `src/models/blackList.model.js`
Defines blacklisted tokens.

Fields:
- `token`

Indexes:
- expires documents after 3 days using `expireAfterSeconds`

Why needed:
- support logout by invalidating JWTs for a limited period
- enables token invalidation without changing secret

Interview Questions:
- beginner: What does `expireAfterSeconds` do?
- intermediate: Why blacklist tokens instead of just deleting cookies?
- advanced: What are scalability issues with token blacklist?

---

## Services

### `src/services/email.service.js`
Purpose:
- send registration and transaction email notifications

Imports:
- `nodemailer`

Key setup:
- Gmail OAuth2 transport using environment variables
- `transporter.verify()` logs connectivity status

Functions:
- `sendEmail(to, subject, text, html)`
- `sendRegistrationEmail(userEmail, name)`
- `sendTransactionEmail(userEmail, name, amount, toAccount)`
- `sendTransactionFailureEmail(...)`

Why needed:
- not required for core ledger functionality, but adds notification behavior
- if missing, there would be no email notifications

Interview Questions:
- beginner: What does `nodemailer.createTransport()` do?
- intermediate: Why use OAuth2 in email transport?
- advanced: What happens if email sending fails during transaction creation?

---

## API Summary

### Auth
- `POST /api/auth/register`
  - body: `{ email, password, name }`
  - registers user, returns token
- `POST /api/auth/login`
  - body: `{ email, password }`
  - logs in user, returns token
- `POST /api/auth/logout`
  - body: none
  - blacklists token, clears cookie

### Accounts
- `POST /api/accounts/`
  - protected
  - creates a new account for logged-in user
- `GET /api/accounts/`
  - protected
  - returns all user accounts
- `GET /api/accounts/balance/:accountId`
  - protected
  - returns balance for the account

### Transactions
- `POST /api/transactions/`
  - protected
  - body: `{ fromAccount, toAccount, amount, idempotencyKey }`
  - transfers funds
- `POST /api/transactions/system/initial-funds`
  - protected by system user middleware
  - body: `{ toAccount, amount, idempotencyKey }`
  - credits initial funds

---

## Data flow for a request

### Example: create account
```
Client sends POST /api/accounts/
↓
src/app.js middleware parse JSON and cookies
↓
src/routes/account.routes.js matches route
↓
authMiddleware validates token, attaches req.user
↓
accountController.createAccountController executes
↓
accountModel.create() writes new account to DB
↓
Response sent back with account data
```

### Example: transaction
```
Client sends POST /api/transactions/
↓
app.js middleware parse JSON and cookies
↓
transaction.routes.js matches route
↓
authMiddleware validates token and user
↓
transactionController.createTransaction validates inputs
↓
accountModel finds from/to accounts
↓
account.getBalance() reads ledger entries
↓
MongoDB transaction begins
↓
transactionModel creates PENDING transaction
↓
ledgerModel creates DEBIT record
↓
delay 15s
↓
ledgerModel creates CREDIT record
↓
transaction status updated to COMPLETED
↓
session commits
↓
emailService sends notification
↓
Response sent
```

---

## Authentication flow

### Registration
1. client POST `/api/auth/register`
2. controller checks duplicate email
3. creates user with hashed password
4. creates JWT token
5. sets cookie `token`
6. returns user info and token
7. sends welcome email

### Login
1. client POST `/api/auth/login`
2. controller finds user and includes password
3. compares password with bcrypt
4. creates JWT token
5. sets cookie
6. returns user info and token

### Protected routes
- token read from cookie or Authorization header
- blacklist checked
- JWT verified
- `req.user` assigned
- route proceeds

### Logout
- token read
- token stored in blacklist
- cookie cleared
- response returned

### System user route
- same token verification
- additionally checks `user.systemUser`
- if not system user, returns 403 forbidden

---

## Database flow

### Models
- `user` stores login credentials and role flag
- `account` stores ownership, status, currency
- `ledger` stores immutable debit/credit entries
- `transaction` stores transfer metadata and status
- `tokenBlackList` stores invalidated tokens

### Schema relationships
- `account.user` → `user._id`
- `ledger.account` → `account._id`
- `ledger.transaction` → `transaction._id`
- `transaction.fromAccount` / `toAccount` → `account._id`

### CRUD operations
- Create user/account/transaction/ledger entry
- Read account list and balance
- No update/delete operations for ledger entries
- Transaction update status is the only update

### Aggregation
- `account.getBalance()` uses aggregation to compute:
  - total debit
  - total credit
  - balance = credit - debit

---

## Key file connections

```
server.js
└── src/app.js
    ├── src/routes/auth.routes.js
    │   └── src/controllers/auth.controller.js
    │       ├── src/models/user.model.js
    │       ├── src/models/blackList.model.js
    │       └── src/services/email.service.js
    ├── src/routes/account.routes.js
    │   ├── src/middleware/auth.middleware.js
    │   └── src/controllers/account.controller.js
    │       └── src/models/account.model.js
    └── src/routes/transaction.routes.js
        ├── src/middleware/auth.middleware.js
        └── src/controllers/transaction.controller.js
            ├── src/models/transaction.model.js
            ├── src/models/ledger.model.js
            ├── src/models/account.model.js
            └── src/services/email.service.js
```

---

## Real-life analogies

- **Express Router**: like a shipping clerk who directs letters to the correct department.
- **Middleware**: like airport security checks before you enter the terminal.
- **JWT**: like a signed boarding pass proving your identity.
- **Cookies**: like a card in your wallet that the server reads on every visit.
- **Database model**: like a form template describing what fields a record must contain.
- **Ledger entry**: like a bank transaction slip that cannot be edited later.

---

## Major issues and improvements

### Security / correctness
- `transaction.controller.js` uses a 15 second delay inside active DB transaction.
- Failure path in transactions does not abort or end the session properly.
- `userLogoutController` blacklists tokens but tokens still remain valid until checked elsewhere. Works, but relies on blacklist read on every request.
- No validation middleware or schema validation besides manual checks.
- If `TOKEN` not sent, auth routes fail with generic 401 message.
- `email.service` is synchronous from controller perspective; email send failures are not handled in a way that retries or compensates.

### Performance / scalability
- `account.getBalance()` computes balance with aggregation each time.
- This is fine for small data, but large ledger tables would slow down balance queries.
- `blackList.model` uses unique token; if a token is reused in a retry request, duplicate error may occur.

### Best practices
- Add centralized error handling middleware.
- Add schema validation with `Joi`, `Zod`, or `express-validator`.
- Add explicit transaction abort in catch blocks.
- Add `finally` to end sessions.
- Add a separate `src/services/auth.service.js` if logic grows.
- Add request logging and API docs.

---

## Summary

This project is a **backend ledger API** built with Node, Express, MongoDB, and JWT. It supports:
- user registration and login
- protected account management
- transactions with ledger entries
- email notifications

Key files:
- `server.js` starts the app
- `src/app.js` configures routes and middleware
- `src/config/db.js` connects MongoDB
- `src/routes/*` define endpoints
- `src/controllers/*` handle business logic
- `src/models/*` define database structures
- `src/middleware/auth.middleware.js` secures routes
- `src/services/email.service.js` sends emails

This codebase illustrates a typical backend architecture:
- route → middleware → controller → model → database → response

If you want, I can continue now with a second pass that explains **every line in each file**, one file at a time, starting with `server.js`.
