<template>
  <div class="q-pa-md">
    <q-table
      title="Log de Pedidos"
      :rows="rows"
      :columns="columns"
      row-key="name"
      style="max-width: 100%;"
      wrap-cells
      
      v-model:pagination="pagination"
      :loading="loading"
      no-data-label="No hay data disponible"
      no-results-label="No se han encontrado resultados"
      loading-label="Cargando Información"
      rows-per-page-label="Registros por página"
      binary-state-sort
      @request="onRequest"
    >
    <template v-slot:top-right>
        <div>
          {{ this.pedidoId }}
        </div>
      </template>
    </q-table>
  </div>
</template>

<script>
import { ref, toRaw } from 'vue'
import { date } from 'quasar'
import { InfoTableRows } from '@/assets/Seeds.mjs'

const produ = false

const columns = [
  {
    name: 'fecha_de_movimiento',
    required: true,
    label: 'Fecha de movimiento',
    align: 'left',
    field: row => row.fecha_de_movimiento,
    format: val => {
      const timeStamp = new Date(val)
      const formattedString = date.formatDate(timeStamp, 'DD/MM/YYYY')
      return `${formattedString}`
    },
    sortable: true
  },
  { name: 'tipo', align: 'center', label: 'Tipo', field: 'tipo', sortable: true },
  { name: 'direccion', label: 'Dirección', field: 'direccion', sortable: true },
  { name: 'resumen', align: 'center', label: 'Resumen', field: 'resumen', sortable: true, style: 'max-width: 900px; word-wrap: break-word;'}
]
const ENDPOINT = ''

const fetchData = async (id_pedido) => {
    try {
      const response = await fetch(`${ENDPOINT}/logs_pedido?pedidoId=${id_pedido}`, {
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

export default {
  props: {
    pedidoId: {
      type: String, // o el tipo de dato correcto
      required: true
    }
  },
  setup (props) {
    const rows = ref([]);
    const fetchedRows = ref([])
    const loading = ref(false)
    const pagination = ref({
      sortBy: 'desc',
      descending: false,
      page: 1,
      rowsPerPage: 5,
      rowsNumber: 10
    })
    async function fetchFromServer (startRow, count, sortBy, descending, produ) {
      let fetchedData;
      let data;
      
      const filterAndSortData = (data) => {

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
        await fetchData(props.pedidoId).then((fetchedObject) => {
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
        fetchedData = InfoTableRows;

        // Lógica de filtrado y ordenación
        data = filterAndSortData(fetchedData);
      }

      // Devuelve los datos filtrados y ordenados
      return data.length > 0? data.slice(startRow, startRow + count) : data 
    }

    function getRowsNumberCount () {
      let sampleRows = produ? toRaw(fetchedRows.value) : InfoTableRows 
      let count = 0;
      const filteredRows = sampleRows;

      filteredRows.forEach(pedido => {
        if (pedido) {
          ++count;
        }
        
      });

      return count;
    }

    async function onRequest (props) {
      const { page, rowsPerPage, sortBy, descending } = props.pagination

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
        const returnedData = await fetchFromServer(startRow, fetchCount, sortBy, descending, produ)
        // update rowsCount with appropriate value
        
        if (returnedData && returnedData.length > 0) {
          pagination.value.rowsNumber = getRowsNumberCount()
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
        const returnedData = await fetchFromServer(startRow, fetchCount, sortBy, descending, produ)

        pagination.value.rowsNumber = getRowsNumberCount()

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
    return {
      columns,
      rows: InfoTableRows,
      pagination,
      loading,
      onRequest
    }
  }
}
</script>