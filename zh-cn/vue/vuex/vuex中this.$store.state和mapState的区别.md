> this.$store.state和mapState都可以获取到state中的数据，只是mapState不能修改，所以如果项目中只展示该数据的话，可以使用mapState，如果要修改state中的数据的话使用mapState,所以input中的v-model就不能使用mapState中的参数

下面是错误示范：
```
<template>
    <div>
    <!--这样是错的，因为mapState中的值不可修改-->
        <el-input v-model='name'>
        </el-input>
    </div>
</template>
<script>
export default {
  name: 'RegisterInfo',
  data () {},
  computed: {
      ...mapState(['name'])
  }
}
</script>
```
下面是正确示范
```
<template>
    <div>
        <el-input v-model='name'>
        </el-input>
    </div>
</template>
<script>
export default {
  name: 'RegisterInfo',
  // 使用this.$store.state
  data () {
    var name = this.$store.state.name
    return {
    // 这是es6简写，等价于name: name
      name
    }
  }
}
</script>
```