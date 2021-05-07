<template>
  <div class="main" @paste="handleTPaste">
    <div class="header">
      <wired-link elevation="1" href="https://github.com/xlzy520/bilibili-img-uploader/blob/master/README.md"
                  target="_blank">如何获取SESSDATA？如何使用图片样式？</wired-link>
    </div>
    <div class="token">
      <label>SESSDATA：</label>
      <wired-input name="SESSDATA" placeholder="你的哔哩哔哩SESSDATA" maxlength="32"
                   :value="token" @input="token = $event.target.value"
                   ref="nameInput" class="token-input"></wired-input>
      <wired-button elevation="3" class="submit-btn" @click="saveToken">保存</wired-button>
    </div>
    <div class="upload">
      <el-upload drag
                 ref="upload"
                 accept="image/*"
                 name="file_up"
                 :with-credentials="true"
                 :data="uploadData"
                 :file-list="fileList"
                 :on-success="uploadSuccess"
                 multiple
                 action="https://upload.cnblogs.com/imageuploader/CorsUpload">
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">支持粘贴、拖动、点击文件上传</div>
        <wired-button elevation="3" @click.stop="clearFileList" class="clear-btn">清空</wired-button>
      </el-upload>
    </div>
    <div class="result">
      <div class="link-area">
        <div class="link" v-for="link in links" :key="link.name">
          <label>{{link.name}}：</label>
          <div class="content">
            <wired-input type="text" :id="link.id" :value="link.value" class="link-result"
                         @input="link.value = $event.target.value"></wired-input>
            <wired-button elevation="3" @click="copyToClipboard(link.value)">复制</wired-button>
          </div>
        </div>
      </div>
    </div>
    <div class="footer">
      <img src="../../public/icons/favicon.png" width="32"/>
      By
      <wired-link elevation="1" class="author" href="https://github.com/xlzy520"
                  target="_blank">执笔看墨花开
      </wired-link>
    </div>
  </div>
</template>

<script>
  import { copyToClipboard, parseTime, getPasteImg } from "../utils";
  import Idb from 'idb-js'
  import db_img_config from '../db_img_config'
  import uuid from 'uuidjs'

  export default {
    name: 'Uploader',
    data() {
      return {
        token: 'CfDJ8L-rpLgFVEJMgssCVvNUAjtVVaymuhkz6-ow1kfD05JKi6UMD8k8DWKW6R09WrRwobblDYZCizZKpxALsDbaqdaT8DtkxSTDtJaoWOvkcNFTfyITOUv1OxIrRAqzB1sH9g-h35ATkJYPjVGcR-vBAJUdoFCHxzJjXHUdeF5gGFHS4pMMTLUck_uNKYhWUXO3aOCUGhRY6CAbtQaVdMN4jVvXC53D7J3p4YFG-ej4Wlyz4HlmfLDWnGAa-jVpu4cf6yhDqkwEB_odRGKOX0jWiqZoIIeMd3wKQ89maHg_R1SPMrT7jcCl6aaljxGSyl_IzdT0qcyMyozYazLW0GP_QICc953hCuV98vX68D3IAtiSFfsQGMMcPf-r_qdJ7DjLfAMteVWKC9G5rV6I2pPfjeMzo5erbbMRH0TXbPwFewWUAqnaF3gnrmpIjYfnbyZlCpeFK5ejTKmkPzZ3khLKtjNAmalaNg0Ve-6LqtucGKiojbqSW3Yf95ukohZ1hNQfSwbvHKU2d0j9U2CENQD_-OBLabOelLRznGYEm-xEjDIrM5K0nlokVCFQFYM1-LV7Lg',
        links: [
          {name: '图片链接', id: 'img', value: ''},
          {name: 'MarkDown', id: 'markdown', value: ''},
          {name: '博客园', id: 'cnblog', value: ''},
        ],
        uploadData: {
          category: 'daily',
          biz: 'draw'
        },
        fileList: []
      }
    },
    methods: {
      handleTPaste (event) {
        const image = getPasteImg(event)
        if (image) {
          this.fileList.push(image)
          this.$nextTick(() => {
            this.$refs.upload.submit();
          })
        }
      },
      clearFileList(){
        this.$refs.upload.clearFiles()
        this.$message('清空上传列表')
      },
      uploadSuccess(res, file){
        if (res.success) {
          const link = res.message;
          this.links[0].value = link
          this.links[1].value = `![](${link})`
          this.links[2].value = `<div align='center'><img src=${link}></div>`
          const img = document.querySelector('#img')
          const markdown = document.querySelector('#markdown')
          const cnblog = document.querySelector('#cnblog')
          img.value = this.links[0].value
          markdown.value = this.links[1].value
          cnblog.value = this.links[2].value
          this.copyToClipboard(img.value)
          Idb(db_img_config).then(img_db=>{
            img_db.insert({
              tableName: "img",
              data: {
                id: uuid.generate(),
                name: file.name,
                url: link,
                width: 0,
                height: 0,
                date: Date.now()
              },
              success: () => console.log("添加成功")
            });
          })
        } else {
          this.$message('上传失败:' + res.message,'error')
        }
      },
      copyToClipboard(input) {
        copyToClipboard(input)
      },
      saveToken() {
        if (this.token.length !== 32) {
          this.$message('请输入32位SESSDATA', 'info')
        } else {
          localStorage.setItem('SESSDATA', this.token)
          chrome.cookies.set(
            {
              url: 'https://upload.cnblogs.com', name: 'Cnblogs.AspNetCore.Cookies', value: this.token
            }, (data) => console.log(data)
          );
          this.$message('保存成功')
        }
      }
    },
    mounted() {
      const token = localStorage.getItem('SESSDATA')
      if (token) {
        this.token = token
      }
    },
  }
</script>
<style lang="less">
  :host {
    display: block;
    padding: 16px;
  }

  a {
    color: #F596AA;
    cursor: pointer;
  }

  label {
    width: 100px;
  }

  .main {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: center;

    .header {
      word-break: break-all;
    }

    .token {
      display: flex;
      align-items: center;
      /*justify-content: space-between;*/

      .token-input {
        font-size: 15px;
        width: 310px;
        /*height: 40px;*/
        line-height: 40px;
      }

      .submit-btn {
        height: 33px;
      }
    }

    .upload {
      text-align: center;
      padding: 15px 0;

      .el-upload-dragger {
        height: 120px;

        .el-icon-upload {
          margin: 20px 0 16px;
        }
      }
      .clear-btn{
        position: relative;
        left: 146px;
        bottom: 27px;
        background: bisque;

      }
    }

    .result {
      .link-area {
        .link {
          display: flex;
          align-items: center;
          margin-bottom: 10px;

          .link-result {
            width: 310px;
          }
        }
      }
    }

    .footer {
      display: flex;
      align-items: center;
      justify-content: center;
      width: 100%;
      color: gray;

      .author {
        color: rosybrown;
      }
    }
  }

</style>
