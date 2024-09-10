## Cheatsheet for client side



## Starting React 

```js

npx create-react-app client  

npm install axios (api)

npm install react-router-dom (routing)



```


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
        <Route path="*" element={< Navigate to={"/stride/home"} />} />
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