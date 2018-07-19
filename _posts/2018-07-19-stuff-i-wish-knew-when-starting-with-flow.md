---
layout: post
title: Stuff I wish I knew when starting with Flow
---

Typing in dynamic languages is hard. Especially in small projects where interaction with libraries is hiding on every corner.
I tried to get into Flow at least two times before. Now I finally think I got far enough to experience enough of important obstacles that suffice to write about them.

## What editor should I use

Technically, both Atom and VS Code should work with Flow. However, VS Code wasn't working well for me. On the other hand [Flow for Atom IDE](https://github.com/flowtype/ide-flowtype) works great.

## Where to start

With Flow you can pick the file you want to start with. Flow will treat imported untyped files as type `any`.
I started with a React component which accepts simple object just to get my feet wet and get familiar with the syntax. After this exploration phase it's good to try getting business logic typed.

## Interacting with HTML elements

Finding the correct way to handle type casting of HTML elements took me surprisingly a lot of the time. But the solution turned out to be pretty simple. Flow looks for `instanceof` conditions and for following code it works with correct type. Useful tool for asserting HTML elements is `invariant`. It takes condition and error message and raises the error message when the condition isn't met.

```js
const form = document.getElementById(formId);
invariant(
  form != null && form instanceof HTMLFormElement,
  'Could not find a form.',
);
```

## Arrays - type syntax can actually differ

When specifying a type for an array I automatically typed `[Item]`. This is valid syntax in Flow, but it means something which you won't need most of the time. It means array with one item of type `Item`. You are most likely looking for `Item[]`. This took me quite some time to realize.

## Higher-Order Components

HOCs are sometimes tough to type correctly because it's so easy to mix in `any` type and lose the type checking. Let's try typing this `withLoader` HOC. I confess that I haven't figured out how to write fully generic HOC.

```jsx
function withLoader(LoaderComponent) {
  function innerComponent(WrappedComponent) {
    return props =>
      props.loading === true ? (
        <LoaderComponent {...props} />
      ) : (
        <WrappedComponent {...props} />
      )
  }
  return innerComponent
}
```

It's just a condition wrapped in two functions, which shouldn't be that hard. First call supplies loader component and second call supplies the component we are wrapping.

```js
const WrappedComponentWithLoader = withLoader(LoaderComponent)(WrappedComponent)
```

Looking at [Flow docs](https://flow.org/en/docs/react/types/#toc-react-componenttype) I found that React uses two types to define component types. I will use `React.StatelessFunctionalComponent` to keep things "simple".

```jsx
function withLoader(LoaderComponent: React.StatelessFunctionalComponent<any>) {
  function innerComponent<Props: { loading: boolean }>(
    WrappedComponent: React.StatelessFunctionalComponent<Props>
  ): React.StatelessFunctionalComponent<Props> {
    return props =>
      props.loading === true ? (
        <LoaderComponent />
      ) : (
        <WrappedComponent {...props} />
      );
  }
  return innerComponent;
}
```

After adding types this HOC looks pretty intimidating. I will try to walk you through without scaring you off.

`LoaderComponent: React.StatelessFunctionalComponent<any>` means that function `withLoader` accepts any stateless component. Then there is my generic wrapped component. `innerComponent<Props: { loading: boolean }>` part defines that function has generic type which is required to be object with `loading` field of type `boolean`. I used this type to define `Props` of my wrapped component. This is the only solution I found that passes up the types defined in decorated component so the type restrictions are correctly propagated and property `loading` is also required. The last line that is related to flow is `): React.StatelessFunctionalComponent<Props> {` which actually defines that new component will have type discussed previously.

You can play with this component on [Try Flow](https://flow.org/try/#0PTAEAEDMBsHsHdQGcAuAnAlgYxQWjlgIbQBQGAtgA6xoqgBUohSoASgKaE6iRqzmgARGk45BAbhIlIAVwB2ODLDmh4GFAAsAMrEIATdmgAUO-YYDC-anPZyUALjaiUAOgDKKQinbR2SJABi8orKxJZUyrYoADyEcgCeAHwAlKAA3iSgPMEoSioYcjZo4dZR0QAKfJRIjmmgcPoFAOaOAEawsL5xoAC+iUaZWaAA6miElJTseiWRdo4cXK4eXj5+gTl5YVazMZWw1YmDyfPO7p7evv5BCrmh0DM2dhVVSInpg1kiKDJoKpQvoAAvIchkN-vskC4GnpmkDAYDQOgZOxQAB+UADUGg6KmAzFbaPOjAEFY1KOTFYrLRUbjSbTAlRdIuZng6o9UDEj5DZKSLI9QZfH75QoWBl2ST8khYZSoUC4wxAjGpYGgaIwgBuiWhzWZLmiwA1iUkUpldAAsvEHoyEUY6t4AB50Hq1RHsR2OVCYORNXrKt4ZT7sb6-VWGtIOlA9fWGiVSaVyWUAES8hEVak08uMmeSRgtVrsPLjptAAEEJoqjH6MYM1RhNVzosnPPVdDDvYDw2hkeyI4DBJoMCxB6AsGMAF7xACEgg5JKpTdT2vbne7s4bC9djr7A6HLFHhAn07XVINdcOhZIbuotFABkghBk0DoZco4iAAa) and see that Flow warns us about missing props.