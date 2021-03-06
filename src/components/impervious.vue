<template>
    <div>
        <div class="text-center">
            <div class="report-record-highlight">
                <svg class="icon icon-impervious"><use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-impervious"></use></svg>
                <h2>You have</h2>
                <h1 v-if="results">{{ total() | area }}</h1>
                <h3>of Impervious Surface</h3>
                <h4></h4>
            </div>
            <table v-if="results && results.length > 0">
                <caption>Details</caption>
                <thead>
                    <tr>
                        <th>Type</th>
                        <th class="text-right">Area</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(item, index) in results">
                        <td>
                            {{item.subtheme}}
                        </td>
                        <td class="nowrap text-right">
                            {{ item.area | area }}
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
        <div class="report-moreinfo">
            <p>Impervious surfaces are mainly artificial structures, such as pavements (roads, sidewalks, driveways and parking lots) and rooftops that are covered by impenetrable materials such as asphalt, concrete, brick, wood and stone.</p>
            <h5>For more information, please visit:</h5>
            <ul class="list-unstyled">
                <li><a href="http://charmeck.org/stormwater/FeesandBilling/Pages/Default.aspx" target="_blank"  rel="noopener">Charlotte-Mecklenburg Storm Water Services</a></li>
            </ul>
        </div>
    </div>
</template>

<script>
import jsonToURL from "../js/jsontourl";
import format from "format-number";

export default {
  name: "impervious",

  data: function() {
    return {
      results: null
    };
  },

  computed: {
    selected () {
      return this.$store.getters.selected
    },
    show () {
      return this.$store.getters.show
    }
  },

  watch: {
    selected: "getResults",
    show: "getResults"
  },

  filters: {
    area: function(num) {
      return format({
        truncate: 0,
        suffix: " Sq. Ft."
      })(num);
    }
  },

  mounted: function() {
    this.getResults();
  },

  methods: {
    getResults: function() {
      let _this = this;
      if (
        _this.selected.lnglat &&
        _this.show.indexOf("impervious") !== -1
      ) {
        let params = {
          columns: "sum(sum_of_area) as area, subtheme",
          filter: `commonpid='${_this.selected.pid}'`,
          sort: "subtheme",
          group: "subtheme"
        };
        fetch(
          `https://mcmap.org/api/query/v1/impervious_surface_area?${jsonToURL(
            params
          )}`
        )
          .then(function(response) {
            return response.json();
          })
          .then(function(data) {
            _this.results = data;
          })
          .catch(function(ex) {
            console.log("parsing failed", ex);
          });
      }
    },
    total: function() {
      let total = 0;
      this.results.forEach(function(item) {
        total = total + Number(item.area);
      });
      return total;
    }
  }
};
</script>