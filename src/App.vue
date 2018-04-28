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

    <b-row>
      <b-button @click="showModal">
        Edit
      </b-button>
    </b-row>

    <b-modal v-model="modalShow">
      <div class="mb-5">Move groceries between Fridge 1 and Fridge 2</div>
      <b-row>
        <b-col md="5">
          <b-form-select v-model="fridge1Selection" :options="fridge1Items" class="mb-3" :select-size="10" multiple>
          </b-form-select>
          <div>Selected: <strong>{{ fridge1Selection }}</strong></div>
        </b-col>
        <b-col md="2" class="button-container">
          <b-button @click="moveFrom2To1" class="mb-5"> << </b-button>
          <b-button @click="moveFrom1To2"> >> </b-button>
        </b-col>
        <b-col md="5">
          <b-form-select v-model="fridge2Selection" :options="fridge2Items" class="mb-3" :select-size="10" multiple>
          </b-form-select>
          <div>Selected: <strong>{{ fridge2Selection }}</strong></div>
        </b-col>
      </b-row>
      <div slot="modal-footer" class="w-100">
        <b-row>
          <b-col class="md-6 centered">
            <b-btn variant="primary" @click="applyChanges">
              Apply
            </b-btn>
          </b-col>
          <b-col class="md-6 centered">
            <b-btn variant="primary" @click="cancel">
              Cancel
            </b-btn>
          </b-col>
        </b-row>        
      </div>
    </b-modal>

    <b-row class="mb-4 mt-4">
      <h3>Add Grocery</h3>
    </b-row>

    <b-form @submit="onSubmit" novalidate validated=true class="was-validated">
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

export default {
  data () {
    let items = localStorage.getItem('groceries');
    if (items) {
      items = JSON.parse(items);
    } else {
      items = [];
    }
    return {
      modalShow: false,
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
      ],
      fridge1Items: [],
      fridge2Items: [],
      fridge1Selection: [],
      fridge2Selection: []
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
      this.updateGroceries();
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
        if (item.name === this.form.name && item.location === this.form.fridge) {
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
      this.updateGroceries();
    },
    updateGroceries() {
      localStorage.setItem('groceries', JSON.stringify(this.items));
    },
    showModal() {
      this.modalShow = !this.modalShow;
      this.fridge1Items = this.items.filter(item => item.location === 'Fridge 1').map(item => {
        return {
          value: item.name,
          text: item.name,
          amount: item.amount
        }
      });
      this.fridge2Items = this.items.filter(item => item.location === 'Fridge 2').map(item => {
        return {
          value: item.name,
          text: item.name,
          amount: item.amount
        }
      });
    },
    moveFrom2To1() {
      // console.log('fridge2 selection: ', this.fridge2Selection)
      if (this.fridge2Selection.length === 0) return;
      this.fridge2Selection.forEach(selection => {
        let selectedIndex = this.fridge2Items.findIndex(item => item.value === selection);
        if (selectedIndex >= 0) {
          let itemIndex = this.fridge1Items.findIndex(item => item.value === selection);
          if (itemIndex < 0) {
            this.fridge1Items.push(this.fridge2Items[selectedIndex]);
          } else {
            this.fridge1Items[itemIndex].amount += this.fridge2Items[selectedIndex].amount;
          }        
          this.fridge2Items.splice(selectedIndex, 1);
        }
      });
      this.fridge2Selection = [];      
    },
    moveFrom1To2() {
      if (this.fridge1Selection.length === 0) return;
      this.fridge1Selection.forEach(selection => {
        let selectedIndex = this.fridge1Items.findIndex(item => item.value === selection);
        if (selectedIndex >= 0) {
          let itemIndex = this.fridge2Items.findIndex(item => item.value === selection);
          if (itemIndex < 0) {
            this.fridge2Items.push(this.fridge1Items[selectedIndex]);
          } else {
            this.fridge2Items[itemIndex].amount += this.fridge1Items[selectedIndex].amount;
          }        
          this.fridge1Items.splice(selectedIndex, 1);
        }
      });
      this.fridge1Selection = [];
    },
    applyChanges() {
      this.items = [];
      this.fridge1Items.forEach(item => {
        this.items.push({
          name: item.value,
          amount: item.amount,
          location: 'Fridge 1'
        });
      });
      this.fridge2Items.forEach(item => {
        this.items.push({
          name: item.value,
          amount: item.amount,
          location: 'Fridge 2'
        });
      });
      this.updateGroceries();
      this.modalShow = false;
    },
    cancel() {
      this.modalShow = false;
    }
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

.button-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.centered {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
