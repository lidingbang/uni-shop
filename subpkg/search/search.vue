<!-- 我的页面  -->
<template>
  <view>
		<view class="search-box">
		  <uni-search-bar @confirm="search" @input="input" :radius="100"  cancelButton="none"></uni-search-bar>
		</view>
		
		<!-- 搜索建议 -->
		<view class="sugg-list" v-if="searchResult.length !== 0">
			<view class="sugg-item" v-for="(item, index) in searchResult" :key="index" @click="gotoDetail(item)">
				<view class="goods-name">{{ item.goods_name }}</view>
				<uni-icons type="arrowright" size="16"></uni-icons>
			</view>
		</view>
		
		<!-- 搜索歷史 -->
		<view class="history-box" v-else>
			<view class="history-title">
				<text>搜索歷史</text>
				<uni-icons type="trash" size="17" @click="clean"></uni-icons>
			</view>
			<!-- 標題列表 -->
			<view class="history-list">
				<uni-tag :text="item" v-for="(item, index) in historyList" :key="index" @click="gotoGoodsList(item)"></uni-tag>
			</view>
		</view>
	</view>
</template>

<script>
  // 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
  // 例如：import 《组件名称》 from '《组件路径》'

  export default {
    name: '',
    // import引入的组件需要注入到对象中才能使用
    props: {},
    data () {
      // 这里存放数据,
      return {
				timer: null,
				kw: '',
				searchResult: [],
				historyList: ['aa', 'bb']
      }
    },
    // 监听属性 类似于data概念
    computed: {
			history() {
				// 注意，由於數組是應用類型，所以不要直接基於原數組調用 reverse 方法 ，以免修改原數組中元素的順序
				return [...this.historyList.reverse()]
			}
		},
    // 监控data中的数据变化
    watch: {},
    // 生命周期 - 创建完成（可以访问当前this实例）
    created () {

    },
    // 生命周期 - 挂载完成（可以访问DOM元素）
    mounted () {

    },
    beforeCreate () { }, // 生命周期 - 创建之前
    beforeMount () { }, // 生命周期 - 挂载之前
    beforeUpdate () { }, // 生命周期 - 更新之前
    updated () { }, // 生命周期 - 更新之后
    beforeDestroy () { }, // 生命周期 - 销毁之前
    destroyed () { }, // 生命周期 - 销毁完成
    activated () { }, // 如果页面有keep-alive缓存功能，这个函数会触发
    onLoad () {
			this.historyList = JSON.parse(uni.getStorageSync('kw') || '[]')
    },
    // 方法集合
    methods: {
			input(e) {
				console.log(e)
				clearTimeout(this.timer)
				this.timer = setTimeout(() => {
					this.kw = e.value
					
					this.getSearchList()
				}, 500)
			},
			
			async getSearchList() {
				// 判断关键词是否为空
				if (this.kw.length === 0) {
					this.searchResult = []
					return
				}
				
				const { data: res } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: this.kw })
				if (res.meta.status !== 200) return uni.$showMsg()
				this.searchResult = res.message
				
				// 在創建列表前調用搜索歷史
				this.saveSearchHistory()
			},
			
			gotoDetail (item) {
				uni.navigateTo({
					url: '/subpkg/goods_detail/goods_detail?goods_id=' + item.goods_id
				})
			},
			
			saveSearchHistory () {
				const set = new Set(this.historyList)
				set.delete(this.kw)
				set.add(this.kw)
				this.historyList = Array.from(set)
				
				// 調用 uni.setStorageSync(key, value) 將搜索歷史記錄持久化存儲到本地
				uni.setStorageSync('kw', JSON.stringify(this.historyList || '[]'))
			},
			
			clean () {
				this.historyList = [],
				uni.setStorageSync('kw', '[]')
			},
			
			gotoGoodsList (kw) {
				uni.navigateTo({
					url: '/subpkg/goods_list/goods_list?query=' + kw
				})
			}
    }
  }
</script>

<style lang='scss' scoped>
	.search-box {
		position: sticky;
		top: 0;
		z-index: 999;
	}
	.history-box {
		padding: 0 5px;
		.history-title {
			display: flex;
			justify-content: space-between;
			height: 40px;
			align-items: center;
			font-size: 13px;
			border: 1px solid #efefef;
		}
		.history-list {
			display: flex;
			flex-wrap: wrap;
			
			.data-v-1480e53a {
				margin-top: 5px;
				margin-right: 5px;
			}
		}
	}
</style>
