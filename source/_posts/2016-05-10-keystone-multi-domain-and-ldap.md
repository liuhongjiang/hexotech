title: keystone设置多个domain和使用ldap作为identity
date: 2016-05-10 17:38:00
tags:
categories: openstack
---

最近的一个项目中，需要对openstack的keystone做一些配置，目的是实现使用公司的账号登录openstack

公司的账号存放在Active Directory中，但是AD权限是只读的。

目前openstack的keystone是支持identity和assignment分离的，所以参考了这篇文章进行的配置： [KEYSTONE: LDAP FOR IDENTITY, SQL FOR ASSIGNMENT](http://www.mattfischer.com/blog/?p=545)


最终的效果是：添加一个新的domain，这个domain的用户信息是保存在AD (Active Directory)，这个domain的用户可以使用ldap的用户名和密码在horizon中登录。

<!--more-->

## 1. Enable domain-specific drivers

  添加 /etc/keystone/keystone.conf 以下配置

  ```
  [identity]
  domain_specific_drivers_enabled = True
  domain_config_dir = /etc/keystone/domains
  ```

## 2. 添加ldap的配置

  在`/etc/keystone/domains/keystone.{DOMAIN_NAME}.conf`文件中添加ldap的配置

  {DOMAIN_NAME}需要和真正的domain name保持一致

  下面是ldap的一个配置实例， 需要重点关注的是：

  指定ldap为identity的driver, 然后在ldap的section中配置ldap的参数

  ```
  [identity]
  driver = keystone.identity.backends.ldap.Identity

  [ldap]
  .....
  ```

  指定sql为assignment的driver，相当于默认使用openstack的assignment

  ```
  [assignment]
  driver = keystone.assignment.backends.sql.Assignment
  ```

  完整的keystone.{DOMAIN_NAME}.conf配置文件

  ```
  [identity]
  driver = keystone.identity.backends.ldap.Identity

  [ldap]
  url = ldap://<ldap host>:389
  user = CN=<account>,CN=Users,DC=corp,DC=<company dc>,DC=com
  password = <account passwd>
  suffix = DC=corp,DC=DC=<company dc>,DC=com
  use_dumb_member = False
  allow_subtree_delete = False

  query_scope = sub

  user_tree_dn = OU=user,DC=corp,DC=<company dc>,DC=com
  user_objectclass = user
  user_id_attribute = cn
  user_name_attribute = cn
  user_mail_attribute = mail
  user_filter = (&(objectClass=user)(cn=*))

  ;user_enabled_attribute = userAccountControl
  ;user_enabled_default = 512
  ;user_enabled_mask = 2
  ;user_enabled_emulation = False

  group_tree_dn = OU=group,DC=corp,DC=<company dc>,DC=com
  group_objectclass = group
  group_filter = (&(objectClass=group)(cn=*))
  group_id_attribute = cn
  group_name_attribute = name
  group_member_attribute = member

  ; Read only for user
  user_allow_create = False
  user_allow_update = False
  user_allow_delete = False

  ; Read only for group
  group_allow_create = False
  group_allow_update = False
  group_allow_delete = False

  ; open all debug log for ldap driver
  debug_level = -1

  [assignment]
  driver = keystone.assignment.backends.sql.Assignment

  ```
  完成文件后，需要注意让文件的用户和组都为keystone, `chown -R keystone:keystone /etc/keystone/domains`

  其中下面的几个配置项说明：
  ```
  ;user_enabled_attribute = userAccountControl
  ;user_enabled_default = 512
  ;user_enabled_mask = 2
  ;user_enabled_emulation = False
  ```

  user_enabled_attribute配置项, 如果directory servers没有提供了boolean attribute，就要使用mask，例如AD(active Directory). 在AD中，使用第二位代表用户是否是enabled的，所以使用mask=2.

  参考[Configuring for an LDAP Backend](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux_OpenStack_Platform/3/html/Installation_and_Configuration_Guide/Configuring_for_an_LDAP_Backend.html)
  和[How to use the UserAccountControl flags to manipulate user account properties](https://support.microsoft.com/en-us/kb/305144)

  user_enabled_emulation是一个work round，当用户的LDAP system没有提供 enabled这个属性的时候，可以用这个做为work round，方法就是创建一个cn，专门用来放那些user是enabled。

  参考这个blog：[KEYSTONE: USER ENABLED EMULATION (FOLLOW-UP)](http://www.mattfischer.com/blog/?p=550)



## 3. 创建domain

  由于是使用kolla部署的，所以在部署完成以后，会生成一个`/etc/kolla/admin-openrc.sh `文件

  source 以后，就可以使用`openstack domain create`命令了
  ```
  openstack domain create {DOMAIN_NAME}
  ```

  如果环境变量中没有openstack的认证信息(source admin-openrc.sh就会export相应的环境变量)， 那么运行`openstack domain create`命令会报这样的错误：`Unknown command ['domain']`。

  最后重启keystone的keyston的apache

## 4. 为domain添加project

  重启以后，使用{DOMAIN_NAME}和ldap中的用户进行登录，如果用户名和密码正确，那么horizon会报这样的错误：`You are not authorized for any projects or domains.`

  那么就要创建相应的项目，同样使用openstack cli：

  ```
  openstack project create --domain {DOMAIN_NAME} domain_default
  ```

  之后创建role `openstack role create admin`

  然后通过下面的命令将用户添加到项目中：

  ```
  openstack role add --project domain_default --user 762cb3fc4d534311caff693b5e586a2dae31a810ae9c3479f6608d39ba5feeb1 admin
  ```

  注意：`762cb3fc4d534311caff693b5e586a2dae31a810ae9c3479f6608d39ba5feeb1`是用户的uuid，在ldap中只能查到用户的name，可以通过下面的命令查找用户id

  ```
  openstack user list --domain {DOMAIN_NAME} | grep {username}
  ```

## 5. 登录

  现在就应该可以从horizon中通过DOMAIN_NAME和ldap的用户名密码登录了。

  注：在horizon中需要enable multi domain的支持：

  ```
  OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True
  OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'default'
  ```


## Reference:

[openstack wiki: How to integrate Keystone with AD](https://wiki.openstack.org/wiki/HowtoIntegrateKeystonewithAD)

[Integrate Identity back end with LDAP](http://docs.openstack.org/admin-guide/keystone_integrate_identity_backend_ldap.html#integrate-identity-backend-ldap)

[Create projects, users, and roles](http://docs.openstack.org/liberty/install-guide-obs/keystone-users.html)

[KEYSTONE: LDAP FOR IDENTITY, SQL FOR ASSIGNMENT](http://www.mattfischer.com/blog/?p=545)
