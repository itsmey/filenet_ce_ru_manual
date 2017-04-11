# Списки выбора

Список выбора (Choice list) - набор предопределённых значений. Списки выбора особенно удобны, когда требуется выбор из сравнительно небольшого и неизменного множества вариантов. Например:

* названия субъектов РФ
* пол: "мужчина"/"женщина"
* времена года
* и т.п.

В CE список выбора состоит, упрощенно говоря, из коллекций пар "значение для отображения" (типа String) - "внутреннее значение" (типа Integer или String).

## ChoiceList

[com.filenet.api.admin.ChoiceList](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/admin/ChoiceList.html) - собственно, список выбора. 

* определяется в хранилище объектов: свойство хранилища ChoiceLists типа ChoiceListSet - это коллекция объектов ChoiceList в хранилище.
* создавать и извлекать (по Id) инстансы можно фабричными методами: [Factory.ChoiceLists](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Factory.ChoiceList.html).
* минимальный набор данных, которые нужно задать при создании: любое количество элементов варианта, а также свойства DisplayName и DataType
* имеет свойство Permissions, т.е. можно настраивать права доступа

Свойство | Тип | Что делает
------------ | ------------- | -------------
Permissions|AccessPermissionList|Права доступа. По умолчанию список выбора имеет те же права, что и хранилище: полные права для администраторов, чтение свойств для обычных пользователей
DisplayName|String|Имя списка для UI
DataType|TypeID|Тип данных. Может быть STRING или LONG. Определяет тип значения элементов вараинта
DescriptiveText|String|Опциональное описание списка выбора
HasHierarchy|Boolean|Иерархический ли список. Список иерархический, если хотя бы один из элементов варианта в коллекции ChoiceValues является группой (имеет непустую коллекцию ChoiceValues)
ChoiceValues|com.filenet.api.collection.ChoiceList|Коллекция элементов варианта

## Choice

[Choice](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/admin/Choice.html) - элемент варианта. 

* создаётся методами фабричного класса [Factory.Choice](https://www.ibm.com/support/knowledgecenter/SSGLW6_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Factory.Choice.html). Этот же класс предоставляет метод для создания пустого списка элементов варианта com.filenet.api.collection.ChoiceList
* может быть одиночным элементом или группой элементов. Является группой, если свойство ChoiceValues не пустое. Группы позволяют создавать иерархические списки выбора
* может быть одного из четырёх типов (свойство ChoiceType):
  * INTEGER - внутреннее значение целочисленного типа. Не является группой элементов
  * MIDNODE_INTEGER - внутреннее значение целочисленного типа. Является группой элементов
  * STRING -внутреннее значение строкового типа. Не является группой элементов
  * MIDNODE_STRING - внутреннее значение строкового типа. Является группой элементов

Свойство | Тип | Что делает
------------ | ------------- | -------------
DisplayName|String|Имя для отображения в UI - то, что видит пользователь при выборе из списка
DisplayNames|LocalizedStringList|Вариант свойства DisplayName, поддерживающий различные локали
ChoiceIntegerValue|Integer|Внутреннее значение. Имеет смысл при типе элемента INTEGER или MIDNODE_INTEGER
ChoiceStringValue|String|Внутреннее значение. Имеет смысл при типе элемента STRING или MIDNODE_STRING
ChoiceType|ChoiceType|Тип элемента (см. выше)
ChoiceValues|com.filenet.api.collection.ChoiceList|Коллекция элементов варианта

## Использование в инстансах

Шаблон свойства (PropertyTemplate) имеет свойство ChoiceList типа ChoiceList. 
* к шаблону свойства типа Integer32 (PropertyTemplateInteger32) можно прикрепить список выбора типа LONG
* к шаблону свойства типа String (PropertyTemplateString) можно прикрепить список выбора типа STRING

Шаблон производит определение свойства (PropertyDefinition), а оно, в свою очередь, добавляется к определению класса. Таким образом, инстансы могут иметь свойства типа Integer или String, ассоцииорванные со списками выбора.

Кроме того, список выбора можно связать с определением свойства напрямую.

## Примеры

### Создание списка выбора

1. Создать необходимые элементы выбора Choice при момощи метода Factory.Choice.createInstance(). У каждого объекта Choice заполнить свойства
* ChoiceType
* DisplayName (если не используется локализация) или DisplayNames (если используется)
* ChoiceIntegerValue или ChoiceStringValue, в зависимости от установленного ранее ChoiceType
2. Создать пустой список com.filenet.api.collection.ChoiceList при помощи метода Factory.Choice.createList() и добавить в список ранее созданные элементы выбора
3. Создать объект com.filenet.api.admin.ChoiceList при помощи метода Factory.ChoiceList.createInstance()
4. Установить свойство ChoiceValues объекта ChoiceList 

Рассмотрим на примере списка выбора типа Integer:

```java
// создать элементы варианта
Choice objChoiceMidInt1 = Factory.Choice.createInstance();
objChoiceMidInt1.set_ChoiceType(ChoiceType.INTEGER);
objChoiceMidInt1.set_DisplayName("Seattle");
objChoiceMidInt1.set_ChoiceIntegerValue(11);

Choice objChoiceMidInt2 = Factory.Choice.createInstance();
objChoiceMidInt2.set_ChoiceType(ChoiceType.INTEGER);
objChoiceMidInt2.set_DisplayName("Kirkland");
objChoiceMidInt2.set_ChoiceIntegerValue(12);

Choice objChoiceMidInt3 = Factory.Choice.createInstance();
objChoiceMidInt3.set_ChoiceType(ChoiceType.INTEGER);
objChoiceMidInt3.set_DisplayName("Bellevue");
objChoiceMidInt3.set_ChoiceIntegerValue(13);

// создаём и заполняем группу элементов варианта
Choice objChoiceInt1 = Factory.Choice.createInstance();
objChoiceInt1.set_ChoiceType(ChoiceType.MIDNODE_INTEGER);
objChoiceInt1.set_DisplayName("Washington");
objChoiceInt1.set_ChoiceValues(Factory.Choice.createList());
objChoiceInt1.get_ChoiceValues().add(objChoiceMidInt1);
objChoiceInt1.get_ChoiceValues().add(objChoiceMidInt2);
objChoiceInt1.get_ChoiceValues().add(objChoiceMidInt3);
objChoiceInt1.set_ChoiceIntegerValue(1);

Choice objChoiceInt2 = Factory.Choice.createInstance();
objChoiceInt2.set_ChoiceType(ChoiceType.INTEGER);
objChoiceInt2.set_DisplayName("Oregon");
objChoiceInt2.set_ChoiceIntegerValue(2);

Choice objChoiceInt3 = Factory.Choice.createInstance();
objChoiceInt3.set_ChoiceType(ChoiceType.INTEGER);
objChoiceInt3.set_DisplayName("California");
objChoiceInt3.set_ChoiceIntegerValue(3);

// создаём список выбора целочисленного типа
com.filenet.api.admin.ChoiceList objChoiceListInt = Factory.ChoiceList.createInstance(objObjectStore);
objChoiceListInt.set_DataType(TypeID.LONG);

// добавляем в него элементы варианта и задаём необходимые свойства
objChoiceListInt.set_ChoiceValues(Factory.Choice.createList());
objChoiceListInt.get_ChoiceValues().add(objChoiceInt1);
objChoiceListInt.get_ChoiceValues().add(objChoiceInt2);
objChoiceListInt.get_ChoiceValues().add(objChoiceInt3);
objChoiceListInt.set_DisplayName("Integer State List");

//сохраняем
objChoiceListInt.save(RefreshMode.REFRESH);
```

Сохраненный список выбора будет принадлежать хранилищу - коллекции ChoiceListSet, возвращаемой методом ObjectStore.get_ChoiceLists().

### Связывание списка выбора с шаблоном свойства

Напишем функцию, принимающую объект ObjectStore, имя шаблона и список выбора, и связывающую шаблон со списком:

```java
public static boolean bindChoiceList(ObjectStore objectStore, String propertyTemplateName, ChoiceList choiceList) {
    Iterator iter = objectStore.get_PropertyTemplates().iterator();
    PropertyTemplate objPropertyTemplate = null;
    
    while (iter.hasNext())
    {
       objPropertyTemplate = (PropertyTemplate) iter.next();
       prpSymbolicName = objPropertyTemplate.get_SymbolicName();

       if (prpSymbolicName.equalsIgnoreCase(propertyTemplateName))
       {
          objPropertyTemplate.set_ChoiceList(objChoiceList);
          objPropertyTemplate.save(RefreshMode.REFRESH);
          
          return true;
       }
    }
    
    return false;
}
```

### Отмена связи со списком выбора

Если определение свойства должны быть отвязаны от списка выбора, это можно сделать следующим образом:

```java
    objPropertyDefinition.set_ChoiceList(null);

    //сохраняем определение класса, в котором было это определение свойства
    objClassDefinition.save(RefreshMode.REFRESH);
```

Отвязываение шаблона свойства:

```java
    objPropertyTemplate.set_ChoiceList(null);
    objPropertyTemplate.save(RefreshMode.REFRESH);
```

## Дополнительная информация

* [Working with choice lists](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.ce.doc/choicelist_procedures.htm)
