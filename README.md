## vue结合Graphql

### 安装依赖

```js
yarn add vue-apollo graphql apollo-boost
```
### 创建实例

```js
import ApolloClient from 'apollo-boost'

const apolloClient = new ApolloClient({
  // 你需要在这里使用绝对路径
  uri: 'http://localhost:4000/graphql'
})

export default apolloClient
```

### 安装插件到 Vue

```js
import VueApollo from 'vue-apollo'
import apolloClient from './client'
Vue.use(VueApollo)
const apolloProvider = new VueApollo({
  defaultClient: apolloClient,
})
new Vue({
  router,
  store,
  apolloProvider, //注册
  render: h => h(App)
}).$mount('#app')
```

### 使用

- 查询：

> 这里有两种，一种是带参数的，第二种属于不带参数的查询：

```js
//不带参数的查询
//query的index文件
import gql from 'graphql-tag'
export let movies = gql`query{
    movies{
      id
      title
      genres
      rating
    }
}`
//App.vue文件
import  { movies } from './graphql/query'
export default {
  data(){
    return{
      movies:[]
    }
  },
  apollo:{
    movies
  },
}
//这里注意两个地方：
① 组件在首次加载的时候首先会获取一次请求，所以我们不用做任何操作就会发现data中含有数据。
② 是apollo里面对应的字段必须在data的对象中有一个相同名字，为什么：因为当组件开始加载的时候就首先会触发一次请求获取数据，我们会发现获取到的数据{data:{movies:...}}这种结构，其中里面的movies对应data里面的movies，所以不会做任何操作就会赋值到data中。 
```
> 带参数并且data中的字段和query中的字段是不相同的：

```js
//query的index文件
import gql from 'graphql-tag'
export let hello = gql`query ($name:String){
    hello(name:$name)
}`
//App.vue文件
import  { hello } from './graphql/query/hello.js'
export default {
  data(){
    return{
      title:"",//注意看这里和hello中的query的字段是不一样的，所以如果我们按照第一种不带参数的写法会发现会报错。
    }
  },
  mounted(){
    this.getName()
  },
  methods:{
    getName(){
      //addSmartQuery 手动添加一个智能查询
      this.$apollo.addSmartQuery('hello',{
        query:hello,
        //variables 用来传递参数
        variables:{
          name:"xinxin"
        },
        result(res){
          //我们在result中获取
          this.title = res.data.hello
        },    
      })
    }
  }
}
//这里注意的地方除了以上添加的内容之外还有
addSmartQuery 中的第一个参数一定要和query中的字段名对应，不然报Missing xxx attribute on result
//这样我们就完成了一个带参数的查询，并且是自己想什么时候查询就什么时候查询，并且是data中的字段和query中的字段是不同的操作
```

- 变更

> 变更操作主要是增删改的操作，相对应schema的mutation字段

```js
import gql from 'graphql-tag'
export let addmovie =gql`mutation ($title:String!,$genres:String!,$rating:Float,$theater:Int){
  insert(title:$title,genres:$genres,rating:$rating,theater:$theater){
    title,
    genres,
    rating,
    theater
  }
}`
addMovie(){
    this.$apollo.mutate({
    // 查询语句
    mutation: addmovie,
    // 参数
    variables: {
            title:"英伦对决",
            genres:"悬疑，热血，男神",
            rating:8.8,
            theater:2
        },
    }).then((res)=>{
        console.log(res)
    }).catch(error=>{
        console.log(error)
    })
}
//这里目前来说是没有什么要说的点。。还没有遇到高危操作
```