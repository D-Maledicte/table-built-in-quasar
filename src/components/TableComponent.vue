<template>
  <div class="q-pa-md main-section">
    <q-table
      flat bordered
      ref="tableRef"
      row-key="id_cotizacion"
      :rows="rows"
      :columns="columns"
      v-model:expanded="expanded"
      v-model:pagination="pagination"
      :loading="loading"
      :insurances="insurances"
      :filter="filter"
      :visible-columns="visibleColumns"
      no-data-label="No hay data disponible"
      no-results-label="No se han encontrado resultados"
      loading-label="Cargando Información"
      rows-per-page-label="Registros por página"
      binary-state-sort
      @request="onRequest"
    >
      <template v-slot:top-left="props" >
        <div class="q-gutter-md row">
          <q-input
            borderless
            dense
            debounce="300"
            v-model="filter"
            placeholder="Buscar"
          >
          <template v-slot:append>
            <q-icon name="search" />
          </template>
          </q-input>
          <q-expansion-item
            expand-separator
            :icon="expItem"
            @click="onExpansionItem()"
            label="Filtrar por seccion"
            style="width: 40rem; "
          >
            <div class="row">
              <q-select
                
                borderless
                dense
                v-model="insurances"
                :options="insuranceOptions"
                label="Aseguradora"
                color="orange"
                multiple
                emit-value
                map-options
                style="max-height: 40px;"
                class="col-6"
              >
                <template v-slot:option="{ itemProps, opt, selected, toggleOption }">
                  <q-item v-bind="itemProps">
                  <q-item-section>
                    <q-item-label v-html="opt.label" />
                  </q-item-section>
                  <q-item-section side>
                    <q-toggle :model-value="selected"  @update:model-value="toggleOption(opt)" @click.stop.prevent="onRequest(props)" color="orange"/>
                  </q-item-section>
                  </q-item>
                </template>
                <template v-slot:append>
                  <q-icon name="close"  class="cursor-pointer" @click.stop.prevent="insurances = []; onRequest(props)">
                    <q-tooltip class="bg-bg3">Reestablecer filtros</q-tooltip>
                  </q-icon>
                </template>
              </q-select>
              <q-select
                
                borderless
                dense
                v-model="tipoDeEstado"
                :options="estadoOptions"
                label="Estado"
                color="orange"
                multiple
                emit-value
                map-options
                style="max-height: 40px;"
                class="col-6"
              >
                <template v-slot:option="{ itemProps, opt, selected, toggleOption }">
                  <q-item v-bind="itemProps">
                  <q-item-section>
                    <q-item-label v-html="opt.label" />
                  </q-item-section>
                  <q-item-section side>
                    <q-toggle :model-value="selected"  @update:model-value="toggleOption(opt)" @click.stop.prevent="onRequest(props)" color="orange"/>
                  </q-item-section>
                  </q-item>
                </template>
                <template v-slot:append>
                  <q-icon name="close"  class="cursor-pointer" @click.stop.prevent="tipoDeEstado = []; onRequest(props)">
                    <q-tooltip class="bg-bg3">Reestablecer filtros</q-tooltip>
                  </q-icon>
                </template>
              </q-select>
              <q-select
                
                borderless
                dense
                v-model="tipoDeRiesgo"
                :options="riesgoOptions"
                label="Riesgo"
                color="orange"
                multiple
                emit-value
                map-options
                style="max-height: 40px;"
                class="col-6"
              >
                <template v-slot:option="{ itemProps, opt, selected, toggleOption }">
                  <q-item v-bind="itemProps">
                  <q-item-section>
                    <q-item-label v-html="opt.label" />
                  </q-item-section>
                  <q-item-section side>
                    <q-toggle :model-value="selected"  @update:model-value="toggleOption(opt)" @click.stop.prevent="onRequest(props)" color="orange"/>
                  </q-item-section>
                  </q-item>
                </template>
                <template v-slot:append>
                  <q-icon name="close"  class="cursor-pointer" @click.stop.prevent="tipoDeRiesgo = []; onRequest(props)">
                    <q-tooltip class="bg-bg3">Reestablecer filtros</q-tooltip>
                  </q-icon>
                </template>
              </q-select>
              <q-input
                borderless
                dense
                :model-value="`${dateRangeInstance.from == undefined? '' : dateRangeInstance.from } - ${dateRangeInstance.to == undefined? '' : dateRangeInstance.to}`"
                :label="dateOption == 'creado'? 'Creado' : 'Cierre Estimado'"
                color="orange"
                readonly
                class="col-6">
                  
                  <template v-slot:append>
                    <q-icon name="close" class="cursor-pointer"  @click.stop.prevent="onClearDate();onRequest(props)">
                    <q-tooltip class="bg-bg3">Reestablecer filtros</q-tooltip>
                    </q-icon>
                    <q-icon name="event" class="cursor-pointer">
                      <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                        <q-date v-model="dateRangeInstance" range  color="orange" mask="DD-MM-YYYY" minimal>
                          <div class="q-px-md q-pt-md q-gutter-md">
                            <div>
                              <q-btn-toggle v-model="dateOption" size="xs" spread toggle-color="orange" :options="dateOptions"/>
                            </div>
                            <div class="row justify-center">
                              <q-btn v-close-popup label="Filtrar" size="sm" color="orange" flat @click.stop.prevent="onRequest(props)" />
                            </div>
                          </div>
                        </q-date>
                      </q-popup-proxy>
                      <q-tooltip class="bg-bg3">Seleccione un rango de fecha</q-tooltip>
                    </q-icon>
                  </template>
              </q-input>
            </div>
              
          </q-expansion-item>
          <q-icon side size="md" name="delete" class="cursor-pointer"  @click.stop.prevent="insurances = [];tipoDeEstado = [];tipoDeRiesgo = [];onClearDate();onRequest(props)" style="align-self: center;">
            <q-tooltip class="bg-bg3">Reestablecer filtros</q-tooltip>
          </q-icon>
        </div>
      </template>

      <template v-slot:top-right>
        <q-btn
          color="orange"
          icon-right="archive"
          no-caps
          size="sm"
          @click="exportTable"
        >
          <q-tooltip class="bg-bg3">Exportar a CSV</q-tooltip>
        </q-btn>
        <q-toggle
            v-model="darkMode"
            checked-icon="nightlight_round"
            color="black"
            unchecked-icon="dark_mode"
            @click.stop.prevent="onDarkMode()"
        >
        <q-tooltip class="bg-bg3">Activar modo oscuro</q-tooltip>
        </q-toggle>
      </template>
      
      <template v-slot:header="props">
        <q-tr :props="props">
          <q-th auto-width />

          <q-th
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
          >
            {{ col.label }}
          </q-th>
        </q-tr>
      </template>

      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td auto-width>
            <div>
              <q-btn-dropdown color="orange" size="sm" label="" :dropdown-icon="props.row.dropdownIcon" @click="onClicked(props)">
              </q-btn-dropdown>
              <q-tooltip class="bg-bg3">Pedidos Relacionados</q-tooltip>
            </div>
            
          </q-td>

          <q-td
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
          >
          <!-- Condición para renderizar botones en la columna "accion" -->
          <template v-if="col.name === 'accion'">
              <q-btn icon="close" size="sm" color="negative" @click="onClose(props.row)">
                <q-tooltip class="bg-bg3">Cerrar cotizacíon</q-tooltip>
              </q-btn>
          </template>
          <!-- Renderizar el valor de la columna para otras columnas -->
          <template v-else>
              {{ col.value }}
          </template>
          </q-td>
        </q-tr>
        <!-- Renderizar la anidacion de tabla de pedidos enviando su id para procesar la solicitud -->
        <q-tr v-show="props.expand" :props="props">
          <q-td colspan="100%">
            <div class="text-left">
              <ExpansionTable :cotizacionId="props.row.id_cotizacion" :expand="props.expand"></ExpansionTable>
            </div>
          </q-td>
        </q-tr>
      </template>
      <template  v-slot:no-data="{ message }">
        <div class="full-width row flex-center text-black q-gutter-sm" >
          <q-icon color="orange" size="2em" name="hourglass_full" />
          <span >
            {{ message }}
          </span>
        </div>
      </template>
    </q-table>
  </div>
</template>

<script>
import { ref, onMounted, toRaw } from 'vue'
import { exportFile, date, useQuasar } from 'quasar'
import ExpansionTable from './Anidacion/ExpansionTable.vue'
import { MainTableRows } from '@/assets/Seeds.mjs';

let produ = false;

const ENDPOINT = ''

//API request on Wordpress DB
const fetchData = async (insurancesInstance) => {
    try {
      const aseguradorasQueryParam = insurancesInstance.length > 0 ? `&aseguradora=${insurancesInstance.join(',')}` : '';
      console.log(aseguradorasQueryParam)
      const response = await fetch(`${ENDPOINT}/cotizaciones_abiertas?${aseguradorasQueryParam}`, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json',
          // eslint-disable-next-line
          'X-WP-Nonce': user_info.nonce,
        },
        credentials: 'include',
      });
      return await response.json();
    }
    catch (error) {
      console.error('Error al obtener datos:', error);
    }
};

//Esquema de columnas de la tabla principal
const columns = [
  {
    name: 'id_cotizacion',
    label: 'ID Cotización',
    align: 'left',
    field: row => row.id_cotizacion.toString(),
    sortable: true,
    format: val => `${val}`,
    sort: (a, b) => a.localeCompare(b, undefined, { numeric: true, sensitivity: 'base' }),
  },
  {
    name: 'titulo',
    label: 'Título',
    field: 'titulo',
    align: 'left',
    sortable: true,
    sort: (a, b) => a.localeCompare(b) // Sorter alfabético para Título
  },
  {
    name: 'estado',
    label: 'Estado',
    field: 'estado',
    align: 'left',
    sortable: true,
    sort: (a, b) => a.localeCompare(b) // Sorter alfabético para Estado
  },
  {
    name: 'tomador',
    label: 'Tomador',
    field: 'tomador',
    align: 'left',
    sortable: true,
    sort: (a, b) => a.localeCompare(b)
  },
  {
    name: 'riesgo',
    label: 'Riesgo',
    field: 'riesgo',
    align: 'left',
    sortable: true,
    sort: (a, b) => a.localeCompare(b) // Sorter alfabético para Riesgo
  },
  {
    name: 'fecha_creacion',
    label: 'Creado',
    field: 'fecha_creacion',
    align: 'left',
    sortable: true,
    format: val => {
      const timeStamp = new Date(val)
      const formattedString = date.formatDate(timeStamp, 'DD/MM/YYYY')
      return `${formattedString}`
    },
    sort: (a, b) => new Date(a) - new Date(b) // Sorter para fechas en Creado
  },
  {
    name: 'fecha_cierre',
    label: 'Cierre Estimado',
    field: 'fecha_cierre',
    align: 'left',
    sortable: true,
    format: val => {
      const timeStamp = new Date(val)
      const formattedString = date.formatDate(timeStamp, 'DD/MM/YYYY')
      return `${formattedString}`
    },
    sort: (a, b) => new Date(a) - new Date(b)
  },
  {
    name: 'accion',
    label: 'Accion',
    align: 'center',
  }
]



export default {
  components: { ExpansionTable },
  watch: {
    dateRangeInstance(newVal) {
      if (newVal == null) {
        this.dateRangeInstance = []
      }
    }
  },
  setup () {
    const $q = useQuasar()
    const darkMode = ref(false);
    const tableRef = ref()
    const rows = ref([])
    const fetchedRows = ref([])
    const filter = ref('')
    const loading = ref(false)
    const insurances = ref([])
    const insuranceOptions = [
        {
          label: 'Chubb',
          value: 'Chubb'
        },
        {
          label: 'Federacion Patronal',
          value: 'Federacion Patronal'
        },
        {
          label: 'Mercantil Andina',
          value: 'Mercantil Andina'
        },
        {
          label: 'Provincia',
          value: 'Provincia'
        },
        {
          label: 'La Segunda',
          value: 'La Segunda'
        }
      ]
    const tipoDeEstado = ref([])
    const estadoOptions = [
        {
          label: 'Cerrado ganado',
          value: 'Cerrado ganado'
        },
        {
          label: 'Sin contactar',
          value: 'Sin contactar'
        },
        {
          label: 'Cerrado perdido',
          value: 'Cerrado perdido'
        }
      ]
    const tipoDeRiesgo = ref([])
    const riesgoOptions = [
        {
          label: 'Aeronavegacion',
          value: 'Aeronavegacion'
        },
        {
          label: 'ICO / TRO',
          value: 'ICO / TRO'
        },
        {
          label: 'TRO',
          value: 'TRO'
        }
      ]
    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      rowsPerPage: 10,
      rowsNumber: 10
    })
    const dateOption = ref('creado')
    const dateOptions = [{label: 'Creado', value: 'creado'},{label: 'Cierre Estimado', value: 'cierre_estimado'}]
    const dateRangeInstance = ref({from: '', to: ''})
    const expItem = ref('menu')
    const onExpansionItem = () => {
      if (expItem.value == 'menu') {
        expItem.value = 'menu_open'
      } else {
        expItem.value = 'menu'
      }
    }
    const onClicked = (props) => {
        props.expand = !props.expand;
        if (props.expand == false) {
          props.row.dropdownIcon = 'remove';
        } else {
        props.row.dropdownIcon = 'add';
        }
    }
    const onClose = (col) => {
      window.open(`https://www.google.com/search?q=${col.id_cotizacion}`, '_blank')
    }
    const onClearDate = () => {
      if (dateRangeInstance.value.from && dateRangeInstance.value.to) {
        dateRangeInstance.value.from = ''
        dateRangeInstance.value.to = ''
      }
    }
    const onDarkMode = () => {
      $q.dark.toggle()
    }
    const dateConvertion = (dateString, separation) => {
      const [day, month, year] = dateString.split(separation? separation : '-');
      return `${year}-${month}-${day}`;
    }
    //Funcion de filtrado y procesado de la data proveniente de la DB para su correcto despliegue
    async function fetchFromServer(startRow, count, sortBy, descending, filterInstance, insurancesInstance, estadoInstance, riesgoInstance, dateTypeInstance, produ) {
      let fetchedData;
      let data;
      const fromAtribute = new Date(dateConvertion(dateRangeInstance.value.from)) 
      const toAtribute = new Date(dateConvertion(dateRangeInstance.value.to))

      const filterAndSortData = (data) => {
        // Lógica de filtrado por aseguradora
        if (insurancesInstance.length !== 0) {
          data = data.filter(row => insurancesInstance.includes(row.aseguradora));
        }
        if (estadoInstance.length !== 0) {
          data = data.filter(row => estadoInstance.includes(row.estado));
        }

        if (riesgoInstance.length !== 0) {
          data = data.filter(row => riesgoInstance.includes(row.riesgo));
        }

        if (dateTypeInstance == 'creado' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          data = data.filter(row => {
            const createdDate = new Date(row.fecha_creacion);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        } else if (dateTypeInstance == 'cierre_estimado' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          data = data.filter(row => {
            const createdDate = new Date(row.fecha_cierre);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        }

        // Lógica de filtrado y ordenación
        if (filterInstance) {
          data = data.filter(row => row.titulo.toLowerCase().includes(filterInstance.toLowerCase()) || row.riesgo.toLowerCase().includes(filterInstance.toLowerCase()) || row.tomador.toLowerCase().includes(filterInstance.toLowerCase()));
        }

        if (sortBy) {
          const sortFn = sortBy === 'desc'
            ? (descending
              ? (a, b) => (a.name > b.name ? -1 : a.name < b.name ? 1 : 0)
              : (a, b) => (a.name > b.name ? 1 : a.name < b.name ? -1 : 0)
            )
            : (descending
              ? (a, b) => (parseFloat(b[sortBy]) - parseFloat(a[sortBy]))
              : (a, b) => (parseFloat(a[sortBy]) - parseFloat(b[sortBy]))
            );

          data.sort(sortFn);
        }

        return data;
      };

      if (produ) {
        // En produccion realiza una solitud por medio de fetchData, procesa la informacion y le da un formato
        await fetchData(insurancesInstance).then((fetchedObject) => {
          if (fetchedObject.data && fetchedObject.data.length > 0) {
            fetchedRows.value = fetchedObject.data;
            fetchedData = fetchedRows.value;
            // Lógica de filtrado y ordenación
            data = filterAndSortData(fetchedData);
          } else {
      // Manejar el caso en el que no hay datos
          data = [] 
      // Puedes asignar un valor predeterminado a 'data' o realizar otra acción según tu lógica
          }
        });
      } else {
        // En desarrollo utiliza un set preestablecido de informacion para realizar pruebas de logica y estilo grafico
        fetchedData = MainTableRows;
        // Lógica de filtrado y ordenación
        data = filterAndSortData(fetchedData);
      }

      // Devuelve los datos filtrados y ordenados
      return data.length > 0? data.slice(startRow, startRow + count) : data
    }
    // Funcion que provee de un conteo adaptado a las filas que seran renderizadas en la tabla para el elemento de paginacion
    function getRowsNumberCount (filter, insurances, estado, riesgo, dateType) {
      let sampleRows = produ? toRaw(fetchedRows.value) : MainTableRows 
      let count = 0;

      const fromAtribute = new Date(dateConvertion(dateRangeInstance.value.from)) 
      const toAtribute = new Date(dateConvertion(dateRangeInstance.value.to))

      let filteredRows = sampleRows;
      if (insurances.length !== 0) {
        filteredRows = filteredRows.filter(row => insurances.includes(row.aseguradora))
      }

      if (estado.length !== 0) {
        filteredRows = filteredRows.filter(row => estado.includes(row.estado))
      }

      if (riesgo.length !== 0) {
        filteredRows = filteredRows.filter(row => riesgo.includes(row.riesgo))
      }

      if (dateType == 'creado' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          filteredRows = filteredRows.filter(row => {
            const createdDate = new Date(row.fecha_creacion);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        } else if (dateType == 'cierre_estimado' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          filteredRows = filteredRows.filter(row => {
            const createdDate = new Date(row.fecha_cierre);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        }

      if (!filter) {
        return filteredRows.length;
      }

      filteredRows.forEach(cotizacion => {
        if (
          cotizacion.titulo.toLowerCase().includes(filter.toLowerCase()) ||
          cotizacion.riesgo.toLowerCase().includes(filter.toLowerCase()) ||
          cotizacion.tomador.toLowerCase().includes(filter.toLowerCase())
        ) {
        ++count;
        }
      });

      return count;
    }
    // Funcion que haciendo uso de las funciones previamente comentadas ensambla el componente tabla para su correcto despliegue visual
    async function onRequest (props) {
      const { page, rowsPerPage, sortBy, descending } = props.pagination
      const filterInstance = filter.value
      const insurancesInstance = toRaw(insurances.value)
      const estadoInstance = toRaw(tipoDeEstado.value)
      const riesgoInstance = toRaw(tipoDeRiesgo.value)
      const dateTypeInstance = dateOption.value? dateOption.value : ''
      loading.value = true

      // emulate server
      if (produ) {
        /// PROBLEMA DE DECLARACION E INICIALIZACION: FETCHCOUNT Y STARTROW DEBEN IR ANTES DE RETURNED PERO FETCHCOUNT LABURA
        // CON DATO DE PAGINATION.VALUE.NUMBER // SOLUCIONADO PROBAR RENDIMIENTO
        // get all rows if "All" (0) is selected
        const fetchCount = rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage

        // calculate starting row of data
        const startRow = (page - 1) * rowsPerPage

        // fetch data from "server"
        const returnedData = await fetchFromServer(startRow, fetchCount, sortBy, descending, filterInstance, insurancesInstance, estadoInstance, riesgoInstance, dateTypeInstance, produ)
        
        if (returnedData && returnedData.length > 0) {
          // update rowsCount with appropriate value
          pagination.value.rowsNumber = getRowsNumberCount(filterInstance, insurancesInstance, estadoInstance, riesgoInstance, dateTypeInstance)


          const modifiedData = returnedData.map(row => ({ ...row, expand: false, dropdownIcon: 'add' }));

          // clear out existing data and add new
          rows.value.splice(0, rows.value.length, ...modifiedData)
        } else {
          rows.value = []
        }
        // don't forget to update local pagination object
        pagination.value.page = page
        pagination.value.rowsPerPage = rowsPerPage
        pagination.value.sortBy = sortBy
        pagination.value.descending = descending

        // ...and turn of loading indicator
        loading.value = false
      } else {
        setTimeout(async () => {
        // update rowsCount with appropriate value
        pagination.value.rowsNumber = getRowsNumberCount(filterInstance, insurancesInstance, estadoInstance, riesgoInstance, dateTypeInstance)

        // get all rows if "All" (0) is selected
        const fetchCount = rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage

        // calculate starting row of data
        const startRow = (page - 1) * rowsPerPage

        // fetch data from "server"
        const returnedData = await fetchFromServer(startRow, fetchCount, sortBy, descending, filterInstance, insurancesInstance, estadoInstance, riesgoInstance, dateTypeInstance, produ)

        const modifiedData = returnedData.map(row => ({ ...row, expand: false, dropdownIcon: 'add' }));

        // clear out existing data and add new
        rows.value.splice(0, rows.value.length, ...modifiedData)

        // don't forget to update local pagination object
        pagination.value.page = page
        pagination.value.rowsPerPage = rowsPerPage
        pagination.value.sortBy = sortBy
        pagination.value.descending = descending

        // ...and turn of loading indicator
        loading.value = false
      }, 1500)
      }
    }
    //Realiza la primer solicitud al servidor una vez que la tabla se carga en la pagina
    onMounted(() => {
      // get initial data from server (1st page)
      tableRef.value.requestServerInteraction()
    })

    function wrapCsvValue (val, formatFn, row) {
      let formatted = formatFn !== void 0
        ? formatFn(val, row)
        : val

      if (!isNaN(formatted / 0)) {
        formatted = `"${formatted}"`;
      }

      if (typeof formatted === 'string' && date.isValid(formatted)) {
        formatted = dateConvertion(formatted, '/');
      }

      formatted = formatted === void 0 || formatted === null
        ? ''
        : String(formatted)

      formatted = formatted.split('"').join('""')
      /**
       * Excel accepts \n and \r in strings, but some other CSV parsers do not
       * Uncomment the next two lines to escape new lines
       */
      // .split('\n').join('\\n')
      // .split('\r').join('\\r')

      return `"${formatted}"`
    }

    const exportTable = () => {
        // naive encoding to csv format
        const content = [columns.map(col => wrapCsvValue(col.label))].concat(
          rows.value.map(row => columns.map(col => wrapCsvValue(
            typeof col.field === 'function'
              ? col.field(row)
              : row[ col.field === void 0 ? col.name : col.field ],
            col.format,
            row
          )).join(','))
        ).join('\r\n')

        const timeStamp = Date.now()
        const fecha = date.formatDate(timeStamp, 'YYYY-MM-DD')
        const hora = date.formatDate(timeStamp, 'HH:mm:ss')

        
        const status = exportFile(
          
          `Cotizaciones Abiertas-${fecha}.csv`,
          content,
          'text/csv'
        )

        if (status !== true) {
          $q.notify({
            message: 'Browser denied file download...',
            color: 'negative',
            icon: 'warning'
          })
        } else {
          $q.notify({
            message: 'Browser file download succeded',
            caption: `Horario de Creacion:${hora}`,
            color: 'primary',
            icon: 'warning'
          })
        }
    }
    return {
      tableRef,
      darkMode,
      filter,
      loading,
      pagination,
      columns,
      visibleColumns: ref([ 'titulo', 'estado', 'tomador', 'riesgo', 'fecha_creacion', 'fecha_cierre', 'accion' ]),
      rows,
      fetchedRows,
      expanded: ref([]),
      expItem,
      onClose,
      onClicked,
      onRequest,
      onClearDate,
      onExpansionItem,
      onDarkMode,
      exportTable,
      insurances,
      insuranceOptions,
      tipoDeEstado,
      estadoOptions,
      tipoDeRiesgo,
      riesgoOptions,
      dateRangeInstance,
      dateOption,
      dateOptions,
    }
    }
}
</script>

<style>
.main-section {
  font-size: 0.95em;
  white-space: normal;
  margin-top: 4px;
}
</style>