<template>
  <div class="outer">
    <div @click="showMore" class="title">
      {{ data.rooms }} rum, {{ data.squareMeters }} kvm. Våning {{ data.floor }}.
      <span v-if="shortRentalPeriod">9 månaders hyra. </span>
      <span :title="priceInfo" class="tooltip">{{ data.rent }} kr</span>
      <br>
      {{ data.area }}, {{ data.address }}
    </div>
    <div v-if="extendedInfo">
      <pre>{{ details }}</pre>
      <a :href="blueprintURL">
        <img :src="blueprintURL" alt="Ritning">
      </a>
    </div>
  </div>
</template>

<script>
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

    blueprintURL() {
      return `https://www.afbostader.se${this.details.blueprint}`;
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
};
</script>

<style scoped>
  .outer {
    padding: 10px;
  }

  .outer .title {
    cursor: pointer;
  }

  .tooltip {
    text-decoration: underline dotted;
    cursor: help;
  }
</style>
