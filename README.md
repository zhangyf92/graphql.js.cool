# 参与贡献

该项目Github Pages部署在 `Master` 分支, 英文源码在 `source` 分支, 中文源码在 `cn` 分支.

网站是用 `js` 和 `markdown` 进行编写, 文件存放在 `site/` 目录, 核心代码位于 `site/_core/`.

### 如何进行改动

安装项目依赖:

```bash
npm install
# 或
yarn
```

然后执行:

```bash
npm start
# 打开 http://localhost:8444/
```

在你改动内容之后, 只需要刷新浏览器就能够观察到变化.

### 发布到网站

`cn` 分支通过 Travis CI 自动部署到 https://graphql.js.cool/
