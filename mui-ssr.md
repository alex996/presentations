# Material-UI - Server-side Rendering (SSR)

Render the React app on the server, and boot it up on the client.

## Why

- faster first meaningful paint
  - initial render much quicker (ex: 10s vs. 3s on slow 3G)
- better SEO (search engine optimization)

## How

- client sends a `GET` request to the server
- server renders the component tree into an `HTML` string
- it extracts critical `CSS` and injects it inline
- it sends off `HTML` & `CSS` as a response to the client
- client paints the page with `HTML` & `CSS` and loads `JS` async (***viewable***)
- once `JS` is loaded, React
  - hydrates server-rendered markup, and
  - attaches event listeners to existing markup (***interactive***)

## Build Setup

- have to transpile `JSX` server-side `preset-react`
- might as well use `import` (`ESM`) and transpile to `require` (`CJS`)
- for *dev*, could use `@babel/node` to compile on the fly in memory
- for *prod*, could use `@babel/cli` to precompile our code for execution
- might as well bundle with `webpack` since it's already part of build setup

## Integration

### Server

- `JssProvider`
  - *sheets registry*: `SheetsRegistry`
  - *class name generator*: `createGenerateClassName`
- `MuiThemeProvider`
  - *theme* *instance*: `createMuiTheme`
  - *sheets manager*: `Map`

### Client

- `JssProvider`
  - *class name generator*: `createGenerateClassName`
- `MuiThemeProvider`
  - *theme instance*: `createMuiTheme`

## Caveats

Integration is a *very* intricate procedure! You must make sure to

- provide a **new** *sheets manager* on each request
- provide a **new** *class name generator* on each request
- use `JssProvider` with a class name generator on the **client**
- use the **same** version of `material-ui` on the client & server
