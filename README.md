# Angular Signals

# 1. Introduction

Please for a better explanation see these youtube videos: https://www.youtube.com/playlist?list=PLErOmyzRKOCr07Kcnx75Aqh6PWSbIokPB

To clarify the concepts of **Signal** and **Zone.js** in **Angular**, especially considering their roles and differences in **Angular 17**, we need a brief understanding of both technologies within the context of Angular, a platform and framework for building single-page client applications using HTML and TypeScript.

**Zone.js** is a library that Angular has used for **automatic change detection**. It works by monkey-patching asynchronous APIs in the browser (like setTimeout, Promise, etc.) to notify Angular when to run change detection. This means that when you perform an asynchronous operation, Zone.js ensures Angular knows when the operation completes so that it can **update the UI with any changes**. This process is crucial for keeping the application's state and the UI in sync.

**Signal**, on the other hand, is a new addition to Angular's ecosystem, introduced as a more efficient and straightforward way **to manage change detection and state updates** in Angular applications. It offers a simpler API and aims to **replace Zone.js** in many cases. Signals provide a **reactive programming model** that makes it easier to **manage and propagate changes across components**. With Signals, developers can more directly control when and how changes are detected and applied, leading to potentially more performant and predictable applications.

![image](https://github.com/luiscoco/Angular_Signals-Sample/assets/32194879/c74a6b21-0a06-43ca-832e-f566f7d57b77)

Here are the key differences and considerations when **comparing Signal with Zone.js in Angular 17**:

**Performance and Efficiency**: Signals are designed to offer a more efficient, less intrusive way of tracking and reacting to changes in application state. They could lead to performance improvements, especially in complex applications, by reducing the overhead associated with automatic change detection via monkey-patching in Zone.js.

**Control and Predictability**: With Signals, developers gain finer control over change detection, making the behavior of applications more predictable. This contrasts with Zone.js, where the automatic triggering of change detection can sometimes lead to performance issues or unexpected behavior in complex scenarios.

**Simplicity and Usability**: Signals provide a simpler, more intuitive API for managing state and reactivity in Angular applications. This can make the framework easier to use and learn, especially for new developers or those coming from other reactive programming backgrounds.

**Migration and Compatibility**: For existing Angular applications that rely heavily on Zone.js, migrating to Signals may require some refactoring to achieve the best results. However, Angular aims to support both models to ensure backward compatibility and give developers the flexibility to choose the approach that best fits their needs.

**Future Direction**: Angular's introduction of Signals signifies a move towards a more reactive, function-based approach to building applications. This aligns Angular more closely with modern JavaScript and TypeScript development practices, potentially making it more attractive to a broader range of developers.

In summary, while **Zone.js** has been an integral part of Angular for **managing change detection** through automatic patching of asynchronous operations, the introduction of **Signals** in Angular 17 offers a new, **more efficient**, and **more controllable** way to handle reactivity and state changes. The choice between using Signal or continuing with Zone.js depends on the specific needs and complexity of your project, as well as your preferences for application **performance, control, and development simplicity**.

We are going to start explaining signals in Angular with a simple example.

# 2. Signal sample

## 2.1. Signals component Typescript

We first define a "counter" signal and we initialize to zero value. The signal keyword creates a Signal that can be "set" or "updated" directly.
```
 counter = signal(0);
```

In the component constructor we call the "effect()" function to write in the internet browser console the "counter" value:
```
constructor() {
    effect(() => console.log(this.counter()));
  }
```

When we call the increment() function then we call the "counter" signal update() method. 
The update() method updates the value of the signal based on its current value, and notify any dependents.
```
 increment() {
    this.counter.update((oldCounter) => oldCounter + 1);
  }
```

Another option is to set the signal value:
```
this.counter.set(this.counter() + 1);
```

This is the component whole Typescript code:
```Typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-signals',
  templateUrl: './signals.component.html',
  standalone: true,
})
export class SignalsComponent {
  counter = signal(0);
  constructor() {
    effect(() => console.log(this.counter()));
  }
  increment() {
    this.counter.update((oldCounter) => oldCounter + 1);
  }
  decrement() {
    this.counter.update((oldCounter) => oldCounter - 1);
  }
}
```

## 2.2. Signals component Template

We show in a paragraph HTML element the "counter" signal value:
```
<p id="counter-output">Counter: {{ counter() }}</p>
```

When we press the Increment button then we call the increment() function:
```
<button (click)="increment()">Increment</button>
```

This is the template whole code:
```HTML
<h1>Signals</h1>

<div id="counter">
  <p id="counter-output">Counter: {{ counter() }}</p>
  <div id="counter-btns">
    <button (click)="decrement()">Decrement</button>
    <button (click)="increment()">Increment</button>
  </div>
</div>
```

# 3. Writable signals

Writable signals provide an API for updating their values directly. You create writable signals by calling the **signal** function with the **signal's initial value**:

```typescript
const count = signal(0);
// Signals are getter functions - calling them reads their value.
console.log('The count is: ' + count());
```

To change the value of a writable signal, either **.set()** it directly:

```typescript
count.set(3);
```

or use the **.update()** operation to compute a new value from the previous one:

```typescript
// Increment the count by 1.
count.update(value => value + 1);
```

Writable signals have the type **WritableSignal**.

# 4. Computed signals

**Computed signal** are **read-only signals** that **derive their value from other signals**.

You define computed signals using the computed function and specifying a derivation:

```typescript
const count: WritableSignal<number> = signal(0);
const doubleCount: Signal<number> = computed(() => count() * 2);
```

The **doubleCount** signal depends on the **count** signal. Whenever **count** updates, Angular knows that **doubleCount** needs to update as well.

## 4.1. Computed signals are not writable signals

You cannot directly assign values to a computed signal. That is,

```typescript
doubleCount.set(3);
```

produces a compilation error, because **doubleCount** is not a **WritableSignal**.

## 4.2. Computed signals are both lazily evaluated and memoized

doubleCount's derivation function does not run to calculate its value until the first time you read doubleCount. 

The calculated value is then cached, and if you read doubleCount again, it will return the cached value without recalculating.

If you then change count, Angular knows that doubleCount's cached value is no longer valid, and the next time you read doubleCount its new value will be calculated.

As a result, you can safely perform computationally expensive derivations in computed signals, such as filtering arrays.

## 4.3. Computed signal dependencies are dynamic

Only the signals actually read during the derivation are tracked. For example, in this computed the count signal is only read if the showCount signal is true:

```typescript
const showCount = signal(false);
const count = signal(0);
const conditionalCount = computed(() => {
  if (showCount()) {
    return `The count is ${count()}.`;
  } else {
    return 'Nothing to see here!';
  }
});
```
When you read conditionalCount, if showCount is false the "Nothing to see here!" message is returned without reading the count signal. 

This means that if you later update count it will not result in a recomputation of conditionalCount.

If you set showCount to true and then read conditionalCount again, the derivation will re-execute and take the branch where showCount is true, returning the message which shows the value of count.

Changing count will then invalidate conditionalCount's cached value.

Note that dependencies can be removed during a derivation as well as added. 

If you later set showCount back to false, then count will no longer be considered a dependency of conditionalCount.

## 4.4. Effects

Signals are useful because they notify interested consumers when they change. An effect is an operation that runs whenever one or more signal values change. You can create an effect with the effect function:

```typescript
effect(() => {
  console.log(`The current count is: ${count()}`);
});
```

Effects always run at least once. When an effect runs, it tracks any signal value reads. 

Whenever any of these signal values change, the effect runs again.

Similar to computed signals, effects keep track of their dependencies dynamically, and only track signals which were read in the most recent execution.

Effects always execute asynchronously, during the change detection process.




