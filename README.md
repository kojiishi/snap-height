CSS Snap Heights
================

This document tries to simplify [CSS Line Grid].

The current draft has gone through a great simplification process
since its original proposal.
This document tries further simplification,
and see how well or not well such simplification can serve use cases.

The basic idea of this simplification is
not to snap to grids, but instead
control heights of objects
so that objets aligns to vertical rhythms
as the result of stacking.

For the background, benefits, or use cases of the feature,
please refer to [CSS Line Grid].

# Proposals

## 1. The `snap-height` property

```
Name: snap-height
Value: none | line
Initial: none
Applies to: block elements
Inherited: yes
```

When this property is set to `line`,
the used value of `line-height` is rounded _up_ to the closest multiple of
the used value of `line-height` of the root element.

The half of the additional space is inserted before the line box,
and the rest after.

For block-level replaced elements,
the rounding applies to their logical heights of the margin-boxes
by increasing the used value of margin-before and margin-after equally.

**ISSUE**:
Adding half spaces to before/after in this case works good for Asian typography,
has similarity with CSS line height calculation,
but how does Latin typography adjust spaces in this case?

## 2. The `rlh` unit

The `rlh` unit is a relative length unit that computes to
the used value of `line-height` of the root element.

This unit is useful to add margins
while keeping the height in the multiple of the unit height.

## 3. Snap widths

```
Name: snap-width
Value: none | rem
Initial: none
Applies to: block elements
Inherited: yes
```

When this property is set to `rem`,
the logical width of content-box is rounded _down_ to the closest multiple of `1rem`
by increasing the used value of `margin-right`.

This feature is useful for Han ideograph-based scripts.

# Other considerations

## Multi-line headings

If we go only with proposed features above,
multi-line headings do not work well.
See [heading example] in the Background section of [CSS Line Grid].

I think we can live without features for multi-line headings in the _minimum_ level,
but this section tries to see how much easy or complicated it is.

### Workaround

First, let's see what authors could do without any additional features.
One possible workaround is to add a span:
```
<h1><span>long long long heading text</span></h1>
```
and then with CSS:
```
h1 > span {
  display: inline-block;
  snap-height: none;
}
```
So there's a way to look good, though not pretty.

### Rounding the block height

A possible additional feature is:
```
Name: snap-height
Value: none | line | margin-box
```

When the `snap-height` property is set to `margin-box`,
the rounding is applied to the logical heights of the margin-boxes,
instead of line heights.

The complex part of this value is to offset boxes after they were layout
because margin-before is determined by its logical height.
And since it's a block, it could be very long,
such as a table of 3000 rows.
It could contain absolute positioned objects using static positions.

We could add limitations without sacrificing use cases, such as:

* When the block is fragmented across columns, pages, etc.,
...
* When the block contains absolute positioned object,
...
* When the block height exceeds, say, 100vh,
...
* Does this work well with flexbox/grids? I'm not sure...

Is this valuable to consider further with such limitations,
or is it enough to let authors to workaround in the _minimum_ level spec?

## Borders

Borders are one of the things where this proposal behaves differently from [CSS Line Grid].
However, snapping the next line of a box with borders isn't likely what authors would want,
because borders are usually much thinner than the unit line height,
thus snapping to grid is likely to give more spaces than desired.

That said,
either in this proposal or in [CSS Line Grid],
authors need to give enough considerations if s/he wants to use borders
in vertically-rhythmed documents.

One possible solution is to define a border that does not affect layout,
but it's out of scope of this document.

## Margins/paddings

Authors have the tool to make it to work; the `rlh` unit.
It should be authors' responsibility to use it wisely.
[CSS Line Grid] can snap to grid even if authors use fractional margins,
but the result is not likely to help much anyway,
because it expands margins/paddings unexpectedly and uncontrollably.

The amount of margins/paddings matter a lot in visual designs.
We can trust authors to do it correctly
if s/he cares about vertical rhythms.

## Tables

Tables usually have cell padding,
or almost always have cell borders in East Asia.
Due to that,
it's unlikely that neither `snap-height` nor line grid can help much
for the same reasons as borders/margins/paddings.

We could add a UA stylesheet to avoid possible confusions:
```
table { snap-height: none; }
```
or `margin-box` (see "Multi-line headings" below.)

[CSS Line Grid]: https://drafts.csswg.org/css-line-grid/
[heading example]: https://drafts.csswg.org/css-line-grid/#example-93bb7545
