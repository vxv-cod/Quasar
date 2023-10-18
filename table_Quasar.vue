<template>
  <!-- <p><h6>zeroModifiedCells: {{ zeroModifiedCells }}</h6></p> -->
  <!-- <div class="blackout"></div> -->


  <p><h6>menuShow: {{ menuShow }}</h6></p>
  <p><h6>filterColumns: {{ filterColumns }}</h6></p>
  <p><h6>showEditIcon: {{ showEditIcon }}</h6></p>
  <p><h6>colorEditCells: {{ colorEditCells }}</h6></p>
  <p><h6> RowsInCols: {{ RowsInCols }}</h6></p>
  <!-- <p><h6>rows: {{ rows }}</h6></p> -->
  <!-- <p><h6>blackout: {{ blackout }}</h6></p> -->
  <!-- <p><h6> selectAllFil: {{ selectAllFil }}</h6></p> -->
  <!-- <p><h6> createRowsInCols: {{ createRowsInCols }}</h6></p> -->
  <p><h6> columnItem: {{ columnItem }}</h6></p>
  <p><h6> rows: {{ rows }}</h6></p>

  <div class="q-pa-md">
    <q-table
      flat bordered
      :rows="rows"
      :columns="columns"
      row-key="id"
      selection="multiple"
      v-model:selected="selected"
      class="my-sticky"
      :loading="loading"
      v-model:pagination="pagination"
      :rows-per-page-options="[10]"
      :sort-method="customSort"

    >

    <!--
      :sort-method="customSort"

      virtual-scroll
      :rows-per-page-options="[0]"
      :virtual-scroll-sticky-size-start="48"
      style="height: 400px"
      hide-pagination
     -->

     <!-- todo Строка над таблицей -->

     <!-- title="Treats" -->
    <template v-slot:top="props">



      <!-- <div class="q-table__title">Название таблицы</div> -->
      <q-btn flat round color="white" size="sm" icon="mdi-sigma"/>
      <div style="border-right: 2px solid white; border-left: 2px solid white;">
        <!-- Редактируем таблицу -->
        <q-btn flat round color="white" size="sm" icon="mdi-pencil"
          @click="showEditIcon = !showEditIcon, !showEditIcon ? saveEditData() : null"
          :class="showEditIcon ? `text-amber` : null"
        >
        <q-tooltip anchor="top middle" self="bottom middle">Редактировать таблицу</q-tooltip>
      </q-btn>

      <!-- Отменить сохранение изменений-->
      <q-btn flat round color="white" size="sm" icon="mdi-arrow-u-left-top"
        @click="cancelEdits"
        :class="modifiedCells.every(e => Object.keys(e).length === 0) ? null : `text-amber` "
      />
    </div>

      <div style="">
        <q-btn flat round color="white" size="sm" icon="lock_open"/>
        <q-btn flat round color="white" size="sm" icon="mdi-checkbox-multiple-blank-outline"/>
        <q-btn flat round color="white" size="sm" icon="mdi-checkbox-multiple-marked-outline"/>
        <q-btn flat round color="white" size="sm" icon="mdi-magnify"/>
        <q-btn flat round color="white" size="sm" icon="mdi-filter-variant"/>
        <q-btn flat round color="white" size="sm" icon="mdi-lock-outline"/>
        <q-btn flat round color="white" size="sm" icon="mdi-lock-open-variant-outline"/>
        <q-btn flat round color="white" size="sm" icon="mdi-arrow-u-right-top"/>
        <q-btn flat round color="white" size="sm" icon="undo"/>
        <q-btn flat round color="white" size="sm" icon="redo"/>
      </div>

      <q-space />

      <q-pagination
        v-model="pagination.page"
        :max="pagesNumber"
        :min="1"
        :max-pages="6"
        boundary-numbers
        color="white"
        active-design="outline"
      />

      <q-btn
        flat round dense
        :icon="props.inFullscreen ? 'fullscreen_exit' : 'fullscreen'"
        @click="props.toggleFullscreen"
        size="md"
        color="amber"
      />

    </template>

     <!-- todo Заполняем заголовки колонок таблицы -->
    <template v-slot:header="props">
      <q-tr class="text-white">
        <q-th ref="fistTd"
          style="
            `position: sticky; left: 0; z-index: 5;`
          "
          >
          <q-checkbox class="text-white" v-model="props.selected" size="xs" dense style=""/>
        </q-th>

        <q-th
          v-for="(column, idx) in props.cols" :key="column.name"
          :props="props"
          class="text-black"
          :ref="column.fixed ? 'fixRef' : 'notfixRef'"
          :abbr="`${column.label}`"
          :id="column.name"
          :style="collFix(column, 5)"
          >
          <!-- <div class="my-flex-container" > -->
            <div class="my-flex-inner-left">
              <span >{{ column.label }}</span>
            </div>

            <!-- <pre>{{ column }}</pre> -->

            <div class="my-flex-inner-right" @click.stop="customSort">
              <div class="my-flex-inner-right">
                <menuFilter
                    v-model:menuShow="menuShow"
                    v-model:columnItem="columnItem"
                    :columnTitle="column.label"
                    :columnKey="column.name"
                    :RowsInColsItem="RowsInCols[column.name]"
                    :outsideSelect="selectAllFil[column.name]"
                    :colorBtn="Object.keys(filterColumns).includes(column.name) && column.name !=[] ? `text-amber` : null"
                    :showEditIcon="showEditIcon"
                    />
              </div>

              <!--
                v-model:columnItem="columnItem"
                    :outSelectedFilter="(value) => columnItem = value"
                    :showEditIcon="showEditIcon"


                :outsideSelect="[... new Set(selectAllFil.value[column.name])]"
                    v-model:outsideSelect="selectAllFil.value[column.name]"
                    :colorBtn="Object.keys(filterColumns).includes(column.name) && column.name !=[] ? `text-amber` : null"

                @outSelectedFilterfff="(value) => columnItem = value"
               -->

              <q-btn flat round dense color="white" size="sm"
                :class="column.fixed ? `text-amber` : null"
                :icon="columns[idx].fixed ? 'mdi-lock-outline' : 'mdi-lock-open-variant-outline'"
                @click="columns[idx].fixed = !columns[idx].fixed"
                />
            </div>

            <!--
               :outMenuShow="(value) => menuShow = value"
               v-model="menuShow"
               :mymenuShow="menuShow"
              -->
            <!-- </div> -->
          <!-- </div> -->
        </q-th>
      </q-tr>
  </template>

  <!-- todo Заполняем строки таблицы -->
  <template v-slot:body="props">
    <q-tr @click="!showEditIcon ? selectedRowsChanged(props.row) : null">

      <q-td :class="selected.includes(props.row) ? `bg-${colorTabSelectRow}` : null"
        :style="`position: sticky; left: 0; z-index: 4; background-color: white`"
        >
        <q-checkbox v-model="props.selected" size="xs" dense />
      </q-td>

      <q-td v-for="(column, index) in props.cols" :key="column.name" valign="middle"
        :props="props"
        :class="selected.includes(props.row) ? `bg-${colorTabSelectRow}` : null"
        :contenteditable="showEditIcon"
        style=""
        :style="[
            collFix(column),
            colorEditCells[column.name][props.rowIndex] !== rows[props.rowIndex][column.name] ? 'color: #0091EA;' : null,
            colorEditCells[column.name][props.rowIndex] !== rows[props.rowIndex][column.name] && column.fixed === true ?
              'background-color: white; box-shadow: none' : null,
        ]"
        @input="(val) => [tdInput(val, column.name, props.rowIndex), ]"
        >
        <div v-if="!loading">
          <p v-if="showEditIcon" style="" v-text="column.value"></p>
          <p v-if="!showEditIcon" style="user-select: none;" v-text="rows[props.rowIndex][column.name]"></p>
        </div>
        <div v-if="loading">
          <q-skeleton animation="wave"  type="text" width="100%" />
        </div>
      </q-td>
    </q-tr>

  </template>


  <!-- <template v-slot:bottom="scope">
    <q-space />
      <q-pagination
        v-model="pagination.page"
        :max="pagesNumber"
        :min="1"
        :max-pages="6"
        boundary-numbers
        color="white"
        active-design="outline"
      />
</template> -->

<!-- <template v-slot:bottom="scope"></template> -->


  <!-- todo Строка с суммой по колонкам -->
  <!-- <template v-slot:bottom-row>
      <q-tr style="
        display: flex;
        background-color: red;
        box-shadow: 0px -5px 10px #78909C
      ">
        <q-td colspan="100%">
          Строка с суммой по колонкам
        </q-td>
      </q-tr>
  </template> -->

  <!-- todo Настраиваем анимацию загрузки данных в таблицу -->
  <template v-slot:loading>
    <q-inner-loading showing color="primary" />
  </template>

  </q-table>


  <!-- todo Варианты пагинации -->
  <!-- <div class="row justify-center q-mt-md">
      <q-pagination
        v-model="pagination.page"
        color="grey-8"
        size="sm"
        :max-pages="6"
        :max="6"
        boundary-numbers
        />
  </div> -->
  <!-- <div class="q-pa-lg flex flex-center">
    <q-pagination
      v-model="pagination.page"
      color="purple"
      :max="pagesNumber"
      :max-pages="6"
      input
      size="sm"
    />
  </div> -->
  <!-- <div class="q-pa-lg flex flex-center">
    <q-pagination
      v-model="pagination.page"
      color="amber"
      :max="pagesNumber"
      :min="1"
      :max-pages="6"
      boundary-numbers
    />
  </div> -->


  <div class="q-mt-md">
    Selected: <pre>{{ selected }}</pre>
    <pre>showEditIcon: {{ showEditIcon }}</pre>
  </div>

</div>
<q-toggle v-model="loading" label="Loading state" class="q-mb-md" />



<!-- <q-resize-observer @resize="(size) => {console.log(column.label, size)}"/> -->

  <blackout :menuShow="menuShow"/>

</template>




<script>
import loadRowsCols from "../store/loadRowsCols.json"
import menuFilter from "./vxv_menu_filter.vue"
import blackout from "./blackout.vue"

// import { useMousePosition } from "./useMousePosition.js"
import { ref, reactive, computed, toRefs, watch, onMounted, watchEffect, isRef,
  isReactive,
  watchPostEffect,
  onBeforeMount,
  readonly,
  toRaw,
  markRaw,
} from 'vue'
console.log("---------------------------------------------------------")
const fixRef = ref([])
const notfixRef = ref([])
const fistTd = ref('')
const fistTdWidth = ref(1)

// const blackout = ref(document.querySelector('.blackout'))
// const blackout = document.querySelector('.blackout')

// console.log("blackout = ", blackout.value)
// console.log("blackout.value.style = ", blackout.value);
// blackout.value.style = "display: block"


const {loadRows, loadColumns} = loadRowsCols

loadColumns[0]["field"] = row => row.name
loadColumns[0]["format"] = val => `${val}`
loadColumns[6]["sort"] = (a, b) => parseInt(a, 10) - parseInt(b, 10)
loadColumns[7]["sort"] = (a, b) => parseInt(a, 10) - parseInt(b, 10)

const rows = ref(loadRows)
const columns = reactive(loadColumns)

export default {
  components: {
    menuFilter,
    blackout,
  },

  setup (props, context) {

    const selected = ref([])
    const colorTabSelectRow = "blue-grey-2"
    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      // rowsPerPage: 4
    })

    const selectedRowsChanged = (item) => {
      if (selected.value.includes(item) == false) {selected.value.push(item)}
      else {selected.value.splice(selected.value.indexOf(item), 1)}
    }

    console.log("rows_ = ", rows)

    function customSort(rows, sortBy, descending) {
      console.log("rows_customSort = ", rows)
      const data = ref(rows)
      console.log("data = ", data)
      // const data = [...rows]
    console.log("sortBy = ", sortBy)

        if (sortBy) {
          data.value.sort((a, b) => {
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

        console.log("data_customSort = ", data.value)

        return data.value
      }

    // ----------------------------------------------------------------------------------------------
    // Фиксация колонок
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

    const columnKeys = columns.map(e => e.name)
    const columnKeysReverse = [...columnKeys].reverse()

    const IndentsTD = (side) => {
      fistTdWidth.value = fistTd.value.$el.offsetWidth
      // const fistTdWidthDoc = document.getElementsByTagName('th')[0].offsetWidth
      // Собираем списки с ширинами колонок и их именами
      let widths = []
      let keys = []
      const columnKeysList = side === 'right' ? columnKeysReverse : columnKeys
      if(fixRef.value !== null) {columnKeysList.forEach(x =>{
        fixRef.value?.forEach(e => {
          if(x === e.$el.id) {
            widths.push(e.$el.clientWidth)
            keys.push(e.$el.id)
          }})
      })}
      // Определяем отступы от правой границ
      let sumitem = 0
      keys?.forEach((e, i) => {
        sumitem += widths[i]
        columns.forEach(header => {
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
    // ----------------------------------------------------------------------------------------------
    // Редактирование таблицы
    const showEditIcon = ref(false)
    const colorEditCells = ref({})
    const modifiedCells = ref([])
    const RowsInCols = ref({})
    const selectAllFil = ref({})

    // const createRowsInCols = () => {
    //   const obj = {}
    //   columnKeys.map(col => obj[col] = rows.map(e => e[col]))
    //   return obj
    // }

    const createRowsInCols = {}
    columnKeys.map(col => createRowsInCols[col] = rows.value.map(e => e[col]))

    // console.table("columnKeys = ", columnKeys)
    // console.table("rows.value = ", rows.value)
    // console.table("createRowsInCols = ", createRowsInCols)

    Object.entries(createRowsInCols).map(([key, vals]) => {
        RowsInCols.value[key] = [... new Set(vals)]
        selectAllFil.value[key] = [... new Set(vals)]
        colorEditCells.value[key] = vals.slice()
      })
      modifiedCells.value = rows.value.map(() => new Object())
      // loadRows.map((e, i) => modifiedCells[i] = new Object())


    function tdInput(val, name, index) {
      let value = val.target.textContent
      !isNaN(value) ? value = Number(value.replace(',', '.')) : null
      colorEditCells.value[name][index] = value
      value !== rows.value[index][name] ?
      modifiedCells.value[index][name] = value : delete modifiedCells.value[index][name]
    }

    function saveEditData() {
      modifiedCells.value.map((e, i) => {
        if(Object.keys(e).length !== 0) {
          Object.entries(e).forEach(([key, val]) => {
            RowsInCols.value[key][i] = val
            selectAllFil.value[key][i] = val
            rows.value[i][key] = val
          })
        }
        modifiedCells.value[i] = new Object()
      })
    }

    function cancelEdits() {
      showEditIcon.value = false
      rows.value.map((e, i) => {
        if(Object.keys(e).length !== 0) {
          Object.entries(e).forEach(([key, val]) => {
            if(key !== "id"){
              RowsInCols.value[key][i] = val
              selectAllFil.value[key][i] = val
              colorEditCells.value[key][i] = val
              modifiedCells.value[i] = new Object()
            }
          })
        }
      })
    }

    // ----------------------------------------------------------------------------------------------
    // Фильтрация колонок
    const menuShow = ref(false)
    const columnItem = ref({})
    const filterColumns = ref({})
    const idxListfilter = ref([])


    watch(() => columnItem.value, (item) => {
        // Определяем ключ и значение отправленное из диалога фильтра
        let itemKey = Object.keys(item)[0].slice()
        let itemVals = Object.values(item)[0].slice()

        // Собираем объект из выбранных строк в фильтрах по столбцам
        filterColumns.value[itemKey] = itemVals
        // Значение автовыбора всех строк в текущем фильтре

        if(Object.values(filterColumns.value).length >= 1) {
          let key0 = Object.keys(filterColumns.value)[0]
          const Rows0 = createRowsInCols[key0].slice()
          idxListfilter.value = []

          // Находим индескы из исходного списка по полученным значениям выбора в фильтрах
          createRowsInCols[itemKey].forEach((e, i) => {if (itemVals.includes(e)) {idxListfilter.value.push(i)}})

          // Из полного списка строк в таблице фильтруем те, которые есть в списке выбранных индексов
          rows.value = loadRows.filter((e, i) => idxListfilter.value.includes(i) )

          //  Корректируем фильтра в столбцах, согласно полученным индексам
          Object.entries(createRowsInCols).forEach(([key, vals]) => {
            colorEditCells.value[key] = vals.filter((e, i) => idxListfilter.value.includes(i))

            if (key !== itemKey) {
              let arr = vals.filter((e, i) => idxListfilter.value.includes(i))
              RowsInCols.value[key] = arr.slice()
              selectAllFil.value[key] = [... new Set(arr)]
              // Удаляем все фильтра кроме 1го
              if (key !== key0) {delete filterColumns.value[key]}
              } else {selectAllFil.value[key] = itemVals}
            })
          // Для первого фильтра всегда полный список строк
          RowsInCols.value[key0] = [...Rows0]
        }
        // Убираем читсим список для снятия цвета кнопки фильтра
        if(itemVals.length ===  [... new Set(RowsInCols.value[itemKey])].length)
          {delete filterColumns.value[itemKey]}
          idxListfilter.value = []
    })
    // ----------------------------------------------------------------------------------------------

    return {
      selected,
      columns,
      rows,
      colorTabSelectRow,
      loading: ref(false),
      pagination,
      pagesNumber: computed(() => Math.ceil(rows.value.length / pagination.value.rowsPerPage)),
      selectedRowsChanged,
      collFix,
      // Фиксация колонок
      fixRef,
      notfixRef,
      fistTd,
      fistTdWidth,
      // Редактирование таблицы
      showEditIcon,
      colorEditCells,
      modifiedCells,
      RowsInCols,
      selectAllFil,
      tdInput,
      customSort,
      saveEditData,
      cancelEdits,
      // Фильтрация по колонкам
      menuShow,
      columnItem,
      filterColumns,

      }

    }
  }



</script>





<style lang="sass">


.q-inner-loading
    border-radius: inherit !important

$color: #90A4AE
$column: 2
$max-width: 600px
$border: 1px solid #cccccc77
$outline: 0.5px solid white


// class="my-sticky"

.my-sticky
  // height: 510px

  //  Стили для первой колонки постоянны
  thead th
    z-index: 3
    position: sticky
    top: 0
    // left: 0
    outline: $outline
    box-shadow: 0px 5px 10px #78909C
    background-color: $color

  th:first-child
    width: 1px

  th:nth-child(2)
    text-align: center
    vertical-align: center

  // thead tr:first-child th
  //   background-color: $color

  tbody td
    height: inherit
    z-index: 1
    outline: $border
    background-color: white
    p
      margin: auto
      padding: auto

  // top
  .q-table__top
    background-color: $blue-grey
    padding: 5px
    margin: 0
    box-shadow: 0px 5px 10px #78909C
    outline: $outline
    z-index: 6

  // tfoot
  .q-table__bottom-row
    background-color: $blue-grey
    box-shadow: 0px -5px 10px #78909C
    height: 30px
    outline: $outline

  .q-table__bottom
    background-color: $blue-grey
    box-shadow: 0px -5px 10px #78909C
    height: 30px
    outline: $outline

    // padding: 0
    // margin: 0

  &.q-table--loading
    color: #cccccc77
    background-color: #cccccc77
    z-index: 1
    box-shadow: none



  .sortable
    // background-color: red
    // order: 1
  .q-table__sort-icon
    // background-color: green
    // display: none

  .my-flex-container
    display: flex
    flex-wrap: nowrap
    justify-content: center
    align-items: center
    position: relative

  .my-flex-inner-left
    display: inline-block
    // flex-grow: 1
    // order: 2

  .my-flex-inner-right
    // display: flex
    // align-items: center
    // justify-content: center
    // text-align: center
    display: inline-block

</style>
