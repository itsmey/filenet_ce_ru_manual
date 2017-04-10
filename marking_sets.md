# Наборы маркировок

Как писалось [ранее](permissions.md), CE предоставляет доступ к объекту пользователю или группе согласно списку прав доступа (ACL) - свойство Permissions типа AccessPermissionList.

Наборы маркировок - это ещё один механизм регулирования доступа, действующий наряду с основным.

## Основные положения

* маркировка ([Marking](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.0/com.ibm.p8.ce.dev.java.doc/com/filenet/api/security/Marking.html)) - объект, имеющий следующие свойства:
** **markingValue** - значение маркировки (String)
** **Permissions** - список прав доступа типа (AccessPermissionList)
** **ConstraintMask** - маска ограничения (Integer) 
* набор маркировок ([MarkingSet](https://www.ibm.com/support/knowledgecenter/SSNW2F_5.2.1/com.ibm.p8.ce.dev.java.doc/com/filenet/api/security/MarkingSet.html)) - множество маркировок с разными значениями
* наборы маркировок определяются на уровне домена. Каждое хранилище в домене может использовать наборы домена. Домен (Domain) имеет свойство MarkingSetSet - коллекция, содержащая все наборы маркировок
* шаблон свойства типа String (PropertyTemplateString) имеет свойство Marking типа MarkingSet. Через это свойство можно связать шаблон с определенным набором маркировок 
* шаблон свойства имеет метод createClassProperty(), производящий описание свойства (PropertyDescription). Описание свойства может быть добавлено к описанию класса (ClassDescription). Соответсвенно, доступ к инстансам таких классов зависит уже не только от свойства Permissions, и от свойства типа String, которое было связано с набором маркировок
