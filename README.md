# Marionette-Todo-
$(function(){
	//创建Application的目的就是把所有东西全部绑定在上面
	var TodoApp = new Backbone.Marionette.Application();
	
	//创建todo的model
	TodoApp.Todo = Backbone.Model.extend({
		defaults : {
			 title : '',
			 done: false
		},
		toggle : function(){
			this.save({done : !this.get('done')})
		}
	})
	
	//创建todo的collection
	TodoApp.TodoList = Backbone.Collection.extend({
		model : TodoApp.Todo,
		localStorage : new Backbone.LocalStorage('todos-backbone-marionette'),
		done : function(){
		 return	this.filter(function(todo){return todo.get('done')})
		},
		remaining: function() {
		  return this.without.apply(this, this.done());
		}
	})
	var TodoCollection = new TodoApp.TodoList()
	//创建头部的模板视图
	TodoApp.TodoItemView = Backbone.Marionette.ItemView.extend({
		el : '#template-header',
		template : false,
		//这个对象指定了需要对应的名称与选择器
		ui : {
			input : '#btn',
			inpuText : '#new-todo'
		},
		events : {
			'click @ui.input' : 'createOne'
		},
		createOne : function(){
			// $.trim() 去掉字符串首尾空格
			var val = this.$el.find('#new-todo').val().trim();
			if(val){
				TodoCollection.create({title : val});
				this.$el.find('#new-todo').val('')
			}
		},
	});
	//item 视图
	TodoApp.TodoView = Marionette.ItemView.extend({
		tagName : 'li',
		template : '#item-template',
		events : {
			"dblclick .view" : 'edit',
			'click .destroy' : 'clear',
			'blur .edit'     : 'clos',
			'click .toggle'  : 'toggleDone'
		},
		//用于监听view的model（相当于backbone的listenTo，但是比listenTo简化多）
		modelEvents: {
			'change': 'docRender'
		},
		//当数据发生改变时重新渲染
		docRender: function(){
			this.render();
		},
		edit : function(){
			this.$el.addClass('editing');
			this.$('.edit').focus()
		},
		clear : function(){
			this.model.destroy()
		},
		clos : function(){
			var value = this.$el.find('.edit').val();
			if(!value){
				return false
			}else{
				this.model.save({title:value})
				this.$el.removeClass('editing');
			}
		},
		toggleDone : function(){
			this.model.toggle()
		}
	});
	//footer视图
	TodoApp.TodofooterView = Marionette.ItemView.extend({
		el:'#todoapp',
		statsTemplate : _.template($('#stats-template').html()),
		collectionEvents : {
			all : 'render'
		},
		render : function(){
			this.main = this.$el.find('#main');
			this.footer = this.$el.find('#footer')
			var done = TodoCollection.done().length;
		    var remaining = TodoCollection.remaining().length;
			if(TodoCollection.length){
				this.main.show();
				this.footer.show();
				this.footer.html(this.statsTemplate({done: done, remaining: remaining}));
			}else{
				this.main.hide();
				this.footer.hide();
			}
			/*alert(!remaining)
			this.$el.find("#toggle-all").checked = !remaining
			alert(this.$("#toggle-all").checked)*/
		},
		events : {
			'click #clear-completed' : 'clearCompleted'
		},
		clearCompleted : function(){
			_.invoke(TodoCollection.done(), 'destroy');
		}
	});
	
	
	TodoApp.TodoCollectionView = Marionette.CompositeView.extend({
		
		template : '#item-header',
		childView : TodoApp.TodoView,
		childViewContainer : '#todo-list',
		
		events : {
			'click #toggle-all': 'onToggleAllClick'
		},
		onToggleAllClick : function(){
		  var done = this.$el.find("#toggle-all")[0].checked
		  TodoCollection.each(function(todo){ todo.save({done:done})})
		},
		
	});
	
	TodoApp.addRegions({
		footerRegion : '#footer',
		mainRegion : '#main'
	});

	//在初始化事件执行完成之后
	TodoApp.on('start',function(){
		
		var addItem = new TodoApp.TodoItemView({});
		
		var todoCollectionView = new TodoApp.TodoCollectionView({
			collection: TodoCollection
		});
		
		var footItem = new TodoApp.TodofooterView({
			collection: TodoCollection
		});
		
		TodoCollection.fetch();
		//添加一个视图到region里。这个视图也会立即自动渲染
		TodoApp.mainRegion.show(todoCollectionView)
		/*TodoApp.footerRegion.show(footItem)*/
	})
	//start 去开启一个TodoApp
	TodoApp.start()
})
