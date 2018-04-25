<template>
  <div class="hello">
    <div id="container">
      <div id="destroute_datePicker">
        <section class="m-dp-day">
          <div>日</div>
          <div>一</div>
          <div>二</div>
          <div>三</div>
          <div>四</div>
          <div>五</div>
          <div>六</div>
        </section>
        <article class="g-dp-bd maxH" :class="[{ 'iphoneBTM': isIPhoneX}]">
          <section class="m-dp-date" v-for="(monthObj, index) in monthNumber">
            <p>{{monthObj.year}}年{{monthObj.month}}月</p>
            <div class="m-dp-d">
              <span class="" v-for="(frontDate, index) in monthObj.frontDates">{{frontDate.displayDay}}</span>
              <!-- <span class="usable hasPrice isVocRest selected">31<i>¥2628</i><i>起</i></span>                      -->
              <span class=" isVocRest" :class="[{ 'usable hasPrice':mainMonthObj.price!='' }]" v-for="mainMonthObj in monthObj.MainDates"
                @click="resultDate(monthObj.year,monthObj.month,mainMonthObj.realDay,mainMonthObj.price)">
                {{mainMonthObj.displayDay}}
                <i>{{mainMonthObj.price}}</i>
              </span>
            </div>
          </section>
        </article>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'priceDate',
    data() {
      return {
        isIPhoneX: false,
        monthNumber: [],
      }
    },
    created() {
      this.getDate();
    },
    mounted() {

    },
    watch: {
      // 如果路由有变化，会再次执行该方法
    },
    methods: {
      getDate(callBack) {
        var self = this;
        axios.get(
          'https://m.lvmama.com/nticket/router/rest.do?method=api.com.csa.wifi.order.getWifiTimePrice&version=1.0.0&goodsId=2586329&startDate=&firstChannel=TOUCH&secondChannel=LVMM'
        ).then((res) => {
          console.log(res.data.data);

          if (res.data.code == '1') {
            var priceTimeData = res.data.data;
            var monthNumber = self.getAllMonthNumber();
            var indexOfOuter = 0,
              outerLength = monthNumber.length;
            var indexOfPrice = 0;
            var priceTimeDataLength = priceTimeData.length;
            var nowDay = new Date().getDate();
            var getTodayFlag = 0;
            for (; indexOfOuter < outerLength; indexOfOuter++) { // displayMonth
              var indexOfWeek = monthNumber[indexOfOuter].week;
              //月份前空白日期
              monthNumber[indexOfOuter].frontDates = [];
              for (var i = 0; i < indexOfWeek; i++) {
                var data = {};
                data.displayDay = "";
                data.price = "";
                data.priceText = "";
                monthNumber[indexOfOuter].frontDates.push(data);
              }
              var innerLength = monthNumber[indexOfOuter].date;
              var indexOfInner = 0;
              monthNumber[indexOfOuter].MainDates = [];

              var loopFlag = false;
              for (; indexOfInner < innerLength; indexOfInner++) { // 每个月的天数
                var displayDay = indexOfInner + 1;


                indexOfWeek = (indexOfWeek + 1) % 7;
                /**************月份格式化**************/
                if (monthNumber[indexOfOuter].month < 10) {
                  var fill_name = monthNumber[indexOfOuter].year + '-0' + monthNumber[indexOfOuter].month;
                } else {
                  var fill_name = monthNumber[indexOfOuter].year + '-' + monthNumber[indexOfOuter].month;
                }
                if (displayDay < 10) {
                  fill_name = fill_name + '-0' + displayDay;
                } else {
                  fill_name = fill_name + '-' + displayDay;
                }
                fill_name = fill_name + " 00:00:00";
                /**************月份格式化**************/

                if (getTodayFlag == 2) {
                  displayDay = '后天';
                  getTodayFlag = getTodayFlag + 1;
                }
                if (getTodayFlag == 1) {
                  displayDay = '明天';
                  getTodayFlag = getTodayFlag + 1;
                }
                if (displayDay == nowDay && indexOfOuter == 0) {
                  displayDay = '今天';
                  getTodayFlag = getTodayFlag + 1;
                }


                if (indexOfPrice < priceTimeDataLength && fill_name != priceTimeData[indexOfPrice].specDate) {
                  var data = {};
                  data.displayDay = displayDay;
                  data.realDay = indexOfInner + 1;
                  data.price = "";
                  monthNumber[indexOfOuter].MainDates.push(data);
                } else if (indexOfPrice < priceTimeDataLength) {
                  var data = {};
                  data.displayDay = displayDay;
                  data.realDay = indexOfInner + 1; //实际值
                  data.price = "￥" + priceTimeData[indexOfPrice].price;
                  //                                        data.priceText="起";
                  monthNumber[indexOfOuter].MainDates.push(data);
                  indexOfPrice++;
                  loopFlag = true;
                }
              }

              if (loopFlag) {

                self.monthNumber.push(monthNumber[indexOfOuter]);
              }
            }

          }
        }).catch(function (err) {
          console.log(err);
        });

      },
      getAllMonthNumber() {
        var self = this;
        var displayMonth = 11; //不能大于12  从当前显示几个月

        var startdayInfo = {};

        //当天
        var today = new Date();

        //开始时间
        var startday = new Date();
        startday.setDate(1);
        startdayInfo.year = startday.getFullYear();
        startdayInfo.month = startday.getMonth(); //代表0,1,,,11
        startdayInfo.day = startday.getDate(); //代表0,1,,,11

        //今年和明年的月份天数信息
        var nowMonthNumber = self.getMonthNumber(today.getFullYear());
        var nextMonthNumber = self.getMonthNumber(today.getFullYear() + 1);

        //最终显示的month组合  只有两年的时间可用
        var monthNumber = [];
        if (startdayInfo.month + displayMonth > 12) { //跨年
          var first = nowMonthNumber.splice(startdayInfo.month, 12);
          var second = nextMonthNumber.splice(0, startdayInfo.month + displayMonth - 12);
          monthNumber = monthNumber.concat(first, second);
        } else { //不跨年的情况
          monthNumber = nowMonthNumber.splice(startdayInfo.month, displayMonth);
        }
        return monthNumber;
      },
      getMonthNumber(year) {
        var tmpMonthNumber = [];
        for (var i = 1; i < 13; i++) { //代表 1,2,,,12月
          var infoItem = {};
          infoItem.year = year;
          infoItem.month = i;
          infoItem.date = new Date(infoItem.year, i, 0).getDate();
          infoItem.week = new Date(infoItem.year, i - 1, 1).getDay();
          tmpMonthNumber.push(infoItem);
        }
        return tmpMonthNumber;
      },
      resultDate(year, month, realDay, price) {
        var self = this;
        if (price) {
          var selecteDate = year + '-' + (month >= 10 ? month : '0' + month) + '-' + (realDay >= 10 ? realDay : '0' +
            realDay);
          var selectWeekNo = new Date(selecteDate).getDay(),
            selectWeek;
          switch (selectWeekNo) {
            case 0:
              selectWeek = '周日';
              break;
            case 1:
              selectWeek = '周一';
              break;
            case 2:
              selectWeek = '周二';
              break;
            case 3:
              selectWeek = '周三';
              break;
            case 4:
              selectWeek = '周四';
              break;
            case 5:
              selectWeek = '周五';
              break;
            case 6:
              selectWeek = '周六';
              break;
          }
          if (self.type == 'wifi') {
            if (self.startDate != '') {
              window.sessionStorage.setItem('backDate_wifi', selecteDate);
              window.sessionStorage.setItem('backWeek_wifi', selectWeek);
            } else {
              window.sessionStorage.setItem('selecteDate_wifi', selecteDate);
              window.sessionStorage.setItem('selectWeek_wifi', selectWeek);
              window.sessionStorage.setItem('backDate_wifi', '请选择');
              window.sessionStorage.setItem('backWeek_wifi', '');
            }
          } else {
            window.sessionStorage.setItem('selecteDate_ph', selecteDate);
            window.sessionStorage.setItem('selectWeek_ph', selectWeek);
          }
          window.sessionStorage.removeItem('nowCoupon_wifi');
          history.back()
        }
      }

    }
  }

</script>
<style scoped>
  .g-dp-bd {
    position: absolute;
    top: 73px;
    bottom: 0;
    left: 0;
    right: 0;
    overflow-x: hidden;
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
    -o-overflow-scrolling: touch;
  }

  .iphoneBTM {
    bottom: 34px;
  }

  .m-dp-day {
    width: 100%;
    background-color: #fff;
    line-height: 28px;
    height: 28px;
    font-size: 14px;
    border-bottom: 1px solid #ddd;
    position: relative;
  }

  .m-dp-day>div {
    float: left;
    width: 14.2%;
    text-align: center;
  }

  .m-dp-day>div:first-child,
  .m-dp-day>div:last-child {
    color: #D30976;
  }

  .m-dp-date {
    background-color: #fff;
    overflow: hidden;
  }

  .m-dp-date:first-of-type {
    margin-top: -1px;
  }

  .m-dp-date p {
    line-height: 30px;
    text-align: center;
    color: #000;
    border-bottom: 1px solid #eee;
    background: #f8f8f8;
  }

  .m-dp-date>div>span {
    float: left;
    width: 14.2%;
    height: 60px;
    line-height: 60px;
    border-bottom: 1px solid #ddd;
    border-image: url(../assets/gray.gif) 2 stretch;
    border-width: 0 0 1px;
    text-align: center;
    color: #aaa;
    position: relative;
  }

  .m-dp-date>div>span.hasPrice {
    line-height: 22px !important;
    padding-top: 10px;
  }

  .m-dp-date>div>span.selected,
  .m-dp-date>div>span.selected>i,
  .m-dp-date>div>span.selected>i:after {
    background-color: #D30976;
    color: #fff !important;
  }

  .m-dp-date>div>span.highLight.usable {
    color: #D30976;
  }

  .m-dp-date>div>span.highLight.usable>i {
    color: #D30976;
  }

  .m-dp-date>div>span.usable {
    color: #000;
  }

  .m-dp-date>div>span.usable>i {
    font-size: 10px;
    display: block;
    color: #D30976;
  }

</style>
