# 03: Node.js REST API 1

## Introduction

You consume **APIs** daily. They enable applications to communicate with each other, internally or externally.

The type of **API** you will develop and eventually consume is **REST** or **RESTful**. It is a set of architectural constraints. What it is not is a protocol or a standard. When a request is sent, it transfers a representation of the resource's state to the endpoint. This representation is delivered in one of many formats via **HTTP** such as **JSON**, **HTML**, etc.

### Anatomy of a REST API

The following table describes the different **HTTP methods**:

| HTTP Method | Description                              |
| ----------- | ---------------------------------------- |
| GET         | Provides read-only access to a resource. |
| POST        | Creates a new resource.                  |
| PUT         | Updates an existing resource.            |
| DELETE      | Removes a resource.                      |

There are a few others, but you will only be concerned with the four above.

## Express

As described, **Express** is a lightweight **Node.js** web application framework that provides a set of robust features for creating applications.

Assume you have **Node.js** installed. Create a new directory called `03-node-js-rest-api-1`. Change your current directory to `03-node-js-rest-api-1`. Now, it should be your working directory.

```bash
mkdir 03-node-js-rest-api-1
cd 03-node-js-rest-api-1
```

Open this directory in **Visual Studio Code**.

```bash
code .
```

Create a `package.json` file for your **REST API**.

```bash
npm init
```

It will prompt you to enter your **REST API's** name, version, etc. For now, you can press <kbd>Return</kbd> to accept the default values except for the following:

```bash
entry point: (index.js)
```

Enter `app.js` or whatever you want the name of the entry point file to be.

Install **Express** as a dependency. You can check whether it has been installed in `package.json`.

```bash
npm install express
```

Majority of the online **Node.js** examples use **CommonJS**. You are going to use **Modules** instead. In `package.json`, you need to add `"type": "module",` under `"main": "app.js",`.

**Resources:**

- <https://expressjs.com>
- <https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules>

## In-memory storage

You will explore **database storage** next week. For now, you will use **in-memory storage** via a local file.

Create a new file called `data.js`. This file will contain an **array** of **objects**.

```javascript
const institutions = [
    { id: 1, name: "Otago Polytechnic" },
    { id: 2, name: "Southern Institute of Technology" }
]

export { institutions }
```

You need to export it to use `institutions` outside of `data.js`.

Feel free to add more **objects** to the **array** or even more properties.

## Controllers

In the root directory, create a new directory called `controllers`. In this directory, create a new file called `institutions.js`. In `institutions.js`, you will write functions associated with the **HTTP methods** mentioned above.

To access `institutions` from `data.js`, you need to add the following:

```javascript
import { institutions } from '../data.js'
```

### Get function

All functions have at least two parameters - `req` and `res`. If you go to the route that is mapped to this function, you will be presented with a **JSON** response that contains `success` and `data`.

```javascript
const getInstitutions = (req, res) => {
    res
        .status(200)
        .json({ success: true, data: institutions })
}
```

### Create function

There is a little more going on here. When you make a **POST** request, you will send data with it. Also, it is important to validate the data before it is sent.

```javascript
const createInstitution = (req, res) => {
    const { name } = req.body

    if (!name) {
        return res
            .status(400)
            .json({ success: false, msg: 'Please provide a name' })
    } 

    const id = institutions[institutions.length - 1].id + 1
    
    institutions.push({
        id: id,
        name: name
    })

    res
        .status(201)
        .send({ success: true, data: institutions })
}
```

### Update function

You need to find the `institution` you want to update.

```javascript
const updateInstitution = (req, res) => {
    const { id } = req.params

    const institution = institutions.find(
        (institution) => institution.id === Number(id)
    )

    // Check if institution does exist
    if (!institution) {
        return res
            .status(404)
            .json({ success: false, msg: `No institution with the id ${id}` })
    }

    const newInstitutions = institutions.map((institution) => {
        if (institution.id === Number(id)) { // If institution does exist, update its name
            institution.name = req.body.name
        }
        return institution
    })

    res
        .status(200)
        .json({ success: true, data: newInstitutions })
}
```

### Delete function

Similar to the update function.

```javascript
const deleteInstitution = (req, res) => {
    const { id } = req.params

    const institution = institutions.find(
        (institution) => institution.id === Number(id)
    )

    // Check if institution does exist
    if (!institution) {
        return res
            .status(404)
            .json({ success: false, msg: `No institution with the id ${id}` })
    }

    const newInstitutions = institutions.filter(
        (institution) => institution.id !== Number(id) // If institution does exist, delete it
    )

    return res
        .status(200)
        .json({ success: true, data: newInstitutions })
}
```

You need to export these functions to use them outside of `institutions.js`.

```javascript
export {
    getInstitutions,
    createInstitution,
    updateInstitution,
    deleteInstitution
}
```

It may be hard to visualise at the moment, but it will become clearer soon. 

**Resource:** <https://developer.mozilla.org/en-US/docs/Web/HTTP/Status>

## Routes

In the root directory, create a new directory called `routes`. In this directory, create a new file called `institutions.js`. In `institutions.js`, you will create four routes and map them to the functions imported from `controllers/institutions.js`.

```javascript
import { Router } from 'express'
const router = Router() // Create a new router object. This allows to handle various requests

// Importing the four functions
import { 
    getInstitutions,
    createInstitution,
    updateInstitution,
    deleteInstitution 
} from '../controllers/institutions.js'

// Four routes that are mapped to the functions above
router.route('/').get(getInstitutions)
router.route('/').post(createInstitution)

router.route('/:id').put(updateInstitution)
router.route('/:id').delete(deleteInstitution)

// You can chain these if you wish. For example:
// router.route('/').get(getInstitution).post(createInstitution)
// router.route('/:id').put(updateInstitution).delete(deleteInstitution)

export default router // You do not need to enclose router in curly braces
```

## Entry point

In the root directory, create a new file called `app.js`. It is your **REST API's** entry point. Open a terminal and run the following command:

```bash
node app.js
```

Nothing happened. Why? It is because `app.js` is empty.

You need to add the following:

```javascript
import express, { urlencoded, json } from "express"

// Access all the routes exported from routes/institutions.js
import institutions from './routes/institutions.js'

const app = express()

const PORT = process.env.PORT || 3000

// Express middleware
app.use(urlencoded({ extended: false }))
app.use(json())

// To make it clear to the consumer that the application is an API, prefix the endpoint with /api
app.use('/api/institutions', institutions)

// Listening on port 3000
app.listen(PORT, () => {
    console.log(`Server is listening on port ${PORT}`)
})
```

Rerun the following command:

```bash
node app.js
```

Navigate to <http://localhost:3000/api/institutions>

## Postman

**Postman** is an **API** development environment that allows you to design, mock & test your **APIs**. The examples below are using the **desktop client**. Alternatively, you can use the **web client**. The interface is much the same on both **clients**.

To use **Postman**, you need to create an account or sign in. There are two sign-in options - username/password and **Google**.

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-1.JPG" />

Once you sign in, it will navigate you to **Home**.

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-2.JPG" />

Next to the **Home** tab is the **Workspaces** dropdown. Click this and create a **New Workspace**.

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-3.JPG" />

To keep things consistent with the **MongoDB Atlas** database, you will create next week, name your new workspace `id607001-<Your OP username>`. Once you have done this, click the **Create workspace and team** button in the bottom right-hand corner.

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-4.JPG" />

Click the **Go to Workspace** button.

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-5.JPG" />

You should see something like this:

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-6.JPG" />

Click the **+** button next to the **Overview** tab.

Here you will make your first request. Enter a request URL, i.e., http://localhost:3000/api/institutions, then click the **Send** button.

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-7.JPG" />

**POST** request example:

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-8.JPG" />

**Note**: Creating a new institution will not mutate the `institutions` **array** of **objects** in `data.js`. Though, it will mutate the copy imported.

**PUT** request example:

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-9.JPG" />

**Note:** This will change the institution's name.

**DELETE** request example:

<img src="https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/master/resources/img/03-node-js-rest-api-1/03-node-js-rest-api-10.JPG" />

**Note:** Much like creating a new institution, it will not mutate the `institutions` **array** of **objects** in `data.js`.

## Final thoughts

## Formative assessment