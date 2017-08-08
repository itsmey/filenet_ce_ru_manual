# Наборы маркировок

Наборы маркировок позволяют назначить объекту дополнительную защиту, поверх той, которая задается [правами доступа](permissions.md). 
Будем считать, что "базовая защита" определяется свойством Permissions, и согласно ACL, хранящемся в этом свойстве, вычисляется права текущего пользователя на объект - некоторая битовая маска.

Однако если объект обладает **маркировкой**, то на базовую защиту накладывается модификатор, определяемый
* маской ограничения маркировки
* правами текущего пользователя на данную маркировку, а если точнее, правом пользователя "использовать" данную маркировку

Давайте рассмотрим пример. Пусть есть три группы пользователей:
* читатели
* редакторы
* администраторы

Пусть есть три маркировки:

1. "Полный доступ". Маска ограничения: нет. Могут использовать: все
2. "Чтение и редактирование". Маска ограничения: запретить удаление. Могут использовать: редакторы, администраторы
3. "Только чтение". Маска ограничения: запретить редактирование, запретить удаление. Могут использовать: админстраторы

Рассмотрим документ с маркировкой "Только чтение". Маркировку могут использовать только администраторы, значит, для них права на документ определяются своством Premissions. Читатели и редакторы теряют возможность редактровать и удалять документ.

На документ документ с "Полным доступом" все пользователи имеют свои "базовые права". Документ "Чтение и редактирование" не могут удалить читатели.

**Набор маркировок** - совокупность логически связанных маркировок, как в примере. Набор маркировок можно привязать к свойству типа String. Значение этого свойства считается текущей маркировкой объекта. 

Разумеется, значение свойства можно изменить на другую маркировку, и в этом случае модификатор защиты поменяется.
Также объект может иметь сразу несколько маркировок (если несколько его свойств привязаны к наборам маркировок). Тогда маски ограничения объединяются.

## В CE

Наборы маркировок определяются на уровне домена (Domain):

`MarkingSetSet get_MarkingSets()`

Коллекция MarkingSetSet состоит из наборов маркировок [MarkingSet](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/security/MarkingSet.html). MarkingSet, в свою очередь, состоит из маркировок [Marking](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/security/Marking.html)

Маркировка имеет следующие свойства:

**свойство** | **значение**
------------ | -------------
`Id Id` | Идентификатор маркировки. Нельзя изменить
`String MarkingValue` |Значение маркировки, например, "Только чтение"
`int ConstraintMask` |Битовая маска ограничения. "1" в определенном месте означает, что это право необходимо исключить ([вычесть](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.0/com.ibm.p8.ce.dev.prop.doc/props_Marking.htm#MarkingUseGranted)) для пользователя, не использующего маркировку. См. AccessRight
`AccessPermissionList Permissions` |Список прав доступа, определяющий, что и какие пользователи могут делать с этой маркировкой. Состоит из правил AccessPermission, которые рассмотрены [здесь](permissions.md). В битовых масках правил имеют смысл только три бита: ADD_MARKING (может ли пользователь назначить эту маркировку), REMOVE_MARKING (сможет ли снять ее) и USE_MARKING (может ли использовать объект с такой маркировкой)

Метод

`int get_MarkingUseGranted()`

возвращает маску прав на маркировку для пользователя, от чьего имени вызывается метод. В маске могут быть установлены биты ADD_MARKING, REMOVE_MARKING, USE_MARKING. Этот метод в чём-то аналогичен `IndependentlyPersistableObject.getAccessAllowed()`.

## Привязка набора маркировок к свойству

Как уже было сказано, наборы маркировок определяются на уровне домена. Каждое хранилище в домене может использовать наборы домена. Домен имеет свойство **MarkingSets** - коллекция, содержащая все наборы маркировок.

Шаблон свойства типа String (PropertyTemplateString) имеет свойство **Marking** типа MarkingSet. Через это свойство можно связать шаблон с определенным набором маркировок.

Шаблон свойства имеет метод createClassProperty(), производящий определение свойства (ClassDefinition). Определение свойства может быть добавлено к определению класса (ClassDefinition). Теперь на объекты этих классов можно назначать маркировки.

Рассмотрим пример кода (*примеч. автора: пример не компилировался, это примерный набросок*):

```java
// СОЗДАНИЕ НАБОРА МАРКИРОВОК

// сначала создадим набор разрешений для маркировки
AccessPermissionList permissions = Factory.AccessPermission.createList();

// одно правило для одного пользователя
AccessPermission permission = Factory.AccessPermission.createInstance();
permission.set_GranteeName("admin@my.domain.ru");
permission.set_AccessType(AccessType.ALLOW);
permission.set_AccessMask(AccessRight.ADD_MARKING.getValue() | AccessRight.REMOVE_MARKING.getValue() | AccessRight.USE_MARKING.getValue());

// аналогичным образом создаются правила для других юзеров
// ...

// добавляем правила в набор
permissions.add(permission);
// ...

// создаем маркировку
Marking readOnlyMarking = Factory.Marking.createInstance();
readOnlyMarking.set_MarkingValue("Только чтение");
readOnlyMarking.set_Permissions(permissions);
readOnlyMarking.set_ConstraintMask(AccessRight.DELETE.getValue() | AccessRight.WRITE.getValue());

// таким же образом создаем и другие маркировки
// ...

// создаем набор маркировок
MarkingSet markingSet = Factory.MarkingSet.createInstance(objectStore.get_Domain());

// добавляем маркировки в набор
markingSet.get_Markings().add(readOnlyMarking);
// ...

markingSet.save(RefreshMode.NO_REFRESH)

// ПРИВЯЗКА К СВОЙСТВУ

// создадим новый шаблон свойства для нашего набора маркировок
// (это необязательно - можно привязать набор маркировок к существующему шаблону)
PropertyTemplate template = Factory.PropertyTemplateString.createInstance(objectStore());
template.set_SymbolicName("NewProperty");
LocalizedStringList nameList = Factory.LocalizedString.createList();
LocalizedString name = Factory.LocalizedString.createInstance();
name.set_LocaleName("ru");
name.set_LocalizedText("Новое свойство");
nameList.add(name);
template.set_DisplayNames(nameList);
template.set_Settability(PropertySettability.READ_WRITE);
template.set_Cardinality(Cardinality.SINGLE);
template.set_MarkingSet(markingSet); //здесь связываем набор маркировок и шаблон
template.save(RefreshMode.NO_REFRESH);

// получим определение класса, объекты которого должны обладать маркировками
ClassDefinition classDef = Factory.ClassDefinition.fetchInstance(objectStore(), "SomeClass", null);

//добавим к определением свойств класса новое, произведенное из шаблона
classDef.get_PropertyDefinitions().add(template.createClassProperty());

classDef.save(RefreshMode.NO_REFRESH);
```


