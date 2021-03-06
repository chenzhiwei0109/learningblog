---
sidebarDepth: 2
---

# vuecli

## 资源路径处理规则

当你在 `JavaScript`、`CSS `或 `*.vue `文件中使用相对路径 (必须以 . 开头) 引用一个静态资源时，该资源 将被webpack处理。

### 转换规则

- 如果 URL 是一个绝对路径 (例如 `/images/foo.png`)，它将会被保留不变。他会去当前服务器的public文件下去查找。

  ```html
  <img alt="Vue logo" src="/assets/logo.png">
  ```

- 如果 URL 以 `.` 开头，它会作为一个相对模块请求被解释且基于你的文件系统中的目录结构进行解析。

- 如果 URL 以 `~` 开头，其后的任何内容都会作为一个模块请求被解析。这意味着你甚至可以引用 Node 模块中的资源：

  ```html
  <img src="~some-npm-package/foo.png">
  ```

- 如果 URL 以 `@` 开头，它也会作为一个模块请求被解析。它的用处在于 Vue CLI 默认会设置一个指向 `<projectRoot>/src` 的别名 `@`。**(仅作用于模版中)**

### 何时使用 public 文件夹

任何放置在 `public` 文件夹的静态资源都会被简单的复制，而不经过 webpack。你需要通过绝对路径来引用它们。

注意我们推荐将资源作为你的模块依赖图的一部分导入，这样它们会通过 webpack 的处理并获得如下**好处**：

- 脚本和样式表会被压缩且打包在一起，从而避免额外的网络请求。
- 文件丢失会直接在编译时报错，而不是到了用户端才产生 404 错误。
- 最终生成的文件名包含了内容哈希，因此你不必担心浏览器会缓存它们的老版本。

**使用场景：**

- 你需要在构建输出中指定一个文件的名字。
- 你有上千个图片，需要动态引用它们的路径。
- 有些库可能和 webpack 不兼容，这时你除了将其用一个独立的 `<script>` 标签引入没有别的选择。

`public` 目录提供的是一个**应急手段**，当你通过绝对路径引用它时，留意应用将会部署到哪里。

如果你的应用没有部署在域名的根部，那么你需要为你的 URL 配置 [publicPath](https://cli.vuejs.org/zh/config/#publicpath) 前缀：

```js
// vue.config.js
module.exports = {
    publicPath: process.env.NODE_ENV === 'production'
    ? '/cart/'
    : '/'
}
```

- 在 `public/index.html` 或其它通过 `html-webpack-plugin` 用作模板的 HTML 文件中，你需要通过 `<%= BASE_URL %>` 设置链接前缀：

  ```html
  <link rel="icon" href="<%= BASE_URL %>favicon.ico">
  ```

- 在模板中，你首先需要向你的组件传入基础 URL：

  ```js
  data () {
    return {
      publicPath: process.env.BASE_URL
    }
  }
  ```

  然后：

  ```html
  <img :src="`${publicPath}my-image.png`">
  ```

## css相关

### 使用预处理器

```js
# Sass
npm install -D sass-loader node-sass
# Less
npm install -D less-loader less
# Stylus
npm install -D stylus-loader stylus
```

范例：App.vue修改为Sass

```html
<style scoped lang="scss">
    $color: #42b983;
    a {
        color: $color;
    }
</style>

```

### 自动化导入样式

```
npm i -D style-resources-loader
```

```js
//vue.config.js
const path = require('path')

function addStyleResource(rule) {
    rule.use('style-resource')
        .loader('style-resources-loader')
        .options({
        patterns: [
            path.resolve(__dirname, './src/styles/imports.scss'),
        ],
    })
}

module.exports = {
    chainWebpack: config => {
        const types = ['vue-modules', 'vue', 'normal-modules', 'normal']
        types.forEach(type =>
                      addStyleResource(config.module.rule('scss').oneOf(type)))
    },
}
```

### scoped css

深度作用选择器：使用 >>> 操作符可以使 scoped 样式中的一个选择器能够作用得“更深”，例如影响 子组件

```html
<style scoped>
    #app >>> a {
        color: red
    }
</style>
```

Sass 之类的预处理器无法正确解析 >>> 。这种情况下你可以使用 /deep/ 或 ::v-deep 操作符 取而代之

```html
<style scoped lang="scss">
#app {
/deep/ a {
color: rgb(196, 50, 140)
}
::v-deep a {
color: rgb(196, 50, 140)
}
}
</style>

```

### CSS Modules

[CSS Modules](https://github.com/css-modules/css-modules) 是一个流行的，用于模块化和组合 CSS 的系统。`vue-loader` 提供了与 CSS Modules 的一流集成，可以作为模拟 scoped CSS 的替代方案。

添加module

```html
<style module lang="scss">
    .red {
        color: #f00;
    }
    .bold {
        font-weight: bold;
    }
</style>

```

模板中通过$style.xx访问

```
<a :class="$style.red">awesome-vue</a>
<a :class="{[$style.red]:isRed}">awesome-vue</a>
<a :class="[$style.red, $style.bold]">awesome-vue</a>
```

JS中访问

```html
<script>
    export default {
        created () {
            // -> "red_1VyoJ-uZ"
            // 一个基于文件名和类名生成的标识符
            console.log(this.$style.red)
        }
    }
</script>
```

## 接口相关

设置开发服务器代理选项可以有效避免调用接口时出现的跨域问题。

```js
devServer: {
    proxy: 'http://localhost:3000'
}
```

## 路径问题汇总

- 普通的js引入使用@即可

  ```
  import Home from '@/views/home/Home.vue'
  ```

- 如果路径以 ~ 开头，其后的部分将会被看作模块依赖，既可以加载含有别名的静态资源，又可以加载node-modules中的资源

  ```
  <img src="~some-npm-package/foo.png">
  ```

  ```
  @import url(~@/assets/styles/reset.css);
  ```

  ```
  <img src="~[npm包名]/xxx/logo.png" alt="">
  ```

- 路径懒加载+js分离chunk

  ```
  component: () => import(/* webpackChunkName: "about" */ '@/views/list/List.vue')
  ```
