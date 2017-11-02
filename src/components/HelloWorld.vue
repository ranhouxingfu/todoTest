<template>
	<!--<div class="hello">
    <h1>{{ msg }}</h1>
    <h2>Essential Links</h2>
    <ul>
      <li><a href="https://vuejs.org" target="_blank">Core Docs</a></li>
      <li><a href="https://forum.vuejs.org" target="_blank">Forum</a></li>
      <li><a href="https://chat.vuejs.org" target="_blank">Community Chat</a></li>
      <li><a href="https://twitter.com/vuejs" target="_blank">Twitter</a></li>
      <br>
      <li><a href="http://vuejs-templates.github.io/webpack/" target="_blank">Docs for This Template</a></li>
    </ul>
    <h2>Ecosystem</h2>
    <ul>
      <li><a href="http://router.vuejs.org/" target="_blank">vue-router</a></li>
      <li><a href="http://vuex.vuejs.org/" target="_blank">vuex</a></li>
      <li><a href="http://vue-loader.vuejs.org/" target="_blank">vue-loader</a></li>
      <li><a href="https://github.com/vuejs/awesome-vue" target="_blank">awesome-vue</a></li>
    </ul>
  </div>-->
	<div class="contain-box">
		<!--title-->
		<h3>日程安排列表</h3>
		<div class="add-rule">
			<div>
				<p>添加任务:</p>
				<input placeholder="添加任务" v-on:keyup.13='addRule' v-model="inputValue" />
			</div>
			<div class="unfinish-box">
				<span class="unfinishLen">{{unfinishLength}}个任务未完成</span>
				<ul class="rule-kids">
					<li>
						<a href="#all">所有任务</a>
					</li>
					<li>
						<a href="#unfinished">未完成任务</a>
					</li>
					<li>
						<a href="#finished">已完成任务</a>
					</li>
				</ul>
			</div>
			<div>
				<p>任务列表:</p>
				<div class="no-rule" v-show='!ruleList.length'>还没有添加任何任务</div>
				<ul class="rule-list">
					<li v-for='item in fitlterList' v-on:dblclick="editRule(item)" :class="{'computed':item.isChecked,edit:item==editTodo}">
						<div><span><input type="checkbox" v-model="item.isChecked"/></span>
							<span>{{item.title}}</span><span v-on:click="removeRule(item)">删除</span>
						</div>
						<input v-foucs='editTodo==item' v-model='item.title' @blur="editComplete(item)" @keyup.13="editComplete(item)" @keyup.esc="cancelEdit(item)" class="edit-input" />
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>
	//localstorage缓存
	var store = {
		save(key, value) {
			localStorage.setItem(key, JSON.stringify(value)); //缓存为json字符串
		},
		fetch(key) {
			return JSON.parse(localStorage.getItem(key)) || [];
		}
	}
	var list = store.fetch('todoList');
	/*删选数据*/
	var filter = {
		all: function(list) {
			return list;
		},
		unfinished: function(list) {
			return list.filter(function(item) {
				return !item.isChecked
			})
		},
		finished: function(list) {
			return list.filter(function(item) {
				return item.isChecked
			})
		}
	}
	export default {
		data() {
			return {
				ruleList: list, //任务列表
				inputValue: '',
				editTodo: '', //正在编辑的数据
				beforeTodo: '', //记录正在编辑的原始title
				visibility: 'all'
			}
		},
		watch: { //监听数据的变化
			/*
			 * 一下这种方式为浅监控，不能遍历其中所有的属性
			 */
			//			ruleList: function() { //监听ruleList这个属性的变化，一旦变化就会执行这个函数
			//				store.save('todoList', this.ruleList)
			//			}
			/*
			 * 一下方法为深层监控，能够遍历其中所有的属性，保存状态
			 */

			ruleList: {
				handler: function() {
					store.save('todoList', this.ruleList);
				},
				deep: true
			}
		},
		computed: {
			unfinishLength: function() { //过滤未勾选的列表
				return this.ruleList.filter(function(item) {
					return !item.isChecked
				}).length
			},
			fitlterList: function() { //过滤已完成、未完成、全部数据
				//存在filter[this.visibility]就返回过滤后的数据，否则返回所有数据
				return filter[this.visibility] ? filter[this.visibility](list) : list;
			}
		},
		mounted() {
			this.watchHashChange;
			window.addEventListener('hashchange', this.watchHashChange)
		},
		methods: {
			addRule(ev) { //添加
				this.ruleList.push({
					title: ev.target.value,
					isChecked: false
				});
				this.inputValue = ''
			},
			removeRule(rule) { //删除
				var index = this.ruleList.indexOf(rule); //获取当前这条数据的下标index
				this.ruleList.splice(index, 1)
			},
			editRule(rule) { //双击出现input编辑
				//编辑任务的时候，记录一下title，方便取消编辑时候重新给之前的title
				this.beforeTodo = rule.title;
				this.editTodo = rule;
			},
			editComplete(rule) { //编辑成功完成
				this.editTodo = ''
			},
			cancelEdit(rule) { //取消编辑
				rule.title = this.beforeTodo;
				this.beforeTodo = '';
				this.editTodo = '';
			},
			watchHashChange() {
				var hash = window.location.hash.slice(2);
				this.visibility = hash;
				console.log(this.visibility)

			}
		},
		directives: { //自定义指令，编辑时，自动获取焦点
			"foucs": {
				update(el, binding) { //钩子函数,el为当前的元素
					if(binding.value) {
						el.focus();
					}
				}
			}
		}
	}
</script>

<style scoped>
	.contain-box {
		width: 100%;
		margin: 0 auto;
		/*border: 1px solid red;*/
	}
	
	.contain-box>h3 {
		padding: 15px 0;
		background: rgb(65, 92, 119);
		color: white;
		margin: 0;
		text-align: center;
	}
	
	.add-rule {
		width: 50%;
		margin: 0 auto;
		text-align: left;
	}
	
	.add-rule>div {
		width: 100%;
	}
	
	.add-rule>div>input {
		padding: 5px;
		border-radius: 5px;
		outline: none;
		border: 1px solid gainsboro;
		width: 100%;
	}
	
	.unfinishLen {
		color: red;
		text-align: left;
	}
	
	.unfinish-box {
		margin: 15px 0;
	}
	
	.rule-kids {
		float: right;
	}
	
	.rule-kids>li {
		display: inline-block;
	}
	
	.rule-kids>li>a {
		color: white;
		text-decoration: none;
	}
	
	.rule-kids>li a:hover {
		color: red;
	}
	
	.rule-list {
		width: 100%;
		list-style: none;
		background: white;
	}
	
	.rule-list>li {
		height: 35px;
		line-height: 35px;
		padding: 5px 15px 3px 15px;
		border-bottom: 1px solid gainsboro;
	}
	
	.rule-list>li:hover {
		background: #DCDCDC;
	}
	
	.rule-list>li span:last-of-type {
		color: red;
		float: right;
		cursor: pointer;
	}
	
	.no-rule {
		padding: 5px;
		background: white;
	}
	
	.computed {
		text-decoration: line-through;
	}
	
	.edit-input {
		padding: 8px;
		width: 80%;
		border: 1px solid gainsboro;
	}
	
	.edit-input {
		display: none;
	}
	
	.edit>div {
		display: none;
	}
	
	.edit .edit-input {
		display: block;
	}
</style>