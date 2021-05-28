
Create fast, SEO optimized frontend web apps with zero config



1. Allows to **build the content on the server** so the 1st thing a user or bot sees on landing is **fully rendered HTML.** 
Hence, amazing SEO 
2. After that traditional CSR takes over and it works just like a regular web app
Hence, amazing UX

```bash
npx create-next-app app-name
```

## Key concepts

- Client Side Rendering (CSR)
    1. A traditional React App is rendered client side
    2. Browser starts with **a shell of an HTML page** (empty) lacking any rendered content
    3. From there, the browser fetches the app.js file containing the React code to the page and make it more interactive
- Static Site Generation (SSG)

    Pages are **rendered at build time** 

    **FEATURES :**

    1. Generates HTML pages at build time
    2. HTML is **rendered on the server** and uploaded to a storage bucket or static host
    3. Delivered with high performance over a CDN

    **DRAWBACKS**

    1. *Data may become stale* ⇒ need to rebuild and redeploy site when server side data changes, to implement the changes
    2. Gotta rebuild all files even if there is 1 small change
    3. *Hard to scale* ⇒ **Difficult to render all pages if website has too many pages

    **APPLICATION**

    1. Data that doesn't change often
    2. Sites that have relatively low number of total pages (eg : blog ⇒ few 100 pages that don't change on a daily bases)

- Server Side Rendering (SSR)

    Pages are **generated at request time**

    **FEATURES**

    1. Generate each page at request time.
    2. Ideal for data that changes constantly as end user always gets the **latest data** from whatever data source it exists on.

    **DRAWBACKS**

    1. *Slower* ⇒ Far less efficient as we gotta respond to requests instead of caching it all on a Global CDN.
    2. *Inefficient data caching.*
- Incremental Static Regeneration (ISR)

    **Regenerate pages in the background**

    1. By simply adding a **revalidate** option to the SSG function (atleast in Next JS),  a page can be regenerated after a fixed time interval
    2. A **server rendering strategy** midway between SSG and SSR ****
