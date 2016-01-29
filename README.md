CSS Snap Heights
================

This document tries to simplify [CSS Line Grid].

The current draft has gone through a great simplification process
since its original proposal.
This document tries further simplification,
and see how well or not well such simplification can serve use cases.

For the background, benefits, or use cases of the feature,
please refer to [CSS Line Grid],
or [Examples] for how this feature helps web developers.

How common is this?

* [Search for "vertical rhythm css"](https://www.google.com/#q=vertical+rhythm+css)
hits 37K.
* [Compass](http://compass-style.org/), an open-source CSS authoring framework, has
[Vertical Rhythm](http://compass-style.org/reference/compass/typography/vertical_rhythm/)
functions.
* [typecast by Monotype](http://typecast.com/how-it-works) lists this feature
at 6th in the feature list.

To see a sample CSS,
click <q>Get the CSS</q> button in [Vertical Rhythm Tool](http://soqr.fr/vertical-rhythm/).

# Overview

This proposal tries to simplify the current [CSS Line Grid]
in the following 2 points:

1. Instead of snapping to the line grid,
it controls heights of lines and boxes
so that lines align to grids as a consequence.
2. Instead of defining grids,
it relies on [CSS Custom Properties]
for authors to assign a value to multiple boxes.

# Proposals

## 1. Snapping Heights: the `snap-height` property

### Snapping Line Boxes

```
Name: snap-height
Value: none | <length> <number>?
Initial: none
Applies to: block elements
Inherited: yes
```

When this property is set to `<length>`,
the used value of `line-height` of line boxes in the element
is rounded _up_ to the closest multiple of the `<length>`.

The half of the additional space is inserted before the line box,
and the rest after.

For block-level replaced elements,
the rounding applies to their logical heights of the margin-boxes
by increasing the used value of margin-before and margin-after equally.

Example
(the picture borrowed from
[monotype blog](http://typecast.com/blog/4-simple-steps-to-vertical-rhythm)):

```css
:root {
  --my-font-size: 12pt;
  --my-grid: 18pt;
  font-size: var(--my-font-size);
  snap-height: var(--my-grid);
}
h1 {
  font-size: calc(1.618 * var(--my-font-size));
  margin-before: calc(2 * var(--my-grid));
}
.side {
  font-size: calc(0.707 * var(--my-font-size));  
}
```

![](http://typecast.com/images/uploads/side-column-every-line.png)

### Snapping Baseline

When the optional `<number>` is specified,
and it is between 0 and 1,
it defines the baseline position within the line.
In this case,
the space distribution is changed as below:

* _space-over_ = (L + BP - (1em - BP)) / 2 - mod((LH + A - D) / 2, L)
* _space-under_ = (L - BP + (1em - BP)) / 2 - mod((LH - A + D) / 2, L)
* L: the specified `<lenght>`.
* BP: the specified `<number> * font-size`.
* LH: the line height of the element.
* A, D: Ascent/Descent of the primary font of the element.

**ISSUE**: Do we want to apply the same logic to block-level replaced elements?

Example:

```css
:root {
  snap-height: var(--my-grid) .8;
}
```

A sample picture from [Smashing Magazine](https://www.smashingmagazine.com/2012/12/css-baseline-the-good-the-bad-and-the-ugly/):

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2012/10/accurate-alignment.jpg)

### Snapping Block Boxes

For block-level non-replaced elements,
we can add another keyword
since snapping line boxes and snapping block-level elements are mutually exclusive.

```
Value: none | snap-block-keyword <length> <number>?
snap-block-keyword: ...
```

It's similar to line boxes, except:

* It snaps the logical height of the boxes, instead of the line heights.
* It uses _magin-before_ and _margin-after_ to adjust,
instead of _space-before_ and _space-after_.

The current [CSS Line Grid] defines following keywords.

<table>
<tr><th>value
<th>block-start
<th>(first) baseline
<th>block-end
<th>center
<th>last baseline
<tr><td>Use cases
<td colspan=2>Multi-line headings, block quotes, footnotes
<td>?
<td colspan=2>Multi-line headings
<tr><td>Adjust from computed values
<td>N/A
<td><i>margin-before</i>
<td colspan=3>N/A
<tr><td>Adjust after layout
<td colspan=2><i>margin-after</i>
<td><i>margin-before</i>
<td colspan=2><i>margin-before</i> and <i>margin-after</i>
<tr><td>Complexity guesstimate
<td colspan=2>Low
<td colspan=3>Higher: Change top after height was computed
</table>

## 2. Snap widths

```
Name: snap-width
Value: none | <length>
Initial: none
Applies to: block elements
Inherited: yes
```

When this property is set to `<length>`,
the logical width of the content-box is rounded _down_
to the closest multiple of the `<length>`
by increasing the used value of `margin-right`.

This feature improves justification for Han ideograph-based scripts,
and probably not applicable for other scripts.

# Naming

There was a feedback that this idea is not really a grid system that
we should not call it "line grid".
Some ideas:

* CSS Snap Lines
* CSS Vertical Rhythm
* CSS Line Rhythm
* Property names and value syntaxes should be brushed up.

# Examples

People are hacking CSS/SASS to implement vertical rhythms.
Here are some examples:

* [Vertical Rhythm in Compass](http://compass-style.org/reference/compass/typography/vertical_rhythm/)
* [Vertical Rhythm Tool](http://soqr.fr/vertical-rhythm/)
* [Aesthetic Sass 3: Typography and Vertical Rhythm](https://scotch.io/tutorials/aesthetic-sass-3-typography-and-vertical-rhythm)

[CSS Line Grid]: https://drafts.csswg.org/css-line-grid/
[CSS Custom Properties]: https://drafts.csswg.org/css-variables/
[heading example]: https://drafts.csswg.org/css-line-grid/#example-93bb7545
[Examples]: #examples
