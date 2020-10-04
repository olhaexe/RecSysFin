# RecSysFin
Final Project for Recommendation Systems

Описание проекта

Несмотря на то, что добавила в Recommender много вариантов получения кандидатов и попробовала огромное количество параметров, оптимальным оказался вариант решения, также реализованный на основании предыдущих покупок пользователя без применения машинного обучения, на фильтрации и сортировке. Теоретически рекомендации на основании собственных покупки пользователя должен выдавать ItemItemRecommender с k=1, но он работает хуже ручной обработки. Оставляю два варианта кода: 
- полная версия с включением ML-вариантов получения кандидатов, которые дают плачевные метрики (при других способах обработки и параметрах - максимум около 12%): https://github.com/olhaexe/RecSysFin/blob/main/final_recs-full.ipynb и 
- урезанный до рабочей схемы https://github.com/olhaexe/RecSysFin/blob/main/final_recs_light.ipynb
HTML: https://github.com/olhaexe/RecSysFin/blob/main/final_recs_light.html
Рекомендации: https://github.com/olhaexe/RecSysFin/blob/main/OBerezovskaya_recs.csv

Метрика на отложенной (3 недели) выборке — 0.20003298, общее число рекомендуемых товаров 2282. На той, что в гит — 0.170276,  общее число рекомендуемых товаров 2135.


Что сделано:

Предфильтрация:
1. убираем товары с нулевым количеством и выручкой;
2. убираем товары с ценой 1 доллар и ниже;
3. убираем слишком дорогие товары (свыше 50 долларов, выбрала по лучшей метрике);
4. убираем товары, где дисконт больше 0 (наценка или ошибка, вряд ли способствует продажам); 
5. убираем товары, где купонный дисконт -10 и ниже (купили по купону с большой скидкой и вряд ли купят еще);
6. убираем товары, которые не продавались за последние несколько недель ( выбрала по лучшей метрике);
7. убрала явно сезонные (по commodity_desc) товары - не нашла подтверждения их актуальности;
8. дальше пробовала возможности, которые потом отключила: фильтр по частоте (доле покупателей), убрать категории с дешевыми товарами (department) — не сработало. Опытным путем сделала список категорий, который немного улучшил метрику. Убираю те товары, которые купили меньше 2 пользователей. В параметрах указан топ-22000 популярных товаров, но по факту их на этом этапе остается меньше (при уменьшении численности топа метрика падает).

Выбор рекомендаций:
Выбираем топ покупок каждого пользователя, удаляем разовые покупки и сортируем по количеству покупок (basket_id) и цене при одинаковом количестве.
Выбираем такой же общий топ покупок с сортировкой по количеству покупок (по цене и выручке получается хуже).
Еще один топ — дорогих товаров, беру его по нижней границе цены, чтобы уменьшить знаменатель в метрике (34% пользователей тестового датасета не покупали дорогих товаров). Также вытащила самый популярный дорогой товар без верхней границы цены (16 долларов), но он ухудшал метрики и в финал не попал.
Для удобства сделала словари цен и ранее купленных товаров.
Разделила пользователей на обычных и вип-категорию по признаку доли купленных товаров ценой более 4,5 долларов от всех купленных товаров (0.3). Делала также по среднему чеку, получается чуть хуже.
Опытным путем ограничила число кандидатов для каждого пользователя 110 товарами.

Постфильтрация:
Ранжирование всех рекомендаций по цене дало плохой результат, а вот в вип-сегменте повысило метрику.
Выбираю дорогие товары — сначала из списка рекомендаций, затем из общего топа популярных. Добавляю первый товар в финальные и категорию товара в список использованных.
Выбираю новые для пользователя товары — из рекомендованных (при моем способе выбора кандидатов их не будет, при МЛ возможны), затем из общего топа. Если дорогой товар является новым, беру в финальные рекомендации один товар из списка, если нет — два. Добавляю использованные категории. Дополняю финальный список рекомендованными (предварительно отсортированными по частоте покупки для всех и по цене для випов) товарами из неиспользованных категорий.

Проверка соответствия бизнес-метрикам проходит.
