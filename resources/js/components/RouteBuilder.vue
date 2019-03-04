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
				draggedMarker: null,
			}
		},
		
		mounted() {
			this.loadMap();
			this.setMapOptions();
			this.registerEvents();
			this.createPolyline();
		},

		methods: {
			setMapOptions() {
				this.map.setOptions({draggableCursor:'crosshair'});
			},

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
			
			onMarkerDrag(marker) {
				this.draggedMarker = marker;
				// drag start
			},

			updateMarkerLocation(marker, location) {
			   for (let i in this.markers) {
			     if (this.markers[i] == marker) {
			        this.markers[i].location = newLocation;
			        break;
			     }
			   }
			},

			onMarkerDragEnd(marker) {
				console.log(marker.latLng);
				this.polyline.getPath().clear();
				this.updateMarkerLocation(this.draggedMarker, marker.latLng);
				this.markers.forEach((marker) => {
					this.polyline.getPath().push(marker.position);
				});
				this.draggedMarker = null;
			},

			addMarker(location, map) {
				this.index++;
				let marker = new google.maps.Marker({
					map: map,
					draggable:true,
					position: location,
					label: this.index.toString(),
				});
				marker.addListener('drag', this.onMarkerDrag);
    			marker.addListener('dragend', this.onMarkerDragEnd);

				this.polyline.getPath().push(location);
				this.markers.push(marker);
				this.createArea(marker);
			},

			createArea(marker) {
				this.areas.push({
					'radius': 10,
					'latitude' : '',
					'longitude': '',
					'marker': marker,
					'default_navigation': '',
					'title': this.areas.length.toString(),
				});
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
