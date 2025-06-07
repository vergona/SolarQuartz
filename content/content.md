
# Подсчет заметок и свойств
= = 
> [!info] Подсчет свойств в текущей папке
> ```dvjs
> const folderPath = dv.page("").file.path.split('/').slice(0, -1).join('/');
> // автоматически указывает путь папке текущей заметки
> const notes = dv.pages(`"${folderPath}"`).where(p => p.file.name != null);
> 
> dv.header(4, `Количество заметок в папке "${folderPath}": ${notes.length}`);
> 
> // Словарь для хранения свойств и количества их появлений
> const propertiesCount = {};
> 
> // Получение свойств из заметок и подсчет их количеств
> for (const note of notes) {
>     // Используйте Object.keys чтобы получить все ключи (свойства)
>     const properties = Object.keys(note);
>     
>     for (const prop of properties) {
>         // Пропускаем стандартные свойства файла
>         if (prop !== 'file' && prop !== 'file.name' && prop !== 'file.path') {
>             propertiesCount[prop] = (propertiesCount[prop] || 0) + 1;
>         }
>     }
> }
> 
> // Сортировка свойств по количеству заметок
>const sortedProperties = Object.entries(propertiesCount).sort((a, b) => b[1] - a[1]);
> 
> // Вывод списка свойств и количества заметок
> const table = dv.table(["Свойство ", "Количество заметок"], sortedProperties);
> 
> ```

===
## Хранилище
```dvjs
const propertyName = "vaultpart";
const folderPath = dv.page("").file.path.split('/').slice(0, -1).join('/');
const pages = dv.pages(`"${folderPath}"`).where(p => p.file.name != null);

dv.header(4,`Количество заметок в папке ${folderPath}: ${pages.length}, с ${propertyName}: ${pages.where(p => p[propertyName]).length}`);


// Создаем множество для хранения уникальных значений
const uniqueValues = new Set();
// Перебираем все страницы и добавляем значения в множество
pages.forEach(page => {
    const value = page[propertyName];
    // Если значение - массив
    if (Array.isArray(value)) {value-forEach(v => uniqueValues.add(v));}
	    // Если значение - одно
	    else {uniqueValues.add(value + ' ('+pages.where(p => p[propertyName] === value).length+')');} 
});

// Преобразуем множество в массив и сортируем
const sortedValues = Array.from(uniqueValues).sort(); 
dv.list(sortedValues);


```

## Все заметки и значения выбранного поля
```dvjs
const folderName =  dv.page("").file.path.split('/').slice(0, -1).join('/');
// автоматически указывает путь папке текущей заметки
const propertyName = "vaultpart"; 
const pages = dv.pages(`"${folderName}"`).where(p => p[propertyName] !== undefined);

// все заметки из указанной папки с нужным свойством
 const sortedPages = pages.sort(p => p[propertyName], 'desc'); 

dv.header(4, `Заметки в "${folderName}" с свойством "${propertyName}" (${pages.length})`);

// таблица с заметками и значениями свойства
dv.table(
	["Персонаж",  propertyName], 
	sortedPages.map(p => [p.file.link, p[propertyName]])   );
```
```dvjs
 // Укажите здесь вашу папку и имя свойства
 const folderName =  dv.page("").file.path.split('/').slice(0, -1).join('/');
 // автоматически указывает путь папке текущей заметки
 const propertyName = "tags"; // замените на имя вашего свойства
 const pages = dv.pages(`"${folderName}"`).where(page => !page[propertyName])
  // Фильтруем заметки, которые не содержат указанное свойство
 
 
 // Выводим заголовок
 dv.header(4, `Заметки в "${folderName}" без свойства "${propertyName}" (${pages.length})`);
 
 // Выводим список заметок
 if (pages.length > 0) {
    dv.list(pages.map(page => page.file.link+" // "+ page.file.tags));
 } else {
   dv.paragraph("Заметок без свойства " + propertyName + " не найдено.");
}
```
===


## Все-все
> [!NOTE]- все свойства со всеми значениями
> ```dvjs
> // Задайте путь к папке
> const folderPath = dv.page("").file.path.split('/').slice(0, -1).join('/'); // например, "Проекты/Задачи"
> 
> // Получаем все файлы в папке
> const pages = dv.pages(`"${folderPath}"`).where(p => p.file.name != null);
> 
> // Создаем объект для хранения свойств и их значений
> const propertiesMap = {};
> 
> // Проходим по всем заметкам и собираем свойства
> for (const page of pages) {
>     if (page) {
>         const frontmatter = page-file?.frontmatter;
>         if (frontmatter) {
>             for (const [key, value] of Object.entries(frontmatter)) {
>                 if (!propertiesMap[key]) {
>                     propertiesMap[key] = {
>                         types: new Set(),
>                         values: new Set()
>                     };
>                 }
>                 // Определяем тип значения
>                 const type = typeof value;
>                 if (Array.isArray(value)) {
>                     propertiesMap[key].types.add('array');
>                     value-forEach(v => {
>                         propertiesMap[key].values.add(v);
>                     });
>                 } else {
>                     propertiesMap[key].types.add(type);
>                     propertiesMap[key].values.add(value);
>                 }
>             }
>         }
>     }
> }
> 
> // Вывод иерархического списка
> dv.header(2, `Иерархия свойств в папке: ${folderPath}`);
> 
> // Перебираем свойства и выводим их иерархически
> for (const [propName, data] of Object.entries(propertiesMap).sort()) {
>     const typesList = Array.from(data.types).join(', ');
>     dv.list([`**${propName} (тип: ${typesList})**`]);
> 
>     // Под каждым свойством — список всех уникальных значений
>     if (data.values.size > 0) {
>         dv.list(Array.from(data.values).map(v => `${v}`).sort());
>     } else {
>         dv.list([`Нет значений`]);
>     }
> }
> 
> ```

===


# Линковка
==Ссылки-сироты в заметках==
`file.folder=this.file.folder`
```dataview
TABLE without id
  file.link as "Заметка",
  rows.Outlink as "Пустая ссылка"
FLATTEN file.outlinks as Outlink
WHERE contains(file.path, this.file.folder + "/") AND
  !Outlink.file and
  !startswith(meta(Outlink).path, "images")
    GROUP BY file
```

> [!NOTE]- ссылки-сироты в другом виде
> ```dataview
> TABLE without id 
> out AS "Uncreated", file.link as "Origin"
> FLATTEN file.outlinks as out
> WHERE !(out.file) AND !contains(meta(out).path, "/")
> AND contains(file.path, this.file.folder + "/")
> SORT Uncreated ASC
> ```

## Dataview broken and orphaned links
= ==Заметки, не имеющие двухсторонних связей==
- Out=In=0 изолированные, вообще заметки не имеют связей
- Out=0 нет исходящих, то есть заметки тупиковые, дальше не ведут
- In=0 нет входящих, то есть не ясно, как выйти на заметку
```dataview
TABLE 
length(file.outlinks) as "Out", 
length(file.inlinks) as "In"
WHERE contains(file.path, this.file.folder + "/") AND 
		(length(file.outlinks) = 0 OR length(file.inlinks) = 0)

sort length(file.outlinks)+length(file.inlinks) asc 
```

