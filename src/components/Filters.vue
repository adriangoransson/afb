<template>
  <div>
    <select v-model="area">
      <option :value="null">Alla områden</option>
      <option
        v-for="(areaCount, area) in areas"
        :key="area"
        :value="area"
      >{{ area }} ({{ areaCount }})</option>
    </select>

    <label for="min-rooms">Rum minst</label>
    <input v-model.number="minRooms" id="min-rooms" type="number" min="1">

    <label for="min-square-meters">Kvadratmeter minst</label>
    <input v-model.number="minSquareMeters" id="min-square-meters" type="range" min="0" step="5">
    {{ minSquareMeters }} kvm

    <label for="9-month-rental">9-månaders hyra</label>
    <input v-model="shortRentalPeriod" id="9-month-rental" type="checkbox">

    <label for="max-rent">Max hyra</label>
    <input v-model.number="rent" id="max-rent" type="range" min="0" :max="maxRent" step="100">
    {{ rent }} kr
  </div>
</template>

<script>
export default {
  props: {
    areas: {
      required: true,
    },
    filters: {
      required: true,
    },
    maxRent: {
      required: true,
    },
  },

  computed: {
    area: {
      get() {
        return this.filters.area;
      },

      set(value) {
        this.$emit('area', value);
      },
    },

    minRooms: {
      get() {
        return this.filters.minRooms;
      },

      set(value) {
        this.$emit('minRooms', value);
      },
    },

    minSquareMeters: {
      get() {
        return this.filters.minSquareMeters;
      },

      set(value) {
        this.$emit('minSquareMeters', value);
      },
    },

    shortRentalPeriod: {
      get() {
        return this.filters.shortRentalPeriod;
      },

      set(value) {
        this.$emit('shortRentalPeriod', value);
      },
    },

    rent: {
      get() {
        return this.filters.rent;
      },

      set(value) {
        this.$emit('rent', value);
      },
    },
  },
};
</script>
