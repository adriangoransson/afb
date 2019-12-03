<template>
  <div class="filters">
    <div class="form-field">
      <label for="areas">Områden</label>
      <select v-model="selectedAreas" id="areas" multiple required>
        <option
          v-for="(key, index) in sortedAreas"
          :key="index"
          :value="key"
        >{{ key }} ({{ areas[key] }})</option>
      </select>
    </div>

    <div class="form-field">
      <label for="min-rooms">Rum minst</label>
      <input v-model.number="minRooms" id="min-rooms" type="number" min="1" pattern="[0-9]*">
    </div>

    <div class="form-field">
      <label for="min-square-meters">Kvadratmeter minst: {{ minSquareMeters }} kvm</label>
      <input v-model.number="minSquareMeters" id="min-square-meters" type="range" min="0" step="5">
    </div>

    <div class="form-field">
      <label for="9-month-rental">Endast 9-månaders hyra</label>
      <input v-model="shortRentalPeriod" id="9-month-rental" type="checkbox">
    </div>

    <div class="form-field">
      <label for="show-first-floor">Visa lägenheter på första våningen</label>
      <input v-model="firstFloor" id="show-first-floor" type="checkbox">
    </div>

    <div class="form-field">
      <label for="max-rent">Max hyra: {{ rent }} kr</label>
      <input
        v-model.number="rent"
        id="max-rent"
        type="range"
        min="1000"
        :max="maxRent"
        step="100"
      >
    </div>
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
    sortedAreas() {
      return Object.keys(this.areas).sort();
    },

    selectedAreas: {
      get() {
        return this.filters.areas;
      },

      set(value) {
        this.$emit('filters', 'areas', value);
      },
    },

    minRooms: {
      get() {
        return this.filters.minRooms;
      },

      set(value) {
        this.$emit('filters', 'minRooms', value);
      },
    },

    minSquareMeters: {
      get() {
        return this.filters.minSquareMeters;
      },

      set(value) {
        this.$emit('filters', 'minSquareMeters', value);
      },
    },

    shortRentalPeriod: {
      get() {
        return this.filters.shortRentalPeriod;
      },

      set(value) {
        this.$emit('filters', 'shortRentalPeriod', value);
      },
    },

    rent: {
      get() {
        return this.filters.rent;
      },

      set(value) {
        this.$emit('filters', 'rent', value);
      },
    },

    firstFloor: {
      get() {
        return this.filters.firstFloor;
      },

      set(value) {
        this.$emit('filters', 'firstFloor', value);
      },
    },
  },
};
</script>

<style scoped>
  .filters {
    padding-top: 10px;
  }

  .form-field {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    line-height: 1.4rem;
    padding: 5px 0;
  }

  .form-field label {
    flex: 1 0 180px;
  }

  .form-field input, .form-field select {
    flex: 1 1 100px;
    font-size: initial;
  }

  .form-field input[type=checkbox] {
    flex: 0 15px;
  }
</style>
