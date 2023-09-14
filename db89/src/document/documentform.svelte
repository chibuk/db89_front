
<div class="box">
    <h5 class="title is-5 has-text-centered">Товарно-транспортная накладная</h5>
    <form>
        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Номер и дата</label>
            </div>
            <div class="field-body">
                <div class="field is-grouped is-justify-content-space-between">
                    <p class="control">
                        <input type="text" class="input is-small" placeholder="номер"
                            name="number" maxlength="15" required>
                    </p>
                    <p class="control">
                        <input type="text" class="input is-small has-text-right" value="23.08.2023" readonly>
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Город</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <input type="text" class="input is-small" placeholder="город" 
                            name="city" maxlength="100">
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Номер ТС</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <input type="text" class="input is-small" placeholder="№ ТС"
                            name="truck" maxlength="20">
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Отправитель</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div class="control">
                        <div class="select is-small is-fullwidth">
                            <select name="sender">
                                <option>ИП Горелов И.В.</option>
                                <option>ООО "ох-хо-хо-хо-хо"</option>
                            </select>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Получатель</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div class="control">
                        <div class="select is-small is-fullwidth">
                            <select name="receiver">
                                <option>ИП Горелов И.В.</option>
                                <option>ООО "ох-хо-хо-хо-хо"</option>
                            </select>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Плательщик</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <div class="control">
                        <div class="select is-small is-fullwidth">
                            <select name="payer">
                                <option>ИП Горелов И.В.</option>
                                <option>ООО "ох-хо-хо-хо-хо"</option>
                            </select>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-touch">
                <label for="" class="label">Перевозимый груз</label>
            </div>
            <div class="field-body"> -->
                <div class="field">
                    <div class="control">
                        <slot />
                    </div>
                </div>
            <!-- </div>
        </div> -->

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Особые отметки</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <textarea class="textarea is-small" name="spec_notes" placeholder="особые отметки" rows="2"></textarea>
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Адрес доставки</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <input type="text" class="input is-small" placeholder="адрес доставки"
                            name="destination_address" maxlength="256">
                    </p>
                </div>
            </div>
        </div>

        <div class="field is-horizontal">
            <div class="field-label is-small is-hidden-mobile">
                <label for="" class="label">Условия договора</label>
            </div>
            <div class="field-body">
                <div class="field">
                    <p class="control">
                        <textarea class="textarea is-small" name="text" id="" placeholder="условия договора" rows="3"></textarea>
                    </p>
                </div>
            </div>
        </div>
        
        <div class="field is-horizontal">
            <div class="field-label is-small">
                <label for="" class="label"></label>
            </div>
            <div class="field-body">
                <div class="field is-grouped">
                    <p class="control is-expanded">
                        <button class="button is-small is-link is-outlined is-fullwidth" type="submit">Сохранить</button>
                    </p>
                    <p class="control">
                        <button class="button is-small is-outlined" type="reset">Отмена</button>
                    </p>
                </div>
            </div>
        </div>
    </form>
</div>
