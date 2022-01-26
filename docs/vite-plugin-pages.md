官方文档传送门：[vite-plugin-pages](https://github.com/hannoeru/vite-plugin-pages) 
## 一、作用
配合vue-router动态生成路由信息，也就是可以根据对应的文件结构来生成路由信息，无需手动配置路由表。
## 二、基础用法

``` 
import Pages from "vite-plugin-pages";
export default {
  plugins: [
    Pages(),
  ],
};
```
## 三、配置属性
大部分配置都在vite.config.ts->defineConfig->plugins->Pages中。
- dirs：指定需要生成路由的文件夹
	- 类型：string | array
	- 默认值：'src/pages'

可以自行修改路由文件夹
```
Pages({
  dirs: [
   { dir: 'src/pages', baseRoute: '' },
   // src/views/pages文件夹下会生成/views/filename这样的路由
   { dir: 'src/views/pages', baseRoute: 'views' },
   // 会识别components下多个分类下pages的文件
   { dir: 'src/components/**/pages', baseRoute: 'components' },
  ],
})

```
- extensions：指定需要生成路由的文件

```
Pages({
  // 识别带有vue和md后缀的文件为路由（md文件需要有插件支持）
  extensions: ['vue', 'md'],
}

```
- importMode: 加载方式
	- 类型：'sync' | 'async' | (string) => 'sync' | 'async'

可以直接设定全局的路由加载方式，也可以通过设置`syncIndex`配置项来转换同步加载<br>
也可以通过传入方法，返回`sync` | `async`来确定加载方式
```
Pages({
  importMode: 'async',
  // 包含test的路由，就会变为异步懒加载
  // importMode(path) {
  //   return path.includes('test') ? 'async' : 'sync'
  // },
})

```
- routeBlockLang：路由块语言类型
	- 类型：string
	- 默认值：'json5'

使用vite-plugin-pages自动化路由后，路由信息不再写在config配置文件中，因此在SFC文件中可以通过route标签来填写必要的路由信息，而routeBlockLang是route路由信息的解析方式。
```
Pages({
  routeBlockLang: 'json5',
})

```
默认json5方式：
```
<route>
{
  meta: {
    layout: 'test'
  }
}
</route>

```
指定yaml方式编写route代码块
```
<route lang="yaml">
meta:
  layout: 'test'
</route>

```
此处route代码块中路由信息可以通过route对象访问，如`route.meta.layout`。<br>

**扩展**<br>
route代码块中layout属性通常结合插件`vite-plugin-vue-layouts`在页面路由的基础上实现动态布局功能，即在路由时可按照layout配置切换布局，将页面嵌入对应的test布局文件中。
- extendRoute：route处理方法
	- 类型：function

开发者可以通过这个方法获取到单个路由对象，并对其进行一定的操作，如可以设置layout等
```
      extendRoute(route) {
        // console.log(route.path)
        // 测试此方法的效果，判断路由中包含了字符串的，更换layout
        if (route.path.includes('test')) {
          return {
            ...route,
            meta: { layout: 'test' },
          }
        }
        return route
      },

```
- onRoutesGenerated：routes处理方法
	- 类型：function

通过此方法可以获取到完整的路由array，可循环遍历对route进行处理
```
      // 整体处理整个routes的信息，然后进行相应的处理
      onRoutesGenerated(routes) {
        const temp_routes = JSON.parse(JSON.stringify(routes))
        temp_routes.forEach((item: any) => {
          // 这里依然是判断路由中包含了test字符串的，更换layout
          if (item.path.includes('test')) {
            item.meta = {
              layout: 'test',
            }
          }
        })
        return temp_routes
      },

```
- onClientGenerated：clientCode处理方法
	- 类型： function

可以获取到插件渲染之后的代码，这里开发者可以通过字符串操作，对代码进行最后的拦截处理。
```
      // 这里涉及到更改客户端代码
      onClientGenerated(clientCode) {
        // 能够完整获取到我们实际生成的路由的完整代码，string格式的，
        // 感觉可以通过正则对这个方法进行替换，或者各种字符串骚操作进行替换
        return clientCode
      },

```