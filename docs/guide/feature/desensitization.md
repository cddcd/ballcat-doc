# 脱敏工具

为防止隐私或敏感数据的泄露，项目开发中经常需要对特定的数据进行脱敏处理，BallCat 为此提供了一个脱敏工具包


## 依赖安装

此工具包已发布到 Maven 中央仓库，可以将以下内容添加到您的 POM 文件中以使用

```xml-vue
<dependency>
  <groupId>org.ballcat</groupId>
  <artifactId>ballcat-desensitize</artifactId>
  <version>{{ $frontmatter.ballcatVersion }}</version>
</dependency>
```

## 脱敏类型

根据不同的脱敏方式，BallCat 划分出了四种脱敏类型

### Simple（简单脱敏）

只需根据原始数据即可推出脱敏后的数据的处理方式。

> 例如简单粗暴的将数据替换为 6个 `*` 号的处理方式：`a1234` => `******`  
> 或者将 IPv4 后几位替换为 `*` ：`192.168.2.1`  => `192.*.*.*` 

### Regex （正则脱敏）

将原始数据根据正则表达式进行替换处理。

> 例如邮箱处理后第一个字符和'@'之后的原文显示，中间的显示为4个*：`12123124213@qq.com` => `1****@qq.com`

### Slide（滑动脱敏）

根据设置的左右明文数来控制明文展示，剩余的全部替换为 *，支持反转规则，好处在于会保留原始数据位数，提升辨析度。

> 例如手机号处理，明文保持前三后二：`15805516789` => `158******89`，反转结果为`***055167**`

### Rule（基于规则脱敏）

根据设置的index规则控制明文展示，剩余的全部替换为 *，支持反转规则。

> 例如指定规则位"3-6，8，10-"表示第4，5，6，7，9，11以及11之后的位替换处理：`43012319990101432X` => `430****9*9********`，反转规则则结果为`***1231*9*0101432X`

## 使用方法


### 基于 DesensitizationUtil

为方便使用，Ballcat 提供了工具类 `DesensitizationUtil`，内置了一些常用的脱敏方法。

- 支持常用类型的脱敏：如姓名、身份证、银行卡号、手机号、密码、加密密文、座机、邮箱、地址、IP等
- 支持自定义前后保留多少位脱敏，可自定义脱敏占位符
- 支持基于自定义规则的脱敏：如指定"3-6，8，10-"表示第4，5，6，7，9，11以及11之后的位使用加密字符替换

#### 姓名脱敏

中文姓名只显示第一个姓和最后一个汉字（单名则只显示最后一个汉字），其他隐藏为星号
```java
DesensitizationUtil.desensitizeChineseName("张梦") = "*梦"
DesensitizationUtil.desensitizeChineseName("张小梦") = "张*梦"
```

#### 身份证号脱敏

身份证(18位或者15位)显示前六位, 四位，其他隐藏。
```java
DesensitizationUtil.desensitizeIdCardNo("43012319990101432X") = "430123********432X"
```

#### 手机号脱敏

移动电话前三位，后四位，其他隐藏
```java
DesensitizationUtil.desensitizePhoneNumber("13812345678") = "138****5678"
```

#### 地址脱敏

地址脱敏，只显示到地区，不显示详细地址
```java
DesensitizationUtil.desensitizeAddress("北京市西城区金城坊街2号") = "北京市西城区******"
```

#### 邮箱脱敏

电子邮箱脱敏，邮箱前缀最多显示前1字母，前缀其他隐藏，用星号代替，@及后面的地址显示
```java
DesensitizationUtil.desensitizeEmail("test.demo@qq.com") = "t****@qq.com"
```

#### 银行卡号脱敏

银行卡号脱敏，显示前六位后四位
```java
DesensitizationUtil.desensitizeBankCardNo("62226000000043211234") = "622260**********1234"
```

#### 密码脱敏

密码脱敏，用******代替
```java
DesensitizationUtil.desensitizePassword(password) = "******"
```

#### IP脱敏

IPv脱敏，支持IPv4和IPv6
```java
DesensitizationUtil.desensitizeIP("192.168.2.1") = "192.*.*.*"
DesensitizationUtil.desensitizeIP("2001:0db8:02de:0000:0000:0000:0000:0e13") = "2001:*:*:*:*:*:*:*"
```

#### 加密密文脱敏

密文脱敏，前3后2，中间替换为 4个 *
```java
DesensitizationUtil.desensitizeKey("0000000123456q34") = "000****34"
```

#### 基于简单脱敏处理器脱敏

根据简单类型脱敏处理器脱敏。
```java
DesensitizationUtil.desensitizeBySimpleHandler(password, SixAsteriskDesensitizationHandler.class) = "******"
```

#### 基于正则脱敏

根据`RegexDesensitizationTypeEnum`枚举类型脱敏
```java
DesensitizationUtil.desensitizeByRegex("test.demo@qq.com", RegexDesensitizationTypeEnum.EMAIL) = "t****@qq.com"
```

#### 基于正则脱敏（自定义正则）

自定义替换正则和替换模板。
```java
DesensitizationUtil.desensitizeByRegex("test.demo@qq.com", "(^.)[^@]*(@.*$)", "$1****$2") = "t****@qq.com"
```

#### 基于滑动脱敏

根据`SlideDesensitizationTypeEnum` 枚举类型脱敏，支持反转
```java
DesensitizationUtil.desensitizeBySlide("01089898976", SlideDesensitizationTypeEnum.PHONE_NUMBER) = "010******76"

DesensitizationUtil.desensitizeBySlide("01089898976", SlideDesensitizationTypeEnum.PHONE_NUMBER, true) = "***898989**"
```

#### 基于滑动脱敏（灵活参数）

不受`SlideDesensitizationTypeEnum` 枚举类型限制，可自定义左边和右边长度以及替换字符。
```java
DesensitizationUtil.desensitizeBySlide("Hello World", 2, 3) = "He******rld"

DesensitizationUtil.desensitizeBySlide("Hello World", 2, 3, true) = "**llo Wo***"

DesensitizationUtil.desensitizeBySlide("Hello World", 2, 3, "#") = "He######rld"
```

#### 基于规则脱敏

支持自定义index规则（集）脱敏
```java
DesensitizationUtil.desensitizeByRule("43012319990101432X", "1", "4-6", "9-")) = "4*01***99*********"

DesensitizationUtil.desensitizeByRule("43012319990101432X", true, "1", "4-6", "9-")) = "4*01***99*********"
```


### 基于脱敏处理器

DesensitizationUtil 是在脱敏处理器之上做的一层封装，用户也可以直接使用脱敏处理器进行脱敏处理。

BallCat 提供了 `DesensitizationHandlerHolder` 类，来对系统内的所有脱敏处理器进行归集，方便使用的时候直接获取，而不必再创建新的对象。

#### 简单脱敏

对于简单脱敏类型，BallCat 只内置了`SixAsteriskDesensitizationHandler`和`IPDesensitizationHandler`脱敏处理器。
`SixAsteriskDesensitizationHandler`不管原文是什么，一律返回6个 *，`IPDesensitizationHandler`则对IP地址进行脱敏处理。
:::warning 提示
使用者可以定制自己的简单脱敏处理器，具体步骤请参看扩展使用一节。
:::


使用示例：

```java
// 获取简单脱敏处理器
SimpleDesensitizationHandler desensitizationHandler = 
        DesensitizationHandlerHolder.getSimpleHandler(SixAsteriskDesensitizationHandler.class);
String origin = "你好吗？";  // 原始字符串
String target = desensitizationHandler.handle(origin); // 替换处理
System.out.println(target);  // 结果：******

SimpleDesensitizationHandler desensitizationHandler = 
        DesensitizationHandlerHolder.getSimpleHandler(IPDesensitizationHandler.class);
String origin = "192.168.2.1";  // 原始字符串
String target = desensitizationHandler.handle(origin); // 替换处理
System.out.println(target);  // 结果：192.*.*.*
String origin = "2001:0db8:02de:0000:0000:0000:0000:0e13";  // 原始字符串
String target = desensitizationHandler.handle(origin); // 替换处理
System.out.println(target);  // 结果：2001:*:*:*:*:*:*:*
```

#### 正则脱敏

正则脱敏使用时，除了原始字符串之外，还需要提供正则表达式，以及占位替换表达式，以便处理数据。

使用示例：

```java
  // 获取正则脱敏处理器
  RegexDesensitizationHandler desensitizationHandler =
          DesensitizationHandlerHolder.getRegexDesensitizationHandler();
  String origin = "12123124213@qq.com"; // 原始字符串
  String regex = "(^.)[^@]*(@.*$)";    // 正则表达式
  String replacement = "$1****$2";     // 占位替换表达式
  String target = desensitizationHandler.handle(origin, regex, replacement); // 替换处理
  System.out.println(target);  // 结果：1****@qq.com
```

由于 BallCat 默认提供了 Email 类型的脱敏正则，所以可以使用以下代码来简化正则的定义：

```java
  // 使用内置的正则脱敏类型
  String target2 = desensitizationHandler.handle(origin, RegexDesensitizationTypeEnum.EMAIL);
  System.out.println(target2);  // 结果：1****@qq.com
```



#### 滑动脱敏

滑动脱敏则除了原始字符串之外，还需要提供左边和右边各自需要展示的明文数量，明文数量可以为 0。支持反转结果。

使用示例：

```java
  // 获取滑动脱敏处理器
  SlideDesensitizationHandler desensitizationHandler =
          DesensitizationHandlerHolder.getSlideDesensitizationHandler();
  String origin = "15805516789"; // 原始字符串
  String target1 = desensitizationHandler.handle(origin, 3, 2); // 替换处理
  System.out.println(target1);  // 结果：158******89
```

和正则脱敏一样，由于 BallCat 默认提供了 PhoneNumber 类型的滑动规则，所以可以使用以下代码来简化：

```java
  // 使用内置的滑动脱敏规则
  String target2 = desensitizationHandler.handle(origin, SlideDesensitizationTypeEnum.PHONE_NUMBER);
  System.out.println(target2); // 结果：158******89
```


#### 基于规则脱敏

基于规则脱敏则除了原始字符串之外，还需要提供需要脱敏的index规则（集）。支持反转规则。

使用示例：

```java
  // 获取基于规则脱敏处理器
  RuleDesensitizationHandler desensitizationHandler = DesensitizationHandlerHolder
      .getRuleDesensitizationHandler();
  String origin = "43012319990101432X"; // 原始字符串
  String target1 = desensitizationHandler.handle(origin, "1", "4-6", "9-"); // 替换处理
  System.out.println(target1);  // 结果：4*01***99*********

  String target2 = desensitizationHandler.handle(origin, true, "1", "4-6", "9-"); // 替换处理
  System.out.println(target2);   // 结果：*3**231**90101432X
```




### 基于注解的脱敏  

目前的注解脱敏处理实现基于 Jackson。
  
#### 注解分类

- `@JsonSimpleDesensitize` ：简单类型脱敏
- `@JsonRegexDesensitize`：正则类型脱敏
- `@JsonSlideDesensitize`：滑动类型脱敏
- `@JsonRuleDesensitize`：规则类型脱敏

#### 注解添加

在需要进行脱敏处理的实体属性上添加对应的脱敏注解：

```java
@Data
@Accessors(chain = true)
public class DesensitizationUser {

	/**
	 * 用户名，不脱敏
	 */
	private String username;

	/**
	 * 密码脱敏
	 */
	@JsonRegexDesensitize(type = RegexDesensitizationTypeEnum.ENCRYPTED_PASSWORD)
	private String password;

	/**
	 * 邮件
	 */
	@JsonRegexDesensitize(type = RegexDesensitizationTypeEnum.EMAIL)
	private String email;

	/**
	 * 手机号
	 */
	@JsonSlideDesensitize(type = SlideDesensitizationTypeEnum.PHONE_NUMBER)
	private String phoneNumber;

	/**
	 * 测试自定义脱敏
	 */
	@JsonSimpleDesensitize(handler = TestDesensitizationHandler.class)
	private String testField;

	/**
	 * 测试规则脱敏
	 */
	@JsonRuleDesensitize(rule = { "1", "4-6", "9-" })
	private String ruleDesensitize;

	/**
	 * 测试规则脱敏（反转）
	 */
	@JsonRuleDesensitize(rule = { "1", "4-6", "9-" }, reverse = true)
	private String ruleReverseDesensitize;
}
```

#### 定义 Json 序列化修改器

如果需要使用 json 注解脱敏，则需要将 BallCat 提供的 `JsonSerializerModifier` 注册到 Jackson 的 `ObjectMapper` 中。

示例如下：

```java
// 1.创建 ObjectMapper 对象
ObjectMapper objectMapper = new ObjectMapper();
// 2.实例化 JsonSerializerModifier
JsonSerializerModifier modifier = new JsonSerializerModifier();
// 3.将自定义序列化构建器 注册进 ObjectMapper
objectMapper.setSerializerFactory(objectMapper.getSerializerFactory().withSerializerModifier(modifier));
```

如果是 Spring Boot 项目，推荐使用以下方式进行 JSON 脱敏序列化修改器的注册。

```java
// 注册 Jackson 的脱敏模块
@Bean
public JsonDesensitizeModule jsonDesensitizeModule() {
    JsonDesensitizeSerializerModifier desensitizeModifier = new JsonDesensitizeSerializerModifier();
    return new JsonDesensitizeModule(desensitizeModifier);
}
```


#### 脱敏操作
:::warning 提示
Spring 项目可以直接通过注入的方式获取 ObjectMapper 实例。
::: 

```java
void test(){
   DesensitizationUser user = new DesensitizationUser()
        .setEmail("chengbohua@foxmail.com")
        .setUsername("xiaoming")
		.setPassword("admina123456")
        .setPhoneNumber("15800000000")
        .setTestField("这是测试属性")
		.setRuleDesensitize("43012319990101432X")
		.setRuleReverseDesensitize("43012319990101432X");;
	String value = objectMapper.writeValueAsString(user);
    Assert.isTrue("{\"username\":\"xiaoming\",\"password\":\"adm****56\",\"email\":\"c****@foxmail.com\",\"phoneNumber\":\"158******00\",\"testField\":\"TEST-这是测试属性\",\"ruleDesensitize\":\"4*01***99*********\",\"ruleReverseDesensitize\":\"*3**231**90101432X\"}"
						.equals(value));
}
```



#### Web 脱敏

在 SpringMvc（SpringBoot） 项目中，在 ObjectMapper 中完成了注册并在实体字段上添加了对应注解，就已经完成了脱敏处理。

当响应的 Json 数据类型为该实体时，即会自动进行脱敏处理。

> 注意，这里需要保证 SpringMvc 中使用的 ObjectMapper 注册了脱敏处理器



## 扩展使用

### 自定义简单类型脱敏处理器

a) 首先定义自己的 `SimpleDesensitizationHandler` 实现类:

```java
public class SimpleDesensitizatioHanderSPIExample implements SimpleDesensitizationHandler {
    @Override
    public String handle(String s) {
        return "------";
    }
}
```

b) BallCat 会利用 java 的 SPI 机制来加载简单脱敏处理器

所以只需在项目的 `resources/META-INF/services` 目录下新建名为

`org.ballcat.desensitize.handler.SimpleDesensitizationHandler` 的文件。

文件内容为自定义的脱敏处理器的全类名，多个实现则每个实现类名单独一行。

c) 使用示例

获取自定义处理器，入参为该处理器类目

```java
		// 获取简单脱敏处理器
		SimpleDesensitizationHandler desensitizationHandler = 
				DesensitizationHandlerHolder.getSimpleHandler(SimpleDesensitizatioHanderSPIExample.class);
```

配合 JSON 注解使用时，只需指定 Handler 类型为该类即可

```java
        /**
         * 测试自定义脱敏
         */
        @JsonSimpleDesensitize(handler = SimpleDesensitizatioHanderSPIExample.class)
        private String testField;
```


### 脱敏策略

JSON 处理时，根据某些逻辑判断是否进行脱敏，需要用户进行脱敏策略接口的实现：

```java
    public interface DesensitizeStrategy {
        /**
         * 判断是否忽略字段
         * @param fieldName {@code 当前字段名称}
         * @return @{code true 忽略 |false 不忽略}
         */
        boolean ignoreField(String fieldName);
    }
```

在注册脱敏修改器的时候，进行

```java
	// 自定义策略，当前用户是管理员时忽略 phoneNumber 字段的脱敏处理
    DesensitizeStrategy strategy = (fieldName) -> {
        return fieldName.equals("phoneNumber") && isAdmin;
    };
    JsonSerializerModifier modifier = new JsonSerializerModifier(strategy);
```


### 自定义 JSON 脱敏注解示例

#### 新增自定义注解

```java
@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface CustomerDesensitize {
	/**
	 * 类型字段
	 * @return type
	 */
	String type();
}
```

#### 注册自定义脱敏类型处理器

```java
//实现自定义脱敏处理器
CustomDesensitisedHandler customDesensitisedHandler = new CustomDesensitisedHandler();
//将自定义脱敏处理器绑定	
DesensitizationHandlerHolder.addHandler(CustomDesensitisedHandler.class, customDesensitisedHandler);
```

#### 注册注解处理器

```java
//注册注解 处理器
AnnotationHandlerHolder.addHandleFunction(CustomerDesensitize.class, (annotation, value) -> {
    CustomerDesensitize customerDesensitize= (CustomerDesensitize) annotation;
    String type = customerDesensitize.type();
    log.info("注解上的参数{}",type);
    CustomDesensitisedHandler handler = (CustomDesensitisedHandler) DesensitizationHandlerHolder.getHandler(CustomDesensitisedHandler.class);
    return handler.handle(value);
});
```

#### 在实体字段上指定自定义注解

```java
@CustomerDesensitize(type = "自定义注解")
private String customDesensitize;
```
