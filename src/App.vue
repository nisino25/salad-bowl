<template>
  <div>
    <ul v-if="!isFetchingData && sheetData">
      <li v-for="(row, index) in sheetData" :key="index">
        <!-- Display your data here -->
        {{ row }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isFetchingData: false,
      sheetData: null,
      baseData: null,
      
    };
  },
  methods: {
    async fetchSheetData() {
      this.isFetchingData = true;

      const sheetID = '1VVcGRFrpePsE-F9g-SS4LWj_Xzf_ymTki-eVaAFZxfk'
      const sheetName = '予約情報'
      const APIKey = 'AIzaSyAPiDSKoO-WxsVVnkm4frjMvczC84r6G1o'
      
      // Replace with your Google Sheets API URL
      const URL = `https://sheets.googleapis.com/v4/spreadsheets/${sheetID}/values/${sheetName}?key=${APIKey}`;
      
      try {
        const res = await fetch(URL);
        const json = await res.json();
        // Assuming your data is in a simple array format in the 'values' property
        // You may need to adjust this depending on the actual structure of your sheet data
        this.sheetData = json.values;
        // If your data needs further processing to fit into your application, do it here
        console.table(this.sheetData);
      } catch (error) {
        console.error(error);
      } finally {
        this.isFetchingData = false;
      }
    },

  },
  mounted() {
    this.fetchSheetData();
  }
};
</script>
