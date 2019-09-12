1. Do not use bounded types as return types.Rather than providing additional flexibility of your users, it would force them to use wildcard types in client code.
2. 关于Java8的default method：
- 设计初衷：在Java8之前，如果我们有一个接口A，它拥有实现类B、C、D，现在我们又需要在接口A中添加一个新的抽象方法，那么我们必须在B、C、D中都需要提供对应的实现。而现实情况是只有D类有具体的逻辑要求，而B、C并没有，这样导致的现象就是B、C中需要有空实现。这样的情况在语言设计者为了JDK标准库的支持函数式风格就会遇到，比如Collection接口中stream方法， 


<!--stackedit_data:
eyJoaXN0b3J5IjpbNjAzODM2NTQ3LC01ODExMTc5NTUsMjA0MD
I5NzYyMl19
-->