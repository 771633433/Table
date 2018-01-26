<template>
    <div>
        <div class="bar">
            <!--  左侧的search查询结果图标  -->
            <div class="searchIcon" @click="searchResult">
                <p class="p_search">查询</p>
                <p class="p_icon">
                    <Icon type="search" color='#284e8d' size='21' />
                </p>               
            </div>

            <!--  批量删除  -->
            <div class="batchDeletion" @click="batchDelete">
                <p class="p_delete">删除</p>
                <p class="p_icon_delete">
                    <Icon type="android-delete" color='#284e8d' size='21' />
                </p>
            </div>
            <!--  右侧的3个按钮  -->
                <div class="menau">
                    <!--    添加按钮,可以添加   -->
                    <div class="add" @click='addMes' title="新增">
                        <i class="ivu-icon ivu-icon-plus-circled" style="font-size: 18px;color:#284e8d;line-height: 30px;"></i>
                    </div>
                    <!--  导入按钮  -->
                    <div class="importFile" @click='importData' title="导入">
                        <i class="ivu-icon ivu-icon-ios-cloud-download" style="font-size: 18px;color:#284e8d;line-height: 32px;"></i>
                    </div>
                    <!--  导出    -->
                    <div class="exportExcel" @click="exportData (1)" title="导出">
                        <i class="ivu-icon ivu-icon-ios-cloud-upload" style="font-size: 18px;color:#284e8d;line-height: 31px;"></i>
                    </div>
                </div>
        </div>
        <!--  写一个模态框,用于用户查询记录  -->
        <Modal v-model="searchRecord"
            title="查询结果"
            @on-ok="getResult"
            @on-cancel="cancel">
            <p>用户id:<input type="text" v-model='userid' @keyup.13="getResult" name=""></p>
            <br/>
            <!-- <p>性别: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <select v-model='sex' style="width: 150px;">
                    <option value="" selected="true" disabled="true">请选择</option>
                        <option>男</option>
                        <option>女</option>
                </select>
            </p> -->
            <br/>
            <!-- <p>身份证号: &nbsp;<input type="text" v-model='idNumber' name=""></p>   -->     
        </Modal>
        <!--  写一个模态框,用于用户新增记录  -->
        <Modal v-model="addRecord"
            title="新增记录"
            @on-ok="ok"
            @on-cancel="cancel">
            <p>用户名:&nbsp;&nbsp;&nbsp; &nbsp;<input type="text" v-model='personName' name=""></p>
            <br/>
            <p>性别: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <select v-model='sex' style="width: 150px;">
                    <option value="" selected="true" disabled="true">请选择</option>
                        <option>男</option>
                        <option>女</option>
                </select>
            </p>
            <br/>
            <p>身份证号: &nbsp;<input type="text" v-model='idNumber' name=""></p>       
        </Modal>

        <!--  再写一个模态框,用于用户编辑修改当前这条记录  -->
        <Modal v-model="modifyRecord"
            title="编辑记录"
            @on-ok="modify"
            @on-cancel="cancel">
            <p>用户名:&nbsp;&nbsp;&nbsp; &nbsp;<input type="text" v-model='modifyName' name=""></p>
            <br/>
            <p>性别: &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <select v-model='modifySex' style="width: 150px;">
                    <option value="" selected="true" disabled="true">请选择</option>
                    <option>男</option>
                    <option>女</option>
                </select>
            </p>
            <br/>
            <p>身份证号: &nbsp;<input type="text" v-model='modifyidNumber' name=""></p>       
        </Modal>
        <!--  上面是模态框的代码   -->
        <Table border highlight-row :columns="columns7" :data="data6" :width="1300" :height="600" style="margin:0 auto"  @on-selection-change="showMe" ref="table"></Table>
    </div>
</template>
<script>
import $ from 'jquery';
    export default {
        created(){
            // ajax 请求数据
            $.ajax({
                type: "GET",
                contentType: "application/json;charset=utf-8",
                url: 'http://172.20.22.43:8183/static-resource/userinfo?size=1000' ,
                dataType: "json",
                success:(res)=>{
                    //console.log(res.content.list);
                    this.data6=res.content.list;
                },
                error:(XMLHttpRequest, textStatus, errorThrown)=>{
                    alert("服务调用失败，请检查输入字段或联系管理员");
                }
            });
        },
        data () {
            return {
                searchRecord:false,
                addRecord:false,
                modifyRecord:false,
                sex:'',
                personName:'',
                idNumber:'',
                modifyName:'',
                modifySex:'',
                modifyidNumber:'',
                modifyindex:'',
                userid:'',
                data6: [],
                columns7: [
                    {
                        type: 'selection',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: 'id',
                        key: 'id'
                    },
                    {
                        title: '姓名',
                        key: 'name'
                    },
                    {
                        title: '身份证号',
                        key: 'userName'
                    },
                    {
                        title: '状态',
                        key: 'cstate'
                    },
                    {
                        title: '操作',
                        key: 'action',
                        width: 150,
                        align: 'center',
                        render: (h, params) => {
                            return h('div', [
                                h('Button', {
                                    props: {
                                        type: 'primary',
                                        size: 'small'
                                    },
                                    style: {
                                        marginRight: '5px'
                                    },
                                    on: {
                                        click: () => {
                                            this.show(params.index);
                                        }
                                    }
                                }, '编辑'),
                                h('Button', {
                                    props: {
                                        type: 'error',
                                        size: 'small'
                                    },
                                    on: {
                                        click: () => {
                                            this.remove(params.index);
                                            this.$Message.info('删除成功！');
                                        }
                                    }
                                }, '删除')
                            ]);
                        }
                    }
                ],
            }
        },
        methods: {
            // 左侧查询按钮点击事件
            searchResult(){
                // 弹出模态框
                this.searchRecord=true;
            },
            // 查询按钮点击后,出来的那个模态框输入了用户id,点击确定后调用的事件
            getResult(){
                console.log(this.userid); // this.userid 是用户在input框里面输入的具体内容

                // ajax请求数据,将查询到的结果全部显示到页面
                $.ajax({
                    type: "GET",
                    contentType: "application/json;charset=utf-8",
                    url: 'http://172.20.22.43:8183/static-resource/userinfo?size=1000'+'&'+'id='+this.userid,
                    dataType: "json",
                    success:(res)=>{
                        //console.log(res.content.list);
                        this.data6=res.content.list;
                        this.$Message.info('查询成功！');
                    },
                    error:(XMLHttpRequest, textStatus, errorThrown)=>{
                        alert("服务调用失败，请检查输入字段或联系管理员");
                    }
                });
                // ajax请求查询数据到此结束
            },
            // 用户点击编辑的时候
            show (index) {
                this.modifyRecord=true;
                console.log(this.data6[index].id );
                this.modifyindex=index;
            },
            remove (index) {
                // 删除当前这条记录 
                //console.log(index);
                $.ajax({
                    type: "DELETE",
                    contentType: "application/json;charset=utf-8",
                    async:false,
                    url: 'http://172.20.22.43:8183/static-resource/userinfo?id='+this.data6[index].id ,
                    dataType: "json",
                    success:(res)=>{
                        console.log(res);
                    },
                    error:(XMLHttpRequest, textStatus, errorThrown)=>{
                        alert("服务调用失败，请检查输入字段或联系管理员");
                    }
                });

                this.data6.splice(index, 1);         
            },
            // 导出事件

            exportData (type) {
                if (type === 1) {
                    this.$refs.table.exportCsv({
                        filename: 'Export'
                    });
                } else if (type === 2) {
                    this.$refs.table.exportCsv({
                        filename: 'Sorting and filtering data',
                        original: false
                    });
                } else if (type === 3) {
                    this.$refs.table.exportCsv({
                        filename: 'Custom data',
                        columns: this.columns8.filter((col, index) => index < 4),
                        data: this.data7.filter((data, index) => index < 4)
                    });
                }
            },
            // 新增记录按钮
            addMes(){
                this.addRecord=true;
            },  

            // 新增用户信息后,用户点击确定按钮后的事件
            ok(){
                this.modal=false;    
                this.$Message.info('添加成功！');  
                //console.log(this.personName,this.sex,this.idNumber);   

                // 调用post请求,将用户填写的信息添加到数据库
                $.ajax({
                    type: "POST",
                    contentType: "application/json;charset=utf-8",
                    dataType: "json",   
                    url: 'http://172.20.22.43:8183/static-resource/userinfo' ,
                    data: JSON.stringify(
                        {
                            name:this.personName,
                            userName:this.idNumber
                        }
                    ),
                    success:(res)=>{
                        //console.log(res);
                        this.data6.unshift({
                            cstate:'',
                            id:"",
                            name:this.personName,
                            userName:this.idNumber
                        });
                    },
                    error:(XMLHttpRequest, textStatus, errorThrown)=>{
                        alert("服务调用失败，请检查输入字段或联系管理员");
                    },
                });    
       
           
            },
            // 在模态框中,用户点击了取消按钮的事件
            cancel () {
                //this.$Message.info('点击了取消');
            },
            //  点了编辑出了模态框后,用户点击确定的事件
            modify(){
                // this.data6[this.modifyindex].id 是点击编辑的时候,这一行数据的id
                // 通过id,提交post请求修改数据
                console.log(this.data6[this.modifyindex].id);
            },

            // 从外面导入 excel
            importData(){

            },

            // 点击选择时候触发的事件
            showMe(name){
                 console.log(name);
            },

            // 批量删除用户选中的条目
            batchDelete(){

            }
        }
    }
</script>


<style type="text/css">
.bar{
    width: 1300px;
    height: 30px;
    margin: 0 auto;
    margin-bottom: 2px;
    /*border: 1px solid gray;*/
}

.menau{
    width: 10%;
    height: 30px;
    float: right;
    /*border: 1px solid gray;*/
}

.searchIcon{
    width: 6%;
    height: 30px;
    line-height: 44px;
    margin-left: 8px;
    cursor: pointer;
    /*border: 1px solid red;*/
    float: left;
}

/*   添加新数据  */
.add{
    width: 20px;
    height: 30px;
    line-height: 30px;
    float: left;
    cursor: pointer;
    /*border: 1px solid gray;*/
}

/*   导入 */
.importFile{
    width: 20px;
    height: 30px;
    float: left;
    margin-left: 22px;
    cursor: pointer;
    /*border: 1px solid gray;*/
}

.exportExcel{
    width: 20px;
    height: 30px;
    float: left;
    margin-left: 22px;
    cursor: pointer;
}

.p_icon{
    width: 22px;
    height: 30px;
    line-height: 45px;
    /*border: 1px solid gray;*/
    float: left;
}

.p_search{
    width: 28px;
    height: 30px;
    font-size: 13px;
    line-height: 35px;
    color: #284e8d;
    font-weight: 900;
    /*border: 1px solid gray;*/
    float: left;
}

.batchDeletion{
    width: 80px;
    height: 30px;
    cursor: pointer;
    /*border: 1px solid gray;*/
    float: left;
}


.p_delete{
    width: 28px;
    height: 30px;
    font-size: 13px;
    line-height: 35px;
    color: #284e8d;
    font-weight: 900;
    /*border: 1px solid gray;*/
    float: left;
}

/*  删除那个垃圾桶一样的图标  */
.p_icon_delete{
    width: 18px;
    height: 30px;
    font-size: 13px;
    line-height: 38px;
    color: #284e8d;
    font-weight: 900;
    /*border: 1px solid gray;*/
    float: left;
}
</style>
