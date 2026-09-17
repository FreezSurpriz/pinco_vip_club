# CMS Notes: Daily Cashback Layout1 

Этот файл ведем как накопительный лог.
Правило: новые кейсы только добавляем, старые записи не удаляем.

## Как дополнять
1. Добавлять новый блок в `## История правок` с датой.
2. Указывать источник (`CMS`, `клиентский файл`, `manual`).
3. Фиксировать что сломалось, что поменяли и где (`index.html`, `turkish_ver.html`).

## История правок

### 2026-06-08 — CMS-hardening из файла клиента
Источник: `gemini-code-1780920221093.html`

Проблема:
- CMS может вырезать `<link>` из `<head>`, заворачивать контент в лишние `<p>` и ломать скрытые переключатели или сетки.

Доп. правило по типографике:
- В клиентском файле жёстко фиксировать `font-family`, `font-size`, `line-height`, `font-weight` и `font-synthesis` через `!important`, чтобы CMS не могла перетирать размеры и насыщенность шрифтов.
- Если CMS всё равно подменяет шрифт, дублировать `font-family: 'Poppins', Arial, sans-serif !important;` не только на `#html-content-type-area *`, но и на текстовые теги/классы (`h1`, `h2`, `p`, `a`, `span`, `label`, `button`, CTA и табы).

Что внесено:
- Шрифт `Poppins` продублирован через `@import` внутри `<style>`, чтобы не зависеть только от `<link>`.
- Для всех элементов внутри `#html-content-type-area` принудительно задан `font-family: 'Poppins', Arial, sans-serif !important;`.
- Скрытые `radio` переведены в полностью скрытый доступный паттерн:
  - `position: absolute !important`
  - `width/height: 1px !important`
  - `margin: -1px !important`
  - `clip: rect(0,0,0,0) !important`
- Для оберток, которые CMS может оборачивать в лишние `<p>`, применен `display: contents !important;`.
- Для интерактивной зоны календаря добавлен отдельный wrapper, чтобы переключение табов и панелей не зависело от структуры вне блока.
- Для карточек и сеток использованы `!important`, `width: 100%`, `max-width` и `min-width: 0`, чтобы CMS-обертки не ломали раскладку.

Важно:
- Если CMS снова начнет вырезать `<link>` из `head`, оставлять дублирующий `@import` в CSS.
- Если скрытые инпуты снова начнут ловить фокус/скролл, использовать скрытие через `clip` вместо простого `display: none` только после проверки поведения переключателей.
- Если CMS оборачивает карточки в лишние `<p>`, сначала проверять `display: contents` на обертках, а не переписывать всю сетку.

### 2026-06-06 — обязательный скоуп CSS под `#html-content-type-area`
Источник: `manual` + World Cup 2026 LP

Проблема:
- В CMS стили должны быть ограничены контейнером `#html-content-type-area`, иначе они могут не примениться корректно или затронуть соседние блоки страницы.

Что внесено:
- Все селекторы в `<style>` должны начинаться с `#html-content-type-area`.
- Вся HTML-разметка LP должна быть обернута в `<div id="html-content-type-area">...</div>`.
- CSS-переключатели без JS (`input:checked ~ ...`) должны находиться внутри этого контейнера, чтобы скоуп не ломал логику.

Важно:
- Не оставлять глобальные селекторы вида `body`, `img`, `a`, `*`, `.class` без префикса `#html-content-type-area`.
- Если нужен сброс для картинок, ссылок или box-sizing, писать через `#html-content-type-area img`, `#html-content-type-area a`, `#html-content-type-area *`.

### 2026-05-12 — CMS-safe фиксы для карточек и мобильных `<400px`
Источник: `gemini-code-1778596475695.html` + баги в CMS

Проблема:
- CMS иногда заворачивает `.dc-choice` в лишние `<p>/<div>`, из-за этого карточки уходят в 1 колонку.
- На экранах `<400px` искажались текст и кнопки в нижних блоках.

Что внесено:
- Для `.dc-choice` добавлены принудительные правила:
  - `display: grid !important`
  - `grid-template-columns: repeat(2, minmax(0, 1fr)) !important`
  - `width: 100% !important`
- Добавлено выпрямление CMS-оберток:
  - `.dc-choice > p, .dc-choice > div { display: contents !important; margin:0 !important; padding:0 !important; }`
- Добавлен fallback без `:has(...)`:
  - селекторы через `#bonus-left:checked ~ .dc-wrap ...`
  - селекторы через `#bonus-right:checked ~ .dc-wrap ...`
- Добавлен брейкпоинт `@media (max-width: 400px)`:
  - корректировка отступов,
  - уменьшение кнопок в карточках,
  - стабилизация типографики CTA/bonus.

Важно:
- Не переносить автоматически инвертированную логику переключения из сторонних файлов.
- Перед переносом проверять соответствие ТЗ и порядок секций `.dc-bonus-left/.dc-bonus-right`.

## Чеклист перед публикацией
1. Проверить ширины: `414`, `393`, `390`, `384`, `360`.
2. Проверить, что карточки выбора всегда в 2 колонки.
3. Проверить, что `View Details` не наезжает на соседние блоки.
4. Проверить CTA (`Take the survey`) на `<400px`.
5. Проверить логику переключения левого/правого бонуса.

---

## База из прошлых проектов (Welcome Bonus LP)

### Быстрый чеклист перед отправкой в CMS
1. Модалки:
- Не использовать `:target` + `href="#..."`.
- Использовать `input[type="checkbox"]` + `label for="..."` + `:has(...)`.

2. Скрытые переключатели:
- Добавить 5 инпутов:
- `modal-info-toggle`
- `modal-first-toggle`
- `modal-second-toggle`
- `modal-third-toggle`
- `modal-fourth-toggle`

3. Открытие/закрытие модалок:
- Иконки info: `label` с `for="modal-...-toggle"`.
- Кнопка закрытия `×`: тоже `label for="modal-...-toggle"`.
- Backdrop: `label for="modal-...-toggle"` (не `a href="#"`).

4. Ссылки:
- Кнопки Deposit: `href="/deposit"`.
- T&C: `href="/terms-and-conditions"`.
- Не оставлять `href="#"`.

5. CSS для модалок:
- `z-index: 9999`.
- Фон оверлея + блюр:
- `background: rgba(1, 6, 30, 0.6)`
- `backdrop-filter: blur(24px)`
- Показ модалок только через `:has(#modal-...:checked)`.
- Блокировка скролла `body` через `body:has(#modal-...:checked) { overflow: hidden; }`.

6. Ассеты (если требует CMS):
- Использовать absolute URL вида `https://nightwin.com/content/uploads/...`.
- Обычно это минимум:
- `info.png`
- `chevron_up.png`
- баннеры/уголки (если клиент просит хостить в CMS).

7. Мелочи по верстке:
- Все стили обязательно скоупить через `#html-content-type-area`.
- Всю разметку LP оборачивать в `<div id="html-content-type-area">...</div>`.
- Не оставлять глобальные селекторы без `#html-content-type-area`.
- Если CMS может вырезать `<link>` на шрифт, продублировать его через `@import` внутри `<style>`.
- Для скрытых инпутов и CMS-оберток проверять `clip` / `display: contents` / `!important`, если сетка или фокус ломаются.
- `body { overflow-x: hidden; }`.
- Для контейнера:
- `width: 100%`
- `position: relative`
- Для скрытых инпутов:
- `left: 0; top: 0;`

### Мини-шаблон структуры модалки (CMS)
```html
<input class="nw-hidden-input" type="checkbox" id="modal-info-toggle">

<label class="wb-info-trigger" for="modal-info-toggle" aria-label="Open bonus details">
  <img src="https://nightwin.com/content/uploads/info.png" alt="Info">
</label>

<section class="wb-modal" id="bonus-details" aria-label="Bonus Details">
  <label class="wb-modal-backdrop" for="modal-info-toggle" aria-label="Close popup"></label>
  <div class="wb-modal-dialog">
    <div class="wb-modal-head">
      <h3>Bonus Details</h3>
      <label class="wb-modal-close" for="modal-info-toggle" aria-label="Close popup">×</label>
    </div>
    <div class="wb-modal-content">
      ...
    </div>
  </div>
</section>
```

### Проверка перед релизом
1. Все модалки открываются и закрываются по иконке, крестику и фону.
2. При открытой модалке фон не скроллится.
3. Нет `href="#"`.
4. Ссылки ведут на `/deposit` и `/terms-and-conditions`.
5. Контент офферов/FAQ не изменен случайно при CMS-правках.
