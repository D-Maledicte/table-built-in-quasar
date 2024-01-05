<template>
  <div class="q-pa-md">
    <q-table
      
      flat bordered
      ref="tableRef"
      :rows="rows"
      :columns="columns"
      :visible-columns="visibleColumns"
      row-key="id_pedido"
      v-model:pagination="pagination"
      :loading="loading"
      :filter="filter"
      no-data-label="No hay data disponible"
      no-results-label="No se han encontrado resultados"
      loading-label="Cargando Información"
      rows-per-page-label="Pedidos por página"
      binary-state-sort
      @request="onRequest"
    >
      <template v-slot:top-left="props">
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
          style="width: 15rem; max-height: 40px;"
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
          <q-input  borderless dense :model-value="`${dateRangeInstance.from == undefined? '' : dateRangeInstance.from } - ${dateRangeInstance.to == undefined? '' : dateRangeInstance.to}`" :label="dateOption == 'fecha_bloqueo'? 'Fecha de Bloqueo' : 'Cierre Estimado'" color="orange" readonly>
              
            <template v-slot:append>
              <q-icon name="close" class="cursor-pointer"  @click.stop.prevent="onClearDate();onRequest(props)">
                <q-tooltip class="bg-bg3">Reestablecer filtros</q-tooltip>
              </q-icon>
              <q-icon name="event" class="cursor-pointer">
                <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                  <q-date v-model="dateRangeInstance" range  color="orange" mask="DD-MM-YYYY" minimal>
                    <div class="q-px-md q-pt-md q-gutter-md">
                      <div>
                        <q-btn-toggle v-model="dateOption" size="xs" spread toggle-color="orange" :options="dateOptions" />
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
          <q-toggle
            v-model="aviso"
            label="Aviso"
            color="orange"
          />
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
      </template>

      <template v-slot:body="props">
        <q-tr :props="props">
          <q-td
            v-for="col in props.cols"
            :key="col.name"
            :props="props"
          >
          <template v-if="col.name === 'acciones'">
            <q-btn icon="info" size="sm" color="green" @click="onInfo(props.row)" class="q-mx-xs">
              <q-tooltip class="bg-bg3">Mas info</q-tooltip>
            </q-btn>
            <q-btn icon="send" size="sm" color="primary" @click="onSend(props.row)" class="q-mx-xs">
              <q-tooltip class="bg-bg3">Responder pedido</q-tooltip>
            </q-btn>
            <q-btn icon="ios_share" size="sm" color="warning" @click="onUptime(props.row)" class="q-mx-xs">
              <q-tooltip class="bg-bg3">Dar de alta</q-tooltip>
            </q-btn>
          </template>
          <template v-else-if="col.name === 'aviso'">
            <template v-if="props.row.estado === 'A la espera de informacion'">
            <q-icon name="warning" size="sm" color="negative"></q-icon>
            <q-tooltip class="bg-bg3">{{ props.row.estado }}</q-tooltip>
            </template>
          </template>
          <template v-else>
              {{ col.value }}
          </template>
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
    <q-dialog v-model="medium" >
              <q-card style="width: 70vw; max-width: 70vw; heigh: 30vw; max-height: 30vw;" >
                <q-card-section >
                  <LogsTable :pedidoId="pedidoIdToPass"></LogsTable>
                </q-card-section>
              </q-card>
            </q-dialog>
  </div>
</template>
<script>
import { ref, toRaw, onMounted } from 'vue'
import { exportFile, date, useQuasar } from 'quasar'
import LogsTable from './LogsTable.vue'
import { SecondaryTableRows } from '@/assets/Seeds.mjs';

let produ = false;

const ENDPOINT = ''

const fetchData = async (id_cotizacion) => {
    try {
      const response = await fetch(`${ENDPOINT}/cotizaciones_abiertas/pedidos?cotizacionId=${id_cotizacion}`, {
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

const columns = [
  {
    name: 'id_pedido',
    label: 'ID Pedido',
    align: 'left',
    field: row => row.id_pedido,
    sortable: true,
    format: val => `${val}`,
    sort: (a, b) => a - b,
  },
  {
    name: 'aseguradora',
    required: true,
    label: 'Aseguradora',
    field: 'aseguradora',
    align: 'left',
    sortable: true,
    sort: (a, b) => a.localeCompare(b) // Sorter alfabético para Título
  },
  // Repite el patrón para otros atributos
  {
    name: 'estado',
    label: 'Estado',
    field: 'estado',
    align: 'left',
    sortable: true,
    sort: (a, b) => a.localeCompare(b) // Sorter alfabético para Estado
  },
  {
    name: 'fecha_bloqueo',
    label: 'Fecha de Bloqueo',
    field: 'fecha_bloqueo',
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
    name: 'fecha_cot_esp',
    label: 'Cierre Estimado',
    field: 'fecha_cot_esp',
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
    name: 'aviso',
    label: 'Aviso',
    align: 'center',
  },
  {
    name: 'acciones',
    label: 'Acciones',
    align: 'center',
  }
]
export default {
  props: {
    cotizacionId: {
      type: String, // o el tipo de dato correcto
      required: true
    },
    expand: {
      type: Boolean,
      required: true,
    },
  },
  watch: {
    expand(newVal) {
      if (newVal && newVal != false) {
        // La propiedad expand ha cambiado a true, ejecutar la función onRequest
        this.onRequest(this);
      }
    },
    aviso(newVal) {
      if (newVal) {
        this.onRequest(this);
      } else if (newVal == false) {
        this.onRequest(this);
      } 
    },
    dateRangeInstance(newVal) {
      if (newVal == null) {
        this.dateRangeInstance = []
      }
    }
  },
  components: { LogsTable },
  setup (props) {
    const $q = useQuasar()
    const tableRef = ref()
    const rows = ref([]);
    const fetchedRows = ref([])
    const filter = ref('')
    const loading = ref(false)
    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      rowsPerPage: 5,
      rowsNumber: 10
    })
    const medium = ref(false);
    const pedidoIdToPass = ref();
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
        },
        {
          label: 'A la espera de informacion',
          value: 'A la espera de informacion'
        },
        {
          label: 'Aviso enviado',
          value: 'Aviso enviado'
        }
      ]
    const aviso = ref(false)
    const dateOption = ref('fecha_bloqueo')
    const dateOptions = [{label: 'Fecha de Bloqueo', value: 'fecha_bloqueo'},{label: 'Cierre Estimado', value: 'fecha_cot_esp'}]
    const dateRangeInstance = ref({from: '', to: ''})
    const dateConvertion = (dateString, separation) => {
      const [day, month, year] = dateString.split(separation? separation : '-');
      return `${year}-${month}-${day}`;
    }
    
    async function fetchFromServer (startRow, fetchCount, sortBy, descending, filterInstance, estadoInstance, dateTypeInstance, avisoInstance, produ) {
      let fetchedData;
      let data;
      const fromAtribute = new Date(dateConvertion(dateRangeInstance.value.from)) 
      const toAtribute = new Date(dateConvertion(dateRangeInstance.value.to))
      
      const filterAndSortData = (data) => {
        // Lógica de filtrado y ordenación
        if (estadoInstance.length !== 0) {
          data = data.filter(row => estadoInstance.includes(row.estado));
        }

        if (dateTypeInstance == 'fecha_bloqueo' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          data = data.filter(row => {
            const createdDate = new Date(row.fecha_bloqueo);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        } else if (dateTypeInstance == 'fecha_cot_esp' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          data = data.filter(row => {
            const createdDate = new Date(row.fecha_cot_esp);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        }

        if (avisoInstance == true) {
          data = data.filter(row => row.estado == 'A la espera de informacion');
        }

        if (filterInstance) {
          data = data.filter(row => row.aseguradora.toLowerCase().includes(filterInstance.toLowerCase()) || row.estado.toLowerCase().includes(filterInstance.toLowerCase()));
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
        // Si produ es true, utiliza la lógica de fetchData
        await fetchData(props.cotizacionId).then((fetchedObject) => {
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
        // Si produ es false, utiliza originalRows como fuente de datos
        fetchedData = SecondaryTableRows;

        // Lógica de filtrado y ordenación
        data = filterAndSortData(fetchedData);
      }

      // Devuelve los datos filtrados y ordenados
      return data.length > 0? data.slice(startRow, startRow + fetchCount) : data 
    }

    // emulate 'SELECT count(*) FROM ...WHERE...'
    function getRowsNumberCount (filter, estado, dateType, aviso) {
      let sampleRows = produ? toRaw(fetchedRows.value) : SecondaryTableRows 
      let count = 0;
      

      const fromAtribute = new Date(dateConvertion(dateRangeInstance.value.from)) 
      const toAtribute = new Date(dateConvertion(dateRangeInstance.value.to))

      let filteredRows = sampleRows;

      if (estado.length !== 0) {
        filteredRows = filteredRows.filter(row => estado.includes(row.estado))
      }

      if (dateType == 'fecha_bloqueo' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          filteredRows = filteredRows.filter(row => {
            const createdDate = new Date(row.fecha_bloqueo);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        } else if (dateType == 'fecha_cot_esp' && dateRangeInstance.value.from.length !== 0 &&  dateRangeInstance.value.to.length !== 0) {
          filteredRows = filteredRows.filter(row => {
            const createdDate = new Date(row.fecha_cot_esp);
            return createdDate >= fromAtribute && createdDate <= toAtribute;
          });
        }

      if (aviso == true) {
        filteredRows = filteredRows.filter(row => row.estado == 'A la espera de informacion');
        }
      
      if (!filter) {
          return filteredRows.length;
      }

      filteredRows.forEach(pedido => {
        if (
          pedido.aseguradora.toLowerCase().includes(filter.toLowerCase()) ||
          pedido.estado.toLowerCase().includes(filter.toLowerCase())
        ) {
        ++count;
        }
      });

      return count;
    }

    async function onRequest (props) {
      const { page, rowsPerPage, sortBy, descending } = props.pagination
      const filterInstance = filter.value
      const estadoInstance = toRaw(tipoDeEstado.value)
      const dateTypeInstance = dateOption.value? dateOption.value : ''
      const avisoInstance = aviso.value
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
        const returnedData = await fetchFromServer(startRow, fetchCount, sortBy, descending, filterInstance, estadoInstance, dateTypeInstance, avisoInstance, produ)
        // update rowsCount with appropriate value
        
        if (returnedData && returnedData.length > 0) {
          pagination.value.rowsNumber = getRowsNumberCount(filterInstance, estadoInstance, dateTypeInstance, avisoInstance)
        // clear out existing data and add new
          rows.value.splice(0, rows.value.length, ...returnedData)

        // don't forget to update local pagination object
        } else {
          rows.value = []
        }
        
        pagination.value.page = page
        pagination.value.rowsPerPage = rowsPerPage
        pagination.value.sortBy = sortBy
        pagination.value.descending = descending
        // ...and turn of loading indicator
        loading.value = false
      } else {
        setTimeout(async () => {
        // update rowsCount with appropriate value
      
        // get all rows if "All" (0) is selected
        const fetchCount = rowsPerPage === 0 ? pagination.value.rowsNumber : rowsPerPage

        // calculate starting row of data
        const startRow = (page - 1) * rowsPerPage

        // fetch data from "server"
        const returnedData = await fetchFromServer(startRow, fetchCount, sortBy, descending, filterInstance, estadoInstance, dateTypeInstance, avisoInstance, produ)

        pagination.value.rowsNumber = getRowsNumberCount(filterInstance, estadoInstance, dateTypeInstance, avisoInstance)

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
    onMounted(() => {
      // get initial data from server (1st page)
      //tableRef.value.requestServerInteraction()
    });

    const onInfo = (row) => {
      medium.value = true;
      pedidoIdToPass.value = row.id_pedido
    }
    const onSend = (col) => {
      window.open(`https://www.google.com/search?q=${col.id_pedido}`, '_blank')
    }
    const onUptime = (col) => {
      window.open(`https://www.google.com/search?q=${col.id_pedido}`, '_blank')
    }
    const onClearDate = () => {
      if (dateRangeInstance.value.from && dateRangeInstance.value.to) {
        dateRangeInstance.value.from = ''
        dateRangeInstance.value.to = ''
      }
    }

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
          
          `${props.cotizacionId}-Pedidos Relacionados-${fecha}.csv`,
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
      columns,
      visibleColumns: ref([ 'aseguradora', 'estado', 'fecha_bloqueo', 'fecha_cot_esp', 'aviso', 'acciones' ]),
      rows,
      fetchedRows,
      pagination,
      filter,
      onSend,
      onUptime,
      onInfo,
      onClearDate,
      onRequest,
      exportTable,
      pedidoIdToPass,
      medium,
      tipoDeEstado,
      estadoOptions,
      aviso,
      dateOption,
      dateOptions,
      dateRangeInstance,
      loading,
    }
  }
}
</script>
