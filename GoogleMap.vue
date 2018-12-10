<template>
    <div class="bk-map-container">
        <div class="row" v-if="showAutocomplete">
            <div class="col-sm-12">
                <div>
                    <label>{{$t('common.search_add_pin')}}</label>
                    <vue-google-autocomplete id="map" classname="form-control"
                                             v-on:placechanged="getAddressData">
                    </vue-google-autocomplete>
                </div>
            </div>
        </div>
        <br>
        <div class="row">
            <div class="col-sm-12">
                <gmap-map ref="bkMap"
                          :center="center"
                          :zoom="16"
                          style="width:100%;">

                    <gmap-marker ref="bkMarker"
                                 :position="marker.position"
                                 :draggable="dragMarker"
                                 @click="center=marker.position"
                                 @dragend="changedPosition"
                                 :icon.sync="marker.icon"
                    ></gmap-marker>
                </gmap-map>
            </div>
        </div>
    </div>
</template>

<script>
    import ApiFactory from '@/modules/common/factories/ApiFactory';
    import notificationService from '@/modules/common/services/NotificationService';
    import markerImage from '@/assets/marker.png';
    import VueGoogleAutocomplete from 'vue-google-autocomplete';

    export default {
        name: 'GoogleMap',
        props: {
            position: { required: false },
            showAutocomplete: { required: false, default: true },
            showInitialMarker: { required: false, default: false },
            dragMarker: { required: true, type: Boolean }

        },
        components: {
            VueGoogleAutocomplete
        },
        data() {
            return {
                center: { lat: 45.508, lng: -73.587 },
                marker: {
                    position: {},
                    icon: markerImage
                },
                places: [],
                currentPlace: null,
                place: '',
                googleMapKey: process.env.VUE_APP_GOOGLE_MAP_KEY,
            };
        },
        watch: {
            position(newVal) {
                this.marker = newVal;
                this.center = newVal.position;
            }
        },
        mounted() {
            const vm = this;
            if (vm.position) {
                vm.marker = vm.position;
            } else if (!vm.showInitialMarker) {
                vm.geolocate();
            }
        },
        methods: {
            getAddressData(address) {
                const vm = this;
                const streetParts = [];
                if (address.route) {
                    streetParts.push(address.route);
                }
                if (address.locality) {
                    streetParts.push(address.locality);
                }
                if (address.country) {
                    streetParts.push(address.country);
                }
                vm.marker.position = { lat: address.latitude, lng: address.longitude };
                vm.marker.position = { lat: address.latitude, lng: address.longitude };
                vm.center = vm.marker.position;
                vm.marker.street = streetParts.join(', ');
                vm.$emit('newLocation', vm.marker);
            },
            changedPosition() {
                const vm = this;
                this.$refs.bkMarker.$markerPromise.then((marker) => {
                    vm.marker.position = { lat: marker.position.lat(), lng: marker.position.lng() };
                    ApiFactory.get(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${vm.marker.position.lat},
                    ${vm.marker.position.lng}&key=${vm.googleMapKey}`).then(
                        (success) => {
                            vm.marker.street = success.data.results[0].formatted_address;
                        },
                        (error) => notificationService.error(error)
                    );
                    this.$emit('newLocation', vm.marker);
                });
            },
            geolocate() {
                const vm = this;
                navigator.geolocation.getCurrentPosition(position => {
                    vm.center = {
                        lat: position.coords.latitude,
                        lng: position.coords.longitude
                    };
                    vm.marker = {
                        draggable: true,
                        position: vm.center,
                        icon: markerImage
                    };
                    ApiFactory.get(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${vm.marker.position.lat},
                    ${vm.marker.position.lng}&key=${vm.googleMapKey}`).then(
                        (success) => {
                            vm.marker.street = success.data.results[0].formatted_address;
                        },
                        (error) => notificationService.error(error)
                    );
                    setTimeout(() => {
                        vm.$emit('newLocation', vm.marker);
                    }, 500);
                });
            }
        }
    };
</script>
<style>
</style>

