<template>
  <div class="bg">
    <div class="form">
        <form action="">
            <select v-model="selectedMake">
                <option  value="">SELECT YOUR CAR</option>
                <option v-for="car in electricCars" :key="car.id" :value="car.make">{{ car.make }}</option>
            </select>
            <select v-model="selectedModel" :disabled="!selectedMake">
                <option value="">SELECT MODEL</option>
                <option v-for="model in filteredModels" :key="model.id" :value="model.model">{{ model.model }}</option>
            </select>
            <select v-model="selectedVariant" :disabled="!selectedModel">
                <option value="">SELECT VARIANT</option>
                <option v-for="variant in filteredVariants" :key="variant.id" :value="variant.variant">{{ variant.variant }}</option>
            </select>
            <select v-model="selectedState">
                <option value="">SELECT YOUR STATE</option>
                <option v-for="state in states" :key="state" :value="state">{{ state }}</option>
            </select>
            <select v-model="chargingMethod">
                <option value="">Charging Method</option>
                <option value="solar">Solar</option>
                <option value="power">Power Grid</option>
            </select>
            <div>
                <label for="cost">Cost per KWH</label>
                <input type="text" v-model="displayedCost">
            </div>
            <div>
                <label for="distance">Kms driven (per year)</label>
                <input type="text" v-model="kmsDriven" placeholder="Enter kilometers">
            </div>
            <button @click.prevent="calculateCost">Calculate the Cost</button>
        </form>
    </div>
    <div class="result" v-if="showResult">
      <p>Your energy cost for {{ kmsDriven }} kms is ${{ totalEnergyCost }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
// VARIABLES
const selectedMake = ref('');
const selectedModel = ref('');
const selectedVariant = ref('');
const selectedState = ref('');
const chargingMethod = ref('');
const kmsDriven = ref('');
const totalEnergyCost = ref('');
const showResult = ref(false);


//UI-RELATED 
//FILTERING THE OPTIONS BASED ON THE PREVIOUS SELECTION
// Computed property to filter models based on selected make
const filteredModels = computed(() => {
  const selectedCar = electricCars.value.find(car => car.make === selectedMake.value);
  return selectedCar ? selectedCar.models : [];
});

// Computed property to filter variants based on selected model
const filteredVariants = computed(() => {
  const selectedModelData = filteredModels.value.find(model => model.model === selectedModel.value);
  return selectedModelData ? selectedModelData.variants : [];
});



//ACCESSING THE DATA FOR THE CALCULATION
const getStateInformation = () => {
  // Get state-specific information for electricity cost calculation
  const stateInfo = stateInformation[selectedState.value];
  return stateInfo || null;
};

// Computed property to display the cost based on selected state and charging method
const displayedCost = computed(() => {
  const stateInfo = getStateInformation();

  if (stateInfo && chargingMethod.value) {
    return chargingMethod.value === 'solar' ? stateInfo.minimumSolarTariff : stateInfo.averageGridEnergyCost;
  }
  return '';
});

// Computed property to get kmPerKWh of the selected variant
const selectedKmPerKWh = computed(() => {
  const selectedCar = electricCars.value.find(car => car.make === selectedMake.value);
  const selectedModelData = selectedCar.models.find(model => model.model === selectedModel.value);
  const selectedVariantData = selectedModelData.variants.find(variant => variant.variant === selectedVariant.value);

  return selectedVariantData ? selectedVariantData.kmPerKWh : null;
});

// Function to calculate total energy cost
const calculateTotalEnergyCost = () => {
  const stateInfo = getStateInformation();

  if (stateInfo && chargingMethod.value && selectedKmPerKWh.value) {
    const energyCost = chargingMethod.value === 'solar'
      ? stateInfo.minimumSolarTariff
      : stateInfo.averageGridEnergyCost;

    const kmPerKWh = selectedKmPerKWh.value;
    const distance = parseFloat(kmsDriven.value);

    if (!isNaN(distance) && kmPerKWh !== 0) {
      const cost = distance * (1 / kmPerKWh) * energyCost;
      return isNaN(cost) ? null : cost.toFixed(2);
    }
  }

  return null;
};

// Event handler for calculating the total energy cost
const calculateCost = () => {
  totalEnergyCost.value = calculateTotalEnergyCost();
  if (totalEnergyCost.value){
    showResult.value = true; 
  }
};

// Add a watcher to reset result visibility when any form input changes
watch([selectedMake, selectedModel, selectedVariant, selectedState, chargingMethod, kmsDriven], () => {
  showResult.value = false;
});

//DATA
//CARS
const electricCars = ref([
  {
    "id": 1,
    "make": "Tesla",
    "models": [
      {
        "id": 1,
        "model": "Model S",
        "variants": [
          {
            "id": 1,
            "variant": "Long Range",
            "kmPerKWh": 3.7
          },
          {
            "id": 2,
            "variant": "Plaid",
            "kmPerKWh": 3.3
          }
        ]
      },
      {
        "id": 2,
        "model": "Model 3",
        "variants": [
          {
            "id": 3,
            "variant": "Standard Range Plus",
            "kmPerKWh": 4.87
          },
          {
            "id": 4,
            "variant": "Performance",
            "kmPerKWh": 4.2
          }
        ]
      }
    ]
  },
  {
    "id": 2,
    "make": "Nissan",
    "models": [
      {
        "id": 5,
        "model": "Leaf",
        "variants": [
          {
            "id": 6,
            "variant": "SV Plus",
            "kmPerKWh": 3.65
          },
          {
            "id": 7,
            "variant": "S",
            "kmPerKWh": 3.725
          }
        ]
      }
    ]
  },
  {
    "id": 3,
    "make": "Chevrolet",
    "models": [
      {
        "id": 8,
        "model": "Bolt EV",
        "variants": [
          {
            "id": 9,
            "variant": "Premier",
            "kmPerKWh": 3.924
          }
        ]
      }
    ]
  }
]);

//STATES
const states = ref([
        'Australian Capital Territory',
        'New South Wales',
        'Northern Territory',
        'Queensland',
        'South Australia',
        'Tasmania',
        'Victoria',
        'Western Australia',
      ])


//STATE INFORMATION
const stateInformation = {
  'Australian Capital Territory': { averageGridEnergyCost: 0.29, minimumSolarTariff: 0.06 },
  'New South Wales': { averageGridEnergyCost: 0.27, minimumSolarTariff: 0.056 },
  'Northern Territory': { averageGridEnergyCost: 0.27, minimumSolarTariff: 0.083 },
  'Queensland': { averageGridEnergyCost: 0.26, minimumSolarTariff: 0.05 },
  'South Australia': { averageGridEnergyCost: 0.36, minimumSolarTariff: 0.05 },
  'Tasmania': { averageGridEnergyCost: 0.27, minimumSolarTariff: 0.089 },
  'Victoria': { averageGridEnergyCost: 0.28, minimumSolarTariff: 0.05 },
  'Western Australia': { averageGridEnergyCost: 0.3, minimumSolarTariff: 0.025 },
};

</script>

<style>
.bg {
  max-width: 30em;
  margin: auto;
  position: absolute;
  top: 50%;
  left: 75%;
  transform: translate(-50%, -50%);
}
.form {
  max-width: 30em;
  margin: 0;
  background-color: #fffefe;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  padding: 1em;
}

form {
  display: flex;
  flex-direction: column;
  gap: 1em;
}

.select-group,
.input-group {
  margin-bottom: 1em;
}

select,
input {
  width: 100%;
  padding: 0.5em;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  background-color: #087c7c;
  color: #fff;
  padding: 0.5em 1em;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #47a9a9;
}


.result {
  max-width: 30em;
  margin: 1em 0;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  padding-top: 1.5em;
  background-color: #ffffff;
  color: #087c7c;
  text-align: center;
}

</style>