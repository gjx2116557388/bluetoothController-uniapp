<template>
	<view>
		<view>
			<u-swiper
			        :list="list3"
			        indicator
			        indicatorMode="line"
			        circular
			></u-swiper>
		</view>
		
		<view>
			<u-picker :show="show" :columns="columns" @confirm="confirm" @cancel='cancel' closeOnClickOverlay @close="close"></u-picker>
			<!-- <u-button @click="show = true">选择型号</u-button> -->
			<u-button color="linear-gradient(to right, rgb(66, 83, 216), rgb(213, 51, 186))" icon="list-dot" text="选择设备" @click="show = true"></u-button>
			<u-button type="primary" text="确定" @click="bluetoothFn"></u-button>
		</view>
	</view>
</template>

<script>
	export default {
	        data() {
	            return {
					bluetoothData: [
						{
						    deviceName: "阳具",
						    features: [
						      {
						        name: '开关',
						        code: '10offo',
						      },
						      {
						        name: '加热',
						        code: '231xcd',
						      },
						      // 可以添加更多功能
						    ],
						  },
						  // 可以添加更多蓝牙设备
					],
	                list3: [
	                    'https://cdn.uviewui.com/uview/swiper/swiper3.png',
	                    'https://cdn.uviewui.com/uview/swiper/swiper2.png',
	                    'https://cdn.uviewui.com/uview/swiper/swiper1.png',
	                ],
					show: false,
					columns: [
						['阳具', '娃娃', '倒模']
					],
	            }
	        },
			methods:{
				confirm(e) {
					console.log('confirm', e.value)
					this.show = false
				},
				cancel(){
					console.log('cancel')
					this.show = false
				},
				close() {
				        this.show = false
				        // console.log('close');
				},
				bluetoothFn(){
					console.log(21312)
					// 初始化蓝牙
					uni.openBluetoothAdapter({
					  success(res) {
					    console.log(res)
						// 启动蓝牙设备发现
						uni.startBluetoothDevicesDiscovery({
						  services: [],
						  success(res) {
						    console.log(res)
						  }
						})
					  }
					})
				}
				
			}
			
	    }
</script>

<style>
</style>
