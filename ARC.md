“在MRC时代你需要自己retain一个想要保持的对象，而现在不需要了。现在唯一要做的是用一个指针指向这个对象，只要指针没有被置空，对象就会一直保持在堆上。当将指针指向新值时，原来的对象会被release一次”  
https://onevcat.com/2012/06/arc-hand-by-hand/?continueFlag=f17d716399b252c759046586f450405e