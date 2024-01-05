<script>
    // если хоть один чекбокс отмечен, то true для активации кнопки удаления отмеченных элементов
    import { delete_button_enabled } from '../item/stores'
    import { onDestroy } from 'svelte';
    import { onMount } from 'svelte';
    export let href = '/item/';
    export let checkbox_name = 'id';    // значение атрибута name в чекбоксе
    export let url = '';                // #1для fetch
    export let fields = ["id",];     // #1список полей
    export let parseData = undefined;
    const selected_class = 'has-background-warning-light';
    const all_checkboxes_selector = "tablegen-all";
    const quantity = 10;                // количество загружаемых с конца строк в таблицу
    let start_position = 0;             // начальная позиция, чтобы догружать потом
    
    onDestroy(()=>delete_button_enabled.update((n)=>false));
    /**
     * Установка галочки добавляет в коллекцию id элемента,
     * снятие - удаляет его из коллекции.
     */
    const yes = new Set();

    function checkHandler (event) {
        let _tag = event.currentTarget;
        check(_tag);
    }

    const check = (_tag) => {
        const chkbox = document.querySelector('tr th.firstcell label input');
        if (_tag.checked) {
            yes.add(_tag.value);
            delete_button_enabled.update((n)=>true);
            _tag.parentNode.parentNode.parentNode.classList.add(selected_class);
            // @ts-ignore
            document.querySelector("#tgen_change_inform").innerHTML = '' + yes.size;
            chkbox.classList.add('haschecked');
        }
        else {
            // @ts-ignore
            document.querySelector(`#${all_checkboxes_selector}`).checked = false;
            yes.delete(_tag.value);
            document.querySelector("#tgen_change_inform").innerHTML = '' + yes.size;
            if (yes.size == 0) {
                delete_button_enabled.update((n)=>false);
                document.querySelector("#tgen_change_inform").innerHTML = '';
                chkbox.classList.remove('haschecked');
            }
            _tag.parentNode.parentNode.parentNode.classList.remove(selected_class);
        }
    }
    /**
     * Выбрать всё.
     */
    const checkAll = (event) => {
        let _tag = event.currentTarget;
        let checkboxes = document.getElementsByName(checkbox_name);
        if (_tag.checked) { // перебор чекбоксов и устаовка
            for (let checkbox of checkboxes) {
                // @ts-ignore
                checkbox.checked = true;
                check(checkbox);
            }
        }
        else {
            for (let checkbox of checkboxes) {
                // @ts-ignore
                checkbox.checked = false;
                check(checkbox);
            }
        }
    }

    const translate = (t) => {
        const voc = {
            name: "Наименование",
            address: "Адрес",
            inn: "ИНН",
            kpp: "КПП",
            ogrn: "ОГРН",
            bank: "Банк",
            bik: "БИК",
            pay_account: "Номер счёта",
            kor_account: "Корр. счёт",
            phone: "Телефон",
            number: "Номер",
            date: "Дата",
            city: "Город",
            truck: "Авто",
            spec_notes: "Особые отметки",
            destination_address: "Адрес назначения",
            text: "Условия договора",
            sender: "Отправитель",
            receiver: "Получатель",
            payer: "Плательщик",
            created: " ",
            modified: " ",  // если оставить "", то выведет "modified"
            document: "Документ",
            doc_sum: "Сумма"
        };
        return voc[t] ? voc[t] : t;
    }

    // оставим нужные поля
    function _parse(data) {
        if (parseData) return parseData(data);
        const pre = JSON.parse(JSON.stringify(data, fields));
        return pre;
    }
    /**
     * Класс для объекта сортировки. Хранит состояние сортировки (0 | 1 | 2),
     * привязанное к тегу. 
     * Если state=0, то &o=-name,
     * если state=1, то &o=name,
     * если state=2, то ''  (без сортировки)
     */
    class Order {
        constructor (minus1=true, item1='', tag1=undefined, minus2=true, item2='', tag2=undefined,) {
            this.item1 = item1      // имя поля (name), по которому сортируем (a[name=<name>])
            this.minus1 = minus1    // направление сортировки (с "-" или без) (a[state=<0|1|2>])
            this.tag1 = tag1        // нужна ссылка на тег, чтобы можно было отменить сортировку
            this.item2 = item2      // поддерживается сортировка и по второму полю, не глубже.
            this.minus2 = minus2
            this.tag2 = tag2
        }
        add(minus, item, tag) {
            if (item !== this.item1) {
                if (this.tag2) {
                    this.tag2.setAttribute('state', 0)
                    this.tag2.querySelector('i').className = 'fa-light fa-sort'
                }
                this.item2 = this.item1
                this.minus2 = this.minus1
                this.tag2 = this.tag1
                this.item1 = item
                this.tag1 = tag
                tag.style.color = 'chocolate'
                if (this.tag2) this.tag2.style.color = 'grey'
            }
            this.minus1 = minus
            return this.str()
        }
        clear() {
            if (!this.tag1) return ''
            this.item1 = this.item2 = ''
            this.minus1 = this.minus2 = true
            if (this.tag2) {
                this.tag2.setAttribute('state', 0)
                this.tag2.querySelector('i').className = 'fa-light fa-sort'
            }
            this.tag1.setAttribute('state', 0)
            this.tag1.querySelector('i').className = 'fa-light fa-sort'
            this.tag1 = this.tag2 = undefined
            return ''
        }
        str() {
            const minus1 = this.minus1 ? '-' : '';
            const minus2 = this.minus2 ? '-' : '';
            let str1 = this.item1 ? "&o=" + minus1 + this.item1 : ''; // если есть первый элемент
            let str2 = this.item2 ? "&o=" + minus2 + this.item2 : ''; // "&o=-email&o=name"
            return str1 + str2
        }
        has(item) {
            return (this.item1 === item || this.item2 === item) ? true : false
        }
        reverse() {
            if (!this.item1) return
            this.minus1 = !this.minus1
        }
    }
    /**
     * Класс содержит данные фильтров. Это пары ключ=значение, причем каждому ключу
     * сопоставлено едиственное значение. Формирует строку для параметров в GET вида:
     * &key1=value&key2=value
     */
    class Filter {
        constructor () {
            this.data = {}
        }
        add (key, text = '') {
            this.data[key] = text
        }
        del (key) {
            this.add(key, '')
        }
        str () {    // output string
            let _str = ''
            for (let _key in this.data) {
                _str += this.data[_key] ? '&' + _key + '=' + this.data[_key] : ''
            }
            return _str
        }
    }

    let global_filter = new Filter(); // сюда запишем строки поиска при вводе в заголовках таблицы
    let global_order = new Order();
    async function fetch_data() {
        const qt = quantity ? "&q=" + quantity : ""; // количество записей для показа в таблице (defalt: 10)
        const start = start_position ? "&s=" + start_position : ""; // с какой позиции подгрузить следующую порцию (0)
        let urlsearchparams = `${qt}${start}${global_order.str()}${global_filter.str()}`; //вся строка с фильтром | ""
        urlsearchparams = urlsearchparams ? urlsearchparams.replace('&', '?') : '' ; // первый символ '&' заменим на '?'
        const _content = await fetch(`${url}${urlsearchparams}`).then((x) => x.json());
        start_position = (_content.length < quantity) 
            ? -1                            // -1 -- значит больше не загружать (блоьше нет информации)
            : start_position + quantity;
        return _parse(_content);
    }

    class T_cell {
        constructor(content='', tagname='td', attrs={}) {
            this.tagname = tagname
            this.element = createTag(tagname, attrs)
            // @ts-ignore
            if (content) this.element.innerHTML = content;
        }
        append (child) {
            this.element.appendChild(child)
        }
    }
    
    class T_row {
        constructor (attrs={}) {
            this.element = createTag('tr', attrs)
        }
        append (child) {
            this.element.appendChild(child)
        }
        appendCells = (obj) => {        // здесь объект - это одна запись из БД с множеством полей со значениями
            const link = "<a href='" + href + obj['id'] + "'>" + obj[Object.keys(obj)[1]] + "</a>";
            for (const _key in obj) {
                if (_key === 'id') {    // первая ячейка - checkbox;
                    const _input = createTag('input', {
                        type: 'checkbox', 
                        name: checkbox_name,
                        value: obj[_key],
                    })
                    _input.addEventListener('change', checkHandler)
                    const _label = createTag('label')
                    _label.appendChild(_input)
                    const cell = new T_cell("", "td", {class: 'firstcell'})
                    cell.append(_label)
                    this.append(cell.element)
                } else {
                    const cell = new T_cell(obj[_key])
                    this.append(cell.element)
                }
            }
            this.element.children[1].innerHTML = link;
        }
        appendThCells = (obj) => {      // создание ячеек <th>
            for (const _key in obj) {
                const divfield = createTag('div', {class: 'field has-addons'}) // `<div class='field has-addons'>
                let divcontrol = createTag('div', {class: 'control is-expanded'})//    <div class="control is-expanded">
                const input = createTag('input', {                             //      <input class="inpu...
                    class: 'input is-small',
                    type: 'search',
                    placeholder: translate(_key),
                    name: _key
                })
                input.addEventListener('change', filterhandler);
                divcontrol.appendChild(input)
                divfield.appendChild(divcontrol)                               //    </div>
                divcontrol = createTag('a', {class: 'control', name: _key, state: 0}) //    <div class="control">
                divfield.appendChild(divcontrol)
                const span = createTag('span', {class: 'icon'})
                divcontrol.appendChild(span)                                   //      <span class="icon">
                span.appendChild(createTag('i', {class: 'fa-light fa-sort'}))  //        <i class="fa-ligh...
                const cell = new T_cell('', 'th')
                cell.append(divfield)
                this.append(cell.element)
                divcontrol.addEventListener('click', sorthandler);
                // cell.element.innerHTML =                 // еще раз наглядно структура
                // `<div class='field has-addons'>
                //     <div class="control">
                //         <input class="input is-small" type="text" placeholder="${translate(_key)}">
                //     </div>
                //     <div class="control">
                //         <span class="icon">
                //             <i class="fa-light fa-sort"></i>
                //         </span>
                //     </div>
                // </div>`
            }
        }
    }

    function createTag (tag, attrs={}) {
        let elem = document.createElement(tag);
        for (let key in attrs) {
            elem.setAttribute(key, attrs[key])
        }
        return elem
    }

    function sorthandler (e) {
        const i_tag = e.currentTarget;
        const state = i_tag.getAttribute('state');
        const name = i_tag.getAttribute('name');
        const icon_tag = i_tag.querySelector('i');
        const sort_down_class = "fa-light fa-sort-down";
        const sort_up_class = "fa-light fa-sort-up";
        const sort_class = 'fa-light fa-sort';
        i_tag.preventDefault;
        if (state == 0) {
            global_order.add(false, name, i_tag);
            i_tag.setAttribute('state', 1);
            icon_tag.className = sort_down_class;
        } else if (state == 1) {
            global_order.add(true, name, i_tag);
            i_tag.setAttribute('state', 2);
            icon_tag.className = sort_up_class;
        } else { 
            global_order.clear();
            i_tag.setAttribute('state', 0);
            icon_tag.className = sort_class;
        }
        filter_table_content();
    }

    function filterhandler (e) {
        const input = e.currentTarget;
        global_filter.add(input.name, input.value);
        filter_table_content();
    }

    const observer = new IntersectionObserver(append_content, {
            root: document.querySelector('#tablegen_id'),
            threshold: 0.5, });
    let is_observe = false; // метка, чтобы знать, когда остановлено наблюдение
    const tbody = createTag('tbody'); // тело таблицы для дальнейшего изменения содержимого
    
    async function create_table_content() {
        const tab = document.querySelector('#T_gen');       // таблица
        const _thead = createTag('thead');                  // шапка таблицы
        const thead_tr = new T_row();                       // строка шапки
        const table_content = await fetch_data(); // получим данные
        // --- создадим первую ячейку шапки, в ней будет чекбокс выбора всех элементов ---
        const _input = createTag('input', {type: 'checkbox', id: all_checkboxes_selector});
        _input.addEventListener('change', checkAll);
        const _label = createTag('label');
        _label.appendChild(_input);
        const th_first_cell = new T_cell('', 'th', {class: 'firstcell'});
        th_first_cell.append(_label);
        thead_tr.append(th_first_cell.element);
        // --- // 
        const _headers = {...table_content[0]};
        delete _headers.id;
        thead_tr.appendThCells(_headers); // строку шапки заполним ячейками (кроме первой)
        _thead.appendChild(thead_tr.element);
        tab.appendChild(_thead);
        tab.appendChild(tbody);
        // <--- заполняем tbody
        grow_tbody(table_content);
        // --->
        setTimeout(()=>observer.observe(document.querySelector('#observer_target')), 200); // назначить отслеживание не сразу
        is_observe = true;
        document.querySelector('#tgen_change_inform').innerHTML = '';
    }

    /**
     * Для вызова из колбэка на событие фильтрации и/или сортировки содержимого таблицы.
     * Заменяет содержимое tbody данными в соответствии с фильтром.
     * @param filter    фильтр формата: "&f_name=value-name&f_inn=value-inn"
     */
    async function filter_table_content () {
        const tab = document.querySelector('#T_gen');    // таблица
        const tbody = tab.querySelector('tbody');        // тбади
        tbody.innerHTML = '';                            // очистка
        start_position = 0;
        document.querySelector('#observer_target').innerHTML = "...";
        const table_content = await fetch_data(); // получим данные
        grow_tbody(table_content);
        if (is_observe) return
        setTimeout(()=>observer.observe(document.querySelector('#observer_target')), 200); // назначить отслеживание не сразу
        is_observe = true;     
    }

    /**
     * Добавляет строки в конец таблицы, элемента tbody
     * @param rows_data -- массив объектов
     */
    function grow_tbody (rows_data) {
        for (const row of rows_data) {
            const trow = new T_row();
            trow.appendCells(row);
            tbody.appendChild(trow.element);
        }
    }
    /**
     * Коллбэк для обзервера - функция, вызываемая системой, когда целевой блок
     * в конце таблицы становится виден. Загружает очередную порцию данных в конец таблицы.
     * @param entries
     * @param observer
     */
    function append_content (entries, observer) {
        // получаем следующую порцию данных для строк
        const _target = document.querySelector('#observer_target');
        if (start_position === -1) {
            observer.unobserve(_target);
            is_observe = false;
            _target.innerHTML = 'Конец списка';
            return
        }
        entries.forEach(async entry => {
            if (entry.isIntersecting) {
                const _content = await fetch_data(); // получим данные
                grow_tbody(_content);
            }
        });
        observer.unobserve(_target);
        setTimeout(()=>observer.observe(_target), 200); // назначить отслеживание не сразу
    }

    onMount(create_table_content);
</script>

<table class="table is-hoverable is-striped has-background-white-ter" id="T_gen"> </table>
<div id="observer_target" style="width: 15rem; height: 20px; border: none; margin: 0 0 0 1rem; font-size: 0.8rem">...</div>

<style>
    table {
    min-width: 29rem;
    }
    :global(#T_gen thead) {
    position: sticky;
    top: 0;
    font-size: .9rem;
    background-color: white;
    outline: 1px solid lightgrey;
    }
    :global(#tablegen_id) {
        overflow: auto;
        border: 1px solid #909090;
        width: fit-content;
        max-width: 100%;
        height: calc(100% - 3rem);
    }
    :global(#T_gen td a) {
        color: chocolate;
        display: block;
    }
    :global(table tbody td.firstcell, table thead tr th.firstcell) {
        padding: 0;
    }
    :global(table tbody td.firstcell label, table thead tr th.firstcell label) {
        display: block;
        width: 2.5rem;
        height: 2.5rem;
        cursor: pointer;
        text-align: center;
    }
    :global(table tbody td.firstcell label input, table thead tr th.firstcell label input) {
        margin: .9rem;
    }
    :global(table thead tr th input[class='input is-small'], table thead tr th input[class='input is-small']::placeholder) {
        color: #000000aa;
        font-size: .9rem;
        border-color: transparent;
    }
    :global(table thead tr th.firstcell label input.haschecked) {
        filter: drop-shadow(0px 0px 3px #0061ff);
    }
    :global(i.fa-light.fa-sort) {
        color: grey;
    }
    :global(#tgen_change_inform) {
        position: absolute;
        top: .2rem;
        left: 1.8rem;
        font-size: .6rem;
        font-weight: 800;
        color: #b41414;
    }
</style>
