# React18 + Vite项目配置

## 项目初始化

### 代码规范配置

```shell
pnpm create vite # 通过vite创建项目
pnpm add -D eslint-config-prettier eslint-plugin-prettier prettier # 配置eslint + prettier 依赖项
pnpm add -D @trivago/prettier-plugin-sort-imports # import 排序
pnpm add -D stylelint-config-standard stylelint-order stylelint-config-css-modules # stylelint检测css代码规范
pnpm add -D lint-staged # 针对暂存的 git 文件运行 linter
```

```js
// eslint.config.js 配置prettier
{
   extends: [
    ...
    'plugin:prettier/recommended'
  ],
  rules: {
    'prettier/prettier': 'error',
  }
}

```

```js
// V8 .eslintrc.cjs
module.exports = {
  root: true,
  env: { browser: true, es2020: true },
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
    'plugin:prettier/recommended',
  ],
  ignorePatterns: ['dist', '.eslintrc.cjs'],
  parser: '@typescript-eslint/parser',
  plugins: ['react-refresh'],
  rules: {
    'prettier/prettier': 'error',
    'react-refresh/only-export-components': [
      'warn',
      { allowConstantExport: true },
    ],
  },
}
```

> 升级Eslint到v9版本之后，有以下区别点：
>
> 1. 文件名的变化，由后缀可以看出其差别
>
>    - eslint.config.js
>    - eslint.config.mjs
>    - eslint.config.cjs
>
> 2. 配置方式的变化：采用扁平化的方式处理【这是为了兼容之前的配置方式的方法，新方法还没研究】
> 3. 不再需要.eslintignore文件来忽略

```js
// eslint.config.js
import { FlatCompat } from '@eslint/eslintrc'
import js from '@eslint/js'

import path from 'path'
import { fileURLToPath } from 'url'

// mimic CommonJS variables -- not needed if using CommonJS
const __filename = fileURLToPath(import.meta.url)
const __dirname = path.dirname(__filename)

const compat = new FlatCompat({
  baseDirectory: __dirname, // optional; default: process.cwd()
  resolvePluginsRelativeTo: __dirname, // optional
  recommendedConfig: js.configs.recommended, // optional unless you're using "eslint:recommended"
  allConfig: js.configs.all, // optional unless you're using "eslint:all"
})
export default [
  ...compat.extends(
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:react-hooks/recommended',
    'plugin:prettier/recommended',
  ),
  ...compat.env({ browser: true, es2020: true }),
  ...compat.plugins('react-refresh'),
  ...compat.config({
    ignorePatterns: ['dist', '.eslintrc.cjs'],
    parser: '@typescript-eslint/parser',
    rules: {
      'prettier/prettier': 'error',
      'react-refresh/only-export-components': [
        'warn',
        { allowConstantExport: true },
      ],
    },
  }),
  {
    ignores: [
      '**/dev/*',
      '**/dist/*',
      '**/tests/*',
      'tsconfig.json',
      '**/node_modules/*',
    ],
  },
]
```

```json
// 配置import 排序
{
  ...,
  "importOrder": ["^react", "^react-router-dom", "^@", "^[./]"],
  "importOrderSeparation": true,
  "importOrderSortSpecifiers": true,
  "plugins": ["@trivago/prettier-plugin-sort-imports"]
}
```

##### stylelint配置

```js
/** @type {import('stylelint').Config} */
export default {
  processors: [],
  plugins: ['stylelint-order'],
  extends: ['stylelint-config-standard', 'stylelint-config-css-modules'],
  rules: {
    'selector-class-pattern': [
      // 命名规范 -
      '^([a-z][a-z0-9]*)(-[a-z0-9]+)*$',
      {
        message: 'Expected class selector to be kebab-case',
      },
    ],
    'at-rule-empty-line-before': null,
    'at-rule-no-unknown': null,
    'length-zero-no-unit': true, // 禁止零长度的单位（可自动修复）
    'shorthand-property-no-redundant-values': true, // 简写属性
    'declaration-block-no-duplicate-properties': true, // 禁止声明快重复属性
    'no-descending-specificity': true, // 禁止在具有较高优先级的选择器后出现被其覆盖的较低优先级的选择器。
    'selector-max-id': 0, // 限制一个选择器中 ID 选择器的数量
    'max-nesting-depth': 3,
    'order/properties-order': [
      // 规则顺序
      'position',
      'top',
      'right',
      'bottom',
      'left',
      'z-index',
      'display',
      'float',
      'width',
      'height',
      'max-width',
      'max-height',
      'min-width',
      'min-height',
      'padding',
      'padding-top',
      'padding-right',
      'padding-bottom',
      'padding-left',
      'margin',
      'margin-top',
      'margin-right',
      'margin-bottom',
      'margin-left',
      'margin-collapse',
      'margin-top-collapse',
      'margin-right-collapse',
      'margin-bottom-collapse',
      'margin-left-collapse',
      'overflow',
      'overflow-x',
      'overflow-y',
      'clip',
      'clear',
      'font',
      'font-family',
      'font-size',
      'font-smoothing',
      'osx-font-smoothing',
      'font-style',
      'font-weight',
      'line-height',
      'letter-spacing',
      'word-spacing',
      'color',
      'text-align',
      'text-decoration',
      'text-indent',
      'text-overflow',
      'text-rendering',
      'text-size-adjust',
      'text-shadow',
      'text-transform',
      'word-break',
      'word-wrap',
      'white-space',
      'vertical-align',
      'list-style',
      'list-style-type',
      'list-style-position',
      'list-style-image',
      'pointer-events',
      'cursor',
      'background',
      'background-color',
      'border',
      'border-radius',
      'content',
      'outline',
      'outline-offset',
      'opacity',
      'filter',
      'visibility',
      'size',
      'transform',
    ],
  },
}
```

##### lint-staged配置

```json
// package.json
 "lint-staged": {
    "*.{js,jsx,ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.json": [
      "prettier --write",
      "git add"
    ],
    "*.vue": [
      "eslint --fix",
      "prettier --write",
      "stylelint --fix",
      "git add"
    ],
    "*.{css,scss,less,html}": [
      "stylelint --fix",
      "prettier --write",
      "git add"
    ],
    "*.md": [
      "prettier --write",
      "git add"
    ]
  },
```

## git规范制定

采用基本的：`husky` + `commitizen` + `cz-git`

```shell
pnpm install -g commitizen # 全局安装 commitizen,可以快速使用 cz 或 git cz 命令进行启动。
#pnpm add --save-dev @commitlint/{cli,config-conventional} # 下载commitlint #这个其实可以不用 在cz-git的情况下
#echo "export default { extends: ['@commitlint/config-conventional'] };" > commitlint.config.js # 执行该命令创建对应的配置文件
pnpm add --save-dev cz-git commitizen
pnpm add --save-dev husky # 下载husky git hooks
pnpm husky init # 初始化命令，由 husky install换成了init
npm pkg set scripts.commitlint="commitlint --edit" # package.json中添加脚本命令
echo "pnpm commitlint \${1}" > .husky/commit-msg # commit-msg 钩子中添加执行命令
```

1. 配置`cz-git`

```json
// package.json
 "config": {
    "commitizen": {
      "path": "node_modules/cz-git"
    }
  }
```

2. 创建`.commitlintrc.js`进行规则配置

```js
/** @type {import('cz-git').UserConfig} */
export default {
  rules: {
    'type-enum': [
      2,
      'always',
      [
        'build',
        'chore',
        'ci',
        'docs',
        'feat',
        'fix',
        'perf',
        'refactor',
        'revert',
        'style',
        'test',
      ],
    ],
    'subject-empty': [2, 'never'],
    'subject-case': [2, 'never', ['sentence-case']],
  },
  prompt: {
    alias: { fd: 'docs: fix typos' },
    messages: {
      type: "Select the type of change that you're committing:",
      scope: 'Denote the SCOPE of this change (optional):',
      customScope: 'Denote the SCOPE of this change:',
      subject: 'Write a SHORT, IMPERATIVE tense description of the change:\n',
      body: 'Provide a LONGER description of the change (optional). Use "|" to break new line:\n',
      breaking:
        'List any BREAKING CHANGES (optional). Use "|" to break new line:\n',
      footerPrefixesSelect:
        'Select the ISSUES type of changeList by this change (optional):',
      customFooterPrefix: 'Input ISSUES prefix:',
      footer: 'List any ISSUES by this change. E.g.: #31, #34:\n',
      generatingByAI: 'Generating your AI commit subject...',
      generatedSelectByAI: 'Select suitable subject by AI generated:',
      confirmCommit: 'Are you sure you want to proceed with the commit above?',
    },
    types: [
      {
        value: 'feat',
        name: 'feat:     ✨  A new feature',
        emoji: ':sparkles:',
      },
      { value: 'fix', name: 'fix:      🐛  A bug fix', emoji: ':bug:' },
      {
        value: 'docs',
        name: 'docs:     📝  Documentation only changes',
        emoji: ':memo:',
      },
      {
        value: 'style',
        name: 'style:    💄  Changes that do not affect the meaning of the code',
        emoji: ':lipstick:',
      },
      {
        value: 'refactor',
        name: 'refactor: ♻️   A code change that neither fixes a bug nor adds a feature',
        emoji: ':recycle:',
      },
      {
        value: 'perf',
        name: 'perf:     ⚡️  A code change that improves performance',
        emoji: ':zap:',
      },
      {
        value: 'test',
        name: 'test:     ✅  Adding missing tests or correcting existing tests',
        emoji: ':white_check_mark:',
      },
      {
        value: 'build',
        name: 'build:    📦️   Changes that affect the build system or external dependencies',
        emoji: ':package:',
      },
      {
        value: 'ci',
        name: 'ci:       🎡  Changes to our CI configuration files and scripts',
        emoji: ':ferris_wheel:',
      },
      {
        value: 'chore',
        name: "chore:    🔨  Other changes that don't modify src or test files",
        emoji: ':hammer:',
      },
      {
        value: 'revert',
        name: 'revert:   ⏪️  Reverts a previous commit',
        emoji: ':rewind:',
      },
    ],
    useEmoji: true,
    emojiAlign: 'center',
    useAI: false,
    aiNumber: 1,
    themeColorCode: '',
    scopes: [],
    allowCustomScopes: true,
    allowEmptyScopes: true,
    customScopesAlign: 'bottom',
    customScopesAlias: 'custom',
    emptyScopesAlias: 'empty',
    upperCaseSubject: false,
    markBreakingChangeMode: false,
    allowBreakingChanges: ['feat', 'fix'],
    breaklineNumber: 100,
    breaklineChar: '|',
    skipQuestions: [],
    issuePrefixes: [
      { value: 'closed', name: 'closed:   ISSUES has been processed' },
    ],
    customIssuePrefixAlign: 'top',
    emptyIssuePrefixAlias: 'skip',
    customIssuePrefixAlias: 'custom',
    allowCustomIssuePrefix: true,
    allowEmptyIssuePrefix: true,
    confirmColorize: true,
    scopeOverrides: undefined,
    defaultBody: '',
    defaultIssues: '',
    defaultScope: '',
    defaultSubject: '',
  },
}
```

3. 添加husky钩子

```shell
echo "pnpm run lint-staged" > .husky/pre-commit
echo "pnpm commitlint" > .husky/commit-msg
```

3.

## 安装依赖

### 该项目暂时依赖

- antd
- react-router v6
- zustand
- tailwindcss
