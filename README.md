This is a [Next.js](https://nextjs.org/) v11 project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app). This is to demonstrate a build problem.


## How this test was made

As of July 27 2021 using the latest release of Next JS (v11) I am undable to get React Spectrum running.


```bash
npx create-next-app --typescript
```


For comparison, cloned the [React Spectrum Next JS sample](https://github.com/devongovett/rsp-next.git)

Added the Spectrum packages:

```bash
npm i @adobe/react-spectrum@nightly
```

We get endless warnings on dependency not being resolved
```bash
npm WARN Could not resolve dependency:
npm WARN peer @react-spectrum/provider@"^3.0.0-rc.1" from @react-spectrum/view@3.0.0-nightly.1157
```

So let's go for more…

```bash
npm i @spectrum-icons/illustrations@nightly
npm i @spectrum-icons/workflow@nightly 
```

Let's build
```bash
npm run build
```

All good.


Now import some items from React-spectrum to index.tsx


So here we have the first hurdle - the Global CSS cannot be imported from within node_modules.
```bash
error - ./node_modules/@adobe/react-spectrum/node_modules/@react-spectrum/provider/dist/main.css
Global CSS cannot be imported from within node_modules.
```

At this point the next.config.js only includes:

module.exports = {
  reactStrictMode: true,
}



## A custom NextJS config to the rescue?

Now we need the "next-transpile-modules"
```bash
npm install next-transpile-modules --save-dev
```

Note that in the sample code we had those with wildcards. 
```bash
  '@adobe/react-spectrum',
  '@spectrum-icons/.*',
  '@react-spectrum/.*'
```

next-transpile-modules throws an error
Error: Can't resolve '@spectrum-icons/.*'
So I update the config transpile part to:

```bash
  '@adobe/react-spectrum',
  '@spectrum-icons',
  '@react-spectrum'
```
And still - an error - even with trailing slashes.
Error: Can't resolve '@spectrum-icons' in '/Users/muri/LocalDev/rsp-next-ts'

AT THIS POINT I AM SURE THAT THERE IS AN ISSUE WITH 'next-transpile-modules'.

To move further I add the packages with their full paths.

```bash
  '@adobe/react-spectrum',
  '@spectrum-icons/workflow',
  '@react-spectrum/provider'
```

But now when running I get a SyntaxError for
node_modules/@adobe/react-spectrum/node_modules/@react-spectrum/provider/dist/module.js

And this is where it gets tricky.
Before one used to use the '@zeit/next-css/' to work around this but it's kind of obsolete.

But let's try.
```bash
npm i @zeit/next-css
```

Yay:
```bash
npm WARN deprecated @zeit/next-css@1.0.1: Next.js now has built-in support for CSS: https://nextjs.org/docs/basic-features/built-in-css-support. The built-in support solves many bugs and painpoints that the next-css plugin had.
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm WARN deprecated fsevents@1.2.13: fsevents 1 will break on node v14+ and could be using insecure binaries. Upgrade to fsevents 2.
npm WARN deprecated chokidar@2.1.8: Chokidar 2 will break on node v14+. Upgrade to chokidar 3 with 15x less dependencies.
```
Feeling like I still ..must... try..
So installing 'next-compose-plugins'
```bash
npm i next-compose-plugins --save-dev
```

And now the config looks like:
`
const withPlugins = require('next-compose-plugins')
const withCSS = require('@zeit/next-css')
const withTM = require('next-transpile-modules')([
  '@adobe/react-spectrum',
  '@spectrum-icons/workflow',
  '@react-spectrum/provider'
]);

module.exports = withPlugins([withCSS, withTM], {
  webpack5: false, // you want to keep using Webpack 4
});
`

Aaaand

We are stuck
`
SyntaxError: Unexpected token '.'
`

So let's enable Webpack 5 then and run.

`
TypeError: Cannot set property 'styles' of undefined
    at module.exports (/Users/muri/LocalDev/rsp-next-ts/node_modules/@zeit/next-css/css-loader-config.js:25:56)
`

yay.


So let's disable the CSS thing then for fun.

module.exports = withPlugins([withTM]....


Back to some previous state where CSS isn't validated.


So reading this - I changed the config to reference every submodule
https://github.com/adobe/react-spectrum/issues/1156


And still when you visit the page, it just fails to compile

`
Failed to compile
./node_modules/@adobe/react-spectrum/node_modules/@react-spectrum/provider/dist/main.css
Global CSS cannot be imported from within node_modules.
Read more: https://nextjs.org/docs/messages/css-npm
Location: node_modules/@adobe/react-spectrum/node_modules/@react-spectrum/provider/dist/module.js
This error occurred during the build process and can only be dismissed by fixing the error.
`


## Running it...

First, run the development server:

```bash
npm run dev
# or
yarn dev
```

