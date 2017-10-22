<template>
    <div class="arttmpl">
         <!--面包屑 -->
         <el-row>
            <el-col :span="24">
                <div class="abread bt-line">
                    <!-- 使用elementUI中的面包屑导航组件完成 -->
                    <el-breadcrumb separator="/">
                        <el-breadcrumb-item :to="{ path: '/admin' }">首页</el-breadcrumb-item>
                        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
                        <el-breadcrumb-item>商品列表</el-breadcrumb-item>
                    </el-breadcrumb>
                </div>
            </el-col>
         </el-row>
         <!--新增，删除，搜索栏 -->
         <div class="operation">
            <el-row>
                <el-col :span="5">
                    <!-- 新增，删除，全选按钮 -->
                    <el-button>全选</el-button>
                    <el-button>新增</el-button>
                    <el-button>删除</el-button>
                </el-col>
                <!--偏移16格 -->
                <el-col :span="3" :offset="16">
                    <!-- 搜索框 -->
                    <el-input placeholder="请输入搜索条件" icon="search" v-model="searchValue" :on-icon-click="getlist">
                    </el-input>
                </el-col>
            </el-row>
         </div>
        <!-- 数据列表-->
        <el-row>
            <el-col :span="24">
                <!-- 绑定数据的来源：属性名data关联到Vue对象的list属性   style：表格的大小  row-class-name控制表格颜色，bind可以绑定一个方法，会有返回值-->
                <!-- 自动遍历：list-->
                <el-table :data="list" style="width: 100%"   :row-class-name="tableRowClassName"  @selection-change="handleSelectionChange">

                        <!-- table列表的每一列：-->


                        <!--prop对应关联的数据对象list里面的属性。其中全选框，选中整个date，所有数据 -->

                        <el-table-column prop="date"  width="80">
                            <!--当前列的内容区域-->
                            <el-table-column
                                  type="selection"
                                  width="55">
                            </el-table-column>
                        </el-table-column>

                        <el-table-column prop="title" label="标题">
                            <!-- 点击会跳转路由。-->
                            <template scope="scope">
                                <router-link v-bind="{to:'/admin/goodsedit/'+scope.row.id}">
                                    {{scope.row.title}}
                                </router-link>
                            </template>
                        </el-table-column>

                        <el-table-column prop="categoryname" label="类别" width="100">
                        </el-table-column>

                        <el-table-column label="发布人/发布时间" width="200">

                            <!--为什么要添加template标签呢？还有scope是什么东西？ -->
                            <template scope="scope" width="150">
                                {{scope.row.user_name }} / {{scope.row.add_time | date("YYYY-MM-DD")}}
                            </template>
                        </el-table-column>

                        <el-table-column prop="name" label="属性" width="120">
                            <!--表格里的多结构都要使用template，不仅仅是需要多个属性-->

                            <template scope="scope">
                                <el-tooltip class="item" effect="dark" v-bind='{content:(scope.row.is_slide==0?"进入轮播图":"取消轮播图")}'  placement="bottom">
                                    <i v-bind='{class:"ls el-icon-picture"+(scope.row.is_slide==1?" imgactive":"")} ' ></i>
                                </el-tooltip>
                                <el-tooltip class="item" effect="dark" v-bind='{content:(scope.row.is_top==0?"进入轮播图":"取消轮播图")}'  placement="bottom">
                                     <i v-bind='{class:"ls el-icon-upload"+(scope.row.is_top==1?" imgactive":"" )}'  ></i>
                                </el-tooltip>
                                <el-tooltip class="item" effect="dark" v-bind='{content:(scope.row.is_hot==0?"进入轮播图":"取消轮播图")}'  placement="bottom">
                                     <i v-bind='{class:"ls el-icon-star-on"+(scope.row.is_hot==1?" imgactive":"") }' ></i>
                                </el-tooltip>


                            </template>
                        </el-table-column>

                        <el-table-column label="操作" width="80">
                            <template scope="scope">
                                    <router-link  v-bind="{to:'/admin/goodsedit/'+scope.row.id}">
                                           <el-button type="success" size='mini'>编辑</el-button>
                                    </router-link>
                            </template>
                        </el-table-column>
                </el-table>
            </el-col>
        </el-row>
    </div>
</template>
<script>
    export default {
        data() {
            return {
                // 搜索框的绑定属性
                searchValue: '',
                // 表格中的每行数据来源于list，而这个list将来是通过getlist()方法请求后台api接口获取到的
                list: [],
                //勾选上的ID字符串列表
                idlist:''
            }
        },
        //在组件被创建的生命周期时候，就去请求数据然后渲染出来。
        created() {
            // 获取到列表数据
            this.getlist();
        },
        methods: {
            //选择会执行整个方法,参数selection是这一行的数据。
            handleSelectionChange(selection){
                //获取所有的勾选的ID

                this.idlist = '';
                var _this = this;
                selection.forEach(function(value,index,arr){
                      _this.idlist += value.id + ',';
                })
                this.idlist = this.idlist.slice(0,-1);
                //获取到选中ID的字符串。

            },

            // 用axios去发出具体的url的请求获取到数据后绑定到表格中
            getlist() {

                // 1.0 获取url
                var url = '/admin/goods/getlist?pageIndex=1&pageSize=';
                //this.$http = axios对象。
                this.$http.get(url).then(res => {
                    // 判断服务器返回的状态status
                    if (res.data.status == 1) {
                        this.$message.error(res.data.message);
                        return;
                    }

                    // 正常逻辑的处理,把请求回来的数据绑定到model层的list中。
                    this.list = res.data.message;
                });
            },
            // 控制表格的隔行变色
            tableRowClassName(row, index) {
                //    控制奇数和偶数行的颜色
                if (index % 2 == 0) {
                    return 'info-row';
                } else {
                    return 'positive-row';
                }
            }
        }
    }
</script>
<style scoped>

</style>