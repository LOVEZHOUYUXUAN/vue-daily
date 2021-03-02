<template>
	<div class = "daily">
		<div class = "daily-menu">
		   <div class = "daily-menu-item"
		         @click="handleToRecommend"
		         :class = "{on :type === 'recommend'}">每日推荐</div>
		   <div class = "daily-menu-item"
		        :class = "{on: type === 'daily' }"
				@click="showSection = !showSection"
			     >热门栏目</div>
		   <ul v-show = "showSection">
			   <li v-for = "item in sections">
				   <a :class = "{on:item.id === sectionId&&type === 'section'}"
                       @click="handleToSection(item.id)">{{item.name}}</a>
			   </li>
		   </ul>
		</div>
		<div class = "daily-list" ref = "list">
			<template v-if="type === 'recommend'">
				<div v-for = "list in recommendList">
					<div class = "daily-date">{{formatDay(list.date)}}</div>
					<Item
					    v-for = "item in list.stories"
						:data = "item"
						:key = "item.id"
						@click.native = "handleClick(item.id)"
						></Item
					>
				</div>
			</template>
			<template v-if="type === 'section'">
				<Item  
				    v-for = "item in list "
					:data = "item"
					:key = "item.id"
					@click.native = "handleClick(item.id)"
					></Item>
				     
			</template>
		</div>
		<daily-article :id = "articleId"></daily-article>
    </div>		
</template>

<script>
	import $ from './libs/util';
	import Item from './components/item.vue';
	import dailyArticle from './components/daily-article.vue';
	export default {
		components:{Item,dailyArticle},
		data () {
			return {
				sections:[],
				showSection:false,
				type:'recommend',
				list:[],
				sectionId:0,
				dailyTime:$.getTodayTime(),
				recommendList:[],
				isLoading:false,
				articleId:0,
				sectionTime:0//本次请求的时间戳
			}
		},
		methods:{
			//转换为带汉字的月日
			formatDay(date){
				let month = date.substr(4, 2);
				let day = date.substr(6, 2);
				if (month.substr(0, 1) ==='0') month = month.substr(1, 1);
				if (day.substr(0, 1) === '0') day = day.substr(1, 1);
				return `${month} 月 ${day} 日`;
			},
			getSections(){
				//axios 发起get请求
				$.ajaxSection.get('sections').then(res => {
					this.sections = res.data;
				})
			},
			handleToSection(id) {
				//改变菜单分类
				this.type = 'section';
				//设置当前点击子类的主题日报id
				this.sectionId = id;
				//清空中间栏的数据
				this.list = [];
				$.ajaxSection.get('section/'+id).then(res => {
					this.sectionTime = res.timestamp;
					this.list = res.stories;
				})
			},
			handleToRecommend () {
				this.type = 'recommend';
				this.recommendList = [];
				this.dailyTime = $.getTodayTime();
				this.getRecommendList();
			},
			getRecommendList () {
				this.isLoading = true;
				const prevDay = $.prevDay(this.dailyTime+86400000);
				$.ajax.get('news/before/'+prevDay).then(res => {
					this.recommendList.push(res);
					this.isLoading = false;
				})
			},
			handleClick (id) {
				this.articleId = id;
			},
			getSectionList() {
				this.isLoading = true;
				$.ajaxSection.get('section/'+this.sectionId+`/before/${this.sectionTime}`).then(res =>{
					this.sectionTime = res.timestamp;
					for (const value of res.stories) {
						this.list.push(value);
					}
					this.isLoading = false;
				})
			}
			
			
			
			
		},
		mounted () {
			//初始化时调用
			this.getSections();
			this.getRecommendList();
			const $list = this.$refs.list;
			//监听中栏的滚动事件
			$list.addEventListener('scroll',() => {
				//在”主题日报“或正在加载推荐列表时停止操作
				if ( this.isLoading) return ;
				//已经滚动的距离加页面的高度等于整个内容区域高度时，视为接触底部
				if 
				(
				$list.scrollTop
				+ document.body.clientHeight
				>= $list.scrollHeight
				)
				{
					if (this.type === 'recommend'){
					//时间相对减少一天
					this.dailyTime -= 86400000;
					this.getRecommendList();
					}else {
						this.getSectionList();
					}
				}
			});
		}
		
	}
</script>


