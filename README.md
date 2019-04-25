# MyBatis Type Handlers for Encrypt

[![Build Status](https://www.travis-ci.org/drtrang/typehandlers-encrypt.svg?branch=master)](https://www.travis-ci.org/drtrang/typehandlers-encrypt)
[![Coverage Status](https://coveralls.io/repos/github/drtrang/typehandlers-encrypt/badge.svg?branch=master)](https://coveralls.io/github/drtrang/typehandlers-encrypt?branch=master)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.github.drtrang/typehandlers-encrypt/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.github.drtrang/typehandlers-encrypt)
[![License](http://img.shields.io/badge/license-apache%202-brightgreen.svg)](https://github.com/drtrang/typehandlers-encrypt/blob/master/LICENSE)

## Introduction
At the request of the company's security department, sensitive information in the database needs to be encrypted. Due to the large number of fields involved in this demand, manual encryption and decryption is quite inconvenient and the changes are large. A simpler and more versatile solution is imperative.

The `typehandlers-encrypt`project was born in this context. Users do not need to change the business code when using it. Only a small amount of configuration can be used to encrypt and decrypt the specified fields of the database, greatly reducing the impact on users.


## Principle of implementation

Typehandlers-encrypt Based on MyBatis's TypeHandler development, the property that can be converted between JavaType and JdbcType by TypeHandler intercepts SQL with JavaType com.github.trang.typehandlers.alias.Encrypt, when setting parameters in prepared statement (PreparedStatement) Automatically encrypts and automatically decrypts when the value is in the result set (ResultSet).

Note: Because it depends on MyBatis, you need to register EncryptTypeHandler and Encrypt to MyBatis, otherwise it will not take effect. For the registration method, please declare EncryptTypeHandler.

## application
### POM
```xml
<dependency>
    <groupId>com.github.drtrang</groupId>
    <artifactId>typehandlers-encrypt</artifactId>
    <version>1.1.1</version>
</dependency>
```

### Declare EncryptTypeHandler
#### 1. Use MyBatis alone
```xml
<!-- mybatis-config.xml -->
<typeAliases>
    <package name="com.github.trang.typehandlers.alias" />
</typeAliases>

<typeHandlers>
    <package name="com.github.trang.typehandlers.type" />
</typeHandlers>
```

#### 2. Combine with Spring
```java
@Bean
public SqlSessionFactory sqlSessionFactory(Configuration config) {
    SqlSessionFactoryBean factory = new SqlSessionFactoryBean();
    factory.setTypeAliasesPackage("com.github.trang.typehandlers.alias");
    factory.setTypeHandlersPackage("com.github.trang.typehandlers.type");
    return factory.getObject();
}
```

#### 3. Combine with SpringBoot
```yaml
##application.yml
mybatis:
    type-aliases-package: com.github.trang.typehandlers.alias
    type-handlers-package: com.github.trang.typehandlers.type
```

Note: You can choose one of the above configuration methods, please choose according to the actual situation.

### Use EncryptTypeHandler


declare `javaType="encrypt"` on the field that needs to be encrypted in SQL

```xml
<!-- declare `javaType="encrypt"` on the field that needs to be encrypted in resultMap or SQL -->
<resultMap id="BaseResultMap" type="user">
    <id column="id" property="id" jdbcType="BIGINT" />
    <result column="username" javaType="string" jdbcType="VARCHAR" property="username" />
    <result column="password" javaType="encrypt" jdbcType="VARCHAR" property="password" />
</resultMap>

<!-- Declare `javaType="encrypt"` on the field that needs to be encrypted in SQL -->
<insert id="insert" parameterType="user">
    insert into user (id, username, password)
    values (#{id,jdbcType=BIGINT}, #{username,jdbcType=VARCHAR}, #{password, javaType=encrypt, jdbcType=VARCHAR})
</insert>

<1-- declare `javaType="encrypt"` on the field that needs to be encrypted in SQL -->
<update id="update" parameterType="user">
    update user set password=#{password, javaType=encrypt, jdbcType=VARCHAR} where id=#{id}
</update>
```


## Advanced
Typehandlers-encrypt has built-in AES encryption algorithm and default 16-bit key, which is available out of the box, but users can also customize the encryption algorithm and key, just need to declare the corresponding configuration in the configuration file. It should be noted that when both are configured, they must be declared in the same file.

Configuration example:

```properties
encrypt.private.key=xxx
encrypt.class.name=com.github.trang.typehandlers.crypt.SimpleEncrypt
```

### Profile lookup
Method 1: When the project starts, it will search for the Properties file named encrypt, properties/config-common, properties/config, config, and application in the classpath of the project until the file exists and the file contains the name encrypt.private.key. The properties are stopped.

Method 2: If the above file does not exist in the project and you do not want to add it separately, you can also call ConfigUtil.bundleNames("xxx") to specify the file to be read when the project starts. Only the file given by the user will be used. Find in.

**When the corresponding configuration is not found, the project uses the built-in default configuration.**

### Custom key
Typehandlers-encrypt supports custom keys, just declare them in the configuration file.
```properties
encrypt.private.key=xxx
```

### Custom encryption algorithm
Typehandlers-encrypt The default encryption algorithm is AES symmetric encryption. If the default algorithm does not meet the actual requirements, the user can implement the com.github.trang.typehandlers.crypt.Crypt interface and declare the full path of the implementation class in the configuration file.
```properties
encrypt.class.name=com.github.trang.typehandlers.crypt.SimpleEncrypt
```

Hard and wide
The project is currently open source and uploaded to Github. If you are interested, you can read the source code. There is a demo Demo demo typelerlers-encrypt-demo in Github, including the complete use of typehandlers-encrypt, which can be used as a reference.

If you have any questions, you can mention ISSUE on Github, or QQ. The following are the contact methods:
My Github home page: https://github.com/drtrang
Github address for this project: https://github.com/drtrang/typehandlers-encrypt
Github address for Demo: https://github.com/drtrang/typehandlers-encrypt-demo
BeanCopier tool: https://github.com/drtrang/Copiers
QQ: 349096849

note:

Currently EncryptTypeHandler only supports JavaType as String. If you have other requirements, please contact me.
When encrypting, the field only filters out null values, and non-null fields are directly encrypted without any processing.
