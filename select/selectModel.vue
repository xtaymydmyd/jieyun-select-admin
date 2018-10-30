<template>
    <Modal
        v-model="rangeModel"
        :title="config.title"
        class="visible-range"
        width = "800px"
        @on-cancel="cancel">
        <div class="visible-range-container1 flex flex-direction-column">
            <div class="visible-range-result">
                <Tag 
                    v-for="(item , indexTree) in count" 
                    :key="indexTree" 
                    :color="item.type == 0 ? 'green' : item.type  == 1 ? 'yellow' : 'blue'"
                    type="border"
                    :name="item.label" 
                    closable 
                    @on-close="deleteTreeTag(item)">{{item.label}}</Tag>
            </div>
            <div class="visbile-range-list-wrap flex flex-1">
                <div class="range-left flex flex-direction-column" >
                    <div style="height: 350px; " class="flex flex-direction-column">
                        <div class="tab-header">
                            <Tabs :value="tabName" size="small" @on-click="changeTab">
                                <TabPane label="部门" name="depart" v-if="condition.depart"></TabPane>
                                <TabPane label="岗位" name="post" v-if="condition.post"></TabPane>
                                <TabPane label="群组" name="group" v-if="condition.group"></TabPane>
                            </Tabs>
                        </div>
                        <div class="tab-container flex-1" >
                            <div v-if="tabName == 'depart'" class="tree">
                                <el-tree 
                                    :data="completeOrgTree" 
                                    v-loading="loadingTree" 
                                    element-loading-text="加载中..." 
                                    :show-checkbox="config.type == 0"
                                    :default-checked-keys="checkedKeys" 
                                    :check-on-click-node="false"
                                    :expand-on-click-node="false"
                                    :props="defaultProps" 
                                    @check-change="handlechange"
                                    @node-click="nodeClick"
                                    :load="loadNode"
                                    empty-text="暂无数据" 
                                    node-key="id"
                                    ref="tree" 
                                    lazy
                                    check-strictly
                                    :render-content="renderContent"
                                    ></el-tree>
                            </div>
                            <div v-if="tabName == 'post'">
                                <Table 
                                    size="small"
                                    ref="table"
                                    :height="295"
                                    :columns="postColumn" 
                                    :data="postList"
                                    :loading = postLoading
                                    @on-row-click = "currentRow"></Table>
                            </div>
                             <div v-if="tabName == 'group'">
                                <Table 
                                    size="small"
                                    :height="305"
                                    :columns="groupColumn" 
                                    :data="groupList"
                                    :loading = groupLoading
                                    @on-row-click = "currentGroupRow"
                                    ></Table>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="range-right flex flex-direction-column flex-1" style="width:346px;">
                    <div class="search flex flex-align-items">
                        <Tag type="dot" 
                            closable 
                            color="blue" 
                            v-if="(node.id && type == 0)||(node.id && type == 1)"
                            @on-close="clearTag(0)">{{node.label}}</Tag>
                        <Tag type="dot" 
                            closable 
                            color="green" 
                            v-if="(postCurrent.id && type == 0) || (postCurrent.id && type == 1)"
                            @on-close="clearTag(0)">{{postCurrent.name}}</Tag>
                            {{groupCurrent.id}}
                            {{type}}
                        <Tag type="dot" 
                            closable 
                            color="yellow" 
                            v-if="(groupCurrent.groupId && type == 0) || (groupCurrent.groupId && type == 1)"
                            @on-close="clearTag(0)">{{groupCurrent.name}}</Tag>
                        姓名: &nbsp;
                        <div class="flex-1">
                            <Input 
                                v-model="fuzzy" 
                                placeholder="请输入姓名" 
                                @on-keydown.enter="searchByKeyword"></Input>
                        </div>&nbsp;&nbsp;
                        <Button type="primary" @click="searchByKeyword">搜索</Button>
                    </div>
                    <div class="flex-1" style="margin-top:10px;">
                        <Table 
                            ref="selection" 
                            highlight-row
                            :columns="columns" 
                            :data="searchList" 
                            height="260" 
                            size = small
                            :loading="loading"
                            @on-row-click ="onSearchRowClick"
                            ></Table>
                    </div>
                    <div class="pages flex flex-justify-content-right margin-top-24">
                        <Page 
                            :total="total" 
                            :current ="pageNum"
                            :page-size = pageSize
                            @on-page-size-change="onPageSizeChange"
                            @on-change="onPageChange" size="small" show-elevator show-sizer />
                    </div>
                </div>
            </div>
        </div>
        <div slot="footer">
            <Button @click="cancel">取消</Button>
            <Button type="primary" :disabled="count.length == 0" @click="rangeOk">确定</Button>
        </div>
    </Modal>
</template>

<script>

export default {
    data () {
        return {
            title : '',
            type : '',
            showTiele : false,
            tabValue : '1',
            highlight : true,
            count : [],
            loading : false,
            loadingTree : false,
            completeOrgTree:[],
            fuzzy: '',
            pageNum: 1,
            total : 1,
            columns: [
                {
                    title: '账号',
                    key: 'account'
                },
                {
                    title: '姓名',
                    key: 'name'
                },
                {
                    title: '部门',
                    key: 'deptName'
                }
            ],
            searchList : [],
            childrenKey : 'children',

            checkedKeys: [],
            defaultProps: {
                label: 'label',
                children: 'children',
                isLeaf: 'isLeaf'
            },
            deleteStatus : false,
            rangeModel : false,
            postColumn: [{
                title: '岗位名称',
                key: 'name'
            }],
            postCurrent : {
                id : ''
            },
            postList : [],
            status : '',
            pageSize : 10,
            activeIndex : 0,
            node : {
                id : ''
            },
            condition : {
                depart : false,
                post : false,
                group : false
            },
            tabName : 'depart',
            postLoading : false,
            groupColumn: [{
                title: '群组名称',
                key: 'name'
            }],
            groupCurrent : {
                id : ''
            },
            groupList : [],
            groupLoading : false,
        };
    },
    props: {
        config : {
            type : Object,
            default : {
                rootNodeId : '',
                rootNodeType : '',
                title : '选择人员',
                type : 1,
                muliteChoice : 1
            }
        }
    },
    components : {},
    mounted(){
        
    },
    methods: {
        /**
         * 打开弹出框，并初始化数据
        */
        initSelect(count){
            this.count = count ? count : [];
            if(this.config.condition.length > 0){
                for(var i = 0 ; i < this.config.condition.length ; i++){
                    this.condition[this.config.condition[i]] = true;
                }
            }
            this.tabName = 'depart';
            this.activeIndex = 0;
            this.rangeModel = true;
            this.fuzzy = '';
            this.postCurrent.id = '';
            this.node.id = '';
            this.searchList = [];
            this.checkedKeys = [];
            // this.$refs.tree.setCheckedKeys([]);
            this.queryTreeRoot();
        },
        clearTag(type){
            this.node.id = '';
            this.postCurrent.id = '';
            this.fuzzy = ''
            this.pageNum = 1;
            this.total = 1;
            this.searchList = [];
        },
        changeTab(name){
            this.tabName = name;
            this.fuzzy = '';
            this.pageNum = 1;
            this.total = 1;
            this.searchList = [];
            switch(name){
                case 'depart':
                    this.activeIndex = 0;
                    this.postCurrent = {}
                    break;
                case 'post' : 
                    this.activeIndex = 1;
                    this.node = {};
                    this.queryPostList();
                    break;
                case 'group':
                    this.activeIndex = 2;
                    this.node = {};
                    this.queryGroupList();
                    break;
                default : 
                    break;

            }
        },
        /**
         * 获取所有群组列表
        */
        queryGroupList(){
            var url = constGlobal.HostContact + 'gorupList/searchByUserVisable' , _this = this;
            this.groupLoading = true;
            http.apiGet(url).then(res => {
                if (res.status == 0) {
                    this.groupLoading = false;
                    this.groupList = res.data;
                } else {
                    common.toastMsg(res.message)
                }
            })
        },
        /**
         * 获取所有岗位信息列表
        */
        queryPostList(){
            var url = constGlobal.HostContact + 'getPostList' , _this = this;
            var param = {};
            this.postLoading = true;
            http.apiPost(url, param).then(res => {
                if (res.status == 0) {
                    this.postLoading = false;
                    this.postList = res.data;
                } else {
                    common.toastMsg(res.message)
                }
            })
        },
        /**
         * 获取根节点
        */
        queryTreeRoot(){
            var url = '';
            var param = {}
            if(this.config.rootNodeId){
                url = constGlobal.HostContact + 'corpOrDept/searchById';
                param = {
                    id : this.config.rootNodeId,
	                type : this.config.rootNodeType
                }
            }else{
                url = constGlobal.HostContact + 'orgRootNode/search';
            }
            http.apiPost(url ,param).then(res => {
                if (res.status == 0) {
                    this.loading = false;
                    if(res.data.length > 0){
                        for(var i = 0 ; i < res.data.length ; i++){
                            if(res.data[i].nodeType == '0'){
                                res.data[i].disabled = true;
                            }
                        }
                        this.completeOrgTree = res.data;
                        this.completeOrgTree[0].isLeaf = false;
                        this.initTreeSelect();
                    }
                } else {
                    common.toastMsg(res.message)
                }
            })
        },
         /**
         * 点击加载子节点
        */
        loadNode(node, resolve){
            if (node.level === 0) {
                return resolve(this.completeOrgTree);
            }else{
                let data;
                var url = constGlobal.HostContact + 'orgChildrenNode/search';
                var param = {
                    id : node.data.id,
                    nodeType : node.data.nodeType
                }
                http.apiPost(url, param).then(res => {
                    if (res.status == 0) {
                        if(res.data.length > 0){
                            for(var i = 0 ; i < res.data.length ; i++){
                                if(res.data[i].nodeType == '0'){
                                    res.data[i].disabled = true;
                                }
                            }
                            data = res.data;
                            this.initTreeSelect();
                        }
                        
                        return resolve && resolve(data);
                    } else {
                        common.toastMsg(res.message)
                    }
                })
            }
        },
         /**
         * 获取子节点选中状态
        */
        initTreeSelect(){
            this.checkedKeys = [];
            if(this.count.length !=0){
                for(var i = 0 ; i < this.count.length ; i++){
                    if(this.count[i].type == 0){
                        this.checkedKeys.push(this.count[i].relateId)
                    }
                }
            }
        },
        /**
         * 选择节点 checkbox
        */
        handlechange: function(data, value) {
            if(this.config.type == 1){
                return
            }
            var checkedNodes = this.$refs.tree.getCheckedNodes()
            if (value == true) {
                var list = {
                    relateId: data.id,
                    label: data.label,
                    type: 0,
                    data: data
                };
                this.count.push(list);
            }
            if (value == false) {
                this.handleDeleteTags(data);
            }
        },
        /**
         * 选择节点
        */
        nodeClick(node){
            this.node = JSON.parse(JSON.stringify(node));
            this.fuzzy = ''
            this.status = 'dept';
            this.pageNum = 1;
            this.total = 1;
            if(this.config.type == 1 || this.config.type == 0){
                this.queryMemList()
            }
        },
        /**
         * 选择岗位
        */
        currentRow(item){
            if(this.config.type == 0){
                for(var i = 0 ; i < this.count.length ; i++){
                    if(this.count[i].relateId == item.id){
                        common.toastMsg(item.name + '已存在，请勿重复选择');
                        return
                    }
                }
                var param = {
                    relateId: item.id,
                    label: item.name,
                    type: 2
                };
                this.count.push(param);
            }
            this.postCurrent = JSON.parse(JSON.stringify(item));
            this.fuzzy = ""
            this.status = 'post';
            this.queryMemList();
        },
        /**
         * 选择群组
        */
        currentGroupRow(item){
            if(this.config.type == 0){
                for(var i = 0 ; i < this.count.length ; i++){
                    if(this.count[i].relateId == item.id){
                        common.toastMsg(item.name + '已存在，请勿重复选择');
                        return
                    }
                }
                var param = {
                    relateId: item.groupId,
                    label: item.name,
                    type: 3
                };
                this.count.push(param);
            }
            this.groupCurrent = JSON.parse(JSON.stringify(item));
            this.fuzzy = ""
            this.status = 'group';
            this.searchList = item.memberList;
        },
        /**
         * 搜索
        */
        searchByKeyword(){
            if(this.fuzzy == ''){
                common.toastMsg('请输入姓名');
                return
            }
            this.queryMemList();
        },
        /**
         * 根据id获取人员信息
        */
        queryMemList(){
            var url ;
            var param = {}
            this.loading = true;
            this.searchList = [];
            
            url = constGlobal.HostContact + 'userInfoForPage/search';
            param = {
                deptIdList : this.node.id ? [this.node.id] : [],
                fuzzy : this.fuzzy,
                pager : {
                    pageNum : this.pageNum,
                    pageSize : this.pageSize
                },
                userTypeIdList : this.userTypeIds,
            }
            http.apiPost(url, param).then(res => {
                this.loading = false;
                if (res.status == 0) {
                    if(res.data){
                        this.searchList = res.data;
                        // if(!(this.node.id || this.postCurrent.id)){
                            for(var i = 0 ; i < this.searchList.length ; i++){
                                var name = '';
                                if(this.searchList[i].deptNodeId && this.searchList[i].deptNodeId != this.searchList[i].deptParentId && this.searchList[i].deptNodeId !=this.searchList[i].deptId){
                                    name += this.searchList[i].deptNodeName + '/';
                                }
                                if(this.searchList[i].deptParentId && this.searchList[i].deptParentId != this.searchList[i].deptId){
                                    name += this.searchList[i].deptParentName + '/'
                                }
                                name += this.searchList[i].deptName;
                                this.searchList[i].deptName = name;
                            }
                        // }
                        this.total = res.total;
                    }else{
                        this.searchList = [];
                        this.total = 0;
                    }
                    
                } else {
                    common.toastMsg(res.message)
                }
            })
        },
        /**
         * 取消选择节点
        */
        handleDeleteTags: function(info) {
            var index;
            if(this.deleteStatus){
                this.deleteStatus = false;
                return
            }
            Array.prototype.remove = function(val) {
                index = this.indexOf(val);
                if (index > -1) {
                    this.splice(index, 1);
                }
            }
            for(var i = 0 ; i < this.count.length ; i ++){
                if(this.count[i].type == 0 && this.count[i].relateId == info.id){
                    this.count.remove(this.count[i])
                    break;
                }
            }
        },
        /**
         * 删除标签
        */
        deleteTreeTag: function(info) {
            var i;
            var item = {
                label: info.label,
                id: info.relateId,
                type:info.type
            }
            this.$emit("deleteSelectTag" , info);
            if (item.type == 0) {//组织结构
                this.$refs.tree.setChecked( item, false);
                this.deleteStatus = true;
            }
            this.count.splice( this.count.indexOf(info), 1);
        },
        /**
         * 选择某一行
        */
        onSearchRowClick(section){
            var i , param;
            if(this.config.muliteChoice == 1){ //多选人员
                for(i = 0 ; i < this.count.length ; i++){
                    if(this.count[i].relateId == section.id){
                        common.toastMsg(section.name + '已存在，请勿重复选择');
                        break;
                    }
                }
                if(i == this.count.length){
                    param = {
                        type : 1,
                        label : section.name,
                        relateId : section.id,
                    }
                    this.count.push(param)
                }
            }else{
                param = {
                    type : 1,
                    label : section.name,
                    relateId : section.id,
                }
                this.count[0] = param;
            }
            
        },
        
        /**
         * 切换每页条数
        */
        onPageSizeChange(pageSize){
            this.pageSize = pageSize;
            this.pageNum = 1;
            this.queryMemList()
        },
        onPageChange(pageNum){
            this.pageNum = pageNum;
            this.queryMemList()
        },
        /**
         * 保存
        */
        rangeOk () {
            if(this.count.length == 0){
                return;
            }
            this.rangeModel = false;
            this.total = 1;
            this.$emit("submitRangeList" , this.count);
        },
        cancel () {
            this.rangeModel = false;
            this.total = 1;
            this.$emit("cancelRangeList" , this.count);
        },
        /**
         * 树节显示不同类型显示不同的图片
        */
        renderContent(h, { node, data, store }) {
            var createElement = arguments[0];
            var level = arguments[1].node.level;
            if (arguments[1].node.data.nodeType == '0') {
                return createElement('span', [
                    createElement('span', {attrs: {class: 'el-icon-yx-folder'}}),
                    createElement('span', arguments[1].node.label)
                ]);
            } else {
                return createElement('span', [
                    createElement('span', {attrs: {class: 'el-icon-yx-file-text'}}),
                    createElement('span', arguments[1].node.label)
                ]);
            }
        }
    }
};
</script>

<style lang="scss">
    @import '~assets/css/public.scss';
    @import '~assets/css/flexComp.scss';
    .visible-range-container1{
        height : 400px;
        .visible-range-result{
            border: 1px solid #efefef;
            padding: 8px 10px;
            border-radius: 5px;
            height: 46px;
            overflow: auto;
        }
        .visbile-range-list-wrap{
            margin-top:10px;
            .range-left{
                border: 1px solid #efefef;
                border-radius: 5px;
                width:200px;
                .tree-list{
                    width:100%;
                    overflow:auto;
                }
                .ivu-tabs-bar{
                    margin-bottom:2px;
                }
            }
            .range-right{
                padding : 0px 0px 0px 10px;
            }
            .ivu-page{
                text-align:right;
            }
        }
        .tree{
            overflow-x: auto;
            overflow-y: auto;
            width:198px;
            height:300px;
        }
        .el-tree{
            width:100%;
            overflow: initial;
            >.el-tree-node{
                min-width: 100%;
                display: inline-block !important;
            }
        }
        .tab-header{
            .tab{
                height:40px;
                text-align:center;
                line-height:40px;
                font-size:14px;
                border-bottom:1px solid #eaeaea;
                cursor: pointer;
            }
            .tab.active{
                color : #2d8cf0;
                position:relative;
            }
            .tab.active:before{
                content: "";
                position: absolute;
                bottom: 0px;
                border-bottom: 2px solid #2d8cf0;
                width: 100%;
                left: 0px;
            }
        }
        .el-icon-yx-folder{
            background:url("~components/select/assets/img/qy.png");
            display: inline-block;
            width: 15px;
            height: 14px;
            background-size: 15px 15px;
            margin-bottom: -2px;
            margin-right:4px;
        }
        .el-icon-yx-file-text{
            background:url("~components/select/assets/img/bm.png");
            display: inline-block;
            width: 15px;
            height: 14px;
            background-size: 15px 15px;
            margin-bottom: -2px;
            margin-right:4px;
        }
        .ivu-table th{
            height:33px
        }
        .ivu-table-small td{
            height: 36px;
            font-size: 13px;
        }
    }
</style>
