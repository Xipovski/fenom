Aspect templater
================


## Быстрый старт


Отрисовать шаблон `pages/about.tpl`:

```php
  $aspect = Aspect::factory('./templates', './compiled', Aspect::CHECK_MTIME);
  $aspect->display("pages/about.tpl", $data);
```

Получить результат отрисовки шаблона:

```php
  $aspect = Aspect::factory('./templates', './compiled', Aspect::CHECK_MTIME);
  $content = $aspect->fetch("pages/about.tpl", $data);
```

Создание шаблона в реальном времени:

```php
  use MF\Aspect;
  $aspect = new Aspect();
  $tempate = $aspect->compileCode('Hello {$user.name}! {if $user.email?} Your email: {$user.email} {/if}');

  $tempate->display($data);
  $content = $tempate->fetch($data);
```

## Синтаксис

Базовый синтаксис, по бОльшей части, унаследован от Smarty3.

### Переменные

#### Вывод переменной

Вывод значений переменных в шаблонизаторе Aspect идентичен правилам вывода шаблонизатора Smarty

```smarty
{$foo}        <-- displaying a simple variable (non array/object)
{$foo[4]}     <-- display the 5th element of a zero-indexed array
{$foo.4}
{$foo.bar}    <-- display the "bar" key value of an array, similar to PHP $foo['bar']
{$foo.'bar'}
{$foo."bar"}
{$foo[bar]}
{$foo['bar']}
{$foo["bar"]}
{$foo.$bar}   <-- display variable key value of an array, similar to PHP $foo[$bar]
{$foo[$bar]}
{$foo->bar}   <-- display the object property "bar"
{$foo->bar()} <-- display the return value of object method "bar"
```

#### Комбинированные варианты

```smarty
{$foo.bar.baz}
{$foo.$bar.$baz}
{$foo[4].baz}
{$foo[4].$baz}
{$foo.bar.baz[4]}
{$foo->bar($baz, 2, $bar)} <-- passing parameters
{"foo"}       <-- static values are allowed
```

#### Математические операции

```smarty
{$x+$y}                   // will output the sum of x and y.
{$foo[$x+3]}              // as array index
{$foo[$x+3]*$x+3*$y % 3}
```

#### Объявление переменных

```smarty
{var $foo = $x + $y}
{var $foo = $x.y[z] + $y}
{var $foo = strlen($a)}
{var $foo = myfunct( ($x+$y)*3 )}
{var $foo.bar.baz = 1}
```

#### Объявление массивов

```smarty
{var $foo=[1,2,3]}
{var $foo=['y'=>'yellow','b'=>'blue']}
{var $foo=[1,[9,8],3]}   // can be nested
```

#### Работа с объектами

```smarty
{$object->method1($x)->method2($y)}
{var $foo=$object->item->method($y, 'named')}
```


#### Работа со строками

Строки в Aspect обрабатываются идентично правилам подстановки переменных в строки в PHP, т.е. в двойных кавычках переменная заменяется на её значение, в одинарных замены не происходит.
В отличие от Smarty в строках не обрабатываются управляющие конструкции, например "if", но работают модификаторы.

```smarty
{var $foo="Ivan"}
{var $user.name="Ivan"}
{"Hi, $foo"}          выведет "Hi, Ivan"
{"Hi, {$foo}"}        выведет "Hi, Ivan"
{"Hi, {$user.name}"}  выведет "Hi, Ivan"
{'Hi, $foo'}          выведет 'Hi, $foo'
{'Hi, {$foo}'}        выведет 'Hi, {$foo}'
```

### Модификаторы

* Модификаторы позволяют изменить значение переменной перед выводом или использованием в выражении
* Модификаторы записываются после переменной через символ вертикальной черты "|"
* Модификаторы могут иметь параметры, которые записываются через символ двоеточие ":" после имени модификатора
* Параметры модификаторов друг от друга также разделяются символом двоеточие ":"
* В качестве параметров могут использоваться переменные.
* Модификаторы могут составлять цепочки. В этом случае они применяются к переменной последовательно слева направо

```smarty
{var $foo="Ivan"}
{$foo|upper}     выведет "IVAN"
{$foo|lower}     выведет "ivan"
{$looong_text|truncate:80:"..."}  обрежет текст до 80 символов и добавит "..." в конец текста
{$looong_text|lower|truncate:$settings.count:$settings.etc}
{var $foo="Ivan"|upper}    переменная $foo будет содержать "IVAN"
```

Подробнее модификаторы описаны в разделе [модификаторы](docs/modifiers.md)

### Функции

Каждый тэг шаблонизатора либо выводит переменную, либо вызывает какую-либо функцию.
Тег вызова функции начинается с названия функции и содержит список аргументов:

```smarty
{FUNCNAME attr1 = "val1" attr2 = $val2}
```

Это общий формат функций, но могут быть исключения, например функция {var}, использовавшаяся выше.

```smarty
{include file="my.tpl"}
{mailto address="bzick@megagroup.ru" text="Article's author"}
{var $foo=5}
{if $user.loggined}
    Welcome, <span style="color: red">{$user.name}!</span>
{else}
    Who are you?
{/if}
```

#### Аргументы

Аргументы принимают любой формат переменных, в том числе результаты арифметических операций и модификаторов.

```smarty
{funct arg=true}
{funct arg=5}
{funct arg=1.2}
{funct arg='string'}
{funct arg="string this {$var}"}
{funct arg=[1,2,34]}
{funct arg=$x}
{funct arg=$x.c}
```

```smarty
{funct arg="ivan"|upper}
{funct arg=$a.d.c|lower}
```

```smarty
{funct arg=1+2}
{funct arg=$a.d.c+4}
{funct arg=($a.d.c|count+4)/3}
```