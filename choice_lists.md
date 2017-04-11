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


