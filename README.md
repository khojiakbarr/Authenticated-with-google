
# Google OAuth Authentication Example

This project demonstrates how to use Google OAuth2.0 to authenticate users in a Node.js application using Passport.js.

## Prerequisites

Before you begin, ensure you have the following installed on your local machine:
- **Node.js** (https://nodejs.org/)
- **npm** (comes with Node.js)

You will also need to create a Google Cloud project and obtain the following credentials:
- **Google Client ID**
- **Google Client Secret**

Follow these steps to set up the project.

## 1. Clone the Repository

```bash
git clone <repository-url>
cd <repository-folder>
```

## 2. Install Dependencies

```bash
npm install
```

## 3. Create a Google Cloud Project and Get OAuth Credentials

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Create a new project.
3. Enable the **Google+ API** or **Google Identity API**.
4. Configure the OAuth consent screen.
5. Create OAuth 2.0 credentials (Client ID and Client Secret).
6. Set the authorized redirect URL to:
   ```
   http://localhost:4000/google/callback
   ```

Once complete, note down your **Client ID** and **Client Secret**.

## 4. Configure Environment Variables

Create a `.env` file in the root of the project and add the following:

```env
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
```

Replace `your-google-client-id` and `your-google-client-secret` with the values from the Google Cloud Console.

## 5. Project Structure

```
project-folder/
  |-- auth.js
  |-- server.js
  |-- .env
  |-- package.json
  |-- node_modules/
```

- **auth.js**: Contains the Passport.js strategy configuration for Google authentication.
- **server.js**: The main application file that starts the Express server and defines routes.

## 6. How It Works

1. **Google Authentication Flow**
   - The user clicks the login link on the homepage (`/`).
   - The user is redirected to the Google login page.
   - Upon successful login, Google redirects the user to `/google/callback`.
   - If authentication is successful, the user is redirected to the `/protected` route where their profile information is displayed.

2. **Session Management**
   - The app uses `express-session` to maintain user sessions.
   - Passport serializes the user to store it in the session and deserializes it on subsequent requests.

## 7. Usage

1. Run the application:

```bash
node server.js
```

2. Open your browser and go to:

```
http://localhost:4000/
```

3. Click on **Authenticate with Google**.

4. Log in with your Google account.

5. If successful, you'll be redirected to the **/protected** route, and you should see a message that says:

```
Hello! <Your Google Display Name>
```

6. To log out, visit:

```
http://localhost:4000/logout
```

## 8. File Descriptions

### **server.js**

```javascript
const express = require("express");
const session = require("express-session");
const passport = require("passport");
require("./auth");

function isLoggedIn(req, res, next) {
  req.user ? next() : res.sendStatus(401);
}

const app = express();
app.use(session({ secret: "cats", resave: false, saveUninitialized: true }));
app.use(passport.initialize());
app.use(passport.session());

app.get("/", (req, res) => {
  res.send("<a href='/auth/google'> Authenticate with Google</a>");
});

app.get(
  "/auth/google",
  passport.authenticate("google", { scope: ["email", "profile"] })
);

app.get(
  "/google/callback",
  passport.authenticate("google", {
    successRedirect: "/protected",
    failureRedirect: "/auth/google/failure",
  })
);

app.get("/protected", isLoggedIn, (req, res) => {
  res.send("Hello! " + req.user.displayName);
});

app.get("/logout", (req, res) => {
  req.logout();
  req.session.destroy();
  res.send("Goodbye!");
});

app.get("/auth/google/failure", (req, res) => {
  res.send("Failed to authenticate");
});

app.listen(4000, () => console.log("listening on", 4000));
```

### **auth.js**

```javascript
const passport = require("passport");
const GoogleStrategy = require("passport-google-oauth2").Strategy;
require("dotenv").config();

passport.use(
  new GoogleStrategy(
    {
      clientID: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
      callbackURL: "http://localhost:4000/google/callback",
      passReqToCallback: true,
    },
    function (request, accessToken, refreshToken, profile, done) {
      console.log(profile);
      return done(null, profile);
    }
  )
);

passport.serializeUser(function (user, done) {
  done(null, user);
});

passport.deserializeUser(function (user, done) {
  done(null, user);
});
```

## 9. Endpoints

| Endpoint              | Method | Description                    |
|---------------------|--------|----------------------------------|
| `/`                   | GET    | Home page with Google login link |
| `/auth/google`        | GET    | Initiates Google OAuth login     |
| `/google/callback`    | GET    | Callback URL after Google login  |
| `/protected`          | GET    | Protected route, requires login  |
| `/logout`             | GET    | Logs out and ends the session    |
| `/auth/google/failure`| GET    | Failed authentication message    |

## 10. Notes
- Be sure to update the `.env` file with your Google credentials.
- Use **http://localhost:4000/google/callback** as the redirect URL for Google OAuth.
- If you encounter any issues, check if the redirect URL in Google Cloud Console matches the one in `auth.js`.

## 11. License
This project is open-source and free to use under the MIT License.

---
With this guide, you should be able to set up, run, and understand the basics of Google OAuth2.0 authentication in a Node.js application. If you encounter any issues, feel free to ask for help!

