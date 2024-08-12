# How to setup a project with React and Vite

This guide will help you setup your React project with [Vite](https://vitejs.dev/).

This guide was written with Vite 5.3 in mind. In case it no longer is up to date with the current Vite version, please fall back on to it until I update the guide.

I mostly use `yarn` as my package manager. I will try to have my commands in both `npm` and `yarn` form. If you are using a different package manager please make the correct adjustments.

## 1. Create a React + Vite project

Start by running the following command in directory where you want your project folder to be.

`yarn create vite` or `npm create vite`

You will be prompted to enter a project name and select which framework to use.
In our case we want to choose `React` and `TypeScript`.

e.g:
```
yarn create vite

Project name: my-vite-project
? Select a framework: React
? Select a variant: TypeScript

cd my-vite-project
yarn install
yarn dev
```

## 2. Add IE Support

We should always make sure our projects have some "legacy" protection. So to achieve some IE support we can follow these steps to ensure our app at least runs in IE.

1. 
    Install the following package `react-app-polyfill`. 

    `yarn add react-app-polyfill` or `npm install react-app-polyfill`

2. 
    Afterwards we should add the following imports to our `src/main.tsx`

    ```Typescript
    import 'react-app-polyfill/ie11';
    import 'react-app-polyfill/stable';
    ```

3. 
    In your `package.json` add the following `browsersList` section (after the `script` section):
    ```json
    {
        ...
        "browsersList": {
            "production": [
                ">0.2%",
                "not dead",
                "not op_mini all",
                "ie 11"
            ],
            "development": [
                "last 1 chrome version",
                "last 1 firefox version",
                "last 1 safari version",
                "ie 11"
            ]
        }
        ...
    }
    ```

## 3. SVG Support
    Most React project will want to use SVG's. So we might as well support them right out of the gate.

1. 
    Install the `vite-plugin-svgr` package.

    `yarn add vite-plugin-svgr --dev`
    
    or
    
    `npm install vite-plugin-svgr --save-dev`

2. 
    Add the following to the `src/vite-env.d.ts` file.
    ```Typescript
    /// <reference types="vite-plugin-svgr/client" />
    ```

3. 
    Add the following to the `vite.config.ts`.
    ```Typescript
    import svgr from 'vite-plugin-svgr';

    ...
        plugins: [
            ...
            svgr({
                // svgr options: https://react-svgr.com/docs/options/
                svgrOptions: {
                    exportType: 'default', ref: true, svgo: false, titleProp: true,
                },
                include: '**/*.svg',
            }),
        ]
    ```

4. 
    From now on, your SVGs will be imported as ReactComponents. To keep your project consistent, import them with PascalCase names as you would any other component:
    `import MySvgIcon from '../asset/svg/my-svg.svg'`

    And make use of them as you would any other component:
    ```Typescript
    import MySvgIcon from '../somewhere/my-svg.svg';
    
    const Component = () => {
        return (
            <div>
                <MySvgIcon />
            </div>
        )
    }
    ```

## Tests

Tests are important on any project. If you are creating a pet project or just a POC consider this section as optional, for all other projects please consider adding the test support and also write tests, lots of them <3

By default Vite doesn't come with test support out of the box. And even the testing library it has is very bare bones. So instead of using `vitest` we are going to stick to what works and use `jest`.

1. 
    Install `jest` and all the necessary libs into your devDependencies

    `yarn add jest @types/jest ts-jest @testing-library/react @testing-library/user-event jest-environment-jsdom @testing-library/jest-dom test-node @testing-library/dom --dev`

    or

    `npm install jest @types/jest ts-jest @testing-library/react @testing-library/user-event jest-environment-jsdom @testing-library/jest-dom test-node @testing-library/dom --save-dev`

2. 
    Add `"esModuleInterop": true,` to your `tsconfig.app.json` in the `compilerOptions` section. Cut your entire `tsconfig.app.json` into `tsconfig.json` file and make the necessary changes.
    Your `tsconfig.json` should look like this.

    ```json
    {
        "compilerOptions": {
            "composite": true,
            "tsBuildInfoFile": "./node_modules/.tmp/tsconfig.app.tsbuildinfo",
            "target": "ES2020",
            "useDefineForClassFields": true,
            "lib": ["ES2020", "DOM", "DOM.Iterable"],
            "module": "ESNext",
            "skipLibCheck": true,
            "esModuleInterop": true, // We just added this
            
            /* Bundler mode */
            "moduleResolution": "bundler",
            "allowImportingTsExtensions": true,
            "resolveJsonModule": true,
            "isolatedModules": true,
            "moduleDetection": "force",
            "noEmit": true,
            "jsx": "react-jsx",
            
            /* Linting */
            "strict": true,
            "noUnusedLocals": true,
            "noUnusedParameters": true,
            "noFallthroughCasesInSwitch": true
        },
        "include": ["src"],
        "references": [
            // removed the link to tsconfig.app.json
            {
                "path": "./tsconfig.node.json"
            }
        ]
    }
    ```

    Note: You can delete `tsconfig.app.json`. Also you might see a Typescript error regarding your `tsconfig.node.json` file. This is because you should remove the following linting rule from that file `"noEmit": true,`

3. 
    Install the following packages.

    `yarn add identity-obj-proxy jest-transformer-svg --dev`

    or

    `npm install identity-obj-proxy jest-transformer-svg --save-dev`

4. 
    Create a Jest config file `jest.config.json` in the project folder with the following:
    ```json
    {
        "transform": {
            "^.+\\.tsx?$": [
                "ts-jest"
            ],
            "\\.svg$": "jest-transformer-svg"
        },
        "moduleNameMapper": {
            "\\.(css|less|sass|scss)$": "identity-obj-proxy"
        },
        "testEnvironment": "jsdom",
        "setupFilesAfterEnv": ["<rootDir>/src/jest.setup.ts"]
    }
    ```

5. 
    Create a file to configure your tests `src/jest.setup.ts`
    ```Typescript
    import '@testing-library/jest-dom';

    afterEach(() => {
        jest.resetAllMocks();
    });
    ```

6. 
    Add to your `package.json` the following so you can run tests
    ```json
    {
        ...
        "scripts": {
            ...
            "test": "jest"
        }
    }
    ```
7. 
    Write some tests <3

## Optional

### ESLint

1. 
    Install the following packages `eslint`, `eslint-plugin-react`, `eslint-plugin-import`, `@typescript-eslint/parser` and `@typescript-eslint/eslint-plugin` to your devDependencies.

    `yarn add eslint eslint-plugin-react eslint-plugin-import @typescript-eslint/parser @typescript-eslint/eslint-plugin --dev`

    or

    `npm install eslint eslint-plugin-react eslint-plugin-import @typescript-eslint/parser @typescript-eslint/eslint-plugin --save-dev`

2. 
    Add a `.eslintrc.cjs` to your project's folder.

    Note: If your eslint is fixing files like `vite-env.d.ts` maybe you should add the following exception to your eslint file. `ignorePatterns: ['*.d.ts'],`. This is because eslint will try to fix the `///` into `// /`.

3. [Optional] 
    Install the `vite-plugin-eslint` package afterwards into your project devDependencies.
    
    `yarn add vite-plugin-eslint --dev`
    
    or

    `npm install vite-plugin-eslint --save-dev`

    Note: This package was misbehaving on vite version 5.3.4, that is why we are installing it later. If it's not working for you just ignore this or just work linter itself.

3. [Optional]
    Add the following to your `vite.config.ts` file

    ```TypeScript
    import { defineConfig } from 'vite';
    import react from '@vitejs/plugin-react';
    import eslint from 'vite-plugin-eslint';

    // https://vitejs.dev/config/
    export default defineConfig({
        plugins: [
            react(),
            eslint(),
        ],
    });
    ```

    Note: If importing from `vite-plugin-eslint`, is giving you problems with TypeScript, but the project is still running you can consider ignoring them. Currently the package as yet to fix its problems with TypeScript. If you really want to remove the warning you can remove the following rule `"strict": true` from `tsconfig.node.json`.

### Environment variables

To add different environment variables for different releases, then you can create the some env files for your purposes.

1. 
    Example of env files:
    ```
    .env.development // variables that are set in localhost
    .env.develop
    .env.localdev
    .env.release
    .env.master
    ```

    All these files should contain all the variables that change depending on the environment.
    Things like api url, api keys, external services, etc. Make sure that all these variables start with `VITE`.

    ```
    VITE_COMMIT=0
    VITE_API_URL=0
    ```
2. 
    To use these variables we need to create the following file `src/settings.ts` and use them as such:

    ```Typescript
    export const VERSION = import.meta.env.VITE_COMMIT;
    export const API_URL = import.meta.env.VITE_API_URL;
    ```
3. 
    Install the `env-cmd` package:

    `yarn add env-cmd` or `npm install env-cmd`

4. 
    Make sure you update your `scripts` section in your `package.json`:
    ```json
    {
        ...
        "scripts": {
            ...
            "build:develop": "tsc -b && vite build --mode develop",
            "build:localdev": "tsc -b && vite build --mode localdev",
            "build:release": "tsc -b && vite build --mode release",
            "build:master": "tsc -b && vite build --mode master",
        }
        ...
    }
    ```

5. [Optional]
    If you have setup your project to run tests, you might want to mock how your env variables are set in your project. 

    To achieve this either mock every test in the `src/setupTests.ts` or mock in each specific test suite.

    ```Typescript
    jest.mock('./settings', () => ({
        get API_URL() {
            return 'http://test-environment-url.com';
        }
    }));
    ```

### SASS
1. 
    Install `sass` package

    `yarn add sass` or `npm install sass`

2. 
    Create the following file `src/assets/styles/main.scss` or `src/assets/styles/main.sass`

3. 
    Import your created `scss` or `sass` file into your `src/main.tsx`

    ```Typescript
    import '.assets/styles/main.scss'
    import '.assets/styles/main.sass' // in case your file is SASS
    ```

You can now add new `scss` or `sass` files with your own styles.

Note: If you still have your `.css` files, you can now move those styles into your `scss` or `sass` files and delete the old ones, and remove any imports to them.

### Deployed directory

By default the out directory for production is `/dist`. 
If you want to define a new out directory (e.g: `/build`), just open your `vite.config.ts` file and add the following to your `defineConfig`.

```TypeScript
export default defineConfig({
  plugins: [...],
  build: {
    outDir: 'build',
  }
})
```

### .gitignore

Even though vite already adds a `.gitignore` file with some default values. It doesn't ignore some common files and directories, that you might want to ignore adding to your repo. So here's my revamped list.

You can add/remove other files and folders that are irrelevant to your repo.

```
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*
/coverage

# Dependencies
/node_modules
/.pnp
.pnp.js

# Production
dist
dist-ssr
build


# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?
*.local
```
