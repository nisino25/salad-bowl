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
    <div class="form-container">
        <form @submit.prevent="handleSubmit" class="simple-form">
            <div class="form-group">
                <label for="username">Username:</label>
                <input v-model="form.username" type="text" id="username" required>
            </div>
            <div class="form-group">
                <label for="date">Date (YYYYMMDD):</label>
                <input v-model="form.date" type="text" id="date" pattern="\d{8}" required>
            </div>
            <div class="form-group">
                <label for="startTime">Start Time (HHMM):</label>
                <input v-model="form.startTime" type="text" id="startTime" pattern="\d{4}" required>
            </div>
            <div class="form-group">
                <label for="endTime">End Time (HHMM):</label>
                <input v-model="form.endTime" type="text" id="endTime" pattern="\d{4}" required>
            </div>
            <button type="submit" class="submit-button">Submit</button>
        </form>
        <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
    </div>
  </template>

  <template v-if="currentPage == 'calendar'">
    ここにはカレンダーが追加されます<br>（すぐとは言っていない）
  </template>

  

</template>

<script>
  import AOS from 'aos'
  import 'aos/dist/aos.css'
  import { GoogleSpreadsheet } from 'google-spreadsheet';


  export default {
    data() {
      return {
        isFetchingData: false,
        sheetData: null,
        currentPage: null,

        selectedMonth: '',

        form: {
            username: '',
            date: '',
            startTime: '',
            endTime: ''
        },
        errorMessage: '',

        creds: {
          client_email: 'sheet-editor@salada-bowl.iam.gserviceaccount.com',
          private_key: '-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7GXdj/HoHH1uX\nJn6fbldqd1c1hmWEkRiM/zIdMUSTgL6U1Mz48WpeRQQhYI3HlWxFc5odSJiITBxB\n6YyA3xdTA3GdW8zFLfLZRvRZ0uLf8Itwp5b5Go5q1HiuuDZj1cpFBgRZ9mjFlCSI\nLxvpwMPCxXKIDGA1XcG8z1FAUXp1KIqVHiP0FkQS9sTKbUYUo8O1sve4BZVHTjGn\nQvf1lyL4xNPlJyIdKN50za4uixe1xP4FfSta8zcjdnXGBiwGWa0qgTOEjlsZq/LN\n0wBvUx68S85AM1amjIFOptUWWSeZv0IAg11nitycIaDziZnimApCGvEyEDP+Jg/c\nGDioiMVbAgMBAAECggEASvLnpbEE/QdtvD3aaWldMTP/RlzBG/q3t/ueip0q2F+x\neJNKTMsAjiTdg7VW9kWAKs4lRWfIWokKpMi21QUJJeLyR1P30mEWsD1BMx5MbeLB\nKO6phr5BoL/eXDdE6ndA4KeJZLRVwhgXDkq4xsnGYaaQu7khbR9StZzi8n3xLS7R\nqflnC78I/NK/0PEN16OXMlg3bru7RUP7B7iMQ0fpSftqs9FWyHgJaE/757IYTq3E\nponlPOX5aaRXxQqHWB6+lsBa0/Jc9/he0s4nNcQfyDj8RgrbHAoMNU0+S6Z6sUSv\nsm9nbNm9IV/CQIDd1HYaXHDuNLvg+PWygf8rGLLrEQKBgQDpR4wnQ9jD05O8R4s3\nHN3btNBrMv3kmMPE0VTW+knjWm+3poFmqXlTq4bCpD4jDNW38CKtOyR6799xZT8S\nPYqAQKayN7rEGZfhrUMzdc0nvf6KAiVpx9poyUTnIHZmskY50E1GcAE37iukmHhs\ningBZN0iYtPVUPNBmiaJdgrB4wKBgQDNUn52/0OCpdYSjF9OS6sy+0igmRY4MfdB\nknwbgMt+GwtV19lg+0TiTOpR5wQBppp2ur/IAHmEDDd4XZiN3e3+BkntWde0qRCE\n+vOJVzhPZRyX2XgEf40RXDVSSFW8tC/q1fjaZjuBrl8bqhwV5iSrD72LCp7F0TkN\nO/g+/Z7oKQKBgQDbzobJGKzPGDVEW0VaEOEbfCxGVi3Vj/wnH/eI+R1WFIjfywxy\n541iwWShUpEaBaX7Q1HpWKjvDcbE2lmrnkE6x7BKjSh2TodGJjQD8SP2JpgJAiyu\nl5m80qkR/wyRh7mUECpADJmZUdndpa0S2QZqidez5tsjTLtpPQ7Cx28rcQKBgEt6\nc/sSw6KXjCata6vArWLEdWJ0ZHKsC5UTYIRLyILHNleMTeEU/bGjTNBm+FYTBN14\nsV+4rPMZ+ppI7ffZCdBER5D1YhKLxALergBjC9RD+0rnKvOcYCNtnjxtUvdX1fWi\nSyUeR+nYTCZDVqfGPtyToL4oXU2jKDHxRcUCPxRhAoGASb1vd7q3YOza+Nulwpb2\nEJw2ebhziZEs4sFSkkfpoUu4wGY1ANQtqlaHVNAF4G0kB9Fiqo3zxtEv0j/l0Pwc\nYAF9mFHVr3W38ZW4872UUtXdHl0gYcQHI9ZB7zxQd0GN6sFhJ7YAzCXZ8+NWF9PJ\nU2U2xp7n/pFuXEOQLfG51Dg=\n-----END PRIVATE KEY-----\n',
        },
        
        
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

      async test() {
        try {
          // Create an instance of GoogleSpreadsheet and initialize it with your spreadsheet ID
          const doc = new GoogleSpreadsheet('1VVcGRFrpePsE-F9g-SS4LWj_Xzf_ymTki-eVaAFZxfk'); // Replace with your actual spreadsheet ID

          // Authenticate with the service account (assuming you have already loaded the credentials)
          await doc.useServiceAccountAuth(this.creds);

          // Load the document (no authentication required for public access)
          await doc.loadInfo();

          // Access the sheet by its title "予約情報"
          const sheet = doc.sheetsByTitle['予約情報'];

          // Write to the spreadsheet
          const rowData = { Column1: 'Value1', Column2: 'Value2' };
          await sheet.addRow(rowData);
          console.log('Data added to the spreadsheet successfully:', rowData);
        } catch (error) {
          console.error('Error: ', error);
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

      validateForm() {
          if (this.form.date.length !== 8 || !/^\d{8}$/.test(this.form.date)) {
              return "Date must be 8 digits in YYYYMMDD format.";
          }
          if (this.form.startTime.length !== 4 || !/^\d{4}$/.test(this.form.startTime)) {
              return "Start Time must be 4 digits in HHMM format.";
          }
          if (this.form.endTime.length !== 4 || !/^\d{4}$/.test(this.form.endTime)) {
              return "End Time must be 4 digits in HHMM format.";
          }
          return "";
      },
      handleSubmit() {
          this.errorMessage = this.validateForm();
          if (this.errorMessage) {
              console.error('Validation error:', this.errorMessage);
              return;
          }
          console.log('Form submitted:', this.form);
          // Handle the form submission, e.g., send data to an API
      },

      async appendRowToSheet(data) {
          this.isFetchingData = true;

          const sheetID = '1VVcGRFrpePsE-F9g-SS4LWj_Xzf_ymTki-eVaAFZxfk';
          const sheetName = '予約情報';
          const accessToken = 'YOUR_ACCESS_TOKEN'; // Obtain via OAuth 2.0

          const URL = `https://sheets.googleapis.com/v4/spreadsheets/${sheetID}/values/${sheetName}:append?valueInputOption=USER_ENTERED&insertDataOption=INSERT_ROWS`;

          const body = {
              values: [
                  [data.date, data.startTime, data.endTime, data.username]
              ]
          };

          try {
              const res = await fetch(URL, {
                  method: 'POST',
                  headers: {
                      'Authorization': `Bearer ${accessToken}`,
                      'Content-Type': 'application/json',
                  },
                  body: JSON.stringify(body)
              });

              if (!res.ok) throw new Error('Failed to append data');

              console.log('Row appended successfully');
          } catch (error) {
              console.error('Error appending data:', error);
          } finally {
              this.isFetchingData = false;
          }
      },




    },

    async mounted() {
        console.clear()
        AOS.init();


        // await this.test()

        this.currentPage = 'check'
        // this.currentPage = 'input'

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


  /* ----------------- */
  .form-container {
    max-width: 500px;
    margin: 0 auto;
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.form-group {
    margin-bottom: 20px;
    color: black;
}

label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
}

input {
    width: 100%;
    padding: 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-sizing: border-box;
}

.submit-button {
    background-color: #4CAF50;
    color: white;
    padding: 10px 20px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
}

.submit-button:hover {
    background-color: #45a049;
}

</style>