## SIM日志规范

文档版本：1.0

产品版本：${version}

------

### 1.内容目录

{toc}

------



### 2.文档信息

#### 2.1 文档目标

目标读者：开发人员。

目标：统一开发人员规范日志打印、异常码定义以及编写详细的异常码文档；让实施人员能够看懂日志，根据异常码查询异常信息可以初步解决问题。



#### 2.2 修订历史

| 时间       | 版本 | 内容 | 修订人 |
| ---------- | ---- | ---- | ------ |
| 2020-03-10 | 1.0  | 新建 | 周亮   |



#### 2.3 相关术语

| 术语 | 说明 |
| ---- | ---- |
|      |      |

### 3. 日志说明

#### 3.1 操作说明

打印日志时操作如下（ERROR级别日志必须如下操作）：

1. 在ExceptionConsts类中定义异常码，以及简要的异常描述信息。

2. 在com.bizenit.idm.sim.commons.exception目录下查找与第一步关联的自定义异常类，不存在则创建。

3. 维护【SIM异常码参考】文档，文档机构如下：

   | 序号 | 异常名称      | 是否运行时异常 | 异常编号 | 异常说明 | 解决方案 |
   | ---- | ------------- | -------------- | -------- | -------- | -------- |
   | 1    | XxxxException | 否             | E0101    | xxxx     | xxxx     |

- 异常码定义类


```java
com.bizenit.idm.sim.commons.consts.ExceptionConsts
```

- 异常类存放目录


```java
com.bizenit.idm.sim.commons.exception
```

- API接口异常码定义类


```java
com.bizenit.idm.sim.commons.consts.RestCodeConsts
```

#### 3.2 异常码规范

异常码结构：日志级别+模块+序号（示例：E0101）

日志级别(一位)：ERROR：E，WARN：W，INFO：I，DEBUG：D，TRACE：T

业务模块(两位)：用户：01，权限：02 ，系统设置：03  ......（后续梳理）

序号(两位)：两位数序号

#### 3.3 日志格式

- 日志信息格式：异常码+异常描述+参数

```java
LOG.error("E0101|GroupType not found|[userId:[{}],GroupTypeId:[{}]]",userId,GroupTypeId, new Exception("GroupType not found"));
```

如下图：E0101|GroupType not found|[userId:[U123123123],GroupTypeId:[100025]]

![1583914030683](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583914030683.png)

- 抛异常消息格式：异常描述+参数

```java
throw new QueryNotFoundException("Attribute named is not found in db|[userId:[%s],name:[%s]]",userId,name);
```

#### 3.4 日志规则

1. 尽量打印关键业务参数，如操作用户ID、业务ID等；
2. 不能打印密码、电话号、身份证号等私密信息；
3. 多个异常码可以对应一个异常，在抛出异常时尽量将异常信息描述清楚；
4. 接口输入参数需要用TRACE打印，输出参数根据实际情况。

### 4. 模块层级日志处理

1. Dao层、工具类层不允许将异常吃掉；
2. 业务处理层需要处理完所有的异常，不能将异常抛到用户页面。

#### 4.1 Action层异常

注意：必须在Action层将所有异常处理完成，不能将异常再次抛出。

示例：com.bizenit.idm.sis.web.app.AppDataAction

![1583831256322](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583831256322.png)

#### 4.2 Service层异常

注意：尽量在Service层将所有异常处理完成。

示例：com.bizenit.idm.sim.identity.ldap.LdapUserAdapterImpl

![1583829529477](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583829529477.png)



#### 4.3 API业务异常

注意：必须在API Service所有异常处理完成，不能将异常再次抛出。

示例：com.bizenit.idm.sim.rest.api.v3.FlowApiService

![1583830988723](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583830988723.png)

#### 4.4 Dao层异常

注意：该层不允许将异常捕获之后不抛出。

示例：com.bizenit.idm.sim.persists.appresource.impl.AppResourceDaoImpl

![1583830399607](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583830399607.png)

#### 4.5 工具类

注意：该层不允许将异常捕获之后不抛出。

示例：com.bizenit.idm.sim.commons.utils.StringUtil

![1583894329979](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583894329979.png)

### 5.通用基础异常

以下为常用的基础异常类；在底层抛出基础异常时，需要尽量将异常信息、参数信息描述清楚可读。

如果调用的方法抛出了运行时异常，那么在业务层必须处理。

#### 5.1 查询异常

异常类：com.bizenit.idm.sim.commons.exception.QueryNotFoundException

使用示例：com.bizenit.idm.sim.persists.impl.AttributeDaoImpl

![1583916531542](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583916531542.png)

#### 5.2 数据存储异常

异常类：com.bizenit.idm.sim.commons.exception.PersistsException

使用示例：com.bizenit.idm.sim.persists.impl.GroupTypeDaoImpl

![1583896239259](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583896239259.png)



#### 5.3 删除失败异常

异常类：com.bizenit.idm.sim.commons.exception.DeleteFailException

使用示例：com.bizenit.idm.sim.persists.impl.GroupTypeDaoImpl

![1583896444836](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583896444836.png)

#### 5.4 邮件发送异常

异常类：org.springframework.mail.MailException；为运行时异常

使用示例：com.bizenit.idm.sim.service.mail.MailSenderServiceImpl

![1583898634105](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583898634105.png)

JavaMailSenderImpl.send 方法API文档如下：

![1583917174989](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583917174989.png)



#### 5.5 邮件信息异常

异常类：javax.mail.MessagingException

使用示例：com.bizenit.idm.sim.service.mail.MailSenderServiceImpl

![1583834128894](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583834128894.png)

#### 5.6 LDAP连接异常

异常类：com.bizenit.idm.sim.identity.provider.ConnectionException

使用示例：com.bizenit.idm.sim.identity.ldap.LdapGroupAdapterImpl.addUserToGroup

![1583832657882](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583832657882.png)

#### 5.7 LDAP操作异常

异常类：com.unboundid.ldap.sdk.LDAPException

使用示例：com.bizenit.idm.sim.identity.ldap.LdapGroupAdapterImpl.removeUrlFromGroup

![1583833035547](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583833035547.png)

#### 5.8 LDAP查询异常

异常类：com.unboundid.ldap.sdk.LDAPSearchException

使用示例：com.bizenit.idm.sim.identity.ldap.LdapCommonAdapterImpl

![1583833197890](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583833197890.png)

#### 5.9 日期转换异常

异常类：java.text.ParseException

使用示例：com.bizenit.idm.sim.service.appresource.impl.ITSubAccountServiceImpl

![1583832350435](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583832350435.png)

#### 5.10 编码转换异常	

异常类：java.io.UnsupportedEncodingException

使用示例：com.bizenit.idm.sim.service.mail.MailSenderServiceImpl

![1583832506613](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583832506613.png)

#### 5.11 数字转化异常

异常类：java.io.NumberFormatException

使用示例：com.bizenit.idm.sim.identity.ldap.LdapProviderImpl

![1583832256632](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583832256632.png)

#### 5.12 网络IO异常

异常类：java.io.IOException

使用示例：com.bizenit.idm.sim.service.rest.implApiServiceImpl.authAndCheckSign

![1583831915011](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583831915011.png)

#### 5.13 未找到对应类异常

异常类：java.lang.ClassNotFoundException

使用示例：com.bizenit.idm.sim.service.appresource.impl.ITAccountSyncServiceImpl.accountFilter

![1583832157272](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583832157272.png)

#### 5.14 定制服务异常

异常类：com.bizenit.idm.sim.service.exception.ServiceException

使用示例：com.bizenit.idm.sim.service.appresource.filter.ITAccountAddOperDefaultFilter

![1583833456268](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583833456268.png)

#### 5.15 汉字拼音转换异常

异常类：net.sourceforge.pinyin4j.format.exception.BadHanyuPinyinOutputFormatCombination

使用示例：com.bizenit.idm.sim.service.identity.ldap.LdapUserServiceImpl.checkUserAttrs

![1583833575578](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583833575578.png)

#### 5.16 base64转码异常

异常类：org.apache.ws.security.WSSecurityException

使用示例：com.bizenit.idm.sim.identity.ldap.LdapGroupAdapterImpl

![1583833843454](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583833843454.png)

#### 5.17 连接器异常

异常类：com.bizenit.idm.sim.prov.exception.ConnectorException

使用示例：com.bizenit.idm.sim.service.icf.impl.ObjectConnectorOperationImpl

![1583834367975](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583834367975.png)

#### 5.18 证书异常

异常类：GeneralSecurityException

使用示例：com.bizenit.idm.sim.identity.ldap.LdapUserAdapterImpl

![1583834480957](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583834480957.png)

#### 5.19 非法属性异常

异常类：com.bizenit.idm.sim.rest.exception.IllegalParamException

使用示例：com.bizenit.idm.sim.rest.utils.ParamCheckUtils

![1583898884006](C:\Users\mayn\AppData\Roaming\Typora\typora-user-images\1583898884006.png)