<template>
	<view class="page">
		<view class="top">
			<view class="score">
				得分:{{total}}
			</view>
			<view class="time">
				用时:{{allTime}}s
			</view>
		</view>
		        <!-- 此处放你要监听的内容 -->
				<view class="center">
					<wyb-slide-listener @slideUp="onSlide" @slideDown="onSlide" @slideLeft="onSlide" @slideRight="onSlide">
			<view class="mainBox">
				<view class="row" v-for="(row, rowIndex) in gameBoard" :key="rowIndex">
					<view class="cell" v-for="(cell, cellIndex) in row" :key="cellIndex">
						<!-- 	<view :class="cell!==0?'cellBox':''"> -->
						<view
							:class="cellIndex==newArr[0][1]&&rowIndex==newArr[0][0]||cellIndex==newArr[1][1]&&rowIndex==newArr[1][0]?'newBox':cell!==0?'cellBox':''">
							<view class="colorBox"
								:style="{backgroundColor:cell==2?'#ff3a3a':cell==4?'#ff9b29':cell==8?'#ebff31':cell==16?'#34ff31':cell==32?'#369083':cell==64?'#2e3cff':cell==128?'#c12fff':cell==256?'#ff77ed':cell==512?'#ffe9fe':cell==1024?'#fffcd4':cell==2048?'#04010b':''}">
								<text v-show=" cell!==0&&cell!==1">{{ cell }}</text>
							</view>
 
						</view>
 
					</view>
				</view>
			</view>
			</wyb-slide-listener>
		</view>
		
		<view class="bottom">
			<view class="kaishi" v-show="gameStatus==false">
				<view class="flexBox"> <button @click="gameStart()"> 游戏开始</button></view>
			</view>
			<view class="jinxing" v-show="gameStatus==true">
				<view class="flexBox">
					<view class="gameOver">
						<view class="gameOverButton" @click="gameOver()">
							结束
						</view>
					</view>
					 
 
				</view>
 
			</view>
		</view>
	</view>
</template>
 
<script lang="ts" setup>
	import wybSlideListener from '@/components/wyb-slide-listener/wyb-slide-listener.vue'
	import { ref } from 'vue'
	// 游戏状态
	const gameStatus = ref<boolean>(false);
	// 显示的数组
	let gameBoard = ref<number[][]>(Array.from({ length: 4 }, () => Array(4).fill(0)));
	// 新增的俩
	let newArr = ref<number[][]>(Array.from({ length: 2 }, () => Array(2).fill(null)))
	// 得分
	const total = ref<number>();
	// 用时
	const allTime = ref(0)
	const timer1 = ref()
	// 游戏开始
	const gameStart = () => {
		total.value = 0;
		allTime.value = 0
		gameStatus.value = true;
		gameBoard.value = numInit()
		timer1.value = setInterval(() => {
			allTime.value = allTime.value + 1;
		}, 1000)
	}
	// 游戏结束
	const gameOver = () => {
		gameStatus.value = false;
		clearInterval(timer1.value)
		timer1.value = null;
		newArr.value = Array.from({ length: 2 }, () => Array(2).fill(null));
 
	}
	// 获取随机数的函数
	const getRandomlet = (min, max) => {
		min = Math.ceil(min);
		max = Math.floor(max);
		return Math.floor(Math.random() * (max - min + 1)) + min;
	}
	// 随机初始化数值
	const numInit = () => {
		const array = Array.from({ length: 4 }, () => Array(4).fill(0));
		const positions = [];
		// 生成一个包含所有可能位置的数组  
		for (let i = 0; i < 4; i++) {
			for (let j = 0; j < 4; j++) {
				positions.push({ x: i, y: j });
			}
		}
		// 随机选择6个位置  
		const selectedPositions = [];
		for (let i = 0; i < 6; i++) {
			const randomIndex = getRandomlet(0, positions.length - 1);
			selectedPositions.push(positions[randomIndex]);
			positions.splice(randomIndex, 1); // 从数组中移除已选位置，避免重复选择  
		}
		// 设置前4个位置为2  
		for (let i = 0; i < 4; i++) {
			const position = selectedPositions[i];
			array[position.x][position.y] = 2;
		}
		// 对于剩下的2个位置，随机设置为4或8  
		for (let i = 4; i < 6; i++) {
			const position = selectedPositions[i];
			const randomValue = getRandomlet(1, 2) === 1 ? 4 : 8;
			array[position.x][position.y] = randomValue;
		}
		return array;
	}
	// 旋转数组
	const rotate90Clockwise = (matrix) => {
		const n = matrix.length;
		let rotatedMatrix = Array.from({ length: n }, () => []);
 
		// 顺时针旋转90度
		for (let i = 0; i < n; i++) {
			for (let j = 0; j < n; j++) {
				rotatedMatrix[j][n - i - 1] = matrix[i][j];
			}
		}
 
		return rotatedMatrix;
	}
	// 累计与填入
	const addNum = (arr) => {
		let copiedArray = JSON.parse(JSON.stringify(arr));
		let defen = 0;
		for (let i = 0; i < copiedArray.length; i++) {
			for (let j = 0; j < copiedArray[i].length; j++) {
				// 找到第一个不为0
				if (copiedArray[i][j] !== 0) {
					for (let p = 0; p < j; p++) {
						if (copiedArray[i][p] == copiedArray[i][j]) {
							copiedArray[i][p] = copiedArray[i][j] + copiedArray[i][p];
							defen = defen + copiedArray[i][p] / 2;
							copiedArray[i][j] = 0;
						}
						// 移动到第一个0
						if (copiedArray[i][p] == 0) {
							copiedArray[i][p] = copiedArray[i][j];
							copiedArray[i][j] = 0;
						}
					}
				}
 
			}
		}
		total.value = total.value + defen
		return copiedArray;
	}
	// 添加新数字
	const addRandomNumbersToZeros = (arr) => {
		let matrix = JSON.parse(JSON.stringify(arr));
		// 存储所有值为0的元素的坐标  
		let zeroIndices = [];
 
		// 遍历二维数组，找到值为0的元素的坐标  
		for (let i = 0; i < matrix.length; i++) {
			for (let j = 0; j < matrix[i].length; j++) {
				if (matrix[i][j] === 0) {
					zeroIndices.push([i, j]);
				}
			}
		}
 
		// 如果没有0，则无法添加数字  
		if (zeroIndices.length < 2) {
			gameOver()
			return;
		}
 
		// 从所有0的坐标中随机选择两个  
		let randomIndices = zeroIndices.sort(() => 0.5 - Math.random()).slice(0, 2);
 
		// 为这两个坐标对应的元素添加随机数字  
		let randomNumbers = [2, 4, 8];
		for (let index of randomIndices) {
			let [row, col] = index;
			let randomNumber = randomNumbers[Math.floor(Math.random() * randomNumbers.length)];
			matrix[row][col] = randomNumber;
		}
		newArr.value = randomIndices;
		return matrix;
	}
 
	// 移动
	const moveAndMerge = (dir) => {
		if (dir == 'shang') {
			gameBoard.value = addNum(gameBoard.value)
		}
		else if (dir == 'zuo') {
			let newArr = JSON.parse(JSON.stringify(gameBoard.value));
			newArr = rotate90Clockwise(addNum(rotate90Clockwise(rotate90Clockwise(rotate90Clockwise(newArr)))))
			gameBoard.value = newArr
		} else if (dir == 'you') {
			let newArr = JSON.parse(JSON.stringify(gameBoard.value));
			newArr = rotate90Clockwise(rotate90Clockwise(rotate90Clockwise(addNum(rotate90Clockwise(newArr)))))
			gameBoard.value = newArr
		} else if (dir == 'xia') {
			let newArr = JSON.parse(JSON.stringify(gameBoard.value));
			newArr = rotate90Clockwise(rotate90Clockwise(addNum(rotate90Clockwise(rotate90Clockwise(newArr)))))
			gameBoard.value = newArr
		}
		gameBoard.value = addRandomNumbersToZeros(gameBoard.value)
	}
	// 操作  
	 
	const onSlide = (e: any) => {
        let type = e.type
        console.log(e)
        switch(type) {
            case 'slideUp':
                moveAndMerge('shang')
                break
            case 'slideDown':
                moveAndMerge('xia')
                break
            case 'slideLeft':
                moveAndMerge('zuo')
                break
            case 'slideRight':
                moveAndMerge('you')
                break
        }
    }
</script>
 
<style lang="scss" scoped>
	.page {
		width: 100vw;
		overflow: hidden;
		height: 100vh;
		background-color: #c6ffe6;
		display: flex;
		flex-direction: column;
		font-family: cuteFont;
 
		.top {
			width: 80%;
			height: 20vw;
			display: flex;
			align-items: center;
			margin-left: 10%;
			font-size: 2rem;
 
			.score {
				flex: 1;
 
			}
 
			.time {
				flex: 1;
 
			}
		}
 
		.center {
			width: 100vw;
			height: 100vw;
 
 
			.mainBox {
				width: 80%;
				margin: 10% 10%;
				height: 80%;
				border-radius: 15px;
				display: flex;
 
				.row {
					flex: 1;
					display: flex;
					flex-direction: column;
				}
 
				.cell {
					flex: 1;
					border: 1px solid #ff80c2;
					background-color: #b5f2ff;
					display: flex;
					justify-content: center;
					align-items: center;
					color: #ffffff;
					font-size: 2rem;
 
					.newBox {
						width: 90%;
						height: 90%;
						background-color: #9d6fff;
						border-radius: 15px;
						display: flex;
						justify-content: center;
						align-items: center;
						animation: newBox 0.5s;
					}
 
					.cellBox {
						width: 90%;
						height: 90%;
						background-color: #9d6fff;
						border-radius: 15px;
 
					}
 
					.colorBox {
						width: 100%;
						border-radius: 15px;
						height: 100%;
						display: flex;
						justify-content: center;
						align-items: center;
					}
				}
			}
		}
 
		.bottom {
			flex: 1;
			position: relative;
 
			.kaishi {
				width: 100%;
				height: 100%;
				background-color: #86ff61;
				position: absolute;
				.flexBox {
					width: inherit;
					height: inherit;
					display: flex;
					justify-content: center;
					align-items: center;
				}
			}
 
			.jinxing {
				width: 100%;
				height: 100%;
				position: absolute;
 
				.flexBox {
					width: inherit;
					height: inherit;
					display: flex;
					flex-direction: row;
 
					.contorl {
						flex: 1;
 
 
						.shang {
							width: 40px;
							height: 40%;
							position: absolute;
							left: 50%;
							background-color: #ff0777;
							clip-path: polygon(0% 50%, 50% 0%, 100% 50%, 80% 50%, 80% 100%, 20% 100%, 20% 50%);
						}
 
						.shang:hover {
							border: 1px solid #3d37ff;
						}
 
						.xia {
							width: 40px;
							height: 40%;
							position: absolute;
							top: 50%;
							left: 50%;
							background-color: #ff0777;
							clip-path: polygon(20% 0%, 80% 0%, 80% 50%, 100% 50%, 50% 100%, 0% 50%, 20% 50%);
						}
 
						.xia:hover {
							border: 1px solid #3d37ff;
						}
 
						.zuo {
							width: 120px;
							height: 40px;
							position: absolute;
							top: calc(50% - 30px);
							left: calc(50% - 120px);
							background-color: #ff0777;
							clip-path: polygon(0% 50%, 50% 0%, 50% 20%, 100% 20%, 100% 80%, 50% 80%, 50% 100%);
						}
 
						.zuo:hover {
							border: 1px solid #3d37ff;
						}
 
						.you {
							width: 120px;
							height: 40px;
							position: absolute;
							top: calc(50% - 30px);
							left: calc(50% + 40px);
							background-color: #ff0777;
							clip-path: polygon(0% 20%, 50% 20%, 50% 0%, 100% 50%, 50% 100%, 50% 80%, 0% 80%);
						}
 
						.you:hover {
							border: 1px solid #3d37ff;
						}
					}
 
					.gameOver {
						.gameOverButton {
							width: 50px;
							height: 100%;
							font-size: 2rem;
							display: flex;
							justify-content: center;
							align-items: center;
							background-color: #fff;
							border-radius: 0 15px 0 0;
							border: 1px solid #a860ff;
						}
 
					}
 
				}
 
			}
		}
	}
 
	@keyframes newBox {
		0% {
			width: 0%;
			height: 0%;
		}
 
		100% {
			width: 90%;
			height: 90%;
		}
	}
</style>