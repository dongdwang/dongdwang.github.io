[**Home**](https://dongdwang.github.io) / Observer 

------

## Observer mode(观察者模式)

*   观察者模式是定义对象间的一种一对多的依赖关系，以便当一个对象发生改变时，所有依赖他的对象都得到通知并自动刷新。

*	它完美的将观察者和被观察者对象分离，可以在独立的对象（主体）中维护一个对主体感兴趣的依赖项（观察器）列表。

*	让所有观察器各自实现公共的 Observer 接口，以取消主体和依赖性对象之间的直接依赖关系

### Example(举一个实际例子)
	
	典型的:用户注册(验证邮件，用户信息激活)，购物网站下单时邮件/短信通知等

### 例子 

```
interface DbInterface
{
    public function connect();

    public function query();

    public function close();
}

class PdoLibrary implements DbInterface {
	public function connect() {
		echo "已连接pdo";
	}

	public function query() {
		echo "进行pdo-sql语句执行";
	}

	public function close() {
		echo "关闭pdo数据库";
	}
}

class MysqliLibrary implements DbInterface {
	public function connect() {
		echo "已连接mysqli";
	}

	public function query() {
		echo "进行mysqli-sql语句执行";
	}

	public function close() {
		echo "关闭mysqli数据库";
	}
}

class Driver
{
    /**
     * @var array 注册事件的数组
     */
    public $listens = [];

    /**
     * 注册监听某个事件
     *
     * @param null $name  监听的事件名
     * @param null $event 发生此事件时的处理类
     */
    public function listen($name, $event)
    {
        $this->listens[] = [strtolower($name), $event];
    }

    /**
     * 发送事件
     *
     * @param $name 事件名
     * @param $func 函数方法
     * @throws \Exception
     */
    public function fire($name, $func)
    {
        $name = strtolower($name);
        foreach ($this->listens as $listen) {
            if ($listen[0] == $name) {
                $instance = $this->getInstance($listen[1]);
                if (!$instance instanceof DbInterface) {
                    throw new \Exception("事件类型错误:".$name." function:". func);
                }
                call_user_func(array($instance, $func));
            }
        }
    }

    /**
     * 获得事件处理类
     *
     * @param $concrete 实现方式
     * @return mixed|null|string|object
     */
    public function getInstance($concrete)
    {
        $instance = null;
        //如果是字符串
        if (is_string($concrete)) {
            if (class_exists($concrete)) {
                $instance = new $concrete();
            }
        }
        if (is_object($concrete)) {
            if ($concrete instanceof \Closure) {
                $instance = call_user_func($concrete);
            } else {
                $instance = $concrete;
            }
        }
        return $instance;
    }
}



$driver  = new Driver();	

//开启服务，注册时只是字符串，不是实例化，因而资源利用率高
$services = [
	['pdo', 'PdoLibrary'],
	['mysqli', 'MysqliLibrary']
];

//注册事件
array_walk($services, function($service, $key) use ($driver) {

	$driver->listen(...$service);

});

//实例化并发起事件，通知连接
$driver->fire('pdo', 'connect');

```