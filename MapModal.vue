<template>
    <transition name="modal-fade">
        <div class="modal" style="display: block">
            <div class="modal-dialog d-flex align-items-center" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{title}}</h5>
                    </div>
                    <div class="modal-body">
                        <google-map v-on:newLocation="locationUpdated" :dragMarker="true"/>
                    </div>
                    <div class="modal-footer">
                        <button type="button" @click="onConfirm"
                                class="btn bk-outline-button"
                                v-action-loading-spinner="{
                                        loading: loadingButton,
                                        defaultDisable: loadingButton,
                                        defaultValue: $t('common.confirm')}">
                            {{$t('common.confirm')}}
                        </button>
                        <button type="button" @click="onCancel"
                                class="btn bk-button" data-dismiss="modal">
                            {{$t('common.cancel')}}
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </transition>
</template>

<script>
    import GoogleMap from '@/modules/common/components/GoogleMap';
    import ApiFactory from '@/modules/common/factories/ApiFactory';
    import notificationService from '@/modules/common/services/NotificationService';

    export default {
        name: 'MapModal',
        components: {
            GoogleMap
        },
        props: {
            title: { required: true },
            message: { required: true },
            bikeId: { required: true },
            loadingButton: { required: true }
        },
        mounted() {

        },
        methods: {
            onConfirm() {
                const vm = this;
                vm.$emit('confirm', {
                    bikeId: vm.bikeId,
                    location: vm.location
                });
            },
            onCancel() {
                this.$emit('cancel');
            },
            locationUpdated(location) {
                const vm = this;
                setTimeout(() => {
                    ApiFactory.get(`https://maps.googleapis.com/maps/api/geocode/json?latlng=${location.position.lat},
                    ${location.position.lng}&key=${vm.googleMapKey}`).then(
                        (success) => {
                            vm.street = success.data.results[0].formatted_address;
                        },
                        (error) => notificationService.error(error)
                    );
                }, 500);
                vm.location = { location, street: location.street ? location.street : vm.street };
            }
        },
        data() {
            return {
                location: {},
                street: null,
                googleMapKey: process.env.VUE_APP_GOOGLE_MAP_KEY,
            };
        }
    };
</script>
