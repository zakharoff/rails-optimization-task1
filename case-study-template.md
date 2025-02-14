# Case-study оптимизации

## Актуальная проблема
В нашем проекте возникла серьёзная проблема.

Необходимо было обработать файл с данными, чуть больше ста мегабайт.

У нас уже была программа на `ruby`, которая умела делать нужную обработку.

Она успешно работала на файлах размером пару мегабайт, но для большого файла она работала слишком долго, и не было понятно, закончит ли она вообще работу за какое-то разумное время.

Я решил исправить эту проблему, оптимизировав эту программу.

## Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я придумал использовать такую метрику: время выполнения скрипта с объемом данных в 1 000, 2 000, 4 000, 8 000, 16 000 и 200 000 строк.

## Гарантия корректности работы оптимизированной программы
Программа поставлялась с тестом. Выполнение этого теста в фидбек-лупе позволяет не допустить изменения логики программы при оптимизации.

## Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений за менее чем за 10 секунд.

Вот как я построил `feedback_loop`:
* сбор метрики выполнения скрипта на разном количестве данных для выявление асимптотической сложности
* профилирование скрипта для сбора узких мест

## Вникаем в детали системы, чтобы найти главные точки роста
Для того, чтобы найти "точки роста" для оптимизации я воспользовался `ruby-prof` и `stackprof` гемами.

Вот какие проблемы удалось найти и решить:

* Рефакторинг строки `sessions.select { |session| session['user_id'] == user['id'] }` и использование `Set`, что позволило снизить асимптотическую сложность до линейной. Время работы на 200 000 строк сократилось до 7 секунд.

* Избавление от перезаписывания переменных на оператор `<<`. Время работы на 200 000 строк сократилось до 3 секунд.

* Комплексные улучшения: использование гема `oj`, удаление парсинга дат (дата уже в нужном формате), избавление от класса `User`, переход на ключи хешей как символов вместо строк, избавление от парсинга пользователя. Время работы на 200 000 строк сократилось до 2 секунд.

* Рефакторинг метода `collect_stats_from_users`. Время работы на 200 000 строк сократилось до 1 секунды.

## Результаты
В результате проделанной оптимизации наконец удалось обработать файл с данными.
Удалось улучшить метрику системы с данными в 200 000 строк с нескольких минут до 1 секунды и уложиться в заданный бюджет.

Общее время выполнения скрипта на полном объеме данных - 21 секунда (macbook pro 13 2020).

## Защита от регрессии производительности
Для защиты от потери достигнутого прогресса при дальнейших изменениях программы был написан простой `rspec-perfomance` тест, который проверял выполнение программы за 100мс на 16 000 строк, с учетом прогрева.
