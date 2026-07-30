---
aliases:
  - датавью
cssclasses: 
tags:
  - plugin
type:
---


### Dataview

> [!Link]   **Ссылка на репозиторий** 
> https://github.com/blacksmithgu/obsidian-dataview


> [!Notes]  **Описание** 
> Плагин позволяет рассматривать вашу базу с заметками как базу данных, к которой можно делать запросы. Предоставляет API JavaScript и язык запросов на основе конвейера для фильтрации, сортировки и извлечения данных из страниц Markdown
> 

> [!Shell]  **Скриншоты**
> ![[Dataview.png]]
> 

> [!Connect]  **Фишки** и особенности использования
> Написал [[Что такое Dataview и для чего он нужен|отдельную заметку]] с чуть более подробным описанием


### Примеры полезных запросов Dataview

#### Список последних измененных заметок
- !Последние измененные заметки
- !Заметки за последние 7 дней
- В этот день... - крутая штука, для тех, кто ведет дневник

#### Список сиротских заметок
Заметка, у которой нет связей с другими заметками - сирота и потерянная единица контента. Чтобы вывести список все такие заметки, используем запрос. В данном примере исключена папка shablony, что логично. 

> [!Map]- **Заметки-сироты**
>
> ```dataview
> list
> from ""
> where length(file.inlinks) =0 and length(file.outlinks) = 0 AND !contains(file.path, "shablony")
> ```


#### Заметки без тегов
Если в вашем фреймворке ведения записей обязательно использование тегов, используем запрос, который покажет все заметки, не имеющие ни одного тега


> [!Hexagon]- Заметки без тегов  
> ```dataview
> list
> from ""
> where length(file.inlinks) =0 and length(file.outlinks) = 0 and length(file.tags) = 0
> ````
> 

#### Список всех тегов в хранилище
С указанием количества заметок, в которых этот тег используется

> [!Radar]- Все теги
> ```dataview
> TABLE length(rows.file.name) as numfiles  FROM "/" 
> flatten file.tags as tag group by tag 
> sort length(rows.file.name) desc 
> ```
> 


> [!Script]- Второй вариант  
> ```dataview
> Table without id L as "tags"
> Flatten file.tags as L
> Group by L
> ```
> 



#### Записи, созданные для сегодня (в шаблон)
Особенность данного запроса - его необходимо добавлять в [[Шаблоны заметок|шаблон]], и при создании ежедневной заметки при клике на дату в календаре конструкция `{{date:YYYY-MM-DD}}` из шаблона автоматически заменится на ту дату, за которую вы создаете заметку. Плюс небольшая "добавка" - из выдачи исключается сама ежедневная заметка, в который мы выводим этот список. Т.е. если сегодня 15 июля, мы создаем ежедневную заметку с именем 20240715, и в ней запрос выведет все заметки, созданные за 15 июля 2024 года за минусом самой заметки 20240715

```
dataiew
LIST
FROM ""
WHERE file.cday = date({{date:YYYY-MM-DD}}) AND file.name != "{{date:YYYYMMDD}}"
```

#### Записи, созданные за неделю (в шаблон)
По аналогии с предыдущим запросом - этот код вносим в шаблон, который при вставке поменяет конструкции типа {{date}} на текущую дату. Для работы шаблона понадобится [[Templater]]

```
dataview
LIST
FROM ""
WHERE file.cday > date(<% tp.date.now("YYYY-MM-DD", -7) %>) and file.cday <= date(<% tp.date.now('YYYY-MM-DD') %>) 
SORT date asc
```

#### Файлы, созданные за год, с разбивкой по папкам
Выводит списком файлы, который созданы за год, указанный в запросе. 
Кстати, если в запросе поменять year на month и в значении указать номер месяца (07 для июля например), то получим список заметок за определнный месяц

> [!Shell]- Заметки за текущий год  
> ```dataview
> LIST rows.file.link
> FROM ""
> WHERE file.cday.year = 2024
> SORT file.ctime DESC
> GROUP BY file.folder
> ```
> 

#### Список входящих и исходящих ссылок из заметки
Топ-запрос, в Map of Content можно добавлять смело

> [!Sprout]- Входящие и исходящие ссылки  
> ``` dataview 
> TABLE without id 
> map(file.outlinks, (link) => "[[" + meta(link).path + "]]") AS "Исходящие",
> map(file.inlinks, (link) => "[[" + meta(link).path + "]]") AS "Входящие" 
> WHERE file.name = this.file.name
> ```
> 

#### Список несозданных заметок, на которые есть 3 и более ссылок

> [!Connect]- Список еще не созданных заметок, на которые ссылаются как минимум `3` другие заметки
> ```dataviewjs
> const count = 3;
> let d = {};
> function process(k, v) {
>   Object.keys(v).forEach(function (x) {
>     x = dv.fileLink(x);
>     if (d[x]==undefined) { d[x] = []; }
>     d[x].push(dv.fileLink(k));
>   });
> }
> 
> Object.entries(dv.app.metadataCache.unresolvedLinks)
>     .filter(([k,v])=>Object.keys(v).length)
>     .forEach(([k,v]) => process(k, v));
>     
> dv.table(["Имя заметки", "Ссылаются"],
>          Object.entries(d)
>          .filter(([k, v]) => v.length >= count)
>          .sort((a, b) => b[1].length - a[1].length)
>          .map(([k,v])=> [k, v.join(" • ")]))
> ```
>  
>  Данный список можно рассматривать как источник для создания новых заметок. Ссылки вы уже создали, почему же не создать сами заметки?


#### Список созданных заметок, на которые есть 3 и более ссылок
Аналогичный предыдущему запрос, только теперь в таблице будут отражены созданные заметки

> [!Boxes]- Список заметок, на которые ссылаются как минимум `3` другие заметки
> ```dataviewjs
> const count = 3;
> let d = {};
> function process(k, v) {
>   Object.keys(v).forEach(function (x) {
>     x = dv.fileLink(x);
>     if (d[x]==undefined) { d[x] = []; }
>     d[x].push(dv.fileLink(k));
>   });
> }
> 
> Object.entries(dv.app.metadataCache.resolvedLinks)
>     .filter(([k,v])=>Object.keys(v).length)
>     .forEach(([k,v]) => process(k, v));
>     
> dv.table(["Имя заметки", "Ссылаются"],
>          Object.entries(d)
>          .filter(([k, v]) => v.length >= count)
>          .sort((a, b) => b[1].length - a[1].length)
>          .map(([k,v])=> [k, v.join(" • ")]))
> ```


#### Инкубатор идей, реализованный на DataviewJS

^0caee0

Отличается от обычного поискового запроса с аналогичным условием тем, что выводит строки целиком, что дает больше контекста и позволяет понять больше без перехода к источнику.

> [!Map]- Все встречающие в хранилище строки с тегом 💡
> ```dataviewjs
> const regex = /💡/im; // Регулярное выражение для поиска символа 💡
>    const current = dv.current();
>    const targetFolder = "";
>    
>    dv.pages("")
>      .filter(p => p.file.path.includes(targetFolder)) // Фильтр по папке
>      .sort(b => b.file.name)
>      .filter(p => (p.file.path != current.file.path)) // Исключаем текущий файл
>      .forEach(async p => {
>        const context = await dv.io.load(p.file.path);
>        const matches = context.split("\n")
>          .filter(line => regex.test(line)); // Фильтрация строк с тегом 💡
>    
>        if (matches.length > 0) {
>          dv.header(3, "[[" + p.file.name + "]]"); // Вывод имени файла
>          matches.forEach(line => {
>            dv.paragraph(line); // Вывод каждой строки с тегом
>          });
>        }
>      });
>    
> ```


### Выводим случайные заметки из определенной папки

Сценарий использования - чтобы база сама подсовывала нам случайно выбранные заметки из всего хранилища или из определенной папки. В примере ниже - заметки берутся из "Изучение Obsidian", в количестве трех штук. Папку и количество выводимых заметок вы можете "отрегулировать" под себя

> [!Boxes] Случайно выбранные заметки  
> ```dataview
> list
> From "Изучение Obsidian"
> Sort hash(dateformat(date(now), "YYYY-MM-DD-HH-mm-ss"), file.name) asc
> Limit 3
> ```


___

Еще больше примеров можно найти в бесплатных демо-хранилищах, например вот в [этом](https://github.com/xDovos/Dataview-Deep-Dive) или [этом](https://github.com/s-blu/obsidian_dataview_example_vault). 