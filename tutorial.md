# HERE Mobility SDK Tutorial


## Introduction to `INetworkClient` ##

All remote services provided by the Mobility SDK implement [`INetworkClient`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/core_common_api/net/INetworkClient.html).
Clients manage the connection to the server and keep track of any active requests. When you send a request (e.g. `RoutingClient.requestRoute`),you receive a [`ResponseFuture`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/core_common_api/net/ResponseFuture.html) object representing the promise to deliver you the response, once it arrives from the server. There are 3 ways to use `ResponseFuture`:

- `getResponse` waits, blocking the caller thread, until a response arrives and returns it.
     Can throw `HMException`.
- `registerListener` registers a callback to be called, asynchronously, when the response arrives.
     Can return  `HMException` in the onFailure function.
-  `ResponseFuture` implements Guava's `ListenableFuture` interface, which means that you can use it as such. Read [ListenableFuture Explained](https://github.com/google/guava/wiki/ListenableFutureExplained) for more information.
     The get function can throw `HMException` as the cause of the ExecutionException.

The `HMException` is the base exception for all Mobility SDK exceptions.
The exceptions that can be thrown are specified in the documentation of every API call, you can determine the specific exception type using `instanceof` in Java or `is` in Kotlin.

Additionally, you can `cancel` the request while it's still in progress.

The recommended way to work with an `INetworkClient` in the context of an activity is:

1. Create an `INetworkClient` in your activity's `onCreate` method.
2. Send requests as the user interacts with the activity.
3. In your activity's `onStop()` method, call `cancelAllActiveRequests`, in order to prevent callbacks from being called while the activity is stopped. Alternatively, manage the `ResponseFuture`s yourself for more fine-grained control.
4. In your activity's `onDestroy()` method, call `shutdownNow` in order to close the client, including the underlying connection to the server.

## Mobility Demand API

## Map Services API
The Map Services module of the SDK bundles 3 independent APIs:

1. [*Geocoding*](#geocoding), which includes:
	1. [*Forward geocoding*](#forward-geocoding): Lookup of places by a query string.
	2. [*Reverse geocoding*](#reverse-geocoding): Given the geographical coordinates of a location, determine its address.
	3. [*Places around*](#places-around): Places of interest around a given location.
2. [*Routing*](#routing): Instructions for getting from one location to another (typically driving), including the geometry of the route.
3. [*Maps*](#routing): Display an interactive map on the screen.

### Geocoding <a name="geocoding"></a>
Geocoding services are provided by the [`GeocodingClient`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/geocoding/GeocodingClient.html) class. All three geocoding APIs are implemented by the `geocode(GeocodingRequest)` method; which API is called depends on the [`GeocodingRequest`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/geocoding/GeocodingRequest.html) you pass.

The response to a geocoding request is a [`GeocodingResponse`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/geocoding/GeocodingResponse.html) containing a list of [`GeocodingResult`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/geocoding/GeocodingResult.html)s. A `GeocodingResult` consists of

- *type*: either "address" or "place". A "place" result corresponds to a landmark or a point of interest of some sort.
- *title*: The name of the place or the address. Always present for forward geocoding, but is optional for other requests.
- *addressText*: The address of the result, as one string. Present or missing together with *addressText*.
- *address*: A detailed description of the address, by fields (country, city, street etc). Present or missing together with *addressText*.
- *location*: The coordinates of the result.

When a `GeocodingResult` does not have an address, you can obtain it via `GeocodingClient.getAddressDetails`.


#### Forward Geocoding <a name="forward-geocoding"></a>
Forward geocoding, sometimes known as "Address/Place Search" or "Address/Place Autocomplete", is the method of finding the coordinates of a location specified by a query string. The result of a forward geocoding request is typically a list of locations (including their full name). These can be shown to the user, who then selects the location he was looking for.

To issue a forward geocoding request, create a `GeocodingRequest` via one of its `newForwardRequest` methods, and pass it to `GeocodingClient.geocode`. The parameters of `newForwardRequest` are:

- *query*: The query string, a mandatory parameter.
- *location*: The location for anchoring the search. Typically, this should be the user's actual location, or the country/city center if the app is specific to a country/city. The ordering of the results will depend on it. This is a mandatory parameter.
- *countryCode*: The ISO 3166-1 alpha 3 three-letter country code for filtering the results. Only results in this country will be returned. This is an optional parameter; if not specified, results will not be limited to a single country.
- *languageCode*: The ISO 639-1 two-letter language code for the preferred language of the results. This is an optional parameter. If not specified, there is no guarantee as to what language the results will be in.
- *resultTypes*: The types of results you're interested in. Possible values are `Type.ADDRESS` and `Type.PLACE`. `newForwardRequest` overloads which don't take this argument send a request for both types.

The response is a list of [`GeocodingResult`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/geocoding/GeocodingResult.html) instances, each representing a potential match.

#### Reverse Geocoding <a name="reverse-geocoding"></a>
Reverse geocoding is the method of finding the address at a given location, specified by its coordinates.

To issue a reverse geocoding request, create a `GeocodingRequest` via one of its `GeocodingRequest.newReverseRequest` methods, and pass it to `GeocodingClient.geocode`. The parameters of `newReverseRequest` are:

- *location*: The coordinates of the location whose address you want to find.
- *languageCode*: The ISO 639-1 two-letter language code for the preferred language of the resulting address (its city, street names etc). This is an optional parameter.

#### Places Around <a name="places-around"></a>
The "Places Around" API is similar to reverse geocoding, except that it returns a list of "place" results (as opposed to an "address" result). You create a request with one of the `GeocodingRequest.newPlacesAroundRequest` methods. The parameters are the same as in regular reverse geocoding.


### Routing <a name="routing"></a>
Routing services are provided by the [`RoutingClient`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/route/RoutingClient.html) class, via its `    requestRoute(RouteRequest)` method. Create a `RouteRequest` via one of its `create` methods, the parameters of which are:
- *profile*: The travel profile with which to compute the route. Currently only travel by car is supported.
- *waypoints*: A list of locations to travel through. The first location is the origin; the last one is the final destination. The list must contain at least two locations.
- *queryAlternatives*: Whether to return alternative routes, in addition to the main one.
- *instructionMeasurementSystem*: The measurement system (metric, imperial) to use in the routing instructions.
- *instructionsLanguageCode*: The BCP47 language code to use in the routing instructions.

The response to a routing request is a [`RouteResponse`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/route/RouteResponse.html) object, consisting of a list of [`Route`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/route/RouteResponse.html)s. If you haven't asked for alternative routes, it should contain at most one route.

A `Route` object describes the geometry and the instructions for traversing the route. A `Route` is divided into `Leg`s, each leg describing the path between two consecutive waypoints (as specified in the routing request). Each leg, in turn, consists of a list of `Maneuver`s. Each such sub-part of the route can be queried for its length and travel time.

### Maps <a name="maps"></a>

The Maps API lets you display an interactive map. To include a map in your app, add `com.here.mobility.sdk.map.MapFragment` to your layout:

```java
<fragment
	android:name="com.here.mobility.sdk.map.MapFragment"
	android:id="@+id/map_fragment"
	android:layout_width="match_parent"
	android:layout_height="match_parent" />
```

Then retrieve the fragment in your code and initialize the map:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);

	FragmentManager fm = getFragmentManager();
	MapFragment mapFragment = (MapFragment)fm.findFragmentById(R.id.map_fragment);
	mapFragment.loadMapAsync(new MapView.MapControllerListener(){
	    
		@Override
		public void onMapReady(@NonNull MapController map){
		    // Use the map: set its center, add markers etc.
		}
		
		@Override
		public void onMapInitializationFailure(@NonNull Exception e){
		    // Map loading failed
		}
		
		@Override
		public void onAuthenticationFailure(@NonNull UserAuthenticationException e){
		    // Some authentication issue when loading tiles
		    // You should sometimes supply a new authentication credentials (according the 
		    // documentation of the specific exception's error code).
		    // You should also consider calling MapController.requestRender() to refresh the map.
		}
	
	});
}
```

The [`MapController`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/MapController.html) object you get in `onMapReady` is the main class for controlling the map.

#### Retrieve and control the camera

- *Position* via `getPosition`, `setPosition` and `setPositionEased`
- *Zoom* via `getZoom`, `setZoom` and `setZoomEased`.
- *Rotation* via `getRotation`, `setRotation` and `setRotationEased`.
- *Tilt* via `getTilt`, `setTilt` and `setTiltEased`. Additionally, you can control the maximum tilt the user can reach (with the tilt gesture) via `setMaxTilt`.
- *Bounding box*: a convenient way to show a given bounding box are the `showBoundingBox` methods, which compute for you the position and zoom at which a given bounding box will completely fit inside the map's view.

The *eased* method variants move the camera with a specified animation length and/or type.

#### React to interactions with the map

The map is preconfigured with the standard finger gestures:

- Zoom on double-tap.
- Moving the camera when panning.
- Rotating and scaling the map with two-finger rotate and "pinch" gestures.
- Tilting the map with the two-finger "shove" gesture.

You can replace the action taken when these gestures are performed, and also set the actions for tap and long-tap gestures via the `set[Gesture]Responder` methods.

#### Convert between screen and map coordinates

Use the `screenPositionToLatLng` and `latLngToScreenPosition` methods to convert between map (earth) coordinates and screen (pixels, relative to the map view) coordinates.

#### Markers

Use `addMarker` to add markers to the map. A marker is an icon attached to a geographical location. `addMarker` returns a [`Marker`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/Marker.html) object, which you can then configure:

- `setPoint` to set the marker's location.
- `setDrawable` or `setBitmap` to set the marker's icon.
- `setAngle` to rotate the marker's icon.
- `setVisible` to control whether the marker is currently visible.
- `setUserData` to associate your own object with a marker, which can be later retrieved with `getUserData`.

The marker API is currently a work-in-progress, because of this, setting a marker's icon is currently rather inelegant. Until we improve the API, use this template:

```java
@NonNull
public Marker addMarker(@DrawableRes int drawableResId){
	Marker marker = mapController.addMarker();
	Resources resources = context.getResources();
	Drawable drawable = ResourcesCompat.getDrawable(resources, drawableResId, null);
	if (drawable == null){
		throw new Resources.NotFoundException();
	}

	int width = drawable.getIntrinsicWidth();
	int height = drawable.getIntrinsicHeight();

	float density = resources.getDisplayMetrics().density;
	String size = "[" + Math.round(width/density) + "px, " 
	                  + Math.round(height/density) + "px]";
	String styleString = "{ style: 'points'" + 
	                     ", color: 'white'" + 
	                     ", size: " + size + 
	                     ", order: 10000" + 
	                     ", collide: false }";
	marker.setStylingFromString(styleString);
	marker.setDrawable(drawable);

	return marker;
}
```

Remove a marker via `MapController.removeMarker`, or `MapController.removeAllMarkers` to remove all the markers.

#### Polylines

Use `addPolyline(GeoPointsList, PolylineStyle)` to add a polyline to the map. A polyline is a list of geographical points with lines drawn between each two consecutive ones. The `GeoPointsList` and `PolylineStyle` arguments specifies the properties of the polyline, consisting its geometry and style respectively. 
You can customize color, width (in pixels or meters), cap and join style of the polyline.

To remove a polyline you've added use `removePolyline`, passing it the polyline *Polyline* instance returned by `addPolyline`.
To remove all polylines use `removeAllPolylines`.

#### Polygons

Use `addPolygon(GeoPointsList, PolygonStyle)` to add a polygon to the map. A polygon is a list of geographical points that represents the the area between those points, the start point of the polygon is also his end point. The `GeoPointsList` and `PolygonStyle` arguments specifies the properties of the polygon, consisting its geometry and style respectively. 
You can customize color, width (in pixels or meters) and join style of the polygon.

To remove a polygon you've added use `removePolygon`, passing it the polygon *Polygon* instance returned by `addPolygon`.
To remove all polygons use `removeAllPolygons`.


#### User location markers

Use [`UserLocationMarkerManager`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/UserLocationMarkerManager.html), obtained with `MapController.getUserLocationMarkerManager` to control a special marker that tracks the device's actual location. There are three modes you can set via `setMarkerMode`:

- `MARKER_HIDDEN`: The user location marker is not shown on the map.
- `NON_DIRECTIONAL_MARKER_VISIBLE`: A marker that tracks the device's location, but not orientation (rotation, direction) in space, is shown.
- `DIRECTIONAL_MARKER_VISIBLE`: A marker that tracks the device's location and orientation in space is shown. The marker's icon should have an arrow-like shape, and it's rotated by `UserLocationMarkerManager` to the appropriate angle.

The SDK provides default marker icons, but you can change them to match your app's style with `UserLocationMarkerManager.setNonDirectionalMarker` and `UserLocationMarkerManager.setDirectionalMarker`.

`UserLocationMarkerManager` doesn't actually know itself the device's location and orientation in space. You must provide it with a [`UserLocationSource`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/UserLocationSource.html) and [`DeviceDirectionSource`](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/DeviceDirectionSource.html) that supply it with this information. The SDK provides implementations of these two interfaces, but you must still set them up yourself.

[**FusedUserLocationSource**](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/FusedUserLocationSource.html) is an implementation of `UserLocationSource` that uses Google's fused location provider. All you need to do is to specify the `LocationRequest` (or just use the default one).

[**RotationSensorDirectionSource**](https://heremobilitydevelopers.github.io/HERE-Mobility-SDK-Android/com/here/mobility/sdk/map/RotationSensorDirectionSource.html) implements `DeviceDirectionSource` by analyzing the device's gyroscope and magnetometer readings to determine the orientation of the device.

#### Compass

You can configure `MapController` to display a small "compass" view on the map, to point to the north. Use `setCompassVisibility` with one of three modes:

- `HIDDEN_ALWAYS` The compass is always hidden
- `VISIBLE_ALWAYS` The compass is always visible
- `VISIBLE_WHEN_MAP_ROTATED` The compass is only visible when the map is rotated, either via user gestures, or programmatically, with `setRotation`. When the angle is 0, the compass fades out and disappears.

You can also implement your own compass view, and use `MapController.setCompassView` to ask the map to show it.
