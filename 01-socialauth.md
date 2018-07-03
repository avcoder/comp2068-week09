# Authenticating via FB

Q. Why would you want to login via fb or other social platform?
A. Don't have to remember another set of credentials. Also, we as developers now can target users on fb via ads for example.

So last week we explored using passport, as well as passport-local-mongoose.
This week let's goto npmjs.com and search passport-facebook -- let's use same developer.

Notice it will ask us to:

- install passport-facebook
- Under usage, notice we need to create a facebook application which will be issued an app ID and app secret which will be provided to our strategy.
- For example, at latimes.com, you can login with facebook. The facebook modal window is asking permission 'are we going to allow the LA times fb application to access our fb profile? Notice their fb app has an icon and a friendly name 'LA Times'. It tells us what parts of our fb account this fb application has access to. We've all seen this, now we need to develop one. We don't have to create code to do this, just a bunch of cliking and in return we get an app id and secret.
- Once we do that, we then have to configure in app.js, that id and secret in our configuration.
- We also have to provide a callback URL. After a user signs in with fb, what URL should we send the user back to?
- Then notice this anonymous function, we use findOrCreate() method. Currently, our Game model doesn't have this method, but we'll add it. This is like SELECT or INSERT if it can't find it, all in one method.
- Then we'll need these get requests (see under Authenticate Requests). The first app.get() pops up the fb login box asks the if the application can access their fb profile. If allowed, then it will show the fb login screen.
  The 2nd app.get() is our callback. So after the user logs in to fb and comes back, we tell app where to go.

So mostly we'll copy/paste this code and change a few things.

## Install passport-facebook

1.  `npm i passport-facebook` (notice it uses oauth2.0)

## developers.facebook.com

1.  goto: developers.facebook.com (need to login to fb to use it)
1.  Once logged in, top right corner click 'My Apps' > 'Add a New App'
1.  We need a display name which shows up in the fb modal (ex: LA Times)
1.  Email is same email associated with fb account
1.  Category can be anything (Education)
1.  Click `Create App ID`

You'll see a new screen with all kinds of options. What kind of app do you want to create?

1.  Click `Get Started` for "Facebook Login"
1.  By default it enables Client and Web OAuth which is what we want.
1.  Click Dashboard and you'll see our app id and secret
1.  Click Roles - it should show your fb account as an admin. So, You will be able to login to the app with your fb login, but by default, no one will be able to.
1.  Click "App Review" > "Yes" to make your app public. which is probably safe in our case because we have a register page anyway where anyone can sign up for an account. By making it public, by default, those are the things under Approved Items that will ask users to permit
1.  We can specify under App Domains what domains are allowed to use it. Enter http://localhost:3000
1.  Under Facebook Login > Settings > Valid OAuth redirect URIs enter: http://localhost:3000/auth/facebook/callback

So now we need to grab the app id and secret and add that in our variables.env

## Add id/secret to variables.env

In variables.env

```
FACEBOOK_APP_ID=...   (grab this from developers.facebook.com > Dashboard)
FACEBOOK_APP_SECRET=... (grab this from developers.facebook.com > Dashboard)
FACEBOOK_CALLBACK_URL=http://localhost:3000/facebook/callback
```

## Configure app.js

So now we need to link to the passport-facebook package, we need to setup the facebook strategy which uses our secret values, then we need to setup a function when a user tries to authenticate w facebook, we need to use our User model to see if they exist, and if they haven't logged in before, they can create a new account.

1.  Copy/paste the code on npm.im/passport-facebook

In app.js, after `passport.use(User.createStrategy())` but before `passport.serializeUser(...)` because the serialize/deserialize manages user sessions, which we want to happen facebook auth.

In app.js:

```js
const FacebookStrategy = require('passport-facebook').Strategy;

passport.use(
  new FacebookStrategy(
    {
      clientID: process.env.FACEBOOK_APP_ID,
      clientSecret: process.env.FACEBOOK_APP_SECRET,
      callbackURL: process.env.FACEBOOK_CALLBACK_URL,
      profileFields: ['id', 'displayName', 'emails']
    },
    function(accessToken, refreshToken, profile, cb) {
      User.findOrCreate(
        {
          facebookId: profile.id,
          username: profile.emails[0].value
        },
        function(err, user) {
          return cb(err, user);
        }
      );
    }
  )
);
```

### facebookId: profile.id

Notice above, that findOrCreate is trying to look up our mongodb for a field called facebookId which we don't have. So let's put it in our User model.

In models/User.js, let's modify our userSchema. Remember last week, it was blank since mongoose automatically assumes username/pwd. Now we'll add facebookId:

```js
const userSchame = new mongoose.Schema({
  facebookId: String
});
```

### username: profile.emails[0].value

Notice also that username is using the first email that facebook stores in their array.

### profileFields: [... 'emails']

Now by default, fb will give us their id and display name, but it doesn't automatically give us email addresses. So to do this, we add profileFields. So right now, there are no users in our db w a fb id. But when a user logs in via fb, fb will return to us, an id, displayname, list of email addresses. So then we're going to use our User model to test 'Do we have a user with that fb id in our db already?' If yes, then we'll look up that user, and run the callback. If no, it will create one - so we're adding that user to our mongodb and we're going to store their fb id, their username will be their email address from fb. So for any local users, there isn't a fb id. But the 2nd time that user logs in, will pass in the fb id, and hopefully will find that user and won't create them again.

findOrCreate is actually part of passport-facebook, not mongoose, so right now our application cannot run findOrCreate.
Now we could rewrite this so that if find finds nothing, then do create. but instead we can include another package that includes that fn.

### npm i mongoose-findorcreate

1.  `npm i mongoose-findorcreate`
1.  In models/User.js

    ```js
    /* findOrCreate lets us search. Then immediately insert if not found. */
    const findOrCreate = require('mongoose-findorcreate');
    ...
    userSchema.plugin(findOrCreate);
    ```

1.  Download in Blackboard, the image of the button for fb login
1.  In views login.ejs, underneath our login button

        ```html
        <a href="/auth/facebook"><img src="/images/facebook.png"></a>
        ```

    Now if we click the fb button, there's an error because there's no route yet. Also need to include in error.ejs:

    ```js
    res.render('error', {
        .
      user.req.user
    });
    ```

1.  In routes/index.js

    ```js
    /* GET /facebook - show FB login popup */
    app.get(
      '/auth/facebook',
      passport.authenticate('facebook', {
        scope: 'email'
      })
    );

    /* GET /facebook/callback - after fb login attemp, decide where to go */
    app.get(
      '/auth/facebook/callback',
      passport.authenticate('facebook', {
        failureRedirect: '/login',
        scope: 'email'
      }),
      function(req, res) {
        /* Successful authentication, redirect to admin */
        res.redirect('/admin');
      }
    );
    ```

1.  Check inside mlab to see what's in there

# Authenticating via Google

1.  `npm i passport-google-oauth2`
1.
