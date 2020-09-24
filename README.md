# threejsDemo
css3d.html 是真正的最后做出来的全景图切换demo，有场景切换，移动端能缩放场景，上下左右查看，还有箭头摆放上下的小动画  

threejs文件夹下是css3d.html需要的素材和js文件  
这2句是启动实景图的关键：  
```
	    init();//初始化
	    animate();//循环渲染，跟随相机角度变化
```
难点是手势的冲突，一个手指上下左右滑动，2个手指则是缩放    
而手指缩放的时候，经常是先一个手指下去，2个手指一起滑动，然后一个手指抬起，最后都抬起  
所以触发函数顺序是touchstart(case 1)=>touchmove(case 2)*N=>touchmove(case 1)  
lon lan的改动会影响视野上下左右移动，在touchmove 1的时候才会改变，所以  
我们必须保证lon、lan不是夹在双指滑动的前后进行改变，但是前很难判断，因为不知道什么时候会双指，不过touchstart阶段不会直接改lan、lon所以不管，双指期间scalestatus设置为true，表示刚刚双指滑过  
而touchmove（case 1)需要scalestatus为false【刚刚没有双指滑动过】时才能动态修改lan 、lon  
                      若scalestatus为true。延时，等待小段时间后再设置为false，这延时期间不给它修改lan、lon  
```
	               var camerafov=75;
			var start;
			var start0;
			var now;
			var scalestatus=false;
			//当手指触摸屏幕时候触发，即使已经有一个手指放在屏幕上也会触发
			function onDocumentTouchStart( event ) {//移动端开始触碰
				event.preventDefault();
				switch (event.touches.length) {
					case 1:
						console.log('1个手指触碰');
						var touch = event.touches[ 0 ];
								touchX = touch.screenX;
								touchY = touch.screenY;

						start0=touch;
						break;
					case 2:// 第一个触摸点的坐标
						console.log('2个手指触屏');
						 start =event.touches;
				}

			}
			//当手指在屏幕上滑动的时候连续地触发。在这个事件发生期间，调用preventDefault()事件可以阻止滚动
			function onDocumentTouchMove( event ) {//移动端移动
				event.preventDefault();
				switch (event.touches.length) {
					case 1:
						console.log('1个手指Move');
						var touch = event.touches[ 0 ];
							if(scalestatus){//刚刚缩放过
								console.log('刚刚缩放过不滑动');
								setTimeout(function(){
									scalestatus=false;//延时一小会，因为它会滑动一段的
								},60)
								break;
							}else{
								console.log('正常滑动');
								lon -= ( touch.screenX - touchX ) * 0.1;//当鼠标在横向移动时，经度跟着移动；
								lat += ( touch.screenY - touchY ) * 0.1;//当鼠标在纵向向移动时，纬度跟着移动；
								touchX = touch.screenX;
								touchY = touch.screenY;
								scalestatus=false;
								break;
							}
					case 2:
						console.log('2个手指缩放');
						 now = event.touches;
						if(getDistance(now[0], now[1]) < getDistance(start[0], start[1])) {//缩小
							camerafov=camerafov+1;
							if(camerafov>75){camerafov=75;}
							camera.fov = camerafov;
							camera.updateProjectionMatrix();

						}else{
							camerafov=camerafov-1;
							if(camerafov<20)camerafov=20;
							camera.fov = camerafov;
							camera.updateProjectionMatrix();
						}
						scalestatus=true;
			}

			}
```

animatewx.html是百度地图的全景地图api做的一个，不需要怎么开发，仅仅替换经纬度、script ak=去官网可申请替换这个ak，这个文件已上线，可查看https://tvu.zykjedu.cn/mini/animatewx.html  
mapsimple.html 是根据教程做个一个立方体不挺的旋转，没啥功能，仅供参考  
map.html 是网上根据一个网友代码写的全景图切换，不同的是它是球形且只有一张图，cdn失效后换成我本地的js就不行了……，仅供参考  
