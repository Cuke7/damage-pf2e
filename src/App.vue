<template>
  <div class="h-screen flex flex-col">
    <h1 class="font-mono text-5xl text-red-800 mx-auto my-8">
      Calcultateur de dégâts PF2E
    </h1>

    <div class="flex w-1/2 mx-auto space-x-4">
      <div class="flex flex-1 flex-col">
        <div class="text-2xl text-black mt-8 mb-2">
          Description des attaques
        </div>
        <textarea type="text"
          class="p-4 border-red-800 rounded-md bg-transparent border-2 text-black text-xl font-mono h-full"
          v-model="attacksString"></textarea>
      </div>
      <div class="flex flex-1 flex-col">
        <div class="text-2xl text-black mt-8 mb-2">
          Récaptitulatif
        </div>
        <div class="text-black font-mono flex border-2 border-red-800 rounded-md">
          <div class="w-1/2 text-xl flex flex-col items-center">
            <div class="text-red-800 font-bold border-b-2 border-black mb-4">
              Bonus à l'attaque
            </div>
            <div v-for="attack of attacks">
              {{ attack.bonus }}
            </div>
          </div>
          <div class="w-1/2 text-xl flex flex-col items-center ">
            <div class="text-red-800 font-bold border-b-2 border-black mb-4">
              Dégâts
            </div>
            <div v-for="attack of attacks">
              {{ attack.damages }}
            </div>
          </div>
        </div>
      </div>
    </div>


    <div class="text-2xl text-black mx-auto mt-8">
      Dégâts moyens / tours
    </div>
    <div>
      <apexchart class="w-[700px] mx-auto" type="line" :options="options" :series="series"></apexchart>
    </div>

  </div>
</template>

<script lang="ts" setup>
import { ref, computed } from "vue";

const attacksString = ref("15 2d8 1d6\n10 1d10 6")

const attacks = computed(() => {
  let results = []
  if (attacksString.value.length > 0) {
    const attacks = attacksString.value.split("\n");
    for (const attack of attacks) {
      const bonus = attack.split(" ")[0];
      const damages = attack.split(" ").slice(1)
      results.push({
        bonus, damages
      })
    }
    return results
  } else {
    return []
  }
})


const series = computed(() => {
  let out = <any>[{
    data: [
    ]
  }]
  for (let CA = 25; CA < 36; CA++) {
    let total = 0
    for (const attack of attacks.value) {
      let P = (21 - (CA - Number(attack.bonus))) / 20
      if (CA - Number(attack.bonus) <= 0) {
        P = 1
      }
      if (CA - Number(attack.bonus) > 20) {
        P = 0
      }

      for (const damageSource of attack.damages) {
        // If it's a dice
        if (damageSource.split("d").length > 1) {
          total += Number(damageSource.split("d")[0]) * ((Number(damageSource.split("d")[1]) + 1) / 2) * P
        } else {
          total += Number(damageSource) * P
        }
      }
    }
    out[0].data.push({
      x: CA,
      y: total
    })
  }
  out[0].name = "Dégâts moyens / tours"
  return out
})

// [{
//   data: [{
//     x: 19,
//     y: 10
//   }, {
//     x: 20,
//     y: 19
//   },
//   {
//     x: 21,
//     y: 24
//   }],
// }]

const options = {
  legend: {
    show: false,
  },
  markers: {
    size: 2,
    colors: "black",
  },
  stroke: {
    width: 2,
    curve: "smooth",
  },
  tooltip: {
    enabled: true,
  },
  grid: {
    borderColor: 'black',
    yaxis: {
      lines: {
        show: true,
        color: "black"
      },
    },
    xaxis: {
      lines: {
        show: false,
        color: "black"
      },
    },
  },
  chart: {
    animations: {
      enabled: true,
    },
    toolbar: {
      show: false,
    },
  },
  dataLabels: {
    enabled: false,
  },
  xaxis: {
    title: {
      text: "CA adverse",
      style: {
        colors: "black",
        fontSize: "14px",
      },
    },
    axisBorder: {
      show: true,
      color: "black"
    },
    axisTicks: {
      show: true,
      color: "black"
    },
    labels: {
      show: true,
      style: {
        colors: "black",
        fontSize: "14px",
      },
    },
  },
  yaxis: {
    title: {
      text: "Dégâts",
      style: {
        colors: "black",
        fontSize: "14px",
      },
    },
    axisBorder: {
      show: true,
      color: "black",
      offsetX: 0,
      offsetY: 0,
    },
    axisTicks: {
      show: true,
      color: "black",
      width: 6,
      offsetX: 0,
      offsetY: 0,
    },
    labels: {
      style: {
        colors: "black",
        fontSize: "14px",
      },
      formatter: (value: any) => { return value.toFixed(1) },
    },
  },
}

</script>

<style>
body {
  background-image: url("./assets/background.jpg");
  background-repeat: repeat;
}

textarea {
  resize: none;
  outline: none;
}
</style>