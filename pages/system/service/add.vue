<template>
  <view class="container">
    <view class="uni-header">
      <view class="uni-title">新增服务</view>
    </view>
    <view class="uni-container">
      <uni-forms ref="form" :model="formData" validateTrigger="submit">
        <uni-forms-item name="name" label="服务名称" required>
          <uni-easyinput v-model="formData.name" placeholder="请输入服务名称" />
        </uni-forms-item>
        <uni-forms-item name="description" label="服务描述">
          <uni-easyinput type="textarea" v-model="formData.description" placeholder="请输入服务描述" />
        </uni-forms-item>
        <uni-forms-item name="price" label="价格" required>
          <uni-easyinput type="number" v-model="formData.price" placeholder="请输入价格" />
        </uni-forms-item>
        <uni-forms-item name="image_url" label="图片地址">
          <uni-file-picker v-model="formData.files" fileMediatype="image" mode="grid" @success="uploadSuccess" />
        </uni-forms-item>
        <view class="uni-button-group">
          <button type="primary" class="uni-button" @click="submit">提交</button>
          <button type="default" class="uni-button" @click="back">返回</button>
        </view>
      </uni-forms>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        formData: {
          name: '',
          description: '',
          price: '',
          image_url: '',
          files: []
        },
        rules: {
          name: {
            rules: [{
              required: true,
              errorMessage: '请输入服务名称'
            }]
          },
          price: {
            rules: [{
              required: true,
              errorMessage: '请输入价格'
            }, {
              format: 'number',
              errorMessage: '价格必须是数字'
            }]
          }
        }
      }
    },
    onReady() {
      this.$refs.form.setRules(this.rules)
    },
    methods: {
      uploadSuccess(e) {
        this.formData.image_url = e.tempFilePaths[0]
      },
      submit() {
        this.$refs.form.validate().then(async (res) => {
          try {
            const db = uniCloud.database()
            await db.collection('pet-services').add({
              name: this.formData.name,
              description: this.formData.description,
              price: Number(this.formData.price),
              image_url: this.formData.image_url
            })
            uni.showToast({
              title: '添加成功'
            })
            this.back()
          } catch (e) {
            uni.showToast({
              title: '添加失败',
              icon: 'error'
            })
          }
        }).catch((err) => {
          console.log('表单错误:', err)
        })
      },
      back() {
        uni.navigateBack()
      }
    }
  }
</script>

<style>
  .uni-container {
    padding: 15px;
  }

  .uni-button-group {
    margin-top: 50px;
    display: flex;
    justify-content: center;
  }

  .uni-button {
    margin: 0 10px;
  }

  .uni-title {
    margin-bottom: 10px;
  }
</style>