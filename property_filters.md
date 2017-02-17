# Фильтры свойств

Фильтры свойств - мощный и гибкий способ управления тем, какие свойства и из каких объектов нужно извлечь при обращении к хранилищу объектов. Технически этот механизм реализован через методы с аргументами [PropertyFilter](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/PropertyFilter.html):

* `IndependentlyPersistableObject.save(RefreshMode refreshMode, PropertyFilter filter)`
* `IndependentObject.refresh(PropertyFilter filter)`
* `IndependentObject.fetchProperty(java.lang.String propertyName, PropertyFilter filter)`
* `IndependentObject.fetchProperties(PropertyFilter filter)`
* `IndependentObjectSet SearchScope.fetchObjects(SearchSQL searchSQL, java.lang.Integer pageSize, PropertyFilter filter, java.lang.Boolean continuable)`
* `RepositoryRowSet SearchScope.fetchRows(SearchSQL searchSQL, java.lang.Integer pageSize, PropertyFilter filter, java.lang.Boolean continuable)`
* `Factory.XXXX.fetchInstance(...)` - множество разновидностей

Полный список можно посмотреть [здесь](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/class-use/PropertyFilter.html).

Фильтры свойств улучшают быстродействие двумя способами:
* уменьшив объём данных (количество извлекаемых свойств)
* применив рекурсивный способ получения свойств - происходит обработка не только базового объекта, но и тех объектов, которые являются значениями его свойств. Это сокращает число запросов к БД

Типичный алгоритм использования фильтра:

1. создать объект PropertyFilter. Так как PropertyFilter - класс, это делается не через фабрику, а с помощью ключевого слова new.
2. добавить в фильтр включаемые и/или исключаемые свойства методами addIncludeProperty() или addExcludeProperty()
3. передать объект PropertyFilter в один из методов fetchInstance(), fetchPrpperties() или других

В самом простом варианте фильтр работает так же, как метод fetchProperties() с аргументом типа массив строк. Код

```java
PropertyFilter filter = new PropertyFilter();
filter.addIncludeProperty(new FilterElement(null, null, null, "DocumentTitle DateCreated", null));
document.save(RefreshMode.REFRESH, filter);
```

эквивалентен коду

```java
document.fetchProperties(new String[] {"DocumentTitle", "DateCreated"});
document.save(RefreshMode.REFRESH, filter);
```

Основные методы PropertyFilter:

метод | что делает
------------ | -------------
`void addExcludeProperty(java.lang.String value)`|Добавить имена свойств (разделяются пробелами), которые не будут извлечены. Если задано одно и то же свойство для включения и исключения, то исключение имеет приоритет
`void addIncludeProperty(FilterElement fe)`|Добавить свойства для включения. См. [FilterElement](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/FilterElement.html)
`void addIncludeType(FilterElement fe)`|Добавить свойства определённого типа. См. вариант конструктора FIlterElement с параметром FilteredPropertyType
`void setLevelDependents(boolean levelDependents)`|Установить глобальное значение параметра levelDependents (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)
`void setMaxRecursion(int maxRecursion)`|Установить глобальное значение параметра maxRecursion (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)
`void setMaxSize(java.lang.Long maxSize)`|Установить глобальное значение параметра maxSize (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)
`void setPageSize(int pageSize)`|Установить глобальное значение параметра pageSize (глобальное значение используется, когда соответствующее значение не задано (null) в конструкторе FilterElement)

Класс [FilterElement](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/FilterElement.html) содержит информацию о том, что нужно включить в фильтр. Интересны только конструкторы:

* `FilterElement(java.lang.Integer maxRecursion,java.lang.Long maxSize,java.lang.Boolean levelDependents,FilteredPropertyType value,java.lang.Integer pageSize)` - извлечь свойства определённого типа. Тип указывается параметром [FilteredPropertyType](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/constants/FilteredPropertyType.html)
* `FilterElement(java.lang.Integer maxRecursion,java.lang.Long maxSize,java.lang.Boolean levelDependents,java.lang.String value,java.lang.Integer pageSize)` - извлечь свойства, имена которых указаны в параметре value. Если свойств несколько, имена разделяются пробелом

Параметры, управляющие фильтрацией:

* **levelDependents**. Флаг, указывающий, нужно ли при извлечении свойств зависимых объектов углубляться ещё на один уровень, по сравнению с независимыми. Если true, то не нужно.
* **maxRecursion**. Глубина рекурсии при извлечении свойств. По умолчанию - 0, т.е. извлекаются свойства только "базового" объекта.
* **maxSize**. Максимальный размер содержимого для свойств, обладающих содержимым. Если размер содержимого окажется больше, оно не будет получено вообще. По умолчанию - без ограничений.
* **pageSize**. Параметр имеет отношение только к свойствам типа PropertyIndependentObjectSet. Задаёт размер страницы, т.е. то, сколько объектов извлекается за одно обращение.

## Рекурсия

Рекурсия может сократить число обращений ("round-trips") к БД. Смысл в том, что за одно обращение извлекаются свойства не только "базового" объекта, но и у объектов из его свойств, объектов из свойств этих объектов и т.д. (глубина рекурсии настраивается параметром maxRecursion).

Например, нужно построить иерархию папок. Это делается за одно обращение к БД (в примере принята максимальная вложенность 99):

```java
final int MAX_NESTING_LEVEL = 99;

PropertyFilter filter = new PropertyFilter();

filter.addIncludeProperty(new FilterElement(MAX_NESTING_LEVEL, null, null, PropertyNames.FOLDER_NAME, null));
filter.addIncludeProperty(new FilterElement(MAX_NESTING_LEVEL, null, null, PropertyNames.SUB_FOLDERS, null));

Folder folder = Factory.Folder.fetchInstance(objectStore, "/", filter);

inspectFolder(folder, "");
```

Функция inspectFolder() может выглядеть так:

```java
private static void inspectFolder(Folder f, String indent) {
    System.out.println(indent + f.get_FolderName());
    
    FolderSet subFolders = f.get_SubFolders();
    
    Iterator<Folder> iterator = subFolders.iterator();
    while (iterator.hasNext()) {
        inspectFolder(iterator.next(), indent + "  ");
    }
}
```

### Дополнительные материалы

* http://www.notonlyanecmplace.com/understand-the-filenet-property-filters/

