# Akkio Frontend Technical Assessment

## Requirements

- Use [The Color API](https://www.thecolorapi.com/) to build a grid of HSL color swatches that take in user inputs for `saturation` and `lightness`.
- Display the visualized color, color name, and RGB values
- Design for performance and UX

## Summary

This was my first project using Vue, and honestly I had a lot of fun exploring the framework. I used the [Vue Quickstart](https://vuejs.org/guide/quick-start) guide to spin up the application, and spent the first few hours getting my bearings and the basic functionality down (see the ugly but mostly working [initial commit](https://github.com/adlondon/color-swatch/commit/fd50cdfda8f55ff44b53f6c74156898631d68b11)).

At first, I thought that referencing `exact_match_name` would be the best way to filter out duplicate color names. However I soon realized that not all of the queries produced a response that contained that value, so I refactored it to check for duplicates in state. I also added a caching layer that lives in memory and prevents unnecessary requests. The initial queries make all 360 requests, but repeated requests are pulled from the cache. I also toyed around with the idea of debouncing the `@change` callback from the `input`, but found that the UX was not ideal so I ended up adding a submit button.

Error handling is pretty light, with simple checks for character count in the inputs and a basic message to the user. Would have liked to focus more time on that and tightening up the input validation and UX for entering saturation and lightness values in general.

## Setup

```sh
npm install
```

### Compile and Hot-Reload for Development

```sh
npm run dev
```
