<template>
  <div>
    <h1>Vísitala Neysluverðs</h1>
    <div style="width: 100%; height: 30rem">
      <Line ref="lineChart" :data="data" :options="options" />
    </div>
  </div>
</template>

<script lang="ts">
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
} from "chart.js";
import { Line } from "vue-chartjs";
import GhostIcon from "../assets/ghost.svg";

ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
);

export default {
  name: "App",
  components: {
    Line,
  },
  data() {
    return {
      data: {
        labels: [],
        datasets: [
          {
            label: "Visitala neysluverðs",
            backgroundColor: "rgba(255, 99, 132, 0.2)",
            data: [],
          },
        ],
      },
    };
  },
  mounted() {
    async function getXmlData() {
      const response = await fetch(
        "https://cors-anywhere.herokuapp.com/https://www.sedlabanki.is/xmltimeseries/Default.aspx?DagsFra=1990-01-01&DagsTil=2022-12-23&TimeSeriesID=2&Type=xml"
      );
      const xmlString = await response.text();
      const parser = new DOMParser();
      const xmlData = parser.parseFromString(xmlString, "text/xml");
      return xmlData;
    }

    async function getChartData() {
      const xmlData = await getXmlData();

      const seenYears = new Set();
      const labels = Array.from(xmlData.querySelectorAll("Date")).map(
        (node) => {
          const date = new Date(node.textContent?.toString() ?? "");
          const year = date.getFullYear();
          if (year % 5 === 0 && !seenYears.has(year)) {
            seenYears.add(year);
            return year;
          }
          return "";
        }
      );
      const values = Array.from(xmlData.querySelectorAll("Value")).map(
        (node) => node.textContent
      );

      return {
        labels,
        datasets: [
          {
            label: "Visitala neysluverðs",
            backgroundColor: "#f87979",
            data: values,
            pointStyle: GhostIcon,
          },
        ],
        options: {
          responsive: true,
          maintainAspectRatio: false,
          tooltips: {
            interaction: {
              mode: "index",
            },
          },
        },
      };
    }

    getChartData().then((chartData) => {
      this.data = chartData;
    });
  },
};
</script>
