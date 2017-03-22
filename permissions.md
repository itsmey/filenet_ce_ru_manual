# Права доступа

CE имеет гибкую модель управления правами доступа к объектам самых различных типов. Объекты, поддерживающие права доступа, имеют свойство Permissions и, соответственно, методы для доступа к этому свойству:

* `AccessPermissionList get_Permissions()`
* `void set_Permissions(AccessPermissionList value)`

К примеру, такие типы CE, как ClassDefinition, ObjectStore, Containable, Link имеют свойство Permissions. Полный список можно посмотреть [здесь](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/collection/class-use/AccessPermissionList.html).

## AccessPermissionList

Это коллекция представляет собой список контроля доступа (access control list - ACL). ACL, в свою очередь, состоит из элементов контроля доступа - инстансов AccessPermission.

## [AccessPermission](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/security/AccessPermission.html)

Элемент контроля доступа - это правило, согласно которому определённому пользователю или группе разрешается или запрещается определенный набор действий над объектом.

Рассмотрим методы и свойства:

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

* свойство GranteeName определяет полное или краткое имя пользователя или группы
* свойство GranteeType определяет, относится ли данное правило к пользователю (SecurityPrincipalType.USER) или группе (SecurityPrincipalType.GROUP). Только для чтения
* свойство PermissionSource определяет источник данного правила: дефолтные права класса при инстанцировании, наследование от родительского объекта, ручное назначение (подробнее см. [здесь](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/constants/PermissionSource.html)). Только для чтения
* свойство InheritableDepth определяет вид наследования прав доступа. Свойство имеет смысл для тех объектов, которые образуют иерархические деревья:
  * 0 - наследование отсутствует
  * 1 - только прямые потомки
  * -1 - все потомки
* AccessType - разрешительное (AccessType.ALLOW) или запретительное (AccessType.DENY) правило
* AccessMask - битовая маска, каждый бит которой определяет некое действие. Если бит выставлен в 1, действие участвует в правиле, т.е. будет разрешено или запрещено, в зависимости от AccessType. 

## AccessMask

Для конструирования битовой маски прав используется вспомогательный класс [AccessRight](https://www.ibm.com/support/knowledgecenter/SSGLW6_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/constants/AccessRight.html) с набором предопределенных инстансов:

**значение** | **действие**
------------ | -------------
ADD_MARKING|Specifies that the user or group is granted or denied permission to assign a Marking object to an object.
CHANGE_STATE|Specifies that the user or group is granted or denied permission to change the lifecycle state of an object.
CONNECT|Specifies that the user or group is granted or denied permission to connect to an object store.
CREATE_CHILD|Specifies that the user or group is granted or denied permission to create a child object.
CREATE_INSTANCE|Specifies that the user or group is granted or denied permission to create a new instance of an object.
DELETE|Specifies that the user or group is granted or denied permission to delete an object.
LINK|Specifies that the user or group is granted or denied permission to link to an object.
MAJOR_VERSION|Specifies that the user or group is granted or denied permission to create a document major version.
MINOR_VERSION|Specifies that the user or group is granted or denied permission to create a new document minor version.
MODIFY_OBJECTS|Specifies that the user or group is granted or denied permission to modify objects in an object store.
MODIFY_RETENTION|This constant is not supported.
NONE|Specifies that the user or group has no access to objects.
PRIVILEGED_WRITE|Specifies that the user or group is granted or denied permission to set certain system-level properties (Creator, DateCreated, LastModifier, DateLastModified).
PUBLISH|Specifies that the user or group is granted or denied permission to publish an object.
READ|Specifies that the user or group is granted or denied permission to view the properties of an object.
READ_ACL|Specifies that the user or group is granted or denied permission to view an object's security (that is, its PermissionList collection).
REMOVE_MARKING|Specifies that the user or group is granted or denied permission to remove a Marking object from an object.
REMOVE_OBJECTS|Specifies that the user or group is granted or denied permission to delete objects in an object store.
STORE_OBJECTS|Specifies that the user or group is granted or denied permission to create and store new objects in an object store.
UNLINK|Specifies that the user or group is granted or denied permission to unlink from an object.
USE_MARKING|Determines whether or not the constraint mask will be applied.
VIEW_CONTENT|Specifies that the user or group is granted or denied permission to view the content of an object.
VIEW_RECOVERABLE_OBJECTS|Specifies that the user or group is granted or denied permission to retrieve or query all recoverable objects in the object store.
WRITE|Specifies that the user or group is granted or denied permission to modify the properties of an object.
WRITE_ACL|Specifies that the user or group is granted or denied permission to modify an object's security (that is, its PermissionList collection).
WRITE_ANY_OWNER|Specifies that the user or group is granted or denied permission to change the ownership of an object to another user.
WRITE_OWNER|Specifies that the user or group is granted or denied permission to assume the ownership of an object.

