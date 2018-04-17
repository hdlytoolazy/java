两种动态代理（JDK提供和cglib）的介绍这里就不在赘述了，区别是前者基于反射实现（仅能应用于接口）
，后者基于修改字节码实现（可应用于类）。

这里主要记录一下曾经碰到的几个坑。


1，自定义动态代理时，方法返回值为null（这里用JDK做记录，cglib也是一样）

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("前置内容");
        Object obj = method.invoke(object, args);
        System.out.println("后置内容");
	// 这里返回值代替原方法的返回值，若真实情况不关注返回值可为null，若需要则应返回obj
        return null;
    }



2，Spring AOP注解失效及解决（@Transactional @Async等注解不起作用）

1，同一个类中，方法A调用方法B（方法B上加有注解），注解无效
针对所有的Spring AOP注解，Spring在扫描bean的时候如果发现有此类注解，那么会动态构造一个代理对象。
如果你想要通过类X的对象直接调用其中带注解的A方法，此注解是有效的。因为此时，Spring会判断你将要调用
的方法上存在AOP注解，那么会使用类X的代理对象调用A方法。
但是假设类X中的A方法会调用带注解的B方法，而你依然想要通过类X对象调用A方法，那么B方法上的注解是无效的。
因为此时Spring判断你调用的A并无注解，所以使用的还是原对象而非代理对象。接下来A再调用B时，在原对象内B方法的注解当然无效了。

解决方法：
最简单的方式当然是可以让方法A和B没有依赖，能够直接通过类X的对象调用B方法。
但是很多时候可能我们的逻辑拆成这样写并不好，那么就还有一种方法：想办法手动拿到代理对象。
AopContext类有一个currentProxy()方法，能够直接拿到当前类的代理对象。那么以上的例子，就可以这样解决：

// 在A方法内部调用B方法
// 1）.直接调用B，注解失效。
B()
// 2）.拿到代理类对象，再调用B。
((X)AopContext.currentProxy()).B()

2，AOP注解方法里使用@Autowired对象为null

在之前的使用中，出现过在加上注解的方法中，使用其他注入的对象时，发现对象并没有被注入进来，为null。
最终发现，导致这种情况的原因是因为方法为private。因为Spring不管使用的是JDK动态代理还是CGLIB动态代理，
一个是针对实现接口的类，一个是通过子类实现。无论是接口还是父类，显然都不能出现private方法，否则子类或实现类都不能覆盖到。
如果方法为private，那么在代理过程中，根本找不到这个方法，引起代理对象创建出现问题，也导致了有的对象没有注入进去。
所以如果方法需要使用AOP注解，请把它设置为非private方法。


