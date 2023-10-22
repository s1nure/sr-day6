<template>
	<div class="main">
		<div class="form smoothly_show">
			<div class="region smoothly_show">
				<label for="region-select"
					><h2>Оберіть область вашого населеного пункту</h2>
				</label>
				<select v-model="selectedRegion" id="region-select">
					<option
						v-for="(region, index) in regions"
						:key="index"
						:value="region"
					>
						{{ region }}
					</option>
				</select>
			</div>
			<div class="city smoothly_show" v-if="selectedRegion">
				<label for="select-city"><h2>Оберіть населений пункт</h2></label>
				<select v-model="selectedCity" id="select-city">
					<option
						v-for="(cityWithRegion, index) in citiesWithRegions[selectedRegion]"
						:key="index"
						:value="cityWithRegion"
					>
						{{ cityWithRegion[0] }}
					</option>
				</select>
			</div>
      <h3 v-if="noneBranch" class="info">Нажаль віддлень нової пошти не знайдено у вашому населеному пункті</h3>
			<div class="branch smoothly_show" v-if="!noneBranch && selectedCity[0] && selectedCity[1]">
				<label for="select-brach"> <h2>Оберіть відділення</h2></label>
				<select v-model="selectedBranch" id="select-brach">
					<option
						v-for="(cityBranch, index) in ApiBranch.data"
						:key="index"
						:value="cityBranch"
					>
						{{ cityBranch.Description }}
					</option>
				</select>
			</div>
			<div v-if="this.selectedBranch.Description" class="info smoothly_show">
				<h3>Інформація про відділення</h3>
				<h4>Повна адреса: {{ this.selectedBranch.ShortAddress }}</h4>
				<h4>Номер телефону: +{{ this.selectedBranch.Phone }}</h4>
				<h4>Відділення працює:</h4>
				<table>
					<tr>
						<td>Понеділок</td>
						<td>{{ this.selectedBranch.Reception.Monday }}</td>
					</tr>
					<tr>
						<td>Вівторок</td>
						<td>{{ this.selectedBranch.Reception.Tuesday }}</td>
					</tr>
					<tr>
						<td>Середа</td>
						<td>{{ this.selectedBranch.Reception.Wednesday }}</td>
					</tr>
					<tr>
						<td>Четвер</td>
						<td>{{ this.selectedBranch.Reception.Thursday }}</td>
					</tr>
					<tr>
						<td>Пʼятниця</td>
						<td>{{ this.selectedBranch.Reception.Friday }}</td>
					</tr>
					<tr>
						<td>Субота</td>
						<td>{{ this.selectedBranch.Reception.Saturday }}</td>
					</tr>
					<tr>
						<td>Неділя</td>
						<td>{{ this.selectedBranch.Reception.Sunday }}</td>
					</tr>
				</table>
			</div>
		</div>
	</div>
	<h2>{{ regions }}</h2>
	<h2>{{ selectedRegion }}</h2>
	<h2>{{ selectedCity }}</h2>
	<h3>{{ ApiBranch }}</h3>
</template>

<script>
import axios from 'axios'
export default {
	data() {
		return {
			apiResponse: [],
			regions: [],
			citiesWithRegions: {},
			selectedRegion: '',
			selectedCity: [],
			ApiBranch: {},
			selectedBranch: {},
      noneBranch: false,
		}
	},
	methods: {
		getData() {
			axios
				.post('https://api.novaposhta.ua/v2.0/json/', {
					apiKey: '1c3af505b694bf591bb78d52c7017016',
					modelName: 'Address',
					calledMethod: 'getCities',
					methodProperties: {
						Ref: '',
					},
				})
				.then(response => {
					this.apiResponse = response.data
					const uniqueRegions = [
						...new Set(response.data.data.map(item => item.AreaDescription)),
					]
					this.regions = uniqueRegions.sort()

					const citiesAndRegions = {}
					response.data.data.forEach(item => {
						const region = item.AreaDescription
						const city = item.Description
						const cityRef = item.Ref
						if (!citiesAndRegions[region]) {
							citiesAndRegions[region] = []
						}

						citiesAndRegions[region].push([city, cityRef])
					})

					this.citiesWithRegions = citiesAndRegions
				})
		},
		getBranch(selectedCity) {
			axios
				.post('https://api.novaposhta.ua/v2.0/json/', {
					apiKey: '1c3af505b694bf591bb78d52c7017016',
					modelName: 'Address',
					calledMethod: 'getWarehouses',
					methodProperties: {
						CityName: selectedCity[0],
						CityRef: selectedCity[1],
					},
				})
				.then(response => {
					this.ApiBranch = response.data
          this.checkBranch()
				})
		},
		checkFields() {
			if (this.selectedRegion && this.selectedCity[0] && this.selectedCity[1]) {
				this.getBranch(this.selectedCity)
			}
		},
    isSelectedRegion (){
			this.selectedCity = [],
			this.selectedBranch = {}
    },
    isSelectedCity () {
      this.selectedBranch = {}
    },
    checkBranch() {
      if (!this.ApiBranch) {
        this.noneBranch = false
        return
      }
      if (!this.ApiBranch.info) {
        this.noneBranch = false
        return
      }
      if (this.ApiBranch.info.totalCount === 0) {
        this.noneBranch = true
      }
    }
	},
	watch: {
    selectedRegion: function () {
      this.checkFields()
      this.isSelectedRegion()
    },
    selectedCity: function () {
      this.checkFields()
      this.isSelectedCity()
    },
	},
	mounted() {
		this.getData()
	},
}
</script>

<style scoped>
@import url(../assets/main.css);
</style>
