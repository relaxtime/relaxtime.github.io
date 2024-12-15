---
layout: post
title:  "Authenticated Users"
categories: jekyll update
image: /images/suslik.jpg
---

Об ограничении доступа к критически важным учетным записям домена

#### Недостаток
Доступ к полной информации о ряде критически важных учетных записей домена для членов группы «Authenticated Users»  домена не ограничен.

#### Рекомендации
Ограничить доступ к полной информации о ряде критически важных учетных записей домена для членов группы «Authenticated Users»

В Active Directory есть встроенная группа Authenticated Users (Прошедшие проверку), в которую входят все пользователи, успешно прошедшие процедуру аутентификации в домене. По умолчанию для группы Authenticated Users предоставляется право чтения разрешений для организационных единиц в домене. Используя учетную запись пользователя домена злоумышленник может получить информацию о привилегиях, которыми обладают учетные записи в домене, и выбрать приоритетную цель для атаки.

Злоумышленник может собрать информацию об объектах Active Directory, сформировать данные для загрузки в базу данных Neo4j и выполнить запрос для получения информации о привилегиях, которые обладают пользователи в домене.

```
MATCH (n:User {admincount:False}) MATCH (m:User) WHERE NOT m.name = n.name MATCH p=allShortestPaths((n)-[r:AllExtendedRights|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner*1..]->(m)) RETURN p
```

![](/images/authenticated_users/Pasted%20image%2020241215003732.png)

Видим, что пользователь домена `icanchangepassword` имеет право менять пароль  учетных записей `testuser1` и `testuser2`.

При настройке ограничений на просмотр разрешений учетной записи `icanchangepassword` для группы Authenticated Users злоумышленник, обладая непривилегированной учетной записью, не сможет получить информацию о правах пользователя  `icanchangepassword`.

![](/images/authenticated_users/Pasted%20image%2020241215012837.png)

В графе отсутствует вершина `icanchangepassword` и, соответственно, ребра `ForceChangePassword` к вершинам `testuser1` и `testuser2`.

Источники:

https://windowsnotes.ru/activedirectory/active-directory-list-object-mode/

Картинка

https://oldford.ru/foto/post/kartinki-suslik-multyashnij