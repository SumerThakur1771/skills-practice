# Week 5 - Day 2: App Router File-Based Routing and Dynamic Routes

## The core rule - folders become URL paths

In the App Router, the FOLDER STRUCTURE inside app/ directly determines the site's URLs. No routing code is written - the file system itself is the router.

```
app/
  page.js          -> yoursite.com/
  about/
    page.js        -> yoursite.com/about
```

Each folder that contains a `page.js` file becomes a real, visitable route, matching the folder's name and position in the file structure.

## Dynamic routes - the problem

Some pages need a URL that includes a changing value, like a specific user's ID (`/users/5`, `/users/42`) or a blog post's slug. A separate folder for every possible value can't be created in advance - a way is needed to say "this part of the URL can be anything."

## The syntax - square brackets in the folder name

```
app/
  users/
    [id]/
      page.js    -> matches /users/5, /users/42, /users/anything
```

Naming a folder with square brackets, like `[id]`, tells Next.js: this segment of the URL is a VARIABLE - capture whatever value is here, and make it available inside the page.

## Accessing the captured value - the params prop

Next.js automatically passes a special prop called `params` into the page component - an object containing whatever values were captured from the URL's dynamic segments. The property name matches whatever was used inside the square brackets.

```jsx
// app/users/[id]/page.js
export default function UserPage({ params }) {
  return <p>Showing user with ID: {params.id}</p>;
}
```

**Tracing a real example:** visiting `/users/42` matches the `[id]` folder, and Next.js automatically calls the component as `UserPage({ params: { id: "42" } })` - so `params.id` is the string `"42"`.

## My practice exercise

Goal: a blog page reachable at URLs like `/blog/my-first-post` or `/blog/react-tips`, showing the post's slug.

**Mistake made and fixed:** initially named the destructured parameter `{ props }` and accessed `props.postname` - Next.js specifically passes a prop called `params`, not `props`, and the property name must match the folder's bracket name exactly.

**Final correct code:**
```jsx
// app/blog/[postname]/page.js
export default function UserPage({ params }) {
  return (
    <p>Showing post with name: {params.postname}</p>
  );
}
```

Visiting `/blog/my-first-post` matches this `[postname]` folder, and `params.postname` equals `"my-first-post"`.

## Key takeaway (in my own words)

The App Router uses the folder structure inside app/ directly as the routing system - nested folders become nested URL paths, and each folder's page.js file is what displays at that route. Dynamic route segments are created by naming a folder with square brackets (e.g. [id]), which captures whatever value appears in that position of the URL. The captured value is automatically provided to the page component via a `params` prop, with property names matching the bracket names used in the folder structure.