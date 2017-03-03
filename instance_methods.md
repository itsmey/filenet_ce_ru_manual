# Фабричные методы

Класс [Factory](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/Factory.html) предназначен получения объектов других классов. Он содержит большое количество вложенных классов Factory.XXXX, где XXXX - имя интерфейса. Например, методы Factory.Document возвращают объекты типа Document.

К основным фабричным методом относятся:

* createInstance – используется для создания нового экземпляра объекта
* fetchInstance – используется для получения имеющегося в хранилище экземпляра объекта
* getInstance – также используется для получения имеющегося в хранилище экземпляра объекта

## Разница между fetchInstance и getInstance

### getInstance()
Создаёт локальное представление объекта, не извлекая при этом свойств (Properties) объекта. В таком виде над объектом можно производить операции, не затрагивающие свойств (например, удаление). Для получения свойств можно выполнить один из методов .refresh() или .fetchProperties().

getInstance() не обращается к серверу CE. Обращение к серверу происходит во время выхова таких методов, как save() или refresh().

### fetchInstance()
Создаёт локальное представление объекта и, кроме этого, извлекает свойства объекта в локальный кэш согласно аргументу PropertyFilter. Смысл фильтра свойств в том, что необходимо извлекать не все свойства (их может быть много), а только те, которые понадобятся в дальнейшей работе. Поэтому необходимо всегда пользоваься фильтром. Однако, если аргумент PropertyFilter равен null, из хранилища извлекаются все свойства.

fetchInstance() обращается к серверу CE. Это значит, что метод может бросить исключение E_OBJECT_NOT_FOUND, если объект не найден на сервере. Рассмотрим код, определяющий, существует ли документ с данным Id:

```java
boolean isDocumentExists(Id id) {
    try {
        Document doc = Factory.Document.fetchInstance(objectStore, id, null);
    } catch(EngineRuntimeException ex) {
        ExceptionCode e = ex.getExceptionCode();
        if(e == ExceptionCode.E_OBJECT_NOT_FOUND) {
            return false;
        }
        else throw ex;
    }
    return true;
}
```

Код можно переписать, используя getInstance() вместо fetchInstance():

```java
boolean isDocumentExists(Id id) {
    try {
        Document d = Factory.Document.getInstance(objectStore, null, id);
        d.refresh();
    } catch(EngineRuntimeException ex) {
        ExceptionCode e = ex.getExceptionCode();
        if(e == ExceptionCode.E_OBJECT_NOT_FOUND) {
            return false;
        }
        else throw ex;
    }
    return true;
}
```

Методы refresh() и fetchProperties() также поддерживают аргумент PropertyFilter.

## Factory.Document

Рассмотрим набор фабричных методов класса [Factory.Document](https://www.ibm.com/support/knowledgecenter/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/Factory.Document.html). Аналогичными будут методы для классов Factory.Folder, Factory.CustomObject и других. В большинстве фабрик Factory.XXXX также имеются подобные методы, которые работают схожим образом.

метод | что делает
------------ | -------------
`Document createInstance(ObjectStore os, java.lang.String classId)`| Создать новый экземпляр, ID генерируется автоматически. classId должен быть именем класса, являющимся подклассом базового класса. Если classId не задан, используется имя базового класса.
`static Document createInstance(ObjectStore os, java.lang.String classId, Id id)`|Создать новый экземпляр с указанным ID. Если ID = null, он генерируется автоматически. 
`static Document createInstance(ObjectStore os, java.lang.String classId, Id id, Id versionSeriesId, ReservationType reservationType)`|Создать новый экземпляр с указанным ID. Если ID = null, он генерируется автоматически. Одновременно будет создан объект VersionSeries с указанным ID. Также указывается тип резервирования (совместный или эксклюзивный).
`static Document fetchInstance(ObjectStore os, Id objectId, PropertyFilter filter)`|Получить объект по его ID, с набором свойств, задаваемым PropertyFilter. Если PropertyFilter = null, извлекаются все свойства. 
`static Document fetchInstance(ObjectStore os, java.lang.String path, PropertyFilter filter)`|Получить объект по его расположению, с набором свойств, задаваемым PropertyFilter. Если PropertyFilter = null, извлекаются все свойства.
`static Document getInstance(ObjectStore os, java.lang.String className, Id objectId)`|Получить экземпляр объекта по его ID.
`static Document getInstance(ObjectStore os, java.lang.String className, java.lang.String path)`|Получить экземпляр объекта по его расположению.

*Примечание: варианты методов getInstance() и fetchInstance() с расположением (Path) в качестве идетнификатора возможны только для Containable-объектов, к которым отностся объекты Document, Folder и CustomObject.*
