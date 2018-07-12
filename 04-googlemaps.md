# Create Express app
1. `express -e --git goomap`
1. `npm i`

# Google Maps API 
1. Goto https://developers.google.com/maps/documentation/javascript/tutorial
1. Copy their example and paste in views/index.ejs

Run/Test it - it errors because you need a Google API key and secret
1. Add API to index.ejs script url
1. Add marker `var marker = new google.maps.Marker({ position: { lat: -34.397, lng: 150.644 }, map: map });`
1. Make marker accessible beyond function and in console type: `marker.setPosition({ lat: -34, lng: 150, alt: 0 })`

# Geolocation
1. goto https://developer.mozilla.org/en-US/docs/Web/API/Geolocation_API
1. Create index.html; Test if geolocation service exists with desktop browser; if successful put it into phone.ejs
    ```html
    <body>
        <script>
            if ("geolocation" in navigator) {
                /* geolocation is available */
                document.body.textContent = "geolocation is available";
            } else {
                /* geolocation IS NOT available */
                document.body.textContent = "geolocation is NOT available";
            }
        </script>
    </body>
    ```
1. Test if index.html works with mobile browsers
    `lt --port 3000 --subdomain avcoder`

# Getting the current position
1. Copy/paste MDN's code into our index.html script and display coordinates
    ```js
    navigator.geolocation.getCurrentPosition(function (position) {
        document.body.textContent = `lat: ${position.coords.latitude}, lng: ${position.coords.longitude}`;
    });
    ```
1. Test if index.html works within desktop browser/mobile browsers

# Pass current position to route /setLatLng
1.  Change route/index.js:
    ```js
    router.get('/setlatlng', (req, res) => {
        coords.lat = req.query.lat;
        coords.lng = req.query.lng;
        res.send(JSON.stringify(coords));
    });
    ```
1.  Test in desktop browser if /setlatlng?lat=2.3&lng=1.2 works

# Display passed in coordinates
1.  Change route/index.js:
    ```js
    router.get('/', (req, res) => {
        res.render('index', {
            title: `${coords.lat}, ${coords.lng}`,
            lat: coords.lat,
            lng: coords.lng,
        });
    });
    ```

    In index.ejs
    ```js
    function initMap() {
      map = new google.maps.Map(document.getElementById('map'), {
        center: { lat: <%= lat %>, lng: <%= lng %> },
        zoom: 8
      });

      marker = new google.maps.Marker({
        position: { lat: <%= lat %>, lng: <%= lng %> },
        map: map
      });
    }
    ```


