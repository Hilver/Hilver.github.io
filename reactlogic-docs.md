---
layout: page
title: reACTlogic docs
permalink: /reactlogic-docs/
---
## Installation

```
npm install reactlogic
```

## API

### useCheckbox

Returns boolean value from `checkbox` input.

### useInput

Returns provided value as string.

{% include embedIframe.html src="https://codesandbox.io/embed/reactlogic-useinput-1h9wv" %}

### **useRect(refEl?)**

#### RefEl

**Type: `RefObject<HTMLElement>`**

A React ref element of which rect is counted. Properties describing the overall border-box in pixels. Properties other than `width` and `height` are relative to the top-left o the viewport. [More info here.](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)

{% include embedIframe.html src="https://codesandbox.io/embed/keen-colden-kj24k" %}

### useScroll({element?, debounce?, delayTime?, targetElement?})

#### element

**Type: `RefObject<HTMLElement>`**

**Default: `window`**

A React ref element of which scroll event will be measured.

#### debounce

**Type: `Boolean`**

If `true` then scroll event will use `debounce` function as delay, otherwise `throttle` function will be set. If this option is ommited, `throttle` is set as default.

#### delayTime

**Type: `number`**

**Default: `0 ms`**

A number of milliseconds for delay function.

{% include embedIframe.html src="https://codesandbox.io/embed/reactlogic-usescroll-nhs4t" %}

### useSearch({data, search, type?, caseSensitive?})

#### data

**Type: `Array<string | number | object>`**

An `array` of data that will be filtered for a match.

#### search

**Type: `string | number`**

A `string` or `number` to be searched for.

#### type

**Type: `string<keyof data>`**

When input data is an `array` of `objects`, you have to specify which property include to search for. It should be `string` which is key of specified data objects.

#### caseSensitive

**Type: `boolean`**
**Default: `true`**

An addition property which distinguish queries by case sensitive.

{% include embedIframe.html src="https://codesandbox.io/embed/reactlogic-usesearch-nnr83" %}

### useSlider(indexLimit, changeSpeed?)

#### indexLimit

**Type: `number`**

A number of slides.

#### changeSpeed

**Type: `number`**

**Default: `2000`**

A number of miliseconds between each slide change.

{% include embedIframe.html src="https://codesandbox.io/embed/reactlogic-useslider-3e78j" %}
