---
title: The Xcode build system has crashed, или Почему рекурсия — это плохо. Используем swift‑build со своими патчами
url: https://habr.com/ru/companies/yandex/articles/1024298/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog
published: "2026-04-21T07:07:13Z"
feed: yandex
guid: https://habr.com/ru/companies/yandex/articles/1024298/
---

# The Xcode build system has crashed, или Почему рекурсия — это плохо. Используем swift‑build со своими патчами

![](https://habrastorage.org/getpro/habr/upload_files/841/7ef/847/8417ef8472fde705190dc38931556f34.png)

Представьте ситуацию: вы работаете в огромном проекте, где количество модулей давно перевалило за тысячу. Вы решаете обновиться до свежего Xcode 26.2, ожидая прироста производительности, но вместо заветного «Build Succeeded» получаете молчаливое падение: SWBBuildService quit unexpectedly.

Всем привет, меня зовут Алексей Севко, я ведущий разработчик программного обеспечения из команды Delivery & Performance Яндекс Go. В этой статье я расскажу почти детективную историю о том, как:

— Искать иголку в стоге сена: когда падает закрытый бинарник Xcode.

— Стать контрибьютором swift-build: почему иногда проще переписать системный поиск макросов в swift-build, чем ждать фикса от Apple.

— Использовать свою версию билд-системы: как мы внедрили инфраструктуру прозрачной подмены компонентов Xcode через XCBBUILDSERVICE\_PATH, чтобы не ждать релиза Xcode со Swift 6.3 и работать стабильно уже сегодня.

Если ваш проект тоже перерос стандартные инструменты Apple или вам просто интересно, как превратить рекурсию в итерацию и не сойти с ума от 45-минутных дебаг-сессий, — добро пожаловать под кат!

 [Читать далее](https://habr.com/ru/articles/1024298/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog#habracut)
