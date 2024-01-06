> It is a fork of [ts-node](https://www.npmjs.com/package/ts-node) package.

## Feature: ESM Glob Imports

You can use a glob pattern for imports, like `'../../modules/*.ts'` or `'../../meta/**/*.ts'`.

Such import returns default value, which is an object, where **key** is a resolved path and **value** is a loaded module.

For example, let us take such `src` folder structure:
- index.ts
- commands
  - user
    - firstUserCommand.ts
    - secondUserCommand.ts
  - admin
    - firstAdminCommand.ts
    - secondAdminCommand.ts

```ts
// commands/user/firstUserCommand.ts

export const action = 'first'
export const limit = 10
```

```ts
// commands/user/secondUserCommand.ts

export const action = 'second'
export const limit = 25
```

```ts
// commands/admin/firstAdminCommand.ts

export const action = 'first'
export default 'some default value'
```

```ts
// commands/admin/secondAdminCommand.ts

export const action = 'second'
export default 'another default value'
```

Then, usage:

```ts
// index.ts

// @ts-expect-error Glob Import
import commands from './commands/**/*.ts'
```

`commands` would be this:

```ts
{
  './commands/admin/firstAdminCommand.ts': {
    action: 'first',
    default: 'some default value'
  },
  './commands/admin/secondAdminCommand.ts': {
    action: 'second',
    default: 'another default value'
  },
  './commands/user/firstUserCommand.ts': {
    action: 'first',
    limit: 10 
  },
  './commands/user/secondUserCommand.ts': {
    action: 'second',
    limit: 25
  }
}
```

You can also use *dynamic imports*, you might want to do something like this:

```ts
// index.ts

const load = async () => {
  // @ts-expect-error Glob Import
  const commands = (await import('./commands/**/*.ts')).default as ...

  return commands
}
```

> [!NOTE]
> To resolve imports, there's a folder named `.ts_node_temp` will be created everytime you run `ts-node`. Add this folder to your `.gitignore`.

> [!IMPORTANT]
> Do not forget that JavaScript does not support glob imports by itself. Thus, you need to implement a plugin for your build stage that does the same mapping for glob imports as this fork of ts-node does. If such mapping does not work for you, just go [here](https://github.com/datismoa/ts-node/blob/main/dist-raw/node-internal-modules-esm-resolve.js#L880-L887) and [over here](https://github.com/datismoa/ts-node/blob/main/dist-raw/node-internal-modules-esm-resolve.js#L838-L842), then change the output.