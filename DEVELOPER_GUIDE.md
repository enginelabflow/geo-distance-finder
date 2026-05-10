# Developer Guide

This guide explains how `index.html` is organized and how each part of the app works. It is written for a junior developer who wants to understand or safely modify the project.

## Big Picture

Geo Distance Finder is a single-page browser app. Everything lives in `index.html`:

- HTML creates the page structure.
- CSS controls the visual design and responsive layout.
- JavaScript handles form submission, calls the location API, calculates distance, and updates the page.

There is no build step, package manager, framework, or backend server.

## HTML Structure

The page starts with a standard HTML document:

```html
<!DOCTYPE html>
<html lang="en">
```

The `lang="en"` attribute tells browsers and assistive technologies that the page content is in English.

### Head Section

The `<head>` contains metadata, the page title, and the app's CSS.

Important lines:

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Geo Distance Finder</title>
```

- `charset="UTF-8"` supports common text characters.
- `viewport` makes the layout scale correctly on mobile devices.
- `title` controls the browser tab title.

### Main Page Content

The visible app is inside:

```html
<main>
```

Using `<main>` is good semantic HTML because it tells the browser where the primary page content is.

### Header

```html
<header class="app-header">
  <h1>Geo Distance Finder</h1>
  <p class="subtitle">...</p>
</header>
```

This introduces the app. The heading is the main title, and the subtitle explains what the user can do.

### Search Form

```html
<section class="search-panel" aria-labelledby="search-title">
```

This section contains the location inputs and submit button.

Key elements:

- `#distanceForm`: the form JavaScript listens to.
- `#fromLocation`: the starting location input.
- `#toLocation`: the destination input.
- `#findButton`: the button that submits the form.
- `#message`: where loading, validation, and error messages appear.

The inputs use `autocomplete="off"` so the browser does not cover the form with saved suggestions.

### Results Section

```html
<section id="resultCard" class="result-card" aria-live="polite"></section>
```

This starts empty. JavaScript fills it after a successful distance calculation.

`aria-live="polite"` helps screen readers announce updates without interrupting the user.

### Raw JSON Section

```html
<section id="rawPanel" class="raw-panel">
  <details>
    <summary>Raw JSON responses</summary>
    <pre id="rawJson"></pre>
  </details>
</section>
```

This section shows the raw API data returned by Nominatim. It is hidden until a successful lookup happens.

`<details>` and `<summary>` create a built-in expandable panel without extra JavaScript.

## CSS Sections

The CSS is inside:

```html
<style>
```

### CSS Variables

```css
:root {
  --bg: #f6f8f7;
  --panel: #ffffff;
  --text: #1d2522;
}
```

The `:root` block defines reusable custom properties. These are like named design values. For example, `var(--text)` uses the text color.

This makes the design easier to change because common colors are defined in one place.

### Global Box Sizing

```css
* {
  box-sizing: border-box;
}
```

This makes element sizing more predictable. Padding and borders are included inside an element's declared width.

### Body Styling

```css
body {
  margin: 0;
  min-height: 100vh;
  font-family: Arial, Helvetica, sans-serif;
}
```

This removes default browser spacing, sets the page height, chooses the font, and applies the background.

### Layout Container

```css
main {
  width: min(980px, calc(100% - 32px));
  margin: 0 auto;
  padding: 42px 0;
}
```

This centers the app and keeps it from getting too wide on large screens. On smaller screens, it leaves space at the edges.

### Cards and Panels

```css
.search-panel,
.result-card,
.raw-panel {
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 8px;
}
```

The search form, result area, and raw JSON area share the same card styling.

### Form Grid

```css
.form-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
}
```

The two inputs sit side by side on larger screens.

### Responsive Styles

```css
@media (max-width: 720px) {
  .form-grid,
  .place-grid {
    grid-template-columns: 1fr;
  }
}
```

On screens 720px wide or smaller, the two-column layouts become one-column layouts. This makes the app usable on phones.

## JavaScript Overview

The JavaScript is inside:

```html
<script>
```

It does four main jobs:

1. Read user input from the form.
2. Fetch location data from Nominatim.
3. Calculate the distance between two coordinates.
4. Update the HTML with the result or an error message.

## DOM References

At the top of the script, the code stores references to important HTML elements:

```js
const form = document.querySelector("#distanceForm");
const fromInput = document.querySelector("#fromLocation");
const toInput = document.querySelector("#toLocation");
```

`document.querySelector()` finds the first matching element in the page.

Saving these elements in constants makes the rest of the code shorter and easier to read.

## Form Submit Handler

```js
form.addEventListener("submit", async function (event) {
```

This tells the browser: when the form is submitted, run this function.

The function is marked `async` because it uses `await` when calling the API.

### Preventing Page Reload

```js
event.preventDefault();
```

Normally, submitting a form reloads the page. This app does not want that, so `preventDefault()` stops the browser's normal form behavior.

### Reading Input Values

```js
const fromPlace = fromInput.value.trim();
const toPlace = toInput.value.trim();
```

`.value` reads what the user typed. `.trim()` removes extra spaces from the beginning and end.

### Validation

```js
if (!fromPlace || !toPlace) {
  showMessage("Please enter both a from location and a to location.", "error");
  return;
}
```

If either input is empty, the app shows an error and stops the function with `return`.

### Loading State

```js
setLoading(true);
showMessage("Looking up both places...", "info");
```

This disables the button, changes its text, and tells the user the app is working.

### Fetching Both Places

```js
const fromResponse = await searchPlace(fromPlace);
const toResponse = await searchPlace(toPlace);
```

Each call searches Nominatim for one location. `await` pauses until the API response is ready.

### Handling No Results

```js
if (fromResponse.length === 0) {
  showMessage("Sorry, the from location was not found...", "error");
  return;
}
```

Nominatim returns an array. If the array is empty, no matching place was found.

### Picking the Best Match

```js
const fromResult = fromResponse[0];
const toResult = toResponse[0];
```

The API URL uses `limit=1`, so the first item is the single best result.

### Calculating Distance

```js
const distance = calculateDistanceInKilometres(
  Number(fromResult.lat),
  Number(fromResult.lon),
  Number(toResult.lat),
  Number(toResult.lon)
);
```

The API returns latitude and longitude as strings. `Number()` converts them to numbers before doing maths.

### Displaying Output

```js
displayResult(fromResult, toResult, distance);
displayRawJson(fromResponse, toResponse);
hideMessage();
```

After a successful calculation, the app shows the result card, shows the raw API data, and hides the loading message.

### Error Handling

```js
catch (error) {
  showMessage("The location lookup failed...", "error");
}
```

If the API request fails, the user sees a friendly error instead of a broken page.

### Finally Block

```js
finally {
  setLoading(false);
}
```

The `finally` block always runs, whether the request succeeds or fails. It re-enables the button.

## `searchPlace(place)`

```js
async function searchPlace(place) {
```

This function sends a search request to Nominatim.

### Building the URL

```js
const apiUrl = "https://nominatim.openstreetmap.org/search?q="
  + encodeURIComponent(place)
  + "&format=jsonv2&limit=1";
```

`encodeURIComponent(place)` makes user input safe for a URL. For example, spaces become `%20`.

### Fetching Data

```js
const response = await fetch(apiUrl);
```

`fetch()` makes the HTTP request.

### Checking for HTTP Errors

```js
if (!response.ok) {
  throw new Error("Nominatim request failed");
}
```

If the response status is not successful, the function throws an error. That error is caught by the `catch` block in the submit handler.

### Returning JSON

```js
const data = await response.json();
return data;
```

This converts the response body from JSON text into JavaScript data.

## `calculateDistanceInKilometres(...)`

This function uses the Haversine formula to calculate straight-line distance between two points on Earth.

```js
const earthRadiusKm = 6371;
```

The app assumes Earth has a radius of 6371 kilometres.

The function converts latitude and longitude values from degrees to radians because JavaScript trigonometry functions use radians.

The final distance is:

```js
return earthRadiusKm * c;
```

This is an estimate of "as the crow flies" distance. It is not route distance.

## `degreesToRadians(degrees)`

```js
function degreesToRadians(degrees) {
  return degrees * (Math.PI / 180);
}
```

This helper converts degrees to radians. It keeps the distance function easier to read.

## `displayResult(fromResult, toResult, distance)`

This function builds the visible result card.

It:

- Converts coordinates to numbers.
- Builds OpenStreetMap links.
- Creates HTML for the distance and both locations.
- Adds the `is-visible` class so the hidden result card appears.

### Important Safety Detail

```js
escapeHtml(fromResult.display_name)
```

API data is inserted into `innerHTML`, so display names are passed through `escapeHtml()` first. This helps prevent unexpected HTML from being interpreted by the browser.

## `displayRawJson(fromResponse, toResponse)`

```js
rawJson.textContent = JSON.stringify({
  from: fromResponse,
  to: toResponse
}, null, 2);
```

This formats the raw API responses as readable JSON and puts them inside the `<pre>` element.

Using `textContent` is safer than `innerHTML` for raw JSON because it treats the content as text only.

## `createOpenStreetMapLink(lat, lon)`

```js
return `https://www.openstreetmap.org/?mlat=${lat}&mlon=${lon}#map=12/${lat}/${lon}`;
```

This creates a link that opens OpenStreetMap centered on the selected coordinates.

## Message Helpers

### `showMessage(text, type)`

```js
message.textContent = text;
message.className = `message is-${type}`;
```

This updates the message text and applies either an info or error class.

### `hideMessage()`

This clears the message and resets its class.

## Result Reset Helper

```js
function clearResults() {
```

This function clears old messages, result HTML, and raw JSON before a new search starts.

It also removes the `is-visible` class from hidden sections.

## Loading Helper

```js
function setLoading(isLoading) {
```

This disables or enables the submit button.

When loading:

- The button is disabled.
- The button text changes to `Calculating...`.

When finished:

- The button is enabled again.
- The button text returns to `Calculate distance`.

## `escapeHtml(value)`

```js
function escapeHtml(value) {
```

This helper replaces characters that have special meaning in HTML:

- `&`
- `<`
- `>`
- `"`
- `'`

This matters because the app displays place names from an external API.

## Safe Places to Modify

Good beginner-friendly changes:

- Change colors in the `:root` CSS variables.
- Update placeholder examples in the input fields.
- Change the `maximumFractionDigits` value to show more or fewer decimal places.
- Add miles by converting kilometres with `km * 0.621371`.
- Increase `limit=1` if you want to show multiple possible matches later.

## Things to Be Careful With

- Do not remove `encodeURIComponent()` from the API URL.
- Do not insert API values into `innerHTML` unless they are escaped first.
- Remember this app calculates straight-line distance, not travel distance.
- Nominatim is a public service with usage policies, so avoid sending unnecessary repeated requests.

