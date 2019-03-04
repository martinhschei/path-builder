<template>
	<div class="route-builder">
		<div class="row">
			<div class="col-md-8">
				<div id="map"> </div>
			</div>
			<div class="col-md-4">
				<div v-if="selectedWaypoint != null" class="createWaypoint-info">
					<input type="text" class="form-control mb-2" v-model="selectedWaypoint.title" placeholder="Title">
					<input type="text" class="form-control mb-2" v-model="selectedWaypoint.radius" placeholder="Radius">
					<input type="text" class="form-control mb-2" v-model="selectedWaypoint.video_url" placeholder="Video url">
					<input type="text" class="form-control mb-2" v-model="selectedWaypoint.default_navigation" placeholder="Default navigation">
					<input type="text" class="form-control mb-2" v-model="selectedWaypoint"
				</div>
				<div v-else>
					<div class="alert alert-info text-center"> Select / create waypoint </div>
				</div>
				<hr>
				<div class="waypoints">
					<ul class="list-group">
						<li v-for="(waypoint, index) in waypoints" class="hand list-group-item mb-2">
							<span style="font-size:1.2em" @click="select(waypoint)"> #{{ index }} - {{ waypoint.title }} </span> <i @click.prevent="remove(waypoint, index)" class="fa fa-times-circle float-right red pt-1 hand" style="font-size:1.2em"> </i>
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
				map: null,
				markers: [],
				waypoints: [],
				polyline: null,
				draggedMarker: null,
				selectedMarker: null,
				selectedWaypoint: null,
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

			remove(waypoint, index) {
				for (let i = 0; i < this.markers.length; i++) {
					if (this.markers[i] == waypoint.marker) {
						this.markers[i].setMap(null);
						this.markers.splice(i, 1);
						this.polyline.getPath().removeAt(i);
						for (let j = 0; j < this.waypoints.length; j++) {
							if (this.waypoints[i].marker == waypoint.marker) {
								this.waypoints.splice(i, 1);
								break;
							}
						}
						break;
					}
				}
				this.selectedWaypoint = null;
				// TODO: recalc bearings and distances
			},

			select(waypoint) {
				this.selectedWaypoint = waypoint;
			},
				
			createPolyline() {
				this.polyline = new google.maps.Polyline({
				    map: this.map,
				    strokeWeight: 3,
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
			},

			onMarkerDragEnd(marker) {
				this.polyline.getPath().clear();
				this.updateMarkerLocation(this.draggedMarker, marker.latLng);
				this.markers.forEach((marker) => {
					this.polyline.getPath().push(marker.position);
				});
				this.draggedMarker = null;
			},

			updateMarkerLocation(marker, location) {
			   for (let i in this.markers) {
			     if (this.markers[i] == marker) {
			        this.markers[i].location = newLocation;
			        break;
			     }
			   }
			},
			
			selectWaypointFromMarker(marker) {
				this.waypoints.forEach((waypoint) => {
					if (waypoint.marker == marker) {
						this.selectedWaypoint = waypoint;
					}
				});
			},

			addMarker(location, map) {
				let marker = new google.maps.Marker({
					map: map,
					draggable:true,
					position: location,
					icon : { 
						url: "http://maps.google.com/mapfiles/ms/icons/blue-dot.png" 
					},
				});

				google.maps.event.addListener(marker, 'click', () => {
					this.markers.forEach((m) => {
						m.setIcon("http://maps.google.com/mapfiles/ms/icons/blue-dot.png");
					});
					marker.setIcon("http://maps.google.com/mapfiles/ms/icons/yellow-dot.png");
					this.selectedMarker = marker;
					this.selectWaypointFromMarker(marker);
				});

				marker.addListener('drag', this.onMarkerDrag);
    			marker.addListener('dragend', this.onMarkerDragEnd);

				this.polyline.getPath().push(location);
				this.markers.push(marker);

				this.createWaypoint(marker);
			},

			calcBearing(fromLatLng, toLatLng) {
				return google.maps.geometry.spherical.computeHeading(fromLatLng, toLatLng);
			},

			calcDistance(fromLatLng, toLatLng) {
				return google.maps.geometry.spherical.computeDistanceBetween(fromLatLng, toLatLng);
			},

			calcAngle(fromBearing, toBearing) {
				if (fromBearing == undefined || toBearing == undefined) {
					return null;
				}

				let angle = Math.abs(fromBearing - toBearing);
				if (angle > 180) {
				    angle = 360 - angle;
				}
				return angle;
			},
			
			getNavigation(bearing, angle, distance) {
				console.log(bearing, angle, distance);
				if (angle < 10) {
					return `Fortsett rett fram de neste ${distance} meterne`;
				}
				if (angle > 10 && angle < 25) {
					if (bearing < 0) {
						return `Fortsett svakt venstre over de neste ${distance} meterne`;
					}
					else {
						return `Fortsett svakt høyre over de neste ${distance} meterne`;
					}
				}
				// TODO: more finegrained.
			},

			createWaypoint(marker) {
				let bearing;
				let distance;
				let angle;
				let lastMarker = this.markers[this.markers.length - 2];

				if (lastMarker) {
					bearing = this.calcBearing(lastMarker.position, marker.position);
					distance = this.calcDistance(lastMarker.position, marker.position);
					angle = this.calcAngle(this.waypoints[this.waypoints.length - 1].bearing, bearing);
				}

				this.waypoints.push({
					'url': '',
					'radius': 10,
					'latitude' : '',
					'longitude': '',
					'heading': null,
					'marker': marker,
					'angle': angle,
					'bearing': bearing,
					'searchable': false,
					'distance': distance,
					'navigateable': false,
					'title': this.waypoints.length.toString(),
					'default_navigation': this.getNavigation(bearing, angle, distance),
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
		height: 950px;
	}

	.hand {
		cursor: pointer,
	}

	.waypoints {
		height: 900px;
		overflow-y: auto;
	}
</style>
