## Cheatsheet for server

## Terminal

```js

create or touch server.js

npm init -y

npm install express

npm install mongoose dotenv

npm install mongodb

touch .env    // CREATE A .ENV FILE

npm install cookie-parser

npm install jsonwebtoken

npm install bcrypt

touch .gitignore   // CREATE A GITIGNORE FILE

npm install cors // This will install the ability to make cross-origin requests. 

npm install -g nodemon // to automatically restart your node server

nodemon server.js  // TO RUN THE SERVER

```


## .env

```js

PORT=8000
DB=my_db
ATLAS_USERNAME=root
ATLAS_PASSWORD=root

```


## Gitignore

```js

/node_modules
.env


```


## Config.js

```js

const mongoose = require('mongoose');
const dbName = process.env.DB;
const username = process.env.ATLAS_USERNAME;
const pw = process.env.ATLAS_PASSWORD;
// CHANGE YOUR DATABASE
const uri = `mongodb+srv://${username}:${pw}@YOUR_CONNECTION_STRING_HERE/${dbName}?retryWrites=true&w=majority`;
mongoose.connect(uri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
})
    .then(() => console.log("Established a connection to the database"))
    .catch(err => console.log("Something went wrong when connecting to the database", err));



```




## Model

```js

// for your model

const mongoose = require('mongoose');
 
const UserSchema = new mongoose.Schema({
    name: {
        type: String
        // required: [true, "{PATH} is required"],                           ( FOR VALIDATIONS )
        // minLength: [3, "{PATH} must have at least 3 characters"]
    },
    age: {
        type: Number
        // required: [true, "{PATH} is required"],                           ( FOR VALIDATIONS )
        // minLength: [13, "{PATH} must have at be atleast 13 years old"]
    }
}, { timestamps: true });
 
const User = mongoose.model('User', UserSchema);
 
module.exports = User;

```



## server.js

```js

const express = require('express');
const cors = require('cors');
const cookies = require('cookie-parser')
const jwt = require("jsonwebtoken")
const secret = "the secret key"

module.exports.secret = secret
const app = express();
require('dotenv').config();
const port = process.env.PORT;
require('./config/mongoose.config'); // This is new
app.use(cors({
    origin: "http://localhost:3000",
    credentials: true
}));
app.use(express.json()); // This is new
app.use(express.urlencoded({ extended: true })); // This is new
app.use(cookies())
require('./routes/post.routes')(app);

app.use('/user/check_token', (req, res) => {
    const token = req.header('x-auth');
    if (!token || token === '' || token === 'undefined') {
        res.status(400)
        console.log('invalid');
        return;
    }
    //parse token.
    const data = jwt.decode(token);
    //find user by id.
    if (data) {
        User.findById(data.uid).then((userData) => {
            //check is the token exist in the tokens array.
            if (userData.tokens.indexOf(token) !== -1) {
                console.log("success");
            } else {
                res.status(400)
                console.log("invalid");
            }
        }).catch((e) => {
            res.status(400)
            console.log('invalid');
        });
    } else {
        res.status(400)
        console.log('invalid');
    }
});

module.exports.authenticate = (req, res, next) =>{
    jwt.verify(req.cookies.usertoken, secret, (err, payload) => {
        if (err) {
            res.status(400).json({verified: false})
        }else{
            next()
        }
    })
}


app.listen(port, () => console.log(`🎤 Listening on port: ${port}`));

```




## Routes



```js


const PostController = require('../controllers/post.controller');
const Users = require("../controllers/user.controller")

module.exports = app => {
    
    // ROUTES FOR POSTS
    app.get('/api/posts', PostController.findAllPosts);
    app.get('/api/posts/:id', PostController.findOneSinglePost);
    app.get('/api/posts/category/:category', PostController.findAllPostsByCategory);
    app.patch('/api/posts/:id', PostController.updateExistingPost);
    app.post('/api/posts', PostController.createNewPost);
    app.delete('/api/posts/:id', PostController.deleteAnExistingPost);
    app.post('/api/posts/like/:id', PostController.likeThePost);

    // for like and unlike funcion (not being used)
    app.put('/api/post/like', PostController.findByIdAndUpdateLike);
    app.put('/api/post/unlike', PostController.findByIdAndUpdateUnLike);

    // show all the posts belonging to a user given a user id
    app.get("/api/posts/user/:userid", PostController.findAllPostsByUser)

    // ROUTES FOR USERS
    app.post("/api/register", Users.register)
    app.get("/api/users", Users.getuser)
    // app.post("/api/signup", Users.signup)
    app.post("/api/login", Users.login)
    app.post("/api/user/posts", Users.getPostsByUser)
    app.get("/api/user/:id", Users.getUserById)
    app.get("/api/one/user/:id", Users.findOneSingleUser)
    app.patch('/api/users/:id', Users.updateExistingUser);

    //app.get("/api/users", Users.findAllUsers)
}

```




## Controller

```js

//Different commands in controller


const User = require('../models/user.model');
 
module.exports.findAllUsers = (req, res) => {
    User.find()
        .then((allDaUsers) => {
            res.json({ users: allDaUsers })
        })
        .catch((err) => {
            res.json(err)
        });
}
 
module.exports.findOneSingleUser = (req, res) => {
    User.findOne({ _id: req.params.id })
        .then(oneSingleUser => {
            res.json({ user: oneSingleUser })
        })
        .catch((err) => {
            res.json(err)
        });}
 
module.exports.createNewUser = (req, res) => {
    User.create(req.body)
        .then(newlyCreatedUser => {
            res.json({ user: newlyCreatedUser })
        })
        .catch((err) => {
            res.json(err) // DELETE IF OTHER LINE IS UNCOMMENTED
            //res.status(400).json(err)
        });}

// THIS IS FOR EDIT/ UPDATE
module.exports.updateExistingUser = (req, res) => {
    User.findOneAndUpdate(
        { _id: req.params.id },
        req.body,
        { new: true, runValidators: true }
    )
        .then(updatedUser => {
            res.json({ user: updatedUser })
        })
        .catch((err) => {
            res.json(err) // DELETE IF OTHER LINE IS UNCOMMENTED
            //res.status(400).json(err)
        });}

// THIS IS FOR DELETE
module.exports.deleteAnExistingUser = (req, res) => {
    User.deleteOne({ _id: req.params.id })
        .then(result => {
            res.json({ result: result })
        })
        .catch((err) => {
            res.json(err)
        });}


```


## jwt.js

```js

const jwt = require("jsonwebtoken")
const secret = "the secret key"

module.exports.secret = secret

module.exports.authenticate = (req, res, next) =>{
    jwt.verify(req.cookies.usertoken, secret, (err, payload) => {
        if (err) {
            res.status(400).json({verified: false})
        }else{
            next()
        }
    })
}

```