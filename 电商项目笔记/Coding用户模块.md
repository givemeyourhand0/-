###Coding用户模块
####一.用户登录
**整个处理流程是这样的：UserController-->service-->Mybatis->dao**
- 1.在UserController类上加@RequestMapping("/user/")说明所有/user/都会给到该类处理，这样就不需要在UserController类的每个方法上都加@RequestMapping("/user/")注解了。

- 2.在方法前加@ResponseBody注解的作用是：方法返回值的时候，自动通过SpringMVC的jackson插件将返回值序列化成JSON。

- 3.IUserService接口，方便维护和AOP，扩展性也比较强。

- 4.ServerResponse是泛型构造的服务端响应对象，其中有不同的私有构造器，满足不同的返回需求

- 5.添加@JsonIgnore注解的方法不会将返回值加入到JSON的序列化数据中。

- 6.@JsonSerialize(include = JsonSerialize.Inclusion.NOT_NULL)注解会使JSON中不包含空值，这样status为false时，返回给前端的JSON中就不会有data了。

  ​

#### 二.用户登出

- 将session中的key为CURRENT_USER对应的value去掉即可。

  ​

#### 三.用户注册

- ​

#### 四.

  