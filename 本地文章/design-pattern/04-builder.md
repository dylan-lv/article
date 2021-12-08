# 构造器（Builder）

将类型的创建构成抽象成各个部分。

举例1：

```js
造车() {
  造发动机()
  造轮子() * 4
  造内饰()
  ...
}
```

举例2：JSX（作用：精简程序）

```tsx
<Page>
	<TitleBar />
  <Tabs>
  	<Tab title='首页' icon=...>...</Tab>
  	<Tab title='发现' icon=...>...</Tab>
  	<Tab title='个人中心' icon=...>...</Tab>
  </Tabs>
</Page>
```





