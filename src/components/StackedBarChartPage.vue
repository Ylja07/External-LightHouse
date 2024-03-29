<template>
  <v-container fluid>
    <v-layout row>
      <v-col>
        <div id="chart">
          <apexchart
            ref="metaChart"
            type="bar"
            height="250"
            :options="chartOptionsMeta"
            :series="meta"
          ></apexchart>
        </div>

        <div id="chart">
          <apexchart
            ref="cogChart"
            type="bar"
            height="250"
            :options="chartOptionsCog"
            :series="cog"
          ></apexchart>
        </div>
      </v-col>
    </v-layout>
    <v-row align="stretch" no-gutters>
      <v-col md="4">
        <v-card>
          <v-card-title>{{ $t(this.hoverTitle) }}</v-card-title>
          <v-card-text>
            {{ $t(uitleg) }} <br />
            <div
              v-if="
                hoverName !== 'Niet Gedetecteerd' &&
                hoverName !== 'Not detected' &&
                hoverName !== 'EMPTY' &&
                hoverName !== ''
              "
            >
              {{
                $t("PERSONAL.part1") +
                Math.round(personal[personalStart]).toFixed(1) +
                $t("PERSONAL.part2") +
                hoverName +
                $t("PERSONAL.part3") +
                Math.round(personal[personalMins]).toFixed(1) +
                $t("PERSONAL.part4") +
                hoverName +
                "."
              }}
            </div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col md="2">
        <v-card>
          <v-card-title>{{ $t("PERCENTAGES.cog_titel") }}</v-card-title>
          <v-card-text>
            <v-list flat>
              <v-list-item-group color="primary">
                <v-list-item v-for="(item, i) in c_perc" :key="i">
                  <v-list-item-content>
                    <v-list-item v-on:mouseover="translateHover(item.name)">
                      {{ item.name }} :
                      {{ Math.round(item.data * 100).toFixed(0) }}%
                    </v-list-item>
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col md="2">
        <v-card>
          <v-card-title>{{ $t("PERCENTAGES.meta_titel") }}</v-card-title>
          <v-card-text>
            <v-list flat>
              <v-list-item-group color="primary">
                <v-list-item v-for="(item, i) in m_perc" :key="i">
                  <v-list-item-content
                    v-on:mouseover="translateHover(item.name)"
                  >
                    {{ item.name }} :
                    {{ Math.round(item.data * 100).toFixed(0) }}%
                  </v-list-item-content>
                </v-list-item>
              </v-list-item-group>
            </v-list>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col md="4">
        <v-card>
          <v-card-title>{{
            $t("Voorbeeld_reflectievragen.titel")
          }}</v-card-title>
          <v-card-text
            >{{ $t("Voorbeeld_reflectievragen.value1") }} <br /><br />
            {{ $t("Voorbeeld_reflectievragen.value2") }} <br /><br />
            {{ $t("Voorbeeld_reflectievragen.value3") }} <br /><br />
            {{ $t("Voorbeeld_reflectievragen.value4") }}
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import VueApexCharts from "vue-apexcharts";

export default {
  name: "StackedBarChart",
  components: {
    apexchart: VueApexCharts,
  },
  methods: {
    // creates the name to look in the translation dictionary
    // since the data can be in dutch or english there must also be a
    // dutch and english name of the data in translation dictionary
    translateHover(parameter) {
      // `this` inside methods point to the Vue instance
      var temp = parameter.split(" ").join("").toUpperCase();
      this.uitleg = "EXPLANATIONS." + temp;
      this.hoverTitle = "EXP_TITLE." + temp;
      this.hoverName = parameter;
      this.personalMins = parameter + "Mins";
      this.personalStart = parameter + "Start";
    },

    // gets called in the mouse enter event from the charts to update the language of the chart
    updateCharts() {
      // they didnt leave us much choice
      // if you are reading this good luck trying to make it better haha

      // if its needed since it doesnt need to update every time a mouse enters the charts
      if (!(this.$serviceHandler.getLanguage() == this.langauge)) {
        switch (this.$serviceHandler.getLanguage()) {
          case "en": {
            this.meta = this.ENmeta;
            this.m_perc = this.ENm_perc;
            this.cog = this.ENcog;
            this.c_perc = this.ENc_perc;
            this.personal = this.ENpersonal;
            break;
          }
          case "nl": {
            this.meta = this.NLmeta;
            this.m_perc = this.NLm_perc;
            this.cog = this.NLcog;
            this.c_perc = this.NLc_perc;
            this.personal = this.NLpersonal;
            break;
          }
        }
      }
    },
  },

  data: () => ({
    langauge: null,
    uitleg: "EXPLANATIONS.EMPTY",
    hoverTitle: "EXP_TITLE.EMPTY",
    hoverName: "",
    personalMins: null,
    personalStart: null,
    meta: null,
    m_perc: null,
    cog: null,
    c_perc: null,
    personal: null,
  }),

  created() {
    // store all the various translations and on updated change the language and set the data to the language chosen
    let data = this.$serviceHandler.getData("NL");
    this.NLmeta = data.meta;
    this.NLm_perc = data.m_perc;
    this.NLcog = data.cog;
    this.NLc_perc = data.c_perc;
    this.NLpersonal = data.personal;

    data = this.$serviceHandler.getData("EN");
    this.ENmeta = data.meta;
    this.ENm_perc = data.m_perc;
    this.ENcog = data.cog;
    this.ENc_perc = data.c_perc;
    this.ENpersonal = data.personal;

    this.meta = this.NLmeta;
    this.m_perc = this.NLm_perc;
    this.cog = this.NLcog;
    this.c_perc = this.NLc_perc;
    this.personal = this.NLpersonal;
  },

  // both chart options are loaded into computed otherwise events cant be set with methods
  computed: {
    chartOptionsMeta: function () {
      return {
        chart: {
          events: {
            dataPointMouseEnter: (e, chart, opts) => {
              this.translateHover(opts.w.config.series[opts.seriesIndex].name);
              this.updateCharts();
            },
            dataPointMouseLeave: () => {
              this.translateHover("EMPTY");
            },
          },
          type: "bar",
          height: 250,
          stacked: true,
        },
        plotOptions: {
          bar: {
            horizontal: true,
          },
        },
        dataLabels: {
          enabled: false,
        },
        stroke: {
          width: 1,
          colors: ["#fff"],
        },
        title: {
          text: this.$t("PERCENTAGES.meta_titel"),
        },
        xaxis: {
          categories: [""],
          labels: {
            show: true,
          },
        },
        yaxis: {
          title: {
            text: undefined,
          },
          labels: {
            show: false,
            formatter: function (val) {
              return (Math.round(val * 100) / 100).toFixed(2) + " min";
            },
          },
        },
        tooltip: {
          x: {
            show: false,
          },
        },
        fill: {
          opacity: 1,
        },
        legend: {
          show: false,
          position: "top",
          horizontalAlign: "left",
          offsetX: 40,
        },
      };
    },
    chartOptionsCog: function () {
      return {
        chart: {
          events: {
            dataPointMouseEnter: (e, chart, opts) => {
              // you can call Vue methods now as "this" will point to the Vue instance when you use ES6 arrow function
              this.translateHover(opts.w.config.series[opts.seriesIndex].name);
              this.updateCharts();
            },
            mouseLeave: () => {
              this.translateHover("EMPTY");
            },
          },
          type: "bar",
          height: 250,
          stacked: true,
        },
        plotOptions: {
          bar: {
            horizontal: true,
          },
        },
        dataLabels: {
          enabled: false,
        },
        stroke: {
          width: 1,
          colors: ["#fff"],
        },
        title: {
          text: this.$t("PERCENTAGES.cog_titel"),
        },
        xaxis: {
          categories: [""],
          labels: {
            show: true,
          },
        },
        tooltip: {
          x: {
            show: false,
            // formatter: undefined,
          },
        },
        yaxis: {
          title: {
            text: undefined,
          },
          labels: {
            show: false,
            formatter: function (val) {
              return (Math.round(val * 100) / 100).toFixed(2) + " min";
            },
          },
        },
        fill: {
          opacity: 1,
        },
        legend: {
          show: false,
          position: "top",
          horizontalAlign: "left",
          offsetX: 40,
        },
      };
    },
  },
};
</script>
