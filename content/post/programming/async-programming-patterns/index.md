---
title: "Mastering Async Programming Patterns"
description: "Deep dive into asynchronous programming patterns that will level up your concurrent code."
slug: async-programming-patterns
date: 2024-05-20
categories:
  - Programming
tags:
  - async
  - concurrency
  - javascript
  - python
---

Asynchronous programming is essential for building responsive applications. Let's explore the key patterns you need to master.

## Callbacks (The Old Way)

```javascript
fetchUser(userId, (user) => {
  fetchPosts(user.id, (posts) => {
    fetchComments(posts[0].id, (comments) => {
      // Callback hell!
    })
  })
})
```

The problem: Callback hell makes code hard to read and maintain.

## Promises

Promises provide a cleaner way to handle async operations:

```javascript
fetchUser(userId)
  .then(user => fetchPosts(user.id))
  .then(posts => fetchComments(posts[0].id))
  .then(comments => console.log(comments))
  .catch(error => console.error(error))
```

## Async/Await

The modern approach that makes async code look synchronous:

```javascript
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId)
    const posts = await fetchPosts(user.id)
    const comments = await fetchComments(posts[0].id)
    return { user, posts, comments }
  } catch (error) {
    console.error('Failed to fetch data:', error)
  }
}
```

## Parallel Execution

When operations are independent, run them in parallel:

```javascript
async function fetchAllData() {
  const [users, products, orders] = await Promise.all([
    fetchUsers(),
    fetchProducts(),
    fetchOrders()
  ])
  return { users, products, orders }
}
```

## Python Async

Python has similar patterns with `asyncio`:

```python
import asyncio

async def fetch_data():
    user = await fetch_user()
    posts = await fetch_posts(user.id)
    return {'user': user, 'posts': posts}

async def fetch_parallel():
    users, products = await asyncio.gather(
        fetch_users(),
        fetch_products()
    )
    return users, products
```

## Error Handling Patterns

Always handle errors in async code:

```javascript
// Promise.allSettled for handling partial failures
const results = await Promise.allSettled([
  fetchUsers(),
  fetchProducts(),
  fetchOrders()
])

results.forEach(result => {
  if (result.status === 'fulfilled') {
    console.log(result.value)
  } else {
    console.error(result.reason)
  }
})
```

## Conclusion

Master these patterns to write efficient, readable async code. Start with async/await for most use cases, and use Promise.all for parallel operations.
