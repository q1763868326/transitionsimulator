<template>
	<view class="content">
		<view class="title">
			<view class="touxiang">
				<image style="width: 4rem; height: 4rem; background-color: #eeeeee;" mode="aspectFill" :src="touxiangsrc"></image>
			</view>
			<view class="personinfo">
				<view class="username"><text selectable="true" style="text-align: center; width: 100%;">编程变成猪</text></view>
				<view style="display: flex; justify-content: space-around">
					<view class="assets">现金：{{cash}}元</view>
					<view class="rank">
						资产：{{assets}}元
					</view>
				</view>
			</view>
		</view>
		<scroll-view class="products" scroll-y="true" style="height:24rem;">
			<view class="item" v-for="ownproduct in ownproducts" :key="ownproduct">
				<view class="product">
					<image class="productimg" mode="aspectFill" :src="touxiangsrc"></image>
					<view class="productinfo">
						<text>{{ownproduct}}</text>
						<text>成本价：{{getcost(ownproduct)}}元</text>
						<text>现价：{{getprice(ownproduct)}}元</text>
						<text>数量：{{getnum(ownproduct)}}</text>	
					</view>
					<view style="display: flex; align-items: center;">
						<button type="primary" size="mini" @click="open(ownproduct)">出售</button>
					</view>
				</view>
			</view>
		</scroll-view>
		<uni-popup ref="popup" type="dialog">
		<uni-popup-dialog title="出售" mode="input" :duration="2000" @confirm="confirm" :placeholder="placeholder"></uni-popup-dialog>
		</uni-popup>
	</view>
</template>

<script>
	import uniPopup from '@/components/uni-popup/uni-popup.vue'
	import uniPopupDialog from '@/components/uni-popup/uni-popup-dialog.vue'
	import UniPopupMessage from '@/components/uni-popup/uni-popup-message.vue'
	export default {
			components: {uniPopup,uniPopupDialog,UniPopupMessage},
		data() {
			return {
				cash:2000.00,
				assets:2000.00,
				touxiangsrc:"../../static/logo.png",
				names:[],
				prices:[],
				ownproducts:[],
				currentproduct:"",
				currentprice:0,
				maxsail:0,
				placeholder:""
			}
		},
		onLoad() {
			this.getdata()
		},
		onShow() {
			this.getdata()
		},
		methods: {
			open(product){
					this.currentproduct=product
					this.currentprice=this.getprice(product)
					var num = this.getnum(product)
					this.maxsail = num
					 this.placeholder="请输入要出售的数目(最大卖出数："+parseInt(this.maxsail)+")"
			         this.$refs.popup.open()
			      },
				  close(done){
				              // TODO 做一些其他的事情，before-close 为true的情况下，手动执行 done 才会关闭对话框
				              // ...
				              done()
				          },
				          /**
				           * 点击确认按钮触发
				           * @param {Object} done
				           * @param {Object} value
				           */
				  confirm(done,value){
				              // 输入框的值
				            this.cash=parseFloat(this.cash)
							this.currentprice = parseFloat(this.currentprice)
							var value = parseInt(value)
							var ownproductinfo = uni.getStorageSync(this.currentproduct)
							var ownnum = ownproductinfo.ownnum
							var owncost = ownproductinfo.owncost
							var ownproduct = uni.getStorageSync("ownproduct")		
							if(value==ownnum){
								uni.removeStorageSync(this.currentproduct)
								var index = ownproduct.indexOf(this.currentproduct)
								ownproduct.splice(index,1)
								uni.setStorageSync("ownproduct",ownproduct)
							}
							else if(value<ownnum){
								ownnum-=value
								var ownproductinfo = {
									"ownnum":ownnum,
									"owncost":owncost
								}
								uni.setStorageSync(this.currentproduct,ownproductinfo)
							}
							else{
								console.log("error")
								return
							}
				  			this.cash+=value*this.currentprice
				  			this.cash=parseFloat(this.cash).toFixed(2)
				  			uni.setStorageSync("cash",this.cash)
				  			
				  			
				              // TODO 做一些其他的事情，手动执行 done 才会关闭对话框
				              // ...
				              done()
				  			this.update()
				          },
			getdata(){
				uni.request({
				    url: 'http://116.62.47.156/hangyedata',
				    method: 'GET',
				    success: res => {
						this.names=res.data["name"]
						this.prices=res.data["price"]
						this.update()
					},
				    fail: () => {},
				    complete: () => {}
				});
				
			},
			getcost(name){
				var cost = uni.getStorageSync(name).owncost
				return cost
			},
			getprice(name){
				var index = this.names.indexOf(name)
				var price = this.prices[index]
				return price
			},
			getnum(name){
				var num = uni.getStorageSync(name).ownnum
				return num
			},
			update(){
				var cash = uni.getStorageSync("cash")
				if(cash){
					this.cash=cash
				}
				else{
					this.cash=2000.00
				}
				this.assets=this.cash
				this.ownproducts = uni.getStorageSync("ownproduct")
				// console.log(this.ownproducts)
				var sum = 0
				if(this.ownproducts.length>0){
					for(var i=0;i<this.ownproducts.length;i++){
						var index = this.ownproducts[i]
						var productinfo = uni.getStorageSync(index)
						var index2 = this.names.indexOf(index)
						sum+=parseFloat(this.prices[index2])*productinfo.ownnum
					}
					this.assets=sum+parseFloat(this.cash)
					this.assets=parseFloat(this.assets).toFixed(2)
			}
		}
	}
	}
</script>

<style>
	.title{
		display: flex;
		height: 4rem;
	}
	.personinfo{
		display: flex;
		flex-direction: column;
		width: 100%;
	}
	.username{
		height: 2rem;
		width: 100%;
		text-align: center;
		flex-wrap: wrap;
	}
	.products{
		margin-top: 2rem;
	}
	.item{
		display: flex;
		justify-content: space-around;
		margin-bottom: 1.5rem;
	}
	.product{
		display: flex;
		width: 100%;
		height: 4rem;
		justify-content: space-around;
	}
	.productimg{
		width: 3rem;
		height: 3rem;
	}
	.productinfo{
		display: flex;
		height: 6rem;
		flex-direction: column;
		align-items: center;
		font-size: 0.8rem;
	}
</style>
