# How to add path alias' in GatsbyJS and Typescript


Like many developers I regularly find myself trying to organise my projects folder structures to make it very clear where everything is, which usually ends with my project folder looking something similar to this:


```bash
myProject/
    node_modules/
    public/
    src/
      assets/
        images/
          image.png
        svgs/
          logo.svg
      components/
        header/
          header.module.css
          header.tsx
        navbar/
          navbar.module.tsx
          navbar.tsx
        footer/
          footer.module.tsx
          footer.tsx
        card/
          card.module.tsx
          card.tsx
        button/
          button.module.tsx
          button.tsx
      hooks/
        someHook.ts
      layouts/
         main/
            mainLayout.module.css
            mainLayout.tsx
         store/
            storeLayout.module.css
            storeLayout.tsx
      pages/
        index.tsx
        about.tsx
        contact.tsx
        projects.tsx
        store.tsx
      styles/
        reset.css
        global.css
     gatsby-config.js
    .gitignore
    .prettierrc
    .eslintrc
    .package.json
    .package-lock.json
    tsconfig.json
```

everything has a place and everything IN its place, the one issue with this is when importing assets or components into a page we will end up with this ugly thing 


```javascript
// src/layouts/mainLayout.tsx

import {navbar} from '../../components/navbar'  

// YUCK!

```

relative import paths make everything so much more annoying, not only when we want to import something, do we have to do some mental gymnastics to understand where exactly we are in the folder
structure when we prefix another "../" to the path but depending on how far you nest your components it can get much much worse eg


```javascript

import {thatOneComponent} from '../../../../foo/bar/bar'  

```

you can fix some of it with adding index.ts in each folder and re exporting components but it gets very unweildy quickly


A good solution i found was to use typescript path alias, which mean we can declare out imports like this

```javascript

import {thatOneComponent} from '@components/foo/bar/bar'  

```

we dont have to think about relative imports, we say "from the components folder i want this thing!" and if we make good use of an index.ts in the component folder we can get away with

```javascript

// src/layouts/storeLayout.tsx

import {thatOneComponent} from '@components'  

```

muuch better and its super simple to implement

```json
  // tsconfig.json
  
  {
    "compilerOptions: {
      
      ...
      
      "baseUrl": "src",
      "paths": {
        "@components/*": ["components/*"],
        "@assets/*": ["assets/*"],
        "@hooks/*": ["hooks/*"],
        "@images/*": ["images/*"],
        "@layouts/*": ["layouts/*"]
      }
    }
  }

```

broken down

### baseUrl
this sets the base directory to resolve non-absolute module names

in our example we set it to `"src" so typescript will look in src directory for any paths not resolve with an absolute path eg prefixed with "./" or "../"

### paths 
tsc defines this as "A Series of entires which re-map to lookup locations relative to the baseUrl"

which translates to as: an object where the keys are the paths you will use, and the values are what that will resolve to when typescript sees it

in out example we have five folders declared `components`, `assets`, `hooks`, `images` and `layouts` weve prefixed out custom alias' with an `@` to make it clear this is a custom alias
and not a node_module (this is purely for future you and other developers!) this can be any symbol `@` just seems to be the convension used, in our keys we also suffix them with a `/*`,
this is so any nested folders are also included.

and in a pure typescript project that is all you need to do.


## What about in Gatsby?

we gatsby uses typescript by default where all you need is a tsconfig file and to rename your files to `.ts` and `.tsx` and you can enjoy the brillant world of typescript.

the one issue is when using typescripts path alias in gatsby developement environment (theres always a catch!) to resolve this we have to do what we already do when we have an issue with gatsby...

#### download a plugin!


### gatsby-plugin-root-import

this set you can use even without typescript so everyone is happy! but if using typescript you will need to do both to make both gatsby and tsc happy (but thats what you get for tring to people please!)


first of all install it...
```bash
npm install --save-dev gatsby-plugin-root-import
```

then add a config object to your gatsby-config.js
```javascript
  const path = require('path')

  module.exports = {
    ...

    plugins : [
      ...

      {
        resolve: `gatsby-plugin-root-import`,
        options: {
          "@components": path.join(__dirname, `src/components`),
          "@assets": path.join(__dirname, `src/assets`),
          "@hooks": path.join(__dirname, `src/hooks`),
          "@images": path.join(__dirname, `src/images`),
          "@layouts": path.join(__dirname, `src/layouts`),
        }
      }
    ]
  }
```

this one is structured a bit differently, first of, we dont declare a root url to work from, we will have to use nodes `path` module along with the `__dirname` global variable to resolve the paths to correct folders,
we also dont need to suffix a "/*" if we wish to use nested folder as it will just replace `@components` with `<path_to_root>/src/components` so `@components/navbar` will get resolved as 
`<path_to_root>/src/components/navbar`

and thats it!

now go forth and never have to be bothered by aweful relative imports ever again!

if you found this helpful or have any amendments let me know on twitter im @phl3bas


