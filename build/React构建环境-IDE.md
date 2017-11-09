# React构建环境-IDE.md
React开发推荐使用的开发工具用：`Visual Studio Code`

## 常用插件
- ESLint
- EditorConfig for Visual Studio Code
- Reactjs code snippets
- React Standard Style snippets
- JS-CSS-HTML Formatting


## 常用功能
- JSX 中启用 HTML EMMENT 功能

    文件 -> 首选项 -> 设置
    ```
    {
        "emmet.triggerExpansionOnTab": true, //默认false
        "emmet.syntaxProfiles": { "javascript": "jsx" }
    }
    ```

- ESLint 自动修复问题

    ctrl + shift + p -> 输入：eslint -> 选择 ：Eslint: Fix all auto-fixable Problems