
## Singleton(单例模式)
*   单例模式是最常见的一种模式，在Web应用中，常见于运行为某个特定类创建一个可访问的实例。

```
<?php

    final class Product {
        //@var self
        private static $instance;
        //@var mixed
        public $classed;
        
        public static function getInstance() {
            if (!(self::$instance instanceof self)) {
                self::$instance = new self();
            }
            return self::$instance;
        }
     
        private function __construct() {}
     
        private function __clone() {}
    ｝
    
    $first = Product::getInstance();
    $second = Product::getInstance();
     
    $first->classed = 'test';
    $second->classed = 'example';
     
    print_r($first->classed);
    // example
    print_r($second->classed);
    // example
    
```

*   在很多情况下，需要为系统中的多个类创建单例的构造方式

```
    <?php
 
    abstract class FactoryAbstract {
     
        protected static $instances = array();
     
        public static function getInstance() {
            $className = static::getClassName();
            if (!(self::$instances[$className] instanceof $className)) {
                self::$instances[$className] = new $className();
            }
            return self::$instances[$className];
        }
     
        public static function removeInstance() {
            $className = static::getClassName();
            if (array_key_exists($className, self::$instances)) {
                unset(self::$instances[$className]);
            }
        }
     
        final protected static function getClassName() {
            return get_called_class();
        }
     
        protected function __construct() { }
     
        final protected function __clone() { }
    }
     
    abstract class Factory extends FactoryAbstract {
     
        final public static function getInstance() {
            return parent::getInstance();
        }
     
        final public static function removeInstance() {
            parent::removeInstance();
        }
    }
    // using:
     
    class FirstProduct extends Factory {
        public $a = [];
    }
    class SecondProduct extends FirstProduct {
    }
     
    FirstProduct::getInstance()->a[] = 1;
    SecondProduct::getInstance()->a[] = 2;
    FirstProduct::getInstance()->a[] = 3;
    SecondProduct::getInstance()->a[] = 4;
     
    print_r(FirstProduct::getInstance()->a);
    // array(1, 3)
    print_r(SecondProduct::getInstance()->a);
    // array(2, 4)

```