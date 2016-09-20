---
layout:     post
title:      "Angular 2 Animations - Foundation Concepts"

date: 2016-09-16
imageUrl: '/images/banner/angular-2-component-animations.jpg'

summary: "Animation in Angular 2 is now easy and more intuitive... Learn foundational animation concepts and start animating your Angular 2 components!"

categories:
  - angular

tags:
  - angular2
  - animation
  - components
  - relative paths

topic: components

author: thomas_burleson
---

Animations features often are scary goals for developers. And Angular's doctrine

> "... controllers should not directly modify DOM elements!"

made Animation features intimidating as hell. But **Angular 2 animations are not scary!** Templates are
closely associated/integrated with `@Component`. We will notice that animations following a similar pattern.

Let's build a component that hides and shows its contents, uses fade animation effects, and allows external components to
easily trigger those fade effects.

### Our Scenario

Here is a simple Angular 2 component with hide and show features. This sample, however, does not have animations (yet):

{% highlight js %}
{% raw %}
@Component({
  selector: 'my-fader',
    template: `
    <div *ngIf="visibility == 'shown'" >
      <ng-content></ng-content>
      Can you see me? 
    </div>
  `
})
export class MyComponent implements OnChanges {
  visibility = 'shown';

  @Input() isVisible : boolean = true;

  ngOnChanges() {
   this.visibility = this.isVisible ? 'shown' : 'hidden';
  }
}
{% endraw %}
{% endhighlight %}

This component simply publishes an `@Input() isVisible` property; which allows other external components to show/hide the text content... without any animations.

<iframe style="width: 100%; height: 600px" src="http://embed.plnkr.co/vUPTsY/" frameborder="0" allowfullscren="allowfullscren"></iframe>


### Configure Component Animations

We want the `my-fader` component to **fade-in** or **fade-out** its text content. And we want to *animate* those fades effects.

To start animating, let's first add animation metadata to our component.

{% highlight js %}
{% raw %}
@Component({
  ...,
  template : ``,
  animations: [
    ...
  ]
)]
class MyComponent() { ... }
{% endraw %}
{% endhighlight %}

Above we show that the `animations` metadata property is defined in the `@Component` decorator. Just like `template` metadata property!

Since our component has a `visibility` property whose state can change between `shown` and `hidden`, let’s configure animations to trigger and animate during each value change.

{% highlight js %}
{% raw %}
animations: [
  trigger('visibilityChanged', [
  state('shown' , style({ opacity: 1 })), 
  state('hidden', style({ opacity: 0 }))
  ])
]
{% endraw %}
{% endhighlight %}

Before we add more metadata, let's talk about the syntax above. What does it mean... and why is this syntax used?

The techno-speak above is saying that when the `visibilityChanged` property changes and the value `== shown`,
then the target element opacity changes to 1. And when the value changes to `== hidden`, the
target element opacity should change to 0.

> Note: The `[@visibilityChanged]` binding is on `<div>` child content element in the `<my-fader>` component.
It is NOT on the `<my-fader>` element itself. In other words, the animation target in our example
is actually the `<div>` element; not the component **host** element.

Now, you might also wonder where `visibilityChanged` comes from? Because our component property is just called `visibility`.
Hold your wild horses Mr. StageCoach, we'll clarify that soon!" Let's first talk about animation durations.

We want to animate these changes over a time duration instead of instantly hiding/showing the content.
We need to configure a *transition* to specify animation durations. With Angular 2 this is also suprisingly easy:

{% highlight js %}
{% raw %}
animations: [
  trigger(’visibilityChanged', [
    state('shown' , style({ opacity: 1 })), 
    state('hidden', style({ opacity: 0 })),
    transition('* => *', animate('.5s'))
  ])
]
{% endraw %}
{% endhighlight %}

With the above `transition`, we added information to the animation configuration so each trigger value change will have a 500 msec transition. 

So a **fade-in** (opacity 0 -> 1, with a duration of 500 msec) will occur when the value changes from `hidden` to `shown`. 
And likewise the **fade-out** (opacity 1 -> 0, with a duration of 500 msec) will occur when the value changes from `shown` to `hidden`.
By the way, you could also have used `animate('500ms')` to indicate the millsecond duration explicitly.

And what does the `transition('* => *', ...)` mean? Think of `* => *` as a transition from one state to another state;
where `*` is a wildcard to mean **any** state value. If we wanted the *fade-out* to be slower than the *fade-in*,
here is how we would configure the animation metadata:

{% highlight js %}
{% raw %}
animations: [
  trigger(’visibilityChanged', [
    state('shown' , style({ opacity: 1 })),
    state('hidden', style({ opacity: 0 })),
    transition('shown => hidden', animate('600ms')),
    transition('hidden => shown', animate('300ms')),
  ])
]
{% endraw %}
{% endhighlight %}

See how easy this is? This notation is so easy to understand.


#### The Essential Concept

The essential take-away Animation concept is that **Angular 2 Animations** are triggered on component <u>state changes</u>.
And developers should consider <u>state changes</u> to be equivalent to <u>value changes in a property</u> of the component instance.


### Linking Animation to the Component


<br/>
While we configured the Animation metadata,  I am sure you are wondering:

*  How is the animation property `visibilityChanged` actually connected to the component?
*  How are the animations linked to the component’s properties? 

Since data-binding features are already supported between the **component** and its **template**, 
Angular 2 uses a <u>special</u> template animation syntax to support triggering the animation after data-binding changes.
So in the component template, we can do this:

{% highlight html %}
{% raw %}
<div [@visibilityChanged]="visibility">
  Can you see me? I should fade in or out...
</div>
{% endraw %}
{% endhighlight %}


Above the `@visibilityChanged` is the special template animation property and it uses databinding 
`[@visibilityChanged]=“visibility”` to bind the component's visibility property to the animation 
trigger property `visibilityChanged`. 

Here is the entire component definition updated with Animation features:

{% highlight js %}
{% raw %}
import { 
  Component, OnChanges, Input, 
  trigger, state, animate, transition, style 
} from '@angular/core';

@Component({
  selector : 'my-fader',
  animations: [
  trigger('visibilityChanged', [
    state('shown' , style({ opacity: 1 })),
    state('hidden', style({ opacity: 0 })),
    transition('* => *', animate('.5s'))
  ])
  ],
  template: `
  <div [@visibilityChanged]="visibility" >
    <ng-content></ng-content>  
    <p>Can you see me? I should fade in or out...</p>
  </div>
  `
})
export class FaderComponent implements OnChanges {
  @Input() isVisible : boolean = true;
  visibility = 'shown';

  ngOnChanges() {
   this.visibility = this.isVisible ? 'shown' : 'hidden';
  }
}
{% endraw %}
{% endhighlight %}
<br/>


### Reducing Complexity

What if - instead of the using the extra `visibility` property - you just wanted to use the `isVisible` property directly?
This would obviate `ngOnChanges()` and reduce the code complexity to:

 {% highlight js %}
 {% raw %}
 @Component({
   animations: [
     trigger('visibilityChanged', [
       state('shown' , style({ opacity: 1 })),
       state('hidden', style({ opacity: 0 })),
       transition('* => *', animate('.5s'))
     ])
   ],
   template: `
   <div [@visibilityChanged]="isVisible" >
        ...
   </div>
   `
 })
 export class FaderComponent {
   @Input() isVisible : boolean = true;
 }
 {% endraw %}
 {% endhighlight %}

But this will not work without another **important** change to the animation metadata!

> Remember that the `@visibilityChanged` animation trigger property has defined states for the values: `shown` or `hidden`.

If you use the myFader::`isVisible` boolean property, then your animation state values must be changed to `true` and `false`
since those are the possible values of that property.


{% highlight js %}
{% raw %}
import {
  Component, OnChanges, Input,
  trigger, state, animate, transition, style
} from '@angular/core';

@Component({
  selector : 'my-fader',
  animations: [
    trigger('visibilityChanged', [
      state('true' , style({ opacity: 1, transform: 'scale(1.0)' })),
      state('false', style({ opacity: 0, transform: 'scale(0.0)'  })),
      transition('1 => 0', animate('300ms')),
      transition('0 => 1', animate('900ms'))
    ])
  ],
  template: `
  <div [@visibilityChanged]="isVisible" >
    <ng-content></ng-content>
    <p>Can you see me? I should fade in or out...</p>
  </div>
  `
})
export class FaderComponent implements OnChanges {
  @Input() isVisible : boolean = true;
}
{% endraw %}
{% endhighlight %}
<br/>

<iframe style="width: 100%; height: 600px" src="http://embed.plnkr.co/74lprkmzUGjT7UWbiyUr/" frameborder="0" allowfullscren="allowfullscren"></iframe>

> Extra Bonus: The demo has some extra features. The *host* `my-fader` element now has a purple background; when you
hide the `my-fader` content children you will see the host background. This change was added so you can visually see the
differences between the *host* and the *target* animation elements.

### Our Animation Workflow

Above we have an improved the component definition; enhanced with animation features.
Here is a workflow of the [animation] process:

*  the input value for `isVisible`
*  change detection triggers a call to `ngOnChanges()`
*  the component visibilty property changes
*  the template databinding updates the @visibilityChanged property value
*  the animation trigger is invoked
*  the state value is used to determine the animation
*  the target element opacity change animates for 500 msecs


### Philosophy of Animations

One of the design goals for Angular 2 Animations is to make it **easy** for developers. The syntax should be:

*  intuitive
*  declarative and
*  immediately associated with the component...
  *  the `animations` configuration is right above the Class definition!

The best part of Angular 2 Animation design is that the **component->template->animation** binding solution
<u>decouples</u> the animation from the component internals and uses the template as the binding bridge. The developer
decides which component properties should bind to which animation triggers, then simply sets the *state* values accordingly.

All the mechanics of preparing and managing the animations in hidden from the developer. This separation of concerns
provides HUGE benefits to allow developers to easily use Angular 2 Animations with custom architectures & custom implementations.

![super-cool](https://media.giphy.com/media/NUC0kwRfsL0uk/giphy.gif)

### Animations with Components Hierarchies

Components should never be concerned with the details regarding animation of child components. Parent components can monitor
and alter the public **state** of child components, but should never attempt to modify the internals of those components.

In our examples (above), parent components can simply change the state of the child `my-fader` instances and then magically the
contents of the `my-fader` instance will fadeIn or fadeOut.

> Recall that component state value is based on the value of the `isVisible` property.

{% highlight js %}
{% raw %}
@Component({
  selector : 'my-app',
  template: `

  <my-fader [visibility]="showFader">
    Am I visible ?
  </my-fader>

  <button (click)="showFader = !showFader"> Toggle </button>
  `
})
export class MyAppComponent {  
  showFader : boolean = true;
}
{% endraw %}
{% endhighlight %}

<iframe style="width: 100%; height: 600px" src="http://embed.plnkr.co/NbWGjs/" frameborder="0" allowfullscren="allowfullscren"></iframe>



### Summary

The Angular 2 Animation engine and compiler does all the hard work of the preparing, managing, and running the animations.

Developers use the `@Component` metadata to declaratively define the component styles, templates, and [now] animations.
And it is the component **template** that serves as the *bridge* to link the component instance state to the
animation trigger property.

<br/>

### Thanks

Kudos to [Matias Niemelä](https://twitter.com/yearofmoo) for the amazing Animation engine in Angular 2!

![matiasvikinghair](https://cloud.githubusercontent.com/assets/210413/18608523/49b8707c-7cb1-11e6-8d2c-ab43db07ca78.jpg)

These core animation features [discussed above] are available in the Angular 2.0.0 release. And never fear,
Matias and his team are working hard on more amazing, intuitive Animation features. So stay tuned for even MORE cool features and blogs coming soon!