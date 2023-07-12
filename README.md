# Angular Signals
We are going to start explaining signals in Angular with a simple example.

## Signals component Typescript
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

## Signals component Template
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




