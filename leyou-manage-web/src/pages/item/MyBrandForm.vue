<template>
  <!-- -->
  <v-form v-model="valid" ref="myBrandForm">
    <v-layout wrap >
      <v-flex xs12>
        <v-text-field v-model="brand.name" label="名称" required :rules="nameRules"></v-text-field>
      </v-flex>
      <v-flex xs12>
        <v-text-field v-model="brand.letter" label="首字母" required :rules="letterRules"></v-text-field>
      </v-flex>

      <v-flex xs12>
        <v-cascader
          url="/item/category/list"
          multiple
          required
          v-model="brand.categories"
          label="请选择商品分类"/>
      </v-flex>

      <v-flex xs3>
        <span style="font-size: 16px; color: #444">品牌LOGO：</span>
      </v-flex>
      <v-flex xs9>

        <v-upload
          v-model="brand.image"
          url="/upload/image"
          :multiple="false"
          :pic-width="250"
          :pic-height="90"
        />
      </v-flex>

      <v-spacer></v-spacer>
      <v-flex >
        <v-btn color="blue darken-1" flat @click="clear">重置</v-btn>
        <v-btn color="blue darken-1" flat @click="submit">提交</v-btn>
      </v-flex>
    </v-layout>

    <small>*为必填字段</small>
  </v-form>


</template>

<script>
    export default {
      name: "my-brand-form",
      props:{
        oldBrand:{
          type: Object
        },
        isEdit:{
          type:Boolean,
          default:false

        }
      },
      data(){
          return{
            valid:false,
            brand:{
              name:'', // 品牌名称
              letter:'', // 品牌首字母
              image:'',// 品牌logo
              categories:[], // 品牌所属的商品分类数组
            },
            nameRules:[
              v => !!v || "品牌名称不能为空",
              v => v.length > 1 || "品牌名称至少2位"
            ],
            letterRules:[
              v => !!v || "首字母不能为空",
              v => /^[A-Z]{1}$/.test(v) || "品牌字母只能是A~Z的大写字母"
            ],
          }
      },
      methods:{

        // 提交表单方法
        submit(){
          //表单校验
          if (this.$refs.myBrandForm.validate()){
            //定义一个请求参数对象
            const {categories ,letter ,...params} = this.brand;
            // 数据库中只要保存分类的id即可，因此我们对categories的值进行处理,只保留id，并转为字符串
            params.cids = categories.map(c => c.id).join(",");
            // 将字母都处理为大写
            params.letter = letter.toUpperCase();
            // 将数据提交到后台
            // this.$http.post('/item/brand', this.$qs.stringify(params))
            this.$http({
              method: this.isEdit ? 'put' : 'post',
              url: '/item/brand',
              data: this.$qs.stringify(params)
            }).then(() => {
              // 关闭窗口
              this.$emit("close");
              this.$message.success("保存成功！");
            })
              .catch(() => {
                this.$message.error("保存失败！");
              });

          }
        },
        // 重置方法
        clear(){
          this.$refs.myBrandForm.reset();
          this.brand.categories=[]
        }
      },
      watch:{
        oldBrand: {// 监控oldBrand的变化
          handler(val) {
            if(val){
              // 注意不要直接复制，否则这边的修改会影响到父组件的数据，copy属性即可
              this.brand =  Object.deepCopy(val)
            }else{
              // 为空，初始化brand
              this.brand = {
                name: '',
                letter: '',
                image: '',
                categories: [],
              }
            }
          },
          deep: true
        }
      }


    }
</script>

<style scoped>

</style>
