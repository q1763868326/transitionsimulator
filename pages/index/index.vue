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
			<view class="item" v-for="i in 33" :key="i">
				<view class="product">
					<image class="productimg" mode="aspectFill" :src="touxiangsrc"></image>
					<view class="productinfo">
						<text>{{names[i*2-2]}}</text>
						<text>单价：{{prices[i*2-2]}}元</text>
						<button type="primary" size="mini" @click="open(names[i*2-2],prices[i*2-2])">购买</button>
					</view>
				</view>
				<view class="product">
						<image class="productimg" mode="aspectFill" :src="touxiangsrc"></image>
						<view class="productinfo">
							<text>{{names[i*2-1]}}</text>
							<text>单价：{{prices[i*2-1]}}元</text>
							<button type="primary" size="mini" @click="open(names[i*2-1],prices[i*2-1])">购买</button>
						</view>
				</view>
			</view>
		</scroll-view>
		<uni-popup ref="popup" type="dialog">
		<uni-popup-dialog title="购买" mode="input" :duration="2000" @confirm="confirm" :placeholder="placeholder"></uni-popup-dialog>
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
				currentproduct:"",
				currentprice:0,
				maxbuy:0,
				placeholder:""
			}
		},
		onLoad() {
			// uni.clearStorage()
			this.getdata()
		},
		onShow() {
			this.getdata()
		},
		methods: {
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
			update(){
				var cash = uni.getStorageSync("cash")
				if(cash){
					this.cash=cash
				}
				else{
					this.cash=2000.00
				}
				this.assets=this.cash
				var ownproduct = uni.getStorageSync("ownproduct")
				var sum = 0
				if(ownproduct){
					for(var i=0;i<ownproduct.length;i++){
						var index = ownproduct[i]
						var productinfo = uni.getStorageSync(index)
						var index2 = this.names.indexOf(index)
						sum+=parseFloat(this.prices[index2])*productinfo.ownnum
					}
					this.assets=sum+parseFloat(this.cash)
					this.assets=this.assets.toFixed(2)
				}
			},
			open(product,price){
					this.currentproduct=product
					 this.currentprice=price
					 if(this.cash<=0){
						 this.maxbuy=0
					 }
					 else{
						 this.maxbuy=this.cash/this.currentprice
					 } 
					 this.placeholder="请输入购买数目(最大购买数："+parseInt(this.maxbuy)+")"
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
						if(value>this.maxbuy){
							return
						}
						
						this.cash-=value*this.currentprice
						this.cash=this.cash.toFixed(2)
						uni.setStorageSync("cash",this.cash)
						var ownproductinfo = uni.getStorageSync(this.currentproduct)
						var ownnum = ownproductinfo.ownnum
						var owncost = ownproductinfo.owncost
						if(owncost){
							owncost = (parseFloat(this.currentprice)*parseInt(value)+parseFloat(owncost)*parseInt(ownnum))
						}
						else{
							owncost = parseFloat(this.currentprice)
						}
						if(ownnum){
							ownnum=parseInt(ownnum)+parseInt(value)
							owncost = (owncost/ownnum).toFixed(2)
						}
						else{
							ownnum=parseInt(value)
							var ownproduct = uni.getStorageSync("ownproduct")
							if(ownproduct){
								ownproduct.push(this.currentproduct)
								uni.setStorageSync("ownproduct",ownproduct)
							}
							else{
								var a=[this.currentproduct];
								uni.setStorageSync("ownproduct",a)
							}
						}
						
						var ownproductinfo = {
							"ownnum":ownnum,
							"owncost":owncost
						}
						uni.setStorageSync(this.currentproduct,ownproductinfo)
			            // TODO 做一些其他的事情，手动执行 done 才会关闭对话框
			            // ...
			            done()
						this.update()
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
	}
	.product{
		display: flex;
		height: 4rem;
	}
	.productimg{
		width: 3rem;
		height: 3rem;
	}
	.productinfo{
		display: flex;
		height: 4rem;
		flex-direction: column;
		align-items: center;
		font-size: 0.8rem;
	}
	
</style>
