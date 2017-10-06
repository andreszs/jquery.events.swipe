# jquery.events.swipe
jQuery extension for horizontal swipe gestures detection in Android, iOS and Windows Phone with Pointer and Touch events.
Partially based on [detect_swipe](https://github.com/marcandre/detect_swipe).

This extension has been created because most other scripts can't handle both Pointer and Touch events efficiently across browsers, and the browser implementation is not consistent either.

## Events
```javascript
$('.selector').on('swipeleft',  function(){ /* Left swipe gesture */ });
$('.selector').on('swiperight', function(){ /* Right swipe gesture */ });
```
:warning: **Warning: Read the *Implementing the touch-action CSS property* section to add the required CSS attributes**
 
## Settings
*Note: Settings are hard-coded and must be edited manually in the JS file.*
```javascript
var thresholdTime = 500;
```
Specifies the maximum allowed time in ms to consider gesture as a valid swipe.
```javascript
var thresholdDistance = 64;
```
Specifies the minimum distance in px to consider gesture as a valid swipe.
```javascript
var debug = false;
```
Specifies that debug info must be shown in the console using `console.log()`. Useful to inspect relevant touch/pointer events.

## Compatibility
This script was specifically designed for using swipe events within [Apache Cordova](https://cordova.apache.org/) apps, but it will work in any website as well. It has been tested on:
- [x] Android 4.1, 4.2, 4.4 stock browsers
- [x] Android 5.1 with updated Android System WebView
- [x] Chrome 61
- [x] Firefox 55
- [x] Edge 38 on Windows Phone 10
- [x] Internet Explorer Mobile 11 on Windows Phone 8.1
- [x] Safari 11 on iOS 11

## Vertical Gestures
The script does not detect `swipeup` nor `swipedown` gestures, as they will inevitably interfere with regular vertical scrolling. It is intended mainly for mobile sites.

## Pointer Events vs. Touch Events
This script supports both the new [PointerEvent](https://developer.mozilla.org/en-US/docs/Web/API/PointerEvent) and the legacy [TouchEvent](https://developer.mozilla.org/en-US/docs/Web/API/TouchEvent) interfaces, prioritizing the former. Also supports the prefixed [MSPointerEvent](https://msdn.microsoft.com/en-us/library/hh772103.aspx) interface, exclusive to Internet Explorer 10.

## Pointer Event Remarks
Pointer Events, [proposed by Microsoft](https://www.w3.org/TR/pointerevents/), are designed to work in conjunction with the [touch-action](https://developer.mozilla.org/en-US/docs/Web/CSS/touch-action) CSS property.

> The touch-action CSS property specifies whether, and in what ways, a given region can be manipulated by the user via a touchscreen (for instance, by panning or zooming features built into the browser).

### Possible touch-action CSS values and its effects
1. Object with `touch-action` `unset` or `auto`:
   - Will immediately trigger the `pointercancel` event before our swipe is detected, spoiling the gesture. (BAD)
   - As soon as `pointercancel` is triggered, the browser takes control of the gesture to perform regular pan actions.
   - Not even `e.PreventDefault()` will be enough to prevent `pointercancel` from firing.
2. Object with `touch-action: none`:
   - Prevents the `pointercancel` event and allow us to correctly detect the horizontal swipe gesture. (GOOD)
   - Disables browser handling of all panning / scrolling and zooming gestures. (BAD)
3. Object with `touch-action: pan-y`:
   - Prevents the `pointercancel` event and allow us to correctly detect the horizontal swipe gesture. (GOOD)
   - Allows vertical scroll normally. (GOOD)


### Implementing the touch-action CSS property
In order to detect our horizontal gestures, ensure the object with the `swiperight` or `swipeleft` event has the `touch-action` set specifically to `pan-y`:

```javascript
/* Attach event listener to our div.gestures element */
$('div.gestures').on('swipeleft',  function(){ /* Left swipe gesture */ });
```
```css
/* this div alone will detect H-gestures and allow V-scrolling */
div.gestures {
    touch-action: pan-y;
}
```

:warning: **Warning**: Any child elements inside `div.gestures` also need to have the `touch-action` property set, otherwise the browser could fire `pointercancel` too soon to detect the swipe event. Using the asterisk selector is a solution for this:

```css
/* this div and its child elements will detect H-gestures and allow V-scrolling */
div.gestures,
div.gestures * {
    touch-action: pan-y;
}
```

## Touch Event Remarks
The legacy Touch Events interface was implemented before the [touch-action](http://caniuse.com/#feat=css-touch-action) CSS property, therefore it is handled inconsistently among browsers. Some browser could fire `touchcancel` (spoiling the swipe gesture) while others may never fire it. This script considers both scenarios and assumes that `touchcancel` may be fired too soon. To prevent it, the **script will first check the first few pixels swiped as follows**:

1. An initial 16 px horizontal swipe will use `e.preventDefault()` to keep checking for a valid H-swipe event
   - As soon as an horizontal gesture reaches `thresholdDistance` in pixels, `swipeleft` or `swiperight` is fired.
   - Gestures slower than `thresholdTime` milliseconds will be ignored (default 500 ms, half a second)
2. An initial 8 px vertical swipe will ignore our swipe listeners and delegate the gesture to the browser (for V-scrolling).

## Dealing with undesired pointercancel / touchcancel events

> The pointercancel event is fired when the browser determines that there are unlikely to be any more pointer events, or if after the pointerdown event is fired, the pointer is then used to manipulate the viewport by panning, zooming, or scrolling.

> The touchcancel event is fired when a touch point has been disrupted in an implementation-specific manner (for example, too many touch points are created).

The [pointercancel](https://developer.mozilla.org/en-US/docs/Web/Events/pointercancel) event can be prevented using the `touch-action` CSS property as explained earlier, while [touchcancel](https://developer.mozilla.org/en-US/docs/Web/Events/touchcancel) cannot be stopped in pre-`touch-action` browsers (Android 4, Chrome < 55). For this reason the script will listen to the first few pixels swiped before `pointercancel` / `touchcancel` is fired as shown in the previous section and fire `e.preventDefault` only when a potential horizontal swipe is detected.

This behaviour is designed to reduce the interference of the event listeners with regular scrolling and panning actions, and it works perfectly in conjunction with the proper `touch-action` value in both modern and older browsers.
