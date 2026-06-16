---
title: 'Ускорение Яндекс Трекера: в погоне за Velocity Index'
url: https://habr.com/ru/companies/yandex/articles/1010280/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog
published: "2026-03-19T07:00:06Z"
feed: yandex
guid: https://habr.com/ru/companies/yandex/articles/1010280/
---

# Ускорение Яндекс Трекера: в погоне за Velocity Index

![](https://habrastorage.org/getpro/habr/upload_files/b50/659/2f6/b506592f66df975be0303c82518b217b.png)

Внутренний трекер задач — Яндекс Трекер — важная часть Яндекса. В нём хранятся почти все планы: от целей отделов, до тикетов поддержки. RPS на фронтенд измеряется сотнями, а количество хитов в месяц — десятками миллионов. При таком масштабе даже небольшие задержки могут становиться критичными, поэтому мы задались целью ускорить Трекер. Спойлер: всё получилось не совсем так, как мы ожидали. Но обо всём по порядку.

Для измерения скорости сервисов в Яндексе используется метрика Velocity Index — это агрегация метрик Web Vitals ( [FCP](https://web.dev/fcp/), [LCP](https://web.dev/lcp/), [TBT](https://web.dev/tbt/), [INP](https://web.dev/inp/), [CLS](https://web.dev/cls/)). Итоговое значение получается в диапазоне от 0 до 100 баллов. Хорошим результатом считается индекс больше 85.

Мы поставили себе амбициозную цель: увеличить Velocity Index до 85, а заодно подлечить очевидные «узкие места» в скорости и ускорить всё, до чего сможем дотянуться.

Но до заветных 85 баллов мы так и не добрались.

[И вот почему](https://habr.com/ru/articles/1010280/?utm_source=habrahabr&utm_medium=rss&utm_campaign=corporate_blog#habracut)
