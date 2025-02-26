<template>
  <view class="container">
    <view class="uni-header">
      <view class="uni-group">
        <button class="uni-button" type="primary" @click="navigateToAdd">新增</button>
        <button class="uni-button" type="warn" @click="delTable" :disabled="!selectedItems.length">批量删除</button>
      </view>
    </view>
    <view class="uni-container">
      <unicloud-db ref="udb" v-slot:default="{data, pagination, loading, error, options}" :collection="collectionList" field="name,description,price,image_url,create_date" :where="where" page-data="replace" :orderby="orderby" :getcount="true" :page-size="options.pageSize" :page-current="options.pageCurrent" @load="onqueryload">
        <uni-table ref="table" :loading="loading" :emptyText="error.message || '没有更多数据'" border stripe type="selection" @selection-change="selectionChange">
          <uni-tr>
            <uni-th align="center">服务名称</uni-th>
            <uni-th align="center">描述</uni-th>
            <uni-th align="center">价格</uni-th>
            <uni-th align="center">图片</uni-th>
            <uni-th align="center">创建时间</uni-th>
            <uni-th align="center">操作</uni-th>
          </uni-tr>
          <uni-tr v-for="(item,index) in data" :key="index">
            <uni-td align="center">{{item.name}}</uni-td>
            <uni-td align="center">{{item.description}}</uni-td>
            <uni-td align="center">￥{{item.price}}</uni-td>
            <uni-td align="center">
              <image v-if="item.image_url" :src="item.image_url" mode="aspectFit" style="width: 50px; height: 50px;"></image>
            </uni-td>
            <uni-td align="center">{{$formatDate(item.create_date)}}</uni-td>
            <uni-td align="center">
              <view class="uni-group">
                <button class="uni-button" size="mini" type="primary" @click="navigateToEdit(item._id)">编辑</button>
                <button class="uni-button" size="mini" type="warn" @click="confirmDelete(item._id)">删除</button>
              </view>
            </uni-td>
          </uni-tr>
        </uni-table>
        <view class="uni-pagination-box">
          <uni-pagination show-icon :page-size="pagination.size" v-model:current="pagination.current" :total="pagination.count" @change="onPageChanged" />
        </view>
      </unicloud-db>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        collectionList: 'pet-services',
        where: '',
        orderby: 'create_date desc',
        selectedItems: []
      }
    },
    methods: {
      onqueryload(data) {
        this.selectedItems = []
      },
      selectionChange(e) {
        this.selectedItems = e.detail.index
      },
      onPageChanged(e) {
        this.$refs.table.clearSelection()
        this.$refs.udb.loadData({
          current: e.current
        })
      },
      navigateToAdd() {
        uni.navigateTo({
          url: './add'
        })
      },
      navigateToEdit(id) {
        uni.navigateTo({
          url: './edit?id=' + id
        })
      },
      confirmDelete(id) {
        uni.showModal({
          title: '提示',
          content: '确定要删除该服务吗？',
          success: (res) => {
            if (res.confirm) {
              this.deleteService(id)
            }
          }
        })
      },
      async deleteService(id) {
        try {
          const db = uniCloud.database()
          await db.collection('pet-services').doc(id).remove()
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
      },
      async delTable() {
        uni.showModal({
          title: '提示',
          content: '确定要删除选中的服务吗？',
          success: async (res) => {
            if (res.confirm) {
              const db = uniCloud.database()
              const selectedIds = this.selectedItems.map(i => this.$refs.udb.dataList[i]._id)
              try {
                await db.collection('pet-services').where({
                  _id: db.command.in(selectedIds)
                }).remove()
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

  .uni-button {
    margin: 0 5px;
  }
</style>