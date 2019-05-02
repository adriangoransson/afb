<template>
  <div id="app">
    <div>
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

      <div v-if="error" style="text-align: center">
        Fel vid hämtning av data från <a href="https://www.afbostader.se/">AF Bostäder</a>
        <pre>{{ error }}</pre>
      </div>

      <div v-else>
        <small>
          <span v-if="display.length < data.length">Visar {{ display.length }} av </span>

          {{ loading ? 'Hämtar' : data.length }}

          lägenhet{{ data.length !== 1 ? 'er' : '' }}
        </small>

        <Apartment v-for="apt in display" :key="apt.productId" :data="apt" />
      </div>
    </div>
    <div>
      <footer>
        Av <a href="https://adriang.se">Adrian Göransson</a>.
        Mer information på <a href="https://github.com/adriangoransson/afb">GitHub</a>.
      </footer>
    </div>
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
      maxRent: 13000,
      error: null,
      loading: true,
      filters: {
        area: null,
        rent: 0,
        minRooms: 1,
        minSquareMeters: 0,
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

        if (this.filters.rent < rent) {
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
          this.maxRent = Math.max(this.maxRent, rent + 100);
          this.filters.rent = this.maxRent;

          const areaCount = (this.areas[area] || 0) + 1;
          this.$set(this.areas, area, areaCount);

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
      try {
        const resp = await fetch('https://www.afbostader.se/redimo/rest/vacantproducts');
        const json = await resp.json();

        if (json.error !== null) {
          this.error = json.error;
          return;
        }

        this.data = this.parse(json.product);
      } catch (e) {
        this.error = e;
      } finally {
        this.loading = false;
      }
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
    margin: 0;
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

  html, body, #app {
    height: 100%;
  }

  a {
    color: hsl(200, 100%, 30%);
    text-decoration: none;
  }

  a:hover {
    text-decoration: underline;
  }

  #app {
    max-width: 700px;
    margin: auto;
    display: flex;
    flex-direction: column;
  }

  #app div:first-child {
    flex: 1 0 auto;
  }

  #app div:last-child {
    flex-shrink: 0;
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
    cursor: pointer;
  }

  footer {
    text-align: center;
    padding: 1rem;
    font-size: 0.8rem;
  }
</style>
