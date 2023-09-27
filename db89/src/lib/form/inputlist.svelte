<script>
// @ts-nocheck

    import Typeahead from "svelte-typeahead";    // поле ввода с поиском и выпадающим списком
    export let placeholder = 'наименование';
    export let data = [];       // Массив с объектами для поиска с примером
    //     {
    //         "name": "ИП Крутой Хорёк",
    //         "inn": "1234568892",
    //     },
    //     {
    //         "name": "ООО \"Копыта лошади\"",
    //         "inn": "6667543566",
    //         "id": "someid"
    //     },
    //     {
    //         "name": "ООО \"Общество с ответственностью\"",
    //         "inn": "4709565458",
    //     }
    // ]

    /**
     * Из каждого объекта в составе массива data составляем строку для поиска и
     * именования элемента выпадающего списка: перебираем свойства в цикле, и
     * суммируем значения через дефис с пробелами. Т.о. набор свойств может быть
     * любой (id, name || name, inn, ...etc) и любое количество, их не придется изменять.
     * @param item - это объект из массива data
     */
    const extract = (item) => {
        let _strng = '';        // строка
        let _mdash = ' - ';     // разделитель
        for (let key in item) {
            if (key == 'id') continue;  // пропускаем идентификатор для передачи на сервер: id
            _strng += item[key] + _mdash;
        }
        return _strng.slice(0,-3); // срезаем последний разделитель
    }

</script>

<div class='sv-inputlist'>
    <Typeahead hideLabel={true} {data} {extract}
        on:select
        on:clear
        {placeholder}>
        <div slot="no-results">
            Не найдено. <a href="">Создать новую запись?</a>
        </div>
    </Typeahead>
</div>

<style>
    :global(.sv-inputlist ul) {
        margin: 4px 0;
        background-color: rgba(255,255,255,.6);
    }

    :global(.sv-inputlist ul li) {
        font-size: .8rem;
    }

    :global(.sv-inputlist [slot='no-results']) {
        position: fixed;
        z-index: 10;
        border: 1px solid rgba(224,224,224,.9);
        border-radius: 3px;
        width: 20rem;
        background-color: white;
        font-size: .8rem;
        padding: .2rem .4rem;
    }

    :global([data-svelte-typeahead], [data-svelte-search]) {
        width: 100%;
    }

    :global(ul.svelte-typeahead-list mark) {
        background-color: rgba(170,100,100,.3)
    }
</style>
