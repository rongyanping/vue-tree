<template>
  <div class="tree">
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
      @dbclickChangeName="dbclickChangeName">
    </my-tree>

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
          children: [{
            id: 2,
            expand: true,
            name: '相机1',
            type:'camera',
            editable:false,
          }]
        },
          {
            id: 3,
            expand: true,
            name: '分组2',
            type:'group',
            editable:false,
            children: [
              {
                id: 5,
                expand: true,
                name: '分组2-1',
                type:'group',
                editable:false,
                children: [{
                  id: 6,
                  expand: true,
                  name: '相机2-1',
                  type:'camera',
                  editable:false,
                },{
                  id: 7,
                  expand: true,
                  name: '相机2-2',
                  type:'camera',
                  editable:false,
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
        addType:'group'
      }
    },
    methods: {
      /* 操作数据全部在本组件内部 */

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
          // console.log('node=====',node,node.__vue__)
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
  }
  .tree-title{
    border-bottom: 1px solid gray;
    padding-bottom: 10px;
    margin-bottom: 10px;
  }
</style>
