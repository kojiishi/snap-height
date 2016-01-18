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

How common is this idea?

* [Search for "vertical rhythm css"](https://www.google.com/#q=vertical+rhythm+css)
hits 37K.
* [Compass](http://compass-style.org/), an open-source CSS authoring framework, has
[Vertical Rhythm](http://compass-style.org/reference/compass/typography/vertical_rhythm/)
functions.

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
the used value of `line-height` of the descendants
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
it is authors' repsonsibility to keep the sum of margins/paddings
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

From a [monotype blob](http://typecast.com/blog/4-simple-steps-to-vertical-rhythm):

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

\\| Top-align (adjust margin-after) | Center-align (adjust both margins)
----|----|----
Use cases|Multi-line headings in multi-columns, block quotes, notes|Multi-line headings
Impl|Easy|Offset changes after layout can be complicated?

### Adjust after-space

IIUC this isn't hard to implement.
For instance, when:
```css
snap-height: margin-after;
```
increase the used value of `margin-after`
so that the logical height of the margin-box becomes
the multiple of the line height of the root element.

This process runs after the margin collapsing is competed.

### Adjust both before and after-space

IIUC this is harder,
because we need to increase `margin-before` after layout is done.
Doing so can change the logical top of the border-box after the layout.

For instance, when:
```css
snap-height: margin-box;
```
the rounding is applied to the logical heights of the margin-boxes,
instead of line heights.
The additional space is distributed to before and after equally.

We could add limitations without sacrificing use cases, such as:

* When the block is fragmented across columns, pages, etc.,
fallback to `margin-after`?
* When the block contains absolute positioned object,
fallback to `margin-after`?
* When the block height exceeds, say, 100vh,
fallback to `margin-after`?
* Does this work well with flexbox/grids? I'm not sure...

**ISSUE**: how `baseline` scenario fits in this case?

### Workaround

Without using `margin-box`,
one possible workaround authors can do is to add a span:
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
```
Name: snap-height
Value: none | line | baseline
```
When this property is set to `baseline`,
the additional spaces are distributed as below:

* _space-over_ = RLH - mod(A - RA, RLH)
* _space-under_ = RLH - mod(D - RD, RLH)
* RLH: the used line height of the root element.
* A, D: Ascent/Descent of the primary font of the element.
* RA, RD: Ascent/Descent of the primary font of the root element.

This will result in:

* The line height is multiple of RLH.
* The multiple is always an odd number.
* The baseline matches to the baseline of the root element
(except when fallback fonts are used.)

**ISSUE**:
This is likely to cause a ragged-top designs
unless all top heading have the same font sizes.
Is such result expected?

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
