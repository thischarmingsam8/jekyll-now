---
layout: post
title: Football shirts via Sass maps
---

I recently wanted to implement football shirts for my fantasy football side project in solely in CSS, so they rendered sharply at all resolutions / didn't add network requests / require adding to a sprite sheet and fiddling with background positions.

I have to chop'n'change the shirt colours once a season, when teams update their kits and teams are promoted or relegated. This means coming back to some non-trivial CSS almost a year after last looking at it. I wanted the syntax to be human-readable and in one place so I don't miss anything out.

## Enter Sass maps ##

Sass maps allowed me to create human-readable keys which I'd recognise the next time I had to edit them. Each team has a shortcode (also used in the game's mechanics) and a set of options to style with:

- `shirt-colour`,
- `stitching`,
- `sleeves`,
- `stripes` (optional),
- `stripe-width` (optional),
- `left-sleeve-colour` (optional),
- `right-sleeve-colour` (optional)
- `sleeve-stripes` (optional).

These options can satisfy even the pickiest of teams (Crystal Palace's shirt sleeves or Middlesborough's diagonal stripe):

_(Some teams omitted for brevity)_

```sass
$teams : (
    ars : (
        shirt-colour: red,
        stitching: white,
        sleeves : white
    ),
    bou : (
        shirt-colour: red,
        shirt-stripes: linear-gradient(to left, transparent 50%, black 50%),
        stripe-width : 1,
        stitching: black,
        sleeve-stripes: linear-gradient(to left, transparent 50%, black 50%)
    ),
    cry : (
        shirt-colour: purple,
        shirt-stripes: linear-gradient(to left, crimson, crimson 50%, royalblue 50%, royalblue),
        stitching: yellow,
        left-sleeve : crimson,
        right-sleeve: royalblue
    ),
    mid : (
        shirt-colour: red,
        shirt-stripes: linear-gradient(to top right, transparent 0%, white 30%, transparent 75%),
        stripe-width : 2,
        stitching: red
    ),
    sou : (
        shirt-colour: white,
        stripe-width : 1,
        shirt-stripes: linear-gradient(to left, transparent 50%, red 50%),
        stitching: red,
        sleeves: red
    ),
    wba : (
        shirt-colour: white,
        stitching: darkblue,
        shirt-stripes: linear-gradient(to left, transparent 50%, darkblue 50%),
        stripe-width : 1
    ),
    whu: (
        shirt-colour: purple,
        stitching: #B5E0F2,
        sleeves: #B5E0F2
    )
) !default;
```
    
The shirts themselves are made up of an element for the body of the shirt, with it's :before element as the left sleeve, and it's :after element as it's right sleeve. A set of mixins are responsible for iterating through the teams and generating each shirt type:
    
```sass
//colour in each teams shirts
@mixin team-shirts{
     @each $shortcode, $team in $teams {
        &.#{$shortcode}{
            @if(map-get($team, shirt-stripes)) {
                background: map-get($team, shirt-stripes);
            }

            @if(map-get($team, stitching)) {
                border-color: map-get($team, stitching);
            }
            //left sleeve should be defined first
            @if(map-get($team, right-sleeve) and map-get($team, left-sleeve)) {
                &:before {
                    background-color: map-get($team, left-sleeve);
                }
                &:after{
                    background-color: map-get($team, right-sleeve);
                }
            } @else if(map-get($team, sleeve-stripes)) {
                &:before, &:after {
                    background-image: map-get($team, sleeve-stripes);
                    background-color: map-get($team, shirt-colour);
                }
            } @else if(map-get($team, sleeves)) {
                &:before, &:after {
                    background-color: map-get($team, sleeves);
                }
            } @else {
                &:before, &:after {
                    background-color: map-get($team, shirt-colour);
                }
            }
            //bg-colour
            @if(map-get($team, shirt-colour)) {
                background-color: map-get($team, shirt-colour);
            }
        }
     }
}

//work out correct background size for striped teams
@mixin team-bg-size($multiplier: 10) {
    @each $shortcode, $team in $teams {
        @if(map-get($team, stripe-width)) {
            &.#{$shortcode}{
              background-size: (map-get($team, stripe-width) * $multiplier) + px;
            }
        }
    }
}
```

Finally, one placeholder class holds the result of the mixin, which carries the different classes for each individual team shirt: _(actual player class not shown for brevity, because it appears in a few different places)_

```sass
%player-shirt {
    margin: 0 auto;
    width: 22px;
    height: 34px;
    border-radius: 0;
    position: relative;
    //default colours
    background-color: $gray;
    border: 1px solid $gray;

    @include breakpoint(mobile) {
        width: 14px;
        height: 20px;
    }

    &:before, &:after {
        content: '';
        position: absolute;
        width: 8px;
        height: 17px;
        border-width: 1px;
        border-style: solid;
        border-color: inherit;
        //default colour
        background-color: $gray;
        top: 0;

        @include breakpoint(mobile) {
            height: 10px;
            width: 6px;
        }                 
    }

    &:before{
        left: -9px;
        border-right-width: 0;
        border-top-left-radius: 6px;

        @include breakpoint(mobile) {
            left: -7px;
        }
    }

    &:after{
        right: -9px;
        border-left-width: 0;
        border-top-right-radius: 6px;

        @include breakpoint(mobile) {
            right: -7px;
        }
    }

    // adjustments for striped teams
    @include team-bg-size;

    @include team-shirts;
}
```

Here's an example of the shirts in action:

![Bob's team](/images/football-shirts-via-sass-maps.png)