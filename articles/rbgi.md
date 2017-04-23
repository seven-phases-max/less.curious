
## On refactoring Boostrap 3 grid implementation.

### See [Inspiring discussion](https://github.com/less/less.js/issues/1785#issuecomment-31876655) first.

So indeed, every time I write some `.col-md-6` in my HTML I recall how this selector was generated and this actually drives me crazy :)

Here's that "horrible" [`.make-grid-columns`](https://github.com/twbs/bootstrap/blob/v3.2.0/less/mixins/grid-framework.less#L6-L27) code (same goes for [`.float-grid-columns`](https://github.com/twbs/bootstrap/blob/v3.2.0/less/mixins/grid-framework.less#L29-L44)):
```less
.make-grid-columns() {
  // Common styles for all sizes of grid columns, widths 1-12
  .col(@index) when (@index = 1) { // initial
    @item: ~".col-xs-@{index}, .col-sm-@{index}, .col-md-@{index}, .col-lg-@{index}";
    .col((@index + 1), @item);
  }
  .col(@index, @list) when (@index =< @grid-columns) { // general; "=<" isn't a typo
    @item: ~".col-xs-@{index}, .col-sm-@{index}, .col-md-@{index}, .col-lg-@{index}";
    .col((@index + 1), ~"@{list}, @{item}");
  }
  .col(@index, @list) when (@index > @grid-columns) { // terminal
    @{list} {
      position: relative;
      // Prevent columns from collapsing when empty
      min-height: 1px;
      // Inner gutter via padding
      padding-left:  (@grid-gutter-width / 2);
      padding-right: (@grid-gutter-width / 2);
    }
  }
  .col(1); // kickstart it
}
```
Doh!

No, actually, strictly speaking, there's nothing wrong with this code. It's valid Less, it does what it is supposed to do and works like a charm after all. It's just "not elegant". And this "not elegant" is actually enough reason for me (who spent a lot of time and efforts on advertising and pushing "a modern and proper methods of working with Less lists/arrays/loops and related stuff") to try to find a better way to generate equal CSS.

(Well, even more strictly speaking, there *are* other more rational reasons for such "string-based selector manipulation" to be avoided whenever possible, but that's another big story so here I'll stop at just "not elegant").

-
### So the problem:
> There's no straight-forward method to generate and use a list of selectors in Less.

There are some feature requests/ideas that will improve the situation eventually (see [#1421](https://github.com/less/less.js/issues/1421), [#1177](https://github.com/less/less.js/issues/1177) etc.), but they at best are in their early planning stage so it's hard to say when we'll finally be able to rewrite this code "the right way".

### So the task:
> Can we rewrite Bootstrap grid generation code in not so hackish way *right now*?

Currently, the only "Less-native"/"non-hackish" way to generate a ruleset with a list of arbitrary selectors is the [`extend`](http://lesscss.org/features/#extend-feature) feature. So the first (and the simplest) solution is to move the styles of the generated selector list into a ruleset with some predefined name and then just `extend` it by generated classes:

### Method #1 ("Dummy Classes").
```less
.grid-column-any {
    // Common styles for all sizes of grid columns
    position: relative;
    min-height: 1px;
    padding-left:  (@grid-gutter-width / 2);
    padding-right: (@grid-gutter-width / 2);
}

.make-grid-columns() {
    .col(@index) when (@index > 0) {
        .col((@index - 1));
        .col-xs-@{index},
        .col-sm-@{index},
        .col-md-@{index},
        .col-lg-@{index} {
            &:extend(.grid-column-any);
        }
    }
    .col(@grid-columns);
}
```
That's it (actual code for complete `grid-framework.less` will be a bit different to accommodate various scoping and `@media` stuff but the principle remains the same: see [the branch](https://github.com/seven-phases-max/bootstrap/blob/refactoring-grid-framework-m1/less/mixins/grid-framework.less)). The only problem of this approach is that the compiled CSS now contains that "unused" "private" `.grid-column-any` selector and complete `grid-framework.less` would need 4 or 5 of those (Below I will refer to such selectors as just "dummy" classes/selectors/templates).

-
> Getting rid of the dummy selectors...

If Less has the [":extend mixins" feature](https://github.com/less/less.js/issues/1177) we could simply turn the template into a mixin and it won't appear in the CSS, but Less has not... so it's time for some trick.
### Method #2 ("Extending `.col-*-1` classes").
And here we are, set the template styles in the first column classes and than `extend` this first column(s) by subsequent column classes:
```less
.col-xs-1,
.col-sm-1,
.col-md-1,
.col-lg-1 {
    // Common styles for all sizes of grid columns
    position: relative;
    min-height: 1px;
    padding-left:  (@grid-gutter-width / 2);
    padding-right: (@grid-gutter-width / 2);
}

.make-grid-columns() {
    .col(@index) when (@index > 1) {
        .col((@index - 1));
        .col-xs-@{index},
        .col-sm-@{index},
        .col-md-@{index},
        .col-lg-@{index} {
            &:extend(.col-xs-1);
        }
    }
    .col(@grid-columns);
}
```
(Yet again the actual code of `grid-framework.less` would be slightly different: see [the branch](https://github.com/seven-phases-max/bootstrap/blob/refactoring-grid-framework-m2/less/mixins/grid-framework.less).)

Unfortunately this approach has one quite critical problem. While such mixin works just fine on its own, in the framework it may interfere with and be broken by another grid related code. Basically if you for example add another `.col-xs-1` definition/styles anywhere at the global scope of your Less code, e.g.:
```less
.col-xs-1 {
    color: red;
}
```
the styles of this new definition will automatically propagate to *every other* grid column. Fortunately, currently even if other Bootstrap grid mixins *do* generate other first column styles these other styles *do not* break anything simply because their selectors are generated via selector interpolation and `extend` does not work with such dinamically generated selectors. 

So this method may still be considered as a working temporary solution, but only if we don't want the Method #1 with its dummy classes really badly deadly. After all this "extending first column" method will be irrecoverably broken as soon as Less brings "extending dinamicaly generated selectors" feature. And before that happens the mentioned potential issues will keep hiding there with its `.col-xs-1 {color: red}` grenade ready.

-
> Getting rid of the dummy selectors, the other way around...

### Method #3 ("Emulating #1177").
It is possible to emulate the [":extend mixins" feature](https://github.com/less/less.js/issues/1177) by `extend`ing a ruleset defined in a file imported with [`reference`](http://lesscss.org/features/#import-options-reference) option. I.e. we can move our dummy templates to a separate file, `@import (reference) "it";` and voilà, no dummy selectors in the generated CSS:

```less
// ............................................................
// grid-aux.less:

.grid-column-any {
    // Common styles for all sizes of grid columns
    position: relative;
    min-height: 1px;
    padding-left:  (@grid-gutter-width / 2);
    padding-right: (@grid-gutter-width / 2);
}

// ............................................................
// grid-framework.less:

@import (reference) "grid-aux.less";

.make-grid-columns() {
    .col(@index) when (@index > 0) {
        .col((@index - 1));
        .col-xs-@{index},
        .col-sm-@{index},
        .col-md-@{index},
        .col-lg-@{index} {
            &:extend(.grid-column-any);
        }
    }
    .col(@grid-columns);
}
```
Clean and relatively simple (not counting it brings additional file(s) and requires use of `reference` never used in Bootstrap code before). 

Unfortunately the things become not so simple (and even less clean) if we try to apply this approach to other `grid-framework` mixins (the one that generates media depended column rulesets in particular). To be able to `extend` a template ruleset within a `@media` block we need such ruleset to be defined *in* this `@media` block (see ["Scoping / Extend Inside @media"](http://lesscss.org/features/#extend-feature-scoping-extend-inside-media)). It's definitely possible (for example we can import our auxiliary file right within a column generating mixin so it "automatically" brings the necessary templates into corresponding `@media` [blocks](https://github.com/twbs/bootstrap/blob/v3.2.0/less/grid.less#L56) (I actually have a working concept somewhere among my local branches)), but the complete `grid-framework` implementation using this approach becomes so tricky, cryptic and unreadable so it actually spoils the very idea of the refactoring (i.e. we'll just replace one "non-elegant and hackish" implementation with another "maybe a bit more elegant but even yet more hackish" one). 

So I would not consider going this "reference" route seriously (for the Bootstrap code-base at least).

-
### Summary:

**Method #1. "Extending dummy templates":**

- *Cons*: Dummy classes in the compiled CSS.
- *Pros*: The most future-proof and maintainable.

My vote definitely goes for this method. After all who does care of four or five unused identifiers in a CSS with (several?) thousands of those? :)

**Method #2. "Using `col-*-1` rulesets as template styles":**

- *Pros*: The most "optimal" for the moment (no dummy classes in the output).
- *Cons*: Quite flawed. Busts when any additional styles are defined with explicitly specified "col-*-1" selectors (thus it will require extra efforts for tracking if any further changes in the code base do not actually break the grid somehow). It also gets finally broken when Less brings "extending dynamically generated selectors" feature.

**Method #3. "Extending dummy templates with `reference`":**

- *Pros*: No dummy classes in the output. More safe than **#2**.
- *Cons*: Too tricky and too bloating code.

---

### Also see:
* [this ticket](https://github.com/less/less.js/issues/2702#issuecomment-142942530) for "What's wrong with Bootstrap-like media/grid code in general".
