# Navigation System Spec

This prototype uses one shared navigation flyout system. The nav items do not each own separate panels; instead, a single fixed-position shell is moved, resized, and given the active section content.

## Desktop Navigation

The desktop header is a fixed layer at the top of the viewport with a logo on the left and a white pill on the right. The pill contains the current location, main navigation links, profile action, and the booking button. The location, Stay, Work, Meeting & Events, Membership, Help, and Profile controls open flyouts. Eat & Drink, About us, and Book now currently do not open flyouts.

The nav pill uses a 42px interaction row. Text labels are 14px with 20px line-height. The location label uses the same text scale on desktop and a smaller 12px label in mobile layouts. Nav text hit areas are expanded by invisible pseudo-elements so the clickable/hoverable area fills the nav row without changing visual spacing.

On desktop hover, inactive nav items dim to `#7f7c78` while the hovered or active trigger keeps the primary text color. The active class belongs to the trigger that owns the currently visible flyout.

## Mobile Navigation

At widths below 768px, and in the simulated `body.mobile-mode`, the pill contains only the location selector and hamburger button. Regular nav links, profile, and booking controls are hidden. The current code displays the hamburger button but does not attach a mobile menu flyout to it.

The simulated mobile viewport is an optional preview wrapper controlled by the dev tools panel. It clips the page to an iPhone-like frame and shifts the nav padding to account for the status area.

## Flyout Shell

All flyouts render inside `#flyout-panel`. The shell owns:

- fixed positioning
- white background
- 24px radius
- 12px padding
- shared elevation shadow
- viewport clamping
- pointer-event state

The shell contains `.flyout-clip`, an inner layer that handles clipping and content opacity. This keeps the shell shadow visible while the content reveal clips downward.

The panel appears 8px below the bottom of `.nav-pill`. The shell is clamped to a 24px viewport margin. The preferred horizontal position aligns the active nav label or icon to the first meaningful label/icon inside the flyout. If that preferred alignment would overflow the viewport, the panel aligns to the nav pill's right edge and then clamps.

## Flyout Sizes

There are three size modes:

- `flyout-wide`: one list column plus one square featured card
- `flyout-xlwide`: two location columns plus one square featured card
- `flyout-narrow`: one compact list column only

The featured card size is derived from list rows:

- featured rows: 5
- wide row height: 62px
- list gap: 0
- featured size: `5 * 62px = 310px`

Wide and extra-wide panels both use:

`panel padding left/right + column gap + featured size * 2`

With the current values this is:

`24px + 12px + 310px + 310px = 656px`

For the location flyout, the two location columns together occupy the same width as a normal wide list column:

`(310px - 12px) / 2 = 149px` per column.

The narrow flyouts cap at 320px.

## Open, Switch, And Close Motion

Opening from closed:

1. The active section is inserted into the shared shell.
2. The shell is positioned and shown.
3. `.flyout-clip` reveals downward by animating `clip-path`.
4. Items animate upward from a 50px offset.
5. Items use a 600ms duration, 110ms stagger, and 200ms open delay.
6. The featured card uses the same enter motion, delayed 50ms after the first item.

Switching while open:

1. The current content fades out.
2. The shell moves and resizes to the next trigger and size mode in the same animation frame.
3. If the height or width changes, content stays hidden until resize is complete.
4. New content dissolves in over the item switch duration.
5. The shell uses a 480ms switch motion.

The shell must never finish its width/height morph before sliding to its final horizontal position. On every nav switch, the target `left` and target `width` are calculated together and animated together.

Closing:

1. Leaving the trigger, shell, and bridge starts a 300ms grace timer.
2. If the pointer re-enters the hover zone, the close is cancelled.
3. Otherwise the shell fades out while `.flyout-clip` clips upward.
4. After the 600ms flyout duration, active classes, inline dimensions, animations, and highlights are cleared.

## Hover Bridge

`#flyout-bridge` is an invisible fixed element between the nav pill and the flyout shell. It uses the full nav pill width so users can move diagonally between nav labels and flyout content without accidentally closing the menu.

## Sliding Highlights

Each `.flyout-list` and location column gets one `.flyout-highlight` element. The highlight is absolutely positioned behind the items and moved with `transform: translateY(...)`, so hover transitions feel like one sliding pill instead of separate backgrounds.

Highlight colors:

- themed wide flyouts use the section accent color
- location country and city columns use `#1a1510`
- non-themed narrow flyouts use `#f3f2f0`

Selected location rows keep their gray selected background. Hovering a selected location row does not show the dark sliding highlight.

The highlight is temporarily locked during open/switch/item-enter motion to avoid accidental hover states from layout movement.

## Location Flyout

The location flyout has country, city, and featured columns. Countries are static in the HTML. Cities are generated from `locationData`.

Country behavior:

- Hovering a country previews its city list.
- Clicking a country updates the selected country.
- The selected country is displayed with a gray background.

City behavior:

- The first city in the current city list is selected.
- Hovering city rows crossfades the featured image to one of the room images.
- Leaving the city column restores the default featured image.

Location list height is measured from row counts rather than `scrollHeight`, which keeps height animation deterministic while the panel is also resizing.

## Preview/Dev Controls

The dev tools panel toggles blog mode, mobile preview, and pastel duotone blog thumbnails. Four clicks on the Preview trigger toggle header visibility. URL params persist preview state:

- `blog=1|0`
- `mobile=1|0`
- `pastel=1|0`
- `header=1|0`

The default body classes currently start in blog mode with the header hidden. Use `?header=1` or the dev tools unlock behavior to inspect the navigation directly.
