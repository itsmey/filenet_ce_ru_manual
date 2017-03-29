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
`java.lang.String get_GranteeName()` | Возвращает значение свойства GranteeName
`void set_GranteeName(java.lang.String value)` |Задает значение свойства GranteeName
`SecurityPrincipalType get_GranteeType()` |Возвращает значение свойства GranteeType
`java.lang.Integer get_InheritableDepth()` |Возвращает значение свойства InheritableDepth
`void set_InheritableDepth(java.lang.Integer value)` |Задает значение свойства InheritableDepth
`PermissionSource get_PermissionSource()` |Возвращает значение свойства PermissionSource
`AccessType get_AccessType()` |Возвращает значение свойства AccessType
`void set_AccessType(AccessType value)` |Задает значение свойства AccessType
`java.lang.Integer get_AccessMask()` |Возвращает значение свойства AccessMask
`void set_AccessMask(java.lang.Integer value)` |Задает значение свойства AccessMask

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
ADD_MARKING|Назначение маркировки объекту
CHANGE_STATE|Изменение состояния жизненного цикла объекта
CONNECT|Соединение с хранилищем объектов
CREATE_CHILD|Создание дочернего объекта
CREATE_INSTANCE|Создание нового инстанса
DELETE|Удаление объекта
LINK|Связывание объекта с помощью Link
MAJOR_VERSION|Создание старшей версии
MINOR_VERSION|Создание младшей версии
MODIFY_OBJECTS|Изменение объектов в хранилище
NONE|Остутствие любых прав
PRIVILEGED_WRITE|Изменение системных свойств (Creator, DateCreated, LastModifier, DateLastModified).
PUBLISH|Публикация объекта
READ|Просмотр свойств 
READ_ACL|Просмотр прав доступа
REMOVE_MARKING|Удаление маркировки
REMOVE_OBJECTS|Удаление объектов в хранилище
STORE_OBJECTS|Создание и размещение новых объектов в хранилище
UNLINK|Удаление связи Link с объектом
USE_MARKING|
VIEW_CONTENT|Просмотр содержимого
VIEW_RECOVERABLE_OBJECTS|Просмотр восстанавливаемых объектов в хранилище
WRITE|Изменение свойств
WRITE_ACL|Изменение свойств, отвечающих за права доступа
WRITE_ANY_OWNER|Изменение владельца объекта
WRITE_OWNER|Возможность быть владельцем объекта

Важно! К каждому типу, поддерживающиму ACL, применимы не все а только определённые действия. В частности, такие действия, как MODIFY_OBJECTS, REMOVE_OBJECTS, STORE_OBJECTS, VIEW_RECOVERABLE_OBJECTS имеют смысл только в том случае, когда применены к хранилищу объектов.

Чтобы получить целочисленное значение, применяется метод getVaue(). Рассмотрим пример конструирования битовой маски для свойства AccessMask:

```java
private static final int LOAN_CREATOR = AccessRight.READ_ACL.getValue() | AccessRight.CHANGE_STATE.getValue() 
           | AccessRight.CREATE_INSTANCE.getValue() | AccessRight.VIEW_CONTENT.getValue() 
           | AccessRight.MINOR_VERSION.getValue() | AccessRight.UNLINK.getValue()
           | AccessRight.LINK.getValue() | AccessRight.WRITE.getValue() | AccessRight.READ.getValue();
```

## Пользователи и группы

Для работы с иерархией пользователей и групп CE может быть настроен на сервер службы каталогов (directory service), например, Windows Active Directory. Внутри CE пользователи и группы представлены инстансами [User](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/security/User.html) и [Group](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/security/Group.html). Им соответствуют фабричные классы [Factory.User](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/Factory.User.html) и [Factory.Group](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Factory.Group.html). Эти фабричные классы предназначены только для получнеия инстансов по id, полному или краткому имени. Создавать новые инстансы нельзя.

Класс Factory.User, кроме того, имеет метод fetchCurrent() для получения текущего пользователя.

## CREATOR-OWNER и AUTHENTICATED-USERS

В протоколе службы каталогов LDAP есть особые виртуальные (логические) сущности: пользователь `#CREATOR-OWNER` и группа `#AUTHENTICATED-USERS`. Эти названия можно использовать в качестве GranteeName:

* `#CREATOR-OWNER` обозначает пользователя, создавшего объект. Используется для прав доступа по умолчанию (dafault instanse permissions): при создании инстанса класса заменяется на реальное имя пользователя
* `#AUTHENTICATED-USERS` обозначает пользователей, зашедших в систему, используя логин и пароль. Поскольку группа логическая, её состав время от времени меняется

## Права доступа по умолчанию

Определения классов (инстансы ClassDefinition) имеют также свойство DefaultInstancePermissions типа AccessPermissionsList:

* `AccessPermissionList get_DefaultInstancePermissions()`
* `void set_DefaultInstancePermissions(AccessPermissionList value)`

Это ACL по умолчанию. Эти правила, если заданы, будут автоматически присвоены новому инстансу данного класса сразу после его создания.

## Политики безопасности

## Примеры кода

### Назначение прав
### Изменение прав
### Проверка прав
### Описания прав доступа

## Дополнительная информация

* [Working with security](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.1/com.ibm.p8.ce.dev.ce.doc/sec_procedures.htm)
