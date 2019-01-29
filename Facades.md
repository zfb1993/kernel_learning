## Facades（外观模式）
外观模式：为子系统的一组接口提供了一个一致的界面。Facade模式定义了一个高层接口，使用这个接口，使得这个子系统更容易使用。
引入外观角色后，用户只需要直接与外观角色交互，用户与子系统直接的复杂关系由外观角色来实现，从而降低了系统的耦合度。  

for example:一个电源总开关可以控制四盏灯、一个风扇、一台空调和一台电视机的启动和关闭。该电源总开关可以同时控制上述所有电器设备，电源总开关即为该系统的外观模式设计。

实现原理：  
```
	use Illuminate\Support\Facades\Facade;
    
    class Cache extends Facade
    {
        /**
         * 获取组件注册名称
         *
         * @return string
         */
        protected static function getFacadeAccessor() { 
            return 'cache'; 
        }
    }
```
1. 首先我们自定义一个cache类，并实现一个抽象方法```getFacadeAccessor```，
这个方法只返回一个字符串，即服务容器绑定的类的别名。

###如何实现
2. 我们来看看```Illuminate\Support\Facades\Facade```的源码：
```
abstract class Facade
{
    /**
     * The application instance being facaded.
     *
     * @var \Illuminate\Contracts\Foundation\Application
     */
    protected static $app;

    /**
     * The resolved object instances.
     *
     * @var array
     */
    protected static $resolvedInstance;

    /**
     * Get the root object behind the facade.
     *
     * @return mixed
     */
    public static function getFacadeRoot()
    {
        return static::resolveFacadeInstance(static::getFacadeAccessor());
    }

    /**
     * Get the registered name of the component.
     *
     * @return string
     *
     * @throws \RuntimeException
     */
    protected static function getFacadeAccessor()
    {
        throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
    }

	/**
     * Resolve the facade root instance from the container.
     *
     * @param  string|object  $name
     * @return mixed
     */
    protected static function resolveFacadeInstance($name)
    {
        if (is_object($name)) {
            return $name;
        }

        if (isset(static::$resolvedInstance[$name])) {
            return static::$resolvedInstance[$name];
        }

        return static::$resolvedInstance[$name] = static::$app[$name];
    }

    /**
     * Handle dynamic, static calls to the object.
     *
     * @param  string  $method
     * @param  array   $args
     * @return mixed
     *
     * @throws \RuntimeException
     */
    public static function __callStatic($method, $args)
    {
        $instance = static::getFacadeRoot();

        if (! $instance) {
            throw new RuntimeException('A facade root has not been set.');
        }

        return $instance->$method(...$args);
    }
}

```
注意```__callStatic```
```
public static function __callStatic($method, $args)
{
	$instance = static::getFacadeRoot();

	if (! $instance) {
		throw new RuntimeException('A facade root has not been set.');
	}

	return $instance->$method(...$args);
}
```
这是php的一个魔术方法。当以静态的方式调用一个不存在的方法时，这个方法就会被调用。  

再看```getFacadeRoot```方法，这个方法就是从容器中解析出对象：
```
public static function getFacadeRoot(){
    return static::resolveFacadeInstance(static::getFacadeAccessor());
}

```
```
protected static function getFacadeAccessor(){
     throw new RuntimeException('Facade does not implement getFacadeAccessor method.');
}
```
```
protected static function resolveFacadeInstance($name){
    if (is_object($name)) {
        return $name;
     }

    if (isset(static::$resolvedInstance[$name])) {   
        return static::$resolvedInstance[$name];
    }

    return static::$resolvedInstance[$name] = static::$app[$name];
}
```
其中的```getFacadeAccessor```必须要被重写，否则将抛出异常。然后再```resolveFacadeInstance```中，
先判断是否是一个对象，如果是的话直接返回。  
然后会去判断需要解析的对象是否已经解析过了，如果解析过了就直接返回，否则会从容器中去解析再返回，这样不仅仅实现了单例，而且还可以提升性能。
