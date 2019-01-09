## 反射
>  反射：当你需要实例化一个类，但是这个类对你是未知的或者是封闭的，这时候你可以通过创建反射来映射这个类。
通过一系列的探测最终来实例化这个类。
>> 反射是laravel实现依赖注入的基础。
```
class Circle
{
    /**
     * @var int
     */
    public $radius;
    /**
     * @var Point
     */
    public $center;
    const PI = 3.14;
    public function __construct(Point $point, $radius = 1)
    {
        $this->center = $point;
        $this->radius = $radius;
    }
    public function printCenter()
    {
        printf('center coordinate is (%d, %d)', $this->center->x, $this->center->y);
    }
    public function area()
    {
        return 3.14 * pow($this->radius, 2);
    }
}
/**
 * Class Point
 */
class Point
{
    public $x;
    public $y;
    /**
     * Point constructor.
     * @param int $x  horizontal value of point's coordinate
     * @param int $y  vertical value of point's coordinate
     */
    public function __construct($x = 0, $y = 0)
    {
        $this->x = $x;
        $this->y = $y;
    }
}
function make($className)
{
    $reflectionClass = new ReflectionClass($className);
    $constructor = $reflectionClass->getConstructor();
    $parameters  = $constructor->getParameters();
    $dependencies = getDependencies($parameters);
    return $reflectionClass->newInstanceArgs($dependencies);//从给出的参数创建一个新的类实例。
}
function getDependencies($parameters)
{
    $dependencies = [];
    foreach($parameters as $parameter) {
        $dependency = $parameter->getClass();
        if (is_null($dependency)) {
            if($parameter->isDefaultValueAvailable()) {//检查是否有默认值
                $dependencies[] = $parameter->getDefaultValue();//获取默认值
            } else {
                //to easily implement this function, I just assume 0 to built-in type parameters
                $dependencies[] = '0';
            }
        } else {
            $dependencies[] = make($parameter->getClass()->name);
        }
    }
    return $dependencies;
}
$circle = make('Circle');
var_dump($area = $circle->area());
```
## 输出结果
> float(3.14)
> 
```
$dependencies = getDependencies($parameters);
return $reflectionClass->newInstanceArgs($dependencies);//从给出的参数创建一个新的类实例。
```
> 应该是递归获取该类创建的时候所需要的依赖，并通过newInstanceArgs方法生成依赖，以达到实例化类的目的。