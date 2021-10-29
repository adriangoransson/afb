<template>
  <div class="outer">
    <div @click="showMore" class="title">
      <h2>{{ data.rooms }} rum, {{ data.sqrMtrs }} kvm. <small>Våning {{ data.floor }}</small></h2>

      {{ data.rent }} kr <small class="price-info">{{ priceInfo }}</small>
      <span v-if="shortRentalPeriod"> (9 månader)</span><br>

      Inflytt&nbsp;{{ data.moveInDate }}.
      Sista&nbsp;ansökan&nbsp;{{ data.reserveUntilDate }}<br>

      {{ data.area }}, {{ data.address }}
    </div>

    <div v-if="extendedInfo">
      <ApartmentDetails v-if="details.productId" :data="details" />

      <p v-if="loading">
        Hämtar mer information...
      </p>
      <p v-else-if="error">
        Kunde inte hämta mer information
        <pre>{{ error }}</pre>
      </p>
    </div>
  </div>
</template>

<script>
import ApartmentDetails from './ApartmentDetails.vue';

export default {
  props: {
    data: {
      required: true,
    },
  },

  data() {
    return {
      loading: false,
      error: null,
      details: null,
      extendedInfo: false,
    };
  },

  computed: {
    shortRentalPeriod() {
      return this.data.rentalPeriods === 9;
    },

    priceInfo() {
      if (this.shortRentalPeriod) {
        return `${this.pricePerSquareMeter}. ${this.yearlyRent}`;
      }

      return this.pricePerSquareMeter;
    },

    pricePerSquareMeter() {
      return `${Math.round(this.data.rent / this.data.sqrMtrs)} kr/kvm`;
    },

    yearlyRent() {
      return `${Math.round((this.data.rent * 9) / 12)} kr/månad för ett år`;
    },
  },

  methods: {
    async showMore() {
      this.extendedInfo = !this.extendedInfo;

      // Reset and try again
      if (this.extendedInfo && this.error !== null) {
        this.error = null;
        this.details = null;
      } else if (this.details !== null) {
        return;
      }

      this.details = {};
      // Slight delay to avoid flashing loading indicator when response is fast
      setTimeout(() => {
        if (!this.error && !this.details.productId) {
          this.loading = true;
        }
      }, 300);

      try {
        const data = await fetch(`https://api.afbostader.se:442/redimo/rest/vacantproducts/${this.data.productId}`);
        this.details = await data.json();
      } catch (e) {
        this.error = e;
      } finally {
        this.loading = false;
      }
    },
  },

  components: {
    ApartmentDetails,
  },
};
</script>

<style scoped>
  .outer {
    padding: 10px 0;
  }

  .title {
    cursor: pointer;
  }

  .title h2 {
    font-size: 1.4rem;
    margin: 5px 0;
  }

  .title .price-info {
    color: rgb(50, 50, 50);
    font-style: italic;
  }
</style>
