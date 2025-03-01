---
title: Show route directions on a map | Microsoft Azure Maps
description: In this article, you'll learn how to display directions between two locations on a map using the Microsoft Azure Maps Web SDK.
author: eriklindeman
ms.author: eriklind
ms.date: 07/29/2019
ms.topic: conceptual
ms.service: azure-maps
ms.custom: codepen
---

# Show directions from A to B

This article shows you how to make a route request and show the route on the map.

There are two ways to do so. The first way is to query the [Get Route Directions] request in the Azure Maps Route service. The second way is to use the [Fetch API] to make a search request to the [Get Route Directions] request. Both ways are discussed below.

## Query the route via service module

<iframe height='500' scrolling='no' title='Show directions from A to B on a map (Service Module)' src='//codepen.io/azuremaps/embed/RBZbep/?height=265&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true'>See the Pen <a href='https://codepen.io/azuremaps/pen/RBZbep/'>Show directions from A to B on a map (Service Module)</a> by Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

In the above code, the first block constructs a map object and sets the authentication mechanism to use the access token. You can see [create a map] for instructions.

The second block of code creates a `TokenCredential` to authenticate HTTP requests to Azure Maps with the access token. It then passes the `TokenCredential` to `atlas.service.MapsURL.newPipeline()` and creates a [Pipeline] instance. The `routeURL` represents a URL to Azure Maps [Route service].

The third block of code creates and adds a [DataSource] object to the map.

The fourth block of code creates start and end [points] objects and adds them to the dataSource object.

A line is a [Feature] for LineString. A [LineLayer] renders line objects wrapped in the  [DataSource] as lines on the map. The fourth block of code creates and adds a line layer to the map. See properties of a line layer at [LinestringLayerOptions].

A [symbol layer] uses texts or icons to render point-based data wrapped in the [DataSource]. The texts or the icons render as symbols on the map. The fifth block of code creates and adds a symbol layer to the map.

The sixth block of code queries the Azure Maps routing service, which is part of the [service module]. The [calculateRouteDirections] method of the `RouteURL` is used to get a route between the start and end points. A GeoJSON feature collection from the response is then extracted using the `geojson.getFeatures()` method and is added to the datasource. It then renders the response as a route on the map. For more information about adding a line to the map, see [add a line on the map].

The last block of code sets the bounds of the map using the Map's [setCamera] property.

The route query, data source, symbol, line layers, and camera bounds are created inside the [event listener]. This code structure ensures the results are displayed only after the map fully loads.

## Query the route via Fetch API

<iframe height='500' scrolling='no' title='Show directions from A to B on a map' src='//codepen.io/azuremaps/embed/zRyNmP/?height=469&theme-id=0&default-tab=js,result&embed-version=2&editable=true' frameborder='no' loading="lazy" allowtransparency='true' allowfullscreen='true'>See the Pen <a href='https://codepen.io/azuremaps/pen/zRyNmP/'>Show directions from A to B on a map</a> by Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

In the code above, the first block of code constructs a map object and sets the authentication mechanism to use the access token. You can see [create a map] for instructions.

The second block of code creates and adds a [DataSource] object to the map.

The third code block creates the start and destination points for the route. Then, it adds them to the data source. For more information, see [add a pin on the map].

A [LineLayer] renders line objects wrapped in the  [DataSource] as lines on the map. The fourth block of code creates and adds a line layer to the map. See properties of a line layer at [LineLayerOptions].

A [symbol layer] uses text or icons to render point-based data wrapped in the [DataSource] as symbols on the map. The fifth block of code creates and adds a symbol layer to the map. See properties of a symbol layer at [SymbolLayerOptions].

The next code block creates `SouthWest` and `NorthEast` points from the start and destination points and sets the bounds of the map using the Map's [setCamera] property.

The last block of code uses the [Fetch API] to make a search request to [Get Route Directions]. The response is then parsed. If the response was successful, the latitude and longitude information is used to create an array a line by connecting those points. The line data is then added to data source to render the route on the map. For more information, see [add a line on the map].

The route query, data source, symbol, line layers, and camera bounds are created inside the [event listener]. Again, we want to ensure that results are displayed after the map loads fully.

## Next steps

> [!div class="nextstepaction"]
> [Best practices for using the routing service](how-to-use-best-practices-for-search.md)

Learn more about the classes and methods used in this article:

> [!div class="nextstepaction"]
> [Map](/javascript/api/azure-maps-control/atlas.map)

See the following articles for full code examples:

> [!div class="nextstepaction"]
> [Show traffic on the map](./map-show-traffic.md)

> [!div class="nextstepaction"]
> [Interacting with the map - mouse events](./map-events.md)

[Get Route Directions]: /rest/api/maps/route/getroutedirections
[Route service]: /rest/api/maps/route
[Fetch API]: https://fetch.spec.whatwg.org/
[create a map]: map-create.md
[DataSource]: /javascript/api/azure-maps-control/atlas.source.datasource
[add a line on the map]: map-add-line-layer.md
[setCamera]: /javascript/api/azure-maps-control/atlas.map#setcamera-cameraoptions---cameraboundsoptions---animationoptions-
[SymbolLayerOptions]: /javascript/api/azure-maps-control/atlas.symbollayeroptions
[LineLayerOptions]: /javascript/api/azure-maps-control/atlas.linelayeroptions
[add a pin on the map]: map-add-pin.md
[LineLayer]: /javascript/api/azure-maps-control/atlas.layer.linelayer
[symbol layer]: /javascript/api/azure-maps-control/atlas.layer.symbollayer
[Pipeline]: /javascript/api/azure-maps-rest/atlas.service.pipeline
[event listener]: /javascript/api/azure-maps-control/atlas.map#events

[service module]: how-to-use-services-module.md
[calculateRouteDirections]: /javascript/api/azure-maps-rest/atlas.service.routeurl#methods
[LinestringLayerOptions]: /javascript/api/azure-maps-control/atlas.linelayeroptions
[Feature]: /javascript/api/azure-maps-control/atlas.data.feature
[points]: /javascript/api/azure-maps-control/atlas.data.point
