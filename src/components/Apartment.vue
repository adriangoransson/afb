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
    <div v-if="extendedInfo && details.productId" class="details">
      <a target="_blank" :href="`https://www.afbostader.se/lediga-bostader/bostadsdetalj/?mode=0&area=${data.area}&obj=${data.productId}`">
        Öppna hos AFB
      </a>

      <p>
        <table>
          <tr>
            <th>Möbler</th>
            <td>{{ details.furniture }}</td>
          </tr>

          <tr>
            <th>Kök</th>
            <td>{{ details.citchen }}</td>
          </tr>

          <tr>
            <th>Balkong/Uteplats</th>
            <td>{{ details.balkony }}</td>
          </tr>

          <tr>
            <th>Hiss</th>
            <td>{{ details.elevator }}</td>
          </tr>

          <tr>
            <th>Värme/Vatten</th>
            <td>{{ details.heating }}</td>
          </tr>

          <tr>
            <th>El</th>
            <td>{{ details.electricity }}</td>
          </tr>

          <tr>
            <th>Förråd</th>
            <td>{{ details.storeIncluded }}</td>
          </tr>

          <tr>
            <th>Läge</th>
            <td>{{ details.location }}</td>
          </tr>
        </table>
      </p>

      <p>{{ details.descriptionText }}</p>

      <p class="image-container">
        <a target="_blank" :href="blueprintURL"><img :src="blueprintURL" alt="Ritning"></a>
      </p>

      <div class="maps">
        <img :src="mapsURL" alt="Satellite view">
      </div>
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

    mapsURL() {
      const address = encodeURIComponent(`${this.details.addressgroup}, ${this.details.zipcode} ${this.details.city}`);
      return `https://maps.google.com/maps/api/staticmap?center=${address}&zoom=17&size=440x300&maptype=satellite&markers=color:red%7Clabel:%7C${address}&key=AIzaSyCbmn5bUDuTStyK9i09MDR5T2mVAzUNJLI`;
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
    padding: 10px 0;
  }

  .outer .title {
    cursor: pointer;
  }

  .outer .details {
    margin-top: 10px;
    padding: 10px;
    background: hsl(0, 0%, 95%);
  }

  @media (pointer: fine) {
    .tooltip {
      text-decoration: underline dotted;
      cursor: help;
    }
  }

  .maps, .image-container {
    overflow-x: scroll;
    background: white;
  }

  h2 {
    font-size: 1.4rem;
    margin: 5px 0;
  }

  tr {
    margin-top: 3px;
  }

  th {
    text-align: left;
    padding-right: 15px;
  }
</style>
