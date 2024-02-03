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
    <div v-for="group in groupedItemsByDate" :key="group.date" data-aos="fade-up">
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
    </div>
    </template>
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
        }
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
        console.log(this.modifiedData)
        // console.log(this.groupedItemsByMonth)
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

        const groupedData = this.modifiedData.reduce((accumulator, item) => {
          // Extract the year and month from the date
          const yearMonth = item.date.toString().slice(0, 6); // This will give us 'YYYYMM'
          const formattedYearMonth = `${yearMonth.slice(0, 4)}-${yearMonth.slice(4, 6)}`; // Formats as 'YYYY-MM'

          // If the accumulator doesn't have this year-month, add it
          if (!accumulator[formattedYearMonth]) {
            accumulator[formattedYearMonth] = [];
          }

          // Push the current item to the correct group
          accumulator[formattedYearMonth].push(item);
          return accumulator;
        }, {});

        // Sort the grouped data by the bigger number in formattedYearMonth
        const sortedGroupedData = Object.keys(groupedData).sort((a, b) => {
          return parseInt(b.replace('-', ''), 10) - parseInt(a.replace('-', ''), 10);
        }).reduce((accumulator, key) => {
          accumulator[key] = groupedData[key];
          return accumulator;
        }, {});

        return sortedGroupedData;
      },
      groupedItemsByDate() {
        if (!this.modifiedData) return;

        const groupedData = [];

        this.modifiedData.forEach(item => {
          // Format the date as yyyymmdd
          const dateKey = item.date;

          // Check if a group with this dateKey already exists
          const existingGroup = groupedData.find(group => group.date === dateKey);

          if (!existingGroup) {
            // If it doesn't exist, create a new group
            const newGroup = {
              date: dateKey,
              items: [item],
            };
            groupedData.push(newGroup);
          } else {
            // If it exists, add the item to the existing group's items array
            existingGroup.items.push(item);
          }

          
        });

        // Sort the grouped data by date
        groupedData.sort((a, b) => b.date.localeCompare(a.date));

          // Sort items within each date group by item.startTime in ascending order
          groupedData.forEach(group => {
            group.items.sort((a, b) => a.startTime - b.startTime);
          });


        return groupedData;
      },



    },

  };
</script>


<style>
  html{
    background: #051c1c;
    font-family: 'M PLUS Rounded 1c', sans-serif;
    color: white;
  }

  body{
    width: 500px;
    max-width: 90vw;
    margin: auto;
  }

  .menu-container{
    display: flex;
    flex-wrap: nowrap;
    margin: 10px;
  }

  .menu-container span{
    margin-right: 10px;
    background: darkgrey;
    padding: 2.5px 10px;
    border-radius: 2.5px;
    transition: all .5s ease-in-out;
  }

  h2{
    color: #34c064;
  }

  .schedule-list-container{
    list-style-type: none;
    padding: 0;
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