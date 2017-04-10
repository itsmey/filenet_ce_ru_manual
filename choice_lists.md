# Списки выбора

Список выбора (Choice list) - набор предопределённых значений. Списки выбора особенно удобны, когда требуется выбор из сравнительно небольшого и неизменного множества вариантов. Например:

* названия субъектов РФ
* пол: "мужчина"/"женщина"
* времена года
* и т.п.

## ChoiceList

[com.filenet.api.admin.ChoiceList](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/admin/ChoiceList.html) - собственно, список выбора. 

* Определяется в хранилище объектов: свойство хранилища ChoiceLists типа ChoiceListSet - это коллекция объектов ChoiceList в хранилище.
* Создавать и извлекать (по Id) инстансы можно фабричными методами: [Factory.ChoiceLists](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Factory.ChoiceList.html).
* Имеет свойство Permissions, т.е. можно настраивать права доступа. По умолчанию список выбора имеет те же права, что и хранилище: полные права для администраторов, чтение свойств для обычных пользователей


Свойство | Тип | Что делает
------------ | ------------- | -------------
Permissions|AccessPermissionList|Права доступа. По умолчанию список выбора имеет те же права, что и хранилище: полные права для администраторов, чтение свойств для обычных пользователей
DisplayName|String|Имя списка для UI
DataType|TypeID|Тип внутреннего значения. Может быть STRING или INTEGER.


## Choice

[Choice](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/admin/Choice.html#get_ChoiceValues()) - элемент варианта.
