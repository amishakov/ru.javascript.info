Ответ: **ошибка**.

Попробуйте запустить этот код:

```js run
let x = 1;

function func() {
*!*
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
*/!*
  let x = 2;
}

func();
```

В этом примере мы можем наблюдать характерную разницу между "несуществующей" и "неинициализированной" переменной.

Как вы могли прочитать в статье [](info:closure), переменная находится в неинициализированном состоянии с момента входа в блок кода (или функцию). И остается неинициализированной до соответствующего оператора `let`.

Другими словами, переменная технически существует, но не может быть использована до `let`.

Приведенный выше код демонстрирует это.

```js
function func() {
*!*
  // локальная переменная x известна движку с самого начала выполнения функции,
  // но она является неинициализированной до let ("мёртвая зона")
  // следовательно, ошибка
*/!*

  console.log(x); // ReferenceError: Cannot access 'x' before initialization

  let x = 2;
}
```

Эту зону временной непригодности переменной (от начала блока кода до `let`) иногда называют "мёртвой зоной".
