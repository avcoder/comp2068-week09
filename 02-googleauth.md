# Download comp2068-week08-finished

`git clone https://github.com/avcoder/comp2068-lab5`
`npm i`
Copy variables.env from last week
`nodemon`

# Google authentication

1.  `npm i passport-google-oauth`
1.  Download image in Blackboard lesson 9 of Google login button and save it in /images/google.png
1.  Change login.ejs to include google button
    ```html
    <a href="/google">
        <img src="./images/google.png" alt="Sign in with Google" style="border-radius: 10px; margin: 0 auto; padding: 1rem">
    </a>
    ```

We've installed the passport-google package. We'll now need to configure this package in app.js, then we'll need to get a GET method at /google that will invoke the passport-google package and will pop in a google login box for us. So we'll need 2 methods in our controller, one to display the login, and one as a callback. So we'll have to goto google developer portal and setup an app to get secret key etc.

## Config app.js

1.  In app.js

    ```js
    const googleStrategy = require('passport-google-oauth').OAuth2Strategy;
    ```

    We now need to configure the google strategy which will involve 3 global values: app id, secret, callback url
    Q. Where should we store those variables
    A. variables.env

1.  In app.js between `passport.use(...)` and `passport.serializeUser(...)`

    ```js
    passport.use(
      new google({
        clientId: process.env.GOOGLE_APP_ID,
        clientSecret: process.env.GOOGLE_APP_SECRET,
        callbackUrl: process.env.GOOGLE_CALLBACK_URL
      })
    );
    ```

1. Open up variables.env and let's begin to set that up
  ```
  GOOGLE_CLIENT_ID=  
  GOOGLE_CLIENT_SECRET=
  GOOGLE_CALLBACK_URL=http://localhost:3000/google/callback
  ```

    We have to get above values from google

1.  Google: google developers console (or goto https://console.developers.google.com/project/_/apiui/apis/library )
1.  Create new project: comp2068b
1.  Select a project > comp2068b
1.  Dashboard > Enable APIs and get keys
1.  Google+ > Enable
1.  click Create Credentials > Calling it from webserver (node) > User Data > Click 'What credentials do i need'
1.  optionally change name: comp2068b Web client 1
1.  Authorized redirect Uris > http://localhost:3000/google/callback (or your live azure/heroku url later) > Create client ID
1.  optionally change Product name: comp2068b retrogames > click Continue
1.  Copy/paste appID into variables.env > click Done
1.  Copy/paste secretID via click Edit

## back in app.js

1.  Copy code in npm.im/passport-google-oauth2 and paste above `passport.serialize(...)`

    ```js
    passport.use(
      new GoogleStrategy(
        {
          clientID: process.env.GOOGLE_CLIENT_ID,
          clientSecret: process.env.GOOGLE_CLIENT_SECRET,
          callbackURL: process.env.GOOGLE_CALLBACK_URL
        },
        (request, accessToken, refreshToken, profile, done) => {
          User.findOrCreate(
            { username: profile.emails[0].value },
            (err, user) => done(err, user)
          );
        }
      )
    );
    ```

## in index.js

1.  Copy code in npm.im/passport-google-oauth2 and paste in index.js

    ```js
    app.get(
      '/auth/google',
      passport.authenticate('google', {
        scope: [
          'https://www.googleapis.com/auth/plus.login',
          ,
          'https://www.googleapis.com/auth/plus.profile.emails.read'
        ]
      })
    );

    app.get(
      '/auth/google/callback',
      passport.authenticate('google', {
        successRedirect: '/auth/google/success',
        failureRedirect: '/auth/google/failure'
      })
    );
    ```

    which we'll change index.js into:

    ```js
    router.get('/google', authController.googlePre);
    router.get('/google/callback', authController.googlePost);
    ```

    In authController.js

    ```js
    exports.googlePre = passport.authenticate('google', {
      scope: [
        'https://www.googleapis.com/auth/plus.login',
        'https://www.googleapis.com/auth/plus.profile.emails.read'
      ]
    });

    exports.googlePost = passport.authenticate('google', {
      successRedirect: '/admin',
      failureRedirect: '/login'
    });
    ```

    - If you run it you get an error re: findOrCreate

1.  `npm i mongoose-findorcreate
1.  In models/User.js

    ```js
    /* findOrCreate lets us search. Then immediately insert if not found. */
    const findOrCreate = require('mongoose-findorcreate');
    ...
    userSchema.plugin(findOrCreate);
    ```

# Exercise take 15-20 minutes and try adding passport-github. Button image is on Blackboard.

1.  `npm i passport-github`
1.  copy code on npm.im/passport-github and paste in app.js

    ```js
    const GitHubStrategy = require('passport-github').Strategy;

    passport.use(
      new GitHubStrategy(
        {
          clientID: GITHUB_CLIENT_ID,
          clientSecret: GITHUB_CLIENT_SECRET,
          callbackURL: 'http://127.0.0.1:3000/auth/github/callback'
        },
        function(accessToken, refreshToken, profile, cb) {
          User.findOrCreate({ githubId: profile.id }, function(err, user) {
            return cb(err, user);
          });
        }
      )
    );
    ```

    but change it to:

    ```js
    router.get('/github', authController.githubPre);
    router.get('/github/callback', authController.githubPost);
    ```

    then change authController.js:

    ```js
    exports.githubPre = passport.authenticate('github');

    exports.githubPost = passport.authenticate(
      'github',
      { failureRedirect: '/login' },
      (req, res) => {
        // Successful authentication, redirect
        res.redirect('/admin');
      }
    );
    ```

## developer.github.com

1.  goto: https://developer.github.com
1.  Obey instructions found here: https://developer.github.com/apps/building-github-apps/creating-a-github-app/ but click 'New OAuth App' instead of `New Github App`
