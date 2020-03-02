<template>
	<div class="coordinates">
		<div class="uk-grid">
			<div class="uk-width-1-2">
				<div class="uk-grid">
					<label class="uk-width-1-2">
						Longitude
						<input disabled="disabled" type="number" v-model.number="model.longitude" class="uk-form-small uk-width-1-1">
					</label>
					<label class="uk-width-1-2">
						Latitude
						<input disabled="disabled" type="number" v-model.number="model.latitude" class="uk-form-small uk-width-1-1">
					</label>
				</div>
				<div class="uk-grid uk-margin-small-top" style="height:290px">
					<img v-if="apiReady" v-bind:src="imageUrl" class="uk-width-1-1" height="200">
					<p v-else class="uk-width-1-1" style="font-size:.7em;color:red;">Please enter valid coordinates to retrieve an image.</p>
				</div>
			</div>
			<div class="uk-width-1-2">
				<div class="uk-grid">
					<div class="uk-width-1-2">
						<label class="uk-width-1-1">
							Place Name
							<input type="text" v-model="term" class="uk-form-small uk-width-1-1">
						</label>
						<label class="uk-width-1-1">
							Country
							<select v-model="country" class="uk-form-small uk-width-1-1">
								<option value="ch">Switzerland</option>
								<option value="de">Germany</option>
								<option value="global">No selection</option>
							</select>
						</label>
					</div>
					<div class="uk-width-1-2">
						<a class="blok__full-btn uk-margin-small-top" @click="searchPlace">
							<i class="uk-icon-search uk-margin-small-right"/>
						</a>
					</div>
				</div>
				<ul class="uk-list uk-list-striped">
					<li v-for="(res, i) in results" @click="selectResult(i)" v-bind:key="res.id">
						<h4>{{res.text}}</h4>
						<p><small>{{res.place_name}}</small></p>
					</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>

const pluginName = 'coords';

export default {
	mixins: [window.Storyblok.plugin],
	data() {
		return {
			country: 'ch',
			apiReady: false,
			term: "",
			results: []
		}
	},
	methods: {
		initWith() {
			return {
				longitude: 0.0,
				latitude: 0.0,
				plugin: pluginName
			};
		},
		pluginCreated() {
			// eslint-disable-next-line
			console.log(pluginName + ' created');
		},
		showStaticImage() {
			var url = "https://api.mapbox.com/styles/v1/mapbox/satellite-streets-v11/static/";
			var marker = "pin-l-theatre";
			var color = "+00aa93";
			var zoom = "16";
			var size = "800x500@2x";
			var apiKey = this.options.apiKey ? this.options.apiKey : 'pk.eyJ1IjoibWFudWVsa2FtbWVybWFubiIsImEiOiJjazM1MGhhaWYwMDEyM2xuc25ubmpibzB5In0.iY7-5Qmd8VrC_VAGUM_5TQ';

			if(this.model.longitude & this.model.latitude & apiKey.length > 0){
				var end = "?access_token=" + apiKey.toString();
				var lat = this.model.latitude.toString();
				var lon = this.model.longitude.toString();

				this.apiReady = true;
				this.imageUrl = url+marker+color+"("+lon+","+lat+")/"+lon+","+lat+","+zoom+",0/"+size+end;
				this.$forceUpdate();
			}
			else {
				this.apiReady = false;
				this.$forceUpdate();
			}
		},
		async searchPlace() {
			var apiKey = this.options.apiKey;
			var endpoint = "https://api.mapbox.com/geocoding/v5/mapbox.places/";
			var term = encodeURI(this.term);
			var search = ".json?access_token=" + apiKey;

			var query = endpoint + term + search;

			if(this.country && this.country !== 'global') query += '&country=' + this.country

			const jsonResponse = await fetch(query);
			const response = await jsonResponse.json();

			this.results = response.features;
		},
		selectResult(index) {
			this.model.longitude = this.results[index].center[0];
			this.model.latitude = this.results[index].center[1];

			this.results = [];

			this.showStaticImage();
		}
	},
	watch: {
		'model': {
			handler: function (value) {
				this.$emit('changed-model', value);
			},
			deep: true
		}
	}
}
</script>
