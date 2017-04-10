# Обзор типов

Отношения между некоторыми классами и интерфейсами упрощённо показаны на рисунке ниже:

![Class relationships sample](classes.jpg)

Рассмотрим некоторые типы, определяемые различными пакетами CE API.

## com.filenet.api.core

### Классы

* [Batch](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/Batch.html). [Пакетные операции](batch.md)
  * [RetrievingBatch](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/RetrievingBatch.html). Пакетное получение
  * [UpdatingBatch](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_4.5.1/com.ibm.p8.doc/developer_help/content_engine_api/javadocs/com/filenet/api/core/UpdatingBatch.html). Пакетное обновление
* фабричные классы Factory.XXXX. Фабричные классы предоставляют [методы](instance_methods.md) для создания новых инстансов объектов CE и для получения существующих

### Интерфейсы

* [EngineObject](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/EngineObject.html). Базовый интерфейс всех объектов CE, необязательно принадлежащих ObjectStore. Объект EngineObject принадлежит к определенному "классу" (не путать с классами Java) и имеет метаданные (свойства, набор которых определяется этим классом)
* [RepositoryObject](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/RepositoryObject.html). Базовый интерфейс для объектов CE, принадлежащих Object Store. Соответственно, объявляет метод getObjectStore().
* DependentObject. Объекты, которые могут существовать только как составные части других объектов. Например, Объект PropertyDefinition может существовать только в рамках объекта ClassDefinition
* [IndependentObject](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.0.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/IndependentObject.html). Объекты, которые могут существовать независимо от других объектов
* [IndependentlyPersistableObject](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.0.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/IndependentlyPersistableObject.html). Независимые объекты, которые можно создавать, обновлять, удалять
* [Containable](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Containable.html). Базовый интерфейс для объектов, которые могут быть содержимым папок. Подтипы: Document, Folder, CustomObject
* [Versionable](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/Versionable.html). Базовый интерфейс для версионных объектов. Подтипы: Document
* EntireNetwork. Объект, находящийся на самом высоком уровне иерархии объектов CE. Имеет методы для получения объекта Domain и объектов Realm (области логических связанных пользователей и групп)
* Document. Версия документа хранилища
* Folder.	Контейнер, который хранит объекты Containable (сам также является Containable). Не обладает версионностью, не может иметь элементы содержимого
* CustomObject. Простой Containable объект. Не обладает версионностью, не может иметь элементы содержимого
* Connection.	Логическое соединение с CE
* Link.	Связь между двумя объектами типа IndependentObject. Ссылка имеет свойства Head и Tail типа IndependentObject
* [ContentElement](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/ContentElement.html). Базовый класс для элементов содержимого, которые могут приндалежать объектам Document или Annotation
* [VersionSeries](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/core/VersionSeries.html). Представляет набор версий Versionable-объекта 

## com.filenet.api.meta

* ClassDescription. Описание класса. Только для чтения
* PropertyDescription. Базовый интерфейс для описаний свойств различных типов. Только для чтения
* PropertyDescriptionDateTime. Описание свойства типа «Дата и время». Только для чтения
* интерфейсы для описаний свойств разных типов (PropertyDescriptionString и проч.). Только для чтения

## com.filenet.api.admin

* ClassDefinition. Определение класса. Базовый интерфейс для DocumentClassDefinition и других
* PropertyDefinition. Базовый интерфейс для определения свойств различных типов
* PropertyDefinitionDateTime. Определение свойства типа «Дата и время»
* интерфейсы для определений свойств разных типов (PropertyDescriptionString и проч.)
* PropertyTemplate. Базовый интерфейс для шаблонов свойств. Шаблоны позволяют создавать однотипные определения свойств (PropertyDefinition) для разных классов
* DirectoryConfiguration. Экземпляр конфигурации LDAP
* PEConnectionPoint. ТОчка подключения к FileNet Process Engine	
* ServerInstance. JVM, запущенная на сервере приложений
* TableDefinition. Таблица БД хранилища объектов

## com.filenet.api.security

* User. Пользователь CE
* Group. Группа, в которую могут быть добавлены пользователи CE
* AccessPermission. Правило доступа, содержащее битовую маску. 
* MarkingSet. Набор маркировок. Предназначен для управления правами доступа к объекту в зависимости от значения определённого свойства типа String.

## com.filenet.api.query

* SearchScope. Область поиска (чаще всего – в пределах одного ObjectStore). Содержит методы для извлечения объектов и данных.
* RepositoryRow. Экземпляр набора свойств, получаемого при поиске. Располагает объектом Properties

## com.filenet.api.collection

Различные коллекции объектов CE. Делятся на множества (...Set) и списки (...List).

Множества не являются потомками java.util.Collection и java.util.Iterable, поэтому не могут быть использованы в варианте for-each цикла for. Вместо этого являются потомками com.filenet.api.collection.EngineCollection, определяющего методы isEmpty() и iterator()

Списки, как правило, являются потомкми и EngineCollection, и java.util.List

* AccessPermissionList - список [прав доступа](permissions.md)
* ContentElementList. Коллекция объектов ContentReference и ContentTransfer
* FolderSet. Коллекция объектов Folder
* ReporitoryRowSet, IndependentObjectSet - контейнеры для результатов [SQL-запроса](query.md)
* и [многие другие](https://www.ibm.com/support/knowledgecenter/en/SSGLW6_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/collection/package-summary.html)

## com.filenet.api.events

* FileEvent	
* UnfileEvent	
* EventAction	
* InstanceSubscription	

## com.filenet.api.property

* Properties. Коллекция объектов Property
* Property. Базовый интерфейс для свойств
* PropertyDateTime. Свойство типа «Дата и время» мощности single
* PropertyDateTimeList. Свойство типа «Дата и время» мощности list
* PropertyFilter. Набор информации о том, какие свойства объекта CE нужно извлечь. Используется в таких операциях, как fetch и refresh

## com.filenet.api.constants

* AccessRight. Константы прав доступа (READ, WRITE и т.п.)
* Cardinality. Мощность значения свойства (SINGLE, LIST)
* DatabaseType. Тип используемой СУБД (DB2, ORACLE, …)
* PropertyNames. Имена свойств
* ReservationType. Тип резервации документа (COLLABORATIVE, EXCLUSIVE)
* и [многие другие](https://www.ibm.com/support/knowledgecenter/en/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/constants/package-summary.html)
