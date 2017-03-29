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

* свойство GranteeName определяет имя принципала. Для протокола LDAP (см. далее) уникальное имя пользователя (distinguished name) представляет собой строку вида *"CN=OSMI_admin,OU=БД МИ,DC=tn,DC=bunker,DC=ru"*, где CN обозначает имя пользователя, OU - имя отдела, DC - компоненты доменного имени
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

CE поддерживает протокол службы каталогов LDAP, а значит, может быть сконфигурирован для работы с сервером LDAP, например, Windows Active Directory. Сервер AD предоставляет иерархию пользователей и групп. Зная информацию о пользователе, работающим в системе, и его группах, CE может управлять его доступом к объектам и правами.

Внутри CE пользователи и группы представлены инстансами [User](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/security/User.html) и [Group](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/security/Group.html). Им соответствуют фабричные классы [Factory.User](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/Factory.User.html) и [Factory.Group](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Factory.Group.html). Эти фабричные классы предназначены только для получнеия инстансов по id, полному или краткому имени. Создавать новые инстансы нельзя.

Класс Factory.User, кроме того, имеет метод fetchCurrent() для получения текущего пользователя.

## CREATOR-OWNER и AUTHENTICATED-USERS

В протоколе службы каталогов LDAP есть особые виртуальные (логические) сущности: пользователь `#CREATOR-OWNER` и группа `#AUTHENTICATED-USERS`. Эти названия можно использовать в качестве GranteeName:

* `#CREATOR-OWNER` обозначает пользователя, создавшего объект. Используется для прав доступа по умолчанию (dafault instanse permissions): при создании инстанса класса заменяется на реальное имя пользователя
* `#AUTHENTICATED-USERS` обозначает всех пользователей, зашедших в систему, используя логин и пароль. Поскольку группа логическая, её состав время от времени меняется автоматически

## Права доступа по умолчанию

Определения классов (инстансы ClassDefinition) имеют также свойство DefaultInstancePermissions типа AccessPermissionsList:

* `AccessPermissionList get_DefaultInstancePermissions()`
* `void set_DefaultInstancePermissions(AccessPermissionList value)`

Это ACL по умолчанию. Эти правила, если заданы, будут автоматически присвоены новому инстансу данного класса сразу после его создания.

## Примеры кода

### Назначение прав

Рассмотрим функцию, которая принимает два аргумента: документ и имя пользователя, и назначает права пользователя на документ в зависимости от членства в группах Admins, Readers или Editors.

Имя пользователя следует передавать в форме <пользователь или группа>@<домен>, например, *"BDMI_curator@tn.bunker.ru"*.

```java
// Определим маски для разных типов доступа - чтение, чтение и запись, полной контроль
private static final int READER_MASK = 
    AccessRight.READ_ACL.getValue() | 
    AccessRight.READ.getValue() | 
    AccessRight.VIEW_CONTENT.getValue();

private static final int WRITER_MASK = READER_MASK | 
    AccessRight.CREATE_CHILD.getValue() | 
    AccessRight.CREATE_INSTANCE.getValue() | 
    AccessRight.READ.getValue() | 
    AccessRight.MINOR_VERSION.getValue() | 
    AccessRight.MAJOR_VERSION.getValue() | 
    AccessRight.MODIFY_OBJECTS.getValue() | 
    AccessRight.PUBLISH.getValue() | 
    AccessRight.UNLINK.getValue() | 
    AccessRight.WRITE.getValue();

private static final int FULL_CONTROL_MASK = WRITER_MASK | 
    AccessRight.DELETE.getValue() | 
    AccessRight.WRITE_ACL.getValue() | 
    AccessRight.WRITE_OWNER.getValue();

private boolean setPermissions(Document doc, String userName)
{
   //пытаемся получить пользователя по уникальному имени
   //инстанс User нам нужен для того, чтобы узнать группы, в которых состоит пользователь
   PropertyFilter propertyFilter = new PropertyFilter();
   propertyFilter.addIncludeProperty(new FilterElement(1, null, null, "MemberOfGroups", null));
   User user
   try {
       user = Factory.User.fetchInstance(doc.getConnection(), userName, propertyFilter);
   }
   catch (EngineRuntimeException ex) {
       if (ex.getExceptionCode().equals(ExceptionCode.E_OBJECT_NOT_FOUND)) {
          return false;  //такого пользователя не существует
       } else
           throw ex;
   }
   
   AccessPermissionList apList = doc.get_Permissions();
   
   //создаём новое правило для добавления в список
   AccessPermission ap = Factory.AccessPermission.createInstance();
   ap.set_GranteeName(userName);
   ap.set_AccessType(AccessType.ALLOW); 
   ap.set_InheritableDepth(0); 
        
   GroupSet groups = user.get_MemberOfGroups();
   Iterator iter = groups.iterator();
   while (iter.hasNext()) 
   {
      Group group = (Group) iter.next();

      //устанавливаем права в зависимости от группы
      if (group.get_DisplayName().equalsIgnoreCase("Readers"))
      {
          ap.set_AccessMask(READER_MASK);
          apList.add(ap);
          break;
      }
      else if (group.get_DisplayName().equalsIgnoreCase("Writers"))
      {
         ap.set_AccessMask(WRITER_MASK);
         apList.add(ap);
         break;
      }
      else if (group.get_DisplayName().equalsIgnoreCase("Admins"))
      {
         ap.set_AccessMask(FULL_CONTROL_MASK);
         apList.add(ap);
         break;
      }
   }
   
   doc.set_Permissions(apList);
   doc.save(RefreshMode.NO_REFRESH);

   return true;
}
```

### Изменение прав по умолчнаию

В следующем примере извлекается некое определение класса (class description), и для определённой группы изменяются права по умолчанию на инстанс класса: удаляется право на чтение свойств доступа. 

```java
private static final PropertyFilter PF;
static
{
   PF= new PropertyFilter();
   PF.addIncludeProperty(new FilterElement(null, null, null, "DefaultInstancePermissions", null));
}

// плучаем некое определение класса. Нас интересует только свойство DefaultInstancePermissions
ClassDefinition cd=Factory.ClassDefinition.fetchInstance(os, new Id("{3B71BC99-FD11-4E15-9CF5-83887ED1C42F}"), PF);

AccessPermissionList apl = cd.get_DefaultInstancePermissions();
apl = cd.get_Permissions();
    	
// обход всех правил. если правило относится к нужной группе, изменяем маску доступа - обнуляем бит READ_ACL
Iterator iter = apl.iterator();
while (iter.hasNext())
{
   AccessPermission ap =  (AccessPermission)iter.next();
   if (ap.get_GranteeName().equalsIgnoreCase("LOANREVIEWERS@process.auto.acme.com"))
   {
      int rights = ap.get_AccessMask();
      rights &= ~AccessRight.READ_ACL.getValue();
      ap.set_AccessMask(rights);
      apl.add(ap);
      cd.set_Permissions(apl); 
      cd.save(RefreshMode.REFRESH);
      break;
   }
}
```

### Описание прав доступа

Имея описание класса (class description), можно получить список описаний прав доступа как свойство PermissionDescriptions типа AccessPermissionDescriptionList.

Следующий пример выводит информацию о правах для класса Folder:

```java
ClassDescription classDescription = Factory.ClassDescription.fetchInstance(objectStore, "Folder", null);
AccessPermissionDescriptionList permissionsDespriptions = classDescription.get_PermissionDescriptions();

System.out.println(String.format("%-50s%-20s%-10s\n", "Наименование", "Тип", "Маска"));
for (Object apdObject : permissionsDespriptions)
{
    AccessPermissionDescription description = (AccessPermissionDescription)apdObject;
    System.out.println(String.format("%-50s%-20s%-10s", description.get_DisplayName(),
            description.get_PermissionType(), description.get_AccessMask()));
}
```

Вывод будет таким:

```
Наименование                                      Тип                 Маска     

Полное управление                                 LEVEL               999415    
Изменить свойства                                 LEVEL               135159    
Добавить в папку                                  LEVEL               131121    
Просмотреть свойства                              LEVEL_DEFAULT       131073    
Просмотреть все свойства                          RIGHT               1         
Изменить все свойства                             RIGHT               2         
Резерв12 (Внедрение объявлено устаревшим)         RIGHT               4096      
Резерв13 (Архивирование объявлено устаревшим)     RIGHT               8192      
Включить в папку / Аннотировать                   RIGHT               16        
Вынести из папки                                  RIGHT               32        
Создать экземпляр                                 RIGHT               256       
Создать подпапку                                  RIGHT               512       
Удалить                                           RIGHT               65536     
Прочесть разрешения                               RIGHT               131072    
Изменить разрешения                               RIGHT               262144    
Изменить владельца                                RIGHT               524288    
Поддержка младших версий                          RIGHT_INHERIT_ONLY  64        
Поддержка старших версий                          RIGHT_INHERIT_ONLY  4         
Просмотреть содержимое                            RIGHT_INHERIT_ONLY  128       
Изменить состояние                                RIGHT_INHERIT_ONLY  1024      
Опубликовать                                      RIGHT_INHERIT_ONLY  2048   
```

Это перечень прав или наборов прав, которые могут быть применены к объекту - информация, полезная, например, при проектировании UI настроек безопасности. 

Свойство PermissionType может быть:

* RIGHT - одиночное право
* RIGHT_INHERIT_ONLY - одиночное право, применяемое только к дочерним классам
* LEVEL - готовый набор прав, соответствующий определённому уровню доступа
* LEVEL_DEFAULT - набор прав по умолчанию для новых правил

## Дополнительная информация

* [Working with security](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.1/com.ibm.p8.ce.dev.ce.doc/sec_procedures.htm)
