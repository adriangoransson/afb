<template>
  <div id="app">
    <h1>Lediga lägenheter på AF Bostäder</h1>

    <details class="filters"><summary>Filtrera</summary>
    <Filters
      :areas="areas"
      :filters="filters"
      :maxRent="maxRent"
      @area="filters.area = $event"
      @rent="filters.rent = $event"
      @minRooms="filters.minRooms = $event"
      @minSquareMeters="filters.minSquareMeters = $event"
      @shortRentalPeriod="filters.shortRentalPeriod = $event"
      @firstFloor="filters.firstFloor = $event"
    />
  </details>

  <hr>

  <Apartment v-for="apt in display" :key="apt.productId" :data="apt" />

  </div>
</template>

<script>
import Apartment from './components/Apartment.vue';
import Filters from './components/Filters.vue';

export default {
  data() {
    return {
      data: [],
      areas: {},
      maxRent: 0,
      filters: {
        area: null,
        rent: 0,
        minRooms: 1,
        minSquareMeters: 1,
        shortRentalPeriod: false,
        firstFloor: true,
      },
    };
  },

  computed: {
    display() {
      return this.data.filter(({
        area,
        rent,
        rooms,
        rentalPeriods,
        sqrMtrs,
        floor,
      }) => {
        if (this.filters.area && this.filters.area !== area) {
          return false;
        }

        if (this.filters.rent && this.filters.rent < rent) {
          return false;
        }

        if (this.filters.minRooms && this.filters.minRooms > rooms) {
          return false;
        }

        if (this.filters.minSquareMeters && this.filters.minSquareMeters > sqrMtrs) {
          return false;
        }

        if (this.filters.shortRentalPeriod === true && rentalPeriods !== 9) {
          return false;
        }

        if (this.filters.firstFloor === false && floor === 1) {
          return false;
        }

        return true;
      });
    },
  },

  created() {
    this.fetch();
  },

  methods: {
    parse(data) {
      return data
        .filter(item => item.type === 'Lägenhet')
        .map((item) => {
          const {
            area,
            shortDescription,
            rentalPeriods,
            sqrMtrs,
            floor,
          } = item;

          const rent = parseInt(item.rent, 10);
          this.maxRent = Math.max(this.maxRent, rent);
          this.filters.rent = this.maxRent;

          const areaCount = (this.areas[area] || 0) + 1;
          this.areas[area] = areaCount;

          return {
            ...item,
            rent,
            rooms: Number(shortDescription.toLowerCase().match(/([\d]+) rum/)[1]),
            sqrMtrs: Number(sqrMtrs),
            floor: Number(floor),
            rentalPeriods: Number(rentalPeriods),
          };
        });
    },

    async fetch() {
      const resp = await fetch('https://www.afbostader.se/redimo/rest/vacantproducts');
      const json = await resp.json();
      if (json.error !== null) {
        this.error = json.error;
        return;
      }

      this.data = this.parse(json.product);
    },
  },

  components: {
    Apartment,
    Filters,
  },
};
</script>

<style>
  body {
    line-height: 1.2rem;
    font-family:  -apple-system,
                  BlinkMacSystemFont,
                  'Segoe UI',
                  Roboto,
                  Oxygen,
                  Ubuntu,
                  Cantarell,
                  'Open Sans',
                  'Helvetica Neue',
                  sans-serif;
  }

  #app {
    max-width: 700px;
    margin: auto;
  }

  #app h1 {
    line-height: 2rem;
  }

  @media (max-width: 720px) {
    #app {
      margin-left: 10px;
      margin-right: 10px;
    }
  }

  .filters summary {
    font-size: 1.2rem;
    font-weight: bold;
    /* text-align: center; */
  }
</style>
