#  isEventSupports

Author: Yang Cao

Last Update: April 12, 2021



## Contents

* [Motivation](#motivation)
* [Proposed Solution](#proposed-solution)
* [Feature Detection and Polyfilling](#feature-detection-and-polyfilling)
* [Helpful links](helpful-links)

## Motivation

In the process of developing an interactive Web application, developers often need to listen to a specific event through addEventListener to perform specific operations. In the Web standard, all elements are inherited from eventTarget, and developers can listen normally no matter what events they listen to. However, the developer intended to monitor a dispatch from the host environment, such as click or dblclick, but the host environment has no related functions, which will lead to lack of functionality, and at this time the developer cannot make a good judgment at the application layer. Due to the differences between various browsers and the compatibility of some historical versions of the browsers, developers have to add some browser environment or version judgments to the code to complete the coding of related functions. Therefore, we hope to provide a way to perceive whether the current node provides a certain event capability through "feature detection" under the Web standard.



for example:

Take [dblclick](https://developer.mozilla.org/en-US/docs/Web/API/Element/dblclick_event) as an example. On the mobile terminal, browsers such as safari and firefox support dblclick, but chrome and android WebView do not support it. At this time, if there is no "feature detection", it needs to be coded as follows.



```js
if (/Chrome/.test(window.navigator.userAgent)) {
  // Realize by other means, such as judging 2 clicks
} else {
  element.addEventListener('dblclick', (e)=>{
    //...
  });
}
```



In the actual environment, developers need to do more than just judge the browser type. There are also differences in the versions and compatibility of various browsers, which makes environmental judgment extremely complicated.



```js
if ((/Safari/.test(window.navigator.userAgent && version <= 3) || (/Chrome/.test(window.navigator.userAgent))) {
  // Realize by other means, such as judging 2 clicks
} else {
  element.addEventListener('dblclick', (e)=>{
    //...
  });
}
```



When we need compatible Web platforms and many versions, it is difficult to maintain this judgment logic.

## Proposed Solution

The proposed solution is to extend the `isEventSupports` method on [EventTarget](https://dom.spec.whatwg.org/#interface-eventtarget) and provide the upper layer to query whether the class inheriting `EventTarget` supports an event.



```javascript
interface EventTarget {
  // ...
  boolean isEventSupports(DOMString eventType);
}
```

For example, when the developer needs to query whether the `click` event can be listened to on the `div`, to ensure that the browser will throw the corresponding `click` event when the user clicks the div. Developers only need to use the following methods to query whether a certain element in the browser supports a specific event.



```javascript
const element = document.createElement('div');
const isSuportClick = element.isEventSupports('click');
```



Or check whether the window supports the `load` event.



```javascript
window.isEventSupports('load');
```



## Other Considerations

It should be noted that in the DOM0 standard, developers can use the following methods to detect whether an event can be monitored.

```javascript
const element = document.createElement('div');
const isSupportClick = 'onclick' in element;
```

However, the DOM0 and DOM2 standards do not require one-to-one correspondence. Not every event can be pre-on to check whether it can be monitored. We expect a method directly on EventTarget to detect whether an event is supported on a specific object.



In addition, some events are bound to unique elements. Take the following input event as an example.



```javascript
const div = document.createElement('div');
const isSupportInput = 'oninput' in div;
```



At this time, input will return ture normally, but in fact there will be no input event on the div. Then, we expect div.isEventSupports('input') to return a false.

## Feature Detection and Polyfilling

To detect support for isEventSupports, something like this could be used:



```javascript
function supportsIsEventSupports() {
  return EventTarget.prototype.isEventSupports;
}
```



## Helpful links

Discussion: [w3c chinese-ig issue 241](https://github.com/w3c/chinese-ig/issues/241)

