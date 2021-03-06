<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [React Hooks - are great!](#react-hooks---are-great)
- [Context in React is about passing properties between a component and distant components (in the component tree)](#context-in-react-is-about-passing-properties-between-a-component-and-distant-components-in-the-component-tree)
- [Transform the previous class based components into function=hooks based components - clean implementation](#transform-the-previous-class-based-components-into-functionhooks-based-components---clean-implementation)
- [State Management w/ useReducer and useContext](#state-management-w-usereducer-and-usecontext)
  - [Add in Todo Context](#add-in-todo-context)
  - [Consuming the Todo Context](#consuming-the-todo-context)
- [Introduce a new pattern - instead of multiple methods write a single function - a reducer - fix reloading issue](#introduce-a-new-pattern---instead-of-multiple-methods-write-a-single-function---a-reducer---fix-reloading-issue)
- [Memo - Higher Order Component built into React - To Speed Up the App](#memo---higher-order-component-built-into-react---to-speed-up-the-app)
- [Use Custom Hook: Reducer + LocalStorage](#use-custom-hook-reducer--localstorage)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


#----------------------------------------- The modern React Bootcamp notes -----------------------------------------

# React Hooks - are great!

```JavaScript
import React, { useState, useEffect } from "react";
import axios from "axios";

function SWMovies() {
  // number is the property from state that changes with the callback of setNumber ??
  const [number, setNumber] = useState(1); // initial state is 1
  const [movie, setMovie] = useState("");
  
  // useEffect is = to ngOnChanges from Angular - executes wach time detects a change and first time before ngOnInit
 
  useEffect(() => {
    async function getData() {
      const response = await axios.get(`https://swapi.co/api/films/${number}/`);
      setMovie(response.data);
    }
    getData();
  // thus pay attention when fetching data inside useEffect !!! Always pass the second parameter - the condition to
  // execute the async function only when one of the elements from second parameter change (here : 'number')
  }, [number]);

  return (
    <div>
      <h1>Pick A Movie</h1>
      <h4>{movie.title}</h4>
      <p>{movie.opening_crawl}</p>
      <select value={ number } onChange={ e => setNumber(e.target.value) }>
        <option value='1'>1</option>
        <option value='2'>2</option>
        <option value='3'>3</option>
      </select>
    </div>
  );
}
export default SWMovies;
```

# Context in React is about passing properties between a component and distant components (in the component tree)

```JavaScript
// the component where we consume the data (PageContent component)
import { ThemeContext } from "./contexts/ThemeContext";

export default class PageContent extends Component {
  // we set contextType to theme context and thus we are able to read the state directly from context
  // if you whant to consume more then one context, this is not going to work
  
  static contextType = ThemeContext; // this tells the component to look for the ThemeContext somewhere above it
  render() {
    const { isDarkMode, toggleTheme } = this.context;
    // ...
  }

  // In order to update the state we need to pass in the ThmeContext's Provider as value, the new state including the updated property corresponding to the event

  // ..
        <ThemeContext.Provider
        value={{ ...this.state, toggleTheme: this.toggleTheme }}
      ></ThemeContext.Provider>
  // ..

  // ...
  // for those components that consume more than one contexts, we use consumer components
  // create a high order component, wich takes a different component and some props,as argument,
  // and returns that same component, with all it's original props, but also it injects in a property
  // e.g laguageContext, coming from the consumer taht takes the value from consumer
  export const withLanguageContext = Component => props => (
    // Component is just a generatic name for whatever component we pass
    // for those components that consume more than one contexts, we use consumer components
    // these components take as child function components:
    <LanguageContext.Consumer>
      {value => <Component languageContext={value} {...props} />}
    </LanguageContext.Consumer>
  );

  // in the component where we consume the data (Navbar) we use:
  export default withLanguageContext(withStyles(styles)(Navbar));
  // this takes the Navbar that was wrapped with the styles - this new version of the navbar si once again wrapped, this time with withLanguageContext
  // so, it returns a new version of the Navbar, now containing languageContext, as a prop
  ```

# Transform the previous class based components into function=hooks based components - clean implementation

```JavaScript
import React, { createContext, useState } from "react";

export const LanguageContext = createContext();

// we DO NOT HAVE 'this' in a function based component
// we pass props as argument
// we use 'useState' to get the state
export function LanguageProvider(props) {
  const [language, setLanguage] = useState("spanish");
  const changeLanguage = e => setLanguage(e.target.value);
  return (
    <LanguageContext.Provider value={{ language, changeLanguage }}>
      {props.children}
    </LanguageContext.Provider>
  );
}
```

Now there is no need to use that Higher Order Component, we do not need any more to add a second wrapper to the Navbar component, that became a hook based component.


# State Management w/ useReducer and useContext

## Add in Todo Context

```JavaScript
import React, {createContext} from "react";
import useTodoState from "../hooks/useTodoState";

const defaultTodos = [
    {id: 1, task: "Make new notes for Security study", completed: false},
    {id: 2, task: "Release lady bugs into garden", completed: true} 
];

//  create the context itself

export const TodosContext = createContext();
// we already have the useTodoState hook that gives us all the pieces we need
// what is missing is a context that will call this hook and then use these pieces and store them as a value for that context
console.log(TodosContext)
export function TodosProvider(props) {
    // const { todos, addTodo, removeTodo, toggleTodo, editTodo } = useTodoState(defaultTodos);
    // because  we use all the pieces from the hook we can shorten out the object (to todosStuff) that we pass as value in the returned component 
    const { todosStuff } = useTodoState(defaultTodos);

    return (
        <TodosContext.Provider value={{ todosStuff } }>
            {/* the component TodosProvider will wrapp around to whatever the children are */}
            {props.children}
        </TodosContext.Provider>
    )
}
```

Now, in the TodoApp.js we are able to use the TotosProvider wrapper (our functional component):

```JavaScript
   <TodosProvider>
      <TodoForm addTodo={addTodo} />
      <TodoList
        todos={todos}
        removeTodo={removeTodo}
        toggleTodo={toggleTodo}
        editTodo={editTodo}
      />
      {/* <button onClick={()=>setMood("angry")}>Click to get angry</button> */}
  </TodosProvider>
```
## Consuming the Todo Context

We need to include useContext hook in all the components that need access to that context.

```JavaScript
function TodoForm() { // do pass anymore { addTodo} as parameter
  const { addTodo } = useContext(TodosContext); // get the { addTodo}  from the TodosContext
  // ..
```

The TodoList function becomes:

```JavaScript
function TodoList() { // no need for params
  const { todos } = useContext(TodosContext); // we need only the todos list from the TodosContext
  if (todos.length)
    return (
      <Paper>
        <List>
          {todos.map((todo, i) => (
            // To add a key to a fragment, we have to use the long-hand version
            // rather than <> </>, we have to use <React.Fragment>
            <React.Fragment key={i}>
              <Todo
                {...todo}
                key={todo.id} // we removed the addTodo, removeTodo, ... no need because we were just passing those down
              />
              {i < todos.length - 1 && <Divider />}
            </React.Fragment>
          ))}
        </List>
      </Paper>
    );
  return null;
}
```
... do the same for the other components ...

# Introduce a new pattern - instead of multiple methods write a single function - a reducer - fix reloading issue

For each piece of functionality, inside the useTodoState hook, we create a new method and also we have to use the TodosContext and get each this kind of function, in each file we whant to use it.
A cleaner way is to write a single function, called a reducer. This fc. takes an input, and dependig on the action specified as an input, and it will return the appropriate state.

Now there's a third one.

And the way that a context works whenever its value changes it has one single value.

Whenever something in that value changes it's going to pass down new data causing a re render in whatever

components are consuming that context.

So all of our components right now are consuming our TodosContext.

The fact is that the context is changing and it's passing down new information to the components, even thought we're just not using

the exact info that changed, if is part of the context and we're getting all the data because in the todos.contex.js, in the TodosCntext Provider we pass all the data:

```JavaScript
  <TodosContext.Provider value={{ todos, addTodo, removeTodo, toggleTodo, editTodo } }>
      {/* the component TodosProvider will wrapp around to whatever the children are */}
      {props.children}
  </TodosContext.Provider>
```        
The undesired effect and deffect is that each time an element from todos changes, all the components that use the TodosContext are re-rendering (though they only get from the TodosContext, only the methods, wich do not change).
=> we need to split up the context in 2 contexts: 
 - store the methonds wich do not change in a separate context
 - a context for the state that needs to trigger frequenly

 ## useReducer - a built-in Hook - an alternative to useState

 The useReducer accepts a reducer of (type, action) => newState, and returns the current state paired with a dispach method;

 ```JavaScript 
 const [state, dispatch] = useReducer(countReducer, {count: 10});
 // useReducer makes a piece of state, that was set initialy to {count:10} and it returns that piece of state (state), so we have access to it, and also returns the dispatch,
 // which is using the contReducer that, whenever we call dispatch, we pass it an action, while whatever is returned from countReducer at each dispatch, will update the state
 ```

The TodosProvider  becomes:

```JavaScript
export function TodosProvider(props) {
    const [todos, dispatch] = useReducer(todoReducer, defaultTodos);
    return (
        <TodosContext.Provider value={{ todos, dispatch } }>
            {/* the component TodosProvider will wrapp around to whatever the children are */}
            {props.children}
        </TodosContext.Provider>
    )
}
```
Next, update all components that use data from the TodosContext, eg: TodoForm:

```JavaScript
function TodoForm() {
 // ...
  const { dispatch } = useContext(TodosContext); // get the { dispatch}  from the TodosContext

  return (
    <Paper style={{ margin: "1rem 0", padding: "0 1rem" }}>
      <form
        onSubmit={e => {
          e.preventDefault();
          dispatch({type: "ADD", task: value}) // we no longer have addTodo method, but the generic dispatch method
          // ...
```

Fix the issue splitting into 2 contexts:

```JavaScript
export const TodosContext = createContext();
export const DispatchContext = createContext();

// we already have the useTodoState hook that gives us all the pieces we need
// what is missing is a context that will call this hook and then use these pieces and store them as a value for that context
export function TodosProvider(props) {
    const [todos, dispatch] = useReducer(todoReducer, defaultTodos);
    return (
        // in the TodosContext we need to pass just one thing, not an object like before
        <TodosContext.Provider value={todos}>
            {/* pass  the actual value of dispatch - not a new object */}
            <DispatchContext.Provider value={dispatch}>
                {/* the component TodosProvider will wrapp around to whatever the children are */}
                {/* props.children are wrapped in 2 compenents */}
                {props.children}
            </DispatchContext.Provider>
        </TodosContext.Provider>
    )
}
```
#  Memo - Higher Order Component built into React - To Speed Up the App

Because the todos is changing and we are mapping over todos, when a todo is changing, all todos components re-render = deffect;
If we had a class based componend we would use PureComponent, instead of Component:

```JavaScript
class Todos extends PureComponent {

}
```
With functional components we use React.memo component, that we wrapp around my entire functional component.

In Todo.js:

```JavaScript
export default memo(Todo);
```
- with this, React is simply remembering the old component and if there is no change, it will just render the component from cache.
# Use Custom Hook: Reducer + LocalStorage

Instead of using useState in order to operate on LocalStorage, we are going to use useReducer

```JavaScript
// ...
  const [state, dispatch] = useReducer(reducer, defaultVal, () => {
    let value;
    try {
      value = JSON.parse(
        window.localStorage.getItem(key) || String(defaultVal)
      );
    } catch (e) {
      value = defaultVal;
    }
    return value;
  });
  useEffect(() => {
    window.localStorage.setItem(key, JSON.stringify(state));
  }, [state]);

  return [state, dispatch];
}
// ...
```