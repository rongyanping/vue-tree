<template>
  <div class="tree">
    <div id="tree_content" :class="{'stretch':canStretch}">
      <div class="tree-title" >
        树结构：添加同级、添加子级、删除、拖拽、双击修改名称
      </div>
      <my-tree
        @addBrother="addBrother"
        @addChild="addChild"
        @deleteNode="deleteNode"
        :treeData="datas"
        @dragBegin="drag"
        @dragStop="drop"
        @dbclickChangeName="dbclickChangeName"
        @treeClickHandle="treeClickHandle"
        @checkeboxHandle="checkeboxHandle">
      </my-tree>
    </div>


    <div v-show="iptVisible" style="margin-top: 20px">
      <span>分组名：</span>
      <input type="text" v-model="groupName">
      <button @click="submitGroup">提交</button>
    </div>
    <div v-show="iptVisible2" style="margin-top: 20px">
      <span>子级名：</span>
      <input type="text" v-model="groupName2">
      类型：分组/相机
      <input type="text" v-model="addType">
      <button @click="submitChild">提交</button>
    </div>
  </div>
</template>

<script>
  import MyTree from './MyTree'
  export default {
    name: 'tree3',
    components: {
      'my-tree': MyTree
    },
    data () {
      return {
        id: 100,
        datas:[{
          id: 1,
          name: '分组1',
          expand: true,
          type:'group',
          editable:false,
          active:false,
          checked:false,
          children: [{
            id: 2,
            expand: true,
            name: '相机1',
            type:'camera',
            editable:false,
            active:false,
            checked:false,
          }]
        },
          {
            id: 3,
            expand: true,
            name: '分组2',
            type:'group',
            editable:false,
            active:false,
            checked:false,
            children: [
              {
                id: 5,
                expand: true,
                name: '分组2-1',
                type:'group',
                editable:false,
                active:false,
                checked:false,
                children: [{
                  id: 6,
                  expand: true,
                  name: '相机2-1',
                  type:'camera',
                  editable:false,
                  active:false,
                  checked:false,
                },{
                  id: 7,
                  expand: true,
                  name: '相机2-2',
                  type:'camera',
                  editable:false,
                  active:false,
                  checked:false,
                }]
              }]
          }],
        dragData:null,
        dragEvent:null,
        dragIndex:null,

        cid:10,
        groupName:'',
        iptVisible:false,
        brotherData:null,
        brotherEvent:null,

        addChildData:null,
        addChildEvent:null,
        iptVisible2:false,
        groupName2:'',
        addType:'group',

        canStretch:false,

        parentData:null
      }
    },
    mounted:function(){
        const _this = this;
        _this.$nextTick(()=>{
          let treeDom = document.getElementById('tree_content');
          treeDom.addEventListener('mousemove',_this.listStretch);
        })
    },
    methods: {
      /* 操作数据全部在本组件内部 */

      // checkbox
      checkeboxHandle(event,data){
        const _this = this;
        // 点击分组
        // if(data.type==='group'){
        //   _this.recursionCheckedData(data,data.checked);
        // }
        // else {
        //   _this.recursionParentData(_this.datas,data);
        //   let parentData = _this.parentDomData(event.target);
        //   console.log('pdata===',parentData);
        //   let childData = parentData[0].children;
        //   let childLen = childData.length;
        //   let checkCount = 0;
        //   for(let i=0;i<childLen;i++){
        //     if(childData[i].checked){
        //       checkCount++;
        //     }
        //   }
        //   if(checkCount===childLen){
        //     parentData[0].checked=true;
        //   }
        //   else {
        //     parentData[0].checked=false;
        //   }
        // }
        _this.recursionData(this.datas);
      },
      recursionCheckedData(data,boolean){
        const _this = this;
        if(!data.children || data.children.length===0){return;}
        let childrenData = data.children;
        let len = childrenData.length;
        for(let i=0;i<len;i++){
          childrenData[i].checked = boolean;
          if(childrenData[i].children && childrenData[i].children.length>0){
            _this.recursionCheckedData(childrenData[i],boolean);
          }
        }
      },
      recursionParentData(data,target){
        let len = data.length;
        for(let i=0;i<len;i++){
          // console.log('id===',data[i])
          if(data[i].id===target.id){
            console.log('id===',data,this.parentData);
            let checkCount = 0;
            data.forEach((item,index)=>{
              if(item.checked){
                checkCount++;
              }
            });
            if(checkCount===data.length){
              this.parentData.checked=true;
            }
            else {
              this.parentData.checked=false;
            }
          }
          else {
            if(data[i].children && data[i].children.length>0){
              this.parentData = data[i];
              this.recursionParentData(data[i].children,target);
            }

          }
        }
      },
      recursionData(data){
        let len = data.length;
        for(let i=0;i<len;i++){
          // console.log('data===',data[i],data[i].children)
          if(data[i].checked && data[i].children && data[i].children.length>0){
            data[i].children.forEach((item,index)=>{
              item[index].checked = true;
            })
          }
          if(data[i].children && data[i].children.length>0){
            this.recursionChildData(data[i],data[i].children);
            this.recursionData(data[i].children);
          }
        }
      },
      recursionChildData(parentData,item){
        let len = item.length;
        let checkCount = 0;
        console.log('子级count======',parentData,item);
        // 父级
        // if(parentData.checked){
        //   item.forEach((val,key)=>{
        //     item[key].checked = true;
        //   });
        // }
        // else {
        //   item.forEach((val,key)=>{
        //     item[key].checked = false;
        //   });
        // }
        // 子级
        item.forEach((val,key)=>{
          if(val.checked){
            checkCount++;
          }
        });
        if(checkCount===len){
          parentData.checked=true;
          this.recursionData(parentData);
        }
        else {
          parentData.checked=false;
        }


      },
      parentDomData(node){
        const _this = this;
        while (node && node.tagName !== 'BODY') {
          if (node.__vue__ && node.__vue__.$options._componentTag === 'my-tree') {
            console.log('node=====',node.__vue__.$options.parent.treeData)
            if(node.__vue__.$options.parent.treeData){
              _this.parentDomData(node.__vue__.$options.parent);
            }
            else {
              return node.__vue__.$options.parent.treeData;
            }
          }
          node = node.parentNode
        }
        return null
      },
      // 左右拉伸
      listStretch(event){
        const _this = this;
        let doms = document.getElementById('tree_content');
        let maxWidth = doms.offsetWidth + doms.offsetLeft+2;
        let minWidth = doms.offsetWidth + doms.offsetLeft -5;
        let clientX = event.clientX;
        if(clientX>=minWidth && clientX<=maxWidth ){
          _this.canStretch = true;
          // 鼠标按下 拉伸
          doms.onmousedown = function (de) {
            let e = de ||event;
            let x = e.clientX;
            let oBoxW = doms.offsetWidth;
            _this.canStretch = true;
            // console.log('ev==down===',x ,oBoxW);
            document.onmousemove = function (eve) {
              let moveE = eve ||event;
              let xx = moveE.clientX;
              console.log('ev==move===',oBoxW , xx -x );
              if(oBoxW <=800){
                doms.style.width = oBoxW + xx -x + 'px'
              }
              if(oBoxW > 800){
                document.onmousemove = null;
                doms.onmousedown = null;
                _this.canStretch = false;
              }
              return false;
            };
            document.onmouseup = function(){
              document.onmousemove = null;
              document.onmouseup = null;
              _this.canStretch = false;
            };
            if(e.preventDefault){
              e.preventDefault();
            }
          }
        }
        else {
          _this.canStretch = false;
        }

        if(clientX<=minWidth-10){
          _this.showScroll = true;
        }
        else {
          _this.showScroll = false;
        }
      },
      // 点击名字
      treeClickHandle(data){
        // console.log('click======',this.datas);
        this.traverseTree(this.datas,data.id);
      },
      // 遍历树形结构
      traverseSingle(item,paramsId){
        if(item.id === paramsId){
          this.$set(item,'active',true);
          // console.log('========',item.id,paramsId)
        }
        else {
          this.$set(item,'active',false);
        }
      },
      traverseTree(data,paramsId){
        const _this = this;
        if(!data){return;}
        for(let i=0;i<data.length;i++){
          _this.traverseSingle(data[i],paramsId);
          if(data[i].children && data[i].children.length>0){
            _this.traverseTree(data[i].children,paramsId);
          }
        }
        // console.log('==id====',paramsId)
      },

      // 接收新名称
      dbclickChangeName(data,index,changeName){
        // 发送请求：修改名称
        this.$set(data[index],'name',changeName);
        this.$set(data[index],'editable',false);
        console.log('dbclick=====',changeName);
      },

      /* 拖拽  相机--》分组； 分组---》分组中 且拖拽到的那一级 必须是分组级 不能含有相机 */
      drag(event,data,index){
        this.dragData = data;
        this.dragEvent = event;
        this.dragIndex = index;
        // console.log('dragStart=====',this.dragData,index);
      },
      drop(event,data,index){
        /* 拖拽只能是拖到某个分组内 成为该分组的子级 */
        /* 松开鼠标---调取修改接口，成功后添加新数据，删除之前位置*/
        const _this = this;
        data[index].children.push(this.dragData[this.dragIndex]);

        // 删除原有数据
        setTimeout(function () {
          console.log('dragOver=====',_this.dragEvent,_this.dragData);
          _this.deleteNode(_this.dragEvent,_this.dragData[_this.dragIndex])
        },10);
      },

      // 添加同级
      addBrother (event,data) {
        this.iptVisible = true;
        this.id++;
        this.brotherData = data;
        this.brotherEvent = event;
        console.log('addBrother>>>>',this.brotherData);
      },
      submitGroup(){
        let parentData = this.getParentData(this.brotherEvent.target);
        // console.log('parentData========',this.brotherEvent.target);
        if (parentData) {
          let index = parentData.indexOf(this.brotherData);
          if (index !== -1) {
            parentData.splice(index + 1, 0, {
              id: this.id,
              name: this.groupName,
              expand: false,
              type:'group',
              children: []
            })
          }
        }
      },

      // 添加子级
      addChild (event, data) {
        this.iptVisible2 = true;
        this.cid++;
        this.addChildData = data;
        console.log(this.addChildData)
        // // data 父级数据
        // console.log('addchild===tree3==',data);
        // if (!data.children) {
        //   this.$set(data, 'children', [])
        // }
        //   data.children.push(this.newNode('camera'))
        //   console.log('data====',data)
      },
      submitChild(){
        if(!this.addChildData.children){
          this.$set(this.addChildData, 'children', [])
        }
        if(this.addType==='group'){
          this.addChildData.children.push({
            id: this.cid,
            children:[],
            expand: false,
            name: this.groupName2,
            type:'group',
          })
        }
        else {
          this.addChildData.children.push({
            id: this.cid,
            expand: true,
            name: this.groupName2+this.cid,
            type:'camera',
          })
        }
      },

      // 删除组
      deleteNode (event, data) {
        console.log('del====',event,data)
        let parentData = this.getParentData(event.target);
        if (parentData) {
          let index = parentData.indexOf(data);
          if (index !== -1) {
            parentData.splice(index, 1)
          }
        }
      },

      newNode (type) {
        let id = this.id++;
        return {
          id,
          name: type==='group'?'新分组' + id:'新相机'+id,
          expand: false,
          children: [],
          type:type
        }
      },
      getParentData (node) {
        while (node && node.tagName !== 'BODY') {
          console.log('node=====',node.__vue__)
          if (node.__vue__ && node.__vue__.$options._componentTag === 'my-tree') {
            return node.__vue__.treeData
          }
          node = node.parentNode
        }
        return null
      }
    }
  }
</script>
<style lang="scss" scoped>
  .tree{
    padding: 50px 20px;
    text-align: left;
    #tree_content{
      width: 450px;
      min-width: 450px!important;
      max-width: 800px!important;
      background: grey;
      .tree-title{
        border-bottom: 1px solid gray;
        padding-bottom: 10px;
        margin-bottom: 10px;
      }
    }
    .stretch{
      cursor: e-resize;
    }
  }

</style>
