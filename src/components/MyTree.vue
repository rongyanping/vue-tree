<template>
  <div class="my_tree">
    <div
      class="brother"
      v-for="(data, idx) of treeData"
      :key="idx">
      <div class="node"
           @drop="drop($event,treeData,idx)"
           @dragover="allowDrop($event,treeData,idx)">

        <span @click="data.expand=!data.expand">
          <i class="el-icon-caret-bottom" v-if="data.expand && data.type==='group'"></i>
          <i class="el-icon-caret-right" v-if="!data.expand && data.type==='group'"></i>
        </span>

        <div :draggable="!data.editable"
             :class="{'cname_child':true,'cname_child_edit':data.editable}"
             :contentEditable="data.editable"
             @dragstart="drag($event,treeData,idx)"
             @dblclick="dbclickHandle($event,treeData,idx)"
             @click="data.expand=!data.expand">
            {{data.name}}
        </div>

        <button class="operation_btn" v-if=" data.type==='group'" @click="addBrother($event,data)">添加同级</button>
        <button class="operation_btn" v-if=" data.type==='group'" @click="addChild($event,data)">添加子级</button>
        <button class="operation_btn" v-if=" data.type==='group'"  @click="deleteNode($event, data)">删除</button>

      </div>

      <div
        class="children"
        v-if="data.children && data.children.length" v-show="data.expand">
        <my-tree
          :treeData="data.children"
          @addBrother="addBrother"
          @addChild="addChild"
          @deleteNode="deleteNode"
          @dragBegin="drag"
          @dragStop="drop"
          @dbclickChangeName="dbclickChangeName">
        </my-tree>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'MyTree',
    props: {
      treeData: {
        type: Array,
        // default: () => [{
        //   id: 1,
        //   name: '分组1',
        //   expand: true,
        //   type:'group',
        //   children: [{
        //     id: 2,
        //     expand: true,
        //     name: '相机1',
        //     type:'camera',
        //   }]
        // },
        //   {
        //     id: 3,
        //     expand: true,
        //     name: '分组2',
        //     type:'group',
        //     children: [
        //       {
        //         id: 5,
        //         expand: true,
        //         name: '分组2-1',
        //         type:'group',
        //         children: [{
        //           id: 6,
        //           expand: true,
        //           name: '相机2-1',
        //           type:'camera',
        //         }]
        //       }]
        //   }]
      },

    },
    methods: {
      // 双击
      dbclickHandle(event,data,index){
        const _this = this;
        this.$set(data[index],'editable',true);
        let target = event.target;
        setTimeout(function () {
          target.focus();
          let selection = getSelection();
          // 判断选定对象范围是编辑框还是文本节点
          if(selection.anchorNode.nodeName === '#text'){
            // 如果是文本节点则先获取光标对象
            let range = selection.getRangeAt(0);
            // 获取光标对象的范围界定对象，一般就是textNode对象
            let textNode = range.startContainer;
            // 光标移动到到原来的位置加上新内容的长度 rangeStartOffset +
            range.setStart(textNode, target.innerHTML.length);
            // 光标开始和光标结束重叠
            range.collapse(true);
            // 清除选定对象的所有光标对象
            selection.removeAllRanges();
            // 插入新的光标对象
            selection.addRange(range);
          }
          target.onblur = function () {
            _this.$emit('dbclickChangeName',data,index,target.innerText);
            // console.log('失焦---------',target.innerText)
          }
        },10);
      },

      // 传给父级新名称
      dbclickChangeName(data,index,name){
        this.$emit('dbclickChangeName',data,index,name);
        // console.log('====',name)
      },

      // 开始拖拽
      drag(event,data,index){
        this.$emit('dragBegin',event,data,index);
      },
      // 拖拽结束
      drop(event,data,index){
        event.preventDefault();
        this.$emit('dragStop',event,data,index);
        // console.log('drop>>>>>',event,data,index);
      },
      // 在何处放置被拖动的数据
      allowDrop(event,data,index){
        event.preventDefault();
      },

      // 添加子级
      addChild (event, data) {
        this.$emit('addChild', event, data)
      },
      // 添加同级
      addBrother (event, data) {
        this.$emit('addBrother', event, data);
      },
      // 删除
      deleteNode (event, data) {
        this.$emit('deleteNode', event, data)
      },
    }
  }
</script>

<style  scoped lang="scss">
  .my_tree{
    .brother{
      display: flex;
      flex-direction: column;
      padding-top: 20px;
      .node{
        display: flex;
        align-items: center;
        .cname_child{
          border-style: none;
          outline: none;
          width: 100px;
        }
        .cname_child_edit{
          border: 1px solid gray;
        }
        .operation_btn{
          margin-left: 20px
        }
      }
      .children {
        position: relative;
        margin-left: 20px;
      }
    }
  }

  input[disabled],input:disabled,input.disabled{
    color: #333;
    -webkit-text-fill-color:#333;
    opacity: 1;
    background-color:white;
  }
</style>
