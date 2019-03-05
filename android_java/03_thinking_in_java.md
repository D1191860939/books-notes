1. 代码复用体现的方面：
- 直接使用类的对象
- 类与类之间的组合（composition）
- 继承（inheritance）
2. 关于clone：As  a rule, copy functionality is best provided by constructors or factories.但是也有一个例外，那就是对于数组的拷贝.
3. If a package-private top-level class or interface is used by only one class, consider making the top-level class a private static nested class of the sole class that uses it.
4. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY4MjM3NTgzNCwtOTk1ODg4MjYxLDc4OD
UxMjc1NV19
-->