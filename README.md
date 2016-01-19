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

# Overview

The basic idea of this simplification is
not to snap to grids nor to define grids, but instead
control heights of objects
so that objets align to vertical rhythms
as the result of stacking.

How common is this?

* [Search for "vertical rhythm css"](https://www.google.com/#q=vertical+rhythm+css)
hits 37K.
* [Compass](http://compass-style.org/), an open-source CSS authoring framework, has
[Vertical Rhythm](http://compass-style.org/reference/compass/typography/vertical_rhythm/)
functions.
* [typecast by Monotype](http://typecast.com/how-it-works) lists this feature
at 6th in the feature list.

# Base Proposals

## 1. The `snap-height` property

```
Name: snap-height
Value: none | line
Initial: none
Applies to: block elements
Inherited: yes
```

When this property is set to `line`,
the used value of `line-height` of line boxes in the element
is rounded _up_ to the closest multiple of
the used value of `line-height` of the root element.

The half of the additional space is inserted before the line box,
and the rest after.

For block-level replaced elements,
the rounding applies to their logical heights of the margin-boxes
by increasing the used value of margin-before and margin-after equally.

Also see [snapping baselines](#baseline) below.

## 2. The `rlh` unit

The `rlh` unit is a relative length unit that computes to
the line height of the root element.

In this model,
it is authors' responsibility to keep the sum of margins/paddings
to the multiple of the rhythm unit.
This unit helps authors doing so.

For example:
```css
h1 { margin-before: 2rlh; }
```
can produce layout like the picture below
(from [monotype blog](http://typecast.com/blog/4-simple-steps-to-vertical-rhythm)):

![](http://typecast.com/images/uploads/side-column-every-line.png)

If author wants more spaces in before than in after:
```css
h2 { margin-before: .8rlh; margin-after: .2rlh; }
```
because the sum of margins is multiple of `rlh`,
text after the heading are back on the vertical rhythm.

See [Examples] for other examples on the web.

## 3. Snap widths

```
Name: snap-width
Value: none | content-to-rem
Initial: none
Applies to: block elements
Inherited: yes
```

When this property is set to `content-to-rem`,
the logical width of the content-box is rounded _down_ to the closest multiple of `1rem`
by increasing the used value of `margin-right`.

This feature improves justification for Han ideograph-based scripts,
and probably not applicable for other scripts.

# Other considerations

Here are a list of things that are more complicated than the base proposals
and there are some open issues in the requirements/designs,
but they are considered to have some certain needs.

## Interruptions

From a [monotype blog](http://typecast.com/blog/4-simple-steps-to-vertical-rhythm):

> As Robert Bringhurst explains in The Elements of Typographic Style, interruptions like this are ok as long as we resume the rhythm afterwards:

>> _“Headings, subheads, block quotations, footnotes, illustrations, captions and other intrusions into the text create syncopations and variations against the base rhythm of regularly [spaced] lines. These variations can and should add life to the page, but the main text should also return after each variation precisely on beat and in phase.”_

As long as these interruptions are single line,
or every line of such interruptions should also align to grids,
the base proposal supports the requirement
even if font sizes are different.

However, if such interruptions flow multiple lines,
and authors want each line of the interruptions not to align to grids,
the base proposal cannot return to the rhythm.

Such interruptions can be divided to two different requirements:

1. Top-aligned block, adjust by margin-after.
2. Center-alinged block,
adjust by both margin-before and margin-after.

### Adjust after-space

```css
snap-height: margin-after;
```
This value increases the used value of `margin-after`
so that the logical height of the margin-box becomes
the multiple of the line height of the root element.

This process runs after the margin collapsing is completed.

If the block is fragmented across column, pages, etc.,
it applies to the last fragment box.
**ISSUE**: to all? does it cause any differences?

### Adjust both before and after-space

```css
snap-height: margin-box;
```
With this CSS,
the rounding is applied to the logical heights of the margin-boxes,
instead of line heights.
The additional space is distributed to before and after equally.

IIUC this is harder,
because we need to increase `margin-before` after layout is done.
Doing so can change the logical top of the border-box after the layout.
Need experiments to know how much troublesome it is.

We could add limitations without sacrificing use cases, such as:

* When the block is fragmented across columns, pages, etc.,
fallback to `margin-after`?
* When the block contains absolute positioned object,
fallback to `margin-after`?
* When the block height exceeds, say, 100vh,
fallback to `margin-after`?
* Does this work well with flexbox/grids? I'm not sure...

**ISSUE**: how `baseline` scenario fits in this case?

#### If without `margin-box`...

One possible workaround authors can do without `margin-box` value
is to add a span:
```html
<h1><span>long long long heading text</span></h1>
```
and then with CSS:
```css
h1 > span {
  display: inline-block;
  snap-height: none;
}
```
This should give the same result as `margin-box`,
though not pretty.

## Baseline

The current [CSS Line Grid] spec defines a mode
to snap baselines to other baselines.
This can be achieved in this model by:
```css
snap-height: baseline
```
When this property is set to `baseline`,
the additional spaces are distributed as below:

* _space-over_ = (RLH + RA - RD) / 2 - mod((LH + A - D) / 2, RLH)
* _space-under_ = (RLH - RA + RD) / 2 - mod((LH - A + D) / 2, RLH)
* RLH: the used line height of the root element.
* RA, RD: Ascent/Descent of the primary font of the root element.
* LH: the line height of the element.
* A, D: Ascent/Descent of the primary font of the element.

**TODO**: the expressions above need reviews.

This will result in:

* The line height is multiple of RLH.
* The baseline matches to the baseline of the root element.

A sample picture from [Smashing Magazine](https://www.smashingmagazine.com/2012/12/css-baseline-the-good-the-bad-and-the-ugly/):

![](https://media-mediatemple.netdna-ssl.com/wp-content/uploads/2012/10/accurate-alignment.jpg)

## Baseline + after-space (First Baseline)

In Latin, one may want to combine after-space
with baseline of the first line.
```css
snap-height: first-baseline-and-margin-after
```
When the property is set to this value,
the _space-over_ above is added to margin-before
before computing margin-after.

## Every 5th Line

The [monotype blog](http://typecast.com/blog/4-simple-steps-to-vertical-rhythm)
above later makes the sidenote to align every 5th line:

![](http://typecast.com/images/uploads/side-column-incremental-leading.png)

This can be simulated closely with the base proposals as:
```css
.sidenote {
  snap-height: none;
  line-height: .8rlh;
}
```

## Summary

<table>
<tr><th>Type
<th>center-line
<th>baseline
<th>top-align block
<th>first baseline
<th>center-align block
<th>last baseline
<tr><td>Adjust
<td>space-over and space-under, equally
<td>space-over and space-under, unequally
<td>margin-after
<td colspan=3>margin-before and margin-after
<tr><td>Use cases
<td>General
<td>General (Latin)
<td colspan=2>Multi-line headings, block quotes, footnotes
<td colspan=2>Multi-line headings
<tr><td>Complexity guesstimate
<td>Low
<td>Unequal line spaces?
<td colspan=2>Low?
<td colspan=2>Change top after layout is unknown
</table>

## Naming

There was a feedback that this idea is not really a grid system that
we should not call it "line grid".

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
[heading example]: https://drafts.csswg.org/css-line-grid/#example-93bb7545
[Examples]: #examples
