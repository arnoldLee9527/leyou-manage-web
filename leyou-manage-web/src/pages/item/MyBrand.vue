<template>
  <div>
    <v-card>

      <v-card-title>
        <!--新增start-->
        <v-btn slot="activator" color="primary" @click="addBrand()">新增</v-btn>

        <v-dialog v-model="dialog" persistent max-width="600px">
          <v-card>
            <v-card-title>
              <span class="headline">{{isEdit ? '修改':'新增'}}品牌</span>
              <v-spacer/>
              <v-btn icon @click="closeWindow" ><v-icon color="red">close</v-icon></v-btn>
            </v-card-title>
            <v-card-text>

              <!--引入表单内容-->
              <my-brand-form @close="closeWindow" :oldBrand="oldBrand" :isEdit="isEdit"></my-brand-form>

            </v-card-text>

            <v-card-actions>

            </v-card-actions>
          </v-card>
        </v-dialog>

        <!--新增end-->

        <v-spacer/>

        <!--搜索start-->
        <v-flex xs12 sm6>
          <v-text-field
            v-model="key"
            label="请输入关键字"
            clearable
            append-icon="search"
            hide-details
          ></v-text-field>
        </v-flex>
        <!--搜索end-->

      </v-card-title>

      <v-divider/>

      <v-card-text>
        <v-data-table
          :headers="headers"
          :items="brands"
          :pagination.sync="pagination"
          :total-items="totalBrands"
          :loading="loading"
          class="elevation-1"
        >
          <v-progress-linear slot="progress" color="blue" indeterminate></v-progress-linear>
          <template slot="items" slot-scope="props">
            <td class="text-xs-center">{{ props.item.id }}</td>
            <td class="text-xs-center">{{ props.item.name }}</td>
            <td class="text-xs-center"><img :src="props.item.image"></td>
            <td class="text-xs-center">{{ props.item.letter }}</td>
            <td class="text-xs-center" >
              <v-btn icon @click="editItem(props.item)"  ><v-icon small color="primary"> edit </v-icon></v-btn>
              <v-btn icon @click="deleteItem(props.item)" ><v-icon small color="red"> delete </v-icon></v-btn>
            </td>
          </template>
        </v-data-table>
      </v-card-text>

    </v-card>


  </div>

</template>

<script>
    import VSpec from "./specification/Specification";
    import MyBrandForm from "./MyBrandForm"
    import Icon from "iview/src/components/icon/icon";
    export default {
      name: "my-brand",
      components: {
        Icon,
        VSpec,
        MyBrandForm
      },
      data() {
        return {
          key:"",
          totalBrands: 0,
          loading: true,
          brands: [],
          pagination: {},
          headers: [
            {text: 'id', align: 'center', sortable: true, value: 'id'},
            {text: '名称', align: 'center', sortable: false, value: 'name'},
            {text: 'Logo', align: 'center', sortable: false, value: 'image'},
            {text: '首字母', align: 'center', sortable: true, value: 'letter'},
            {text: '修改', align: 'center', sortable: false, value: 'letter'},
          ],
          dialog:false,
          oldBrand:{},
          isEdit:false
        }
      },
      mounted(){ // 渲染后执行
        // 查询数据
        this.getDataFromServer();

      },
      watch:{
        //监控页码数据变化
        pagination:{
          deep:true,
          handler() {
            // 变化后的回调函数，这里我们再次调用getDataFromServer即可
            this.getDataFromServer();
          }
        },
        key:{
          handler(){
            this.getDataFromServer();
          }
        },

      },
      methods:{
        getDataFromServer(){
          this.$http.get("/item/brand/page",{
              params:{
                key: this.key, // 搜索条件
                page:this.pagination.page,//当前页
                rowsPerPage:this.pagination.rowsPerPage,//每页显示条数
                sortBy:this.pagination.sortBy, //排序字段
                desc:this.pagination.descending //是否降序
              }
          }
            )
            .then(resp=>{
              console.log(resp)
              this.brands = resp.data.items
              this.totalBrands = resp.data.total

              this.loading = false
            })
            .catch(resp => {
              console.log("数据查询失败")
            })
        },
        addBrand(){
          //修改标记
          this.isEdit = false;
          this.dialog = true;
        },
        closeWindow(){
          //关闭窗口
          this.dialog = false;
          //重新加载
          this.getDataFromServer();
          this.oldBrand=null
        },
        editItem (oldBrand) {
          this.$http.get("/item/category/bid/"+oldBrand.id)
            .then(({data})=>{
              //标记是否是修改
              this.isEdit = true;
              this.dialog = true
              this.oldBrand = oldBrand
              this.oldBrand.categories = data;
          })

        },
        deleteItem(oldBrand){
          confirm("确定删除id为"+oldBrand.id+"，名称为"+oldBrand.name+"的商品吗？")
          this.$http.delete("/item/brand/delete/"+oldBrand.id)
            .then(()=>{
              this.getDataFromServer();
              this.$message.success("删除成功");
            })
            .catch(()=>{
              this.$message.error("删除失败")
            })

        }
      }


    }
</script>

<style scoped>

</style>
