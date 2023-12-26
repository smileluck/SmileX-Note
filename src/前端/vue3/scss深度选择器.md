# 前言

在带有scoped属性的style标签中，无法影响到子组件的样式，此时我们会使用到深度选择器，来处理该问题。

# 使用和现象

我们可能常见的直接使用!important是不生效的。

```html
<style lang="scss" scoped>
.editor {
  ol {
    list-style: auto !important;
  }
}
</style>
```

或者使用/deep/。这时会抛出异常，不支持/deep/选择。

```html
<style lang="scss" scoped>
.editor {
  /deep/ ol {
    list-style: auto !important;
  }
}
</style>
```

# 解决办法

使用::v-deep替代，能够解决scss中无法解析/deep/的问题。

```html
<style lang="scss" scoped>
.editor {
  ::v-deep ol {
    list-style: auto !important;
  }
}
</style>
```

或者使用 :deep(选择器)替代

```html

<style lang="scss" scoped>
.editor {
  :deep(ol){
    list-style: auto !important;
  }
}
</style>
```