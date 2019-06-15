<template>
  <div>
    <div>
      <el-tag
        v-for="(tag,index) in dynamicTags"
        :key="`tag_${index}`"
        closable
        @close="handleClose(tag)"
        effect="dark"
        :type="tag.type"
        :disable-transitions="true"
      >{{tag.label}}</el-tag>
      <el-popover
        trigger="click"
        width="300"
        placement="bottom-start"
        class="popover"
        :visible-arrow="false"
        v-model="popVisible"
      >
        <!-- page1 -->
        <div v-if="page==1">
          <!-- 头部 -->
          <div class="pop-head">
            <el-input
              ref="saveTagSelect"
              v-model.trim="selectValue"
              @keyup.enter.native="create"
              placeholder="搜索标签"
            ></el-input>
            <!-- <i class="el-icon-plus"></i> -->
          </div>
          <!-- 可选标签列表 -->
          <div v-if="selectValue" class="customize">
            <div class="colorList">
              <span
                class="color"
                :class="{'el-icon-check':selectType == name}"
                :style="{'background-color':color}"
                v-for="(color,name) in colorMap"
                :key="color"
                @click="selectType=name"
              ></span>
            </div>
            <el-button type="primary" class="createBtn" @click="create">创建</el-button>
          </div>
          <ul v-else>
            <li class="item" v-for="(item,idx) in dynamicTags" :key="`item_${idx}`">
              <span class="color" :style="{'background-color':colorMap[item.type]}"></span>
              <span class="name">{{item.label}}</span>
              <span class="option">
                <i class="el-icon-edit" @click="editTag(item)"></i>
              </span>
            </li>
          </ul>
        </div>
        <!-- page2 -->
        <div v-else>
          <div class="pop-head">
            <h4>编辑标签</h4>
            <i class="el-icon-back left" @click="reset"></i>
            <!-- <i class="el-icon-close right"></i> -->
          </div>
          <div class="customize">
            <el-input v-model.trim="selectValue"></el-input>
            <div class="colorList">
              <span
                class="color"
                :class="{'el-icon-check':selectType == name}"
                :style="{'background-color':color}"
                v-for="(color,name) in colorMap"
                :key="color"
                @click="selectType=name"
              ></span>
            </div>
            <div class="btnList">
              <el-button type="danger" @click="deleteTag">删除</el-button>
              <el-button type="primary" @click="create">完成</el-button>
            </div>
          </div>
        </div>
        <!-- 添加按钮 -->
        <i
          slot="reference"
          class="initAddBtn"
          :class="{'el-icon-plus': hasTags}"
          @click="showSelect"
        >{{hasTags ? '' : '添加标签'}}</i>
      </el-popover>

      <!-- add -->
      <!-- <div v-else @click="showSelect" class="initAddBtn">添加标签</div> -->
    </div>
  </div>
</template>

<script>
import Vue from "vue";
export default {
  mounted(){
    console.log(this.dynamicTags,this.lastId);
    this.dynamicTags = JSON.parse(localStorage.getItem('tag')) ||this.dynamicTags;
    this.lastId =  JSON.parse(localStorage.getItem('lastId')) || this.lastId;
    console.log(this.dynamicTags,this.lastId);
    
  },
  data() {
    return {
      colorMap: {
        success: "#67C23A",
        warning: "#E6A23C",
        danger: "#F56C6C",
        info: "#909399"
      },
      dynamicTags: [],
      selectVisible: false,
      selectValue: "",
      selectType: "info",
      lastId: 0,
      curId: 0,
      popVisible: false,
      page: 1
    };
  },
  methods: {
    reset(){
      this.page=1;
      this.curId=0;
      this.selectValue='';
      this.selectType='info';
    },
    deleteTag(){
      let index = this.dynamicTags.findIndex(el=>el.id==this.curId);
      this.dynamicTags.splice(index, 1);
      this.reset();
    },
    handleClose(tag) {
      this.dynamicTags.splice(this.dynamicTags.indexOf(tag), 1);
    },

    showSelect() {
      this.selectVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveTagSelect.$refs.input.focus();
      });
    },
    editTag(item) {
      this.page = 2;
      this.curId = item.id;
      console.log("editTag", this.curId);
      this.selectType = item.type;
      this.selectValue = item.label;
    },
    create() {
      if(!this.selectValue)return;
      if (this.curId !== 0) {
        //更新
        let newTag = {
          id: this.curId,
          type: this.selectType,
          label: this.selectValue
        };

        let target = this.dynamicTags.findIndex(el => el.id == this.curId);

        this.$set(this.dynamicTags, target, newTag);
        this.reset();
      } else {
        //创建
        this.lastId++;
        let newTag = {
          id: this.lastId,
          type: this.selectType,
          label: this.selectValue
        };
        if (newTag.label) {
          this.dynamicTags.push(newTag);
        }
      }
      this.reset();
    }
  },
  computed: {
    hasTags() {
      return this.dynamicTags.length !== 0;
    }
  },
  watch:{
    dynamicTags:{
      deep:true,
      handler(n,o){
        localStorage.setItem('tag',JSON.stringify(this.dynamicTags));
      }
    },
    lastId(){
      localStorage.setItem('lastId',this.lastId);
    }
  }
};
</script>

<style >
i {
  font-style: normal;
  outline: none;
}
.initAddBtn {
  color: #aaa;
  cursor: pointer;
  display: inline-block;
  margin: 10px;
  
}
.initAddBtn:hover {
  color: rgb(94, 164, 235);
}

.el-tag {
  border-radius: 16px !important;
  margin: 6px;
}
.el-tag .el-tag__close {
  display: none;
}
.el-tag:hover .el-tag__close {
  display: inline-block !important;
}
.el-icon-plus {
  font-size: 14px;
  padding: 4px;
  color: #fff;
  background-color: #ccc;
  border-radius: 50%;
  margin-left: 10px;
  cursor: pointer;
}
.el-popover {
  padding: 0px !important;
}
.pop-head {
  position: relative;
  border-bottom: 1px solid #ccc;
}
.pop-head i {
  position: absolute;
  top: 50%;
  /* z-index: -1; */
  transform: translateY(-50%);
}
.pop-head i.left {
  left: 10px;
  display: inline-block;
}
.pop-head i.right {
  right: 10px;
}
.pop-head h4{
  text-align: center;
  margin:0;
  line-height: 32px;
}
.pop-head .el-input__inner {
  border: none !important;
  outline: none !important;
  padding: 0 26px !important;
}
.pop-head i {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  color: red;
  /* background-color: #fff; */
}
ul {
  list-style-type: none;
  margin-left: 0;
  padding: 0;
}
.item,
.colorList,
.btnList {
  height: 38px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.btnList{
  justify-content: space-evenly;

}
.colorList {
  margin: 20px 0;
}
.color.el-icon-check {
  /* display: none; */
  color: #fff;
  text-align: center;
  vertical-align: middle;
  line-height: 38px;
}
.item:hover .color.el-icon-check {
  /* display: block; */
}
.item {
  cursor: pointer;
}
.item:hover {
  background-color: rgba(0, 0, 0, 0.05);
}
.customize {
  padding: 12px 26px;
}
.item span.option {
  width: 20px;
  padding-left: 8px;
  padding-right: 8px;
  margin-right: 8px;
  font-size: 18px;
  /* margin: 20px; */
}
.item span.option i {
  display: none;
}
.item:hover span.option i {
  display: block;
  color: #f56c6c;
}
.item span.name {
  flex-grow: 1;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.item span.color {
  border-radius: 50%;
  width: 12px;
  height: 12px;
  margin: 10px;
}
.customize .colorList span.color {
  width: 38px;
  height: 38px;
  border-radius: 50%;
}
.el-icon-check {
  font-size: 20px;
}
.createBtn {
  margin-top: 20px !important;
  width: 100%;
}
</style>

