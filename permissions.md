# Права доступа

CE имеет гибкую модель управления правами доступа к объектам самых различных типов. Объекты, поддерживающие права доступа, имеют свойство Permissions и, соответственно, методы для доступа к этому свойству:

* `AccessPermissionList get_Permissions()`
* `void set_Permissions(AccessPermissionList value)`

К примеру, такие типы CE, как ClassDefinition, ObjectStore, Containable, Link имеют свойство Permissions. Полный список можно посмотреть [здесь](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/collection/class-use/AccessPermissionList.html).

## AccessPermissionList

Это коллекция представляет собой список контроля доступа (access control list - ACL). ACL, в свою очередь, состоит из элементов контроля доступа - инстансов AccessPermission.

## [AccessPermission](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/security/AccessPermission.html)

Элемент контроля доступа - это правило, согласно которому определённому пользователю или группе разрешается или запрещается определенный набор действий над объектом.

Рассмотрим методы:

**метод** | **что делает**
------------ | -------------
java.lang.String get_GranteeName() | Возвращает значение свойства GranteeName
void set_GranteeName(java.lang.String value) |Задает значение свойства GranteeName
SecurityPrincipalType get_GranteeType() |Возвращает значение свойства GranteeType
java.lang.Integer get_InheritableDepth() |Возвращает значение свойства InheritableDepth
void set_InheritableDepth(java.lang.Integer value) |Задает значение свойства InheritableDepth
PermissionSource get_PermissionSource() |Возвращает значение свойства PermissionSource
AccessType get_AccessType() |Возвращает значение свойства AccessType
void set_AccessType(AccessType value) |Задает значение свойства AccessType
java.lang.Integer get_AccessMask() |Возвращает значение свойства AccessMask
void set_AccessMask(java.lang.Integer value) |Задает значение свойства AccessMask

