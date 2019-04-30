<template>
  <div class="outer">
    <div @click="showMore" class="title">
      <h2>{{ data.rooms }} rum, {{ data.sqrMtrs }} kvm. <small>Våning {{ data.floor }}</small></h2>

      <span :title="priceInfo" class="tooltip">{{ data.rent }} kr</span>
      <span v-if="shortRentalPeriod"> (9 månader)</span><br>

      Inflytt&nbsp;{{ data.moveInDate }}.
      Sista&nbsp;ansökan&nbsp;{{ data.reserveUntilDate }}<br>

      {{ data.area }}, {{ data.address }}
    </div>

    <ApartmentDetails v-if="extendedInfo && details.productId" :data="details" />
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

      if (this.details !== null) {
        return;
      }

      this.details = {};

      const data = await fetch(`https://www.afbostader.se/redimo/rest/vacantproducts/${this.data.productId}`);
      this.details = await data.json();
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

  @media (pointer: fine) {
    .tooltip {
      text-decoration: underline dotted;
      cursor: help;
    }
  }

  .title {
    cursor: pointer;
  }

  .title h2 {
    font-size: 1.4rem;
    margin: 5px 0;
  }
</style>
