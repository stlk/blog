---
layout: post
title: Tips for writing maintainable redux apps
---

I attended great talk by [Gabe Scholz](https://github.com/garbles) which was full of useful and concise tips for writing better and more maintainable redux apps. It struck me that I was just recently coping with these issues so I decided to share my view on these issues.

## Data selector functions

Instead of having same data retrieval logic in multiple places you can extract it to function and place it alongside your reducer. This makes refactoring so much easier.

{% highlight js %}
export function isLoaded(globalState) {
  return globalState.transactions && globalState.transactions.loaded;
}
{% endhighlight %}

## Normalize conditionals at the boundary

This cryptic name hides pretty simple idea. Sometimes it's not easy to decide when you should create new action or just add parameter to existing one. The best way might be to write a new action for each user action. Don't try to optimize the code by grouping together multiple user actions that result in similar outcomes.

There are two benefits to this approach. Actions are easier to typecheck and it's easier to remove functionality.

{% highlight js %}
case 'NEXT_STEP': {
  const newStepName = state.steps[state.currentStepName].success;
}
case 'NEXT_STEP_FAILURE': {
  const newStepName = state.steps[state.currentStepName].failure;
}
case 'NEXT_STEP_TIME_EXCEEDED': {
  const newStepName = state.steps[state.currentStepName].time_exceeded;
}
{% endhighlight %}

## Try redux-observable instead of redux-saga

When you have difficulties learning redux-saga, you might want to try redux-observable. They are both solving same issue. If  you are more familiar with RxJS than generators, redux-observable might be better fit for you.

## Deterministic reducers

When working with time or generating ID's in initial state it might be better to initialize these values in action. This makes reducers easier to test.

{% highlight js %}
const initialState = {
  steps: [],
  startTime: new Date(),
};
{% endhighlight %}

Having "random" value in initial state makes snapshot-based testing a bit harder. Isolate this value in action so you can use different value in tests.

## Avoid logic in components

Write dumb components and use HOC when needed. When components are almost like templates, there's almost no reason to test them. In this example `formatTime` is tested on its own, so I got away with not testing this component.

{% highlight js %}
export default ({ currentTime, duration }) => {
  return (
    <div className="seekInfo">
      {formatTime(currentTime)}
      /
      {formatTime(duration)}
    </div>
  );
};
{% endhighlight %}


### Want more?

Subscribe to my four week series of react basics on [https://www.getdrip.com/forms/83041389/submissions/new](https://www.getdrip.com/forms/83041389/submissions/new).
