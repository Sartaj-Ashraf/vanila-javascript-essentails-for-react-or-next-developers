# Vanila javascript essentails for react or next developers
React or Nextjs Developers' Guide to Vanilla JavaScript
These notes will help you become more versatile developer and gain a deeper grasp of the web platform by guiding you through the re-implementation of common React patterns and features using only Vanilla JS.

Overview
These notes are for you if you are at ease using React and its ecosystem to create apps but lack confidence when working with raw JavaScript and browser APIs. We'll presume you have constructed React apps and understand the fundamentals of web development (HTML, CSS, and JS), but you may not have developed sophisticated apps without a framework. By demonstrating how React's abstractions translate to straightforward JavaScript methods, we'll close that gap.

You'll discover how to construct dynamic interfaces, control state, put components together, manage routing & use the platform alone to carry out side effects (no frameworks). To solidify your grasp, we'll go over the foundational concepts of JavaScript, such as closures and the event loop. After that, we'll start creating actual projects with Vanilla JS, ranging from straightforward to-do lists to more complex applications like movie explorers and more. As we go along, we'll stress utilizing a browser and editor instead of a build toolchain whenever possible. After that, we'll demonstrate how to leverage bundlers (using Vite) and even a little Node.js for backend concepts to get ready for production.

A certain facet of app development is covered in each chapter. Explanations, code samples, and analogies to how you would accomplish the same with React are all included in the chapters. We've included a number of practical projects and tutorials to help you practice the ideas. 

# Chapter 1. Essential JavaScript Concepts

Review key JavaScript principles before coding. React handles many details automatically. Master these to recreate React behaviors in plain JavaScript.

### 1.1 Scope and Function Closures

Functions act as first-class objects and capture closures. A closure lets an inner function access variables from its outer scope after the outer function completes.

Consider this counter setup:
```text
function setupTracker() {
let total = 0;
return function update() {
total += 1;
console.log(total);
};
}

const tracker = setupTracker();
tracker(); // logs 1
tracker(); // logs 2
```
The update function holds onto total across calls. Closures enable data privacy in plain JS. React's useState hook depends on closures to persist values across renders.

### 1.2 Asynchronous Execution and Event Loop

React effects and callbacks depend on JS async behavior, such as data fetches. Grasp the event loop to manage non-blocking operations.

JS runs on one thread with a call stack for sync code and queues for async tasks. setTimeout, promises, and events enter queues. The loop shifts tasks to the stack when empty.
​

This keeps the interface responsive during delays. A fetch().then() callback waits for stack clearance before executing. Use this knowledge to prevent state races or invalid DOM changes.
​

### 1.3 Object Inheritance via Prototypes

React relies on objects for props and state. Prototypes drive JS object sharing.

Objects link to prototypes for property lookup. If absent locally, JS searches the chain upward.
​

Arrays like let items = gain push from Array.prototype. Classes build on prototypes:
```
class Vehicle {
constructor(name) {
this.name = name;
}
move() {
console.log(${this.name} moves forward);
}
}

const v = new Vehicle('Honda');
v.move(); // Honda moves forward
```
Vehicle.prototype.move supplies the method. Recognize prototypes in library code or custom class designs.
​

### 1.4 Context with this Binding

Functional React components limit this usage. Plain JS DOM code needs it for methods and events.

this points to the invocation context:
```text
const user = {
id: 'Bob',
sayHi() {
console.log(Hi from ${this.id});
}
};
user.sayHi(); // Hi from Bob
```
Extracted as const hi = user.sayHi; hi(); loses context, setting this to undefined in strict mode. Arrow functions or bind fix this in handlers. DOM events set this to the target element.

### 1.5 Key Takeaways

Closures, async loop, prototypes, and this underpin UI patterns. Apply them in projects ahead. Reference as you code. Proceed to DOM construction next.

# Chapter 2. Direct DOM Control: UIs in Plain JS

React virtualizes DOM updates. In plain JS, handle creation, changes, and removal yourself. Build dynamic interfaces and sync with data shifts.

### 2.1 Element Creation and Display

React JSX compiles to DOM calls. Plain JS uses createElement or innerHTML.

Target a root like const root = document.getElementById('root');

Build safely:
```text
const startBtn = document.createElement('button');
startBtn.textContent = 'Start';
root.appendChild(startBtn);

#Add traits:

startBtn.className = 'start-btn';
startBtn.setAttribute('aria-label', 'Begin app');
```
For trees, chain creates or clone <template> elements. Avoid raw innerHTML with user input to block scripts.
​

### 2.2 Targeted DOM Changes

React diffs virtually then patches. Plain JS targets exact nodes for speed.

Track a score display:
```text
let score = 0;
const scoreNode = document.createElement('span');
scoreNode.textContent = score;
root.appendChild(scoreNode);

const addBtn = document.createElement('button');
addBtn.textContent = '+1';
root.appendChild(addBtn);

addBtn.addEventListener('click', () => {
score += 1;
scoreNode.textContent = score;
});
```
Alter only text. Batch multiples in DocumentFragment to cut reflows. Use requestAnimationFrame for groups.
​

### 2.3 Event Management

React props like onClick attach handlers. Plain JS uses addEventListener.

Toggle a like heart:
```text
const heart = document.createElement('span');
heart.textContent = '♡';
heart.style.cursor = 'pointer';
let liked = false;
heart.addEventListener('click', () => {
liked = !liked;
heart.textContent = liked ? '♥' : '♡';
});
root.appendChild(heart);
```

Closure tracks liked state. Prefer addEventListener over onclick for multiples. Delegate on parents for lists: check event.target.

### 2.4 Declarative vs Direct Approach

React declares UI from state; it updates DOM. Plain JS issues direct commands.

React example:
```text
function Scoreboard() {
const [score, addScore] = useState(0);
return (
<div>
<span>{score}</span>
<button onClick={() => addScore(score + 1)}>+1</button>
</div>
);
}
```

# Chapter 3. Using Vanilla JavaScript to Manage State
The core of any dynamic application is state management. React handles state via useState, useReducer, Context, or third-party libraries like Redux/MobX. Without React, state management is still necessary; there isn't a built-in global solution, so you'll have to utilize custom patterns, closures, or plain objects. The good news is that you can create your own light state management system using JavaScript's flexibility.
 
 ### 3.1 What Does "State" Mean Outside of React?
State, to put it simply, is any data that affects the UI's display, which is subject to change. State can exist in a Vanilla JS application in:

variables or objects in your script module, such as a memory array of to-do lists.
The DOM itself: DOM properties contain many sorts of state, such as the content of an input field, the checked status of a checkbox input, etc.
Browser APIs include the URL (which stores state in query parameters or hash) and localStorage, which maintains state throughout page loads.
In-memory singleton store: To store global state and alert certain areas of your application to update, you may either build a custom object or utilize an ES6 class or closure (much like a Redux store). 
Let's consider a simple global state example. Suppose we want a state object that holds a count and allows components to subscribe to changes:
```text
// A simple state store in Vanilla JS
const appState = (() => {
  let state = { count: 0 };
  const listeners = [];
  return {
    getState() {
      return state;
    },
    setState(newState) {
      state = { ...state, ...newState };
      listeners.forEach(fn => fn(state));
    },
    subscribe(fn) {
      listeners.push(fn);
      return () => { // return unsubscribe function
        const index = listeners.indexOf(fn);
        if (index > -1) listeners.splice(index, 1);
      };
    }
  };
})();
```
This is a very trimmed-down store (inspired by Redux patterns). It uses a closure to hold a private state and listeners. Components can call appState.subscribe(renderFunction) to be notified when state changes, and appState.setState({count: 5}) to update state. For example:
```text
// Component A: display count
const countDisplay = document.createElement('div');
function renderCount(state) {
  countDisplay.textContent = `Count: ${state.count}`;
}
appState.subscribe(renderCount);
renderCount(appState.getState());
appRoot.appendChild(countDisplay);

// Component B: increment button
const incBtn = document.createElement('button');
incBtn.textContent = "Increment";
incBtn.onclick = () => {
  const current = appState.getState().count;
  appState.setState({ count: current + 1 });
};
appRoot.appendChild(incBtn);
```
Now, whenever incBtn is clicked, the state's count is updated and our subscribed render function updates the display. We've essentially mimicked a React useState + re-render in Vanilla JS! This approach uses the observer pattern (listeners). It's rudimentary but works for small apps.


### 3.2 State Module Pattern and Closures
One method of encapsulating state is to use closures, such as the IIFE mentioned above. Using an ES module alone is another popular method. You can have a state.js module since each module is a singleton, meaning it is evaluated just once:
```text
// state.js
export const state = {
  todos: [],
  filter: 'all'
};
```
The same object will be referenced by any module that imports state. if your code changes state in any way.Other parts notice the difference, too. This is an ad hoc, albeit basic, state container. The lack of an event system to alert you to changes is a drawback. To change the user interface, you may use this in conjunction with manual function calls or custom events (covered next).

### 3.3 Managing State Changes with Custom Events
You can create and dispatch your own events using the CustomEvent constructor that browsers offer. Without a complete pub-sub implementation, this can be a simple method of communicating state changes.

For instance, each time our state changes, we could send out a custom event:
```text
// When state changes:
const event = new CustomEvent('stateChange', { detail: state });
document.dispatchEvent(event);
```
```text
document.addEventListener('stateChange', e => {
  const newState = e.detail;
  // update UI accordingly
});
```
The document (or any primary component) is treated as an event bus in this method. Although Redux doesn't use DOM events, it's not too dissimilar from how Redux might notify subscribers. Decoupling is advantageous since components can listen for events without importing a particular state module.

### 3.4 State in Memory vs State in the DOM

A concise guide to choosing the correct source of truth in frontend applications.

#####  What Is State
State represents the current data of an application.  
Examples include input values, selected filters, and completed tasks.


##### DOM State
The browser already stores some state inside UI elements.

##### Examples
- `input.value`
- `checkbox.checked`
- `select.value`

##### When to Use
- Short lived UI interactions
- Data needed only at the moment of interaction

##### Typical Use Cases
- Search input text
- Temporary focus state
- Open or closed dropdowns


##### In Memory State
State stored in JavaScript objects or variables.  
This state drives logic and persistence.

##### Typical Use Cases
- Todo items
- User profile data
- Form submission payloads
- Filters affecting multiple components

##### Single Source of Truth
Each piece of data must have one authoritative source.

##### Rule of Thumb
- Application data lives in memory
- Ephemeral UI interaction lives in the DOM

Multiple sources without synchronization lead to inconsistent UI and bugs.


##### Syncing DOM and Memory
Some UI elements reflect stored data.

 ##### Example
Todo checkbox flow:
1. User clicks the checkbox
2. JavaScript updates `todo.completed` in memory
3. UI renders based on updated data

Data drives the UI. The UI does not drive the data.


##### Key Takeaways
- Do not treat the DOM as application data storage
- Avoid duplicating state without sync logic
- Maintain a clear single source of truth

### 3.5 Persistence and Local Storage
The ability to access localStorage, sessionStorage, IndexDB, and other resources without the need for other libraries is a significant benefit of writing direct browser code. A straightforward key-value store called localStorage keeps information for the same domain across page loads. Although synchronous, it works well with tiny data sets (5–10MB in most browsers).

To prevent data loss in the event of a user refresh, it is typical practice in Vanilla JS applications to save state to localStorage (e.g. keeping our to-do list saved).

Example: Using localStorage to load and save state:
```text
// Save todos array to localStorage
localStorage.setItem('todos', JSON.stringify(state.todos));
```

 - Later, on page load, initialize state from storage:
```text
const saved = localStorage.getItem('todos');
if (saved) {
  state.todos = JSON.parse(saved);
  renderTodos();  // update UI to reflect saved todos
}
```

This will be incorporated into our to-do app project. It's the Vanilla JS counterpart of accessing an API or utilizing a persistence layer in a React application.

# Chapter 4. Component Composition and Code Organization
One reason for React's popularity is the component model – breaking the UI into self-contained, reusable pieces that manage their own state and logic. In Vanilla JS, we don't have a built-in component system, but we can create our own structures and patterns to achieve similar modularity. This chapter explores ways to compose UIs from smaller parts using functions, classes, and web standards.

### 4.1 Components-Based Thinking (Without a Framework)
Dividing your user interface into sensible sections is a good idea, even if you don't use React. A to-do app, for instance, may feature a form, a list, and a header. Each chunk can be represented by a function that generates its DOM elements and returns an element or collection of elements.

Vanilla JS Functional Components: You can create a function that yields a DOM node. For example:
```text
function createTodoItem(todo) {
  const li = document.createElement('li');
  li.textContent = todo.title;
  if (todo.done) li.classList.add('done');
  li.addEventListener('click', () => {
    todo.done = !todo.done;
    li.classList.toggle('done');
  });
  return li;
}
```
This function takes some data (todo) and returns a DOM element (<li>). You could use it like:
```text
const list = document.createElement('ul');
state.todos.forEach(todo => {
  list.appendChild(createTodoItem(todo));
});
```

With internal behavior (toggling done), we have effectively created a Vanilla JS "component" for a to-do list item. It's a building block, but it's not as reliable as a React component (it utilizes the todo object directly and has no lifecycle beyond what we coded).

Composite Components: One component may make use of another. For instance, as shown above, a TodoList component may loop through todos and call createTodoItem for each one. Alternatively, a Header component might generate a <header> element with a title element and possibly some buttons.

You may easily reuse and maintain the focus of each component by creating functions.

### 4.2 Using Classes to Encapsulate Behavior

State, DOM generation, and event handlers can all be conveniently combined in one location with ES6 classes. A class instance can be compared to a component instance.
For example, let's create a simple Counter component as a class:
```text
class CounterComponent {
  constructor(initialCount = 0) {
    this.count = initialCount;
    // Create DOM elements
    this.el = document.createElement('div');
    this.display = document.createElement('span');
    this.display.textContent = this.count;
    const incBtn = document.createElement('button');
    incBtn.textContent = '+1';
    // Append elements
    this.el.appendChild(this.display);
    this.el.appendChild(incBtn);
    // Event handler
    incBtn.addEventListener('click', () => this.increment());
  }
  increment() {
    this.count++;
    this.display.textContent = this.count;
  }
  mount(parent) {
    parent.appendChild(this.el);
  }
}
```
Let's build a basic Counter component as a class, for instance:This class contains the DOM (this.el and children), the counter's state (this.count), and the updating algorithm (increment). The component is connected to a parent using the mount technique. If we wanted to remove it and perform any cleanup, we could also add an unmount function (in this straightforward scenario, removing the DOM node removes the event listener automatically because it is connected to that node).
Using it:
```text
const counter = new CounterComponent(5);
counter.mount(document.body);
```
This class contains the DOM (this.el and children), the counter's state (this.count), and the updating algorithm (increment). The component is connected to a parent using the mount technique. If we wanted to remove it and perform any cleanup, we could also add an unmount function (in this straightforward scenario, removing the DOM node removes the event listener automatically because it is connected to that node).

This method is comparable to writing a component in simple JavaScript or even utilizing light frameworks like Stimulus. It's organized but necessary.
Advantages: You group related code together. The class can include all of the logic if the component becomes more complicated (for example, a counter with +/- and reset).

Cons: This binding requires caution and is a little verbose; we used an arrow function for the event, which accurately represents this.

Although we won't utilize classes for every component in the book, it's helpful to be aware of this option, particularly for widgets that have a lot of internal state management.

### 4.3 Templates and HTML Fragments
Using HTML templates is another method for defining the structure of a component. In HTML, the <template> tag is inert content that can be copied in JavaScript. For instance:

```text
<template id="itemTemplate">
  <li class="item">
    <span class="title"></span>
    <button class="delete">✕</button>
  </li>
</template>
```
```
const template = document.getElementById('itemTemplate');
function renderItem(item) {
  const fragment = template.content.cloneNode(true);
  fragment.querySelector('.title').textContent = item.title;
  fragment.querySelector('.delete').onclick = () => removeItem(item);
  return fragment;
}
```
This yields a DocumentFragment with the item's <li> filled in. Some developers prefer that HTML structure and JS functionality be kept apart through the use of templates. It's similar to JSX in that you use HTML to construct the structure and JS to fill in the data.
Although templates won't be used extensively in our examples (to keep everything in one location), be mindful of this capability for bigger projects or situations where you need to create a lot of static HTML.

### 4.4 Web Elements (Custom Components)
A collection of web platform APIs known as "Web Components" are used to create reusable custom elements with encapsulated HTML, styling, and behavior. They are essentially a component system for the browser. You can use your element using a custom tag after defining a class that extends HTMLElement.

A simple illustration of a web component
```text
class HelloWorld extends HTMLElement {
  connectedCallback() {
    this.textContent = "Hello, World!";
  }
}
customElements.define('hello-world', HelloWorld);
```
Now in HTML/JS, you can do <hello-world></hello-world> and it will invoke that class. Web Components can even have their own internal shadow DOM for encapsulated styling.
Despite their strength, Web Components may be too much for our purposes. They excel at developing cross-framework compatible design systems or UI libraries. 

### 4.5 File and Module Organization
Think about dividing components into modules (files) as you construct them. It's possible that you have components/Header.js, components/TodoList.js, and so on that export classes or functions. After that, your main script may import those and put the application together. For instance:
```text
// main.js
import { Header } from './components/Header.js';
import { TodoList } from './components/TodoList.js';

document.body.appendChild(Header());
document.body.appendChild(TodoList());
```


Each file remains focused when ES module imports are utilized. You can load imports in a no-build setup by including <script type="module" src="main.js"></script> in your HTML (note: owing to module CORS restrictions, you must serve over HTTP, not file://).

Modules will be used to organize project code and introduce bundling in later chapters. For the time being, be aware that composition occurs not only in code structures but also in the way your codebase is organized.

### 4.6 Parameterization and Reusability
It is possible to configure a Vanilla JS "component" using parameters. A createButton(text, onClick) function, for example, can produce various buttons as required. This reminds me of React props.
```text
function createButton(label, onClick) {
  const btn = document.createElement('button');
  btn.textContent = label;
  if (onClick) btn.addEventListener('click', onClick);
  return btn;
}
// usage
const saveBtn = createButton('Save', () => saveData());
const cancelBtn = createButton('Cancel', () => cancel());
```
You may add flexibility to your Vanilla components by sending callbacks and other data. By writing them, you may construct a whole toolbar or form from discrete parts.

### 4.7 Summary
We've explored multiple strategies to compose and reuse UI pieces without React:

Functions that return DOM (like our createTodoItem), Classes to encapsulate more complex components (CounterComponent), Template-driven rendering, Even Web Components as an advanced option.

# Chapter 5: Application Navigation on a Single Page
The majority of contemporary apps are Single-Page Applications (SPAs), which means that even when the user switches between "pages" or views, the user interface is updated without requiring a complete page reload. To deal with this in React, you may utilize React Router. The History API (or the more traditional hash-based method) can be used to accomplish routing in Vanilla JS. The implementation of client-side routing from scratch is covered in this chapter.
### 5.1 The API for Browser History
We can change the browser's URL and history stack using the History API without having to restart the page. Important components of the API:

A new item with a new URL is added to the history stack using history.pushState(stateObj, title, url) (the domain must stay the same). (How is the window used?API for history?-GreatFrontEnd.
The current entry is replaced by history.replaceState(stateObj, title, url).
When the active history entry changes (for example, when the user clicks Back or Forward, or when you call pushState), the popstate event is triggered on the window.
We may alter the URL as though navigating by using pushState. For instance:
```text
// Navigate to "/about" without reloading
history.pushState({}, "", "/about");
```
As a result, the URL displayed in the address bar will now be /about. The server must handle /about if the user presses enter or refresh at this point (or you need a fallback to index.html). However, we should change the user interface to display the "About" text within our SPA after using pushState.

We are able to respond to back-and-forth navigation by listening to Popstate. For example:
```text
window.addEventListener('popstate', (event) => {
  // event.state contains the stateObj we passed (if any)
  renderCurrentRoute();
});
```
We'll likely use location.pathname to determine what to show.
