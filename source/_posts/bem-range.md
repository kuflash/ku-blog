title: Как я БЭМ-компонент писал
date: 2015-03-15 14:58:58
tags:
---
28 января 2015 года ребята из [LoftBlog](http://loftblog.ru/) совместно с компанией Яндекс провели [вебинар](https://ru.bem.info/talks/loftschool-music-2015/), где рассказали и показали, как использовать БЭМ-стек в своих проектах. В конце мероприятия [Николай Ильченко](https://events.yandex.ru/lib/people/610400/) объявил о старте конкурса, в рамках которого, используя БЭМ-стек технологий, необходимо было разработать проект (статическая HTML страница, веб-приложение, библиотека блоков и т.п.). Тут я для себя и решил, что пора попробовать полный БЭМ-стек на вкус. До этого приходилось использовать только способ именования CSS классов.

##Выбор темы

Я долго не мог определиться с темой, но, однажды в твиттере, наткнулся на [ссылку](https://twitter.com/webstandards_ru/status/565465353128783873) с примерами стилизации HTML 5 слайдера `<input type=”range”>`. Вдохновившись этими примерами было решено, что стоит написать такой компонент используя БЭМ-стек.

##Стартуем!

Для начала я стал изучать устройство библиотеки [bem-components](https://ru.bem.info/libs/bem-components/v2.0.0/), чтобы понять как оформлять код будущего блока. Оказалось, что библиотека -  это просто набор блоков со стандартной организацией файлов и каталогов. А раз так, то можно спокойно приступать к работе.

Перво-наперво я установил yoman генератор [generator-bem-stub](https://ru.bem.info/tools/bem/bem-stub/), для создания заготовки проекта. Это идеальный способ для старта разработки проекта на БЭМ-стеке. При запуске генератора  вам предложат выбрать технологи, которые буду использоваться при разработке. Я не буду описывать подробности работы данного генератора. Необходимую информацию вы найдете по этой [ссылке](https://ru.bem.info/tools/bem/bem-stub/).

Запустить проект можно командой `enb server`

##Структура блока

Далее необходимо было представить компонент в терминах БЭМ-методологии. Получилось следующее:
- блок **range** - основа блока;
  - модификатор **theme** - служит для изменения внешнего вида слайдера;
  - элемент **title** - заголовок слайдера;
  - элемент **box** - необходим для группировки слайдера с элементом для отображения текущего значения;
  - элемент **control** - представление тега `<input type=”range”>`;
  - элемент **value** - необходим для отображения текущего значения слайдера;
    - модификатор **type**- служит для определения поведения элемента. Имеет два состояния:
      - **static** - просто статичное отображение значения;
      - **tooltip** - при изменении значения слайдера появляется под курсором мышки.

На самом деле это окончательный вариант структуры. До него я несколько раз все менял. Сказывается неопытность в выделении сущностей согласно БЭМ-методологии.

##Стили

После создания структуры блока я принялся писать стили для слайдера. В качестве препроцессора в стандартной комплектации БЭМ-проекта предоставляется Stylus. Опыта работы с ним не было, но у меня есть небольшой опыт использования Less, поэтому я быстро втянулся. Даже если вы не пользовались препроцессорами, то не пугайтесь, Stylus поддерживает обычный CSS синтаксис.

Для начала стили для блока **range**:

```css
.range {
  position: relative;
}
```
Все. Этого достаточно. Большая часть стилей будет как раз для компонента `<input type=”range”>`.

Далее идут элементы. Первым будет **title** - заголовок/лэйбл слайдера:

```css
.range__title {
  display: inline-block;
  vertical-align: middle;
  margin-right: 10px;
}
```

Следующий элемент **box** - обертка для тега input и элемента со значением слайдера:

```css
.range__box {
  position: relative;
  display: inline-block;
}
```

Элемент **value** - необходим для отображения выбранного значения слайдера:

```css
.range__value {
  display: inline-block;
  vertical-align: middle;
  margin-left: 10px;
}
```

Ну и напоследок самое вкусное)) Стили для элемента **control**, который является тегом `<input type=”range”>`:

```css
.range__control {
  position: relative;
  display: inline-block;
  vertical-align: middle;
}
```
Вы наверное сейчас удивились. Что тут интересного?? Где стили для внешнего вида слайдера??

На самом деле вся кастомизация находится в модификаторе **theme**. Это создано для того, чтобы по умолчанию слайдер имел стандартный системный вид. А уж если разработчик захочет изменить внешний вид, то он просто создаст новую тему, или воспользуется уже поставляемой в библиотеке. Сейчас я все опишу.

В стилизации HTML 5 слайдера мне помогли примеры [Анны Тьюдор](https://twitter.com/anatudor). Она создает восхитительные [примеры](http://codepen.io/collection/DgYaMj/) по стилизации этого компонента. Для изменения внешнего вида слайдера используются *вендорные псевдоэлементы*. Каждый браузер имеет по два псевдоэелемента:
- псевдоэлемент для стилей “трека” (линии), по которому двигается “бегунок”;
- псевдоэлемент для стилей “бегунка”.

Исключением является IE (да, да, опять IE). Но в этот раз он опередил своих товарищей. Помимо перечисленных выше, у него есть два дополнительных псевдоэлемента:
- псевдоэлемент для стилей выбранного промежутка;
- псевдоэлемент для стилей не выбранного промежутка.

Данную функциональность у других браузеров приходится реализовывать с помощью JavaScript. Способ состоит в том, чтобы динамически изменять свойство `background-size` у псевдоэлемента для "трека", путем добавления стилей в тег `<style>` на странице. К сожалению другого способа я не нашел.

[Вот](https://github.com/kuflash/bem-range/blob/master/common.blocks/range/_theme/range_theme_islands.styl) пример стилей для темы **Islands**.

##JavaScript

После написания стилей я приступил к реализации блока в JavaScript технологии. В БЭМ-стеке имеется фреймворк [i-bem.js](https://ru.bem.info/technology/i-bem/v2/i-bem-js/), который предоставляет удобное API для работы с блоками в БЭМ-терминах. Весь код я расположил в одном файле [range.js](https://github.com/kuflash/bem-range/blob/master/common.blocks/range/range.js) в корневом каталоге. Большая его часть относится как раз к реализации заливки выбранного диапазона. Сейчас я опишу все по порядку.

Во первых, делаем обертку модуля нашего блока:

```js
modules.define('range', ['i-bem__dom', 'jquery'], function (provide, BEMDOM, $) {
  provide(BEMDOM.decl(this.name,
    { /* методы экземпляра блока */ },
    { /* статические методы блока */ }
  );
});
```

Тут мы определяем название блока и его зависимости. Наш слайдер будет зависеть от элемента dom блока i-bem, т.к. слайдер имеет DOM представление. А также нужен jquery, для работы с DOM деревом.

Далее напишем что должно происходить при инициализации блока (эта часть описывается в области для методов экземпляра блока):
```js
onSetMod: {
  js: {
    inited: function () {
      // сохраняем ссылку на элемент
      this.control = this.findElem('control');

      // сохраняем идентификатор элемента control
      this._id = this.control.attr('id');

      // слушаем изменение значения слайдера
      this.bindTo('control', 'change', this._onChange, this);

      // слушаем клик на слайдере
      this.bindTo('control', 'mousedown', this._onMouseDown, this);

      // слушаем когда пользователь закончил работать со слайдером
      this.bindTo('control', 'mouseup', this._onMouseUp, this);

      // сразу же вызываем событие изменения слайдера.
      // Чтобы при инициализации все необходимые стили
      // установились к нашему блоку.
      this.control.trigger('change');
    }
  }
}
```

Опишу сразу метод создания стилей для заливки выбранного диапазона, т.к. он используется в методах-слушателях (он также описывается в области для методов экземпляра блока):
```js
 // список вендорных псевдоэлементов
_trackSelectors: ['::-webkit-slider-runnable-track', '::-moz-range-track'],

_updateFillTrack: function () {

  // максимальное значение слайдера
  var max = Number(this.control.attr('max'));

  // минимальное значение слайдера
  var min = Number(this.control.attr('min'));

  // на основе выше полученных данных
  // вычисляем значение выбранного диапазона в процентах
  // и формируем строку с новым значением свойства background-size.
  var property = 'background-size:' + 100 * (this.getVal() - min) / (max - min) + '% 100%, 100% 100%';

  // переменная для хранения правила для определенного псевдоэлемента
  var rule = '';

  //переменная для хранения всех стилей для блока
  var rules = '';

  // пробегаемся по всем псевдоэлементам и формируем css блок
  this._trackSelectors.forEach(function (value, index) {
    var selector = '#' + this._id + value;
    rule += selector + '{' + property + '}';
  }, this);

  // сохраняем стиль каждого бока в статическую переменную
  this.__self.cssRules[this._id] = rule;

  // пробегаемся по всем стилям всех блоков и формируем строку для тега style.
  for (var index in this.__self.cssRules) rules += this.__self.cssRules[index];

  // обновляем стили в теге style
  $('style.range-styles').text(rules);
}
```

Код конечно не идеален, но я планирую рефакторинг этой части + при подведении итогов конкурса пообещали предоставить ревью проекта.

Теперь можно разобрать код слушателей событий:

```js
// основной слушатель изменения значения слайдера
_onChange: function (event) {

  // проверка если элемент со значением слайдера имеет тип tooltip,
  // то необходимо изменить его позицию
  if (this.hasMod(this.elem('value'), 'type', 'tooltip')) this._updateTooltipPosition(event.originalEvent);

  // изменяет содержимое элемента со значение слайдера
  // на обновленное значение
  this.findElem('value').text(this.getVal());

  // обновляем заливку выбранного диапазона
  this._updateFillTrack();

  // также необходимо вызвать собственное событие,
  // чтобы другие блоки могли его слушать
  this.emit('change');
},

// событие, которые вызвается при нажатии на слайдер
_onMouseDown: function (event) {
  // необходимо показать элемент со значением слайдера, если он является тултипом
  if (this.hasMod(this.elem('value'), 'type', 'tooltip')) this.elem('value').show();

  // подписываемся на событие движения мышкой на слайдере
  this.bindTo('control', 'mousemove', this._onChange, this);

  // при нажатии также необходимо изменить значение слайдера
  this._onChange(event);
},

// событие при окончании работы со сладйером
// (когда пользователь отпускает левую кнопку мыши)
_onMouseUp: function (event) {
  //  скрываем элемент со значением слайдера, если он является тултипом
  if (this.hasMod(this.elem('value'), 'type', 'tooltip')) this.elem('value').hide();

  // отписываемся на событие движения мышки по слайдеру
  this.unbindFrom('control', 'mousemove', this._onChange);
}
```
Остальная часть javascript кода не очень интересна. Полный листинг вы можете посмотреть по [этой](https://github.com/kuflash/bem-range/blob/master/common.blocks/range/range.js) ссылке.

##Шаблонизация

Самым сложным в разработке для меня оказалось понимание шаблонизатора **BEMHTML**. Некоторые моменты до сих пор остаются для меня не понятными, но думаю это дело времени и практики. В BEMHTML для описания шаблонов используется декларативный подход, что несколько ломает мозг))) Если упростить описание, то это что-то типа CSS для HTML (не помню где вычитал это сравнение). Суть в том, что мы пишем условия, при выполнении которых возвращается определенная BEMJSON структура блока. Документация по BEMHTML находится по этой [ссылке](https://ru.bem.info/technology/bemhtml/v2/intro/). Сейчас я постараюсь описать свою логику составления шаблонов для слайдера.

Зачем вообще нужны шаблоны для блоков? Представьте, что мы разрабатываем блок с очень сложной структурой, где есть разные варианты вложенности элементов. Программисту, использующему наш  блок, не очень хочется описывать всю эту структуру в своем BEMJSON дереве. Было бы здорово написать так:

```js
{
  block: 'block-name',
  param1: 'value1',
  param2: 'value2',
}
```

А это все развернулось бы в нужный для блока BEMJSON объект. Вот и ответ на вопрос “Зачем нужны шаблоны для блоков?”

Напомню структуру нашего блока:

```
range
  __title // необязательный элемент
  __box
    __control
    __value // необязательный элемент
```

Помимо структуры блока имеются еще и различные атрибуты:
- id - уникальный идентификатор блока;
- max - максимальное значение слайдера;
- min - минимальное значение слайдера;
- value - первоначальное значение слайера.

Учитывая вышесказанное, определим BEMJSON структуру блока для удобного использования в разных проектах.

```js
{
  // собственно сам блок
  block: 'range',

  // его тема
  mods: { theme: 'islands' },

  // заголовок, если его не передать то выполнится шаблон
  // без добавления заголовка
  title: 'Range 1',

  // свойство определяющее тип элемента для отображения
  // текущего значения слайдера. Если его не передать,
  // то выполнится шаблон без вставки этого элемента
  displayedValue: { type: 'tooltip' },

  // атрибут минимального значения слайдера
  min: 0,

  // атрибут для максимального значения слайдера
  max: 50,

  // атрибут для первоначального значения слайдера
  val: 40
}
```
Теперь переходим непосредственно к самим шаблонам:

Шаблон [range.bemhtml](https://github.com/kuflash/bem-range/blob/master/common.blocks/range/range.bemhtml)

```js
// говорим шаблону чтобы он отрабатывался, если встречается блок range
block('range')(
  // говорим, что блок имеет js реализацию
  js()(true),

  // этот кусок выполняется всегда,
  // тут мы просто запоминаем текущий контекст,
  // чтобы можно было к нему обратится во внутренних шаблонах
  def()(function(){
    applyNext({_range: this.ctx});
  }),

  // а вот и пошли условия.
  // тут мы проверяем, если пользователь передал в BEMJSON поле title,
  // то вернем структуру для этого элемента
  content()(
    match(this.ctx.title)
    (
      function() {
        return [{
          elem: 'title',
          tag: 'label',
          attrs: {
            // автоматическая генерация атрибута for
            for: this.generateId()
          },
          // вставляем переданный текст в этот элемент
          content: this.ctx.title
        }]
      }
    )
  ),

  // этот кусок выполняется всегда.
  // он генерирует обертку и элемент control,
  // передавая все нужные атрибуты
  content()(
    function () {
      return [
        applyNext(),
        {
          elem: 'box',
          content: {
            elem: 'control',
            tag: 'input',
            attrs: {
              type: 'range',
              id: this.generateId(),
              min: this.ctx.min || 0,
              max: this.ctx.max || 100,
              value: this.ctx.val
            }
          }
        }
      ]
    }
  )
);
```

Ниже представлен код шаблона для элемента **box**. Отдельный шаблон необходим, так как мы имеем в этом элементе необязательный элемент **value**:

Шаблон [range__box.bemhtml](https://github.com/kuflash/bem-range/blob/master/common.blocks/range/__box/range__box.bemhtml)

```js
// шаблон выполнится когда в BEMJSON будет найден элемент **box**
block('range').elem('box')(

  // тут мы обращаемся к ранее сохраненному контексту
    // и проверяем было ли определено поле displayedValue
  content()
    (match(this._range.displayedValue)
    (
      function() {
        return [
          applyNext(),
          {
            elem: 'value',
            tag: 'span',
            mods: this._range.displayedValue
          }
        ]
      }
    )
  )
)
```

По шаблонам все.

Если вы не сталкивались с написанием шаблонов на BEMHTML, то советую для начала почитать [документацию](https://ru.bem.info/technology/bemhtml/v2/reference/), так как это не очень простая тема. А если вы уже знаток, то буду рад советам по исправлению моих косяков))

##Зависимости

Осталось создать последний необходимый файл - **deps.js**, необходимый для описания зависимостей блока (какие элементы и модификаторы ему нужны для нормальной работы). Вот содержимое этого файла:

```js
({
  // т.к. наш блок имеет js реализацию,
  // то прописываем зависимость от элемента dom блока i-bem
  mustDeps : { block : 'i-bem', elem : 'dom' },
  shouldDeps: [

    // зависимость от всех необходимых элементов
    { elems: ['box', 'control', 'title', 'value'] },

    // зависимость от модификатора type_static элемента value
    { elem: 'value', mods: { 'type': 'static'} },

    // зависимость от модификатора type_tooltip элемента value
    { elem: 'value', mods: { 'type': 'tooltip'} }
  ]
})
```

Все, что необходимо для нормальной работы блока написано. Я написал еще [документацию](https://github.com/kuflash/bem-range/blob/master/common.blocks/range/range.ru.md), где описал все нюансы использования блока, а также создал [демо-страницу](http://bem-range-demo.bitballoon.com/).

##Заключение

В итоге мы получили рабочий БЭМ-блок, который можно использовать в своих проектах. Я продолжу его улучшать и исправлять найденные баги. Необходимо будет добавить возможность создавать вертикальные слайдеры.

##Полезные ссылки

1. [Официальный сайт БЭМ-методологии](https://ru.bem.info) - различная документация по методологии, статьи, руководства, а также замечательный форум, где вам обязательно помогут
2. [Репозиторий проекта](https://github.com/kuflash/bem-range/)
3. [Документайия блока](https://github.com/kuflash/bem-range/wiki/Documentation)
4. [Демо-страница](http://bem-range-demo.bitballoon.com/)

*__P.S.__ Не знаю много ли людей дочитают эту статью до конца. В любом случае всем спасибо за уделенное внимание. Это была моя первая статья. Уверен, что есть огрехи. Буду рад услышать конструктивные замечания и постараюсь все исправить.*








