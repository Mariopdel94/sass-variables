# Sass map variables

## Using Sass functions to access variable maps
The basic idea is that you can store groups of Sass variables in a JSON-ish map:

Example:

```scss
$colors: (
  primary: #FFBB00,
  secondary: #0969A2
);
```

These can then be accessed using the map-get function:

SCSS:
```scss
// using `map-get` in Sass
h1 { color: map-get($colors, primary); }
```

CSS output:
```css
h1 { color: #FFBB00; }
```

### Inception
Now maps, can be nested for added specificity and better organization:

```scss
// color variable map
$colors: (
  // nested map inception
  primary: (
    base: #FFBB00,
    light: lighten(#FFBB00, 15%),
    dark: darken(#FFBB00, 15%),
    trans: transparentize(#FFBB00, 0.5)
  ),
  // and another. is the totem still spinning?
  secondary: (
    base: #0969A2,
    light: lighten(#0969A2, 15%),
    dark: darken(#0969A2, 15%),
    trans: transparentize(#0969A2, 0.5)
  )
);
```

To access a nested map, you simply nest the `map-get`:

```scss
h1 {
  color: map-get(map-get($colors, primary), base);
}
```

The inner `map-get` grabs our `primary` map, then the last value `base` grabs that `primary` map’s `base` value.

### Functions
Functionally, nested maps are awesome. Unfortunately the syntax is a bit...meh. We can fix that nonsense with a `@function`:

```scss
// retrieve color from $colors map ie. `color(primary, base)`
@function color($color-name, $color-variant) {
  // map inception
  @return map-get(map-get($colors, $color-name), $color-variant);
}

// now we use the function
h1 {
  color: color(primary, base);
}
```

We can even take it a step further. Let’s say we want to have nested maps and non-nested values in our map:

```scss
// color variable map
$colors: (
  // non-nested values
  text: #FFF,
  background: #333,
  // nested map inception
  primary: (
    base: #FFBB00,
    light: lighten(#FFBB00, 15%),
    dark: darken(#FFBB00, 15%),
    trans: transparentize(#FFBB00, 0.5)
  ),
  secondary: (
    base: #0969A2,
    light: lighten(#0969A2, 15%),
    dark: darken(#0969A2, 15%),
    trans: transparentize(#0969A2, 0.5)
  )
);
```

We can then update our function to *optionally* receive the nested variant name:

```scss
// retrieve color from $colors map ie. `color(base, primary)`
@function color($color-name, $color-variant:null) {
  // color variant is optional
  @if ($color-variant != null) {
    // map inception
    @return map-get(map-get($colors, $color-name), $color-variant);
  } @else {
    @return map-get($colors, $color-name);
  }
}
```

And use it like this:

```scss
// using the function to get an non-map color
body {
  background-color: color(background);
}
// using the function to get a nested map color
h1 {
  color: color(primary, base);
}
```

### Redundancy
Some of you may have noticed and not enjoyed the redundency in our variable maps:

```scss
// color variable map
$colors: (
  primary: (
    base: #FFBB00,
    light: lighten(#FFBB00, 15%),
    dark: darken(#FFBB00, 15%),
    trans: transparentize(#FFBB00, 0.5)
  ),
  // ...
);
```

Notice how the base color `#FFBB00` is repeated in hex form across all the values in the `darken`, `lighten`, and `transparentize` functions. Unfortunately, we cannot refer to a sibling value in the other values (ie `@this.base`), so the hex must be repeated. The workaround? Simply extract the hex into a variable outside of the map.

```scss
// base color defs
$color-primary: #FFBB00;
// color variable map
$colors: (
  primary: (
    base: $color-primary,
    light: lighten($color-primary, 15%),
    dark: darken($color-primary, 15%),
    trans: transparentize($color-primary, 0.5)
  ),
  // ...
);
```

We can afford to be more verbose with our variable definition `$color-primary` because we don’t have to write it everywhere, just in our map definitions. Then, if we want to change our color scheme around, we can swap a single value instead of four values. Doing it this way is more a matter of preference, but I enjoy making the map completely color-agnostic by manipulating a color variable instead of hardcoding it.

If you want to be an ultra-streamline magic person, you can store your `darken`, `lighten`, and `transparentize` amounts and use them across the board for each of your colors:

```scss
// base color defs
$color-primary: #FFBB00;
$color-secondary: #0969A2;
$color-shade-amount: 15%;
$color-trans-amount: 0.5;
// color variable map
$colors: (
  primary: (
    base: $color-primary,
    light: lighten($color-primary, $color-shade-amount),
    dark: darken($color-primary, $color-shade-amount),
    trans: transparentize($color-primary, $color-trans-amount)
  ),
  secondary: (
    base: $color-secondary,
    light: lighten($color-secondary, $color-shade-amount),
    dark: darken($color-secondary, $color-shade-amount),
    trans: transparentize($color-secondary, $color-trans-amount)
  )
);
```

### Creating iterables
We even can take this a step further and create a map of the color declarations and do an `@each` to iterate on each color of the map and create the `color variants` there.

```scss
/* Variables */
$color-map: (
  primary: #6d61cb,
  secondary: #1c105f,
  warning: #fc6485,
  accent: #39e2d7,
  neutral: #1a1a1a,
  gray: #a4a4a4,
);

$color-shade-amount: 12%;
$color-trans-amount: 0.5;

$colors: ();

@each $code, $color in $color-map {
  $color-variants: (
    $code: (
      base: $color,
      light: lighten($color, $color-shade-amount),
      dark: darken($color, $color-shade-amount),
      trans: transparentize($color, $color-trans-amount)
    )
  );
  $colors: map-merge($colors, $color-variants);
}
```

If you want to add colors without the variants add them later of the `@each`

```scss
/* This colors will NOT have a base, light, dark, and trans variants */
$obligatory-colors: (
  light: #ffffff,
  dark: #000000,
);

$colors: map-merge($colors, $obligatory-colors);
```

### Generate common used classes
Since we have all colors with the variants now we can create commonly used classes for each of them, for example `color-primary-base` or `background-secondary-light`, etc.

Same as before, we can iterate on the map we have:

```scss
/* Generate all the color classes (see css file output) */
@each $code, $color in $colors {
  @if type-of($color) == 'map' {
    @each $colorVariant, $colorCode in $color {
      @if ($colorVariant == 'base') {
        .color-#{$code} { color: map-get($color, $colorVariant); }
        .bg-#{$code} { background-color: map-get($color, $colorVariant); }
      } @else {
        .color-#{$code}-#{$colorVariant} { color: map-get($color, $colorVariant); }
        .bg-#{$code}-#{$colorVariant} { background-color: map-get($color, $colorVariant); }
      }
    }
  } @else {
    .color-#{$code} { color: $color; }
    .bg-#{$code} { background-color: $color; }
  }
}
```

And this will generate:

```css
.color-primary {
  color: #6d61cb;
}

.bg-primary {
  background-color: #6d61cb;
}

.color-primary-light {
  color: #988fda;
}

.bg-primary-light {
  background-color: #988fda;
}

[etc...]
```