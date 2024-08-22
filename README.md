## Next.js App Router Course - Starter

This is the starter template for the Next.js App Router Course. It contains the starting code for the dashboard application.

For more information, see the [course curriculum](https://nextjs.org/learn) on the Next.js Website.

## Passo 1
```
npx create-next-app@latest nextjs-dashboard --use-npm --example "https://github.com/vercel/next-learn/tree/main/dashboard/starter-example"
```

## CLSX
There may be cases where you may need to conditionally style an element based on state or some other condition.
clsx is a library that lets you toggle class names easily. We recommend taking a look at documentation for more details, but here's the basic usage:
```
import clsx from 'clsx';
 
export default function InvoiceStatus({ status }: { status: string }) {
  return (
    <span
      className={clsx(
        'inline-flex items-center rounded-full px-2 py-1 text-sm',
        {
          'bg-gray-100 text-gray-500': status === 'pending',
          'bg-green-500 text-white': status === 'paid',
        },
      )}
    >
    // ...
)}
```

## fonts.ts
```
import { Inter, Lusitana, Aladin } from 'next/font/google';
 
export const inter = Inter({ subsets: ['latin'] });

export const lusitana = Lusitana({
  subsets: ['latin'],
  weight: ['400','700']
});

export const aladin = Aladin({
  subsets: ['latin'],
  weight: '400'
});
```
```
import { lusitana, aladin } from './ui/fonts';
<span className={`${inter.className} antialiased`}>Log in</span> <ArrowRightIcon className="w-5 md:w-6" />
```

Images without dimensions and web fonts are common causes of layout shift due to the browser having to download additional resources.

## next/Image
Here, you're setting the width to 1000 and height to 760 pixels. It's good practice to set the width and height of your images to avoid layout shift, these should be an aspect ratio identical to the source image.
```
<Image
  src="/hero-desktop.png"
  width={1000}
  height={760}
  className="hidden md:block"
  alt="Screenshots of the dashboard project showing desktop version"
/>
```

## Chapter 4 - Creating Layouts and Pages

Creating the dashboard layout
Dashboards have some sort of navigation that is shared across multiple pages. In Next.js, you can use a special layout.tsx file to create UI that is shared between multiple pages. Let's create a layout for the dashboard pages!

Inside the /dashboard folder, add a new file called layout.tsx and paste the following code:

```
import SideNav from '@/app/ui/dashboard/sidenav';
 
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex h-screen flex-col md:flex-row md:overflow-hidden">
      <div className="w-full flex-none md:w-64">
        <SideNav />
      </div>
      <div className="flex-grow p-6 md:overflow-y-auto md:p-12">{children}</div>
    </div>
  );
}
```

Root layout
In Chapter 3, you imported the Inter font into another layout: /app/layout.tsx. As a reminder:

This is called a root layout and is required. Any UI you add to the root layout will be shared across all pages in your application. You can use the root layout to modify your \<html> and \<body> tags, and add metadata (you'll learn more about metadata in a later chapter).

Since the new layout you've just created (/app/dashboard/layout.tsx) is unique to the dashboard pages, you don't need to add any UI to the root layout above.

## cap. 5 - Why optimize navigation?
To link between pages, you'd traditionally use the \<a> HTML element. At the moment, the sidebar links use \<a> elements, but notice what happens when you navigate between the home, invoices, and customers pages on your browser.

Did you see it?

There's a full page refresh on each page navigation!

```
import Link from 'next/link';

const links = [
  { name: 'Home', href: '/dashboard', icon: HomeIcon },
  {
    name: 'Invoices',
    href: '/dashboard/invoices',
    icon: DocumentDuplicateIcon,
  },
  { name: 'Customers', href: '/dashboard/customers', icon: UserGroupIcon },
];

export default function NavLinks() {
  return (
    <>
      {links.map((link) => {
        const LinkIcon = link.icon;
        return (
          <Link
            key={link.name}
            href={link.href}
            className="flex h-[48px] grow items-center justify-center gap-2 rounded-md bg-gray-50 p-3 text-sm font-medium hover:bg-sky-100 hover:text-blue-600 md:flex-none md:justify-start md:p-2 md:px-3"
          >
            <LinkIcon className="w-6" />
            <p className="hidden md:block">{link.name}</p>
          </Link>
        );
      })}
    </>
  );
}
```
Além disso, na produção, sempre que os componentes \<Link> aparecem na janela de visualização do navegador, o Next.js pré-busca automaticamente o código da rota vinculada em segundo plano. No momento em que o usuário clicar no link, o código da página de destino já estará carregado em segundo plano, e é isso que torna a transição da página quase instantânea!

# Cap 6 - Setting Up Your Database

Create a Postgres database
Next, to set up a database, click Continue to Dashboard and select the Storage tab from your project dashboard. Select Connect Store → Create New → Postgres → Continue.

Navigate to your code editor and rename the .env.example file to .env. Paste in the copied contents from Vercel.
Important: Go to your .gitignore file and make sure .env is in the ignored files to prevent your database secrets from being exposed when you push to GitHub.
Finally, run 
##  npm i @vercel/postgres
in your terminal to install the Vercel Postgres SDK.

## Chapter 7 - Fetching Data
API layer
APIs are an intermediary layer between your application code and database. There are a few cases where you might use an API:

If you're using 3rd party services that provide an API.
If you're fetching data from the client, you want to have an API layer that runs on the server to avoid exposing your database secrets to the client.
In Next.js, you can create API endpoints using Route Handlers.

Database queries
When you're creating a full-stack application, you'll also need to write logic to interact with your database. For relational databases like Postgres, you can do this with SQL, or an ORM like Prisma.

There are a few cases where you have to write database queries:

When creating your API endpoints, you need to write logic to interact with your database.
If you are using React Server Components (fetching data on the server), you can skip the API layer, and query your database directly without risking exposing your database secrets to the client.

## Using SQL - cap. 7 - Fetching Data
### Using Server Components to fetch data

By default, Next.js applications use React Server Components. Fetching data with Server Components is a relatively new approach and there are a few benefits of using them:

Server Components support promises, providing a simpler solution for asynchronous tasks like data fetching. You can use async/await syntax without reaching out for useEffect, useState or data fetching libraries.
Server Components execute on the server, so you can keep expensive data fetches and logic on the server and only send the result to the client.
As mentioned before, since Server Components execute on the server, you can query the database directly without an additional API layer.

### Using SQL

### Parallel data fetching
A common way to avoid waterfalls is to initiate all data requests at the same time - in parallel.

In JavaScript, you can use the Promise.all() or Promise.allSettled() functions to initiate all promises at the same time. For example, in data.ts, we're using Promise.all() in the fetchCardData() function:
```
export async function fetchCardData() {
  try {
    const invoiceCountPromise = sql`SELECT COUNT(*) FROM invoices`;
    const customerCountPromise = sql`SELECT COUNT(*) FROM customers`;
    const invoiceStatusPromise = sql`SELECT
         SUM(CASE WHEN status = 'paid' THEN amount ELSE 0 END) AS "paid",
         SUM(CASE WHEN status = 'pending' THEN amount ELSE 0 END) AS "pending"
         FROM invoices`;
 
    const data = await Promise.all([
      invoiceCountPromise,
      customerCountPromise,
      invoiceStatusPromise,
    ]);
    // ...
  }
}
```

## Chapter 8 - Static and Dynamic Rendering

### What is Static Rendering?
- With static rendering, data fetching and rendering happens on the server at build time (when you deploy) or during revalidation. The result can then be distributed and cached in a Content Delivery Netw
- Static rendering is useful for UI with no data or data that is shared across users, such as a static blog post or a product page. It might not be a good fit for a dashboard that has personalized data that is regularly updated.

### What is Dynamic Rendering?
With dynamic rendering, content is rendered on the server for each user at request time (when the user visits the page). There are a couple of benefits of dynamic rendering:

- Real-Time Data - Dynamic rendering allows your application to display real-time or frequently updated data. This is ideal for applications where data changes often.
- User-Specific Content - It's easier to serve personalized content, such as dashboards or user profiles, and update the data based on user interaction.
- Request Time Information - Dynamic rendering allows you to access information that can only be known at request time, such as cookies or the URL search parameters.

### Making the dashboard dynamic
- By default, @vercel/postgres doesn't set its own caching semantics. This allows the framework to set its own static and dynamic behavior.

- You can use a Next.js API called unstable_noStore inside your Server Components or data fetching functions to opt out of static rendering. Let's add this.

- In your data.ts, import unstable_noStore from next/cache, and call it the top of your data fetching functions:

```
import { unstable_noStore as noStore } from 'next/cache';
 
export async function fetchRevenue() {
  // Add noStore() here to prevent the response from being cached.
  // This is equivalent to in fetch(..., {cache: 'no-store'}).
  noStore();
 
  // ...
}
```
Note: unstable_noStore is an experimental API and may change in the future. If you prefer to use a stable API in your own projects, you can also use the Segment Config Option export const dynamic = "force-dynamic".
https://nextjs.org/docs/app/api-reference/file-conventions/route-segment-config

## Chapter 9 - Streaming

### What is streaming?
- Streaming is a data transfer technique that allows you to break down a route into smaller "chunks" and progressively stream them from the server to the client as they become ready.

- By streaming, you can prevent slow data requests from blocking your whole page. This allows the user to see and interact with parts of the page without waiting for all the data to load before any UI can be shown to the user.

- Streaming works well with React's component model, as each component can be considered a chunk.

There are two ways you implement streaming in Next.js:

At the page level, with the loading.tsx file.
For specific components, with \<Suspense>.
Let's see how this works.

reduce your page's overall loading time.

### Streaming a whole page with loading.tsx

app/dashboard/loading.tsx
```
export default function Loading() {
  return <div>Loading...</div>;
}
```

### Adding loading skeletons
- A loading skeleton is a simplified version of the UI. Many websites use them as a placeholder (or fallback) to indicate to users that the content is loading. Any UI you embed into loading.tsx will be embedded as part of the static file, and sent first. Then, the rest of the dynamic content will be streamed from the server to the client.

```
import DashboardSkeleton from '@/app/ui/skeletons';
 
export default function Loading() {
  return <DashboardSkeleton />;
}
```

parei aqui : Grouping components
https://nextjs.org/learn/dashboard-app/streaming#grouping-components


### Cap 10 - Partial Prerendering (Optional)


How does Partial Prerendering work?
Partial Prerendering leverages React's Concurrent APIs and uses Suspense to defer rendering parts of your application until some condition is met (e.g. data is loaded).

The fallback is embedded into the initial static file along with other static content. At build time (or during revalidation), the static parts of the route are prerendered, and the rest is postponed until the user requests the route.

It's worth noting that wrapping a component in Suspense doesn't make the component itself dynamic (remember you used unstable_noStore to achieve this behavior), but rather Suspense is used as a boundary between the static and dynamic parts of your route.

The great thing about Partial Prerendering is that you don't need to change your code to use it. As long as you're using Suspense to wrap the dynamic parts of your route, Next.js will know which parts of your route are static and which are dynamic.

### Cap. 11 Adding Search and Pagination