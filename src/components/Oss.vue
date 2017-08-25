<template>
  <div class="page">
    <!-- 调用文件列表查询方法 -->
    <button class="btn"
      @click="list">查询列表</button>
    <div class="list-container">
      <a v-for="item in fileList"
        :href="item.url">
        {{item.name}}
      </a>
    </div>
  
    <input type="text"
      class="txt-item"
      placeholder="请输入文件名"
      v-model="fileName">
    <textarea placeholder="请输入文件内容"
      class="txt-item"
      rows="5"
      type="text"
      v-model="content"></textarea>
  
    <!-- 调用文件上传方法 -->
    <button class="btn"
      @click="upload">上传文件</button>
  </div>
</template>

<script>
import { co } from 'co'

export default {
  data () {
    return {
      client: {},
      fileList: [],
      fileName: '',
      content: ''
    }
  },
  created () {
    // 在这里定义你的OSS的参数
    var bucket = '你的bucket名称'
    var region = 'oss-cn-hangzhou'

    // sts令牌生产参见文档：https://help.aliyun.com/document_detail/32069.html?spm=5176.doc31931.6.738.oed5WT

    // 这里用的是co的同步方法，如果要用异步方法，需要将此语句改为：new windows.OSS.Wrapper({...})
    this.client = new window.OSS({
      region: region,
      accessKeyId: 'STS-accessKeyId',
      accessKeySecret: 'STS-accessKeySecret',
      stsToken: 'STS-stsToken',
      bucket: bucket
    })
  },
  methods: {
    // 列出你的BUCKET名称中的最新的5个文件
    list () {
      const _this = this
      co(function* () {
        _this.client.useBucket('weixt-pageupload')
        var result = yield _this.client.list({
          'max-keys': 100
        })
        console.log(result)
        var objects = result.objects.sort(function (a, b) {
          var ta = new Date(a.lastModified)
          var tb = new Date(b.lastModified)
          if (ta > tb) return -1
          if (ta < tb) return 1
          return 0
        })
        _this.fileList = objects.slice(0, 5)
      }).catch(function (err) {
        console.log(err)
      })
    },
    // 将文本框的内容上传到OSS，并刷新文件列表
    upload () {
      const _this = this
      co(function* () {
        // put from Buffer
        yield _this.client.put(_this.fileName, new Buffer(_this.content))
        _this.list()
      }).catch(function (err) {
        console.log(err)
      })
    }
  }
}
</script>

<style scoped>
.page {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.list-container {
  display: flex;
  flex-direction: column;
  align-content: flex-start;
  border: 1px solid #666;
  margin: 10px;
  width: 300px;
  min-height: 50px;
}

.btn {
  width: 120px;
  height: 30px;
  font-size: 14px;
}

.txt-item {
  width: 180px;
  margin: 10px
}
</style>
