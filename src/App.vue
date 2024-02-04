<template>
  <div class="menu-container">
    <span v-bind:style="{ backgroundColor: currentPage == 'check'? '#c03461' : '' }"  @click="currentPage = 'check'">利用者チェック</span>
    <span v-bind:style="{ backgroundColor: currentPage == 'input'? '#c03461' : '' }"   @click="currentPage = 'input'">入力</span>
    <span v-bind:style="{ backgroundColor: currentPage == 'calendar'? '#c03461' : '' }"   @click="currentPage = 'calendar'">カレンダー</span>
    
    <!-- 
    <div>認証</div>
    <div>データ</div>
    <div>流れている音楽について</div> -->
  </div>
  
      
        
      
  <template v-if="currentPage == 'check'">
    <template v-if="modifiedData">
       <!-- Month Selection Dropdown -->
    <select v-model="selectedMonth" @change="selectMonth(selectedMonth)">
      <option value="">Select a Month</option>
      <!-- Generate options dynamically based on available data -->
      <option v-for="group in groupedItemsByMonth" :key="group.date" :value="group.date">
        {{ group.date.substring(0, 4) }}-{{ group.date.substring(4, 6) }}
      </option>
    </select>

    <div v-if="selectedMonth">
      <h2 style="float: right; margin: -45px auto 0;">{{ selectedMonth.substring(0, 4) }}-{{ selectedMonth.substring(4, 6) }}</h2>
      <div v-for="item in filteredItems.dates" :key="item" class="schedule-list-container" data-aos="fade-up">
        <h2>{{ getDay(item.date) }}</h2>
        <ul v-for="el in item.items" :key="el" class="schedule-list">
          <div class="list-top">
            {{ getTime(el.startTime) }} - {{ getTime(el.endTime) }}
          </div>
          <div class="list-bottom">
            {{ el.userName }}
          </div>
        </ul>
        <!-- {{ item.name }} - {{ item.date }} -->
      </div>
    </div>
      <!-- <div v-for="group in groupedItemsByDate" :key="group.date" data-aos="fade-up">
        <h2>{{ getDay(group.date) }}</h2>
        <ul class="schedule-list-container">
          <div v-for="item in group.items" :key="item.id" class="schedule-list">
            <div class="list-top">
              {{ getTime(item.startTime) }} - {{ getTime(item.endTime) }}
            </div>
            <div class="list-bottom">
              {{ item.userName }}
            </div>
          </div>
        </ul>
      </div> -->
    </template>
  </template>

  <template v-if="currentPage == 'input'">
    ここでアプリから、予約できるようにします<br>（すぐ！）
  </template>

  <template v-if="currentPage == 'calendar'">
    ここにはカレンダーが追加されます<br>（すぐとは言っていない）
  </template>

  

</template>

<script>
  import AOS from 'aos'
  import 'aos/dist/aos.css'

  export default {
    data() {
      return {
        isFetchingData: false,
        sheetData: null,
        currentPage: null,

        selectedMonth: '',
        
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
          this.sheetData = json.values;

        } catch (error) {
          console.error(error);
        } finally {
          this.isFetchingData = false;
          console.log(this.groupedItemsByMonth[0].date)
          this.selectedMonth = this.groupedItemsByMonth[0].date
        }
      },

      selectMonth(month) {
        this.selectedMonth = month;
        console.log(this.filteredItems)
      },
      convertDate(numericDate) {
        const dateStr = String(numericDate);
        
        
        const year = parseInt(dateStr.substring(0, 4));
        const month = parseInt(dateStr.substring(4, 6)) - 1; // Months are zero-based
        const day = parseInt(dateStr.substring(6, 8));
        
        const daysOfWeek = ['日曜日', '月曜日', '火曜日', '水曜日', '木曜日', '金曜日', '土曜日'];
        const date = new Date(year, month, day);
        const dayOfWeek = daysOfWeek[date.getDay()];

        const formattedDate = dateStr.substring(0, 4) + "年" + dateStr.substring(4, 6) + "月" + dateStr.substring(6, 8) + "日" + dayOfWeek;

        return formattedDate
      },

      getDay(numericDate) {
        const dateStr = String(numericDate);
        const year = parseInt(dateStr.substring(0, 4));
        const month = parseInt(dateStr.substring(4, 6)) - 1; // Months are zero-based
        const day = parseInt(dateStr.substring(6, 8));
        
        const daysOfWeek = ['日', '月', '火', '水', '木', '金', '土'];
        const date = new Date(year, month, day);
        const dayOfWeek = daysOfWeek[date.getDay()];
        const formattedDate = dateStr.substring(0, 4) + "年" + dateStr.substring(4, 6) + "月" + dateStr.substring(6, 8) + "日" + "（"+dayOfWeek + "）";

        return formattedDate;
      },
      getTime(time){
        const timeStr = String(time);
        if (timeStr.length >= 2) {
          const lastIndex = timeStr.length - 2;
          const firstPart = timeStr.substring(0, lastIndex);
          const lastPart = timeStr.substring(lastIndex);
          return firstPart + ':' + lastPart;
        } else {
          return timeStr;
        }
      },

    },

    async mounted() {
        console.clear()
        AOS.init();

        this.currentPage = 'check'

        await this.fetchSheetData();
        // console.log(this.modifiedData)
        console.log(this.groupedItemsByMonth)
    },

    computed: {
      modifiedData() {
        if(!this.sheetData) return
        const data = this.sheetData.slice(1).map((item) => {
          return {
            date: item[0],
            startTime: item[1],
            endTime: item[2],
            userName: item[3],
            status: item[4],
            result: item[5],
          };
        });

        return data;
      },

      groupedItemsByMonth() {
        if (!this.modifiedData) return;

        const groupedData = [];

        this.modifiedData.forEach(item => {
            // Convert the number to a string and extract the year, month, and date
            const dateString = item.date.toString(); // Assuming item.date is in YYYYMMDD format
            const yearMonth = dateString.slice(0, 6); // Get the YYYYMM part of the date
            const fullDate = dateString; // Use the full YYYYMMDD date for daily grouping

            // Find or create the group for this yearMonth
            let monthGroup = groupedData.find(group => group.date === yearMonth);
            if (!monthGroup) {
                monthGroup = { date: yearMonth, dates: {} };
                groupedData.push(monthGroup);
            }

            // Find or create the date group within the month group
            if (!monthGroup.dates[fullDate]) {
                monthGroup.dates[fullDate] = [];
            }
            monthGroup.dates[fullDate].push(item);
        });

        // Sort the grouped data by year and month
        groupedData.sort((a, b) => b.date.localeCompare(a.date));

        // Sort each month's dates and their items
        groupedData.forEach(monthGroup => {
            Object.keys(monthGroup.dates).forEach(date => {
                // Sort the items within each date by startTime
                monthGroup.dates[date].sort((a, b) => a.startTime - b.startTime);
            });

            // Convert dates from object to array and sort by date
            monthGroup.dates = Object.entries(monthGroup.dates).sort((a, b) => a[0].localeCompare(b[0])).map(([date, items]) => ({ date, items }));
        });

        return groupedData;
      },

      filteredItems() {
        if (!this.selectedMonth) return {};

        // Use find to return the first matching group or undefined
        const foundGroup = this.groupedItemsByMonth.find(group => group.date === this.selectedMonth);
        
        // If foundGroup is undefined, return an empty object instead
        return foundGroup || {};
      },





    },

  };
</script>


<style>
  html{
   
    font-family: 'M PLUS Rounded 1c', sans-serif;
    color: white;
  }


  body{
    background: #051c1c;
    width: 500px;
    max-width: 90vw;
    margin: auto;
  }

  .menu-container{
    display: flex;
    flex-wrap: nowrap;
    margin: 20px auto;
    justify-content: space-around;
  }

  .menu-container span{
    /* margin-right: 10px; */
    background: darkgrey;
    padding: 2.5px 10px;
    border-radius: 2.5px;
    transition: all .5s ease-in-out;
  }

  

  .schedule-list-container{
    list-style-type: none;
    padding: 0;
  }

  .schedule-list-container h2{
    color: #34c064;
  }

  .schedule-list{
    background: #e1f8e9 ;
    color: #051720;
    margin-bottom: 20px;
    padding: 5px 15px;
    border-radius: 5px;
  }

  .schedule-list .list-top{
    font-weight: bold;
    font-size: 1.5em;
  }

  .schedule-list .list-bottom{
    text-align: right;
  }

</style>