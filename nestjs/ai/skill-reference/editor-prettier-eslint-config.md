# Configure EditorConfig, Prettier and ESLint

Set up editor rules, code formatting and linting to maintain consistency across the project. These three tools work together:

- **EditorConfig** tells your editor (VSCode, WebStorm, etc.) how to display and insert indentation.
- **Prettier** auto-formats your code on save or when running `npm run format`.
- **ESLint** lints your code and runs Prettier as a rule via `eslint-plugin-prettier`.

All three must agree on indentation style and width, otherwise they will conflict with each other.

## 1. Configure EditorConfig

EditorConfig defines editor-level rules at the project level. NestJS does not create this file by default, so you need to create it manually.

### 1.1. Create `.editorconfig`

Create a `.editorconfig` file in the root of your project:

> .editorconfig
```ini
root = true

[*.ts]
indent_style = tab
indent_size = 4
end_of_line = lf
charset = utf-8
trim_trailing_whitespace = true
insert_final_newline = true
```

### 1.2. What does each rule do?

| Rule | Value | Description |
|------|-------|-------------|
| `indent_style` | `tab` | Use tab characters for indentation |
| `indent_size` | `4` | Each tab is displayed as 4 columns wide |
| `end_of_line` | `lf` | Use Unix-style line endings |
| `charset` | `utf-8` | Use UTF-8 encoding |
| `trim_trailing_whitespace` | `true` | Remove trailing whitespace on save |
| `insert_final_newline` | `true` | Add a newline at the end of files |

## 2. Configure Prettier

Prettier controls how code is auto-formatted. Without explicit configuration, Prettier uses its defaults (spaces, width 2), and running `npm run format` would replace all tabs with spaces.

### 2.1. Update `.prettierrc`

Add `useTabs` and `tabWidth` to the existing `.prettierrc` file:

> .prettierrc
```json
{
  "singleQuote": true,
  "trailingComma": "all",
  "useTabs": true,
  "tabWidth": 4
}
```

### 2.2. What do the new properties do?

| Property | Value | Description |
|----------|-------|-------------|
| `useTabs` | `true` | Prettier uses tab characters instead of spaces when indenting |
| `tabWidth` | `4` | Each tab equals 4 columns of width |

## 3. Configure ESLint

NestJS projects use `eslint-plugin-prettier`, which runs Prettier as an ESLint rule. Among other rules, the `prettier/prettier` rule accepts inline options that **override** what `.prettierrc` says. By default, it only has `{ endOfLine: "auto" }`, so ESLint uses Prettier defaults (spaces, width 2) for everything else, causing format conflicts that show up as lint errors.

### 3.1. Update `eslint.config.mjs`

Replace the default `eslint.config.mjs` rules with the following:

> eslint.config.mjs
```js
// ... other config

{
  rules: {
    '@typescript-eslint/no-explicit-any': 'warn',
    '@typescript-eslint/no-floating-promises': 'warn',
    '@typescript-eslint/no-unsafe-argument': 'warn',
    '@typescript-eslint/no-unsafe-call': 'off',
    "prettier/prettier": ["error", { endOfLine: "auto", useTabs: true, tabWidth: 4 }],
  },
}
```

## 4. Reformat the Project

After applying all the changes above, reformat the codebase so every file follows the new rules:

```bash
npm run format
```

Optionally, run the linter to catch any residual errors:

```bash
npm run lint
```
