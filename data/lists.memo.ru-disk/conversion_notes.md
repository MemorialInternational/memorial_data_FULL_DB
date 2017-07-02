# Замечания по конвертации базы в единую денормализованную таблицу

Скрипт `./glue_tables.sh` склеивает все таблицы дампа в одну большую таблицу `memorial_lists.tsv` и запаковывает в 7-Zip архив. В репозитории лежит как получившаяся таблица, так и скрипт кодирования, который можно использовать для обновления таблицы или изменить для перекодирования в другую форму, если это потребуется.
Таблица записана в tab-separated values файл (разновидность csv). Таблицы перекодированы из CP1251 в UTF-8. В распакованном виде таблица занимает 1.3Gb, в запакованной форме - 80Mb. Перед тем как работать с таблицей её следует распаковать командой:
`7z e memorial_lists.tsv.7z`

***Каждая строка таблицы - вся имеющаяся в базе информация про одного человека.***

Колонки разделены одним символом табуляции. В редких случаях (всего пара десятков случаев, каждый случай был проверен вручную), когда в исходных данных содержалась табуляция внутри поля, символ табуляции заменялся на пробел. Таким образом разделение значений по табуляции всегда позволяет получить значения полей. Так как табуляция и переносы строки в значениях полей отсутствуют, экранирование полей никогда не требуется и потому отсутствует. Все пустые поля заменены на `None` (это помогает работать с утилитами типа GNU join, которые плохо обрабатывают пустые поля в таблице). Экранирование, которое существовало ранее, было, по возможности, стёрто.

Здесь таится проблема: экранирование в исходных таблицах не соответствовало формату csv, поэтому алгоритм удаления кавычек был собственный упрощенный: удалить кавычки по краям, если есть; заменить пары кавычек, идущие подряд, на одиночные кавычки. Пробелы в начале и конце полей удалены. Однако остались случаи, когда поле начинается с кавычки (а заканчивается, например, не кавычкой, а точкой). В этих случаях Excel (по-меньшей мере Libre Office так себя ведёт) может посчитать, что это экранированное поле и сбить форматирование, "зажевав" несколько следующих строк. Будьте внимательны, кавычки в этом файле не должны интерпретироваться как спецсинтаксис csv-формата!
Возможно, ручное курирование приблизительно 3500 записей, начинающихся с кавычки, (преимущественно из колонок: sentence, occupation, rehabilitation_reason) поможет сделать файл корректным csv-файлом с учётом правил расстановки кавычек.

Таким образом, следует признать, что этот файл больше подходит для программной обработки, чем для использования стандартных инструментов типа Excel.

Даты, закодированные одним числом, были преобразованы к человеко-читаемому формату: "день.месяц.год". Если конкретное поле в дате, неизвестно указывается подчерк, например `_.5.1939`, т.е. май 1939го. Когда дата неизвестна совсем, указано значение `None`.
Алгоритм преобразования дат следующий: младшие 5 бит значения в исходной таблице кодируют число месяца, следующие 4 бита - месяц, оставшиеся биты кодируют величину (год - 1800). Если закодированное число равно нулю, это означает, что данное поле неизвестно. Проверено, что это преобразование взаимнооднозначное и возможно провести обратную операции закодирования дат, получив те же значения, что хранятся в таблицах мемориала.

Поле birthdate во всех случаях имеет точность не лучше, чем до года, поэтому было преобразовано в год рождения birthyear, также есть вспомогательное поля birthyear_comment и age.

Поля пол и флаг приговора к расстрелу были оставлены, фактически в исходном виде.

Значения из вспомогательных таблиц были "приклеены" к persons по указанному id. Таблица geoplaces приклеена сразу к двум полям: место рождения и место проживания. Все таблицы c префиксом link описывают отношение 1-к-1: один человек (по person_id) к одной записи во вспомогательной таблице с соответствующим именем (по record_id), что позволяет без потери информации подклеить их к таблице людей. В исходных данных она, вероятно, использовалась для сокращения объема таблицы для полей, которые указаны лишь у небольшого числа людей.

# Поля в итоговом файле:
- person_id: номер записи (взят в полном соответствии с таблицей persons.csv)
- surname: фамилия
- name: имя
- patronimic: отчество
- birth_year: год рождения (как число или None)
- birth_year_comment: год рождения (комментарий к дате рождения в текстовой форме)
- age: возраст на момент ареста (когда дата рождения неизвестна)
- sex: пол

- birth_place: место рождения
- nation: национальность
- citizenship: гражданство/подданство
- occupation: профессия
- live_place: постоянное место жительства
- education: образование
- party: партийная принадежность
- family: информация о семье

- arest_date: дата ареста
- arest_organ: орган, произведший арест
- arest_type: тип ареста
- criminal_case: номер дела
- judicial_organ: судебный орган
- process_date: дата суда
- criminal_article: статья, по которой осужден человек
- sentence: приговор
- was_executed: был ли человек приговорен к смерти (T - true, F - false)
- death_place: место смерти
- death_date: дата смерти
- previous_repression: информация о предыдущей (относительно описанной в этой записи) репрессии
- next_repression: информация о следующей (относительно описанной в этой записи) репрессии

- rehabilitation_organ: орган, реабилитировавший человека
- rehabilitation_reason: основание реабилитации
- rehabilitation_date: дата реабилитации

- all_name_variations: некоторая комбинация следующих четырех полей
- name_variations: вариации имени
- surname_variations: вариации фамилии
- patronimic_variations: вариации отчества
- fullname_variations: вариации полного имени

- memory_book: книга памяти, являвшаяся источником информации
- memory_book_url: URL книги памяти (вероятно, что эта информация имеет отношение только к диску и не имеет смысла в рамках таблицы)

# Техническая памятка для начинающих заниматься анализом данных
## (краткое введение в консольные инструменты для работы с табличными данными)

Многие консольные утилиты, работающие с таблицами (такие как `cut`, `sort`, `join`, `paste`), по-умолчанию воспринимают символ табуляции как разделитель, что делает существенно упрощает работу с этим файлом. Рассмотрим несколько простых примеров того, как работать с этими файлами в командной строке, которые помогут новичкам освоится и поэкспериментировать с предоставленными данными. Это, разумеется, не учебник по работе с командной строкой, а лишь иллюстрация того, как с её помощью эффективно можно работать с представленными табличными данными.

Для начала, рассмотрим пример того, как выбрать из файла только колонки: ФИО, национальность, приговор.

Работа в командной строке часто строится по принципу "трубопровода". Сначала запускается одна команда, затем с помощью "пайпа" (значка `|`) результат её работы передается как входные данные для следующей программы и так далее. Т.е. вы должны продумать последовательность преобразований, которые вы совершите с файлом. Обычно эти преобразования пишутся постепенно: вы делаете один этап обработки, смотрите результат, отлаживаете его. Затем прибавляете второй этап. Потом третий - и так, пока не сможете получить данные в той форме, которая вам нужна.

Обычно первая команда просто выводит все данные на экран (или в следующую программу в трубопроводе). Давайте выведем исходную таблицу со всеми данными. Для этого служит команда `cat <имя файла>`. Но если мы постараемся вывести на экран всю таблицу, это займет много часов. Часто, чтобы понять, как выглядит результат вашей команды, вам достаточно взглянуть лишь на несколько строк. В этом нам помогут команды `head` (взять только первые 10 строк) и `less -S` показать результат с возможностью листать его вперед-назад и скроллить очень длинные строки вправо-влево. Применяется это так:
`cat memorial_lists.tsv | head` или `cat memorial_lists.tsv | less -S`  (чтобы `less` закрылся, нужно нажать клавишу `q`).
Стоит заметить, что если вы ошиблись в команде или если она работает слишком долго и вы хотите её переписать, нажмите `Ctrl+C` - это прервет выполнение программы.

Сначала выведем номера колонок (`cat` распечатает содержимое файла; `head -1` возьмёт первую строку; `tr '\t' '\n'` заменяет табуляцию в этой единственной строке на символы новой строки, таким образом каждая колонка окажется на новой строке; `cat -n` выведет все те же строки, но уже с нумерацией):
`cat memorial_lists.tsv | head -1 | tr '\t' '\n' | cat -n`

Теперь мы находим в полученном списке номера нужных нам полей. ФИО располагаются в колонках 2,3,4; национальность - в 10й, приговор - в 24й. Отлично, распечатаем только эти колонки и сохраним результат в отдельный файл `sentence_by_nation.tsv`:
`cat memorial_lists.tsv | cut -f 2,3,4,10,24  > sentence_by_nation.tsv`

Или можем, например, найти подсчитать, сколько людей каждой национальности было выслано на спецпоселение. Для этого оставим только эти две колонки, отфильтруем (при помощи grep) только те, где встречается упоминание спецпоселения и подсчитаем число упоминаний разных национальностей:
`cat memorial_lists.tsv | cut -f 10,24 | grep -i 'спецпоселение' | cut -f 1 | sort | uniq -c | sort --ignore-leading-blanks -n -k1`
В этой составной команде `grep -i` фильтрует только те строки, где упоминается спецпоселение (ключ `-i` говорит, что оно может быть написано хоть большими буквами, хоть маленькими). Каждая строка - это один человек, хоть от него и осталось только национальность и приговор.
`cut -f 1` оставляет только первую колонку (стоит напомнить, что после предыдущего вызова cut колонок у нас осталось только две: национальность и приговор, а их нумерация не имеет никакого отношения к исходной нумерации (колонки имеют номера 1 и 2 соответственно). Теперь нам следует сгруппировать людей одинаковых национальностей и подсчитать их. За это отвечает комбинация `sort | uniq -c`
И, наконец, `sort -n --ignore-leading-blanks -k1` выведет все упомянутые национальности с числом отправленных на спецпоселение в возрастающем порядке. Ключ `-n` отвечает за то, чтобы сортировать строки не по алфавиту, а как числа. `-k1` говорит, что сортировать надо по первому столбцу, а `--ignore-leading-blanks` нужно, поскольку `uniq -c` печатает не просто число встреченных записей, а еще вставляет несколько пробелов перед числом, чтобы выровнять колонки; мы просим sort эти пробелы игнорировать.