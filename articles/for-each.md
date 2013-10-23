#Generic [for-each](../src/for-each.less) Structure in [LESS](http://lesscss.org/) Using Mixins

###Basic Usage
LESS code:
<pre lang="less"><code>@import <a href="../src/for-each.less">"for-each"</a>;

@list: banana, apple, pear, potato, carrot, peach;

basic-usage {
    .for-each(@list); .-for-each(@value) {
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

###Practical Example
LESS code:
<pre lang="less"><code>@import <a href="../src/for-each.less">"for-each"</a>;

.transition(@properties, @value...) {
    .for-each(@properties); .-for-each(@property) {
        transition+: @property @value;
    }
}

// ............................................................................

div {
    @properties: color, background-color, border-color;
    .transition(@properties, 2s, ease-out);
}

.button {
    .transition(background-color border-color, 1s ease-in);
}

.other {
    .transition(width, height; 3s, linear 1s);
}

.another {
    .transition(all, 4s);
}
</code></pre>
CSS output:
```css
div {
  transition: color 2s ease-out, background-color 2s ease-out, border-color 2s ease-out;
}
.button {
  transition: background-color 1s ease-in, border-color 1s ease-in;
}
.other {
  transition: width 3s, linear 1s, height 3s, linear 1s;
}
.another {
  transition: all 4s;
}
```
