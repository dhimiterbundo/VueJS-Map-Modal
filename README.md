# VueJS-Map-Modal
This is a Google Map Modal that use current location to get coordinates and transform it into a location using GoogleApis.

Install GoogleMap from https://www.npmjs.com/package/vue2-google-maps .
Create GoogleMap component and after use it as a modal with modal Component. 

-At Modal component there is a googleApi request to tranfortm coordinated into location String. 
-Google Apis requeire a key to make the request , 
- Remove the headers from request while it makes the requst because it will not work.


At your ApiFactory.js add this:




At main.js

include: 
import * as VueGoogleMaps from 'vue2-google-maps';

Vue.use(VueGoogleMaps, {
    load: {
        key: process.env.VUE_APP_GOOGLE_MAP_KEY, // google key, use it as a constant ..
        libraries: 'places'
    }
});


