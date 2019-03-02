<template>
	<div class="route-builder">
		<div class="row">
			<div class="col-md-8">
				<div id="map"> </div>
			</div>
			<div class="col-md-4">
				<div v-if="selectedArea != null" class="area-info">
					<input type="text" class="form-control mb-2" v-model="selectedArea.title" placeholder="Title">
					<input type="text" class="form-control mb-2" v-model="selectedArea.radius" placeholder="Radius">
					<input type="text" class="form-control mb-2" v-model="selectedArea.default_navigation" placeholder="Default navigation">

				</div>
				<div v-else>
					<div class="alert alert-info text-center"> Select/create area </div>
				</div>
				<hr>
				<div class="areas">
					<ul class="list-group">
						<li @click="select(area)" v-for="(area, index) in areas" class="list-group-item">
							#{{ index }} - {{ area.title }}
						</li>
					</ul>
				</div>
			</div>
		</div>
	</div>
</template>

<script>
	export default {
		data() {
			return {
				index: 0,
				map: null,
				areas: [],
				markers: [],
				polyline: null,
				selectedArea: null,
			}
		},
		
		mounted() {
			this.loadMap();
			this.registerEvents();
			this.createPolyline();
		},
		
		methods: {
			select(area) {
				this.selectedArea = area;
			},

			createPolyline() {
				this.polyline = new google.maps.Polyline({
				    map: this.map,
				    strokeWeight: 1,
				    strokeOpacity: 2.0,
				    strokeColor: '#000000',
				});
			},

			registerEvents() {
				 google.maps.event.addListener(this.map, 'click', (event) => {
					this.addMarker(event.latLng, this.map);
				});
			},

			loadMap() {
				this.map = new google.maps.Map(document.getElementById('map'), {
		         	zoom: 16,
		         	center: {lat: 59.412472, lng: 10.485530},
		        });
			},

			addMarker(location, map) {
				let marker = new google.maps.Marker({
					map: map,
					position: location,
					label: this.areas.length.toString(),
				});
				this.markers.push(marker);
				this.polyline.getPath().push(location);
				this.createArea(marker);
			},

			createArea(marker) {
				this.areas.push({
					'radius': 10,
					'latitude' : '',
					'longitude': '',
					'default_navigation': '',
					'title': this.areas.length.toString(),
				});
			},
		}
	};
</script>

<style scoped>
	#map {
		height: 900px;
	}
</style>
