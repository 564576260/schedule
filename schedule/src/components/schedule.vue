<template>
	<div id="scheduleTemplate">
		<div class="cover" v-show="cardOpen" @click="closeCard"></div>
		<h1 class="scheduleTemplateTitle">Schedule template</h1>
		<div class="logBtn" @click="showLog = true;">查看用户操作日志</div>
		<div class="logBg" v-show="showLog" @click="showLog = false">
			<div class="logContent" @click.stop>
				<h5>操作日志</h5>
				<p v-html="log"></p>				
			</div>
		</div>
		<div class="schedule clearfix">
			<div class="timeLine">
				<div v-for="time in timeLineForDisplay" :key='time'>{{time}}</div>
			</div>
			<div class="calendar clearfix">
				<div class="dayList" v-for="(us,i) in userSchedule" :key='i'>
					<div class="dayName">{{us.dayName}}</div>
					<div class="dayBlocks">
						<div class="halfHourBlock" v-for="(block,bIndex) in timeLineForTable" :key='bIndex' @click="newCard(i,bIndex)"></div>
					</div>
					<div class="daySchedule" :style="{width:calendar.length*100+'%'}">
						<div class="card" v-for="(c,j) in us.schedule" :key='j' :class="{cardActive: c.active}" :style="{width:100/calendar.length+'%',height:((c.endTimeInt-c.startTimeInt)/30*40) + 'px',top:(c.startTimeInt-str2min(timeLineStart))/15*20+40+'px',left:i/calendar.length*100+'%',background:c.bgcolor}">
							<div v-if="c.active" class="lgCard">
								<div class="lgCardLeft">
									<div class="lgTime" @click="changeTime(c,i,j)">{{c.startTimeStr}} - {{c.endTimeStr}}</div>
									<div class="lgTitle">
										<!--<span>{{c.title}}</span>-->
										<input v-model="c.title" type="text" name="" id="" value="" />
									</div>
								</div>
								<div class="lgCardRight">
									<div class="deleteCard" @click="deleteCard(i,j)"></div>
									<div class="closeCard" @click="closeCard"></div>
									<!--<p class="lgContent">{{c.content}}</p>-->
									<textarea name="" rows="" cols="" v-model="c.content"></textarea>
								</div>
							</div>
							<div v-else class="smCard" @click="openCard(c,i)">
								<div class="smTime">{{c.startTimeStr}} - {{c.endTimeStr}}</div>
								<p class="smTitle">{{c.title}}</p>
							</div>
						</div>
					</div>
				</div>
				<div class="timeChooser" v-show="chooserData.show">
					<select name="" v-model="chooserData.startTime">
						<option  v-for="(time1,t1Index) in chooserData.startTimes" :key='t1Index' v-bind:value="time1">{{time1}}</option>
					</select>
					<span> - </span>
					<select name="" v-model="chooserData.endTime">
						<option  v-for="(time2,t2Index) in chooserData.endTimes" :key='t2Index' v-bind:value="time2">{{time2}}</option>
					</select>
					<i @click="confirmTime"></i>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		name: 'schedule',

		//props对象是引用当前组件可能需要传的参数
		props: {

		},

		//data函数，返回一个对象，双向绑定的数据
		data() {
			return {
				
				
				//日程起止时间，24小时制，可修改(需要为整点，格式为xx:00，结束时间必须大于开始时间)
				timeLineStart:'09:00',
				timeLineEnd:'18:00',
				//星期，可修改
				calendar:['Monday','Tuesday','Wednesday','Thursday','Friday'],
				//卡片背景色池,为空则默认#777
				bgcolor:['#577f92','#443453','#f6b067','#a2b9b2'],
				
				
				//显示用时间轴
				timeLineForDisplay: [],
				//表格用时间轴
				timeLineForTable:[],
				//用户选择时间轴
				timeLineForUser:[],
				//用户日程数据
				userSchedule:[
//					{
//						dayName:'Monday',
//						schedule:[
//							{startTimeStr:'09:00',startTimeInt:540,endTimeStr:'10:00',endTimeInt:600,title:'Work Time',content:'balabalabala',bgcolor:'',active:false},
//							{startTimeStr:'10:00',startTimeInt:600,endTimeStr:'10:30',endTimeInt:630,title:'Work Time',content:'balabalabala',bgcolor:'',active:false}
//						]
//					},
				],
				//卡片展开状态
				cardOpen:false,
				chooserData:{
					show:false,
					startTime:'',
					endTime:'',
					startTimes:[],
					endTimes:[],
					currentCard:[]
				},
				//日志数据
				log:'',
				showLog:false
			}
		},

		//watch监听页面数据变化,具体到某值-- xxx(){}
		watch: {
			'chooserData.startTime':function(){
				let s = this.chooserData.startTime;
				let e = this.chooserData.endTimes[this.chooserData.endTimes.length-1];
				this.chooserData.endTimes = [];
				for (let i = this.str2min(s)+15;i<=this.str2min(e);i+=15){
					this.chooserData.endTimes.push(this.min2str(i));
				}
			},
			'chooserData.endTime':function(){
				let s = this.chooserData.startTimes[0];
				let e = this.chooserData.endTime;
				this.chooserData.startTimes = [];
				for (let i = this.str2min(s);i<this.str2min(e);i+=15){
					this.chooserData.startTimes.push(this.min2str(i));
				}
			},
			

		},

		//computed计算属性 处理复杂的逻辑
		computed: {
			

		},

		//事件方法
		methods: {
			//字符串转分
			str2min(str){
				let min = 0;
				let h = str.split(':')[0];
				let m = str.split(':')[1];
				min = parseInt(h*60)+parseInt(m);
				return min
			},
			//分转字符串
			min2str(min){
				let str ='';
				let m = min%60;
				let h = (min-m)/60;
				if(m == '0'){
					m = '00';
				}
				if (h<10) {
					str = '0'+h.toString()+':'+m.toString();
				} else{
					str = h.toString()+':'+m.toString();
				}
				return str
			},
			//初始化时间轴数据
			initTimeLine(){
				let startTime = this.str2min(this.timeLineStart);
				let endTime = this.str2min(this.timeLineEnd);
				for (let i = startTime;i<=endTime;i+=60) {
					this.timeLineForDisplay.push(this.min2str(i))
				}
				for (let i = startTime;i<=endTime;i+=30) {
					this.timeLineForTable.push(this.min2str(i))
				}
				for (let i = startTime;i<=endTime;i+=15) {
					this.timeLineForUser.push(this.min2str(i))
				}
			},
			//初始化用户数据
			initUserSchedule(){
				for(let i in this.calendar){
					let obj = {
						dayName:this.calendar[i],
						schedule:[]
					}
					this.userSchedule.push(obj);
				}
			},
			//卡片操作
			openCard(c,i){
				this.cardOpen = true;
				c.active = true;
				this.log += this.getTimestamp() + ' 查看' + this.calendar[i]+' '+c.startTimeStr+'-'+c.endTimeStr+'日程<br/>';
			},
			closeCard(){
				this.chooserData.show = false;
				this.cardOpen = false;
				for (let i in this.userSchedule) {
					for (let j in this.userSchedule[i].schedule) {
						this.userSchedule[i].schedule[j].active = false;
					}
				}
			},
			deleteCard(i,j){
				this.chooserData.show = false;
				this.cardOpen = false;
				this.userSchedule[i].schedule.splice(j,1);
				this.log += this.getTimestamp() + ' 删除一条' + this.calendar[i]+'日程<br/>';
			},
			newCard(i,bIndex){
				let newData = {
					startTimeStr:'',
					startTimeInt:0,
					endTimeStr:'',
					endTimeInt:0,
					title:'Title',
					content:'write something...',
					bgcolor:'',
					active:false
				};
				let prevIndex = 0;
				newData.startTimeInt = this.str2min(this.timeLineStart)+bIndex*30;
				for (let j in this.userSchedule[i].schedule) {
					if (newData.startTimeInt>=this.userSchedule[i].schedule[j].startTimeInt && newData.startTimeInt<this.userSchedule[i].schedule[j].endTimeInt) {
						newData.startTimeInt+=15;
					}
				}
				for (let j in this.userSchedule[i].schedule) {
					if (j == this.userSchedule[i].schedule.length-1) {
						prevIndex = j;
					}else if (newData.startTimeInt>=this.userSchedule[i].schedule[j].endTimeInt && newData.startTimeInt<this.userSchedule[i].schedule[j*1+1].startTimeInt) {
						prevIndex = j;
					}
				}
//				if (this.userSchedule[i].schedule[prevIndex] && this.userSchedule[i].schedule[prevIndex*1+1]) {
//					console.log('case1')
//					while (newData.bgcolor == this.userSchedule[i].schedule[prevIndex].bgcolor || newData.bgcolor == this.userSchedule[i].schedule[prevIndex*1+1].bgcolor){
//						newData.bgcolor = this.bgcolor[Math.floor((Math.random()*this.bgcolor.length))];
//					}
//				}else if (this.userSchedule[i].schedule[prevIndex]) {
//					console.log('case2')
//					while (newData.bgcolor == this.userSchedule[i].schedule[prevIndex].bgcolor){
//						newData.bgcolor = this.bgcolor[Math.floor((Math.random()*this.bgcolor.length))];
//					}
//				}else if (this.userSchedule[i].schedule[prevIndex*1+1]) {
//					console.log('case3')
//					while (newData.bgcolor == this.userSchedule[i].schedule[prevIndex*1+1].bgcolor){
//						newData.bgcolor = this.bgcolor[Math.floor((Math.random()*this.bgcolor.length))];
//					}
//				}else{
//					console.log('case4')
//					newData.bgcolor = this.bgcolor[Math.floor((Math.random()*this.bgcolor.length))];
//				}
				let index = Math.floor((Math.random()*this.bgcolor.length))
				newData.bgcolor = this.bgcolor[index];
				newData.startTimeStr = this.min2str(newData.startTimeInt);
				newData.endTimeInt = newData.startTimeInt+15;
				newData.endTimeStr = this.min2str(newData.endTimeInt);
				newData.active = true;
				this.userSchedule[i].schedule.splice(prevIndex*1+1,0,newData);
				this.cardOpen = true;
				this.userSchedule[i].schedule = this.userSchedule[i].schedule.sort(this.sortTime('startTimeInt'))
				this.log += this.getTimestamp() + ' 新增一条' + this.calendar[i]+'日程<br/>';
			},
			getTimestamp(){
				let obj = new Date();
				let h = obj.getHours()
				let m = obj.getMinutes()
				let s = obj.getSeconds()
				if (h<10) {
					h = '0'+h;
				}
				if (m<10) {
					m = '0'+m;
				}
				if (s<10) {
					s = '0'+s;
				}
				return h+':'+m+':'+s
			},
			sortTime(property){
				return function(a,b){
					let t1 = a[property];
					let t2 = b[property];
					return t1 - t2
				}
				
			},
			//c:当前卡片的数据;i,j:数据在this.userSchedule中的位置
			changeTime(c,i,j){
				this.chooserData.currentCard = [i,j];
				this.chooserData.show = true;
				this.chooserData.startTime = c.startTimeStr;
				this.chooserData.endTime = c.endTimeStr;
				if (this.userSchedule[i].schedule.length == 1) {
					for (let i = this.str2min(this.timeLineStart);i<c.endTimeInt;i+=15) {
						this.chooserData.startTimes.push(this.min2str(i));
						
					}
					for (let i = c.startTimeInt+15;i<=this.str2min(this.timeLineEnd);i+=15){
						this.chooserData.endTimes.push(this.min2str(i));
					}
				}else if (j == 0) {
					let st = this.userSchedule[i].schedule[j*1+1].startTimeInt
					for (let i = this.str2min(this.timeLineStart);i < st;i+=15) {
						this.chooserData.startTimes.push(this.min2str(i));
					}
					for (let i = c.startTimeInt+15;i <= st;i+=15){
						this.chooserData.endTimes.push(this.min2str(i));
					}
				}else if(j == this.userSchedule[i].schedule.length-1){
					let st = this.userSchedule[i].schedule[j-1].endTimeInt;
					for (let i = st;i<c.endTimeInt;i+=15) {
						this.chooserData.startTimes.push(this.min2str(i));
						
					}
					for (let i = c.startTimeInt+15;i<=this.str2min(this.timeLineEnd);i+=15){
						this.chooserData.endTimes.push(this.min2str(i));
					}
				}else{
					let st =this.userSchedule[i].schedule[j-1].endTimeInt
					let et =this.userSchedule[i].schedule[j*1+1].startTimeInt
					for (let i = st;i<c.endTimeInt;i+=15) {
						this.chooserData.startTimes.push(this.min2str(i));
						
					}
					for (let i = c.startTimeInt+15;i<=et;i+=15){
						this.chooserData.endTimes.push(this.min2str(i));
					}
				}
			},
			confirmTime(){
				this.userSchedule[this.chooserData.currentCard[0]].schedule[this.chooserData.currentCard[1]].startTimeStr = this.chooserData.startTime;
				this.userSchedule[this.chooserData.currentCard[0]].schedule[this.chooserData.currentCard[1]].endTimeStr = this.chooserData.endTime;
				this.userSchedule[this.chooserData.currentCard[0]].schedule[this.chooserData.currentCard[1]].startTimeInt = this.str2min(this.chooserData.startTime);
				this.userSchedule[this.chooserData.currentCard[0]].schedule[this.chooserData.currentCard[1]].endTimeInt = this.str2min(this.chooserData.endTime);
				this.chooserData.show = false;
			}

		},

		//注册组件
		components: {
			// 'other-component':OtherComponent,
		},

		//生命周期函数
		beforeCreate() {
			console.log('beforeCreate');

		},
		created() {
			console.log('created');
			this.initTimeLine();
			this.initUserSchedule();

		},
		beforeMount() {
			console.log('beforeMount');

		},
		mounted() {
			console.log('mounted');

		},
		beforeUpdate() {
			console.log('beforeUpdate');

		},
		updated() {
			console.log('updated');

		},
		beforeDestroy() {
			console.log('beforeDestroy');

		},
		destroyed() {
			console.log('destroyed');

		}
	}
</script>

<style scoped>
	.cover{
		position: fixed;
		width: 100%;
		height: 100%;
		background: rgba(0,0,0,0.6);
		z-index: 1;
	}
	.scheduleTemplateTitle{
		text-align: center;
	}
	.schedule{
		width: 100%;
		padding: 20px;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
	}
	.timeLine{
		float: left;
		width: 60px;
		text-align: center;
		margin-top: 50px;
		font-size: 14px;
	}
	.timeLine>div{
		margin-bottom: 60px;
		line-height: 20px;
	}
	.calendar{
		float: left;
		width: calc(100% - 60px);
		margin-top: 20px;
		position: relative;
	}
	.dayList{
		width: 20%;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
		float: left;
	}
	.dayName{
		text-align: center;
		border: 1px solid #eee;
		line-height: 38px;
	}
	.dayBlocks{
		border-left: 1px solid #eee;
		border-right: 1px solid #eee;
	}
	.halfHourBlock{
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
		border-bottom: 1px solid #eee;
		height: 40px;
	}
	.halfHourBlock:hover{
		background: #f7f7f7;
	}
	.daySchedule{
		height: 0;
		top: 40px;
	}
	.card{
		position: absolute;
		/*transition: all 0.5s;*/
		cursor: pointer;
		background: #777;
	}
	.card:hover{
		box-shadow: 0px 2px 10px #999;
	}
	.cardActive{
		position: absolute;
		cursor: auto;
		z-index: 2;
		width: 700px!important;
		height: 400px!important;
		left: 50%!important;
		top: 50%!important;
		transform: translate(-50%,-50%);
	}
	.cardActive:hover{
		box-shadow: none;
	}
	.smCard{
		color: #fff;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}
	.smTime{
		font-size: 12px;
		line-height: 16px;
		margin: 5%;
		margin-bottom: 0;
	}
	.smTitle{
		font-size: 20px;
		margin: 5%;
		margin-top: 0;
	}
	.lgCard{
		width: 100%;
		height: 100%;
	}
	.lgCardLeft{
		float: left;
		width: 35%;
		padding: 10px;
		color: #fff;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
	}
	.lgTime{
		font-size: 14px;
		line-height: 30px;
		margin-bottom: 10px;
		cursor: pointer;
		padding-left: 5px;
	}
	.lgTitle{
		font-size: 20px;
		cursor: pointer;
	}
	.lgTitle>input{
		border: none;
		background: transparent;
		font-size: 20px;
		color: #fff;
		width: 100%;
		padding: 5px;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
	}
	.lgCardRight{
		float: left;
		width: 65%;
		height: 100%;
		background: #fff;
		padding: 30px;
		font-size: 16px;
		line-height: 24px;
		position: relative;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
	}
	.closeCard{
		position: absolute;
		top: 10px;
		right: 10px;
		width: 15px;
		height: 15px;
		background: url(../assets/close.png) 0 0 / 100% 100%  no-repeat;
		cursor: pointer;
		transition: all 0.5s;
	}
	.closeCard:hover{
		transform: rotate(180deg);
	}
	.deleteCard{
		position: absolute;
		top: 8px;
		right: 40px;
		width: 18px;
		height: 18px;
		background: url(../assets/delete.png) 0 0 / 100% 100%  no-repeat;
		cursor: pointer;
		transition: all 0.5s;
	}
	.deleteCard:hover{
		transform: rotate(-130deg);
	}
	.lgCardRight>textarea{
		width: 100%;
		height: 100%;
		line-height: 20px;
		border: 0;
		resize: none;
		padding: 10px;
		-webkit-box-sizing: border-box;
		-moz-box-sizing: border-box;
		box-sizing: border-box;
	}
	.timeChooser{
		position: absolute;
		z-index: 3;
		left: calc(50% - 342px);
		top: calc(50% - 184px);
		width: 230px;
	}
	.timeChooser>select{
		width: 90px;
	}
	.timeChooser>span{
		display: inline-block;
		width: 10px;
		text-align: center;
		color: #fff;
	}
	.timeChooser>i{
		display: inline-block;
		width: 20px;
		height: 20px;
		vertical-align: middle;
		margin: 0 10px;
		background: url(../assets/right.png) 0 0 / 100% 100% no-repeat;
		cursor: pointer;
		transition: all 0.5s;
	}
	.timeChooser>i:hover{
		transform: translateY(-3px);
	}
	.logBtn{
		position: absolute;
		right: 30px;
		top: 30px;
		cursor: pointer;
	}
	.logBtn:hover{
		color: #42B983;
	}
	.logBg{
		position: fixed;
		top: 0;
		left: 0;
		z-index: 4;
		width: 100%;
		height: 100%;
		background: rgba(0,0,0,0.6);
	}
	.logContent{
		width: 700px;
		height: 70%;
		overflow-y: auto;
		position: absolute;
		left: 50%;
		top: 50%;
		transform: translate(-50%,-50%);
		background: #fff;
	}
	.logContent>h5{
		text-align: center;
		font-size: 20px;
		line-height: 40px;
	}
	.logContent>p{
		padding: 20px;
		line-height: 30px;
	}
</style>