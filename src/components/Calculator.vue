<template>
  <div class="form">
    <form action="">
      <table>
        <tr>
          <td class="select-group">
            <div>
              <select v-model="selectedMake">
                <option value="">SELECT YOUR CAR</option>
                <option v-for="car in electricCars" :key="car.id" :value="car.make_slug">{{ car.make_slug }}</option>
              </select>
              <select v-model="selectedModel" :disabled="!selectedMake">
                <option value="">SELECT MODEL</option>
                <option v-for="model in filteredModels" :key="model.model_slug" :value="model.model_slug">{{ model.model_slug }}</option>
              </select>
              <select v-model="selectedVariant" :disabled="!selectedModel">
                <option value="">SELECT VARIANT</option>
                <option v-for="variant in filteredVariants" :key="variant.model_slug" :value="variant.model_slug">{{ variant.model_slug }}</option>
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
            </div>
          </td>
          <td class="costs">
            <!-- Car Efficiency -->
            <div>
              <label for="efficiency">Car Efficiency <span @mouseover="showTooltip('efficiency', $event)" @mouseleave="hideTooltip()">(<i>i</i>)</span></label>
              <input type="text" v-model="displayedEfficiency" @input="calculateCost">
            </div>
            <div v-if="tooltip === 'efficiency'" class="tooltip">
              <p>{{ tooltipContent.efficiency }}</p>
            </div>

            <!-- Cost per KWH -->
            <div>
              <label for="cost">Cost per KWH <span @mouseover="showTooltip('cost', $event)" @mouseleave="hideTooltip()">(<i>i</i>)</span></label>
              <input type="text" v-model="displayedCost" @input="calculateCost">
            </div>
            <div v-if="tooltip === 'cost'" class="tooltip">
              <p>{{ tooltipContent.cost }}</p>
            </div>
            <!-- Supercharger Use Percentage Slider -->
            <div>
              <label for="superchargerPercentage">Supercharger Use Percentage </label>
              <span>{{ superchargerPercentage }}%</span>
              <input type="range" v-model="superchargerPercentage" min="0" max="100" @input="calculateCost">
            </div>
            <!-- Supercharger Fee per KWH -->
            <div>
              <label for="superchargerFee">Supercharger Fee per KWH</label>
              <input type="text" v-model="superchargerFee" @input="calculateCost">
            </div>            
            <!-- Kms driven (per year) -->
            <div>
              <label for="distance">Kms driven (per year) </label>
              <input type="text" v-model="kmsDriven" @input="calculateCost" placeholder="Enter kilometers">
            </div>
        </td>
        </tr>
      </table>
    </form>
  </div>
  <div class="result" v-if="showResult">
    <p>Your energy cost for {{ kmsDriven }} kms is ${{ totalEnergyCost }}</p>
  </div>
</template>



<script setup>
import { ref, computed, watch } from 'vue'

// DATA
// CARS
const electricCars = ref([
  {
    "id": 1,
    "make_slug": "BYD",
    "models": [
      {
        "id": 1,
        "model_slug": "Atto-3",
        "variants": [
          {
            "id": 1,
            "model_slug": "atto-3",
            "range": "345",
            "consumption_kwh_100km": 16.00,
            "battery_capacity_kwh": 55.2,
            "price_from": "48011"
          }
        ]
      }
    ]
  },
  {
    "id": 2,
    "make_slug": "Hyundai",
    "models": [
      {
        "id": 1,
        "model_slug": "Ioniq-5",
        "variants": [
          {
            "id": 1,
            "model_slug": "ioniq-5",
            "range": "451",
            "consumption_kwh_100km": 17.90,
            "battery_capacity_kwh": 80.729,
            "price_from": "72000"
          }
        ]
      },
      {
        "id": 2,
        "model_slug": "Ioniq-6",
        "variants": [
          {
            "id": 1,
            "model_slug": "ioniq-6",
            "range": "614",
            "consumption_kwh_100km": 14.30,
            "battery_capacity_kwh": 87.802,
            "price_from": "74000"
          }
        ]
      }
    ]
  },
  {
    "id": 3,
    "make_slug": "Kia",
    "models": [
      {
        "id": 1,
        "model_slug": "EV6",
        "variants": [
          {
            "id": 1,
            "model_slug": "ev6",
            "range": "528",
            "consumption_kwh_100km": 16.50,
            "battery_capacity_kwh": 87.12,
            "price_from": "72590"
          }
        ]
      },
      {
        "id": 2,
        "model_slug": "Niro",
        "variants": [
          {
            "id": 1,
            "model_slug": "niro",
            "range": "460",
            "consumption_kwh_100km": 16.20,
            "battery_capacity_kwh": 74.52,
            "price_from": "66590"
          }
        ]
      },
      {
        "id": 3,
        "model_slug": "Sorento",
        "variants": [
          {
            "id": 1,
            "model_slug": "sorento",
            "range": "68",
            "consumption_kwh_100km": 16.10,
            "battery_capacity_kwh": 10.948,
            "price_from": "81080"
          }
        ]
      }
    ]
  }
]);

// STATES
const states = ref([
  'Australian Capital Territory',
  'New South Wales',
  'Northern Territory',
  'Queensland',
  'South Australia',
  'Tasmania',
  'Victoria',
  'Western Australia',
]);

// STATE INFORMATION
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

// VARIABLES
const selectedMake = ref('');
const selectedModel = ref('');
const selectedVariant = ref('');
const selectedState = ref('');
const chargingMethod = ref('');
const kmsDriven = ref('');
const totalEnergyCost = ref('');
const showResult = ref(false);
const userCostPerKWH = ref('');
const userCarEfficiency = ref('');
const tooltip = ref('');
const superchargerFee = ref('');
const superchargerPercentage = ref(0);

// UI-RELATED
// FILTERING THE OPTIONS BASED ON THE PREVIOUS SELECTION
// Computed property to filter models based on selected make
const filteredModels = computed(() => {
  const selectedCar = electricCars.value.find(car => car.make_slug === selectedMake.value);
  return selectedCar ? selectedCar.models : [];
});

// Computed property to filter variants based on selected model
const filteredVariants = computed(() => {
  const selectedModelData = filteredModels.value.find(model => model.model_slug === selectedModel.value);
  return selectedModelData ? selectedModelData.variants : [];
});

// Computed property to display the cost based on selected state and charging method
const displayedCost = computed({
  get: () => {
    const stateInfo = getStateInformation();

    if (userCostPerKWH.value !== '') {
      return parseFloat(userCostPerKWH.value);
    } else if (stateInfo && chargingMethod.value) {
      return chargingMethod.value === 'solar' ? stateInfo.minimumSolarTariff : stateInfo.averageGridEnergyCost;
    }

    return '';
  },
  set: (value) => {
    userCostPerKWH.value = value;
  }
});

const displayedEfficiency = computed({
  get: () => {
    if (userCarEfficiency.value !== '') {
      return parseFloat(userCarEfficiency.value);
    }

    const selectedCar = electricCars.value.find(car => car.make_slug === selectedMake.value);
    const selectedModelData = selectedCar ? selectedCar.models.find(model => model.model_slug === selectedModel.value) : null;
    const selectedVariantData = selectedModelData ? selectedModelData.variants.find(variant => variant.model_slug === selectedVariant.value) : null;

    return selectedVariantData ? selectedVariantData.consumption_kwh_100km : '';
  },
  set: (value) => {
    userCarEfficiency.value = value;
  }
});

// Tooltip information
const tooltipContent = {
  efficiency: "kWh/100km", 
  cost: "depends on your state and charging method", 
}

const showTooltip = (label) => {
  tooltip.value = label; 
}

const hideTooltip = () => {
  tooltip.value = '';
  const tooltipElement = document.querySelector('.tooltip');
  if (tooltipElement) {
    tooltipElement.style.top = '0';
    tooltipElement.classList.remove('show');
  }
};

// ACCESSING THE DATA FOR THE CALCULATION
const getStateInformation = () => {
  // Get state-specific information for electricity cost calculation
  const stateInfo = stateInformation[selectedState.value];
  return stateInfo || null;
};

// Computed property to get consumption_kwh_100km of the selected variant
const selectedConsumption = computed(() => {
  const selectedCar = electricCars.value.find(car => car.make_slug === selectedMake.value);
  const selectedModelData = selectedCar.models.find(model => model.model_slug === selectedModel.value);
  const selectedVariantData = selectedModelData.variants.find(variant => variant.model_slug === selectedVariant.value);

  return selectedVariantData ? selectedVariantData.consumption_kwh_100km : null;
});

// Function to calculate total energy cost
// Function to calculate total energy cost
const calculateTotalEnergyCost = () => {
  const stateInfo = getStateInformation();

  if (stateInfo && chargingMethod.value && displayedEfficiency.value) {
    const energyCost = chargingMethod.value === 'solar'
      ? stateInfo.minimumSolarTariff
      : displayedCost.value;

    const consumption_kwh_100km = displayedEfficiency.value;
    const distance = parseFloat(kmsDriven.value);

    if (!isNaN(distance) && consumption_kwh_100km !== 0) {
      const homeChargePercentage = 100 - superchargerPercentage.value;
      const homeChargeCost = (homeChargePercentage / 100) * distance * energyCost;
      const superchargerChargeCost = (superchargerPercentage.value / 100) * distance * superchargerFee.value;

      const cost = homeChargeCost + superchargerChargeCost;
      return isNaN(cost) ? null : cost.toFixed(2);
    }
  }

  return null;
};


// Event handler for calculating the total energy cost
const calculateCost = () => {
  totalEnergyCost.value = calculateTotalEnergyCost();
  if (totalEnergyCost.value) {
    showResult.value = true;
  }
};

// Add a watcher to reset result visibility when any form input changes
watch([selectedMake, selectedModel, selectedVariant, selectedState, chargingMethod, kmsDriven, superchargerFee, superchargerPercentage], () => {
  showResult.value = false;
});


</script>

<style>
.form {
  max-width: 60em;
  margin: 0;
  background-color: #fffefe;
  border: 1px solid #ccc;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  margin-bottom: 2em;
  margin-top: 1em;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.select-group {
  flex: 1;
  padding: 2em;
  border: 2px solid #087c7c;
  border-radius: 8px;
  margin: 0;
  width: 50%;
  box-shadow:5px 0px 10px rgb(113, 114, 114);
}

.select-group div {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.costs {
  flex: 1;
  padding: 1em 0 1em 2em;
}

.select-group,
.input-group {
  margin-bottom: 1em;
}

select,
input {
  width: 80%;
  padding: 0.5em;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin: 0.3em 0;
}

select, .costs label{
  font-size: 0.85em;
}

button {
  background-color: #087c7c;
  color: #fff;
  padding: 0.5em 1em;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 2em;
}

button:hover {
  background-color: #47a9a9;
}

.result {
  border: 1px solid #ccc;
  border-radius: 8px;
  padding: 1.5em 1em 0em 1em;
  background-color: #087c7c;
  color: #ffffff;
  text-align: center;
  width:99%;
  margin: 0.3em;
}
.tooltip {
  background:#087c7c;
  color: #ffffff;
  position: absolute;
  z-index: 1;
  font-size: 0.7em;
  border-radius: 12px;
  padding: 1em;
  max-height: 3em;
  border: 1px solid #ccc;
}
</style>

