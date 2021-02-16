<template>
  <!-- To do ... Convert html to pug -->

      <!--  v-container-->
      <!--    v-row-->
      <!--      v-col(cols)-->
      <!--        DataTable(-->
      <!--          v-if="items.length"-->
      <!--          :items="items"-->
      <!--  :headers:-->
      <!--        )-->
      <!--        v-progress-circular(-->
      <!--          v-else-->
      <!--          width="2"-->
      <!--          color="rs__primary"-->
      <!--          indeterminate-->
      <!--        ).mx-auto-->
  <div>
    <v-container>
      <v-row>
        <v-col>
          <v-data-table v-if="items.length"
                        :items="itemsComputed"
                        :headers="headers"
                        :search="search"
                        class="elevation-1"
                        :options.sync="options"
                        :server-items-length="totalItems"
                        :loading="loading">
            <template v-slot:item.user="{ item }">
                {{ getFullName(item) }}
            </template>
            <template v-slot:top>
              <v-text-field
                v-model="search"
                label="Search"
                class="mx-4"
                append-icon="mdi-magnify"
              ></v-text-field>
            </template>
            <template v-slot:body.prepend>
              <tr>
                <td v-for="header in headers">
                  <v-text-field
                    v-if="header.text === 'Gender'"
                    v-model="gender"
                    type="text"
                    append-icon="mdi-magnify"
                    label="Gender">
                  </v-text-field>
                </td>
              </tr>
            </template>
          </v-data-table>

          <v-progress-circular v-else width="2"
                               color="rs__primary"
                               indeterminate class="mx-auto">
          </v-progress-circular>

        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'

export default {
  components: {
    DataTable
  },
  data() {
    return {
      sales,
      items: [],
      search: '',
      gender: '',
      totalItems: 0,
      loading: true,
      options: {}
    }
  },
  async created() {
    this.getDataFromApi()
  },
  methods: {
    async fetchData() {
      let { page, itemsPerPage } = this.options

      let items = this.sales.results.filter( item => {
        return (item.user.first_name.toLowerCase().includes( this.search.toLowerCase()) ||
          item.user.last_name.toLowerCase().includes( this.search.toLowerCase()) ||
          item.email.toLowerCase().includes( this.search.toLowerCase()) ||
          item.gender.toLowerCase().includes( this.search.toLowerCase()) ||
          item.year.toString().includes( this.search.toLowerCase()) ||
          item.sales.toLowerCase().includes( this.search.toLowerCase()) ||
          item.country.toLowerCase().includes( this.search.toLowerCase())) &&
          item.gender.toLowerCase().includes( this.gender.toLowerCase())
      })

      if (!items.length) {
        items = sales.results
      }

      const total = items.length
      if (items.length && itemsPerPage > 0) {
        items = items.slice((page - 1) * itemsPerPage, page * itemsPerPage)
      }

      await this.delay(100)

      return {items, total}
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
    getFullName(item) {
      return `${item.user.first_name} ${item.user.last_name}`
    },
    getDataFromApi () {
      this.loading = true
      this.fetchData().then(data => {
        this.items = data.items
        this.totalItems = data.total
        this.loading = false
      })
    },
  },
  computed: {
    headers() {
      return [
        {text: 'Name', value: 'user.full_name', align: 'start', width: 16.66 + '%', sortable: false},
        {text: 'Email', value: 'email', width: 16.66 + '%', sortable: false},
        {text: 'Gender', value: 'gender', width: 16.66 + '%', sortable: false},
        {text: 'Year', value: 'year', width: 16.66 + '%', sortable: false},
        {text: 'Sales', value: 'sales', width: 16.66 + '%', sortable: false},
        {text: 'Country', value: 'country', width: 16.66 + '%', sortable: false}
      ];
    },
    itemsComputed() {
      return this.items.map(item => {
        return {...item, user:{...item.user, full_name: `${item.user.first_name} ${item.user.last_name}` }}
      })
    },
  },
  watch: {
    options: {
      handler () {
        this.getDataFromApi()
      },
      deep: true,
    },
    search() {
      if (this.timer) {
        clearTimeout(this.timer);
        this.timer = null;
      }
      this.timer = setTimeout(() => {
        this.getDataFromApi()
      }, 800);
    },
    gender() {
      if (this.timer) {
        clearTimeout(this.timer);
        this.timer = null;
      }
      this.timer = setTimeout(() => {
        this.getDataFromApi()
      }, 800);
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>


<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>
