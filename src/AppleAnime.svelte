<script>
	import numeral from "numeral";

	let name = 'Anime';
	const width = document.documentElement.clientWidth;
	const height = document.documentElement.clientHeight;
	const canvasId = 'scroll-player';
	const imagesLength = 176; // 图片总数量
	let flag = false;
	let boxHeight;

	/**
	 * 获取图片路径
	 */
	function getImagesPath() {
		let images = [];
		const baseUrl = 'https://www.apple.com.cn/105/media/us/airpods-pro/2019/1299e2f5_9206_4470_b28e_08307a42f19b/anim/sequence/large/06-transparency-head/';
		for(let i = 0; i < imagesLength; i++) {
			images.push(`${baseUrl}${numeral(i).format('0000')}.jpg`);
		}

		return images;
	}

	let imagesManager = [];
	let imagesPath = getImagesPath(); // 图片路径数字集合
	let canvas;
	let context;
	/** 加载图片 */ 
	function loadImages() {
		const loadNextImage = (src) => {
			const img = new Image();
			// 同步加载, 可优化为异步
			img.onload = (e) => {
				imagesManager.push(img);
				if (imagesManager.length === imagesLength) {
					// 代表所有图片加载完成 执行回调方法
					flag = true;
					imagesLoadComplete();
				}
			}

			img.src = src;
			if (imagesPath.length === 0) return;
			loadNextImage(imagesPath.shift());
		};

		loadNextImage(imagesPath.shift());
		
	}

	function init() {
		boxHeight = document.querySelector('.scroll-player-container').clientHeight - document.documentElement.clientHeight;
		canvas = document.getElementById(canvasId);
		context = canvas.getContext('2d');
		// 加入scroll事件监听
		document.addEventListener('scroll', handleScroll);
		// 执行加载每一帧的所有图片
		loadImages();
	}

	let scrollIndex = 0; // 当前滚动进度待显示的图片索引值
	let currentIndex = 0; // 当前显示的图片索引值
	let raf = null;

	/** 图片加载完成回调 */ 
	function imagesLoadComplete() {
		console.log('游戏 🎮 开始了哟!');
		GameRun();
	}

	function GameRun() {
		raf = window.requestAnimationFrame(draw);
	}

	/**
	 * 处理滑动边界状态
	 */
	function draw() {
		if (currentIndex <= scrollIndex) {
			drawImages(imagesManager[currentIndex]);
			currentIndex + 1 < scrollIndex && currentIndex++;
		} else if (currentIndex >= scrollIndex) {
			drawImages(imagesManager[currentIndex]);
			currentIndex - 1 > scrollIndex && currentIndex --;
		}

		if (currentIndex > imagesLength) {
			currentIndex = imagesLength;
		}

		raf = window.requestAnimationFrame(draw);
	}

	/**
	 * 画布画图
	 */
	function drawImages(img) {
		context.clearRect(0, 0, width, height);
		context.drawImage(img, 0, 0);
	}

	/**
	 * 鼠标滚动事件回调, 计算出scrollIndex
	 */
	function handleScroll() {		
		const docElement = document.documentElement;
		const scrollHeight = docElement.scrollHeight;
		const clientHeight = docElement.clientHeight;
		const scrollTop = docElement.scrollTop;
		let share = boxHeight / imagesLength;
		// 根据滚动距离, 等比例算出应该滚动到第几张图
		// scrollIndex = Math.round(scrollTop * imagesLength / (scrollHeight - clientHeight));
		scrollIndex = Math.round(document.documentElement.scrollTop / share);
		// console.log('compute', Math.round(boxHeight / imagesLength));
		console.log('compute',document.documentElement.scrollTop, Math.round(document.documentElement.scrollTop / share));
	}


	window.onload = () => {
		init();
	}

</script>

<main>
	
		<div style="visibility: {flag ? 'visible' : 'hidden' };" class="scroll-player-container" > 
			<div class="scroll-sequence" >
				<div class="image-sequence" >
					<div class="canvas-container" >
						<canvas id={canvasId} {width} {height} style="background-color: #000;" ></canvas>
					</div>
				</div>
			</div>
		</div>
		
		<h1 class="loading" style="visibility: {flag ? 'hidden' : 'visible' };" >加载中</h1>

</main>

<style>
.scroll-player-container {
  height: 500vh;
  width: 100%;
  position: relative;
}

.scroll-sequence {
  position: absolute;
  top: 0;
  width: 100%;
  height: 100%;
}

.image-sequence {
  position: sticky;
  top: 0;
  overflow: hidden;
}

.canvas-container {
  position: relative;
  width: 100%;
  height: 100vh;
}

.loading {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translateX(-50%);
}

</style>