## Generic `for-each` Structure in [Less](http://lesscss.org/) Using Mixins

### Basic Usage
Less code:
<pre lang="less"><code>@import <a href="../src/for.less">"for"</a>;

@list: banana, apple, pear, potato, carrot, peach;

#basic-usage {
    .for(@list); .-each(@value) {
        value: @value;
    }
}
</code></pre>

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

### Practical Examples
Less code:
```less

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
Less code:
```less

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

More examples
---------------------
### Nested loops:
```less
#nested-loops {
    .for(a b c); .-each(@a) {
        .for(1 2 3); .-each(@b) {
            x: @a @b;
        }
    }
}
```
output:
```css
#nested-loops {
  x: a 1;
  x: a 2;
  x: a 3;
  x: b 1;
  x: b 2;
  x: b 3;
  x: c 1;
  x: c 2;
  x: c 3;
}
```
