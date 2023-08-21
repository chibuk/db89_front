<script>
    // если хоть один чекбокс отмечен, то true для активации кнопки удаления отмеченных элементов
    import { delete_button_enabled } from '../item/stores'
    
    export let table_content = [];
    export let href = '/item/';
    export let checkbox_name = 'id';    // значение атрибута name в чекбоксе
    const selected_class = 'has-background-warning-light';
    const all_checkboxes_selector = "tablegen-all"
    
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
        if (_tag.checked) {
            yes.add(_tag.value);
            delete_button_enabled.update((n)=>true);
            _tag.parentNode.parentNode.className = selected_class;
        }
        else {
            // @ts-ignore
            document.querySelector(`#${all_checkboxes_selector}`).checked = false;
            yes.delete(_tag.value);
            if (yes.size == 0) delete_button_enabled.update((n)=>false);
            _tag.parentNode.parentNode.className = '';
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
            pay_acount: "Номер счёта",
            kor_acount: "Корр. счёт",
            phone: "Телефон",
            number: "Номер",
            data: "Дата",
            city: "Город",
            truck: "Номер машины",
            spec_notes: "Особые отметки",
            destination_address: "Адрес назначения",
            text: "Условия договора",
            sender: "Отправитель",
            receiver: "Получатель",
            payer: "Плательщик",
        };
        t = voc[t] ? voc[t] : t;
        return t;
    }
</script>
<!-- Построить таблицу по данныи JSON |  class="table is-hoverable is-striped" -->
<div>
<table class="table is-hoverable is-striped has-background-white-ter">
    <thead>
        <tr>
            <th>
                <input type="checkbox" id={all_checkboxes_selector} on:change={checkAll}>
            </th>
            {#each Object.keys(table_content[0]).slice(1) as cell}
                <th  class='has-text-grey'>{translate(cell)}</th>
            {/each}
        </tr>
    </thead>
    <tbody>
        {#each table_content as row_content}{@const cells = Object.values(row_content)}
            <tr>
                <td>
                    <input type="checkbox" name="{checkbox_name}" value="{cells[0]}" on:change={checkHandler}>
                </td>
                <td>
                    <a href="{href}{cells[0]}">{cells[1]}</a>    
                </td>
                {#each cells.slice(2) as cell_content}
                    <td>
                        {cell_content}
                    </td>
                {/each}
            </tr>
        {/each}
    </tbody>
</table>
</div>

<style>
    th {
    position: sticky;
    top: 0;
    font-size: 1.1rem;
    background-color: white;
    }
    div {
        height: 25rem;
        overflow: auto;
        border: 1px solid #909090;
    }
    a {
        color: chocolate;
    }

</style>
