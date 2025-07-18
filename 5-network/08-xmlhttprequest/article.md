
# XMLHttpRequest

`XMLHttpRequest` -- это встроенный в браузер объект, который даёт возможность делать HTTP-запросы к серверу без перезагрузки страницы.

Несмотря на наличие слова "XML" в названии, XMLHttpRequest может работать с любыми данными, а не только с XML. Мы можем загружать/скачивать файлы, отслеживать прогресс и многое другое.

На сегодняшний день не обязательно использовать `XMLHttpRequest`, так как существует другой, более современный метод `fetch`.

В современной веб-разработке `XMLHttpRequest` используется по трём причинам:

1. По историческим причинам: существует много кода, использующего `XMLHttpRequest`, который нужно поддерживать.
2. Необходимость поддерживать старые браузеры и нежелание использовать полифилы (например, чтобы уменьшить количество кода).
3. Потребность в функциональности, которую `fetch` пока что не может предоставить, к примеру, отслеживание прогресса отправки на сервер.

Что-то из этого списка звучит знакомо? Если да, тогда вперёд, приятного знакомства с `XMLHttpRequest`. Если же нет, возможно, имеет смысл изучать сразу <info:fetch>.

## Основы

XMLHttpRequest имеет два режима работы: синхронный и асинхронный.

Сначала рассмотрим асинхронный, так как в большинстве случаев используется именно он.

Чтобы сделать запрос, нам нужно выполнить три шага:

1. Создать `XMLHttpRequest`.

    ```js
    let xhr = new XMLHttpRequest(); // у конструктора нет аргументов
    ```

    Конструктор не имеет аргументов.

2. Инициализировать его.

    ```js
    xhr.open(method, URL, [async, user, password])
    ```

    Этот метод обычно вызывается сразу после `new XMLHttpRequest`. В него передаются основные параметры запроса:

    - `method` -- HTTP-метод. Обычно это `"GET"` или `"POST"`.
    - `URL` -- URL, куда отправляется запрос: строка, может быть и объект [URL](info:url).
    - `async` -- если указать `false`, тогда запрос будет выполнен синхронно, это мы рассмотрим чуть позже.
    - `user`, `password` -- логин и пароль для базовой HTTP-авторизации (если требуется).

    Заметим, что вызов `open`, вопреки своему названию, не открывает соединение. Он лишь конфигурирует запрос, но непосредственно отсылается запрос только лишь после вызова `send`.

3. Послать запрос.

    ```js
    xhr.send([body])
    ```

    Этот метод устанавливает соединение и отсылает запрос к серверу. Необязательный параметр `body` содержит тело запроса.

    Некоторые типы запросов, такие как `GET`, не имеют тела. А некоторые, как, например, `POST`, используют `body`, чтобы отправлять данные на сервер. Мы позже увидим примеры.

4. Слушать события на `xhr`, чтобы получить ответ.

    Три наиболее используемых события:
    - `load` -- происходит, когда получен какой-либо ответ, включая ответы с HTTP-ошибкой, например 404.
    - `error` -- когда запрос не может быть выполнен, например, нет соединения или невалидный URL.
    - `progress` -- происходит периодически во время загрузки ответа, сообщает о прогрессе.

    ```js
    xhr.onload = function() {
      alert(`Загружено: ${xhr.status} ${xhr.response}`);
    };

    xhr.onerror = function() { // происходит, только когда запрос совсем не получилось выполнить
      alert(`Ошибка соединения`);
    };

    xhr.onprogress = function(event) { // запускается периодически
      // event.loaded - количество загруженных байт
      // event.lengthComputable = равно true, если сервер присылает заголовок Content-Length
      // event.total - количество байт всего (только если lengthComputable равно true)
      alert(`Загружено ${event.loaded} из ${event.total}`);
    };
    ```

Вот полный пример. Код ниже загружает `/article/xmlhttprequest/example/load` с сервера и сообщает о прогрессе:

```js run
// 1. Создаём новый XMLHttpRequest-объект
let xhr = new XMLHttpRequest();

// 2. Настраиваем его: GET-запрос по URL /article/.../load
xhr.open('GET', '/article/xmlhttprequest/example/load');

// 3. Отсылаем запрос
xhr.send();

// 4. Этот код сработает после того, как мы получим ответ сервера
xhr.onload = function() {
  if (xhr.status != 200) { // анализируем HTTP-статус ответа, если статус не 200, то произошла ошибка
    alert(`Ошибка ${xhr.status}: ${xhr.statusText}`); // Например, 404: Not Found
  } else { // если всё прошло гладко, выводим результат
    alert(`Готово, получили ${xhr.response.length} байт`); // response -- это ответ сервера
  }
};

xhr.onprogress = function(event) {
  if (event.lengthComputable) {
    alert(`Получено ${event.loaded} из ${event.total} байт`);
  } else {
    alert(`Получено ${event.loaded} байт`); // если в ответе нет заголовка Content-Length
  }

};

xhr.onerror = function() {
  alert("Запрос не удался");
};
```

После ответа сервера мы можем получить результат запроса в следующих свойствах `xhr`:

`status`
: Код состояния HTTP (число): `200`, `404`, `403` и так далее, может быть `0` в случае, если ошибка не связана с HTTP.

`statusText`
: Сообщение о состоянии ответа HTTP (строка): обычно `OK` для `200`, `Not Found` для `404`, `Forbidden` для `403`, и так далее.

`response` (в старом коде может встречаться как `responseText`)
: Тело ответа сервера.

Мы можем также указать таймаут - промежуток времени, который мы готовы ждать ответ:

```js
xhr.timeout = 10000; // таймаут указывается в миллисекундах, т.е. 10 секунд
```

Если запрос не успевает выполниться в установленное время, то он прерывается, и происходит событие `timeout`.

````smart header="URL с параметрами"
Чтобы добавить к URL параметры, вида `?name=value`, и корректно закодировать их, можно использовать объект [URL](info:url):

```js
let url = new URL('https://google.com/search');
url.searchParams.set('q', 'test me!');

// параметр 'q' закодирован
xhr.open('GET', url); // https://google.com/search?q=test+me%21
```

````

## Тип ответа

Мы можем использовать свойство `xhr.responseType`, чтобы указать ожидаемый тип ответа:

- `""` (по умолчанию) -- строка,
- `"text"` -- строка,
- `"arraybuffer"` -- `ArrayBuffer` (для бинарных данных, смотрите в <info:arraybuffer-binary-arrays>),
- `"blob"` -- `Blob` (для бинарных данных, смотрите в <info:blob>),
- `"document"` -- XML-документ (может использовать XPath и другие XML-методы),
- `"json"` -- JSON (парсится автоматически).

К примеру, давайте получим ответ в формате JSON:

```js run
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/example/json');

*!*
xhr.responseType = 'json';
*/!*

xhr.send();

// тело ответа {"message": "Привет, мир!"}
xhr.onload = function() {
  let responseObj = xhr.response;
  alert(responseObj.message); // Привет, мир!
};
```

```smart
В старом коде вы можете встретить свойства `xhr.responseText` и даже `xhr.responseXML`.

Они существуют по историческим причинам, раньше с их помощью получали строки или XML-документы. Сегодня следует устанавливать желаемый тип объекта в `xhr.responseType` и получать `xhr.response`, как показано выше.
```

## Состояния запроса

У `XMLHttpRequest` есть состояния, которые меняются по мере выполнения запроса. Текущее состояние можно посмотреть в свойстве `xhr.readyState`.

Список всех состояний, указанных в [спецификации](https://xhr.spec.whatwg.org/#states):

```js
UNSENT = 0; // исходное состояние
OPENED = 1; // вызван метод open
HEADERS_RECEIVED = 2; // получены заголовки ответа
LOADING = 3; // ответ в процессе передачи (данные частично получены)
DONE = 4; // запрос завершён
```

Состояния объекта `XMLHttpRequest` меняются в таком порядке: `0` -> `1` -> `2` -> `3` -> ... -> `3` -> `4`. Состояние `3` повторяется каждый раз, когда получена часть данных.

Изменения в состоянии объекта запроса генерируют событие `readystatechange`:

```js
xhr.onreadystatechange = function() {
  if (xhr.readyState == 3) {
    // загрузка
  }
  if (xhr.readyState == 4) {
    // запрос завершён
  }
};
```

Вы можете наткнуться на обработчики события `readystatechange` в очень старом коде, так уж сложилось исторически, когда-то не было событий `load` и других. Сегодня из-за существования событий `load/error/progress` можно сказать, что событие `readystatechange` "морально устарело".

## Отмена запроса

Если мы передумали делать запрос, можно отменить его вызовом `xhr.abort()`:

```js
xhr.abort(); // завершить запрос
```

При этом генерируется событие `abort`, а `xhr.status` устанавливается в `0`.

## Синхронные запросы

Если в методе `open` третий параметр `async` установлен на `false`, запрос выполняется синхронно.

Другими словами, выполнение JavaScript останавливается на `send()` и возобновляется после получения ответа. Так ведут себя, например, функции `alert` или `prompt`.

Вот переписанный пример с параметром `async`, равным `false`:

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/article/xmlhttprequest/hello.txt', *!*false*/!*);

try {
  xhr.send();
  if (xhr.status != 200) {
    alert(`Ошибка ${xhr.status}: ${xhr.statusText}`);
  } else {
    alert(xhr.response);
  }
} catch(err) { // для отлова ошибок используем конструкцию try...catch вместо onerror
  alert("Запрос не удался");
}
```

Выглядит, может быть, и неплохо, но синхронные запросы используются редко, так как они блокируют выполнение JavaScript до тех пор, пока загрузка не завершена. В некоторых браузерах нельзя прокручивать страницу, пока идёт синхронный запрос. Ну а если же синхронный запрос по какой-то причине выполняется слишком долго, браузер предложит закрыть "зависшую" страницу.

Многие продвинутые возможности `XMLHttpRequest`, такие как выполнение запроса на другой домен или установка таймаута, недоступны для синхронных запросов. Также, как вы могли заметить, ни о какой индикации прогресса речь тут не идёт.

Из-за всего этого синхронные запросы используют очень редко. Мы более не будем рассматривать их.

## HTTP-заголовки

`XMLHttpRequest` умеет как указывать свои заголовки в запросе, так и читать присланные в ответ.

Для работы с HTTP-заголовками есть 3 метода:

`setRequestHeader(name, value)`
: Устанавливает заголовок запроса с именем `name` и значением `value`.

    Например:

    ```js
    xhr.setRequestHeader('Content-Type', 'application/json');
    ```

    ```warn header="Ограничения на заголовки"
    Некоторые заголовки управляются исключительно браузером, например `Referer` или `Host`, а также ряд других.
    Полный список [тут](https://fetch.spec.whatwg.org/#forbidden-request-header).

    `XMLHttpRequest` не разрешено изменять их ради безопасности пользователей и для обеспечения корректности HTTP-запроса.
    ```

    ````warn header="Поставленный заголовок нельзя снять"
    Ещё одной особенностью `XMLHttpRequest` является то, что отменить `setRequestHeader` невозможно.

    Если заголовок определён, то его нельзя снять. Повторные вызовы лишь добавляют информацию к заголовку, а не перезаписывают его.

    Например:

    ```js
    xhr.setRequestHeader('X-Auth', '123');
    xhr.setRequestHeader('X-Auth', '456');

    // заголовок получится такой:
    // X-Auth: 123, 456
    ```
    ````

`getResponseHeader(name)`
: Возвращает значение заголовка ответа `name` (кроме `Set-Cookie` и `Set-Cookie2`).

    Например:

    ```js
    xhr.getResponseHeader('Content-Type')
    ```

`getAllResponseHeaders()`
: Возвращает все заголовки ответа, кроме `Set-Cookie` и `Set-Cookie2`.

    Заголовки возвращаются в виде единой строки, например:

    ```http
    Cache-Control: max-age=31536000
    Content-Length: 4260
    Content-Type: image/png
    Date: Sat, 08 Sep 2012 16:53:16 GMT
    ```

    Между заголовками всегда стоит перевод строки в два символа `"\r\n"` (независимо от ОС), так что мы можем легко разделить их на отдельные заголовки. Значение заголовка всегда отделено двоеточием с пробелом `": "`. Этот формат задан стандартом.

    Таким образом, если хочется получить объект с парами заголовок-значение, нам нужно задействовать немного JS.

    Вот так (предполагается, что если два заголовка имеют одинаковое имя, то последний перезаписывает предыдущий):

    ```js
    let headers = xhr
      .getAllResponseHeaders()
      .split('\r\n')
      .reduce((result, current) => {
        let [name, value] = current.split(': ');
        result[name] = value;
        return result;
      }, {});

    // headers['Content-Type'] = 'image/png'
    ```

## POST, FormData

Чтобы сделать POST-запрос, мы можем использовать встроенный объект [FormData](https://developer.mozilla.org/ru/docs/Web/API/FormData).

Синтаксис:

```js
let formData = new FormData([form]); // создаём объект, по желанию берём данные формы <form>
formData.append(name, value); // добавляем поле
```

Мы создаём объект, при желании указываем, из какой формы `form` взять данные, затем, если нужно, с помощью метода `append` добавляем дополнительные поля, после чего:

1. `xhr.open('POST', ...)` – создаём `POST`-запрос.
2. `xhr.send(formData)` – отсылаем форму серверу.

Например:

```html run refresh
<form name="person">
  <input name="name" value="Петя">
  <input name="surname" value="Васечкин">
</form>

<script>
  // заполним FormData данными из формы
  let formData = new FormData(document.forms.person);

  // добавим ещё одно поле
  formData.append("middle", "Иванович");

  // отправим данные
  let xhr = new XMLHttpRequest();
  xhr.open("POST", "/article/xmlhttprequest/post/user");
  xhr.send(formData);

  xhr.onload = () => alert(xhr.response);
</script>
```

Обычно форма отсылается в кодировке `multipart/form-data`.

Если нам больше нравится формат JSON, то используем `JSON.stringify` и отправляем данные как строку.

Важно не забыть поставить соответствующий заголовок `Content-Type: application/json`, многие серверные фреймворки автоматически декодируют JSON при его наличии:

```js
let xhr = new XMLHttpRequest();

let json = JSON.stringify({
  name: "Вася",
  surname: "Петров"
});

xhr.open("POST", '/submit')
xhr.setRequestHeader('Content-type', 'application/json; charset=utf-8');

xhr.send(json);
```

Метод `.send(body)` весьма всеяден. Он может отправить практически что угодно в `body`, включая объекты типа `Blob` и `BufferSource`.

## Прогресс отправки

Событие `progress` срабатывает только на стадии загрузки ответа с сервера.

А именно: если мы отправляем что-то через `POST`-запрос, `XMLHttpRequest` сперва отправит наши данные (тело запроса) на сервер, а потом загрузит ответ сервера. И событие `progress` будет срабатывать только во время загрузки ответа.

Если мы отправляем что-то большое, то нас гораздо больше интересует прогресс отправки данных на сервер. Но `xhr.onprogress` тут не поможет.

Существует другой объект, без методов, только для отслеживания событий отправки: `xhr.upload`.

Он генерирует события, похожие на события `xhr`, но только во время отправки данных на сервер:

- `loadstart` -- начало загрузки данных.
- `progress` -- генерируется периодически во время отправки на сервер.
- `abort` -- загрузка прервана.
- `error` -- ошибка, не связанная с HTTP.
- `load` -- загрузка успешно завершена.
- `timeout` -- вышло время, отведённое на загрузку (при установленном свойстве `timeout`).
- `loadend` -- загрузка завершена, вне зависимости от того, как -- успешно или нет.

Примеры обработчиков для этих событий:

```js
xhr.upload.onprogress = function(event) {
  alert(`Отправлено ${event.loaded} из ${event.total} байт`);
};

xhr.upload.onload = function() {
  alert(`Данные успешно отправлены.`);
};

xhr.upload.onerror = function() {
  alert(`Произошла ошибка во время отправки: ${xhr.status}`);
};
```

Пример из реальной жизни: загрузка файла на сервер с индикацией прогресса:

```html run
<input type="file" onchange="upload(this.files[0])">

<script>
function upload(file) {
  let xhr = new XMLHttpRequest();

  // отслеживаем процесс отправки
*!*
  xhr.upload.onprogress = function(event) {
    console.log(`Отправлено ${event.loaded} из ${event.total}`);
  };
*/!*

  // Ждём завершения: неважно, успешного или нет
  xhr.onloadend = function() {
    if (xhr.status == 200) {
      console.log("Успех");
    } else {
      console.log("Ошибка " + this.status);
    }
  };

  xhr.open("POST", "/article/xmlhttprequest/post/upload");
  xhr.send(file);
}
</script>
```

## Запросы на другой источник

`XMLHttpRequest` может осуществлять запросы на другие сайты, используя ту же политику CORS, что и [fetch](info:fetch-crossorigin).

Точно так же, как и при работе с `fetch`, по умолчанию на другой источник не отсылаются куки и заголовки HTTP-авторизации. Чтобы это изменить, установите `xhr.withCredentials` в `true`:

```js
let xhr = new XMLHttpRequest();
*!*
xhr.withCredentials = true;
*/!*

xhr.open('POST', 'http://anywhere.com/request');
...
```

Детали по заголовкам, которые при этом необходимы, смотрите в главе [fetch](info:fetch-crossorigin).

## Итого

Типичный код GET-запроса с использованием `XMLHttpRequest`:

```js
let xhr = new XMLHttpRequest();

xhr.open('GET', '/my/url');

xhr.send();

xhr.onload = function() {
  if (xhr.status != 200) { // HTTP ошибка?
    // обработаем ошибку
    alert( 'Ошибка: ' + xhr.status);
    return;
  }

  // получим ответ из xhr.response
};

xhr.onprogress = function(event) {
  // выведем прогресс
  alert(`Загружено ${event.loaded} из ${event.total}`);
};

xhr.onerror = function() {
  // обработаем ошибку, не связанную с HTTP (например, нет соединения)
};
```

Событий на самом деле больше, в [современной спецификации](https://www.w3.org/TR/XMLHttpRequest/#events) они все перечислены в том порядке, в каком генерируются во время запроса:

- `loadstart` -- начало запроса.
- `progress` -- прибыла часть данных ответа, тело ответа полностью на данный момент можно получить из свойства `responseText`.
- `abort` -- запрос был прерван вызовом `xhr.abort()`.
- `error` -- произошла ошибка соединения, например неправильное доменное имя. Событие не генерируется для HTTP-ошибок как, например, 404.
- `load` -- запрос успешно завершён.
- `timeout` -- запрос был отменён по причине истечения отведённого для него времени (происходит, только если был установлен таймаут).
- `loadend` -- срабатывает после `load`, `error`, `timeout` или `abort`.

События `error`, `abort`, `timeout` и `load` взаимно исключают друг друга - может произойти только одно из них.

Наиболее часто используют события завершения загрузки (`load`), ошибки загрузки (`error`), или мы можем использовать единый обработчик `loadend` для всего и смотреть в свойствах объекта запроса `xhr` детали произошедшего.

Также мы уже видели событие: `readystatechange`. Исторически оно появилось одним из первых, даже раньше, чем была составлена спецификация. Сегодня нет необходимости использовать его, так как оно может быть заменено современными событиями, но на него можно часто наткнуться в старом коде.

Если же нам нужно следить именно за процессом отправки данных на сервер, тогда можно использовать те же события, но для объекта `xhr.upload`.
