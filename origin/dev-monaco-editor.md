# VSCode文本编辑器Monaco开发

# 一、简介

# 二、源代码构建



# 三、开发

## 1、导入

```bash
<script src="monaco-editor/min/vs/loader.js"></script>
```

## 2、初始化

```js
editor = monaco.editor.create(container, {
    value: "",                      // 初始化时编辑器里的内容
    language: language,             // 指定语法高亮的语言（如 "javascript", "json", "html" 等）
    theme: "vs-dark",               // 编辑器主题，"vs", "vs-dark", "hc-black" 等
    contextmenu: true,              // 是否启用右键菜单
    quickSuggestions: true,         // 输入时是否启用快速建议（如代码补全提示）
    wordBasedSuggestions: true,     // 是否启用基于文档中已有单词的建议
    snippetSuggestions: "inline",   // 代码片段（snippets）建议显示方式："top"、"bottom"、"inline" 或 "none"
    autoClosingTags: true,          // 是否自动补全 HTML/XML 标签
    autoClosingBrackets: "always",  // 自动补全括号，"always"（总是），"never"（从不），"languageDefined"（按语言规则）
    autoClosingQuotes: "always",    // 自动补全引号，同上
    minimap: { enabled: false },    // 是否显示右侧小地图（代码缩略图）
    folding: true,                  // 是否启用代码折叠功能
    foldingStrategy: "indentation", // 代码折叠的策略："auto" 或 "indentation"
    showFoldingControls: "always",  // 折叠控件（小三角）的显示策略："always" 或 "mouseover"
    formatOnPaste: true,            // 粘贴时自动格式化代码
});
```

## 3、注册右键功能

```js
editor.addAction({
    id: 'convertToUppercase',
    label: '将所选英文字符转为大写',
    keybindings: [],
    precondition: null,
    keybindingContext: null,
    contextMenuGroupId: 'navigation',
    contextMenuOrder: 1.5,
    run: function (ed) {
        const model = ed.getModel();
        const selection = ed.getSelection();
        if (!selection || selection.isEmpty()) { alert("请先选中要转换的文本"); return; }
        const text = model.getValueInRange(selection);
        // 仅转换英文
        const newText = text.replace(/[a-zA-Z]+/g, (m) => m.toUpperCase());
        ed.executeEdits("uppercase", [{
            range: selection,
            text: newText,
            forceMoveMarkers: true
        }]);
    }
});
```

