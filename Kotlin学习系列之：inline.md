1. inline: Kotlin中的一个关键字，用来修饰function，那么这个function就被称作inline function（内联函数）。最初接触内联函数这个概念还是当初在学校学习C语言时候提到的，Java中是没有这一概念的，如今Kotlin又引入这一特性。那么inline function有什么优势或者说特点呢？
2. 特点：当你调用一个inline function的时候，编译器不会产生一次函数调用，而是会在每次调用时，将inline function中的代码直接嵌入到调用处。我们都知道，一旦触发函数调用，就意味着栈的切换，就会影响效率，所以，inline function的效率会更高
3. 那么在Kotlin中如何去声明一个inline function呢？

<!--stackedit_data:
eyJoaXN0b3J5IjpbNjI0NzEyNDM2LC0yMDg4NzQ2NjEyXX0=
-->