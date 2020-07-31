<template>
  <div id="app">
    <p>{{title}}</p>
    <button @click="addMovie">addmovies</button>
  </div>
</template>


<script>
import { addmovie } from './graphql/mutations'
import  { hello,movies } from './graphql/query/hello.js'
export default {
  data(){
    return{
      title:"",
      movies:[]
    }
  },
  apollo:{
    movies
  },
  mounted(){
    this.getName()
  },
  methods:{
    getName(){
      this.$apollo.addSmartQuery('hello',{
        query:hello,
        variables:{
          name:"jiaxin"
        },
        result(res){
          this.title = res.data.hello
        },    
      })
    },
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
  }
}
</script>


<style lang="stylus">
#app
  font-family Avenir, Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  text-align center
  color #2c3e50
  margin-top 60px
</style>
