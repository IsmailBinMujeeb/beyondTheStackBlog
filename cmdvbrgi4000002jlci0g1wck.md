---
title: "Best Way To Structure Your Api Errors"
seoTitle: "API Error Structuring Guide"
seoDescription: "Learn to structure API errors using an `ApiError` class for consistent and maintainable code, improving debugging and developer experience"
datePublished: 2025-08-03T06:53:08.812Z
cuid: cmdvbrgi4000002jlci0g1wck
slug: best-way-to-structure-your-api-errors
cover: https://cloudmate-test.s3.us-east-1.amazonaws.com/uploads/covers/67aede0738c0c4fff22f37b2/102dff26-7cc1-40e5-b4fe-494b8f6d78cf.png
tags: javascript, nodejs, error-handling, backend, apis, architecture, rest-api, oops

---

In the previous blog, we discussed how to structure our API responses. In this article, we’ll take the same approach for **API errors**. If you haven’t read the previous blog, I recommend you check it out first as this article builds upon it. You can find it [here](https://hashnode.com/post/cmdls3tsk000g02jv3avqa5l2).

## Why Error Handling Matters

Errors are inevitable in backend architecture. They can occur due to user mistakes (e.g., requesting a non-existent user profile) or internal issues (e.g., a database crash). Designing a consistent and informative error structure is essential for debugging and improving the developer experience.

### Some Errors Are

| Status Code | Description |
| --- | --- |
| **400 Bad Request** | User missed a required field or sent invalid data |
| **401 Unauthorized** | User lacks necessary permissions |
| **404 Not Found** | Requested resource does not exist |
| **409 Conflict** | User tried to register with an already existing email |
| **500 Internal Server Error** | A server-side failure (likely a bug in the code) |

## Basic Error Handling in Practice

In the previous article, our controller looked like this:

```javascript
const mongoose = require("mongoose");
const userModel = require("../models/user.model.js");
const { ApiResponse } = require("../utils/ApiResponse.js");

export const profileController = async (req, res) => {
    try {
        const { id } = req.params;

        if (!id.trim() || !mongoose.Types.ObjectId.isValid(id)) {
            return res.status(400).json(new ApiResponse(400, "Invalid user ID", [], ["Invalid parameter"]));
        }

        const user = await userModel.findById(id).select("-password");

        if (!user) {
            return res.status(404).json(new ApiResponse(404, "User not found", [], ["No user with given ID"]));
        }

        return res.status(200).json(new ApiResponse(200, "User data fetched successfully", user));
    } catch (error) {
        console.error(error); // Log the error properly in production
        return res.status(500).json(new ApiResponse(500, "Internal Server Error", [], [error.message]));
    }
}
```

If the user doesn’t exist, the response will look like this:

```json
{
    "status": 404,
    "success": false,
    "message": "User not found",
    "data": [],
    "errors": ["No user with given ID"]
}
```

## Improving the Codebase with `ApiError`

To streamline and standardize error handling, we can introduce an `ApiError` class.

This class inherits from Node’s built-in `Error` class and encapsulates status code, message, and an array of error details. The `success` field is set to `false` by default.

### `ApiError` Class Implementation

```javascript
export class ApiError extends Error {
    
    constructor(status, message, errors=[]) {
        
        super(message);
        this.status = status;
        this.success = false;
        this.message = message;
        this.data = null;
        this.errors = errors;
    }
}
```

## **Updated Controller Using** `ApiError`

Here’s how the controller looks after implementing the `ApiError` class:

```javascript
const mongoose = require("mongoose");
const userModel = require("../models/user.model.js");
const { ApiResponse } = require("../utils/ApiResponse.js");
const { ApiError } = require("../utils/ApiError.js");

export const profileController = async (req, res) => {
    try {
        const { id } = req.params;

        if (!id.trim() || !mongoose.Types.ObjectId.isValid(id)) {
            return res.status(400).json(new ApiError(400, "Invalid user ID", ["Invalid parameter"]));
        }

        const user = await userModel.findById(id).select("-password");

        if (!user) {
            return res.status(404).json(new ApiError(404, "User not found", ["No user with given ID"]));
        }

        return res.status(200).json(new ApiResponse(200, "User data fetched successfully", user));
    } catch (error) {
        console.error(error); // Log the error properly in production
        return res.status(500).json(new ApiError(500, "Internal Server Error", [error.message]));
    }
}
```

Right now, Both `ApiResponse` and `ApiError` provides same functionality, But we will change it slightly by defining static functions in `ApiError` class. This static functions will provide you some basic functionality.

Start with `Not Found`, We will define a `NotFound` method in `ApiError`.

```javascript
export class ApiError extends Error {
    
    constructor(status, message, errors=[]) {
        
        super(message);
        this.status = status;
        this.success = false;
        this.message = message;
        this.data = null;
        this.errors = errors;
    }

    // Not Founc
    static NotFound(errors=[]){
        return new ApiError(404, "Not Found", errors);
    }
}
```

This will prevent your basic typos in the error messages in it helps you to throw errors like easy.

Now define methods for each error,

```javascript
export class ApiError extends Error {
    
    constructor(status, message, errors=[]) {
        
        super(message);
        this.status = status;
        this.success = false;
        this.message = message;
        this.data = null;
        this.errors = errors;
    }

    // Not Founc
    static NotFound(errors=[]){
        return new ApiError(404, "Not Found", errors);
    }

    // Unauthorized
    static Unauthorized(errors=[]){
        return new ApiError(401, "Unauthorized", errors);
    }

    // Bad Request
    static BadRequest(errors=[]){
        return new ApiError(400, "Bad Request", errors);
    }

    // Conflict
    static Conflict(errors=[]){
        return new ApiError(409, "Conflict", errors);
    }

    // Internal Server Error
    static InternalServerError(errors=[]){
        return new ApiError(500, "Internal Server Error", errors);
    }
}
```

This is your updated controller now.

```javascript
const mongoose = require("mongoose");
const userModel = require("../models/user.model.js");
const { ApiResponse } = require("../utils/ApiResponse.js");
const { ApiError } = require("../utils/ApiError.js");

export const profileController = async (req, res) => {
    try {
        const { id } = req.params;

        if (!id.trim() || !mongoose.Types.ObjectId.isValid(id)) {
            return res.status(400).json(ApiError().BadRequest(["Invalid parameter"]));
        }

        const user = await userModel.findById(id).select("-password");

        if (!user) {
            return res.status(404).json(ApiError().NotFound(["No user with given ID"]));
        }

        return res.status(200).json(new ApiResponse(200, "User data fetched successfully", user));
    } catch (error) {
        console.error(error); // Log the error properly in production
        return res.status(500).json(ApiError().InternalServerError([error.message]));
    }
}
```

## **Conclusion**

By introducing an `ApiError` class and using static factory methods like `BadRequest`, `NotFound`, `Conflict`, etc., your code becomes **consistent, maintainable and clean.**

This structure will **save you hours of debugging and rewriting** as your controller count grows from 5 to 50+. More importantly, it brings **predictability** to your API behaviour for frontend developers, third-party consumers, and testers.

Implement this error structure early in your project lifecycle — your future self and your team will thank you.