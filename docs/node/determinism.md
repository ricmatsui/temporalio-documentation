# Determinism in Workflows

Temporal workflows are executed differently than conventional code as they can be restored at any point.
A workflow can sleep for months and even if your server crashes your timer will eventually resolve and your code will continue right where it left off.

For this to be possible workflow code must be completely deterministic, meaning it does the exact same thing every time it is rerun.
Determinism brings limitations, you can't just call an external service, get the current time, or generate a random number as these are all dependant on the state of the world at the time they're called.
The Temporal SDKs come with a set of tools which allow you to overcome these limitations.

### How a workflow is executed

The Temporal NodeJS SDK runs each workflow in a separate v8 isolate - a "sandbox" environment with its own global variables just like in the browser.
The workflow runtime is completely deterministic, functions like `Math.random`, `Date`, and `setTimeout` are replaced by deterministic versions and the only way for a workflow to interact with the world is via activities.
When an activity completes its result is stored in the workflow history to be replayed in case a workflow is restored.

### Imports in workflow code

Workflow code can reliably import [ES modules](https://nodejs.org/api/esm.html#esm_modules_ecmascript_modules).
In order for the Typescript compiler to output ES modules we set the [`module` compiler option](https://www.typescriptlang.org/tsconfig#module) to `es2020` in the initializer project (`npm init @temporalio`).

- [CommonJS](https://nodejs.org/docs/latest/api/modules.html#modules_modules_commonjs_modules) modules are experimentally supported via babel transformation using [babel-plugin-transform-commonjs](https://www.npmjs.com/package/babel-plugin-transform-commonjs).
- Built-in node modules are not supported and will throw an exception on import.

### Sources of non-determinism

- `Math.random` - replaced by the runtime
- `uuid4` - provided by the runtime
- `Date` - replaced by the runtime
- `WeakRef | WeakMap | WeakSet` - can not be used as GC is non-deterministic, deleted by the runtime
- Timers - `setTimeout` and `clearTimeout` are replaced by the runtime, prefer to use the exported `sleep` function
- Activities - use to run non-deterministic code, results are replayed from history