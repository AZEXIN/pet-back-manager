<template>
  <view class="page">
    <view class="uni-header">
      <view class="uni-title">编辑预定</view>
    </view>
    <view class="uni-container">
      <uni-forms ref="form" :model="formData" validateTrigger="submit" labelWidth="80px">
        <uni-forms-item label="服务项目" required>
          <uni-data-select v-model="formData.service_id" :localdata="serviceOptions" placeholder="请选择服务项目" />
        </uni-forms-item>
        <uni-forms-item label="预定时间" required>
          <uni-datetime-picker v-model="formData.booking_time" type="datetime" />
        </uni-forms-item>
        <uni-forms-item label="联系电话" required>
          <uni-easyinput v-model="formData.phone" placeholder="请输入联系电话" />
        </uni-forms-item>
        <uni-forms-item label="备注">
          <uni-easyinput v-model="formData.notes" type="textarea" placeholder="请输入备注信息" />
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
  const db = uniCloud.database()
  export default {
    data() {
      return {
        formData: {
          service_id: '',
          booking_time: '',
          phone: '',
          notes: ''
        },
        serviceOptions: [],
        rules: {
          service_id: {
            rules: [{
              required: true,
              errorMessage: '请选择服务项目'
            }]
          },
          booking_time: {
            rules: [{
              required: true,
              errorMessage: '请选择预定时间'
            }]
          },
          phone: {
            rules: [{
              required: true,
              errorMessage: '请输入联系电话'
            }, {
              pattern: '^1[3-9]\\d{9}$',
              errorMessage: '手机号格式不正确'
            }]
          }
        },
        id: ''
      }
    },
    async onLoad(options) {
      if (options.id) {
        this.id = options.id
        await this.loadServices()
        await this.loadBookingData()
      }
    },
    methods: {
      async loadServices() {
        try {
          const { result } = await db.collection('pet-services').field('_id,name').get()
          this.serviceOptions = result.data.map(item => ({
            value: item._id,
            text: item.name
          }))
        } catch (e) {
          uni.showToast({
            title: '加载服务列表失败',
            icon: 'error'
          })
        }
      },
      async loadBookingData() {
        try {
          const { result } = await db.collection('pet-bookings').doc(this.id).get()
          if (result.data) {
            this.formData = result.data
          }
        } catch (e) {
          uni.showToast({
            title: '加载预定信息失败',
            icon: 'error'
          })
        }
      },
      async submit() {
        try {
          await this.$refs.form.validate()
          await db.collection('pet-bookings').doc(this.id).update(this.formData)
          uni.showToast({
            title: '更新成功'
          })
          this.back()
        } catch (e) {
          uni.showToast({
            title: e.message || '提交失败',
            icon: 'error'
          })
        }
      },
      back() {
        uni.navigateBack()
      }
    }
  }
</script>

<style>
  .uni-button-group {
    margin-top: 30px;
    display: flex;
    justify-content: center;
  }

  .uni-button {
    margin: 0 10px;
  }

  .uni-title {
    margin-bottom: 20px;
    font-size: 16px;
    font-weight: bold;
  }
</style>