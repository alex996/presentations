# Modern Project

- Version control
- Automated CI / CD
- Code quality
- Tooling
- Module support
- Documented API
- Demos

## Build process

Automated sequence of tasks that runs on each push, tag, and/or release.

### Stages

1. install
2. lint
3. test
4. build
5. push
6. deploy

### Jobs

- install
  - clean install - `npm ci`
  - security audit - `npm audit`
- lint
  - linter - `eslint` / `stylelint`
  - formatter `prettier`
- test
  - test suite - `jest` / `mocha` / `ava`
  - code coverage - `nyc` / `codecov` / `coveralls`
- build
  - transpile - `babel` / `typescript` / `flow`
  - pre-process (compile, auto-prefix, etc.) - `sass` / `less` / `postcss`
  - uglify (minify, mingle, optimize, etc.) - `uglify-js` / `terser`
  - bundle (concat, tree-shake, etc.) - `webpack` / `rollup` / `parcel`
  - compress (gzip, etc.)
  - other
    - copy / delete / move files
    - check bundle size
    - strip unused code (ts/flow/proptypes)
- push
  - release - `github` / `bitbucket` / `gitlab`
  - publish - `npm` / other registry
- deploy
  - host - `heroku` / `surge` / `github-pages` / etc.

### Task execution

- CLI (`npm`)
- task runner
  - `grunt`, `gulp`, `brunch`

## CSS Conundrum

### Inline CSS

> `style=""` prop

**Ups**

👍 Easy SSR

👍 Single `import`

👍 No extra build step

**Downs**

👎 Can't use advanced CSS (`keyframe`, etc.)

👎 Very high specificity

### Embedded CSS

> `<style>` tag

**Ups**

👍 Easy SSR

👍 Single `import`

👍 Tree shaking both JS & CSS

**Downs**

👎 Risk of name collisions

👎 Heavier bundles (duplicated CSS)

### External CSS

> `styles.css`

**Ups**

👍 Easy SSR

👍 Lighter bundles (CSS extracted)

**Downs**

👎 Risk of name collisions

👎 No tree shaking in CSS

👎 Two imports JS & CSS

### CSS Modules

**Ups**

👍 Unique class names

👍 Single `import`

👍 Tree shaking both JS & CSS

**Downs**

👎 Hard/impossible SSR

👎 Heavier bundles (duplicated CSS)

### CSS in JS

**Ups**

👍 Unique class names

👍 Single `import`

👍 Tree shaking both JS & CSS

**Downs**

👎 Much heavier bundles (extra dep)

👎 Extra setup in the API
