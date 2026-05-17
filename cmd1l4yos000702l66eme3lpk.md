---
title: "How To Handle Date & Time Like Simple Numbers Using Luxon NPM"
seoTitle: "How To Handle Date & Time Like Simple Numbers Using Luxon NPM"
seoDescription: "Learn how to simplify JavaScript date and time operations using Luxon NPM. Easily add or subtract days, months, and years, and integrate dynamic date ranges"
datePublished: 2025-07-13T11:22:30.172Z
cuid: cmd1l4yos000702l66eme3lpk
slug: how-to-handle-date-and-time-like-simple-numbers-using-luxon-npm
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1752388005246/911ebe8c-f5ef-4468-9921-2b57402a81e7.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1752405666429/944f08b8-91fe-4864-9d5d-0184e7f7ff75.png
tags: javascript, nodejs, npm, typescript, datetime, npm-packages, luxon

---

In the past week, I was working on a **Telegram Bot Assistant**, where I had to work extensively with **dates and times**. Not only that, but I also needed to create **charts for analytics**, which meant handling time-based data in the **MongoDB Aggregation Pipeline**.

If you're interested in how you can manage timestamps efficiently within MongoDB, check out my latest post on that topic [here](https://beyondthestack.hashnode.dev/you-are-handling-timestamps-incorrectly-with-mongodb).

But first—when working with date and time in JavaScript, I found myself needing a package that could handle these operations elegantly and intuitively. After some research, I discovered an **amazing NPM package:** [**Luxon**](https://www.npmjs.com/package/luxon).

## Introducing Luxon

Luxon is a modern JavaScript library for working with **dates and times**. It is built by one of the `Moment.js` developers, aiming to provide a cleaner and more powerful API.

Consider this: today is **13 July 2025**. Suppose you want to retrieve data from your database (MongoDB, MySQL, Firebase, etc.) over a specific time range. Without a good library, you’ll be doing date math the hard way.

**Luxon** makes date manipulation feel like working with **basic math on numbers**.

## Increasing Date Using Luxon

Let’s say you want to increase a date by a few **months**, **days**, or **years**.

### With plain JavaScript:

```javascript
const now = new Date();
const futureDate = new Date(now.setMonth(now.getMonth() + 2)); // Add 2 months
console.log(futureDate);
```

This works, but it’s messy, not chainable, and hard to read.

### With Luxon:

```javascript
import { DateTime } from 'luxon';

const now = DateTime.now();
const futureDate = now.plus({ months: 2 });
console.log(futureDate.toISO()); // Outputs date in ISO format
```

With Luxon, this becomes cleaner and much more readable. You can add `days`, `weeks`, `months`, or `years` like simple objects.

## Decreasing Date Using Luxon

Now, let’s say you want to retrieve data from the **last 7, 14, 30, 60, or 90 days** based on user input.

### With plain JavaScript:

```javascript
const now = new Date();
const pastDate = new Date(now.setDate(now.getDate() - daysBack)); // Subtract 30 days
console.log(pastDate);
```

Not too bad, but again, not very intuitive or readable.

### With Luxon:

```javascript
import { DateTime } from 'luxon';

const daysBack = 30;
const now = DateTime.now();
const pastDate = now.minus({ days: daysBack });

console.log(pastDate.toISO()); // Easily readable and ISO formatted
```

This makes it extremely easy to subtract any unit of time dynamically—just change the value of `daysBack` depending on your need.

## Luxon Makes Dates Feel Like Numbers

One of the best things about Luxon is how **declarative** and **mathematical** it feels. You can chain operations, format output in different ways, and even easily work with **time zones** or **localized formats**.

### Example: Formatting Dates

```javascript
const formatted = DateTime.now().toFormat('yyyy-MM-dd');
console.log(formatted); // 2025-07-13
```

### Example: Setting Specific Date

```javascript
const customDate = DateTime.local(2025, 12, 25); // 25th Dec 2025
console.log(customDate.toLocaleString(DateTime.DATE_FULL)); // December 25, 2025
```

## Real-World Use Case: Dynamic Date Ranges for Analytics

In my Telegram bot project, users can choose to view data over the **last 7, 14, 30, 60, or 90 days**. Here’s how I used Luxon to generate those date ranges for my MongoDB queries:

```javascript
const now = DateTime.now();
const daysBack = 30; // or 7, 14, 60, 90 based on user input
const fromDate = now.minus({ days: daysBack }).toJSDate(); // Convert to JS Date for MongoDB

const mongoQuery = {
  createdAt: { $gte: fromDate }
};
```

It integrates perfectly into MongoDB queries and avoids all the Date arithmetic chaos.

## Installing Luxon

To get started with Luxon, simply install it via NPM:

```bash
npm install luxon
```

Then import it in your code:

```javascript
import { DateTime } from 'luxon';
```

## Conclusion

Handling dates and times doesn't need to be painful. **Luxon** gives you a modern, clean, and intuitive way to work with time like you would with plain numbers. Whether you're building bots, dashboards, or complex analytics, Luxon can save you time and make your code more readable.

If you're still manipulating raw `Date` objects manually, give **Luxon** a try—you'll thank yourself later.

Let me know in the comments if you want a detailed breakdown of how I integrated Luxon with MongoDB aggregation stages or if you'd like a full tutorial on generating analytics dashboards using this stack.