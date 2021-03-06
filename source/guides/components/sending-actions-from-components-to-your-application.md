英文原文：[http://emberjs.com/guides/components/sending-actions-from-components-to-your-application/](http://emberjs.com/guides/components/sending-actions-from-components-to-your-application/)

当组件被用于一个模板中时，其可以发送操作给模板的控制器和路由。这样使得组件在一些重要的事件发生时（如一个用户点击了组件中的一个特定的元素，通知应用。

就像`{{action}}`Handlebars助手一样，组件发出的操作首先到达模板的控制器。如果控制器没有实现一个操作的处理方法，那么操作会被抛到模板的路由，然后再路由的层级中一路往上冒泡。想知道关于冒泡机制的详细细节，可以查看[操作冒泡](/guides/templates/actions/#toc_action-bubbling)。

组件被设计为可以在应用中不同的部分都可以重用。为了达到可以重用的目的，当在模板中使用组建时，指定组件发出的操作非常重要。

或者说，假设在编写一个按钮组件，且不希望发送一个`click`操作，因为这可能会有点模棱两可，会导致与页面上的其他组件发生冲突。作为替代的方案，可以在允许使用组件的人，在使用时指定当点击按钮时，组件将会发送的操作。

幸运的是，组件拥有`sendAction()`方法，它允许在模板中使用组件时设定组件要发送的操作。

### 发送主操作

许多组件只发送一种类型的操作。例如，按钮组件可能只在被点击时发送一个操作；这就是_主操作_。

要设置一个组件的主操作，只需要在Handlebars模板中设置组件的`action`属性即可：

```handlebars
{{my-button action="showUser"}}
```

这里告知`my-button`组件，当其主操作被触发时，将发送`showUser`操作。

因此，该如何触发发送一个组件的主操作呢？在相关的事件发送之后，通过调用不带参数的`sendAction()`方法可以实现：

```js
App.MyButtonComponent = Ember.Component.extend({
  click: function() {
    this.sendAction();
  }
}
```

上面的例子中，在`my-button`组件被点击时，会发送`showUser`操作。

### 发送一个带参数的操作

有时候可能需要提供附件的上下文给路由或控制器来处理一个操作。例如，一个按钮组件可能不止需要告诉控制器_一个_条目被删除了，还需要告诉控制器是_哪一个_条目被删除。

为了使主操作发送参数，需要在调用`sendAction()`方法时指定`"action"`为第一个参数，后续的参数将作为要发送的参数：

```js
this.sendAction('action', param1, param2);
```

例如，假设正在编写一个允许用户删除待办事项的列表：

```js
App.IndexRoute = Ember.Route.extend({
  model: function() {
    return {
      todos: [{
        title: "Learn Ember.js"
      }, {
        title: "Walk the dog"
      }]
    };
  },
  
  actions: {
    deleteTodo: function(todo) {
      var todos = this.modelFor('index').todos;
      todos.removeObject(todo);
    }
  }
});
```

```handlebars
{{! index.handlebars }}

{{#each todo in todos}}
  <p>{{todo.title}} <button {{action "deleteTodo" todo}}>Delete</button></p>
{{/each}}
```

这时候可能需要更新应用，使得在删除一条待办事项时，用户必须确认这是否是他所希望进行的操作。这里就需要实现一个在完成操作前重复检查用户行为的组件。

在组件中，当触发主操作时，将传入一个组件用户可以指定的附加参数：

```js
App.ConfirmButtonComponent = Ember.Component.extend({
  actions: {
    showConfirmation: function() {
      this.toggleProperty('isShowingConfirmation'); 
    },
    
    confirm: function() {
      this.toggleProperty('isShowingConfirmation');
      this.sendAction('action', this.get('param'));
    }
  }
});
```

```handlebars
{{#if isShowingConfirmation}}
  <button {{action "confirm"}}>Click again to confirm</button>
{{else}}
  <button {{action "showConfirmation"}}>{{title}}</button>
{{/if}}
```

现在可以修改原来的模板，用自定义的新组件来替换`{{action}}`助手：

```handlebars
    {{#each todo in todos}}
      <p>{{todo.title}} {{confirm-button title="Delete" action="deleteTodo" param=todo}}</p>
    {{/each}}
```

注意这里通过组件的`action`属性来设置需要发送的特定操作，并且通过组件的`param`属性来设置需要发送的特定的参数。

<a class="jsbin-embed" href="http://jsbin.com/atIgUSi/1/embed?live">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>

### 发送多个操作

根据组件的复杂程度，可能需要根据组件能产生的不同事件，来指定不同的操作。

例如，假设在编写一个用户可以提交或者撤销的表单组件。根据用户点击的按钮，希望可以发送不同的操作给控制器或者路由。

通过传入事件的名称作为`sendAction()`的第一个参数，可以指定_哪一个_操作将被发送。例如，当使用表单组件时，可以指定两个操作：

```handlebars
{{user-form submit="createUser" cancel="cancelUserCreation"}}
```

在这里，可以通过调用`this.sendAction('submit')`来发送`createUser`操作，或者通过调用`this.sendAction('cancel')`来发送`cancelUserCreation`操作。

<a class="jsbin-embed" href="http://jsbin.com/OpebEFO/1/embed?html,js,output">JS Bin</a><script src="http://static.jsbin.com/js/embed.js"></script>

### 没有指定的操作

如果有人在使用组件时没有为一个特定的事件指定一个操作，调用`sendAction()`将不起任何作用。

例如，如果定义了一个组件，该组件在被点击时触发主操作：

```js
App.MyButtonComponent = Ember.Component.extend({
  click: function() {
    this.sendAction();
  }
});
```

如果使用组件时，没有指定主操作，那么用户点击时，不会有任何效果：

```handlebars
{{my-button}}
```

### 关于组件操作的思考

通常情况下，可以认为组件的操作是将_原生事件_（例如鼠标点击、或者`<audio>`元素的`pause`事件）转换为应用中具有特定语义的操作。

这样路由和控制器就可以使用`deleteTodo`或者`songDidPause`这样的命名来取代如`click`或者`pause`这样会给其他开发者在阅读上下文时带来歧义的不明确的名称。

另外一种想法就是认为组件操作是组件的_公共API_。其他要使用组件的开发者主要需要考虑的就是什么事件组件能触发应用中的操作。通常情况下，保持这些事件尽可能的通用，就保证了组件更加易用和容易重用。
