`pnpm dlx shadcn@latest init` will create a new project, if theres any config file for nextjs, tanstack-start or vite-react then it will auto detect and will configure shadcn-ui but if there is no config file then it's a fresh project in that case it will create a new project with next, tanstack-start, vite-react, for the project type it will ask prompt but if framework name provided as flag then it will take it and won't ask.
Every framework support `--src-dir`, this flag will work only if no config file is detected, meaning it's a new project. if `--src-dir` is provided then it will create the project's code in `src` folder.

normal CLI is work, but if user provide `--src-dir` then it is breaking.

Note:

- The `init` command fails if it's a new project not existing.
- `init` has 2 main job (when creating new project)
  1. create a new project with CLi like nextjs, tanstack-start or vite-react
  2. configure shadcn's config like components.json, creating utils.ts in lib and others.

Here are some example to reproduce:

## without `--src-dir` flag in nextjs

```
PowerShell 7.5.4
PS C:\Yeasin\experiment> bunx   shadcn@latest init
√ The path C:\Yeasin\experiment does not contain a package.json file.
  Would you like to start a new project? » Next.js
√ What is your project named? ... ww
✔ Creating a new Next.js project.
√ Which color would you like to use as the base color? » Neutral
✔ Writing components.json.
✔ Checking registry.
✔ Updating CSS variables in app\globals.css
✔ Installing dependencies.
✔ Created 1 file:
  - lib\utils.ts

Success! Project initialization completed.
You may now add components.


```

## With (Nextjs) --src-dir flag

```
PS C:\Yeasin\experiment> bunx   shadcn@latest init --src-dir
√ The path C:\Yeasin\experiment does not contain a package.json file.
  Would you like to start a new project? » Next.js
√ What is your project named? ... ww2
✔ Creating a new Next.js project.
√ Which color would you like to use as the base color? » Neutral
✔ Writing components.json.
✔ Checking registry.
✔ Updating CSS variables in src\app\globals.css
✔ Installing dependencies.
✔ Created 1 file:
  - src\lib\utils.ts

⠋ Updating ..

Something went wrong. Please check the error below for more details.
If the problem persists, please open an issue on GitHub.

ENOENT: no such file or directory, open ''

PS C:\Yeasin\experiment>
```

### nextjs 2 example:

when using directly with project name.

```
PS C:\Yeasin\experiment\test-shadcn> bunx   shadcn@latest init my-app --yes


Something went wrong. Please check the error below for more details.
If the problem persists, please open an issue on GitHub.

Message:
The item at https://ui.shadcn.com/r/styles/new-york-v4/my-app.json was not found. It may not exist at the registry.

Suggestion:
Check if the item name is correct and the registry URL is accessible.

```

## nextjs existing project:

Existing project build using nextjs cli

```
PS C:\Yeasin\experiment\test-shadcn> cd .\my-app\
PS C:\Yeasin\experiment\test-shadcn\my-app> pnpm dlx shadcn@latest init
√ Preflight checks.
√ Verifying framework. Found Next.js.
√ Validating Tailwind CSS config. Found v4.
√ Validating import alias.
√ Which color would you like to use as the base color? » Neutral
√ Writing components.json.
√ Checking registry.
√ Updating CSS variables in src\app\globals.css
√ Installing dependencies.
√ Created 1 file:
  - src\lib\utils.ts

Success! Project initialization completed.
You may now add components.

```

# With Other frameworks.

Same issues if I provide `--src-dir` flag, otherwise it's working.

## With (react-vite) --src-dir flag

```


PS C:\Yeasin\experiment> bunx   shadcn@latest init --src-dir
√ The path C:\Yeasin\experiment does not contain a package.json file.
  Would you like to start a new project? » Vite
√ What is your project named? ... ww3
✔ Creating a new Vite project.
√ Which color would you like to use as the base color? » Neutral
✔ Writing components.json.
✔ Checking registry.
✔ Updating CSS variables in src\index.css
✔ Installing dependencies.
✔ Created 1 file:
  - src\lib\utils.ts

⠋ Updating ..

Something went wrong. Please check the error below for more details.
If the problem persists, please open an issue on GitHub.

ENOENT: no such file or directory, open ''

```

## with Tanstack start:

```

PS C:\Yeasin\experiment> bunx   shadcn@latest init --src-dir
√ The path C:\Yeasin\experiment does not contain a package.json file.
  Would you like to start a new project? » TanStack Start
√ What is your project named? ... ww4
✔ Creating a new TanStack Start project.
√ Which color would you like to use as the base color? » Neutral
✔ Writing components.json.
✔ Checking registry.
✔ Updating CSS variables in src\styles.css
✔ Installing dependencies.
✔ Created 1 file:
  - src\lib\utils.ts

⠋ Updating ..

Something went wrong. Please check the error below for more details.
If the problem persists, please open an issue on GitHub.

ENOENT: no such file or directory, open ''

```
