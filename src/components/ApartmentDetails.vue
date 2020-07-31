<template>
  <div class="details">
    <a target="_blank" :href="afbURL">
      Öppna hos AFB
    </a>

    <p>
      <table>
        <tr>
          <th>Möbler</th>
          <td>{{ data.furniture }}</td>
        </tr>

        <tr>
          <th>Kök</th>
          <td>{{ data.citchen }}</td>
        </tr>

        <tr>
          <th>Balkong/Uteplats</th>
          <td>{{ data.balkony }}</td>
        </tr>

        <tr>
          <th>Hiss</th>
          <td>{{ data.elevator }}</td>
        </tr>

        <tr>
          <th>Värme/Vatten</th>
          <td>{{ data.heating }}</td>
        </tr>

        <tr>
          <th>El</th>
          <td>{{ data.electricity }}</td>
        </tr>

        <tr>
          <th>Förråd</th>
          <td>{{ data.storeIncluded }}</td>
        </tr>

        <tr>
          <th>Läge</th>
          <td>{{ data.location }}</td>
        </tr>
      </table>
    </p>

    <p>{{ data.descriptionText }}</p>

    <p class="image-container">
      <a target="_blank" :href="blueprintURL"><img :src="blueprintURL" alt="Ritning"></a>
    </p>

    <div class="maps">
      <img :src="mapsURL" alt="Satellite view">
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

  computed: {
    afbURL() {
      return `https://www.afbostader.se/lediga-bostader/bostadsdetalj/?mode=0&area=${this.data.area}&obj=${this.data.productId}`;
    },

    blueprintURL() {
      return `https://www.afbostader.se${this.data.blueprint}`;
    },

    mapsURL() {
      const address = encodeURIComponent(`${this.data.addressgroup}, ${this.data.zipcode} ${this.data.city}`);
      return `https://maps.google.com/maps/api/staticmap?center=${address}&zoom=17&size=440x300&maptype=satellite&markers=color:red%7Clabel:%7C${address}&key=AIzaSyCbmn5bUDuTStyK9i09MDR5T2mVAzUNJLI`;
    },
  },
};
</script>

<style scoped>
  .details {
    margin-top: 10px;
    padding: 10px;
    background: hsl(0, 0%, 95%);
  }

  .maps, .image-container {
    overflow-x: scroll;
  }

  .details tr {
    margin-top: 3px;
  }

  .details th {
    text-align: left;
    padding-right: 15px;
  }
</style>
