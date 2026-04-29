# Primitives Collection — Detailed Setup Guide

## Size Scale

A non-linear scale that covers all common UI needs. The scale is denser at small values (where 2px differences matter) and sparser at large values.

```
Name          Value    Common Uses
Size/2        2px      Hairline borders, minimal gaps, radius-xs
Size/4        4px      Small gaps, small padding, radius-s
Size/6        6px      XXS padding, compact list spacing
Size/8        8px      XS padding, small component gaps
Size/10       10px     XXS typography, letter spacing baseline
Size/12       12px     S padding, body text size (small), gaps
Size/14       14px     S-M body text, button text (small)
Size/16       16px     Standard body text, MS padding
Size/20       20px     MS heading, M padding, icon containers
Size/24       24px     M heading, ML padding/gaps
Size/28       28px     ML heading, ML padding (desktop)
Size/32       32px     ML heading (large), L radius
Size/36       36px     L heading, line heights
Size/40       40px     L heading (large), touch targets
Size/42       42px     XL radius, special sizing
Size/48       48px     XL heading, large padding
Size/52       52px     XL heading (desktop), L padding
Size/56       56px     Line heights (large)
Size/60       60px     XXL heading, XL padding
Size/80       80px     XXL padding (desktop), section spacing
Size/100      100px    Large section spacing
Size/120      120px    Maximum section spacing
```

## Neutral Color Scale

Dark-first UIs need more granularity in the dark end. Extend past the typical 50-900 range.

```
Step    Hex         Role
0       #FFFFFF     Pure white (text on dark, card backgrounds in light mode)
50      #FAFAFA     Off-white (page background in light mode)
100     #F5F5F5     Light surface (level-1 background in light mode)
200     #E0E1E5     Light border, dividers in light mode
300     #D4D5D9     Subheading in dark mode, stronger borders in light
400     #C2C3C9     Meta text in dark mode, disabled text in light
500     #9DA0A9     Meta/icon in dark mode, placeholder in light
600     #7D7F89     Secondary text (both modes)
700     #62646D     Subheading in light mode, icon in dark
800     #50525B     Body text in light mode, border in dark
900     #3C3E46     Link in light mode, subtle borders in dark
950     #2A2B31     Deep UI surfaces in dark mode
1000    #1D1E22     Near-black (headings in light, borders in dark)
1100    #131313     Dark surfaces (panel backgrounds, input backgrounds)
1200    #0D0D0D     Deeper dark surfaces (level-1 in dark mode)
1300    #0B0B0B     Very deep dark (level-2)
1400    #090909     Near-black surfaces
1500    #000000     Pure black (page background in dark mode)
```

## Accent Color Ramps

Each accent needs 8 steps from lightest tint to darkest shade.
Steps 100-300 are for backgrounds/tints, 400-500 for mid-tones, 600-800 for text/borders.

Structure per ramp: `Color/{RampName}/{Step}`

Common ramps: Orange (primary accent), Yellow (highlights), Green (teal/success indicators), Blue (info/links).

Add ramps based on your brand — but keep to 3-4 accent ramps maximum. More creates inconsistency.

## Notification Colors

Three semantic colors, each with a Light and Dark variant:
- **Success-Light / Success-Dark** — greens
- **Caution-Light / Caution-Dark** — oranges/ambers
- **Error-Light / Error-Dark** — reds

The Light variant is the full-saturation version (used on dark backgrounds).
The Dark variant is the muted/deeper version (used on light backgrounds or as backgrounds themselves).

## Alpha Channels

Two sets of opacity ramps (10% to 90% in 10% steps):
- **Dark Alpha (a-10 through a-90)** — `rgba(0,0,0,0.1)` to `rgba(0,0,0,0.9)` for overlays on light
- **Light Alpha (a-10 through a-90)** — `rgba(255,255,255,0.1)` to `rgba(255,255,255,0.9)` for overlays on dark

## Font Families

Store as STRING type variables:
- **Font/Family/Headings** — Display/heading typeface
- **Font/Family/Body** — Body copy typeface
- **Font/Family/Meta** — Monospace or condensed typeface for metadata, labels, timestamps

## Font Weights

Store as FLOAT type variables:
```
Font/Weight/Light       300
Font/Weight/Regular     400
Font/Weight/Medium      500 or 600
Font/Weight/Semi-Bold   700
Font/Weight/Bold        800
Font/Weight/Black       900
```

Note: Available weights depend on your typeface. Only create variables for weights your fonts actually support.
