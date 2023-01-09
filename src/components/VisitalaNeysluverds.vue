<template>
  <div>
    <h1>Vísitala Neysluverðs</h1>
    <div style="width: 100%; height: 30rem">
      <!-- show the loading icon if the data is not yet available -->
      <div v-if="!data.datasets.length">
        <ChartLoading />
      </div>
      <!-- show the line chart if the data is available -->
      <Line ref="lineChart" :data="data" v-if="data.datasets.length" />
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
import ChartLoading from "./ChartLoading.vue";

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
    ChartLoading,
    Line,
  },
  data() {
    return {
      data: {
        labels: [] as (number | "")[],
        datasets: [] as {
          label: string;
          backgroundColor: string;
          data: (number | { x: number; y: number } | null)[];
        }[],
        options: {
          responsive: true,
          maintainAspectRatio: false,
          tooltips: {
            interaction: {
              mode: "index",
            },
          },
        },
      },
    };
  },
  mounted() {
    async function getXmlData() {
      const response = await fetch(
        "https://proxy.cors.sh/https://www.sedlabanki.is/xmltimeseries/Default.aspx?DagsFra=1990-01-01&DagsTil=2022-12-23&TimeSeriesID=2&Type=xml",
        {
          headers: {
            // use APIKEY from .env
            "x-cors-api-key": import.meta.env.VITE_APIKEY,
          },
        }
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
        (node) => {
          const value = Number(node.textContent);
          if (isNaN(value)) {
            return null;
          }
          return value;
        }
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
