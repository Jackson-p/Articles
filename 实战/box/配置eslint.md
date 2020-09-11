### eslint for React + ts

对技术栈为React+ts的项目配置eslint，在统一代码规范下，避免发生一些低级错误。

```sh
npm install @typescript-eslint/eslint-plugin @typescript-eslint/parser babel-eslint eslint eslint-plugin-react husky lint-staged -D
```

__pakage.json中__

在scripts加入

```sh
"eslint": "eslint --fix --ext .tsx,.ts"
```

然后在最后加上

```json
"lint-staged": {
    "*.{ts,tsx}": [
      "npm run eslint",
      "git add"
    ]
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  }
```

新建eslint.rc

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  extends: [
    'plugin:@typescript-eslint/recommended', // Uses the recommended rules from the @typescript-eslint/eslint-plugin
    'plugin:react/recommended'
  ],
  parserOptions: {
    ecmaFeatures: {
      jsx: true
    },
    ecmaVersion: 2018,
    sourceType: 'module'
  },
  plugins: [
    'react',
    '@typescript-eslint'
  ],
  rules: {
    'no-trailing-spaces': 'error',
    'eol-last': ['error', 'always'],
    'comma-dangle': ['error', 'always-multiline'],
    indent: ['error', 2, { SwitchCase: 1 }],
    'no-multi-spaces': 'error',
    'max-len': ['warn', { code: 100 }],
    'quote-props': ['error', 'as-needed'],
    quotes: ['error', 'single'],
    'jsx-quotes': ['error', 'prefer-double'],
    'object-curly-spacing': ['error', 'always'],
    'react/display-name': 'off',
    eqeqeq: 'error',
    'no-var': 'error',
    'prefer-destructuring': ["error", { "object": true, "array": false }],
    'prefer-const': 'warn',
    'object-shorthand': 'error',
    "comma-spacing": [2, {"before": false, "after": true}],
    "key-spacing": [2, {"beforeColon": false, "afterColon": true}],
    'semi': 2,
    'no-console': 1,
    'no-debugger': 2,
    'no-catch-shadow': 2,
    "switch-colon-spacing": ["error", {"after": true, "before": false}],
    'keyword-spacing': 2,
    'id-length': [2, { min: 1, max: 30 }],
    'yield-star-spacing': [2],
    'react/prop-types': 0,
    '@typescript-eslint/ban-ts-ignore': 0,
    '@typescript-eslint/explicit-function-return-type': 0,
    '@typescript-eslint/no-explicit-any': ['off'],
    '@typescript-eslint/no-var-requires': 1,
    'react/react-in-jsx-scope': 1
  },
};

```

其实配了eslint,往往大家都会提到的都有prettier，这个并没有安装包，并配置在工程里面，而是通过vscode的prettier插件来进行约束。

最后可以配下vscode配置文件，实现的效果是：保存时eslint会自动修正错误部分，如果需要对代码格式化 -> command + shift + p 选择Format Document即可。

配置文件如下

```json
{
    "editor.fontSize": 24,
    "window.zoomLevel": 0,
    "terminal.integrated.fontSize": 24,
    "workbench.editor.enablePreview": false,
    "fileheader.customMade": {
        "Author": "xxx",
        "Description": "",
        "Date": "Do not edit",
        "LastEditTime": "Do not edit"
    },
    "fileheader.configObj": {
        "autoAdd": false,
    },
    "files.defaultLanguage": "typescript",
    "workbench.iconTheme": "material-icon-theme",
    "workbench.colorTheme": "Community Material Theme Darker High Contrast",
    "editor.tabSize": 2,
    "eslint.alwaysShowStatus": true,
    "prettier.singleQuote": true,
    "prettier.printWidth": 100,
    "prettier.semi": true,
    "[typescriptreact]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[less]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[typescript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode",
    },
    "diffEditor.renderSideBySide": true,
    "[javascript]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "editor.suggestSelection": "first",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "[json]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "javascript.updateImportsOnFileMove.enabled": "always",
    "[vue]": {
        "editor.defaultFormatter": "octref.vetur"
    },
    "javascript.implicitProjectConfig.experimentalDecorators": true,
    "editor.formatOnSave": false,
    "[html]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
}
```





[参考的文章](https://www.jianshu.com/p/fed0fbf95172)