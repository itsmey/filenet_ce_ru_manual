# Фильтры свойств

Фильтры свойств - мощный и гибкий способ управления тем, какие свойства и из каких объектов нужно извлечь при обращении к хранилищу объектов. Технически этот механизм реализован через методы с аргументами [PropertyFilter](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/PropertyFilter.html):

* `IndependentlyPersistableObject.save(RefreshMode refreshMode, PropertyFilter filter)`
* `IndependentObject.refresh(PropertyFilter filter)`
* `IndependentObject.fetchProperty(java.lang.String propertyName, PropertyFilter filter)`
* `IndependentObject.fetchProperties(PropertyFilter filter)`
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

Класс [FilterElement](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/property/FilterElement.html) содержит информацию о том, что нужно включить в фильтр. Интересны только конструкторы:

* `FilterElement(java.lang.Integer maxRecursion,java.lang.Long maxSize,java.lang.Boolean levelDependents,FilteredPropertyType value,java.lang.Integer pageSize)` - извлечь свойства определённого типа. Тип указывается параметром [FilteredPropertyType](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.1.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/constants/FilteredPropertyType.html)
* FilterElement(java.lang.Integer maxRecursion,java.lang.Long maxSize,java.lang.Boolean levelDependents,java.lang.String value,java.lang.Integer pageSize) - извлечь свойства, имена которых указаны в параметре value. Если свойств несколько, имена разделяются пробелом


