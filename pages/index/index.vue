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
			<u-button type="primary" text="连接" @click="bluetoothFn"></u-button>
			<u-button type="primary" text="开机" @click="send('0xff00')"></u-button>
			<u-button type="primary" text="频率开关" @click="send('0xf00f')"></u-button>
			<view class="">
				{{serviceId}}
			</view>
			<view class="">
				{{deviceId}}
			</view>
			<view class="">
				{{characteristicId}}
			</view>
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
					bluetoothName:'YMW_YE2',
					deviceId: '',
					serviceId:'',
					characteristicId:''
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
					// 初始化蓝牙
					uni.openBluetoothAdapter({
					  success:(res)=>{
					    console.log(1);
					    
					    // 启动蓝牙设备发现
					    uni.startBluetoothDevicesDiscovery({
							success:(res)=>{
					        console.log(res);
					
					        // 监听蓝牙设备发现事件
					        uni.onBluetoothDeviceFound((device)=>{
								// console.log(device.devices[0].name,23)
								if(device.devices[0].name === this.bluetoothName){
									this.deviceId = device.devices[0].deviceId;
									console.log("已經找到設備了");
									// 这里就开始连接了
									uni.createBLEConnection({
									        deviceId: this.deviceId,
									        success:(res)=>{
									            console.log('连接成功')
									            console.log(res)
												
												setTimeout(()=>{
													// 获取服务
													uni.getBLEDeviceServices({
															// 设备ID 前面已经保存
															
													        deviceId: this.deviceId, 
													        success:(res)=>{
													            console.log(res.services,"获取成功")
																this.serviceId = res.services[0].uuid
																// 下一步获取特征值
																this.getCharacteristics()
																
													        },
													        fail:(err)=>{
													            console.error(err)
													        }
													})
													
													// 停止搜索
													this.stopDiscovery()
												},1000)
												
									        },
									        fail:(err)=>{
									            console.log('连接失败')
									            console.error(err)
									        }
									})
									
									
								}
							});
							
					      },
					      fail(e) {
					        console.log('启动蓝牙设备发现失败', e);
					      }
					    })
					  },
					  fail(e) {
					    console.log('初始化蓝牙失败', e);
					  }
					});
				},
				// 停止搜索
				stopDiscovery() {
				    uni.stopBluetoothDevicesDiscovery({
				        success:(res)=>{
				            console.log('停止成功')
				            console.log(res)
				        },
				        fail:(err)=>{
				            console.log('停止失败')
				            console.error(err)
				        }
				    })
				},
				// 获取订阅值
				getCharacteristics(){
					uni.getBLEDeviceCharacteristics({
					        deviceId: this.deviceId, // 设备ID
					        serviceId: this.serviceId, // 服务UUID
					        success:(res)=>{
					            console.log(res,"已经获取特征值")
								this.characteristicId = res.characteristics[0].uuid
								// 开启订阅
								this.startNotice()
					        },
					        fail:(err)=>{
					            console.error(err)
					        }
					    })
				},
				// 消息监听暂时不做
				
				// 发送指令
				send(data){
					let order = data
					console.log(order)
					const buffer = new ArrayBuffer(2);
					const dataView = new DataView(buffer);
					// dataView.setUint8(0, 0x0f); // 设置第一个字节为 00001111
					// dataView.setUint8(1, 0xf0); // 设置第二个字节为 11110000
					dataView.setUint16(0, order,false);
					console.log(dataView)
					
					// for(var i = 0;i<3;i++){
						uni.writeBLECharacteristicValue({
						      deviceId: this.deviceId, // 设备ID
						      serviceId: this.serviceId, // 服务UUID
						      characteristicId: this.characteristicId, // 特征值
						      value: buffer,
						      success:(res)=>{
						        console.log(res,"发送成功")
						      },
						      fail:(err)=>{
						        console.error(err)
						      }
						    })
					// }
				},
				
				// 开启订阅
				startNotice(){
					  setTimeout(()=>{
					
					    uni.notifyBLECharacteristicValueChange({
					           deviceId: this.deviceId, // 设备ID
					           serviceId: this.serviceId, // 服务UUID
					           characteristicId: this.characteristicId, // 特征值
					           success:(res)=>{
					               console.log(res,'开启订阅')
					               
					               // 接受消息的方法
					               // listenValueChange()
					           },
					           fail:(err)=>{
					               console.error(err)
					           }
					       })
					
					   },1000)
				}
				
			},
			
			
	    }
</script>

<style>
</style>
