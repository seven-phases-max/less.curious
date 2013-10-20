[for-each.less](//github.com/seven-phases-max/less.curious/blob/master/src/for-each.less)
===============

Usage:

	@import "for-each";
    
    @list: banana, apple, pear, potato, carrot, peach;
    
    #basic-usage {
        .for-each(@list); .-for-each(@value) {
            value: @value;
        }
    }
	
CSS output:
	
    #basic-usage {
      value: banana;
      value: apple;
      value: pear;
      value: potato;
      value: carrot;
      value: peach;
    }
    
