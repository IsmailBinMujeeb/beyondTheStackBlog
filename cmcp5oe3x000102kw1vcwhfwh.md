---
title: "You are handling timestamps incorrectly with MongoDB!"
seoTitle: "You are handling timestamps incorrectly with MongoDB!"
seoDescription: "You're probably not even aware of it, but you use createdAt and updatedAt in your backend services all the time, right?"
datePublished: 2025-07-04T18:36:28.653Z
cuid: cmcp5oe3x000102kw1vcwhfwh
slug: you-are-handling-timestamps-incorrectly-with-mongodb
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1751694689375/3ba18230-ef02-4775-b87e-a745efdefcf1.png
tags: mongodb, nodejs, backend, databases, apis, mongoose, aggregation-pipeline, timestamp, backend-developments, timestring

---

You're probably not even aware of it, but you use `createdAt` and `updatedAt` in your backend services all the time, right?

But what if I told you that you're sending **raw timestamps** in your API responses like this:

```json
{
    "title": "You are handling timestamps incorrectly with MongoDB!",
    "createdAt": "2025-07-01T12:57:29.989+00:00"
}
```

You might say: *"What's wrong with that?"*

And you're right — technically, there's nothing *wrong* with it. You're just fetching data from the database and sending it directly to the frontend. That’s common practice.

**But wait**—let me introduce you to the actual problem.

## The Problem

When you send raw timestamps like `createdAt`, `updatedAt`, or any other custom timestamp field to the frontend, you're forcing your frontend code (and developer) to deal with date formatting.

And that's not ideal.

The frontend's primary responsibility is to provide a smooth and interactive **UI/UX**, *not* to format backend timestamps into readable strings.

Instead of this:

```json
{
    "title": "You are handling timestamps incorrectly with MongoDB!",
    "createdAt": "2025-07-01T12:57:29.989+00:00"
}
```

You should return this:

```json
{
    "title": "You are handling timestamps incorrectly with MongoDB!",
    "createdAt": "1 Jul 2025"
}
```

It’s more readable, more user-friendly, and more professional.

So how do you achieve this? Sure, you can use `new Date()` in Node.js, but there's a more elegant way.

Let’s format this ISO string into `"1 Jul 2025"` using **MongoDB Aggregation Pipelines**.

## Requirements

Before moving forward I want to make sure some of the requirements,

1. Node.js & npm Installed.
    
2. MongoDB Installed or at least you have access to MongoDB Atlas or any other service.
    
3. Mongoose (Yes! we will use mongoose in this article).
    
4. At least have basic understanding of MongoDB Aggregation Pipelines.
    

If you're good with all of the above, let’s continue.

## The Solution

### Step 1: Define a Schema

Create your `User` schema using Mongoose:

```javascript
import mongoose from "mongoose"; // mongoose should be installed

const userSchema = new mongoose.Schema({
    username: {
        type: String,
        required: true,
        unique: true,
    },
    email: {
        type: String,
        required: true,
        unique: true,
    },

    /** More fields if you want */
}, {timestamps: true});

export default mongoose.model("User", userSchema);
```

### Step 2: Configure Database Connection

Create a simple MongoDB connection file:

```javascript
import mongoose from "mongoose";

export default async () => {

    try {
        await mongoose.connect(/** Your database URI or env variable*/);
        console.log("Connected to Database");
    } catch (error) {
        console.log(error);
    }
}
```

### Step 3: Write the Main Entry File

In your main `index.js`:

```javascript
import user from "your/user/schema/path.js";
import db from "your/db/config/path.js";

;(
    async function () {
        try {
            await db();
            
            await user.create({
                username: "jhon123",
                email: "jhon123@example.com",
            });

            
            /** We will see this part later own */
        } catch (error){
            console.log(error);
        }
    }
)()
```

### Step 4: Format Timestamp Using Aggregation

Now for the main part: using an **Aggregation Pipeline** to format the timestamp.

```javascript
const createdUser = await user.aggregate([
    {
        $match: {
            username: "john123"
        }
    },
    {
        $addFields: {
            createdAt: {
                $dateToString: {
                    date: "$createdAt",
                    format: "%d %b %Y", // Outputs: 01 Jul 2025
                    // Optional: timezone, onNull
                }
            }
        }
    }
]);

console.log(createdUser);
```

### Code Explanation

Let’s break it down:

* `user.aggregate([...])`: Calls MongoDB’s aggregation function on the `User` collection.
    
* `$match`: Filters documents. In this case, we match by `username`.
    
* `$addFields`: Adds new or modifies existing fields in the result set.
    
* `$dateToString`: Formats a date object into a human-readable string.
    

#### Breakdown of `$dateToString` options:

* `date`: The date value to format (e.g. `$createdAt`)
    
* `format`: The format string. In our example, `%d %b %Y` gives you `01 Jul 2025`
    
    * `%d`: Day of the month (e.g. 01)
        
    * `%b`: Abbreviated month name (e.g. Jan, Feb, ..., Dec)
        
    * `%Y`: 4-digit year (e.g. 2025)
        
* `timezone` (optional): Specify a time zone. Defaults to UTC.
    
* `onNull` (optional): Value to return if the date is `null`.
    

The final result will give you a clean object with a **nicely formatted** `createdAt` field.

## Conclusion

From now on, don’t just return raw timestamps in your API responses.

Instead, **take advantage of MongoDB’s aggregation capabilities** to offload the formatting work from the frontend.

Clean, human-readable timestamps = better UX and cleaner frontend code.

Your backend should be smart enough to handle this formatting — and now, it is.