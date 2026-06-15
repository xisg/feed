---
title: 'Включаем EPA в FreeTDS и go-mssqldb: приключение на 5 минут'
url: https://habr.com/ru/companies/yandex/articles/1031368/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog
published: "2026-05-12T08:02:54Z"
feed: yandex
guid: https://habr.com/ru/companies/yandex/articles/1031368/
---

# Включаем EPA в FreeTDS и go-mssqldb: приключение на 5 минут

![](https://habrastorage.org/getpro/habr/upload_files/c43/8a7/3fd/c438a73fd7808da106570a226a80d592.png)

Представьте: вы теряете контроль над SCCM — одним из самых критичных инструментов управления инфраструктурой. А точкой входа становится обычное подключение к MSSQL, где он хранит свои данные. Злоумышленник перехватывает NTLM-аутентификацию и перенаправляет её на нужный сервер — так работает NTLM relay. Мы в команде Security Engineering решили не ждать эксплуатации этой уязвимости.

Меня зовут Булат Гафуров, я инженер по информационной безопасности в Яндексе. В этой статье я расскажу, почему стандартного решения оказалось недостаточно и как мы добавили поддержку механизма EPA в популярные библиотеки, чтобы переключить защиту на стороне MSSQL в режим Require, не лишив Linux- и Windows-сервисы доступа к данным.

 [Читать далее](https://habr.com/ru/articles/1031368/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog#habracut)
