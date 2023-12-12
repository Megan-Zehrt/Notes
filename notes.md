## Making a React 

```js

npx create-react-app folder-name (lowercase!!)  // ONLY FOR FRONT END

// FOR FULL STACK
npx create-react-app client  // we can call our project "client". Make sure you are in the same folder level as your "server.js" ( for full stack application )
npm install axios (api)
npm install react-router-dom (routing)

```








## FOR THE CLIENT ( REACT )

## Index.js

```js

// IMPORT AND WRAP AROUND APP

import {BrowserRouter} from 'react-router-dom'

  <React.StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </React.StrictMode>

```


## FOR THE SERVER

## Terminal

```js

create or touch server.js

npm init -y

npm install express

npm install mongoose dotenv

npm install mongodb

touch .env    // CREATE A .ENV FILE


touch .gitignore   // CREATE A GITIGNORE FILE


npm install cors // This will install the ability to make cross-origin requests. 

npm install -g nodemon // to automatically restart your node server

nodemon server.js  // TO RUN THE SERVER

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

## Route

```js

// for your route


const UserController = require('../controllers/user.controller');
 
module.exports = app => {
    app.get('/api/users', UserController.findAllUsers);
    app.get('/api/users/:id', UserController.findOneSingleUser);
    app.patch('/api/users/:id', UserController.updateExistingUser);
    app.post('/api/users', UserController.createNewUser);
    app.delete('/api/users/:id', UserController.deleteAnExistingUser);
}

```

## Server.js

```js


// for your server

const express = require('express');
const cors = require('cors');
const app = express();
require('dotenv').config();
const port = process.env.PORT;
require('./config/mongoose.config'); // This is new
app.use(cors());
app.use(express.json()); // This is new
app.use(express.urlencoded({ extended: true })); // This is new
require('./routes/person.routes')(app); // CHANGE THE PERSON NAME
    
app.listen(port, () => console.log(`Listening on port: ${port}`) );






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

## FUNCTIONS

## for your APP.js in Client

```js

import './App.css';
import Main from './components/Main'
import Edit from './components/Edit'
import Create from './components/Create'
import {Routes, Route, Navigate} from 'react-router-dom'

function App() {
  return (
    <div className="App">
      < Routes>
        <Route path="/users" element={<Main />} />
        <Route path="/users/new" element={<Create />} />
        <Route path="/users/:id/edit" element={<Edit />} />
        <Route path="*" element={< Navigate to={"/users"} />} />
      </Routes>

    </div>
  );
}

export default App;

```

## CREATE FUNCTION + FORM

```js

import React from 'react'
import { Link, useNavigate } from 'react-router-dom'
import axios from 'axios'
import { useState} from 'react'

const Create = (props) => {
    
    const [name, setName] = useState("")
    const [errors, setErrors] = useState([])

    const navigate = useNavigate()

    const submitAuthor = (e) => {
        e.preventDefault();

        const tempObjectToSendToDB = {
            name
        };

        // SUBMITING THE PRODUCTS
        axios.post("http://localhost:8000/api/authors", tempObjectToSendToDB)
            .then(res => {
                console.log("✅✅✅✅", res.data)
                navigate("/authors")
            })
            .catch(err => {
                console.log("❌❌❌❌", err)
                const errorResponse = err.response.data.errors;
                const errorArr = []
                for (const key of Object.keys(errorResponse)) {
                    errorArr.push(errorResponse[key].message)
                }
                setErrors(errorArr)
            }
            )

    }

    return (
        <div >
            <h1>Favorite authors</h1>
            <Link to={"/authors"} >Home</Link>
            <p>Add a new author:</p>
            <form onSubmit={submitAuthor}>
            {errors.map((err, index) => <p key={index}>{err}</p>)}
                <p>Name:</p>
                <input value={name} onChange={(e) => setName(e.target.value)} />
                <div>
                    <Link to={"/authors"} >Cancel</Link>
                    <button type="submit">Submit</button>
                </div>
            </form>
        </div>
    )
}

export default Create

```

## DELETE FUNCTION + SHOW ALL FUNCTION 

```js

import React from 'react'
import { useEffect, useState } from 'react'
import { Link } from 'react-router-dom'
import axios from 'axios'

const Main = (props) => {

  const [author, setAuthor] = useState([])

  useEffect(() => {
    axios.get("http://localhost:8000/api/authors")
      .then(res => {
        console.log("✅✅✅✅", res.data)
        setAuthor(res.data.authors)
      })
      .catch(err => console.log("❌❌❌❌", err))

  }, [])


  const deleteMe = (deletedId) => {
    axios.delete("http://localhost:8000/api/authors/" + deletedId)
      .then(res => {
        console.log("OK DELETED", res.data)
        const filteredAuthor = author.filter((eachAuthor) => {
          return deletedId !== eachAuthor._id;
        });
        setAuthor(filteredAuthor)

      })
      .catch(err => console.log(err))

  }


  return (
    <div>
      <h1>Favorite authors</h1>
      <Link to={"/authors/new"} >Add an author</Link>
      <p>We have quotes by:</p>
      {
        author.map((oneAuthor) => {
          return (
            <table key={oneAuthor._id}>
              <tr>
                <th>Author</th>
                <th>Actions available</th>
              </tr>
              <tr>
                <td>{oneAuthor.name}</td>
                <td>
                  <Link to={`/authors/${oneAuthor._id}/edit`} >Edit</Link>
                  <button onClick={() => deleteMe(oneAuthor._id)}>Delete</button>
                </td>
              </tr>
            </table>
          )
        })
      }
    </div>
  )
}

export default Main

```

## EDIT FUNCTION + PREPOPULATED FORM

```js

import React from 'react'
import { Link, useNavigate } from 'react-router-dom'
import axios from 'axios'
import { useState, useEffect } from 'react'
import { useParams } from 'react-router-dom'

const Edit = (props) => {

    const [name, setName] = useState("")
    const [errors, setErrors] = useState([])

    const navigate = useNavigate()

    const { id } = useParams()

    // SHOW DATA FROM AUTHOR
    useEffect(() => {
        axios.get("http://localhost:8000/api/authors/" + id)
            .then(res => {
                console.log("✅✅✅✅", res.data)

                setName(res.data.author.name)
            })
            .catch(err => console.log(err))

    }, [id])

    // SUBMITTING THE EDITTED AUTHOR DATA
    const submitAuthor = (e) => {
        e.preventDefault();

        const tempObjectToSendToDB = {
            name
        };

        axios.patch("http://localhost:8000/api/authors/" + id, tempObjectToSendToDB)
            .then(res => {
                console.log("✅✅✅✅", res.data)
                navigate("/authors")
            })
            .catch(err => {
                console.log("❌❌❌❌", err)
                const errorResponse = err.response.data.errors;
                const errorArr = []
                for (const key of Object.keys(errorResponse)) {
                    errorArr.push(errorResponse[key].message)
                }
                setErrors(errorArr)
            }
            )

    }

    return (
        <div >
            <h1>Favorite authors</h1>
            <Link to={"/authors"} >Home</Link>
            <p>Edit this author:</p>
            <form onSubmit={submitAuthor}>
                {errors.map((err, index) => <p key={index}>{err}</p>)}
                <p>Name:</p>
                <input value={name} onChange={(e) => setName(e.target.value)} />
                <div>
                    <Link to={"/authors"} >Cancel</Link>
                    <button type="submit">Submit</button>
                </div>
            </form>
        </div>
    )
}

export default Edit

```

## SHOW ONE

```js

import React from 'react'
import { useState, useEffect } from 'react'
import { useParams } from 'react-router-dom'
import axios from 'axios'
import { useNavigate } from 'react-router-dom'

const OneP = (props) => {

    const navigate = useNavigate()

    const { id } = useParams()
    const [thisProduct, setThisProduct] = useState(null)

    useEffect(() => {
        axios.get("http://localhost:8000/api/managers/" + id)
            .then(res => {
                console.log(res.data)
                setThisProduct(res.data.manager)
            })
            .catch(err => console.log("❌❌❌", err))
    }, [id]);

    const deleteMe = (deletedId) => {
        axios.delete("http://localhost:8000/api/managers/" + deletedId)
        .then(res => {
          console.log("OK DELETED", res.data)
          navigate("/products")
        })
        .catch(err => console.log(err))
      }

      

    return (
        <div>
            {
                thisProduct ? (
                    <div>
                        <h2>{thisProduct.title}</h2>
                        <p>{thisProduct.price}</p>
                        <p>{thisProduct.description}</p>
                        <button onClick={() => deleteMe(thisProduct._id)}>Delete</button>
                    </div>
                ) : <h3>Loading...</h3>
        }
        </div>
    )
}

export default OneP

```

## ERRORS

useRef is an installation problem

.map is not a function, Not mapping over an array, (res. data. database name)

when component won't load, check routes ( is there a / when needed? )

check lowercase, uppercase match

check spelling

check brackets, indendation

check imports from