#Building a to do app with radiatr

[Live version](https://nmartinet.github.io/radiatr-todo/)//[Live Editable Version](https://nmartinet.github.io/radiatr-todo/todo.html)

radiatr is a small application to help quickly create data driven react application and documents. I know, it doesn't really fit the [wikipedia description](https://en.wikipedia.org/wiki/Rapid_application_development), it does tallow for rapid application development, and it doesn’t sound too bad as a name - alternative name suggestions are welcome (it used to be reactr but there are already project named reactor).

I feel that a good way to illustrate what it is and how it works is through example. So we're going to build a small to do app. 

The code for the example is available in a [repo here](https://github.com/nmartinet/radiatr-todo). Each step is in its own commit if you want to follow along.

A live editable version is available [here](https://nmartinet.github.io/radiatr-todo/todo.html).

This is not a react tutorial, there are plenty more thorough and better quality tutorials/articles/learning resources available elsewhere. A basic understanding of react is required. 

##Step 1 - Getting radiatr.
You're going to need radiatr to get started. All you need is the single html file. You can get it from the [repo](https://github.com/nmartinet/radiatr) or get a copy by opening the [live version here](https://nmartinet.github.io/radiatr/) and saving it locally (with the save button, not your browser's save file. If you named your file todo.html, you should have the same thing as the step 1 commit.

###Explanation
radiatr is downloaded. All the needed dependencies (react, reactdom, lodash, etc...) are linked from a cdn so you're ready to go.

radiatr gives you out of the box a split view with your document/app on the left and an editor on the right. The editor is based on a principal of data container. The various types of data are held in containers that manage their lifecycle - load, modify, save data - and any additional functions that are required for that data.

There are a few containers defined by default, and most of them have an associated settings container (with the same name as the container but with a -settings postfix)
The following containers are present by defaults:

  - config
   - Holds the configuration for the radiatr app. For the whole page. For example, the page title, links to stylesheets, external scripts, and the datasets.
   - Is controlled by a raw editor, which shows the object's json. 
  - data
    - By default, the data that will drive the application.
    - Is controlled through an object editor, with a ui for manipulating the object. All changes are live.
  - components
   - The react components that power radiatr. All the components are interpreted and are created in the window scope, so they are usable like any normal react class.
   - The application’s components will also be created and saved here.
   - Is controlled by a component editor. 
  - app-css
   - The additional css for the app. Although radiatr uses [Tachyons](http://tachyons.io/) for the css, some additional classes are needed.
   - Is controlled by a raw editor.
  - radiatr
    - The source code for radiatr
    - Holds the global helpers, code for the containers and for radiatr itself.
    - Is controlled by a raw editor

Right now you have everything you need to get started.

##Step 2 - Initial data
Before we start creating our react components, we're going to setup some dummy data to have something to feed into or components. We'll keep thing short and just create an array for our todos, and a simple `{name, completed}` object for each todo.

Go to the data tab.

Starting out, the data is an empty object. There is a field to add keys. Add one name `todos`. Where applicable, little pencil buttons appear to edit the item's properties. Change `todos` type to an array, add an entry to the array, etc... Until you have a data structure that look like: 

``` javascript
{
  todos: [
    {
      name: 'Finish Todo App', 
      completed: 0
    }
  ]
}
```

This is it for the data. Before this step is finished, we should also change the page title.
On the config tab, change the value of the `title` key to `radiatr Todo`. Changes on a raw editor aren't live - we wouldn't want to try to evaluate something while you're typing - so save the changes with the save button above the editor.

We can now save the file, just in case, like a word document. Use the save button on the top right. Either overwrite your current file and reload, or save to a different file and open that one. 
You’re good to go for step 3.

###Explanation
In radiatr, everything is in a container. The container is responsible for the lifecycle of the data. 

The container will hold and manage the data, and will expose functions to manipulate the data. When containers are create they return `data`, the initial data, and a `q` (for query) object which holds the functions to manipulate the data. Containers are also evented and will trigger callbacks on certain operations - for example, the reactr root hooks in a callback to setState after an update. 

The way that the container is represented in radiatr is controlled by the container settings. We can see examples of this in the `data-settings` and `components-settings`.
While `data-settings` if fairly simple and just defines the editor that will be used in radiatr (a `DataEdit` in this case) `components-settings` is more involved. 

`components-settings` has additional settings. It registers more functions that re needed for the data - in this case functions to interpret the string that we will get from the editor and create a react class from it. Functions are added to the container's functions and will be available in the container's `q`. 

When radiatr starts, it loads up the containers defined in its settings dataset. It will try to find a `${dataset name}-settings` object to use. It will also load the config and radiatr code as in memory containers. When the file is saved, radiatr rebuilds the html doc based on the currently in memory containers. So for example the radiatr code in the dataset can be modified, and will only be seen as a string, allowing the current radiatr to function normally. When the file is saved, the modified version is outputted in the template. But for the css, since it has a `postUpdate` hook that applies the CSS, the changes are immediate, without needing a save or reload.

##Step 3 - Creating the TodoRoot
Our app design will be pretty simple and straight forward. Here are the components we'll be creating

```
+---------------------------------+
| TodoRoot                        |
|   +---------------------------+ |
|   |NewTodo                    | |
|   +---------------------------+ |
|                                 |
|   +---------------------------+ |
|   |TodoList                   | |
|   |   +---------------------+ | |
|   |   |TodoEntry            | | |
|   |   +---------------------+ | |
|   +---------------------------+ |
+---------------------------------+
```

To get a hang of thing, we'll just create the root and make it output on the screen.

Go to the components tab.

Add a new component by using the text field above the list of components and name it `TodoRoot`
A new value will be added to the list, selected to see its contents. New components have the default value of: 

```javascript
{
  render: function(){
    return (
      <div>${Component Name}</div>
    )
  }
}
```

Which is recognizable as a simple object that could be passed to `React.createClass`. 
By default, components aren't evaluated. Click apply (top left over the editor) to make it available.

To output it to current document, we're going to have to modify the `Output` component. Change it to:

```javascript
{
  render: function(){
    let {data, q} = this.props;
    return (
      <TodoRoot data={data} q={q} />
    )
  }
}
```

The `TodoRoot` is now the output on the left hand side. And we pass to it the `data` that we created earlier and the `q` to manage it.

###Explanation
We created and modified react components. In radiatr, the components are held in a simple json object, where the actual code for the object - the `{ ... render: ...}` - is stored as a string. When the container is initiated, each component is evaluated and a react class is created. When we modify a component, the new string is evaluated and replaces the old class. Changes are immediate and don't need a reload.

The `{data, q, path}` (we will see `path` a bit later) is used throughout radiatr to normalize components. 

The `data` comes from the root's state and is hooked to update events, making sure that everything is synchronized. 

The `q` allows for a single point of action and exposes a least basic data management operations - read, update, delete etc. - and can be extended through the container's settings. 

And the `path` allow for a more generalized components.  In our case, our `todos` is in the root of the object which is simple. By using a path, as long as the path points to the same data structure - in our case an array of todos - the `TodoRoot` component would work. The path is an array of keys/indexes to point to the current data - with an empty array pointing to the root. In our case, the path to the first todo would be `['todos', 0]`.

##Step 3 - Creating the other components
We now have our app outputting properly. Following our little components schematic, let's create our other components. Create and apply the following components: `TodoList`, `TodoEntry`, `NewTodo`.

Logically, children have to exist before we can reference them in their parents. Now that they exist, let’s update our `TodoRoot`:

```javascript
{
  render: function(){
    let {data, q} = this.props;
    return (
      <div>
        <h1>Radiatr Todo app</h1>
        <NewTodo q={q} path={['todos']} />
        <TodoList todos={data.todos} q={q} path={['todos']} />
      </div>
    )
  }
}
```
###Explanation
This step is pretty basic react components creation. As we said above, we pass in a path to the `todos`. We don't pass the data to the `NewTodo` component as it doesn't need it.

##Step 5 - TodoList and TodoEntry
Our root is done we'll generate our list of todos. Update `TodoList` to: 

```javascript
{
  render: function(){
    let {todos, q, path} = this.props;
    
    let todoList = _.map(todos, function(todo, i){
      let _path = path.concat(i);
      return (
        <TodoEntry key={i}
                   todo={todo}
                   q={q}
                   path={_path} />
      );
    });
    
    return (
      <div>
        {todoList}
      </div>
    )
  }
}
```

Normal react list of components creation. We shouldn't forget to update the path to have the index for the todo.

And update our `TodoEntry` to: 
```javascript
{
  render: function(){
    let {todo, q} = this.props;
    return (
      <div>
        {todo.name}
      </div>
    )
  }
}
```

Our todos are now listed.

###Explanation
Nothing really to explain here, basic react stuff, apart from not forgetting about the path. For fun you can change the name of the todo from the data tab. 

##Step 6 - NewTodo
We're going to need a way to add new todos to the list. Since radiatr components are available, we're not going to reinvent the wheel. We'll be using the `TextInputButton` component. It is the component that allows us to add components on the component editor.

Update the `NewTodo` component to:

```javascript
{
  newTodo: function(name){
    if(name == "") return;
    let {q, path} = this.props;
    
    let newTodoValue = {
        name: name,
        completed: 0,
    }
    
    q.addArray(path, newTodoValue);
    
    return "";  
  },
  render: function(){
    return (
      <div>
        <TextInputButton
            onClick={this.newTodo}
            placeholder="new task"
            buttonValue="add" />
      </div>
    )
  }
}
```

###Explanation
First our render function. We use the `TextInputButton` component, which is a text input with a button. It isn't different than using any other react component.

Then our `newTodo` function. We define this function to be passed to the onClick of the `TextInputButton`. The function take in the name of the new todo - in this case, the value of the input that is passed. This is where we need the `q` and `path`. With these, we can append a new todo to the array. By using a function through the  `q`, the `NewTodo` component doesn't need have a specific state, or need to synch anything. When the `q.addArray` is called, an update event is triggered. The root of radiatr hooked in a setState callback so that it know to update its children.

Returning an empty string is specific to the `TextInputButton`. It uses this value to update the value of the input, and in this case to clear it.

##Step 7 - Changing todo status
We can add todos, but we need a way to mark them as complete. Let’s update our `TodoEntry`:

```javascript
{
  onClick: function(){
    let {todo, q, path} = this.props;
    let newVal = todo.completed == '0' ? '1' : '0';
    q.updateValue(path.concat('completed'), newVal);
  },
  render: function(){
    let {todo, q} = this.props;
    return (
      <div>
        <a href="#" onClick={this.onClick}>
            {todo.name}
        </a>
      </div>
    )
  }
}
```

The data changes, but since we aren't displaying it, it doesn't really make a difference for the user. We're going to need to add some styling.

###Explanation
Very similar to adding a todo. Instead, we use `q.updateValue` to change the value associated with the key. Also, by using the path, we don't need to change the whole object, we can target one of its properties.

Note - It's a bit weird to have the numbers held as strings, but it was to simplify the original development, everything that isn't an object or an array is treated as a string. It should be changed soon.

##Step 8 - Completed styling
We need to have a visual representation of the todo's state. `Tachyons` is used for most styling throughout radiatr, but, some customization is still needed. For that there is the `app-css` container. This container is setup to insert the given CSS in the document's head. The change is applied on save (so no reload required). Try adding `body{ background-color: red }` and clicking save to see


Tachyons doesn't have a class for strikethrough text. Somewhere (at the end to be clean) in the `app-css` add:

```css
.strikethrough{
    text-decoration: line-through;
}
```

We have a class, now we need a way to have conditionally applied class names. To do that, there is a function `cn` (for class name) that reduces the given list of class names into a single string. We can use the ternary operator (shorthand if then else [a ? b : c]) to make it pretty - Update `TodoEntry` to:

```javascript
{
  onClick: function(){
    let {todo, q, path} = this.props;
    let newVal = todo.completed == '0' ? 1 : 0;
    q.updateValue(path.concat('completed'), newVal);
  },
  render: function(){
    let {todo, q} = this.props;
    
    let cName = cn(
      'link',
      'black',
      (todo.completed == '1' ? 'strikethrough gray' : '')
    )
    
    return (
      <div>
        <a className={cName} href="#" onClick={this.onClick}>
            {todo.name}
        </a>
      </div>
    )
  }
}
```

###Explanation
Global helpers are available throughout radiatr. Since all the code ends up being executed for the radiatr script tag/context, any reference put there becomes available throughout. Since the components are actually stored as string, the reference to the function do not need to exist where the function is 'stored' on the page but rather where it is executed. So global helpers can be easily added in the radiatr container.

The cn function is a simplified version of the [classNames utility](https://github.com/JedWatson/classnames). It flattens the arguments, returns the keys whose values are true, and reduces it to a single string. It worked reasonably well up to this point - for radiatr's needs.

###Step 9 - Sorting Todos
A bit out of scope, since it's not radiatr specific, but let's sort the todos between completed and not. Update `TodoList` to:

```javascript
{
  render: function(){
    let {todos, q, path} = this.props;
    
    let todoList = _(todos)
      .map(function(todo, i){
          let _path = path.concat(i)
          return ({
            ui: (<TodoEntry key={i} 
                            todo={todo} 
                            q={q} 
                            path={_path}/>),   
            completed: todo.completed
          })
      })
      .orderBy(['completed'])
      .map((td) => td.ui)
      .value()
      
    return (
      <div>
        {todoList}
      </div>
    )
  }
}
```

###Step 10 - Save button and styling
Our todo app works, but what if we don't want the editor anymore? We'll start by creating a new component - `SaveFileButton` - so that we can still save the file without the whole editor showing. 
Create the component and update it to: 

```javascript
{
  onSave: function(){
    saveToFile()
  },
  render: function(){
    return (
      <div>
      <button className="btn" onClick={this.onSave} >
          Save
      </button>
      </div>    
    )
  }   
}
```

Then add the button to our ui - we'll take this opportunity to update our style a little bit - update the `TodoRoot` to:

```javascript
{
  render: function(){
    let {data, q} = this.props;
    return (
      <div className="mw9 center pa3 ph5-ns">
        <h1>Radiatr Todo app</h1>
        <NewTodo q={q} path={['todos']} />
        <TodoList todos={data.todos} q={q} path={['todos']} />
        <SaveFileButton />
      </div>
    )
  }
}
```

We can now remove our editor. Edit the `RadiatrRoot` so that the `return` of the render function is:

```javascript
return (
  <div className="cf">
    <div className="fl w-100">
      <Output data={self.state.data} q={self.state.funcs.data} />
    </div>
  </div>
)
```

Save the document and reload. You now have a self-contained todo app.

###Explanation
Like `cn`, `saveToFile()` is a global helper defined in the radiatr code. It regenerates a file from the current containers. By removing the radiatr section of the app you end up with an output that only contains your app.

Note - While you end up with only your app outputted, there is still a lot of unnecessary stuff in your file. You could manually remove the unused components and external libraries. Something should come to automate this/export only what's needed in future version.
