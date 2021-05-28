
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
    2. A **server rendering strategy** midway between SSG and SSR

### 1. *"pages"* **directory**

A file structure that mimics the routing setup in the app

1. Directory inside a Next project
2. Each JS file defined here **default** exports a React component that represents a route in the app
3. Next exposes its own Router to make navigation seamless
4. **index.js ⇒**
    - Your landing page
5. **_app.js ⇒** 
    - exists at highest level
    - main entry point into the app
    - every **individual page will start from this template**
    - all your page dependent and page independent (eg : navbar, footer) components are imported and arranged in layout here
    - Code sample

        ```jsx
        import '../styles/globals.css'
        import Navbar from '../components/Navbar'
        import { Toaster } from 'react-hot-toast'
        import {UserContext} from '../lib/contexts'
        import {useUserData} from '../lib/hooks'

        function MyApp({ Component, pageProps }) {
          
          const userValue = useUserData()

          return (
            <>
              <UserContext.Provider value={userValue}>
                <Navbar/>
                <Component {...pageProps} />
                <Toaster/>
              </UserContext.Provider>
            </>
            )
          
        }

        export default MyApp
        ```

6. When a user navigates to an internal route URL (specified by the **PAGE_NAME.js** or **DIR_NAME/PAGE_NAME.js** files), Next will look for the default export component in that file and render it as a component
Thus, every file here must have a default export component
7. **DYNAMIC ROUTING** 
    - [ PAGE_NAME.js] - " **[ ]** " make the route dynamic
    - Basically, helps us generate pages whose content and route is dependent on the URL of the page (slug part of the URL decides which content must be fixed)
8. **useRouter( )** ⇒ hook in Next JS to access the query parameters from the URL

    Using useRouter( )

    ```jsx
    const router = useRouter()
    const { slug } = router.query
    ```

### 2. *"api"* directory

1. Special part of Next for setting up routes that'll act as an **integrated server**
2. Useful since the code written here won't increase the client side JS bundle that needs to be ultimately sent over the net
3. Useful when
    - Work is to be done on the backend
    - You wanna expose on an API for your end users
- Code snippet from Traversy Media - sample blog to understand how client and server are both implemented in Next JS

    **pages/api/articles/[id].js (SERVER)**

    ```jsx
    import { articles } from '../../../data'

    export default function handler({ query: { id } }, res) {
      const filtered = articles.filter((article) => article.id === id)

      if (filtered.length > 0) {
        res.status(200).json(filtered[0])
      } else {
        res
          .status(404)
          .json({ message: `Article with the id of ${id} is not found` })
      }
    }
    ```

    **config/index.js**

    ```jsx
    export const server = isDev ? 'http://localhost:3000' : 'https://yourwebsite.com'
    ```

    **pages/articles/[id].js (CLIENT)**

    ```jsx
    import { server } from '../../../config'
    import Link from 'next/link'
    import { useRouter } from 'next/router'
    import Meta from '../../../components/Meta'

    const article = ({ article }) => {

      **// const router = useRouter()
      // const { id } = router.query**

      return (
        <>
          <Meta title={article.title} description={article.excerpt} />
          <h1>{article.title}</h1>
          <p>{article.body}</p>
          <br />
          <Link href='/'>Go Back</Link>
        </>
      )
    }

    export const getStaticProps = async (context) => {
      const res = await fetch(`${server}/api/articles/${context.params.id}`)

      const article = await res.json()

      return {
        props: {
          article,
        },
      }
    }

    export const getStaticPaths = async () => {
      const res = await fetch(`${server}/api/articles`)

      const articles = await res.json()

      const ids = articles.map((article) => article.id)
      const paths = ids.map((id) => ({ params: { id: id.toString() } }))

      return {
        paths,
        fallback: false,
      }
    }

    export default article
    ```

## 3. Styling

1. *globals.css* ⇒ Styles apply to the whole Next app
2. *.module.css ⇒ Component specific styles
3. Import the stylesheet into a component simply as an object and call styles as if **referencing properties on a JS object**
4. Can write actual CSS syntax inline
- Sample

    ```bash
    export const comp = () => {
    	<>	
    		<h1 className="title" >Hi !</h1>
    		<style jsx>
    			.title{
    					color:"#303045"
    			}
    		</style>
    	</>
    }
    ```

## 4. Server Rendering Strategies

- Implementing SSG

    **1.**  **getStaticProps( )** ⇒  tells Next to **pre-fetch the props** for the component's prerendering

    1. Data is fetched for a component by implementing the **getStaticProps( )** function inside the component file itself
    2. When you build the site, Next auto calls this function and sends the results as props to the component
    3. Can use the **'params'** argument in this function to **use this components route params** while making query to external source (eg : dB, API)

        ```jsx
        // use the params to get all the post data 
        export async function getStaticProps({params}) {
            const { slug } = params 
            
        		// Using slug to make call to API
            const res = await fetch(`http://localhost:1337/posts?Slug=${slug}`)
            console.log(res)
            const data = await res.json()
            const post = data[0]

            return {
                props: {
                    post
                }
            }
        }
        ```

    4. This function must return an **object that has the "props" property defined**. as

        ```jsx
        return {
        	props:{
        		something:somedata
        	}
        }

        ```

    5. These props are passed to the component
    6. In the component code, we can destructure this passed **"props"** and use it in our component's JSX

    **2.**  **getStaticPaths( )** ⇒  tells Next to **pre-fetch the dynamic paths** for the component's prerendering

    1. Since we're working with **dynamic routes**, **** we gotta inform Next about the pages we gonna associate with this route/URL

        eg : To prerender (SSG), all the "id" routes associated with the "cars" route on a website ( Route - website.com/cars/paths ), we gotta inform Next about all these ids in advance

    2. We provide this info by implementing the **getStaticPaths( )** function. 
    This function can also request data from an external source (dB/API) and then it's job is to return a **"paths" object that contains an array with every route for every dynamic URL.** 
    When I say "paths" I simply refer to the slug appendages at the end of the main URL

        ```jsx
        // Std return from getStaticPaths()
        return {
        	paths:pathArrayOfRoutes,
        	fallback: true/false/'blocking'
        }

        // **Example of using getStaticPaths()**
        // tell next js about all the paths that we want to generate these components from
        export async function getStaticPaths() {
            const res = await fetch('http://localhost:1337/posts')
            const posts = await res.json()
            const paths = posts.map(post => (
                {
                    params: {
                        slug:post.Slug
                    }
                }
            ))
            return {
                paths,
                fallback:false
            }
        }

        	
        ```

    3. We map our data from external source to this array of objects included in our **"paths"** object 

- **ISR** Code Sample

    ```jsx
    // Used **params** to access the current route's params to use in our query
    // Implementing **ISR**
    export async function getStaticProps({ params })
    {
        const { username, slug } = params;
        const userDoc = await getUserWithUsername(username)

        if (userDoc)
        {
            const postRef = userDoc.ref.collection('posts').doc(slug)
            const post = postToJSON(await postRef.get())

            const path = postRef.path
            return {
                **props: { post, path }**,
                **revalidate:5000**
            }
        } else
        {
            return {
                notFound:true
            }
        }

    }

    export async function getStaticPaths() {
        // Improve my using Admin SDK to select empty docs
        const snapshot = await firestore.collectionGroup('posts').get();
      
        const paths = snapshot.docs.map((doc) => {
          const { slug, username } = doc.data();
          return {
            params: { username, slug },
          };
        });
      
        return {
          **// must be in this format:
          // paths: [
          //   { params: { username, slug }}
          // ],
          paths,
    			// blocking as fallback to prevent 404 page from showing up in case paths are being fetched**
          **fallback: 'blocking',**
        };
    }

    export default function Post(**props**) {
        const postRef = firestore.doc(props.path);
        const [realtimePost] = useDocumentData(postRef);

        const post = realtimePost || props.post;
      
        return (
          <main className={styles.container}>
      
            <section>
              <PostContent post={post} />
            </section>
      
            <aside className="card">
              <p>
                <AuthCheck
                fallback={
                  <Link href="/enter">
                    <button>Sign Up</button>
                  </Link>
                }
              >
                <HeartButton postRef={postRef} />
              </AuthCheck>
                <strong>{post.heartCount || 0} 🤍</strong>
              </p>
      
            </aside>
          </main>
        );
      }
    ```

- Implementing SSR

    **getServerSideProps( )** ⇒ Fetches the props for the component's rendering at **request time**

    1. Can copy all the code in a getStaticPaths( ) function and use it here. Next will implement the data handling under the hood for us
    2. Don't need **getStaticPaths( )** here as each page is generated at request time so we don't need to inform Next in advance about any routes as there is no prerendering done

- SSR Code Sample

    ```jsx
    // Using the query as input to function to get the current route from the window's current location to use in our db query
    export async function getServerSideProps (**{ params }**) 
    {
        const {username} = params
        const userDoc = await getUserWithUsername(username) 
        
        // JSON serializable data
        let user = null
        let posts = null

        if (userDoc)
        {
            user = userDoc.data()
            const postsQuery = userDoc.ref
                .collection('posts')
                .where('published', '==', true)
                .orderBy('createdAt', 'desc')
                .limit(5)
            
            
            posts = (await postsQuery.get()).docs.map(postToJSON)
            return {
                **props:{user,posts}**
            }
        } else
        {
            return {
                notFound:true
            }
        }

    }

    export default function UserProfilePage(**{ user, posts }**)
    {
    return (
      <main>
            <UserProfile user={user} />    
            <PostFeed posts={posts} />
      </main>
      )
    }
    ```

## 5. SEO

1. Can import **Head** from **"next/head"** and easily apply custom meta tags for each and every single page in the pages directory
2. Anything inside the **Head** component will be rendered at the head of the doc as title, meta-tags, etc.
- Custom MetaTag component
    1. **Abstract** away all the logic for setting the title, meta-tags and description for the **Head** component into a separate custom reusable React component
    2. Pass **props to it from every single page** to get next level custom SEO, for each page
    3. Can set the default props to implement **DRY principle** (DRY - Don't Repeat Yourself)

    ```jsx
    import Head from 'next/head'

    const Meta = ({ title, keywords, description }) => {
      return (
        <Head>
          <meta name='viewport' content='width=device-width, initial-scale=1' />
          <meta name='keywords' content={keywords} />
          <meta name='description' content={description} />
          <meta charSet='utf-8' />
          <link rel='icon' href='/favicon.ico' />
          <title>{title}</title>
        </Head>
      )
    }

    Meta.defaultProps = {
      title: 'WebDev Newz',
      keywords: 'web development, programming',
      description: 'Get the latest news in web dev',
    }

    export default Meta
    ```

## 6. Important resources

1. Building a blog with Next, Strapi and Digital Ocean

    [Build a Fullstack App With Strapi and Next.js | 1-Hour Tech Talk | DigitalOcean](https://www.youtube.com/watch?v=WrmndNpWSJw&t=534s)

2. Websockets and Next

    [Building a realtime chat app with Next.js and Vercel | Ably Blog: Data in Motion](https://ably.com/blog/realtime-chat-app-nextjs-vercel)

3. Server in Next

    [Advanced Features: Custom Server | Next.js](https://nextjs.org/docs/advanced-features/custom-server)

    [Creating a Simple Server with Next.js](https://spin.atomicobject.com/2019/09/18/nextjs-simple-server/)
