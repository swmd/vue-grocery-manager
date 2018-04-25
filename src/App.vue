<template>

  <b-container fluid>
    <!-- Title Area -->
    <b-row class="justify-content-md-center mb-4 mt-4">
      <h1>{{ subtitle }}</h1>
    </b-row>

    <!-- User Interface controls -->
    <b-row>
      <b-col md="6" class="my-1">
        <b-form-group horizontal label="Filter" class="mb-0">
          <b-input-group>
            <b-form-input v-model="filter" placeholder="Type to Search" />
            <b-input-group-append>
              <b-btn :disabled="!filter" @click="filter = ''">Clear</b-btn>
            </b-input-group-append>
          </b-input-group>
        </b-form-group>
      </b-col>
      <b-col md="6" class="my-1">
        <b-form-group horizontal label="Sort" class="mb-0">
          <b-input-group>
            <b-form-select v-model="sortBy" :options="sortOptions">
              <option slot="first" :value="null">-- none --</option>
            </b-form-select>
            <b-form-select :disabled="!sortBy" v-model="sortDesc" slot="append">
              <option :value="false">Asc</option>
              <option :value="true">Desc</option>
            </b-form-select>
          </b-input-group>
        </b-form-group>
      </b-col>
      <b-col md="6" class="my-1">
        <b-pagination :total-rows="totalRows" :per-page="perPage" v-model="currentPage" class="my-0" />
      </b-col>
      <b-col md="6" class="my-1">
        <b-form-group horizontal label="Per page" class="mb-0">
          <b-form-select :options="pageOptions" v-model="perPage" />
        </b-form-group>
      </b-col>
    </b-row>

    <!-- Main table element -->
    <b-table show-empty
             stacked="md"
             :striped=true
             :outlined=true
             :items="items"
             :fields="fields"
             :current-page="currentPage"
             :per-page="perPage"
             :filter="filter"
             :sort-by.sync="sortBy"
             :sort-desc.sync="sortDesc"
             @filtered="onFiltered"
    >
      <!-- <template slot="name" slot-scope="row">{{row.value.first}} {{row.value.last}}</template>
      <template slot="isActive" slot-scope="row">{{row.value?'Yes :)':'No :('}}</template> -->
      <template slot="actions" slot-scope="row">
        <!-- We use @click.stop here to prevent a 'row-clicked' event from also happening -->
        <b-button size="sm" @click.stop="usedUp(row.item, row.index, $event.target)" class="mr-1">
          Used Up
        </b-button>
      </template>
    </b-table>

    <b-row class="mb-4 mt-4">
      <h3>Add Grocery</h3>
    </b-row>

    <b-form @submit="onSubmit">
      <b-form-group id="groceryNameInputGroup"
                    label="Grocery Name:"
                    label-for="groceryName">
        <b-form-input id="groceryName"
                      type="text"
                      v-model="form.name"
                      required
                      placeholder="Grocery name">
        </b-form-input>
      </b-form-group>
      <b-form-group id="fridgeGroup"
                    label="Fridge:"
                    label-for="fridge">
        <b-form-select id="fridge"
                      :options="fridges"
                      required
                      v-model="form.fridge">
        </b-form-select>
      </b-form-group>
      <b-button type="submit" variant="primary">Add</b-button>
    </b-form>
  </b-container>
</template>

<script>
const items = [
  { name: 'bread', amount: 2, location: 'Fridge 1'},
  { name: 'butter', amount: 1, location: 'Fridge 1'},
  { name: 'cheese', amount: 3, location: 'Fridge 2'},
  { name: 'jam', amount: 2, location: 'Fridge 2'},
  { name: 'a', amount: 3, location: 'Fridge 1'},
  { name: 'b', amount: 1, location: 'Fridge 1'},
  { name: 'c', amount: 2, location: 'Fridge 2'},
]

export default {
  data () {
    return {
      subtitle: 'Your Grocery Manager',
      items: items,
      fields: [
        { key: 'name', label: 'Name', sortable: true },
        { key: 'amount', label: 'Amount', sortable: true, 'class': 'text-center' },
        { key: 'location', label: 'Location' },
        { key: 'actions', label: 'Actions' }
      ],
      currentPage: 1,
      perPage: 5,
      totalRows: items.length,
      pageOptions: [ 5, 10, 15 ],
      sortBy: null,
      sortDesc: false,
      filter: null,
      form: {
        name: '',
        fridge: ''
      },
      fridges: [
        {text: 'Fridge 1', value: 'Fridge 1'},
        {text: 'Fridge 2', value: 'Fridge 2'}
      ]
    }
  },
  computed: {
    sortOptions () {
      // Create an options list from our fields
      return this.fields
        .filter(f => f.sortable)
        .map(f => { return { text: f.label, value: f.key } })
    }
  },
  methods: {
    usedUp (item, index, button) {
      if (item.amount > 1) {
        this.items[index].amount = item.amount - 1;
      } else {
        this.items.splice(index, 1);
      }
    },
    onFiltered (filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
    onSubmit (evt) {
      evt.preventDefault();
      if (!this.form.name || !this.form.fridge) return;
      let index = -1;
      this.items.forEach((item, i) => {
        if (item.name === this.form.name) {
          index = i;
        }
      });
      if (index >= 0) {
        this.items[index].amount += 1;
      } else {
        this.items.push({
          name: this.form.name,
          amount: 1,
          location: this.form.fridge
        });
      }
    },
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
