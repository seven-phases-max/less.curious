LESS: Generic For
=================

###  Basic example
LESS code:
	
    #basic-usage {
        .for(1, 6); .-(@i) {
            i: @i;
        }
    }
	
CSS output:
	
    basic-usage {
      i: 1;
      i: 2;
      i: 3;
      i: 4;
      i: 5;
      i: 6;
    }
    
###  Implementation
    
    .for(@n)     {.for(1, @n)}
    .for(@i, @n) {.-(@i)}
    .for(@i, @n) when not (@i = @n) {
        .for((@i + (@n - @i) / abs(@n - @i)), @n);
    }

###  More real-world example
LESS code:
	
    @grid-columns: 5;
    
    .column {
        .for(@grid-columns); .-(@i) {
            &-@{i} {width: (@i / @grid-columns * 100%)}
        }
    }
	
CSS output:
	
    .column-1 {
      width: 20%;
    }
    .column-2 {
      width: 40%;
    }
    .column-3 {
      width: 60%;
    }
    .column-4 {
      width: 80%;
    }
    .column-5 {
      width: 100%;
    }
    
### .For vs. handwritten Loop

<table><tr>
<td><pre>
// hand-written loop 

@grid-columns: 5;

.column(1);
.column(@i) when (@i =&lsaquo; @grid-columns) {                
    .&-@{i} {width: (@i / @grid-columns * 100%)}
    .column((@i + 1));
}
</pre></td>
<td><pre>
// using .for

@grid-columns: 5;

.column {
    .for(@grid-columns); .-(@i) {
        &-@{i} {width: (@i / @grid-columns * 100%)}
    }
}
</pre></td>
</tr></table>
    
More curious examples
---------------------

### Nested loops:
	
    #nested-loops {
        .for(3, 1); .-(@i) {
            .for(0, 2); .-(@j) {
                x: (10 * @i + @j);
            }
        }
    }
        
output:
	
    #nested-loops {
      x: 30;
      x: 31;
      x: 32;
      x: 20;
      x: 21;
      x: 22;
      x: 10;
      x: 11;
      x: 12;
    }
    
### Multiple loops in a rule:

    #multiple-loops {
        & {.for(1, 3); .-(@i) {x: @i}}
        & {.for(4, 6); .-(@i) {y: @i}}
    }
    
output:

    #multiple-loops {
      x: 1;
      x: 2;
      x: 3;
    }
    #multiple-loops {
      y: 4;
      y: 5;
      y: 6;
    }

### Learn the LESS scope:

    happy debugging {
        .foe.and-friend-scope();
        and & to me {
            .foe(0, 9); .-(@i) {a: @i}
        }
    }
    
    .foe(...) {
        .for(0, 2);
        .and-friend-scope() {
            .-(@i) {b: @i} to you {
                .for(9, 7);
                .for(5, 3);
            }
        }
    }
    
output:

    happy debugging to you {
      b: 9;
      b: 8;
      b: 7;
      b: 5;
      b: 4;
      b: 3;
    }
    and happy debugging to me {
      a: 0;
      a: 1;
      a: 2;
    }
