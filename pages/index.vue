<template>
  <div class="container">
    <div class="row">
      <div class="col-lg-3">
        <div class="filter">
          <div class="title">
            Опции тарифа
          </div>
          <div class="content">
            <div class="form-group" v-for="item in tariff_options" :key="item.id">
              <input type="checkbox" :id="item.id" :value="item.value" v-model="tariff_model">
              <label :for="item.id">
                <div class="fake_checkbox"></div>
                <span>{{ item.title }}</span>
              </label>
            </div>
          </div>
        </div>

        <div class="filter">
          <div class="title">Авиакомпании</div>
          <div class="content">
            <div class="form-group" v-for="item in airlines" :key="item.value">
              <input type="checkbox" :id="item.value" :value="item.value" v-model="airlines_model" :disabled="item.value === 'all' && airlines_model[0] === 'all'">
              <label :for="item.value">
                <div class="fake_checkbox"></div>
                <span>{{ item.title }}</span>
              </label>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-9">
        <transition-group tag="div" name="flights">
          <div style="text-align: center" v-if="!flights.length" key="no_results">Нет результатов</div>
          <div class="flight" v-for="item in flights" :key="item.id">
            <div class="container-fluid">
              <div class="row justify-content-between">

                <div class="col-lg-2 carrier">
                  <img :src="`https://aviata.kz/static/airline-logos/80x80/${item.validating_carrier}.png`" :alt="item.itineraries[0][0].carrier_name + 'logo'">
                  <span>{{ item.itineraries[0][0].carrier_name }}</span>
                </div>

                <div class="col-lg-1 date_time">
                  <div class="date">
                    {{ getDate(item.itineraries[0][0].dep_date) }}
                  </div>
                  <div class="time">
                    {{ getTime(item.itineraries[0][0].dep_date) }}
                  </div>
                </div>

                <div class="col-lg-3 route">
                  <div class="line_desc">
                    <div class="destination">
                      {{ item.itineraries[0][0].segments[0].origin_code }}
                    </div>
                    <div class="time">
                      {{ getTravelTime(item.itineraries[0][0].traveltime) }}
                    </div>
                    <div class="destination">
                      {{ item.itineraries[0][0].segments[item.itineraries[0][0].segments.length - 1].dest_code }}
                    </div>
                  </div>

                  <div class="line">
                    <div class="circle"></div>
                    <div class="line"></div>
                    <div class="circle"></div>
                    <div class="line"></div>
                    <div class="circle"></div>
                  </div>

                  <div class="flight_stop" v-if="item.itineraries[0][0].segments.length > 1">
                    {{ `через ${item.itineraries[0][0].segments[0].dest}, 1 ч 50 м` }}
                  </div>
                </div>

                <div class="col-lg-1 date_time">
                  <div class="date">
                    {{ getDate(item.itineraries[0][0].arr_date) }}
                  </div>
                  <div class="time">
                    {{ getTime(item.itineraries[0][0].arr_date) }}
                  </div>
                </div>

                <div class="col-lg-4 price">
                  <div class="number">
                    {{ `${item.price} ${item.currency}` }}
                  </div>
                  <button>
                    Выбрать
                  </button>
                  <div class="description">
                    Цена за всех пассажиров
                  </div>
                  
                </div>
              </div>

              <div class="row align-items-center">
                <a class="col-lg-2 adds" href="#">
                  <span>
                    Детали перелета
                  </span>
                </a>

                <a class="col-lg-2 adds" href="#">
                  <span>
                    Условия тарифа
                  </span>
                </a>

                <div class="col-lg-2 refundable" >
                  <template v-if="!item.refundable">
                    <img src="/refundable.svg" alt="">
                    <span>невозвратный</span>
                  </template>
                </div>

                <div class="col-lg-2"></div>

                <div class="col-lg-4 price_adds">
                  <template v-if="!item.services['20KG']">
                    <span>Нет багажа</span>
                    <button>+ Добавить багаж</button>
                  </template>
                  <template v-else>
                    <span>{{ item.services['20KG'].description }}</span>
                  </template>
                </div>
              </div>
            </div>
          </div>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<script>
import results from "../config/results.js";

let airlines = [];
for(let key in results.airlines){
  airlines.push({ title: results.airlines[key], value: key })
}
airlines.unshift({ title: "Все", value: "all" });

let unfiltered_flights = results.flights;

export default {
  data: ()=>({
    tariff_options: [
      { title: "Только прямые", value: 0, id: "tariff_option1" },
      { title: "Только с багажом", value: 1, id: "tariff_option2" },
      { title: "Только возвратные", value: 2, id: "tariff_option3" }
    ],

    tariff_model: [],
    airlines_model: ["all"],
    unfiltered_flights, airlines,
    
    format: "YYYY/M/D kk:mm"
  }),

  computed: {
    flights(){
      let result = JSON.parse(JSON.stringify(this.unfiltered_flights));;

      if(this.airlines_model[0] !== "all"){
        result = result.filter(item => this.airlines_model.includes(item.validating_carrier))
      }

      if(this.tariff_model){
        if(this.tariff_model.includes(0)){
          result = result.filter(item => item.itineraries[0][0].segments.length === 1)
        }

        if(this.tariff_model.includes(1)){
          result = result.filter(item => item.services['20KG'])
        }

        if(this.tariff_model.includes(2)){
          result = result.filter(item => item.refundable)
        }
      }

      return result
    }
    
  },

  methods: {
    getDate(datetime){
      return this.$moment(datetime, this.format).format("D MMM, ddd");
    },

    getTime(datetime){
      return this.$moment(datetime, this.format).format("HH:mm");
    },

    getTravelTime(traveltime){
      return  this.$moment(traveltime, "X").format("H ч m м");;
    }
  },

  watch: {
    airlines_model: {
      deep: true,
      handler(newValue){
        if(newValue[0] === "all" && newValue.length > 1){
          this.airlines_model.splice(0, 1);
        }

        if(newValue[newValue.length - 1] === "all" && newValue.length > 1){
          this.airlines_model = ["all"]
        }

        if(newValue.length === 0){
          this.airlines_model = ["all"]
        }
      }
    }
  }
}
</script>

<style lang="scss">
  .flights-enter-active, .flights-leave-active {
    transition: .5s;
  }

  .flights-leave-active, .flights-leave-to {
    position: absolute !important;
    width: calc(100% - 30px);
    left: 15px;
  }

  .flights-enter, .flights-leave-to /* .fade-leave-active below version 2.1.8 */ {
    transform: translateX(-5%);
    opacity: 0;
  }

  .flights-move {
    transition: transform .5s;
  }

  .no_results {

  }

  .flight {
    padding-left: 43px;
    background: white;
    box-shadow: 0px 2px 4px rgba(0, 0, 0, 0.15);
    border-radius: 4px;

    min-height: 100px;
    margin-bottom: 12px;

    position: relative;
    // width: 100%;

    .container-fluid {
      .row {
        .price_adds {
          
          background: #F5F5F5;
          padding: 0 20px;
          padding-bottom: 20px;

          font-size: 12px;
          line-height: 16px;

          align-self: stretch;

          display: flex;
          justify-content: space-between;
          align-items: center;

          button {
            background: #EAF0FA;
            border-radius: 2px;

            padding: 3px 8px;

            border: none;
            outline: none;

            color: #5763B3;
            font-weight: 600;
          }
        }

        .refundable {
          display: flex;
          padding-bottom: 20px;
          
          img {
            width: 16px;
            height: 16px;
            margin-right: 7px;
          }

          span {
            font-size: 12px;
            line-height: 14px;

            color: #707276;
          }
        }

        .adds {
          padding-left: 5px;

          font-size: 12px;
          line-height: 16px;
          color: #7284E4;

          text-decoration: none !important;
          padding-bottom: 20px;

          span {
            padding-bottom: 1px;
            border-bottom: 1px dashed #7283e485
          }
        }

        .price {
          background: #F5F5F5;
          padding: 0 20px;
          padding-top: 12px;
          align-self: flex-start;

          display: flex;
          flex-direction: column;
          align-items: center;
          text-align: center;

          .number {
            font-family: Arial;
            font-size: 24px;
            line-height: 28px;
            text-align: center;
            color: #202123;

            margin-bottom: 13px;
          }

          button {
            background: #55BB06;
            border-radius: 4px;

            width: 100%;
            text-align: center;

            font-style: normal;
            font-weight: bold;
            font-size: 18px;
            line-height: 25px;

            padding: 8px 0;
            color: #FFFFFF;

            border: none;

            margin-bottom: 8px;

            outline: none;
          }

          .description {
            font-size: 12px;
            line-height: 16px;

            color: #707276;

            padding-bottom: 16px;
          }

        }

        .route {
          padding-top: 40px;
          padding-bottom: 50px;
          align-self: center;

          .flight_stop {
            font-style: normal;
            font-weight: normal;
            font-size: 12px;
            line-height: 16px;
            text-align: center;
            color: #FF9900;
          }

          .line_desc {
            display: flex;
            justify-content: space-between;
            align-items: center;

            .destination {
              font-size: 10px;
              font-style: normal;
              font-weight: 400;
              line-height: 12px;
              letter-spacing: 0em;
              text-align: center;

              color: #B9B9B9;
            }

            .time {
              font-style: normal;
              font-weight: normal;
              font-size: 12px;
              line-height: 18px;
              /* identical to box height, or 150% */

              text-align: center;

              /* Deep Dark */

              color: #202123;

              margin-bottom: 5px;
            }
          }

          .line {
            display: flex;
            width: 100%;
            align-items: center;

            .circle { 
              border: 1px solid #B9B9B9;

              min-height: 5px;
              min-width: 5px;
              height: 5px;
              width: 5px;
              max-height: 5px;
              max-width: 5px;

              border-radius: 2.5px;

              flex-basis: content;
            }

            .line {
              margin-top: 0.25px;
              border-top: 1px solid #B9B9B9;
            }
          }
        }

        .date_time {
          padding-top: 40px;
          padding-left: 4px;
          padding-right: 4px;

          .date {
            font-size: 12px;
            font-style: normal;
            font-weight: 400;
            line-height: 16px;
            letter-spacing: 0em;
            text-align: left;
          }

          .time {
            font-size: 24px;
            font-style: normal;
            font-weight: 600;
            line-height: 33px;
            letter-spacing: 0em;
            text-align: left;

          }
        }

        .carrier {
          padding-top: 53px;
          padding-right: 5px;
          padding-left: 5px;

          img {
            width: 20px;
            margin-right: 12px;
          }

          span {
            // font-family: Open Sans;
            font-size: 14px;
            font-style: normal;
            font-weight: 600;
            line-height: 19px;
            letter-spacing: 0em;
            text-align: left;

          }
        }
      }
    }
  }

  .container {
    padding-top: 50px;
  }

  .filter { 
    padding: 12px;
    
    background: #F5F5F5;
    border-radius: 4px;

    margin-bottom: 12px;

    .title {
      font-size: 14px;
      font-weight: 700;
      line-height: 20px;
      letter-spacing: 0em;
      text-align: left;

      padding-bottom: 12px;
    }

    .content {
      .form-group {
        position: relative;
        margin-bottom: 10px;

        input[type="checkbox"] {
          visibility: hidden;
          position: absolute;
        }

        label {
          //styleName: Text 12;
          font-size: 12px;
          font-weight: 400;
          line-height: 16px;
          letter-spacing: 0em;
          text-align: left;

          margin-bottom: 1px;

          display: flex;
          align-items: center;

          cursor: pointer;

          .fake_checkbox {
            background: #FFFFFF;
            border: 1px solid #B9B9B9;
            box-sizing: border-box;
            border-radius: 2px;

            height: 12px;
            width: 12px;

            margin-right: 12px;
          }
        }

        input:checked + label .fake_checkbox {
          background-image: url("/checkbox.svg");
          background-position: center;
          background-repeat: no-repeat;
          background-size: 8px 8px;
          background-color: #55BB06;
          border: 1px solid #55BB06;
        }
      }
    }
  }
</style>
