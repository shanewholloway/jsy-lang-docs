# JSY Language Documentation

JSY is **an indented (offside) JavaScript dialect**. We believe indentation is
better at describing code blocks instead of painstakingly matching open/close
sections `{ } () []`.

Think modern JavaScript — ES6, ES2018 — indented similar to [Python][] or [CoffeeScript][].

Inspired by [Wisp][], JSY primarily operates as a scanner-pass syntax
transformation to change indented (offside) code into the corresponding
open/close matching token code. Thus the internal scanning parser only has to
be aware of `/* comments */` and `"string literals"` rules to successfully
transform code. Thus, as a JavaScript dialect, **JSY automatically keeps pace with modern JavaScript editions!**

 [Python]: https://www.python.org
 [CoffeeScript]: https://coffeescript.org
 [Wisp]: http://www.draketo.de/english/wisp


- [Reference]()
  - [Operators](#operators) – `Promise.race @# api_call(), timeout(ms)`
  - [Keyword Operators](#keyword-operators) – `if 0 == arr.length :: «block»`
  - [Uncommon Operators](#operators-uncommon-use-cases)
  - [Idioms](#jsy-idioms)
  - [Integrations](#jsy-integration-with-other-tools)

- [Ecosystem](#ecosystem)
- [Projects Using JSY](#projects-using-jsy)



## Quick Start

Start with [rollup-plugin-jsy-lite](https://github.com/jsy-lang/rollup-plugin-jsy-lite#readme) or use the [playground](https://jsy-lang.github.io/)!

Sample JSY code:

```javascript
const apiUrl = 'http://api.example.com'

class ExampleApi extends SomeBaseClass ::
  constructor( credentials ) ::
    const apiCall = async ( pathName, body ) => ::
      const res = await fetch @ `${apiUrl}/${pathName}`, @{}
        method: 'POST'
        headers: @{}
        	'Content-Type': 'application/json'
        body: JSON.stringify @ body

      return await res.json()

    Object.assign @ this, @{}
      add: data => apiCall @ 'add', data
      modify: data => apiCall @ 'send', data
      retrieve: data => apiCall @ 'get', data
```



## Reference

There are at-based (`@`), double colon-based (`::`), and keyword operators (`if`, `for`, `while`, etc.). All operators wrap until the indentation is equal to or farther out than the current line, similar to Python or CoffeeScript. We refer to this as the *indented block*. For example:

  ```javascript
  function add( a, b ) ::
    return a + b
  ```

The double colon `::` in the preceding example opens a brace `{` when used, then closes the matching brace `}` when the indentation level matches that of the line it was first used on. The *indented block* is the return statement.

### Implicit Leading Commas

Commas are implicit at the first indent under any `@`-prefixed operator.

```javascript
console.log @
  "the"
  answer, "is"
  42
```
Translated to JavaScript:
```javascript
console.log(
   "the"
 , answer, "is"
 , 42 )
```

Explicit commas are respected.

```javascript
console.log @
    "or use"
  , "explicit commas"
```
Translated to JavaScript:
```javascript
console.log(
    "or use"
  , "explicit commas")
```

### Operators

#### `::` Double Colon – Code Blocks

The `::` operator wraps the *indented block* in curly braces `{«block»}`.

```javascript
function add( a, b ) ::
  return a + b
```
Translated to JavaScript:
```javascript
function add( a, b ) {
  return a + b
}
```

#### `@` At – Calling Functions
The `@` operator on its own wraps the *indented block* in parentheses `(«block»)`, where commas are implicit.

```javascript
console.log @
  add @ 2, 3
```
Translated to JavaScript:
```javascript
console.log(
  add( 2, 3 )
)
```

#### `@{}` At Braces – Hashes
The `@{}` operator wraps the *indented block* in curly braces `{«block»}`, where commas are implicit.

```javascript
fetch @ 'http://api.example.com', @{}
  method: 'POST'
  headers: @{}
    'Content-Type': 'application/json'
  body: JSON.stringify @ body
```
Translated to JavaScript:
```javascript
fetch( 'http://api.example.com', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify( body )
})
```

#### `@:` At Colon – Calling with Hash
The `@:` operator wraps the *indented block* in parentheses wrapping curly braces `({«block»})`, where commas are implicit.

```javascript
request @:
  url: 'http://api.example.com'
  method: 'POST'
  headers: @{}
    'Content-Type': 'application/json'
  body: JSON.stringify @ body
```
Translated to JavaScript:
```javascript
request({
  url: 'http://api.example.com',
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify( body )
})
```

#### `@[]` At Brackets - Lists
The `@[]` operator wraps the *indented block* in square brackets `[«block»]`, where commas are implicit.

```javascript
const tri = @[]
  @[]  0.0,  1.0, 0.0
  @[] -1.0, -1.0, 0.0
  @[]  1.0, -1.0, 0.0
```
Translated to JavaScript:
```javascript
const tri = [
  [0.0, 1.0, 0.0],
  [-1.0, -1.0, 0.0],
  [1.0, -1.0, 0.0]
]
```


#### `@#` At Pound – Calling with a List
The `@#` operator wraps the *indented block* in parentheses wrapping square brackets `([«block»])`, where commas are implicit.

```javascript
Promise.all @#
  fetch @ 'http://api.example.com/dosomething'
  fetch @ 'http://api.example.com/dosomethingelse'
  fetch @ 'http://api.example.com/dosomethingmore'
```
Translated to JavaScript:
```javascript
Promise.all([
  fetch('http://api.example.com/dosomething'),
  fetch('http://api.example.com/dosomethingelse'),
  fetch('http://api.example.com/dosomethingmore')
])
```

#### `@=>` At Arrow – Arrow Function Expression with No Arguments

The `@=>` operator wraps the *indented block* in parentheses and begins an arrow function `(()=> «block»)`.

The `@=>>` operator wraps the *indented block* in parentheses and begins an async arrow function `(async ()=> «block»)`.

```javascript
const call = fn => fn()

call @=> ::
  console.log @ 'Do cool things with JSY!'
```
Translated to JavaScript:

```javascript
const call = fn => fn()

call( () => {
  console.log('Do cool things with JSY!')
})
```

#### `@::` At Block – Arrow Function Block with No Arguments

The `@::` operator wraps the *indented block* in parentheses and begins an arrow function block `(()=> { «block» })`.

The `@::>` operator wraps the *indented block* in parentheses and begins an async arrow function block `(async ()=> { «block» })`.


```javascript
describe @ 'example test suite', @::
  it @ 'some test', @::>
    const res = await fetch @ 'http://api.example.com/dosomething'
    await res.json()
```

Translated to JavaScript:

```javascript
describe('example test suite', (() => {
  it('some test', (async () => {
    const res = await fetch('http://api.example.com/dosomething')
    await res.json()}) ) }) )
```

#### Arrow functions with Arguments

```javascript
// multiple arguments
const fn_body = @\ a, b, c ::
  body

const fn_body = @\ ...args =>
  expression


// first argument object destructure
const fn_obj_body = @\: a, b, c ::
  body

const fn_obj_expr = @\: a, b, c =>
  expression

// first argument array destructure
const fn_arr_body = @\# a, b, c ::
  body

const fn_arr_expr = @\# a, b, c =>
  expression
```

Translated to JavaScript:

```javascript
// multiple arguments
const fn_body = (( a, b, c ) => {
  body})

const fn_body = (( ...args ) =>
  expression)


// first argument object destructure
const fn_obj_body = (({ a, b, c }) => {
  body})

const fn_obj_expr = (({ a, b, c }) =>
  expression)

// first argument array destructure
const fn_arr_body = (([ a, b, c ]) => {
  body})

const fn_arr_expr = (([ a, b, c ]) =>
  expression)
```


#### `::!` and `@!` - Bang Immediately Invoked Expressions

The `::!` operator wraps the *indented block* in a function, then invokes it `{(() => {«block»})()}` in a block with no return value.

The `@!` operator wraps the *indented block* in a function, then invokes it `(() => {«block»})()` as an assignable expression.

```javascript
const a_value = @!
  const a = 1
  const b = 2
  return a + b

::!
  console.log @ 'Starting server...'
  startServer @ '1337', ()=> console.log @ 'Server online.'
```

Translated to JavaScript:

```javascript
const a_value = ((() => {
  const a = 1
  const b = 2
  return a + b })())

{(()=>{
  console.log('Starting server...')
  startServer('1337', ()=> console.log('Server online.')) })()}
```

#### `::!>` and `@!>` - Bang Async Immediately Invoked Expressions

The `::!>` operator wraps the *indented block* in an async function, then invokes it `{(async ()=>{«block»})()}` in a block with no return value.

The `@!>` operator wraps the *indented block* in an async function, then invokes it `(async ()=>{«block»})()` as an assignable expression.

```javascript
const promise_value = @!>
  const a = await Promise.resolve @ 1
  const b = await Promise.resolve @ 2
  return a + b

::!>
  console.log @ 'Starting server...'
  await new Promise @\ resolve ::
    startServer @ '1337', resolve

  console.log @ 'Server online.'
```

Translated to JavaScript:

```javascript
const promise_value = ((async () => {
  const a = await Promise.resolve(1)
  const b = await Promise.resolve(2)
  return a + b})())

{(async ()=>{
  console.log('Starting server...')
  await new Promise (( resolve ) => {
    startServer('1337', resolve) })

  console.log('Server online.') })()}
```



### Keyword Operators

For keywords with expressions, when not followed by a paren, everything between the keyword and the double colon `::` is captured as the keyword expression. When followed by a paren, parses as normal JavaScript.

No special parsing is done for keywords *without* expressions.

#### `if`/`else`, `while`, and `do`/`while`

```javascript
if a > b ::
  console.log @ 'JSY is the best!'
else if a < b ::
  console.log @ 'JSY rocks!'
else ::
  console.log @ 'JSY is still awesome!'

while 0 != q.length ::
  console.log @ q.pop()

do ::
  console.log @ 'It is a song that never ends...'
while 1
```

Translated to JavaScript:

```javascript
if (a > b) {
  console.log('JSY is the best!')
} else if (a < b) {
  console.log('JSY rocks!')
} else {
  console.log('JSY is still awesome!')
}

while (0 != q.length) {
  console.log(q.pop())
}

do {
  console.log("It is a song that never ends...");
} while (1);
```

#### `for`

```javascript
for let i = 0; i < 10; i++ ::
  console.log @: i

for const val of [1,'two',0xa] ::
  console.log @: val

for await const ea of someAsyncGenerator() ::
  console.log @: ea
```

Translated to JavaScript:
```javascript
for (let i = 0; i < 10; i++) {
  console.log({i})
}

for (const val of [1,'two',0xa]) {
  console.log({val})
}

for await (const ea of someAsyncGenerator()) {
  console.log({ea})
}
```

#### `try`, `catch`, and `finally`

```javascript
try ::
  if 0.5 > Math.random() ::
    throw new Error @ 'Oops!'
catch err ::
  console.error @ err
finally ::
  console.log @ 'Finally.'
```
Translated to JavaScript:
```javascript
try {
  if (0.5 > Math.random()) {
    throw new Error('Oops!')
  }
} catch (err) {
  console.error(err)
} finally {
  console.log('Finally.')
}
```

#### `switch`

```javascript
switch command ::
  case 'play':
    player.play()
    break
  case 'pause':
    player.pause()
    break
  default:
    player.stop().eject()
```
Translated to JavaScript:

```javascript
switch (command) {
  case 'play':
    player.play()
    break
  case 'pause':
    player.pause()
    break
  default:
    player.stop().eject()
}
```

### Operators (Uncommon Use Cases)

Uncommon use cases for `::`-prefixed operators require explicit commas, whereas the `@`-prefixed operators allow for the implicit use of commas.

| Uncommon Operator | Alias for | Use Instead |
| ----------------- | --------- | ----------- |
| `::{}`            | `::`      | `::`        |
| `::[]`            |           | `@[]`       |
| `::()`            |           | `@`         |
| `::@`             | `::()`    | `@`         |
| `@()`             | `@`       | `@`         |


## JSY Idioms

A few examples of why JSY is awesome.

### Idiom #1 – Arrow Function Returning Hash

```javascript
res.data.map @
  personData => @:
    fullName: `${ personData.info.fName } ${ personData.info.lName }`
    dateOfBirth: moment @ personData.info.dob
```

### Idiom #2 – Arrow Function Compose

```javascript
function double(x) :: return x + x
function add(x, y) :: return x + y

const clamp = (min, max, score) =>
  Math.max @ min,
  	Math.min @ max, score

const calcScore = v => @
  v = double @ v
  v = add @ 7, v
  v = clamp @ 0, 100, v
```

## JSY Integration with Other Tools

### Using `jsy-node` with NodeJS

#### Command Line

```bash
$ npm install -g jsy-node
$ jsy-node some-script.jsy
```

#### [Mocha](https://mochajs.org/)

```bash
$ mocha --require jsy-node/all some-unittest.jsy
```


## Ecosystem

 [Babel]: https://babeljs.io
 [Rollup]: https://rollupjs.org
 [Parcel]: https://parceljs.org

#### Pure JavaScript Transpiler (_stable_)

- [rollup-plugin-jsy-lite](https://github.com/jsy-lang/rollup-plugin-jsy-lite#readme)
  – [Rollup][] JSY syntax transpiler to standard JavaScript — without Babel

- [parcel-plugin-jsy](https://github.com/jsy-lang/parcel-plugin-jsy#readme) (_beta_)
  – [Parcel][] 1.x JSY syntax transpiler to standard JavaScript — without Babel

- [parcel-transform-jsy](https://github.com/jsy-lang/parcel-transform-jsy#readme) (_beta_)
  – [Parcel][] 2.x JSY syntax transpiler to standard JavaScript — without Babel

- [jsy-transpile](https://github.com/jsy-lang/jsy-transpile#readme) (_stable_)
  – Offside (indention) JSY syntax transpiler to standard JavaScript — without Babel

- [jsy-node](https://github.com/jsy-lang/jsy-node#readme) (_beta_)
  – Register runtime require handler for Offside (indention) JSY syntax transpiler to standard JavaScript.


#### Babel Transpiler (_newer, beta_)

- [babel-plugin-jsy-lite](https://github.com/jsy-lang/babel-plugin-jsy-lite#readme) (_**newer**, beta_)
  – [Babel][] 6.x & 7.x and jsy-transpile offside (indention) Javascript syntax extension.

- [babel-plugin-offside-js](https://github.com/jsy-lang/babel-plugin-offside-js#readme) (_older, stable_)
  – [Babel][] 6.x and Babylon offside (indention) Javascript syntax extension.

- [babel-preset-jsy](https://github.com/jsy-lang/babel-preset-jsy#readme) (_stable_)
  – [Babel][] 6.x preset for offside-based javascript syntax building on babel-preset-env

- [rollup-plugin-jsy-babel](https://github.com/jsy-lang/rollup-plugin-jsy-babel#readme) (_stable_)
  – [Babel][] configuration for using `babel-preset-jsy` in [Rollup][]


##### Misc Babel-based Tools

- [jsy-rollup-bundler](https://github.com/jsy-lang/jsy-rollup-bundler#readme)
  – JSY-oriented rollup bundling build chain for Web UI projects.

- [babel-convert-jsy-from-js](https://github.com/jsy-lang/babel-convert-jsy-from-js#readme)
  – Convert JavaScript, Babel or Babylon AST into offside indented JSY formatted source code.


#### Syntax Highlighters

- [jsy-lang/vim-jsy](https://github.com/jsy-lang/vim-jsy#readme)
  - Basic VIM/GVIM support for the JSY JavaScript dialect. Extends the builtin
    VIM javascript syntax, making it much less advanced than extensions like
    [othree/yajs](https://github.com/othree/yajs)
- [jsy-lang/prism-jsy](https://github.com/jsy-lang/prism-jsy#readme)
  - Basic [Prism](https://prismjs.com/) support for the JSY JavaScript dialect.
- Hacked together [JSY CodeMirror mode](https://github.com/jsy-lang/jsy-lang.github.io/blob/master/js/mode/jsy/jsy.js) from the JavaScript mode.


### Real Syntax Highlighters Needed!
Most JavaScript hightlighters work okay, but could certainly be better.

- Highlighter Libraries:
  - for [highlight.js](https://highlightjs.org/)
  - for [Pygments](http://pygments.org/)

- Code Editors:
  - for [CodeMirror](https://codemirror.net/)
  - for [VIM / GVIM](https://www.vim.org/)
  - for Sublime – use the [babel-sublime](https://github.com/babel/babel-sublime#installation) plugin
  - for Atom – use the [language-babel](https://atom.io/packages/language-babel) plugin
  - for VSCode – use the [sublime-babel-vscode](https://marketplace.visualstudio.com/items?itemName=joshpeng.sublime-babel-vscode) or the [vscode-language-babel](https://marketplace.visualstudio.com/items?itemName=mgmcdermott.vscode-language-babel) plugin


## Projects using JSY

- [shanewholloway/](https://github.com/shanewholloway) projects
  - Featured:
    - [msg-fabric-core](https://github.com/shanewholloway/msg-fabric-core)
    - [node-fate-observable](https://github.com/shanewholloway/node-fate-observable)
    - [js-evtbus](https://github.com/shanewholloway/js-evtbus)
    - [js-hashbelt](https://github.com/shanewholloway/js-hashbelt)

  - More:
    - [node-shamir-tss-gf256](https://github.com/shanewholloway/node-shamir-tss-gf256)
    - [js-object-functional](https://github.com/shanewholloway/js-object-functional)
    - [js-revitalize-object](https://github.com/shanewholloway/js-revitalize-object)
    - [js-consistent-fnvxor32](https://github.com/shanewholloway/js-consistent-fnvxor32)


## Thanks!

Special thanks to [Robert Sirois](https://github.com/rpsirois) for making this documentation happen!

Thanks to [Brian Brown](https://github.com/codecleric) for inspiring, pushing, and supporting JSY's creation.

Thanks to [Brandon Brown](https://github.com/RB-Bro) and Larry King for intensive use and feedback on JSY.

## Licenses

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">
<img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" />
</a><br/>
Documentation is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a>.

Examples and source code are licensed under [BSD 2-Clause](./LICENSE).

