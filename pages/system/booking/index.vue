<template>
  <view class="page">
    <view class="uni-header">
      <view class="uni-group">
        <button class="uni-button" type="primary" @click="navigateToAdd">新增</button>
        <button class="uni-button" type="warn" @click="delTable">批量删除</button>
      </view>

      <view class="uni-group">
        <input class="uni-search" type="text" v-model="query" placeholder="请输入搜索内容" />
        <button class="uni-button" type="default" @click="search">搜索</button>
      </view>
    </view>

    <view class="uni-container">
      <unicloud-db ref="udb" v-slot:default="{data, pagination, loading, error, options}" :collection="collectionList"
        field="booking_id,service_id,booking_time,phone,notes,create_time" :where="where" page-data="replace"
        :orderby="orderby" :getcount="true" :page-size="options.pageSize" :page-current="options.pageCurrent">
        <uni-table ref="table" :loading="loading" :emptyText="error.message || '没有更多数据'" border stripe type="selection">
          <uni-tr>
            <uni-th align="center">预定编号</uni-th>
            <uni-th align="center">服务项目</uni-th>
            <uni-th align="center">预定时间</uni-th>
            <uni-th align="center">联系电话</uni-th>
            <uni-th align="center">备注</uni-th>
            <uni-th align="center">创建时间</uni-th>
            <uni-th align="center">操作</uni-th>
          </uni-tr>
          <uni-tr v-for="(item, index) in data" :key="index">
            <uni-td align="center">{{ item.booking_id }}</uni-td>
            <uni-td align="center">{{ item.service_id }}</uni-td>
            <uni-td align="center">{{ item.booking_time }}</uni-td>
            <uni-td align="center">{{ item.phone }}</uni-td>
            <uni-td align="center">{{ item.notes }}</uni-td>
            <uni-td align="center">{{ $formatDate(item.create_time) }}</uni-td>
            <uni-td align="center">
              <view class="uni-group">
                <button class="uni-button" size="mini" type="primary" @click="navigateToEdit(item)">编辑</button>
                <button class="uni-button" size="mini" type="warn" @click="confirmDelete(item)">删除</button>
              </view>
            </uni-td>
          </uni-tr>
        </uni-table>
        <view class="uni-pagination-box">
          <uni-pagination show-icon :page-size="pagination.size" v-model:current="pagination.current"
            :total="pagination.count" @change="onPageChanged" />
        </view>
      </unicloud-db>
    </view>

    <uni-popup ref="popup" type="dialog">
      <uni-popup-dialog type="warning" cancelText="关闭" confirmText="确定" title="通知" @confirm="dialogConfirm"
        @close="dialogClose">
        <text class="dialog-text">是否删除该预定记录？</text>
      </uni-popup-dialog>
    </uni-popup>
  </view>
</template>

<script>
  const db = uniCloud.database()
  export default {
    data() {
      return {
        collectionList: ['pet-bookings', 'pet-services'],
        query: '',
        where: '',
        orderby: 'create_time desc',
        selectedItems: [],
        selectedItem: {}
      }
    },
    methods: {
      navigateToAdd() {
        uni.navigateTo({
          url: './add'
        })
      },
      navigateToEdit(item) {
        uni.navigateTo({
          url: './edit?id=' + item._id
        })
      },
      search() {
        const query = this.query.trim()
        if (query) {
          this.where = `phone == '${query}' || booking_id == '${query}'`
        } else {
          this.where = ''
        }
        this.$refs.udb.loadData({
          clear: true
        })
      },
      onPageChanged(e) {
        this.$refs.table.clearSelection()
        this.$refs.udb.loadData({
          current: e.current
        })
      },
      selectedItems(ids) {
        this.selectedItems = ids
      },
      confirmDelete(item) {
        this.selectedItem = item
        this.$refs.popup.open()
      },
      async dialogConfirm() {
        if (this.selectedItem._id) {
          const bookingId = this.selectedItem._id
          try {
            await db.collection('pet-bookings').doc(bookingId).remove()
            uni.showToast({
              title: '删除成功'
            })
            this.$refs.udb.loadData({
              clear: true
            })
          } catch (e) {
            uni.showToast({
              title: '删除失败',
              icon: 'error'
            })
          }
        }
        this.$refs.popup.close()
      },
      dialogClose() {
        this.$refs.popup.close()
      },
      async delTable() {
        const ids = this.selectedItems.map(item => item._id)
        if (ids.length === 0) {
          uni.showToast({
            title: '请选择要删除的记录',
            icon: 'none'
          })
          return
        }
        uni.showModal({
          title: '提示',
          content: '是否删除选中记录',
          success: async (res) => {
            if (res.confirm) {
              try {
                await db.collection('pet-bookings').where({
                  _id: db.command.in(ids)
                }).remove()
                uni.showToast({
                  title: '删除成功'
                })
                this.$refs.udb.loadData({
                  clear: true
                })
                this.$refs.table.clearSelection()
              } catch (e) {
                uni.showToast({
                  title: '删除失败',
                  icon: 'error'
                })
              }
            }
          }
        })
      }
    }
  }
</script>

<style>
  .uni-group {
    display: flex;
    align-items: center;
  }

  .uni-search {
    height: 30px;
    line-height: 30px;
    padding: 0 10px;
    border: 1px solid #ddd;
    margin-right: 10px;
  }

  .dialog-text {
    text-align: center;
  }
</style>