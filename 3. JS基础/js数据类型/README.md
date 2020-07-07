## js数据类型

### 数据类型

```
js数据类型分为基本数据类型和引用数据类型。

基本数据类型有： Null、Undefined、Number、Boolean、String、Symbol(ES6);

引用数据类型：Object。Object包括Function、Array、Date等;
```


### 存储位置

&ensp;&ensp;两种数据类型的存储位置不同，基本数据类型是存储在栈（`stack`）中，引用数据类型是存储在堆（`heap`）中，同时也将指针存储在栈（`stack`）中。

&ensp;&ensp;栈和堆的区别在于：`heap`是没有结构的，数据可以任意存放，`heap`用于为复杂数据类型（引用类型）分配空间，例如数组对象、`object`对象；`stack`是有结构的，每个区块按照一定次序存放（后进先出），`stack`中主要存放一些基本类型的变量和对象的引用，存在栈中的数据大小与生存期必须是确定的。可以明确知道每个区块的大小，因此，`stack`的寻址速度要快于`heap`。


### `null`和`undefined`的区别与联系

&ensp;&ensp;`null`: 空对象指针。如果需要定义一个将来用于保存对象的变量，最好将其初始化为`null`;

&ensp;&ensp;`undefined`: 声明了一个对象但是没有赋值时，该变量默认值是undefined;

&ensp;&ensp;`undefined`值派生自`null`值，因此，`undefined == null`，返回`true`；但是他们的数据类型不同，所以 `undefined === null`，返回`false`；


### 数据类型检测

```
typeof 123;                     // 'number'
typeof false;                   // 'boolean'
typeof '456';                   // 'string'
typeof 123 + '456';             // 'string'
typeof undefined;               // 'undefined'
typeof null;                    // 'object'
typeof {x: 1};                  // 'object'
typeof [1,2,3];                 // 'object'
typeof new Date();              // 'object'
typeof function() {};           // 'function'
```