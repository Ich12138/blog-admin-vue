<template>
  <el-card>
    <div slot="header">
      <el-row>
        <el-col :span="12">
          <el-page-header @back="goBack" :content="'更新-'+blogInfo.blogTitle"/>
        </el-col>
      </el-row>
    </div>
    <el-row :grunt="24">
      <el-col :span="12">
        标题:
        <el-input :disabled="!markDownEditable.isEditable" v-model="blogInfo.blogTitle"
                  style="width: 300px;margin: 10px"></el-input>
      </el-col>
      <el-col :span="12">
        分类:
        <el-input :disabled="!markDownEditable.isEditable" v-model="blogInfo.blogType"
                  style="width: 300px;margin: 10px"></el-input>
      </el-col>
    </el-row>
    <el-row :grunt="24">
      <el-col :span="12">
        标签:
        <el-tag
            :closable="markDownEditable.isEditable"
            :key="index"
            v-for="(tag,index) in blogInfo.blogTagList"
            style="margin-right: 10px"
            :disable-transitions="false"
            @close="handleClose(tag)">
          {{ tag.name }}
        </el-tag>
        <el-input
            :disabled="!markDownEditable.isEditable"
            style="width: 100px;margin-top: 10px;margin-left: 5px"
            class="input-new-tag"
            v-if="inputVisible"
            v-model="inputValue"
            ref="saveTagInput"
            @keyup.enter.native="handleInputConfirm"
            @blur="handleInputConfirm">
        </el-input>
        <el-button :disabled="!markDownEditable.isEditable" v-else style="margin-top: 10px" class="button-new-tag"
                   @click="showInput">+
        </el-button>
      </el-col>
      <el-col :span="12">
        概要:
        <el-input :disabled="!markDownEditable.isEditable" v-model="blogInfo.blogOverview"
                  style="width: 300px;margin: 10px"></el-input>
      </el-col>
    </el-row>
    <el-row>
      <el-col>
        <el-button style="float: right; margin-right: 10px" type="text" @click="updateBlog">保存</el-button>
        <el-button
            v-if="markDownEditable.isEditable"
            @click="preview"
            type="text"
            style="margin-bottom: 10px;margin-right: 10px; float: right"
            icon="el-icon-view">
          预览
        </el-button>
        <el-button
            v-if="!markDownEditable.isEditable"
            style="margin-bottom: 10px;margin-right: 10px; float: right"
            type="text"
            @click="edit"
            icon="el-icon-edit">
          编辑
        </el-button>
      </el-col>
      <el-col>
        <mavon-editor
            ref=md
            @imgAdd="handleEditorImgAdd"
            @imgDel="handleEditorImgDel"
            :subfield="markDownEditable.subfield"
            :defaultOpen="'preview'"
            :toolbarsFlag="markDownEditable.toolbarsFlag"
            :editable="markDownEditable.editable"
            @save="updateBlog"
            v-highlight
            codeStyle="atom-one-dark"
            :boxShadow="false"
            @change="contentChange"
            class="mk-editor"
            v-model="blogInfo.blogRawContent"/>
      </el-col>
    </el-row>
  </el-card>
</template>

<script>
import {request} from "@/network/request";


//七牛云上传域名: http(s)://upload-z2.qiniup.com
//七牛云的AK: eiHzK04yfyyh-EymPKICBT-WFpanee7HXxXoDG78
//七牛云的SK: 0kHNdQYXaD1cpgERUam2kbNuQpJbPZaHYR4OtU49
export default {
  name: "EditBlog",
  activated() {
    // console.log(this.$route.query.blogNo)
    this.blogInfo.blogNo = this.$route.query.blogNo
    this.selectBlogByBlogNo(this.blogInfo.blogNo)
  },
  data() {
    return {
      inputVisible: false,
      inputValue: '',
      uploadUrl: "",

      //默认预览模式
      markDownEditable: {
        isEditable: false,
        subfield: false,
        defaultOpen: 'preview',
        toolbarsFlag: false,
        editable: false
      },

      //markdown渲染后的值
      renderValue: '',

      //博客属性
      blogInfo: {
        blogNo: '',
        blogTitle: '',
        blogType: '',
        blogOverview: '',
        blogRawContent: '',
        blogTagList: []
      }
    }
  },
  methods: {

    //编辑模式
    edit() {
      this.markDownEditable.isEditable = true;
      this.markDownEditable.subfield = true;
      this.markDownEditable.defaultOpen = 'edit';
      this.markDownEditable.toolbarsFlag = true;
      this.markDownEditable.editable = true;
    },

    //预览模式
    preview() {
      this.markDownEditable.isEditable = false;
      this.markDownEditable.subfield = false;
      this.markDownEditable.defaultOpen = 'preview';
      this.markDownEditable.toolbarsFlag = false;
      this.markDownEditable.editable = false;
    },

    //返回上一级页面
    goBack() {
      this.$router.back();
    },
    handleClose(tag) {
      this.blogInfo.blogTagList.splice(this.blogInfo.blogTagList.indexOf(tag), 1);
    },

    contentChange(value, render) {
      this.renderValue = render;
    },

    showInput() {
      this.inputVisible = true;
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },

    handleInputConfirm() {
      let inputValue = this.inputValue;
      if (inputValue) {
        let tag = {};
        tag.name = inputValue
        this.blogInfo.blogTagList.push(tag);
      }
      this.inputVisible = false;
      this.inputValue = '';
    },

    //一篇博客信息
    selectBlogByBlogNo(blogNo) {
      request({
        url: '/api/blog/query/one?blogNo=' + blogNo,
        method: 'get',
      }).then(res => {
        let resData = res.data;
        // console.log(resData);
        if (resData.status === 200) {
          this.blogInfo = resData.result.data
          this.blogInfo.blogTagList = JSON.parse(this.blogInfo.blogTagList)
        }
      })
    },

    //更新博客
    updateBlog() {
      let blog = {};
      blog.blogCoverUrl = ''
      blog.blogNo = this.blogInfo.blogNo
      blog.blogTagList = JSON.stringify(this.blogInfo.blogTagList);
      blog.blogTitle = this.blogInfo.blogTitle;
      blog.blogType = this.blogInfo.blogType;
      blog.blogRawContent = this.blogInfo.blogRawContent;
      blog.blogOverview = this.blogInfo.blogOverview
      let url = '/api/blog/update/content'
      request({
        url: url,
        method: 'post',
        data: blog
      }).then(res => {
        let resData = res.data;
        if (resData.status === 200) {
          // console.log("操作成功");
          this.$message.success("更新成功")
          this.preview()
        }
      })
    },

    //上传图片
    handleEditorImgAdd(pos, $file) {
      let fileData = new FormData();
      fileData.append('file', $file);
      request({
        url: '/api/blog/upload/img',
        data: fileData,
        method: 'post'
      }).then(res => {
        let resData = res.data;
        // console.log(resData);
        if (resData.status === 200) {
          let imgUrl = resData.result.data.imgUrl
          this.$refs.md.$imglst2Url([[pos, imgUrl]])
        } else {
          this.$message.error("图片上传失败")
        }
      })

    },
    //删除图片
    handleEditorImgDel(pos) {
      // delete this.imgFile[pos]
    }
  },
}
</script>

<style scoped>
.mk-editor {
  height: 800px;
}
</style>

