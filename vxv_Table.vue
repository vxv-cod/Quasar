<template>
<br><br>rows {{ rows }}
<br><br>loadRows {{ loadRows }}
<!-- <br><br>selectAllFil {{ selectAllFil }} -->
<!-- headers {{ headers }} -->
<!-- <br><br>columns {{ columns }} -->
<!-- <br><br>visibleColumns {{ visibleColumns }} -->
<!-- <br><br>filterColumns {{ filterColumns }}
<br><br>defaultColumns {{ defaultColumns }}
<br><br>columnsIndexRow {{ columnsIndexRow }}
<br><br>columnItem {{ columnItem }} -->
<!-- <br><br>selected {{ selected }}
<br><br>headers {{ headers }}
<br><br>fistTdWidth {{ fistTdWidth }}
<br><br>pagination {{ pagination }}
<br><br>pagesNumber {{ pagesNumber }} -->
<br><br>rowsPagsInd {{ rowsPagsInd }}
<br><br>search {{ search }}

<div class="contener" >

  <!-- todo Верхняя панель Кнопки -->
  <div class="panel top" >
    <div class="" style="min-width: 35px; border-right: 1px solid white; ">

      <!-- Редактируем таблицу -->
      <q-btn flat round color="white" size="sm" icon="mdi-pencil"
        @click="showEditIcon = !showEditIcon, !showEditIcon ? saveEditData() : null"
        :class="showEditIcon ? `text-amber` : null"
      >
        <q-tooltip anchor="top middle" self="bottom middle">Редактировать таблицу</q-tooltip>
      </q-btn>

      <q-btn v-show="showEditIcon" @click="undo" flat round color="white" size="sm" icon="undo" />
      <q-btn v-show="showEditIcon" @click="redo" flat round color="white" size="sm" icon="redo" />

      <!-- Отменить ВСЕ изменения -->
      <q-btn flat round color="white" size="sm" icon="mdi-arrow-u-left-top"
        v-show="showEditIcon"
        @click="cancelEdits"
        :class="modifiedCells.every(e => Object.keys(e).length === 0) ? null : `text-amber` "
      />
    </div>

    <div>
      <!-- Сумма по колонкам -->
      <q-btn flat round color="white" size="sm" icon="mdi-sigma">
        <q-tooltip anchor="top middle" self="bottom middle">Сумма по колонкам</q-tooltip>
      </q-btn>

      <!-- Активация фиксации колонок -->
      <q-btn flat round color="white" size="sm" icon="mdi-lock-outline"
        @click="lockOutline = !lockOutline"
        :class="lockOutline ? `text-amber` : null "
        >
        <q-tooltip anchor="top middle" self="bottom middle">Фиксация колонок</q-tooltip>
      </q-btn>

      <!-- Активация фильтрации колонок -->
      <q-btn flat round color="white" size="sm" icon="mdi-filter-variant"
        @click="filterVariant = !filterVariant"
        :class="filterVariant ? `text-amber` : null "
        >
        <q-tooltip anchor="top middle" self="bottom middle">Фильтры по колонкам</q-tooltip>
      </q-btn>

      <!-- Активация Сортировки -->
      <q-btn flat round color="white" size="sm" icon="mdi-sort"
        @click="sortGeneral = !sortGeneral"
        :class="sortGeneral ? `text-amber` : null "
        >
        <q-tooltip anchor="top middle" self="bottom middle">Фильтры по колонкам</q-tooltip>
      </q-btn>

    </div>

    <q-space />

    <!--
      sort-descending
      sort-ascending
     -->

    <!--
      xs (extra small),
      sm (small),
      md (medium),
      lg (large),
      xl (extra large)
     -->

    <!-- Пагинация -->
    <div class="row justify-center" style="margin: 5px;">
      <q-pagination
          v-model="pagination.page"
          :max="pagesNumber"
          :min="1"
          :max-pages="6"
          boundary-numbers
          color="white"
          active-design="outline"
          size="sm"

        />
    </div>



    <div class="" style="display: flex; ">
      <q-knob
        :min="5"
        :max="50"
        :inner-min="5"
        v-model="pagination.rowsPerPage"
        size="20px"
        :thickness="0.2"
        color="black"
        track-color="white"
        style="display: flex; margin-left: 5px;"
      />
      <input type="number"
        v-model.number="pagination.rowsPerPage"
        style="max-width: 60px; font-size: 16px; margin-left: 5px;"
      >
      <q-tooltip anchor="top middle" self="bottom middle">Строк в таблице</q-tooltip>
    </div>



    <div class="search" style="" >
      <input type="text" placeholder="Поиск по таблице" v-model="search">
      <q-icon v-if="search === ''" name="search" size="sm" />
      <q-icon v-else name="clear" class="cursor-pointer" @click="search = ''" size="sm" />
    </div>


    <!-- <q-input autogrow dark dense standout v-model="search" input-class="" class="" style="">
      <template v-slot:append>
        <q-icon v-if="search === ''" name="search" style=""/>
        <q-icon v-else name="clear" class="cursor-pointer" @click="search = ''" />
      </template>
    </q-input> -->



    <visiableColumn
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
      style="margin-right: 5px;"
    />

  </div>

  <div class="tableBox">
    <table class="">
      <!-- todo Заголовок таблицы -->
      <thead >
        <tr>
          <th ref="fistTd"
            :style="`width: 34px; position: sticky; left: 0; z-index: 5;`"
            >
            <q-checkbox class="text-white" size="xs" dense style="" color="black"
              v-model="selectedAll"
              @click="toggleAll"
              />
          </th>

          <th v-for="col in headers" :key="col.idCol"
            :ref="col.fixed ? 'fixRef' : 'notfixRef'"
            :id="col.name"
            :style="[
              visibleColumns.includes(col.visibleCol) ? null : 'display: none; ',
              collFix(col, 5),
            ]"
            >
            <div class="boxTd ">
              <!-- Текст в ячейке -->
              <p >
                {{ col.label }}
              </p>
              <!-- <br><pre>{{ col.countSorted }}</pre> -->

              <div style="display: flex;">

                <!-- фиксации колонок -->
                <q-btn flat round color="white" size="sm" icon="mdi-lock-outline"
                  v-if="lockOutline"
                  @click="col.fixed = !col.fixed"
                  :class="[
                    lockOutline ? `tryaska` : null,
                    col.fixed ? `text-amber` : null
                    ]"
                  style="margin: auto;"
                />

                <!-- фильтрации колонок -->
                <menuFilter
                  v-if="filterVariant"
                  v-model:menuShow="menuShow"
                  v-model:columnItem="columnItem"
                  :columnTitle="col.label"
                  :columnKey="col.name"
                  :showEditIcon="showEditIcon"
                  :columnValue="[... new Set(columns[col.name])]"
                  :outsideSelect="[... new Set(selectAllFil[col.name])]"
                  :colorBtn="Object.keys(filterColumns).includes(col.name) && col.name !=[] ? `text-amber` : null"
                  :class="filterVariant ? `tryaska` : null"
                  />

                <!-- Сортировка в колонках -->
                <q-btn flat round color="white" size="sm"
                  :icon="['mdi-sort', 'mdi-sort-ascending', 'mdi-sort-descending'][col.countSorted]"
                  v-if="sortGeneral"
                  @click="rows = customSort(col); !sortGeneral ? col.countSorted = 0 : null"
                  :class="[
                    sortGeneral ? `tryaska` : null,
                    col.countSorted !==0 ? `text-amber` : null
                  ]
                  "
                  style="margin: auto;"
                  >
                </q-btn>

              </div>

            </div>
          </th>
        </tr>
      </thead>

      <!-- todo Строки -->
      <tbody >
        <tr v-for="row  in rows" :key="row.idRow"
          @click="!showEditIcon ? selectedRowsChanged(row.idRow) : null"
          :style="[
            rowsPagsInd?.includes(row.idRow) ? null : `display: none`,
          ]"
        >

        <td
          :style="[
            `position: sticky; left: 0; z-index: 4; background-color: white`,
            selected.includes(row.idRow) ? 'background-color: #cfd8dc' : null,

          ]"
          >
          <q-checkbox v-model="selected" :val="row.idRow" size="xs" dense color="black" />
        </td>

          <td v-for="col in headers" :key="col.idCol"
            v-text="row[col.name]"
            @input="(val) => tdInput(val, col.name, row.idRow, row[col.name])"
            @contextmenu="console.log('contextmenu => X =', $event.clientX, 'Y =', $event.clientY)"
            :contenteditable="showEditIcon"
            :style="[
              visibleColumns.includes(col.visibleCol) ? null : 'display: none; ',
              modifiedCells[row.idRow][col.name] ? `color: #0091EA; ` : null,
              collFix(col),
              selected.includes(row.idRow) ? 'background-color: #cfd8dc' : null,
              selected.includes(row.idRow) && col.fixed === true ? 'color: black; box-shadow: none' : null,

            ]"
          >
          </td>
        </tr>
      </tbody>

      <!-- <tfoot>
        <tr>
          <td :colspan="headers.length + 1"
            :style="[
              `position: sticky; left: 0; z-index: 4;`,
            ]"
          >
            Футер
          </td>
        </tr>
      </tfoot> -->

    </table>
  </div>
      <div class="panel botton" >
        <p>Нижний Div</p>
      </div>
  <blackout :menuShow="menuShow"/>
  </div>




</template>


<script setup>
  import { Input } from "postcss";
  import menuFilter from "./vxv_Table_column_filter.vue"
  import visiableColumn from "./vxv_Table_visiable_column.vue"
  import blackout from "./blackout.vue"

  // import loadRowsHeaders from "../store/vxv_Table.json"
  // import loadRowsHeaders from "../store/contract.json"
  import loadRowsHeaders from "../store/vxv_Table copy.json"


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
  const colorTabSelectRow = ref("blue-grey-2")

  onUpdated(() => {console.log('onUpdated_Table')})

  const {loadRows, loadHeaders} = loadRowsHeaders

  loadHeaders.forEach((e, i) => {
    e["field"] = e.name
    e['visibleCol'] = e.name
    e['sortable'] = true
    e['descending'] = false
    e['countSorted'] = 0
    e['fixed'] = false
    e['label'].includes('%') ? e['sort'] = (a, b) => parseInt(a, 10) - parseInt(b, 10) : null
  })

  const visibleColumns = ref(loadHeaders.map(e => e['visibleCol']))
  visibleColumns.value.splice(visibleColumns.value.findIndex(e => e === 'idRow'), 1)


  loadHeaders.forEach((e, i) => e['idCol'] = i)
  loadRows.forEach((e, i) => e['idRow'] = i)

  const rows = ref(loadRows)
  const headers = ref(loadHeaders)

  const columnKeys = reactive(headers.value.map(e => e.name))
  const columnLabels = reactive(headers.value.map(e => e.label))
  const columnKeysReverse = [...columnKeys].reverse()


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
          rows.value[idRowSearch][key] = val
          defaultColumns.value[key][i] = val
        })
      }
      modifiedCells.value[i] = new Object()
    })

    columnKeys.map(col => {
      columns.value[col] = defaultColumns.value[col].filter((e, i) =>
        columnsIndexRow.value[col].includes(i)
      )
    })

    selectAllFil.value = perevorot(rows.value)
  }

  const undo = () => document.execCommand('undo')
  const redo = () => document.execCommand('redo')

  function cancelEdits() {
    showEditIcon.value = false
    modifiedCells.value.map(e => undo())
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
  const filterVariant = ref(false)

  const columns = ref({})
  const selectAllFil = ref({})
  const columnItem = ref({})
  const filterColumns = ref({})
  const idxListfilter = ref([])

  const perevorot = (val) => {
    // Переопределение массива из строк в столбцы
    const obj = {}
    columnKeys.map(col => obj[col] = val.map(e => e[col]))
    return obj
  }

  const defaultColumns = computed(() => perevorot(loadRows))
  // const columns = computed(() => perevorot(rows.value))

  // const columnsIndexRow = ref({})
  // let objTemp = {}
  // columnKeys.map(col => {
  //     objTemp = {}
  //     rows.value.map( (e, i) => objTemp[e['idRow']] = e[col] )
  //     columnsIndexRow.value[col] = objTemp
  //   }
  // )

  const columnsIndexRow = ref({})
  columnKeys.map(col => columnsIndexRow.value[col] = rows.value.map(e => e['idRow'] ))

  Object.entries(defaultColumns.value).map(([key, vals]) => {
      columns.value[key] = vals.slice()
      // colorEditCells.value[key] = vals.slice()
      selectAllFil.value[key] = vals.slice()
    })

  watch(() => columnItem.value, (item) => {
    selected.value = []
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

      //  Корректируем фильтра в столбцах, согласно полученным индексам
      Object.entries(defaultColumns.value).forEach(([key, vals]) => {
        // colorEditCells.value[key] = vals.filter((e, i) => idxListfilter.value.includes(i))

        if (key !== itemKey) {
          let arr = vals.filter((e, i) => idxListfilter.value.includes(i))
          // columns.value[key] = arr.slice()
          columns.value[key] = [... new Set(arr)]
          selectAllFil.value[key] = [... new Set(arr)]
          columnsIndexRow.value[key] = [...idxListfilter.value]
          // Удаляем все фильтра кроме 1го
          if (key !== key0) {delete filterColumns.value[key]}
          } else {selectAllFil.value[key] = itemVals}
        })
      // Для первого фильтра всегда полный список строк
      columns.value[key0] = [...Rows0]
      columnsIndexRow.value[key0] = Rows0.map((e, i) => i)

    }
    // Убираем читсим список для снятия цвета кнопки фильтра
    if(itemVals.length === [... new Set(columns.value[itemKey])].length) {
        delete filterColumns.value[itemKey]
      }
      idxListfilter.value = []
  }, { deep: true })

  // Сбрасываем все фильтра
  watch(() => filterVariant.value, (bool) => {
    if(bool === false) {
      if(Object.keys(filterColumns.value).length !== 0) {
        let key0 = Object.keys(filterColumns.value)[0].slice()
        Object.keys(filterColumns.value).reverse().map(e =>
          columnItem.value = {[key0] : [... new Set(columns.value[e])]}
        )
      }
    }
  })

// ----------------------------------------------------------------------------------------------
// Выбор строк таблицы
const selected = ref([])

const selectedRowsChanged = (item) => {
  if (selected.value.includes(item) == false) {selected.value.push(item)}
  else {selected.value.splice(selected.value.indexOf(item), 1)}
}

function toggleAll () {
    if (selected.value.length) {selected.value = []}
    else {selected.value = rows.value.map(e => e.idRow)}
  }

const selectedAll = computed(() => {
  if(selected.value.length > 0 && selected.value.length !== rows.value.length) {return null}
  if(selected.value.length === 0) {return false}
  if(selected.value.length === rows.value.length) {return true}
})

// ------------------------------------------------------------------------------------------------
//. Фиксация колонок

const lockOutline = ref(false)
const fixRef = ref([])
const notfixRef = ref([])
const fistTd = ref('')
const fistTdWidth = ref(1)

// Стили для фиксируемых колонок
const collFix = (column, idx=4) => {
  if (column.fixed) {
    return {
      'position':'sticky',
      'z-Index': idx,
      'left':`${column.left + fistTdWidth.value}px`,
      'right':`${column.right}px`,
      'color':'white',
      'background-color': '#78909C',
      'outline': '1px solid #ccc',
      'box-shadow': '5px 0px 10px #78909C',
    }
  }
}


// const rowColorSelected = (row, col) => {
//   if (selected.value.includes(row.idRow)) {
//     if (col.fixed === true) {return {'color': 'black', 'box-shadow': 'none', 'background-color': '#cfd8dc'}}
//     else {return {'background-color': '#cfd8dc'}}
//   }
// }

watchPostEffect(() => {
  fistTdWidth.value = fistTd.value.clientWidth
})

const IndentsTD = (side) => {
      // Собираем списки с ширинами колонок и их именами
      let widths = []
      let keys = []
      const columnKeysList = side === 'right' ? columnKeysReverse : columnKeys
      if(fixRef.value !== null) {columnKeysList.forEach(x =>{
        fixRef.value?.forEach(e => {
          if(x === e.id) {
            widths.push(e.clientWidth)
            keys.push(e.id)
          }})
      })}
      // Определяем отступы от границ
      let sumitem = 0
      keys?.forEach((e, i) => {
        sumitem += widths[i]
        headers.value.forEach(header => {
          if(header.name === e) {header[side] = sumitem - widths[i]
            header['width'] = widths[i]
        }})
      })
      sumitem = 0
    }

    watch(() => fixRef, (val) =>
      {
        IndentsTD('left')
        IndentsTD('right')
      },
      { deep: true }
    )


  // Сбрасываем все фиксации
  watch(() => lockOutline.value, (bool) => {
    if(bool === false) {
      headers.value.forEach(e => e.fixed = false)
    }
  })

// ------------------------------------------------------------------------------------------------
// Пагинация
const pagination = ref({
  page: 1,
  rowsPerPage: 5,

  // sortBy: 'desc',
  // descending: false,
})

// Метод Math.ceil() - округление вверх. Округляет аргумент до ближайшего большего целого.
const pagesNumber = computed(() => Math.ceil(rows.value.length / pagination.value.rowsPerPage))

const rowsPagsInd = computed(() => {
  const arr = rows.value.map(e => e.idRow)
  const res = []
  const chunkSize = pagination.value.rowsPerPage
  for (let i = 0; i < arr.length; i += chunkSize) {
    const chunk = arr.slice(i, i + chunkSize);
    res.push(chunk);
  }
  return res[pagination.value.page - 1]
})


// console.log(sliceIntoChunks(rows.value, pagination.value.rowsPerPage))

// watch(() => pagination.value.page, (val) => {
//   pagesNumber.value.map(pn => {
//     let count = 1
//     pagination.value.rowsPerPage
//   })
//   rows.value.filter(e => e.idRow > pagination.rowsPerPage)
//   })




// ------------------------------------------------------------------------------------------------
// Сортировка таблицы
const sortGeneral = ref(false)
function customSort(col) {
    console.log("Сортировка таблицы")
    const sortBy = col.name
    const descending = col.descending
    col.countSorted += 1
    if(col.countSorted === 3) {
      col.countSorted = 0
      const idRowList = rows.value.map((e, i) => e.idRow)
      return loadRows.filter(e => idRowList.includes(e.idRow))
    } else {
      const data = [...rows.value]
      if (sortBy) {
        data.sort((a, b) => {
          const x = descending ? b : a
          const y = descending ? a : b
          if (sortBy === 'name') {
            // string sort
            return x[ sortBy ] > y[ sortBy ] ? 1 : x[ sortBy ] < y[ sortBy ] ? -1 : 0
          }
          else {
            // numeric sort
            return parseFloat(x[ sortBy ]) - parseFloat(y[ sortBy ])
          }
        })
      }
      col.descending = !col.descending
      return data
    }
}

watch(() => sortGeneral.value, (val) => {
  if(val === false) {
    headers.value.map(e => {
      e.countSorted = 0
      e.descending = false
      const idRowList = rows.value.map((e, i) => e.idRow)
      rows.value = loadRows.filter(e => idRowList.includes(e.idRow))
    })
  }
  })



// ------------------------------------------------------------------------------------------------
// Поиск по таблице
const search = ref('')

watch(() => search.value, (v) => {
  rows.value = loadRows.filter(e => {for (let i in e) {if ((e[i] || '')
  .toString().toLowerCase().includes((v || '').toString().toLowerCase())) {return e}}})

  // selected.value = selected.value.filter(e => {if (rows.value.includes(e)) {return e}})
  // modifiedCells.value = rows.value.map(() => new Object())
})

// ------------------------------------------------------------------------------------------------



</script>


<style lang="sass">
$color: #90A4AE
$column: 2
$max-width: 600px
$border: 1px solid #cccccc77
$outline: 0.5px solid white

input::-webkit-outer-spin-button,
input::-webkit-inner-spin-button
  -webkit-appearance: none
  // -webkit-appearance: textfield

input[type='number']
  -moz-appearance: textfield
  background-color: $blue-grey
  border: none
  margin: 0
  padding: 0
  width: 22px
  font-size: small
  text-align: center

.search
  display: flex
  margin: 0 10px
  background-color: $blue-grey
  outline: $border
  input
    background-color: inherit
    border: none
    margin: 0
    padding: 0 10px
    // width: 100px
    font-size: medium
    // text-align: center
    display: inline-block

  input:focus, input:focus + .q-icon
    outline: $outline
    background-color: $color
    background-color: white
    color: black


  .q-icon
    // background-color: inherit
    // display: inline-block
    // border: none
    color: white



.search:hover
  background-color: $color
  // background-color: white
  color: black




// .search:hover
//   // .search:click
//   outline: $outline
//   background-color: $color


// .icon-rtl
//   padding-right: 25px
//   background-size: 20px


body
  position: relative
  min-height: 100%
  background: #ccf
  // position: relative
  // display: flex


.contener
  margin: 5px
  background-color: white
  // position: relative
  bottom: 0
  left: 0
  right: 0
  height: 100%
  // height: 500px

  .panel
    display: flex
    background-color: $blue-grey
    min-height: 35px
    align-items: center
    position: sticky
    z-index: 5
    left: 0
    top: 0
    &.botton
      box-shadow: 0px -5px 10px #78909C
      bottom: 0
      background-color: $color
      margin-top: 5px
      outline: $outline
    p
      padding: 0 5px
      margin: auto 5px
  .tableBox
    margin-top: 0.5px
    overflow: auto
    height: inherit


table
  // margin: 10px
  border-collapse: collapse
  vertical-align: middle
  text-align: center
  width: 100%

  overflow: auto
  height: inherit

  transition-property: display opacity height
  transition-duration: 0.5s

  p
    height: 100&
    vertical-align: middle
    margin: auto

  thead
    tr
      box-shadow: 0px 5px 10px #78909C
      position: sticky
      top: 0
      z-index: 7
      height: 35px
    th
      background-color: $color
      padding: 5px
      user-select: none
      outline: $outline
    .boxTd
      display: flex
      vertical-align: middle
      text-align: center

  tbody
    td
      height: inherit
      z-index: 1
      outline: $border
      background-color: white
      padding: 5px 5px
      transition-property: display opacity
      transition-duration: 0.5s
    td:nth-child(3)
      text-align: left



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
