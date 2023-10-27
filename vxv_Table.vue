<template>
modifiedCells {{ modifiedCells }}
<br><br>rows {{ rows }}
<br><br>selectAllFil {{ selectAllFil }}
<!-- headers {{ headers }} -->
<br><br>columns {{ columns }}
<br><br>visibleColumns {{ visibleColumns }}
<br><br>defaultColumns {{ defaultColumns }}
<br><br>columnItem {{ columnItem }}

<div class="contener">

  <table>
    <!-- class="tryaska" -->
    <!-- <caption>
      <div class="box">
          Название
      </div>
    </caption> -->
    <col :style="`display: none`"/>
    <!-- todo Заголовок таблицы Кнопки-->
      <thead>
        <tr >
          <th class="panel" :colspan="headers.length">
            <div class="conthead">

              <q-btn flat round color="white" size="sm" icon="mdi-sigma"/>

              <div class="inline-block" style="border-right: 2px solid white; border-left: 2px solid white;">
                <!-- Редактируем таблицу -->
                <q-btn flat round color="white" size="sm" icon="mdi-pencil"
                  @click="showEditIcon = !showEditIcon, !showEditIcon ? saveEditData() : null"
                  :class="showEditIcon ? `text-amber` : null"

                >
                <q-tooltip anchor="top middle" self="bottom middle">Редактировать таблицу</q-tooltip>
              </q-btn>

            <q-btn @click="undo" flat round color="white" size="sm" icon="undo" />
            <q-btn @click="redo" flat round color="white" size="sm" icon="redo" />

              <!-- Отменить ВСЕ изменения -->
              <q-btn flat round color="white" size="sm" icon="mdi-arrow-u-left-top"
                @click="cancelEdits"
                :class="modifiedCells.every(e => Object.keys(e).length === 0) ? null : `text-amber` "
                />
              </div>

              <!-- Активация фиксации колонок -->
              <q-btn flat round color="white" size="sm" icon="mdi-lock-outline"
                @click.stop="lockOutline = !lockOutline"
                :class="lockOutline ? `text-amber` : null "
              />

              <!-- Активация фильтрации колонок -->
              <q-btn flat round color="white" size="sm" icon="mdi-filter-variant"
                @click="filterVariant = !filterVariant"
                :class="filterVariant ? `text-amber` : null "
              />

              <q-space />

              <vxvMenu
                :iconBtn="`mdi-view-column`"
                :menuTitle="`Колонки`"
                :columnNames="columnKeys"
                :columnLabels="columnLabels"
                v-model:menuShow="menuShow"
                v-model:visibleColumns="visibleColumns"
              />

              <!-- Полноэкранный режим -->
              <q-btn
                flat round dense
                :icon="'fullscreen'"
                @click="toggleFullscreen"
                size="md"
                color="white"
              />
            </div>
          </th>
        </tr>

        <!-- todo Заголовок таблицы Строка-->
        <tr>
          <th
            v-for="col in headers" :key="col.idCol"
              :class="lockOutline || filterVariant ? `tryaska` : null"
              :style="visibleColumns.includes(col.visibleCol) ? null : 'display: none; '"

          >
          <div class="conthead ">
            <!-- Текст в ячейке -->
            <p style="display: inline-block">{{ col.label }}</p>

            <!-- фиксации колонок -->
            <q-btn flat round color="white" size="sm" icon="mdi-lock-outline"
              v-show="lockOutline"
              @click=""
            />

            <!-- фильтрации колонок -->
            <menuFilter
              v-show="filterVariant"
              v-model:menuShow="menuShow"
              v-model:columnItem="columnItem"
              :columnTitle="col.label"
              :columnKey="col.name"
              :showEditIcon="showEditIcon"
              :columnValue="[... new Set(columns[col.name])]"
              :outsideSelect="[... new Set(selectAllFil[col.name])]"


              />

              <!--
              :outsideSelect="[... new Set(selectAllFil[col.name])]"

                :columnValue="columns[col.name]"
              :columnValue="[... new Set(columns[col.name])]"

              v-model:menuShow="menuShow"

                    v-model:columnItem="columnItem"
                    :colorBtn="Object.keys(filterColumns).includes(column.name) && column.name != [] ? `text-amber` : null"

                    :RowsInColsItem="[... new Set(columns[column.name])]"
                    :outsideSelect="[... new Set(selectAllFil[column.name])]

                  -->
          </div>
          </th>

        </tr>
      </thead>

      <!-- todo Строки -->
      <tbody >
        <tr v-for="row  in rows" :key="row.idRow"
        >
          <td v-for="col in headers" :key="col.idCol"
            v-text="row[col.name]"
            @input="(val) => tdInput(val, col.name, row.idRow, row[col.name])"
            @contextmenu="console.log('contextmenu => X =', $event.clientX, 'Y =', $event.clientY)"
            :contenteditable="showEditIcon"
            :style="[
              visibleColumns.includes(col.visibleCol) ? null : 'display: none; ',
              modifiedCells[row.idRow][col.name] ? `color: #0091EA; ` : null,
            ]"

          >
          </td>
        </tr>
      </tbody>

        <!--

        -->

      <tfoot>
        <tr>
          <td :colspan="headers.length">

          </td>
        </tr>
      </tfoot>

  </table>
  <blackout :menuShow="menuShow"/>
  </div>




</template>


<script setup>
  import { Input } from "postcss";
  import loadRowsHeaders from "../store/vxv_Table.json"
  import menuFilter from "./vxv_column_filter.vue"
  import vxvMenu from "./vxv_menu.vue"
  import blackout from "./blackout.vue"



  // import { useMousePosition } from "./useMousePosition.js"
  import { ref, reactive, computed, toRefs, watch, onMounted, watchEffect, isRef,
    isReactive,
    watchPostEffect,
    onBeforeMount,
    readonly,
    toRaw,
    markRaw,
    onUpdated,

  } from 'vue'
  console.log("---------------------------------------------------------")

  onUpdated(() => {console.log('onUpdated_Table')})

  const {loadRows, loadHeaders} = loadRowsHeaders

  loadHeaders.forEach((e, i) => {
    e["field"] = e.name
    e['visibleCol'] = e.name
    e['sortable'] = true
    e['label'].includes('%') ? e['sort'] = (a, b) => parseInt(a, 10) - parseInt(b, 10) : null
  })

  const visibleColumns = ref(loadHeaders.map(e => e['visibleCol']))

  loadHeaders.forEach((e, i) => e['idCol'] = i)
  loadRows.forEach((e, i) => e['idRow'] = i)

  const rows = ref(loadRows)
  const headers = ref(loadHeaders)

  const TrRef = ref([])
  onMounted(() => {
    console.log('- onMounted -')
  //   document.addEventListener('mouseup', event => {
  //     // console.log(event.target.textContent)
  //   console.log(window.getSelection().toString())
  // })

  })

  // ------------------------------------------------------------------------------------------------
  // Редактирование таблицы

  const showEditIcon = ref(false)
  const modifiedCells = ref([])

  modifiedCells.value = rows.value.map(() => new Object())

  function tdInput(val, name, index, old) {
    let value = val.target.textContent
    !isNaN(value) ? value = Number(value.replace(',', '.')) : null
    value !== old ? modifiedCells.value[index][name] = value : delete modifiedCells.value[index][name]
  }

  function saveEditData() {
    console.log("saveEditData")
    modifiedCells.value.map((e, i) => {
      if(Object.keys(e).length !== 0) {
        let idRowSearch = rows.value.findIndex(e => e.idRow === i)
        Object.entries(e).forEach(([key, val]) => {
          // columns.value[key][i] = val
          // selectAllFil.value[key][i] = val
          // rows.value[i][key] = val
          rows.value[idRowSearch][key] = val
          defaultColumns.value[key][i] = val

        })
      }
      modifiedCells.value[i] = new Object()
    })
    columns.value = perevorot(rows.value)
    // selectAllFil.value = perevorot(rows.value)

//  ==================  Не правильно сохраняет списки с выборками в колонках


  }

  const undo = () => document.execCommand('undo')
  const redo = () => document.execCommand('redo')

  function cancelEdits() {
    showEditIcon.value = false
    modifiedCells.value.map(e => undo())
    modifiedCells.value = rows.value.map(() => new Object())
  }

  function toggleFullscreen() {
    let elem = document.querySelector('table')
    if (!document.fullscreenElement) {
      elem.requestFullscreen().catch((err) => {
        alert(
          `Ошибка при попытке включить полноэкранный режим: ${err.message} (${err.name})`,
        );
      });
    } else {
      document.exitFullscreen();
    }
  }

  // ------------------------------------------------------------------------------------------------
  //. Фильтр по колонкам

  const menuShow = ref(false)
  const filterVariant = ref(true)

  const columns = ref({})
  const selectAllFil = ref({})
  const columnItem = ref({})
  const filterColumns = ref({})
  const idxListfilter = ref([])

  const columnKeys = reactive(headers.value.map(e => e.name))
  const columnLabels = reactive(headers.value.map(e => e.label))

  const perevorot = (val) => {
    // Переопределение массива из строк в столбцы
    const obj = {}
    columnKeys.map(col => obj[col] = val.map(e => e[col]))
    return obj
  }

  const defaultColumns = computed(() => perevorot(loadRows))
  // const columns = computed(() => perevorot(rows.value))



  Object.entries(defaultColumns.value).map(([key, vals]) => {
      columns.value[key] = vals.slice()
      // colorEditCells.value[key] = vals.slice()
      selectAllFil.value[key] = vals.slice()
    })

  watch(() => columnItem.value, (item) => {
    // Определяем ключ и значение отправленное из диалога фильтра
    let itemKey = Object.keys(item)[0].slice()
    let itemVals = Object.values(item)[0].slice()

    //  Собираем объект из выбранных строк в фильтрах по столбцам
    filterColumns.value[itemKey] = itemVals
    // Значение автовыбора всех строк в текущем фильтре

    if(Object.values(filterColumns.value).length >= 1) {
      let key0 = Object.keys(filterColumns.value)[0]
      const Rows0 = defaultColumns.value[key0].slice()
      idxListfilter.value = []

      // Находим индескы из исходного списка по полученным значениям выбора в фильтрах
      defaultColumns.value[itemKey].forEach((e, i) => {if (itemVals.includes(e)) {idxListfilter.value.push(i)}})

      // Из полного списка строк в таблице фильтруем те, которые есть в списке выбранных индексов
      rows.value = loadRows.filter((e, i) => idxListfilter.value.includes(i) )

      console.log("rows.value = ", rows.value)

      //  Корректируем фильтра в столбцах, согласно полученным индексам
      Object.entries(defaultColumns.value).forEach(([key, vals]) => {
        // colorEditCells.value[key] = vals.filter((e, i) => idxListfilter.value.includes(i))

        if (key !== itemKey) {
          let arr = vals.filter((e, i) => idxListfilter.value.includes(i))
          // columns.value[key] = arr.slice()
          columns.value[key] = [... new Set(arr)]
          selectAllFil.value[key] = [... new Set(arr)]
          // Удаляем все фильтра кроме 1го
          if (key !== key0) {delete filterColumns.value[key]}
          } else {selectAllFil.value[key] = itemVals}
        })
      // Для первого фильтра всегда полный список строк
      columns.value[key0] = [...Rows0]
    }
    // Убираем читсим список для снятия цвета кнопки фильтра
    if(itemVals.length ===  [... new Set(columns.value[itemKey])].length)
      {delete filterColumns.value[itemKey]}
      idxListfilter.value = []
    })

// ------------------------------------------------------------------------------------------------
//. Фиксация колонок

const lockOutline = ref(false)


// ------------------------------------------------------------------------------------------------

</script>


<style lang="sass">
$color: #90A4AE
$column: 2
$max-width: 600px
$border: 1px solid #cccccc77
$outline: 0.5px solid white

// table caption
//   background-color: $blue-grey
//   padding: 5px 5px
//   outline: $outline
.contener
  margin-right: 20px
  // padding-right: 50px
table
  margin: 10px
  border-collapse: collapse
  vertical-align: middle
  text-align: center
  width: 100%
  p
    height: 100&
    vertical-align: middle
    margin: auto

  thead
    height: 35px
    th
      user-select: none
      padding: 5px
      z-index: 3
      position: sticky
      top: 0
      // left: 0
      outline: $outline
      box-shadow: 0px 5px 10px #78909C
      background-color: $color
    .panel
      text-align: left
      background-color: $blue-grey


  tbody
    td
      height: inherit
      z-index: 1
      outline: $border
      background-color: white
      padding: 3px 5px

      transition-property: display opacity
      transition-duration: 0.5s 0.5s
      // :style="showEditIcon ? 'display: none;' : null"
    td:nth-child(2)
      text-align: left

  tfoot
    tr
      z-index: 100
      margin-top: 50px
      td
        text-align: left
        height: 35px
        padding: 5px 10px
        outline: $outline
        background-color: $color
        box-shadow: 0px -5px 10px #78909C

  .conthead
    display: flex

  .ext-view-off
    display: none

// .tryaska:hover
.tryaska
  animation: example-fab-animate 0.8s cubic-bezier(.36,.07,.19,.97) both
  transform: translate3d(0, 0, 0)
  backface-visibility: hidden
  perspective: 1000px
@keyframes example-fab-animate
  10%, 90%
    transform: translate3d(-1px, 0, 0)
  20%, 80%
    transform: translate3d(2px, 0, 0)
  30%, 50%, 70%
    transform: translate3d(-4px, 0, 0)
  40%, 60%
    transform: translate3d(4px, 0, 0)

</style>
