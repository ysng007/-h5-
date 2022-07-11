##	简单h5页面的国际化
###	背景
对于APP内部嵌套的一些交互比较少甚至是没有交互的页面，只是一个展示的页面，去单独搭建一个脚手架是一种浪费，于是采用静态页面引入插件的方式去做国际化处理是一个较好的选择。
### 安装i18next
```
yarn add i18next or npm install i18next
```
###目录结构
```
├── index.html                    // 静态页面
├── node_modules              // i18next包，后面可以删除，只留下i18next.min.js
├── package.json
├── src
│   ├── i18				 // 国际化文件
│   │   ├── de.json
│   │   ├── en.json
│   │   └── fr.json
│   ├── i18next.min.js         // i18next压缩包
│   └── index.js			// js国际化相关逻辑
└── yarn.lock
```
###文件主要内容
`index.html`

```
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HomeKit</title>
    <script src="./src/i18next.min.js"></script>
</head>

<body>
    <div class="i18nelement" data-i18n="key">Loading</div>
    <div>
        <label>Change language:</label>
        <select disabled="true" id="langSelector" name="lang">
            <option value="en">English</option>
            <option value="de">German</option>
            <option value="fr">French</option>
        </select>
    </div>
    <script src="src/index.js"></script>
</body>

</html>
```
`index.js`

```
function updateContent() {
    const elements = document.getElementsByClassName("i18nelement")
    for (let i = 0; i < elements.length; i++) {
        const element = elements[i]
        const k = element.getAttribute("data-i18n")
        element.innerHTML = i18next.t(k)
    }
}
async function i18Loader() {
    const langs = ["en", "fr", "de"]
    const jsons = await Promise.all(
        langs.map((l) => fetch("src/i18/" + l + ".json").then((r) => r.json()))
    )
    const res = langs.reduce((acc, l, idx) => {
        acc[l] = { translation: jsons[idx] }
        return acc
    }, {})
    await i18next.init({
        lng: "en",
        debug: true,
        resources: res
    })
    updateContent()
    i18next.on("languageChanged", () => {
        updateContent()
    })
    const langSelector = document.getElementById("langSelector")
    langSelector.removeAttribute("disabled")
    langSelector.addEventListener("change", (e) => {
        i18next.changeLanguage(e.target.value)
    })
}

i18Loader()
```
