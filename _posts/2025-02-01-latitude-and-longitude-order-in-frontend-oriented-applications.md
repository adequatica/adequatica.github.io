---
layout: post
title: 'Latitude and Longitude Order in Frontend-oriented Applications'
date: 2025-01-31 21:11:12 +0100
tags: coordinates
---

Long story short, geospatial software has no «correct» coordinate order.

![Latitude and Longitude of the Earth](/assets/2025-01-31/01-latitude-longitude.png)

_Fig. 1. Latitude and Longitude of the Earth ([original image source](https://commons.wikimedia.org/wiki/File:Latitude_and_Longitude_of_the_Earth.svg))._

In my current project, there was considerable debate within the development team about the [coordinate](https://en.wikipedia.org/wiki/Geographic_coordinate_system) order: Latitude, Longitude — as used by several internal and external APIs, or Longitude, Latitude — as a client-side JavaScript library Mapbox GL JS does.

There are cons for each variant:

## Latitude, Longitude:

- Latitude, Longitude order follows the [ISO 6709](https://en.wikipedia.org/wiki/ISO_6709) standard;
- **All real-world maps** represent coordinates in the Latitude, Longitude order — it is a maritime, navigational, geographical, and cartographical tradition;
- People learned to determine latitude ([through celestial observations](https://en.wikipedia.org/wiki/Celestial_navigation)) much earlier [than longitude](https://en.wikipedia.org/wiki/History_of_longitude) so that it may take precedence due to historical and «discovery» reasons;
- Since latitude represents north-south positioning, looking at the first coordinate value (latitude) allows for a rough estimation of an object’s distance from the equator — this is relatively convenient.

Highlights from the ISO standard:

- Latitude comes before longitude;
- North latitude is positive;
- East longitude is positive;
- Coordinate values (latitude, longitude, and altitude) should be delimited by spaces;
- Latitude and longitude should be displayed using sexagesimal fractions (i.e., minutes and seconds).

For example: `44°41′45.5″N 20°30′52″E 511m`

## Longitude, Latitude:

- Longitude, Latitude order is used in the [GeoJSON](https://geojson.org) format, which follows the [RFC 7946](https://datatracker.ietf.org/doc/html/rfc7946) standard;
- Longitude, Latitude is a conventional order **in mathematics, where the horizontal (X) coordinate (Longitude) precedes the vertical (Y) coordinate (Latitude)** (see explanation in [Cartesian](https://en.wikipedia.org/wiki/Cartesian_coordinate_system) and [spherical](https://en.wikipedia.org/wiki/Spherical_coordinate_system) coordinate systems);
- The ISO standard specifies the representation and input of coordinates but does not dictate how software applications store them — therefore, data can be stored in any convenient format;
- Most databases ([MySQL](https://dev.mysql.com/doc/refman/8.4/en/gis-point-property-functions.html), [PostGIS](https://postgis.net/documentation/tips/lon-lat-or-lat-lon/), [Oracle](https://docs.oracle.com/en/database/oracle/oracle-database/19/spatl/coordinate-systems-concepts.html#GUID-5CDBB4BD-2721-43A1-99DD-C195B909F85B), [Redis](https://redis.io/docs/latest/develop/interact/search-and-query/advanced-concepts/geo/)) store points as Longitude, Latitude (X, Y).

Highlights from the RFC:

- A position is an array of numbers;
- The first two elements are longitude and latitude (or easting and northing), precisely in that order and using decimal numbers. Altitude or elevation may be included as an optional third element;
- The coordinate reference system for all GeoJSON coordinates is a geographic coordinate reference system using the [World Geodetic System 1984 (WGS 84)](https://en.wikipedia.org/wiki/World_Geodetic_System#WGS_84) datum, with longitude and latitude units in decimal degrees.

For example: `[44.695972, 20.514444]`

---

To make an informed decision on which coordinate order is «better», I collected data on how different geospatial software represent coordinate order. The selected projects reflect only my domain area: frontend development, JavaScript APIs, and JS libraries.

#### Standards and formats

| Latitude Longitude                                                          | Longitude latitude                                                                                             |
| --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| ISO 6709 <sup>[docs](https://en.wikipedia.org/wiki/ISO_6709)</sup>          | GeoJSON (RFC 7946) <sup>[docs](https://datatracker.ietf.org/doc/html/rfc7946#section-4)</sup>                  |
| GeoRSS <sup>[docs](https://www.ogc.org/publications/standard/georss/)</sup> | KML <sup>[docs](https://developers.google.com/kml/documentation/kmlreference#elements-specific-to-point)</sup> |
| OSM JSON <sup>[docs](https://wiki.openstreetmap.org/wiki/OSM_JSON)</sup>    |                                                                                                                |

#### URLs

| Latitude Longitude                                                                                                                                                                      | Longitude latitude                                                                       |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| Google Maps <sup>[docs](https://developers.google.com/maps/documentation/urls/get-started#constructing-valid-urls), [e.g.](https://www.google.com/maps/@44.8198261,20.436645,16z)</sup> | 2GIS <sup>[e.g.](https://2gis.ae/dubai/geo/13933647002594323/55.27434%2C25.197091)</sup> |
| Apple Maps <sup>[e.g.](https://beta.maps.apple.com/?ll=44.818161959837006%2C20.443788177820323&spn=0.040468247493485876%2C0.09146203301651212)</sup>                                    | EPSG <sup>[e.g.](https://epsg.io/map#srs=4326&x=20.442413&y=44.819579&z=17)</sup>        |
| Bing Maps <sup>[e.g.](https://www.bing.com/maps?cp=44.823653%7E20.450316&lvl=17.5)</sup>                                                                                                | Yandex Maps <sup>[e.g.](https://yandex.com/maps/?ll=20.453578%2C44.817094)</sup>         |
| HERE WeGo <sup>[e.g.](https://maps.here.com/?map=44.82377,20.45185)</sup>                                                                                                               |                                                                                          |
| Geojson.io <sup>[e.g.](https://geojson.io/#id=gist:anonymous/&map=15.87/44.823377/20.448848)</sup>                                                                                      |                                                                                          |
| OpenStreetMap <sup>[e.g.](https://www.openstreetmap.org/#map=19/44.823027/20.447236)</sup>                                                                                              |                                                                                          |

#### JavaScript APIs

| Latitude Longitude                                                                                                                              | Longitude latitude                                                                                                                            |
| ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Google Maps JavaScript API <sup>[docs](https://developers.google.com/maps/documentation/javascript/reference/coordinates)</sup>                 | 2GIS MapGL JS API <sup>[docs](https://docs.2gis.com/en/mapgl/reference/Map)</sup>                                                             |
| HERE Maps API for JavaScript <sup>[docs](https://www.here.com/docs/bundle/maps-api-for-javascript-api-reference/page/H.Map.html#.Options)</sup> | ArcGIS Maps SDK for JavaScript <sup>[docs](https://developers.arcgis.com/javascript/latest/maps-2d/#set-the-visible-portion-of-the-map)</sup> |
| Kokao Maps Web API <sup>[docs](https://apis.map.kakao.com/web/documentation/#LatLng)</sup>                                                      | Azure Maps Web SDK <sup>[docs](https://learn.microsoft.com/en-us/azure/azure-maps/how-to-use-map-control)</sup>                               |
| MapQuest.js <sup>[docs](https://developer.mapquest.com/documentation/sdks/mapquest-js/)</sup>                                                   | Mapbox GL JS <sup>[docs](https://docs.mapbox.com/mapbox-gl-js/api/geography/#lnglat)</sup>                                                    |
| Yandex Maps JavaScript API\* <sup>[docs](https://yandex.com/dev/jsapi-v2-1/doc/en/v2-1/ref/reference/meta#coordinatesOrder)</sup>               | MapTiler Client JS <sup>[docs](https://docs.maptiler.com/client-js/coordinates/)</sup>                                                        |
|                                                                                                                                                 | Tomtom Maps SDK for Web <sup>[docs](https://developer.tomtom.com/maps-sdk-web-js/documentation#Maps.LngLat)</sup>                             |

✻ In Yandex JavaScript API, developers can set a preferred order setting: [latitude, longitude] or [longitude, latitude], but the first one is the default.

Among other SDKs, [Apple MapKit](https://developer.apple.com/documentation/mapkitjs/mapkit.coordinate/mapkit.coordinate) (for iOS) and [Google Maps SDK](https://developers.google.com/maps/documentation/android-sdk/coordinates) (for Android) both use Latitude, Longitude order.

#### Libraries

| Latitude Longitude                                                     | Longitude Latitude                                                                                    |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Leaflet <sup>[docs](https://leafletjs.com/reference.html#latlng)</sup> | CesiumJS <sup>[docs](https://cesium.com/learn/cesiumjs-learn/cesiumjs-quickstart/)</sup>              |
|                                                                        | D3 <sup>[docs](https://github.com/d3/d3-geo)</sup>                                                    |
|                                                                        | DataMaps <sup>[docs](https://github.com/markmarkoh/datamaps/blob/master/README.md)</sup>              |
|                                                                        | Deck.gl <sup>[docs](https://deck.gl/docs/developer-guide/coordinate-systems)</sup>                    |
|                                                                        | GeoExt <sup>[docs](https://geoext.github.io/geoext/)</sup>                                            |
|                                                                        | MapLibre GL JS <sup>[docs](https://maplibre.org/maplibre-gl-js/docs/API/classes/LngLat/)</sup>        |
|                                                                        | OpenLayers <sup>[docs](https://openlayers.org/en/latest/apidoc/module-ol_proj.html#.fromLonLat)</sup> |
|                                                                        | TurfJS <sup>[docs](https://turfjs.org/docs/getting-started)</sup>                                     |

Most JavaScript libraries (except Leaflet) use the GeoJSON format, which provides remarkable consistency within the JavaScript ecosystem.

---

Looking at the tables above, some identifiable patterns emerge in the use of coordinate order:

- In URLs, it’s better to use the Latitude, Longitude order. In fact, **for all _human-readable presentations_ and coordinate inputs, Latitude, Longitude is preferable;**
- If the application relies on APIs and libraries based on the GeoJSON format, it is more pragmatic to use the Longitude, Latitude order;
- In other cases, the choice is up to you — **there is no «correct» way.** It depends on the application’s context.

However, the problem runs deeper. Should coordinates be represented as an [ordered pair](https://en.wikipedia.org/wiki/Ordered_pair) (like `[44.69, 20.51]`) or as an object (like `{lat: 44.69, lng: 20.51}`)? Regardless of the choice, due to the use of multiple APIs and libraries with different coordinate notations, we will still need to convert coordinates.

> _«If using multiple libraries or a different combination of software in a GIS workflow, this may lead to mistakes and frustration.»_ [Lat Lon or Lon Lat](https://observablehq.com/@clhenrick/lat-lon-or-lon-lat)?

In my current project, we chose the `{lat: 0, lng, 0}` object format for several reasons:

- The `lat` and `lng` (or `lon` or `long`) keys in JSON explicitly indicate the position, eliminating the need for developers to think about coordinate order;
- Object keys are [commutative](https://en.wikipedia.org/wiki/Commutative_property), unlike an ordered pair in an array, reducing the risk of mixing up coordinates;
- Coordinates stored as keys can be used within extended objects, not just as a pair;
- Since we frequently convert coordinates between different third-party APIs and libraries, it’s better to store intermediate data in a convenient, explicit, and human-readable format to [minimize errors](https://adequatica.github.io/2022/09/26/field-notes-in-software-testing.html#on-coordinates).

To prevent errors when converting, receiving, or transferring coordinates, tests should verify the following:

- Latitude ranges between -90 and 90 degrees, inclusive;
- Longitude ranges between -180 and 180 degrees, inclusive;
- Coordinates `0, 0` is a valid point, known as [Null Island](https://en.wikipedia.org/wiki/Null_Island).

Read further:

- [Latitude, Longitude, and Coordinate System Grids](https://gisgeography.com/latitude-longitude-coordinates/);
- [Lon lat lon lat](https://macwright.com/lonlat/) and [longitude, latitude is the right way](https://macwright.com/2016/07/15/longitude-latitude-is-the-right-way) by Tom MacWright;
- [Lon Lat Lon Lat](https://news.ycombinator.com/item?id=30228981) (Hacker News discussion);
- [Preferred order of writing latitude & longitude tuples in GIS services](https://stackoverflow.com/questions/7309121/preferred-order-of-writing-latitude-longitude-tuples-in-gis-services) (Stack Overflow discussion);
- [Displaying coordinates and inputs as LatLon or LonLat](https://gis.stackexchange.com/questions/6037/displaying-coordinates-and-inputs-as-latlon-or-lonlat) (GIS Stack Exchange discussion);
- [Latitude and Longitude](https://www.open.edu/openlearn/history-the-arts/history/history-science-technology-and-medicine/history-science/latitude-and-longitude);
- [Longitude: The True Story of a Lone Genius Who Solved the Greatest Scientific Problem of His Time](https://www.amazon.com/Longitude-Genius-Greatest-Scientific-Problem/dp/080271529X) by Dava Sobel.

Copy @ [Medium](https://adequatica.medium.com/latitude-and-longitude-order-in-frontend-oriented-applications-0de61453ed4a)
