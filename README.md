XMPP Jabber - Server
=============

[Опис протоколу можна знайти тут](http://xmpp.org/rfcs/rfc6121.html)

*Налаштування*

- Для коректної роботи сервреа у режимі BOSH обов'язково необхідно у файлі **Jabber.Net.Server/Connections/BoshXmppConnection.cs** у методі 'Send' додати рядок:
`context.Response.Headers.Add("Access-Control-Allow-Origin", "*");`
- У проекті Jabber.Net, файлі App.config необіхдно вказати усі необхідні налаштування:
  - instance(для домена, користувача, конференцій і т.д.
  - `<property name="Url" value="http://ecampus.kpi.ua" />`
  - за необхідності також необхідно вказати відповідні порти для усіх типів з'єднаннь ("розділ" <!-- Listeners -->)

*Авторизація*

Авторизація відбувається в декілька етапів:
- Клієнт надсилає запит серверу на авторизацію. Запит містить у собі наступні поля: realm, nonce, cnonce, qop, username, charset, algorithm, rspauth, nc, digest-uri, response. Ці параметри використовуються для налаштування алгоритму авторизації і для генерування відповіді.
- Використовуючи  username через campus.API отримуються дані про користувача
- Далі проводиться хешування параметрів користувача і сервера, виконується наступне:
  - генерується хеш з даними користувача
    - якщо клієнт використовує md5-sess, то хеш формується так a1 = ComputeMd5(username + ":" + Realm + ":" + password, false) + ":" + Nonce + ":" + Cnonce;
    - якщо клієнт використовує md5, то a1 = username + ":" + Realm + ":" + password;
  - генерується хеш з даними сервера: a2 = method + ":" + DigestUri;
  - генерація відповіді клієнту:
    - якщо заданий параметр qop, то відповідь така:
      ComputeMd5( ComputeMd5(a1, true) + ":" + Nonce + ":" + Nc + ":" + Cnonce + ":" + Qop + ":" +
            ComputeMd5(a2, true),
            true);
    - інакше ComputeMd5(ComputeMd5(a1, true) + ":" + Nonce + ":" + ComputeMd5(a2, true), true);
- Відповідь надсилається клієнту
- Клієнт також генерує хеші по тим самим правилам, якщо результати співпадають, то авторизація пройшла успішно.
У випадку виникнення помилки на будь-якому із етапів клієнт отримає відповідне повідомлення про помилку.

*Клас XmppUserStorage*

Даний клас відповідає за роботу з клієнтами, але потребує значних доопрацювань:
- **void SaveUser(XmppUser user)** відповідає за збереження користувача в БД. На разі лише перевіряється, щоб поля логіну і пароля не були пустими. Якщо користувачі мають реєструватись у системі через кампус безпосередньо, то необхідно прибрати із сервера таку можливість, або зробити реєстрацію нового клієнта через АПІ кампуса;
- **RosterItem GetRosterItem(Jid user, Jid contact)** має повертати статус contact, якшо user має таке право, але по факту є заглушкою і просто повертає новий RosterItem
- **void SaveRosterItem(Jid user, RosterItem ri)** просто перевіряє чи вхідні параметри не пусті. Також потребує доопрацювань
- **bool RemoveRosterItem(Jid user, Jid contact)** дивись попередній пункт. Завжди повертає true

*XmppElementStorage*

Відповідає за роботу з елементами запитів, що використовуються у протоколі XMPP. Але абсолютний фейк. Має виконувати CRUD операції над об'єктами, але по факту прото заглушки, які тільки перевіряють вхідні аргументи.
