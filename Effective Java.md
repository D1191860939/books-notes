1. Do not use bounded types as return types.Rather than providing additional flexibility of your users, it would force them to use wildcard types in client code.
2. 关于Java8的default method：
- 设计初衷：在Java8之前，如果我们有一个接口A，它拥有实现类B、C、D， 为了JDK标准库的函数式风格而设计的。


<!--stackedit_data:
eyJoaXN0b3J5IjpbMTQ2MDY2NjczMiwtNTgxMTE3OTU1LDIwND
AyOTc2MjJdfQ==
-->