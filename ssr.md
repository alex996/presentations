## Server-Sider Rendering

#### Web Applications / Websites

- Client-Side Rendering (**`CSR`**)

- Server-Side Rendering (**`SSR`**)

#### Client-Side Rendered apps

`SPA` (*Single-Page Apps*) - each page is rendered on the **client**, _not_ on the server

- the server serves up a *single* static `index.html` document
- the browser then parses `HTML` & loads `CSS`, `JS`, `img` & fonts
- the `JS` bundle, once loaded, fetches data from an `API` & constructs the `DOM`
- the user navigates between routes via the `history API`, *not* page reloads
- the app lazy-loads remaining `JS` chunks on demand (*code splitting*)
- the site can be served from a `CDN` to speed up the load time

##### Pros

- decreased load on the server
- better UI/UX interactivity
- static hosting & caching
- decoupled architecture

##### Cons

- slow initial render
- [heavy `JS` dependency](https://kryogenix.org/code/browser/everyonehasjs.html)
- non-SEO friendly
- poor accessibility

##### Caveats

- initial render depends on `JS`, and can thus be *very* slow
  - bundle size (framework + app code), network latency, hardware limitations
- the page is technically ready to render once `HTML` & `CSS` are loaded
  - it doesn't need to wait on JS to load & execute before it can paint

#### Server-Side Rendered apps

`MPA` (*Multi-Page Apps*) - each page is rendered on the **server**, *not* on the client

- the server serves up a new `HTML` document on *each* request

- the browser loads & parses `CSS` to paint the UI

- the browser then loads & parses *deferred* `JS` to enhance the UI*

- the user navigates between routes by requesting a *new* page

- the browser caches assets (`CSS`, `JS`, `img`, etc.) for faster page load (just like with `CSR`)

- the server can cache static `HTML` on a `CDN` to decrease the load


\* `JS` can be made non render blocking; often done by

- `<script>` at the end of body to only load blocking `JS` *after* other assets
- `<script defer>` attribute to continue parsing `HTML` while loading `JS`

##### Pros

- fast first meaningful paint
- better perceived performance
- SEO-friendly

##### Cons

- increased load on the server
- latency due to page requests / reloads
- low UI/UX interactivity

#### CSR vs SSR?

Case in Point: [Walmart](https://medium.com/walmartlabs/the-benefits-of-server-side-rendering-over-client-side-rendering-5d07ff2cefe8)

#### Best of Both Worlds?

`React` is **isomorphic** (universal)

- can run in a browser, server, native app, or VR headset
- app can be server-rendered for faster paint
- then client-rendered to take over as an `SPA`!

#### Server-rendered React SPA

- the server pre-renders the app into an `HTML` string and serves it up
- the browser loads & parses `CSS` to make the page ***viewable***
- the browser then loads the `JS` bundle and boots the React SPA to make it ***interactive***
- React *doesn't need* to re-render if server and client markup are identical

## Create React App

#### ***Create-React-App***, or **`CRA`**

- zero configuration incubator for single-page React apps by Facebook
- sets up a dev environment with a *bundler*, *transpiler*, *linter*, *test runner*, & *live server*

- **Bundler**: bundles up assets (`JS` = modules + deps, `CSS`, `img`, fonts, etc)

- **Transpiler**: transforms ES6+ down to ES5 for better support across browsers

- **Linter**: anaylizes source for stylistic & logical errors (formatting, spacing, etc.)
- **Test runner**: allows for unit testing and code coverage reports
- **Live server**: dev server that watches files, and recompiles and auto-reloads as they change

#### Philosophy (from [README.md](https://github.com/facebook/create-react-app#philosophy))

- One build dependency (react-scripts: `Webpack`, `Babel`, `ESLint`, etc.)
- Zero configuration (sensible defaults, optimized production build)
- No lock-in (beginner-friendly customizable setup)

#### Best for

- Learning React & tooling
- SPA or static sites
- Rapid prototyping, demo apps, side projects

#### Caveats

- **ES6+ support**
  - stage 2 (draft), 3 (candidate) & 4 (finished): `async`/`await`, dynamic `import()`, `class fields`, etc. + `JSX` & `Flow`
  - but not stage 1 (proposal) or 0 (strawman) (e.g. `do expressions` or `function bind ::`)
  - also doesn't support `@decorators` (stage 2)
- **Polyfills**
  - but only few: `Promise`, `fetch`, `Object.assign`
  - and not others: `Array.find`, `Array.includes`, `Symbol`, `Intl`
  - will need to install & cherry-pick from `core-js` or `babel-polyfill`
- **Linting**
  - comes with `ESLint`, but needs a plugin for the editor, and/or `Prettier` for auto-formatting
- **Testing**
  - `Jest` test runner with `jsdom` to write test cases & assertions
  - doesn't isolate component that's being tested from its children
  - for isolated unit tests still need `Enzyme` or react-testing-library (with `jest-dom`)
- **Optimizations**
  - *minifies*, *uglifies*, and *compresses* code for prod. build
  - also includes *hashes* (for caching) and *source maps* (for debugging)
  - may need a tool e.g. `source-map-explorer` to analyze bundle size
- **Out of Scope**
  - <u>CI/CD</u> (`TravisCI`, `CircleCI`)
  - <u>Deployment</u> (`Firebase`, `Now`, `Surge`)
  - <u>UI/UX</u> or <u>other libraries</u> (`Bootstrap`, `Material-UI`)
  - though well documented

#### Downsides

- No SSR support (but does support [pre-rendering](https://github.com/facebook/create-react-app/blob/master/packages/react-scripts/template/README.md#pre-rendering-into-static-html-files))
- `SPA` only, no support for `MPA`
- Can't pass env vars at run-time
  - only at build time because the `HTML`/`CSS`/`JS` bundle is static
  - will need a server (e.g. `Node`) to load `HTML` & populate placeholders
- opinionated (some may say *bloated*) - may not need most tools / libs
  - though ejectable & configurable
- steep learning curve (elaborate config, though well documented)

#### Solution?

A custom project setup.
