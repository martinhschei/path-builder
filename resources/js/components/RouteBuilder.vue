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
					<div class="alert alert-info text-center"> Select / create area </div>
				</div>
				<hr>
				<div class="areas">
					<ul class="list-group">
						<li v-for="(area, index) in areas" class="hand list-group-item">
							<span @click="select(area)"> #{{ index }} - {{ area.title }} </span> <i @click.prevent="remove(area, index)" class="fa fa-times-circle float-right red pt-1" style="font-size:1.2em"> </i>
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
			remove(marker) {
				for (let i = 0; i < this.markers.length; i++) {
					if (this.markers[i] == marker) {
						this.markers[i].setMap(null);
						this.markers.splice(i, 1);
						this.polyline.getPath().removeAt(i);
						for (let j = 0; j < this.areas.length; j++) {
							if (this.areas[i].marker == marker) {
								this.areas.splice(i, 1);
							}
						}
					}
				}
			},

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
				this.index++;
				let marker = new google.maps.Marker({
					map: map,
					position: location,
					label: this.index.toString(),
				});

				google.maps.event.addListener(marker, 'click', () => {
					this.remove(marker);
				});

				this.polyline.getPath().push(location);
				this.createArea(marker);
			},

			createArea(marker) {
				this.areas.push({
					'radius': 10,
					'latitude' : '',
					'longitude': '',
					'marker': marker,
					'default_navigation': '',
					'url': 'http://www.vg.no',
					'title': this.areas.length.toString(),
				});

				this.markers.push(marker);
			},
		}
	};
</script>

<style scoped>
	.red {
		color: red;
	}

	#map {
		height: 900px;
	}

	.hand {
		cursor: pointer,
	}
</style>
