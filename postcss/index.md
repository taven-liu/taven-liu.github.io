# postcss
PostCSS是一个使用JavaScript插件来转换CSS的工具,PostCSS本身很小，其只包含CSS解析器，操作CSS节点树的API，资源生成器，以及一个节点树字符串化工具。

## 插件
postcss所有的黑魔法都是通过利用插件实现的。目前postcss的生态圈的插件非常丰富，利用这些插件可以大大的提高工作效率。

### postcss-cssnext
通过此插件我们可以直接用下一代`css`语法来书些我们的`css`

- 自定义属性
```
:root {
  --mainColor: red;
}

a {
  color: var(--mainColor); /* color: red; */
}
```
> 后面可结合`postcss-import`使用，将自定义属性提取到单独文件

- 计算属性
```
:root {
  --fontSize: 1rem;
}

h1 {
  font-size: calc(var(--fontSize) * 2); /* font-size: 2rem; */
}
```
- 自定义媒体查询
```
/* 申明 */
@custom-media --only-medium-screen (width >= 500px) and (width <= 1200px);

@media (--only-medium-screen) { /* @media (min-width:500px) and (max-width:1200px) */
  /* your styles */
}
```

- 自定义选择器
```
@custom-selector :--button button, .button;
@custom-selector :--enter :hover, :focus;

:--button {
  
}
/* 转义后
  button,
  .button {
  }
*/

:--button:--enter {
}
/* 转义后
  button:hover,
  .button:hover,
  button:focus,
  .button:focus {
  }
*/
```
- 嵌套
```
a {
  /* 直接书写a标签的样式 */
  color: red;
  
  /* a标签下的span标签的样式 */
  & span {
    color: white;
  }

  /* span标签下a标签的样式 */
  @nest span & {
    color: blue;
  }

  /* 媒体查询 */
  @media (min-width: 30em) {
    color: yellow;
  }
}
```
```
转义后
```
```
a {
  /* 直接书写a标签的样式 */
  color: red
  
  /* a标签下的span标签的样式 */
}
a span {
  color: white;
}
a {
  /* span标签下a标签的样式 */
}
span a {
  color: blue;
}
a {
  /* 媒体查询 */
}
@media (min-width: 30em) {
  a {
    color: yellow;
  }
}
```

> 详细见[postcss-cssnext](https://github.com/MoOx/postcss-cssnext)

### postcss-import
通过此插件我们可以将一些公共的样式抽取到一个文件中，通过`import`方式导入。
```
css/theme.css
```
```
.success {
  color: green;
}

.error {
  color: red;
}

.warning {
  color: yellow;
}

```
```
css/style.css
```
```
@import "./theme.css";

.foo {
  color: black;
}

```
> 详细见 [postcss-import](https://github.com/postcss/postcss-import)

### postcss-apply
通过此插件我们可以自定义css属性
```
:root {
  --toolbar-theme: {
    background-color: rebeccapurple;
    color: white;
    border: 1px solid green;
  };
}

.Toolbar {
  @apply --toolbar-theme;
}
```
通常我们结合`postcss-import`一起使用并将申明的属性单独存放在一个文件中，在需要的`css`文件中通过`import`引入
> 详细见 [postcss-apply]https://github.com/pascalduez/postcss-apply)

### postcss-mixins
通过此插件我们可以轻松的在`css`中书写函数
```
/* 申明一个mixin */
@define-mixin clearfix {
  zoom: 1;
  &:before,
  &:after {
    content: " ";
    display: table;
  }
  &:after {
    clear: both;
    visibility: hidden;
    font-size: 0;
    height: 0;
  }
}

/* 使用 */
.foo {
  color: red;
  background-color: var(--bg);
  @mixin clearfix;
}
```
带参`mixin`
```
/* 申明一个mixin */
@define-mixin icon $name {
  padding-left: 16px;
  background-url: url(/icons/$(name).png);
}

/* 使用 */
.test {
  color: red;
  @mixin icon search;
}

/*
  转义后
  .test{
    color:red;
    padding-left:16px;
    background-url:url(/icons/search.png)
  }
*/
```

## webpack集成
使用`postcss-loader`集成
```
/* loaders添加如下loader */
{
  test: /\.css$/,
  exclude: /node_modules/,
  use: [
    'style-loader',
    'css-loader',
    {
      loader: 'postcss-loader',
      options: {
        sourceMap: true,
        sourceComments: true,
        plugins: [
          require('postcss-import')(),
          require('postcss-apply')(),
          require('postcss-cssnext')(),
          require('postcss-mixins')()
        ]
      }
    }
  ]
}

```
