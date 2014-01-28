##Generic for-each Structure in [LESS](http://lesscss.org/) Using Mixins

###Basic Usage
LESS code:
```less
@import "for";

@list: banana, apple, pear, potato, carrot, peach;

#basic-usage {
    .for(@list); .-each(@value) {
        value: @value;
    }
}
```
CSS output:
```css
basic-usage {
  value: banana;
  value: apple;
  value: pear;
  value: potato;
  value: carrot;
  value: peach;
}
```

###Practical Examples
LESS code:
```less
@import "for";

.transition(@properties, @value...) {
    .for(@properties); .-each(@property) {
        transition+: @property @value;
    }
}

div {
    @properties: color, background-color, border-color;
    .transition(@properties, 2s, ease-out);
}

.button {
    .transition(background-color border-color, 1s ease-in);
}

.another {
    .transition(all, 4s);
}
```
CSS output:
```css
div {
  transition: color 2s ease-out, background-color 2s ease-out, border-color 2s ease-out;
}
.button {
  transition: background-color 1s ease-in, border-color 1s ease-in;
}
.another {
  transition: all 4s;
}
```
LESS code:
```less
@import "for";

#icon {
    .for(home ok cancel error book); .-each(@name) {
        &-@{name} {
            background-image: url("../images/@{name}.png");
        }
    }
}
```
CSS output:
```css
#icon-home {
  background-image: url("../images/home.png");
}
#icon-ok {
  background-image: url("../images/ok.png");
}
#icon-cancel {
  background-image: url("../images/cancel.png");
}
#icon-error {
  background-image: url("../images/error.png");
}
#icon-book {
  background-image: url("../images/book.png");
}
```
