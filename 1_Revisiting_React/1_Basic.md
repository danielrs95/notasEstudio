# Basic

## ESLint

ESLint is a JS linting open source project and is used to _find problems and syntax issues_ in your code. Some popular options are

- `no-unused-vars`
- `no-extra-bind`
- `keyword-spacing`
- `comma-style`

## Prettier

Prettier is a code style formatter, different from ESLint, Prettier only format the code style, and does not look for syntax problems. Some popular options are

- `max-len`
- `no-mixed-spaces-and-tabs`
- `no-extra-bind`
- `no-implicit-globals`

1. Linters usually contain not only _code quality rules_, but also _stylistic rules_. Most stylistic rules are unnecessary whe using Prettier (and also, might conflict with Prettier)

2. To turn off rules use package --> `eslint-config-prettier`

3. Runs Prettier as an ESLint rule and report --> `eslint-plugin-prettier`

4. If we using some standard (like airbnb) there's no need to use our own `.prettierrc` config, maybe if we want to override something

## Husky

Husky runt before a commit or push. Basically use it to link your commit messages, run test, lint code, etc. Supports Git hooks

## lint-staged

Run linters against staged git files