# Week 04

## Previous Class

Link to the previous class: [Week 03](https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/s2-24/lecture-notes/week-03.md)

---

## Before We Start

Open your **s2-24-intro-app-dev-repo-GitHub username** repository in **Visual Studio Code**. Create a new branch called **week-04-formative-assessment** from **week-03-formative-assessment**.

> **Note:** There are a lot of code examples. It is strongly recommended to type the code examples rather than copying and pasting. It will help you remember the code better. Also, read the comments in the code examples. It will help you understand where to type the code.

---

## API Versioning

API versioning is the process of versioning your API. It is important to version your API because it allows you to make changes to your API without breaking the existing clients. There are different ways to version your API. Some of the common ways are:

1. **URL Versioning:** In URL versioning, the version number is included in the URL. For example, `https://api.example.com/v1/products`.

2. **Query Parameter Versioning:** In query parameter versioning, the version number is included as a query parameter. For example, `https://api.example.com/products?version=1`.

3. **Header Versioning:** In header versioning, the version number is included in the header. For example, `Accept: application/vnd.example.v1+json`.

4. **Media Type Versioning:** In media type versioning, the version number is included in the media type. For example, `Accept: application/vnd.example.v1+json`.

---

### Controllers and Routes - Refactor

In the `controllers` and `routes` directories, create a new directory called `v1`. Move the `institution.js` files to the `v1` directories. Update the import path in the `routes/v1/institution.js` file.

```javascript
import {
  createInstitution,
  getInstitutions,
  getInstitution,
  updateInstitution,
  deleteInstitution,
} from "../../controllers/v1/institution.js";
```

Also, update the **Swagger** comments. For example, `/api/v1/institutions:` instead of `/api/institutions:`.

---

### Main File - Refactor

In the `app.js` file, update the import path for the `routes/v1/institution.js` file.

```javascript
// This should be declared under import indexRoutes from "./routes/index.js";
import institutionRoutes from "./routes/v1/institution.js";
```

Also, update the following code.

```javascript
// This should be declared under app.use("/", indexRoutes);
app.use(`/api/v1/institutions`, institutionRoutes);
```

---

### API Versioning Example

Run the application and go to <http://localhost:3000/api-docs>. You should see the following.

![](<../resources (ignore)/img/04/swagger-1.png>)

---

## Content Negotiation

Content negotiation is the process of selecting the best representation of a resource based on the client's preferences. There are different ways to perform content negotiation. Some of the common ways are:

1. **Accept Header:** In the Accept header, the client specifies the media types it can accept. For example, `Accept: application/json`.

2. **Content-Type Header:** In the Content-Type header, the client specifies the media type of the request body. For example, `Content-Type: application/json`.

3. **Query Parameter:** In the query parameter, the client specifies the media type. For example, `https://api.example.com/products?format=json`.

In this class, we will use the **Accept Header** to perform content negotiation.

---

### Middleware

In the root directory, create a new directory called `middleware`. In the `middleware` directory, create a new file called `utils.js`. Add the following code.

```javascript
const isContentTypeApplicationJSON = (req, res, next) => {
  // Check if the request method is POST or PUT
  if (req.method === "POST" || req.method === "PUT") {
    // Check if the Content-Type header is application/json
    const contentType = req.headers["content-type"];
    if (!contentType || contentType !== "application/json") {
      return res.status(400).json({
        error: {
          message: "Content-Type must be application/json",
        },
      });
    }
  }
  next();
};

export { isContentTypeApplicationJSON };
```

---

### Main File

In the `app.js` file, add the following code.

```javascript
// This should be declared under import institutionRoutes from "./routes/v1/institution.js";
import { isContentTypeApplicationJSON } from "./middleware/utils.js";

// This should be declared under const swaggerDocs = swaggerJSDoc(swaggerOptions);
app.use(isContentTypeApplicationJSON);
```

---

### Institution Router

In the `routes/v1/institution.js` file, update the following code.

```javascript
/**
 * @swagger
 * /api/v1/institutions:
 *   post:
 *     summary: Create a new institution
 *     tags:
 *       - Institution
 *     requestBody:
 *       required: true
 *       content:
 *         text/plain:
 *           schema:
 *             $ref: '#/components/schemas/Institution'
```

> **Note:** The `content` property has been updated to `text/plain`. It is for testing purposes only. Ensure that you update it to `application/json` after testing.

---

### Content Negotiation Example

Run the application and go to <http://localhost:3000/api-docs>. Click on the **POST** request for `/api/v1/institutions`. Click on the **Try it out** button. Add the following code in the **Request body**.

```json
{
  "name": "Otago Polytechnic",
  "region": "Otago",
  "country": "New Zealand"
}
```

Click on the **Execute** button. 

![](<../resources (ignore)/img/04/swagger-2.png>)

In the **Responses** section, you should see the following.

![](<../resources (ignore)/img/04/swagger-3.png>)

---

## Relationships

---

## Repository Pattern


---

## Next Class

Link to the next class: [Week 05](https://github.com/otago-polytechnic-bit-courses/ID607001-intro-app-dev-concepts/blob/s2-24/lecture-notes/week-05.md)