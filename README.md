To use JSON Web Token (JWT) authentication with Passport.js and Node.js, you can follow these general steps:

- Install the necessary packages:
```
npm install passport passport-jwt jsonwebtoken
```

- Create a user model with fields such as username, password, and email. You can use a library like Mongoose to define the schema and interact with a MongoDB database.

- Create a new strategy for Passport.js that uses JWT. This strategy will check for a valid JWT in the Authorization header of incoming requests, and decode it to get the user's ID.

- In the route handling function, use passport.authenticate('jwt', { session: false }) to authenticate the user.

- In the route handling function, use req.user to access the user's ID and other information.

- In the login function, use the jsonwebtoken package to create a JWT with the user's ID as the payload.

Here is some sample code to demonstrate the above steps:
```javascript
const passport = require('passport');
const JwtStrategy = require('passport-jwt').Strategy;
const ExtractJwt = require('passport-jwt').ExtractJwt;
const jwt = require('jsonwebtoken');
const mongoose = require('mongoose');

const User = mongoose.model('users', new mongoose.Schema({
  username: String,
  password: String,
  email: String
}));

const jwtOptions = {
  jwtFromRequest: ExtractJwt.fromHeader('authorization'),
  secretOrKey: 'secret'
};

const jwtAuth = new JwtStrategy(jwtOptions, (payload, done) => {
  User.findById(payload.sub)
    .then(user => {
      if (user) {
        done(null, user);
      } else {
        done(null, false);
      }
    })
    .catch(err => {
      done(err, false);
    });
});

passport.use(jwtAuth);

app.post('/login', (req, res, next) => {
  passport.authenticate('local', { session: false }, (err, user, info) => {
    if (err || !user) {
      return res.status(400).json({
        message: 'Something is not right',
        user: user
      });
    }
    req.login(user, { session: false }, (err) => {
      if (err) {
        res.send(err);
      }
      // generate a signed json web token with the contents of user object and return it in the response
      const token = jwt.sign(user.toJSON(), 'secret');
      return res.json({ user, token });
    });
  })(req, res);
});

app.get('/profile', passport.authenticate('jwt', { session: false }), (req, res) => {
  res.send(req.user);
});
```
Please note that the above example is just a sample and should not be used in production without additional security checks.
