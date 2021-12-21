PHP 8.0 是 PHP 语言的一个主版本更新。
它包含了很多新功能与优化项， 包括 **JIT**、命名参数、联合类型、注解、构造器属性提升、match 表达式、nullsafe 运算符、，并改进了类型系统、错误处理、语法一致性。



## 即时编译

PHP 8 引入了两个即时编译引擎。 Tracing JIT 在两个中更有潜力，它在综合基准测试中显示了三倍的性能， 并在某些长时间运行的程序中显示了 1.5-2 倍的性能改进。

### 关于 JIT 对 PHP 8 性能的贡献

![image-20210108175520328](https://img-note.langyastudio.com/20210108175520.png?x-oss-process=style/watermark)



## 命名参数 [RFC](https://wiki.php.net/rfc/named_params)

PHP 7

```php
htmlspecialchars($string, ENT_COMPAT | ENT_HTML401, 'UTF-8', false);
```

PHP 8

```php
htmlspecialchars($string, double_encode: false);
```

- 仅仅指定必填参数，跳过可选参数。
- 参数的顺序无关、自己就是文档（self-documented）



## 注解 [RFC](https://wiki.php.net/rfc/attributes_v2) [Doc](https://www.php.net/manual/zh/language.attributes.php)

PHP 7

```php
class PostsController
{
    /**
     * @Route("/api/posts/{id}", methods={"GET"})
     */
    public function get($id) { /* ... */ }
}
```

PHP 8

```php
class PostsController
{
    #[Route("/api/posts/{id}", methods: ["GET"])]
    public function get($id) { /* ... */ }
}
```

现在可以用 PHP 原生语法来使用结构化的元数据，而非 PHPDoc 声明。



## 构造器属性提升 [RFC](https://wiki.php.net/rfc/constructor_promotion) [文档](https://www.php.net/manual/zh/language.oop5.decon.php#language.oop5.decon.constructor.promotion)

PHP 7

```php
class Point {
  public float $x;
  public float $y;
  public float $z;

  public function __construct(
    float $x = 0.0,
    float $y = 0.0,
    float $z = 0.0
  ) {
    $this->x = $x;
    $this->y = $y;
    $this->z = $z;
  }
}
```

PHP 8

```php
class Point {
  public function __construct(
    public float $x = 0.0,
    public float $y = 0.0,
    public float $z = 0.0,
  ) {}
}
```

更少的样板代码来定义并初始化属性。



## 联合类型 [RFC](https://wiki.php.net/rfc/union_types_v2) [文档](https://www.php.net/manual/zh/language.types.declarations.php#language.types.declarations.union)

PHP 7

```php
class Number {
  /** @var int|float */
  private $number;

  /**
   * @param float|int $number
   */
  public function __construct($number) {
    $this->number = $number;
  }
}

new Number('NaN'); // Ok
```

PHP 8

```php
class Number {
  public function __construct(private int|float $number)
  {}
}

new Number('NaN'); // TypeError
```

相较于以前的 PHPDoc 声明类型的组合， 现在可以用原生支持的联合类型声明取而代之，并在运行时得到校验。



## Match 表达式 [RFC](https://wiki.php.net/rfc/match_expression_v2) [文档](https://www.php.net/manual/zh/control-structures.match.php)

PHP 7

```php
switch (8.0) {
  case '8.0':
    $result = "Oh no!";
    break;
  case 8.0:
    $result = "This is what I expected";
    break;
}
echo $result;
//> Oh no!
```

PHP 8

```php
echo match (8.0) {
  '8.0' => "Oh no!",
  8.0 => "This is what I expected",
};
//> This is what I expected
```

新的 match 类似于 switch，并具有以下功能：

- Match 是一个表达式，它可以储存到变量中亦可以直接返回。
- Match 分支仅支持单行，它不需要一个 break; 语句。
- Match 使用**严格比较**。



## Nullsafe 运算符 [RFC](https://wiki.php.net/rfc/nullsafe_operator)

PHP 7

```php
$country =  null;

if ($session !== null) {
  $user = $session->user;

  if ($user !== null) {
    $address = $user->getAddress();
 
    if ($address !== null) {
      $country = $address->country;
    }
  }
}
```

PHP 8

```php
$country = $session?->user?->getAddress()?->country;
```

现在可以用新的 nullsafe 运算符链式调用，而不需要条件检查 null。 如果链条中的一个元素失败了，整个链条会中止并认定为 Null。



## 字符串与数字的比较更符合逻辑 [RFC](https://wiki.php.net/rfc/string_to_number_comparison)

PHP 7

```php
0 == 'foobar' // true
```

PHP 8

```php
0 == 'foobar' // false
```

PHP 8 比较数字字符串（numeric string）时，会按数字进行比较。 不是数字字符串时，将数字转化为字符串，按字符串比较。



## 内部函数类型错误的一致性。 [RFC](https://wiki.php.net/rfc/consistent_type_errors)

PHP 7

```php
strlen([]); // Warning: strlen() expects parameter 1 to be string, array given

array_chunk([], -1); // Warning: array_chunk(): Size parameter expected to be greater than 0
```

PHP 8

```php
strlen([]); // TypeError: strlen(): Argument #1 ($str) must be of type string, array given

array_chunk([], -1); // ValueError: array_chunk(): Argument #2 ($length) must be greater than 0
```

现在大多数内部函数在参数验证失败时抛出 Error 级异常。



## 类型系统与错误处理的改进

- 算术/位运算符更严格的类型检测 [RFC](https://wiki.php.net/rfc/arithmetic_operator_type_checks)
- Abstract trait 方法的验证 [RFC](https://wiki.php.net/rfc/abstract_trait_method_validation)
- 确保魔术方法签名正确 [RFC](https://wiki.php.net/rfc/magic-methods-signature)
- PHP 引擎 warning 警告的重新分类 [RFC](https://wiki.php.net/rfc/engine_warnings)
- 不兼容的方法签名导致 Fatal 错误 [RFC](https://wiki.php.net/rfc/lsp_errors)
- 操作符 @ 不再抑制 fatal 错误。
- 私有方法继承 [RFC](https://wiki.php.net/rfc/inheritance_private_methods)
- Mixed 类型 [RFC](https://wiki.php.net/rfc/mixed_type_v2)
- Static 返回类型 [RFC](https://wiki.php.net/rfc/static_return_type)
- 内部函数的类型 [Email thread](https://externals.io/message/106522)
- 扩展 [Curl](https://php.watch/versions/8.0/resource-CurlHandle)、 [Gd](https://php.watch/versions/8.0/gdimage)、 [Sockets](https://php.watch/versions/8.0/sockets-sockets-addressinfo)、 [OpenSSL](https://php.watch/versions/8.0/OpenSSL-resource)、 [XMLWriter](https://php.watch/versions/8.0/xmlwriter-resource)、 [XML](https://php.watch/versions/8.0/xmlwriter-resource) 以 Opaque 对象替换 resource。



## 其他语法调整和改进

- 允许参数列表中的末尾逗号 [RFC](https://wiki.php.net/rfc/trailing_comma_in_parameter_list)、 闭包 use 列表中的末尾逗号 [RFC](https://wiki.php.net/rfc/trailing_comma_in_closure_use_list)
- 无变量捕获的 catch [RFC](https://wiki.php.net/rfc/non-capturing_catches)
- 变量语法的调整 [RFC](https://wiki.php.net/rfc/variable_syntax_tweaks)
- Namespace 名称作为单个 token [RFC](https://wiki.php.net/rfc/namespaced_names_as_token)
- 现在 throw 是一个表达式 [RFC](https://wiki.php.net/rfc/throw_expression)
- 允许对象的 ::class [RFC](https://wiki.php.net/rfc/class_name_literal_on_object)



## 新的类、接口、函数

- [Weak Map](https://wiki.php.net/rfc/weak_maps) 类
- [Stringable](https://wiki.php.net/rfc/stringable) 接口
- [str_contains()](https://wiki.php.net/rfc/str_contains)、 [str_starts_with()](https://wiki.php.net/rfc/add_str_starts_with_and_ends_with_functions)、 [str_ends_with()](https://wiki.php.net/rfc/add_str_starts_with_and_ends_with_functions)
- [fdiv()](https://github.com/php/php-src/pull/4769)
- [get_debug_type()](https://wiki.php.net/rfc/get_debug_type)
- [get_resource_id()](https://github.com/php/php-src/pull/5427)
- [token_get_all()](https://wiki.php.net/rfc/token_as_object) 对象实现
- [New DOM Traversal and Manipulation APIs](https://wiki.php.net/rfc/dom_living_standard_api)