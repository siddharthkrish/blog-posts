# Announcing Grab Maps V2 API Support for Amazon Location Service in Southeast Asia

**Author:** [Your Name]
**Date:** [Publication Date]
**Tags:** Amazon Location Service, Grab Maps, ASEAN, Maps, Geospatial, Southeast Asia

---

Building location-aware applications in Southeast Asia comes with a unique set of challenges. Local roads, dense urban areas, hyperlocal points of interest, and address formats in countries like Indonesia, Thailand, Vietnam, and the Philippines require map data that truly reflects the region. That's why we're excited to announce that the **Amazon Location Service V2 APIs are now available for ASEAN customers using Grab Maps** — bringing a dramatically improved developer experience, new API capabilities, and the same high-quality Grab map data that millions of users across Southeast Asia rely on every day.

If you're building ride-hailing, food delivery, logistics, or any location-aware application serving customers in ASEAN, this announcement is for you.

---

## Background: Grab Maps and Amazon Location Service

[Amazon Location Service](https://aws.amazon.com/location/) makes it easy to add location functionality — maps, geocoding, routing, and geofencing — to your applications without compromising data security or user privacy. As part of the service, AWS offers a curated selection of map data providers so you can choose the data that's best suited to your geography and use case.

[Grab](https://www.grab.com/), Southeast Asia's leading superapp, provides high-quality map data specifically optimised for the ASEAN region. Since Grab began building GrabMaps in 2017, the platform has grown to power the Grab Superapp — covering food delivery, ride-sharing, and logistics services across Singapore, Malaysia, Thailand, Indonesia, Philippines, Vietnam, Myanmar, and Cambodia. GrabMaps takes a community-based approach to data collection, continuously validated by millions of orders and rides, giving it exceptional accuracy for local roads, address formats, and over 50 million POIs across the region.

When we launched the original Grab Maps integration with Amazon Location Service, customers could provision Maps, Place Index, and Route Calculator resources via AWS CloudFormation and use Grab as their data provider for their web and mobile applications. Today, we're building on that foundation with the V2 APIs, which make the entire experience dramatically simpler — you can go from zero to a working map in minutes, with no resource pre-provisioning required.

---

## What's New in V2

The V2 Amazon Location Service APIs represent a significant evolution in how developers interact with location services on AWS. Here are the headline changes that matter most for Grab Maps users.

### No More Pre-Provisioned Resources

The biggest change in V2 is that **resource creation is no longer required**. In V1, you first needed to deploy a CloudFormation stack to create named resources — a Map, Place Index, and Route Calculator — and configure a Cognito Identity Pool before making any API calls. Those resource names were then threaded through every call in your application code.

With V2, you simply create an API key and start making calls immediately. No CloudFormation, no Cognito pool, no pre-created resource names to manage. This dramatically reduces setup friction for new projects and makes it far easier to experiment, prototype, and iterate.

> **TODO:** Add a side-by-side code block comparing the V1 CloudFormation + Cognito setup vs. the V2 API key approach, showing how much simpler it is to get started.

> **TODO:** Add screenshot of the Amazon Location Service Console showing the new API keys management page.

### API Key Authentication

V2 introduces **API Key authentication** as a first-class option. API keys can be scoped to specific actions — for example, you might create a key that allows only `GetTile` and `GetStyleDescriptor` for your map-rendering frontend, and a separate key that allows `SearchText` and `SearchNearby` for your search service. For production applications, keys should be configured with an **Expire time** and **Referers** (allowed domains) to restrict usage. When you're ready to retire a key, deactivating it starts a 90-day deactivation period before permanent deletion — or you can bypass that window when you're certain it's no longer needed.

For ASEAN developers building web and mobile apps, API key support means a much simpler and more secure authentication story for client-side code, replacing the need for Cognito Identity Pools in many common use cases.

> **TODO:** Add screenshot of the API key creation flow in the AWS Console, showing the action scoping options (GetStaticMap, GetTile, Geocode, GetPlace, SearchNearby, SearchText, CalculateIsolines, SnapToRoads) and the Expire time / Referers fields.

---

## New Maps API Features

The V2 Maps API introduces two new capabilities alongside the existing map tile rendering: `GetStyleDescriptor` and `GetStaticMap`.

### GetStyleDescriptor — Build Interactive Maps Faster

`GetStyleDescriptor` allows you to retrieve the full style definition for a Grab map and use it directly as the style source in MapLibre GL JS. Instead of manually constructing a style object or hosting your own style file, a single URL call hands you everything MapLibre needs to render the map — tiles, fonts, sprites, and layer definitions included.

Grab Maps provides two professionally designed general-purpose styles for ASEAN:

| Style ID | Display Name | Description |
|---|---|---|
| `VectorGrabStandardLight` | Street Light | Clean, light-background map for daytime UIs |
| `VectorGrabStandardDark` | Street Dark | Dark-background map for night mode or low-light environments |

Both styles include detailed land use layers, area names, local road networks, and points of interest — designed specifically for Southeast Asian geographies.

> **TODO:** Add code block showing how to initialise a MapLibre GL JS map using `GetStyleDescriptor` with the `VectorGrabStandardLight` style and a V2 API key, rendering an interactive map centred on a city like Kuala Lumpur or Singapore.

> **TODO:** Add screenshot showing the `VectorGrabStandardLight` and `VectorGrabStandardDark` styles rendered side by side in a browser, centred on an ASEAN city.

### GetStaticMap — Generate Static Map Images

`GetStaticMap` is a new capability in V2 that generates a static map image based on coordinates, zoom level, and image dimensions you specify. This is ideal for embedding maps in emails, reports, printed materials, social media posts, or anywhere an interactive map isn't practical. You can also overlay custom data such as points, lines, and polygons on top of the map image.

Static maps are requested via a simple URL:

```
https://maps.geo.<Your AWS Region>.amazonaws.com/v2/static/map?center=<lon>,<lat>&zoom=<zoom>&width=<px>&height=<px>&key=<Your API Key>
```

> **TODO:** Add a code block or HTML `<img>` example using `GetStaticMap` with Grab map tiles, centred on a recognisable ASEAN location (e.g., the Marina Bay area in Singapore or the Grand Palace in Bangkok).

> **TODO:** Add screenshot of the static map image output rendered in a browser or embedded in an HTML page.

---

## New Places API Features

The V2 Places API introduces two new endpoints — `SearchText` and `SearchNearby` — backed by Grab's database of over 50 million addresses and POIs across Southeast Asia.

### SearchText — Find Places by Name or Category

`SearchText` allows your users to search for specific locations, addresses, or categories of places by sending a POST request with a text query and optional location bias. Results include rich POI data — names, addresses, categories, and coordinates — making it easy to display candidates on a map or populate a search results list.

This replaces the V1 `SearchPlaceIndexForText` call and removes the need for a pre-created Place Index resource.

> **TODO:** Add code block showing a `SearchText` POST request using the V2 API with a Grab data provider, querying for something locally relevant (e.g., "nasi lemak restaurants in Kuala Lumpur"), and rendering the results as map markers using MapLibre GL JS.

> **TODO:** Add screenshot of the search results rendered on a Grab map centred on an ASEAN city.

### SearchNearby — Discover POIs Around a Location

`SearchNearby` retrieves POI data around a specified coordinate — perfect for "find stores near me" or "what's nearby my delivery stop" use cases. Like `SearchText`, it returns rich candidate point data that can be visualised directly on a map.

> **TODO:** Add code block showing a `SearchNearby` POST request, passing a coordinate in an ASEAN city and a search radius, and rendering the nearby POI results as markers on a Grab map.

> **TODO:** Add screenshot of the `SearchNearby` results on a map — for example, showing nearby convenience stores or pharmacies around a point in Jakarta or Bangkok.

---

## New Routes API Features

The V2 Routes API adds two powerful new capabilities that are especially relevant for logistics and mobility applications in Southeast Asia: `CalculateIsolines` and `SnapToRoads`.

### CalculateIsolines — Visualise Reachable Areas

`CalculateIsolines` calculates the reachable area from a given point within a specified time or distance. The result is polygon data (an isoline, or isochrone) that you can render on a map to show users what's accessible from a location — in 5, 15, or 30 minutes, for example. Common use cases include visualising delivery coverage zones, helping customers understand service areas, or helping field teams evaluate property or depot locations.

> **TODO:** Add code block showing a `CalculateIsolines` POST request from a point in an ASEAN city (e.g., a warehouse in Jakarta), with 15- and 30-minute drive-time isolines, rendered as filled polygons on a Grab map.

> **TODO:** Add screenshot of the isoline polygons rendered on a Grab map in an ASEAN city.

### SnapToRoads — Correct GPS Traces to the Road Network

`SnapToRoads` is a new V2 Routes capability that takes raw GPS location data and corrects it to the nearest road in the Grab road network. This is especially valuable for ride-hailing and delivery applications where GPS readings from vehicles can drift off-road due to signal noise or urban canyon effects. By snapping traces to the road, you get cleaner route data for tracking, playback, and analysis.

> **TODO:** Add code block showing a `SnapToRoads` POST request with a series of GPS coordinates in an ASEAN city, and a MapLibre GL JS visualisation showing the raw GPS trace (in one colour) alongside the snapped route (in another colour).

> **TODO:** Add screenshot of the before/after route comparison — raw GPS trace vs. road-snapped route — rendered on a Grab map.

---

## Getting Started

Here's how to get up and running with Grab Maps and the Amazon Location Service V2 APIs.

### Prerequisites

- An AWS account with access to Amazon Location Service in the **AWS Asia Pacific (Singapore)** Region (`ap-southeast-1`)
- A modern browser or a JavaScript/mobile development environment
- MapLibre GL JS (version 3.x recommended) for interactive map rendering

> **Note:** Grab Maps data is designed for ASEAN customers. Coverage is optimised for Singapore, Malaysia, Thailand, Indonesia, Philippines, Vietnam, Myanmar, and Cambodia.

### Step 1: Create an API Key

Navigate to the Amazon Location Service Console, select **API keys** under **Manage resources**, and select **Create API key**. Give your key a name and select the actions you want to permit. For a full-featured Grab Maps integration, you'll want to enable:

- `GetStaticMap`, `GetTile`, `GetStyleDescriptor` — for map rendering
- `SearchText`, `SearchNearby`, `Geocode`, `GetPlace` — for search and geocoding
- `CalculateIsolines`, `SnapToRoads` — for routing features

For production use, configure an **Expire time** and lock the key to specific **Referers** (your app's domain). Once created, copy the API key value and store it securely.

> **TODO:** Add screenshot of the API key creation screen in the console, with the action checkboxes visible.

### Step 2: Render a Grab Map

> **TODO:** Add full HTML/JavaScript code block showing how to initialise a MapLibre GL JS map using the `GetStyleDescriptor` endpoint with the `VectorGrabStandardLight` style and your API key, centred on an ASEAN location such as Kuala Lumpur (`longitude: 101.656125, latitude: 3.1506347`).

> **TODO:** Add screenshot of the rendered map in a browser.

### Step 3: Add Place Search

> **TODO:** Add code block wiring a text input field to the `SearchText` endpoint, displaying results as markers on the map. Show how to add a secondary "Search Nearby" button that calls `SearchNearby` based on the map's current centre.

### Step 4: Add Routing and Isolines

> **TODO:** Add code block showing a route calculation between two points in Southeast Asia, then a `CalculateIsolines` call to show the reachable area from the destination — both rendered on the Grab map.

---

## Migrating from V1 to V2

If you're currently using the V1 Grab Maps integration with Amazon Location Service, here's what the migration looks like at a high level:

1. **Replace CloudFormation resources with an API key** — V1 required deploying a CloudFormation stack that created Map, Place Index, and Route Calculator resources, plus a Cognito Identity Pool for authentication. In V2, you simply create an API key in the console and reference it directly in your code.
2. **Update authentication** — Replace Cognito Identity Pool credential flow with the new API key. For server-side calls, IAM credentials remain supported.
3. **Update API method names** — Several V1 methods have been renamed in V2, for example `SearchPlaceIndexForText` → `SearchText` and `SearchPlaceIndexForSuggestions` → `SearchNearby`. Review the [V2 API reference](https://docs.aws.amazon.com/location/latest/APIReference/Welcome.html) for the complete mapping.
4. **Remove resource name references** — V1 required passing a named resource (e.g., `GrabDemoPlaceIndex`) in each API call. V2 API calls don't require this — the data provider is selected by the API key configuration.

> **TODO:** Add a before/after code block showing the V1 CloudFormation config + Cognito auth + resource-name-based API call side by side with the equivalent V2 API key + direct API call.

---

## Why Grab Maps for ASEAN?

No two regions are alike when it comes to maps. Southeast Asia presents unique mapping challenges that generic global providers often underserve:

- **Community-validated data** — Grab receives and validates daily updates from millions of rides and orders, with real-time feedback from drivers and delivery partners on road closures, business changes, and new POIs. The result is map data that reflects how Southeast Asia actually works, not how it looked six months ago.
- **ASEAN-specific transport modes** — Grab Maps supports routing for car, motorcycle, bicycle, and walking — including motorcycle routing that accounts for lane-splitting and local traffic rules common in markets like Vietnam and Indonesia.
- **Hyperlocal POI coverage** — With over 50 million addresses and POIs across the region, Grab Maps covers everything from major landmarks to small alleyways, local hawker stalls, and subdivisions that global providers frequently miss.
- **Local road network accuracy** — Grab Maps accounts for vehicle restrictions by licence plate, local road hierarchies, and transportation modes specific to Southeast Asia — critical for last-mile delivery and ride-hailing accuracy.
- **Operational trust** — Grab's data powers millions of trips and deliveries across ASEAN every day. When you use Grab Maps through Amazon Location Service, you're using the same underlying data that Grab relies on for its own products.

For developers building apps that serve ASEAN customers — whether in ride-hailing, logistics, food delivery, real estate, travel, or fintech — Grab Maps via Amazon Location Service V2 offers a compelling combination of data quality, developer experience, and AWS-native integration.

---

## Additional Resources

- [Amazon Location Service Documentation](https://docs.aws.amazon.com/location/)
- [Amazon Location Service V2 API Reference](https://docs.aws.amazon.com/location/latest/APIReference/Welcome.html)
- [Amazon Location Service Pricing](https://aws.amazon.com/location/pricing/)
- [Amazon Location Service GitHub — Sample Projects](https://github.com/aws-samples/amazon-location-samples)
- [MapLibre GL JS Documentation](https://maplibre.org/maplibre-gl-js/docs/)

---

## Conclusion

The launch of V2 API support for Grab Maps in Amazon Location Service is a major step forward for developers building location-aware applications in Southeast Asia. By eliminating the need for pre-provisioned resources, introducing API key authentication, and adding powerful new capabilities — `GetStyleDescriptor`, `GetStaticMap`, `SearchText`, `SearchNearby`, `CalculateIsolines`, and `SnapToRoads` — V2 makes it faster and simpler than ever to build with Grab's industry-leading ASEAN map data on AWS.

Whether you're starting a new project or migrating an existing V1 integration, we hope this makes Grab Maps on Amazon Location Service the easiest choice for your next ASEAN application.

Happy building! 🗺️

---

*This post is part of the AWS Mobile & Location series. For the latest updates on Amazon Location Service, subscribe to the [AWS Blog](https://aws.amazon.com/blogs/).*
