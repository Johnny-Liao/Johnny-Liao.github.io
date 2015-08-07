#IoC容器的实现
IoC(Inverse of Control 控制反转)：依赖对象的获得被反转了。基于这个结论为控制反转取了个更好听的名字依赖注入。
Spring的IoC容器它可以在对象生成或初始化时直接将数据注入到对象中。

IoC中容器的设计与实现：
两个主要的容器系列：
一个是实现BeanFactory的简单容器系列，只实现了容器的最基本功能。
另一个是实现ApplicationContext应用上下文，作为容器的高级形式存在。
| First Header  | Second Header |
| ------------- | ------------- |
| Content Cell  | Content Cell  |
| Content Cell  | Content Cell  |