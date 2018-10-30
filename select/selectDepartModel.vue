<template>
    <Modal
        v-model="rangeModel"
        :title="config.title"
        class="visible-depart-range-one"
        width = "450px"
        @on-cancel="cancel"
        @on-visible-change ="visibleChange">
        <div class="visible-range-container flex flex-direction-column">
            <div class="visible-range-result">
                <Tag v-for="(item , index) in count"
                    :key="index"
                    v-if="count.length > 0"
                    color="blue"
                    type="border"
                    >{{item.label}}</Tag>
            </div>
            <div class="visbile-range-list-wrap flex flex-1">
                <div class="range-left1 flex flex-direction-column" >
                    <div style="height: 390px; " class="flex flex-direction-column">
                        <div class="tab-header flex">
                            <div 
                                class="flex-1 tab"
                                :class="{'active' : activeIndex == 0}" 
                               >部门</div>
                        </div>
                        <div class="tab-container flex-1" >
                            <div v-show="activeIndex == 0" class="tree1">
                                <el-tree 
                                    :data="completeOrgTree" 
                                    v-loading="loadingTree" 
                                    element-loading-text="加载中..." 
                                    empty-text="暂无数据" 
                                    node-key="id" 
                                    :default-checked-keys="checkedKeys" 
                                    :show-checkbox="config.muliteChoice == 1"
                                    :expand-on-click-node="false"
                                    :check-on-click-node="false"
                                    ref="tree" 
                                    :props="defaultProps" 
                                    @check-change="handlechange"
                                    @node-click="nodeClick"
                                    :load="loadNode"
                                    lazy
                                    check-strictly
                                    :render-content="renderContent">
                                    </el-tree>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
        <div slot="footer">
            <Button @click="cancel">取消</Button>
            <Button type="primary" :disabled="count.id == ''" @click="rangeOk">确定</Button>
        </div>
    </Modal>
</template>

<script>

export default {
    data () {
        return {
            tabValue : '1',
            highlight : true,
            count : {
                name : '',
                id : ''
            },
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
            checkPostColumn: [{
                title: '岗位名称',
                key: 'name'
            }],
            postList : [],
            status : '',
            pageSize : 10,
            activeIndex : 0,
            node : {
                id : ''
            },
            deleteStatus : false,
        };
    },
    components : {
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
        initSelect(count){
            if(count == undefined){
                this.count = {
                    name : '',
                    id : ''
                }
            }else{
                this.count = count;
            }
            this.$refs.tree.setCheckedKeys([]);
            this.node.id = '';
            this.rangeModel = true;
            this.queryTreeRoot();
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
                    for(var i = 0 ; i < res.data.length ; i++){
                        if(res.data[i].nodeType == '0'){
                            res.data[i].disabled = true;
                        }
                    }
                    this.completeOrgTree = res.data;
                    this.loading = false;
                    this.initTreeSelect();
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
                        for(var i = 0 ; i < res.data.length ; i++){
                            if(res.data[i].nodeType == '0'){
                                res.data[i].disabled = true;
                            }
                        }
                        data = res.data;
                        this.initTreeSelect();
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
         * 选择节点
        */
        nodeClick(data){
            if(this.config.muliteChoice == 1){
                return
            }
            var param = {
                relateId: data.id,
                label: data.label,
                type: 0,
                data: data
            }
            this.count[0] = param;
            this.count.push();
        },
        /**
         * 选择节点 checkbox
        */
        handlechange: function(data, value) {
            if(this.config.muliteChoice == 2){
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
        visibleChange(){
            if(this.rangeModel == false){
                this.$emit("cancelRangeList" , this.count);
            }
        },
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
        },
    }
};
</script>

<style lang="scss">
    @import '~assets/css/public.scss';
    @import '~assets/css/flexComp.scss';
    .visible-depart-range-one{
        .visible-range-result{
            border: 1px solid #efefef;
            padding: 8px 10px;
            border-radius: 5px;
            height: 46px;
            overflow: auto;
        }
        .visbile-range-list-wrap{
            margin-top:10px;
            .range-left1{
                border: 1px solid #efefef;
                border-radius: 5px;
                width:100% !important;
                .tree-list{
                    width:100% !important;
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
        .tree1{
            overflow-x: auto;
            overflow-y: auto;
            width:100%;
            height: 297px;
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
            background:url("~assets/img/qy.png");
            display: inline-block;
            width: 15px;
            height: 14px;
            background-size: 15px 15px;
            margin-bottom: -2px;
            margin-right:4px;
        }
        .el-icon-yx-file-text{
            background:url("~assets/img/bm.png");
            display: inline-block;
            width: 15px;
            height: 14px;
            background-size: 15px 15px;
            margin-bottom: -2px;
            margin-right:4px;
        }
    }
</style>
