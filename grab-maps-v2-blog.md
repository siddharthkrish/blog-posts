# Using Grab Maps with Amazon Location Service V2 APIs

**Author:** [Your Name]
**Date:** [Publication Date]
**Tags:** Amazon Location Service, Grab Maps, ASEAN, Maps, Geospatial, Southeast Asia, Migration

---

In 2023, we announced [GrabMaps as a data provider for Amazon Location Service](https://aws.amazon.com/blogs/mobile/building-high-quality-cost-effective-apps-in-southeast-asia-with-amazon-location-service/), giving developers in Southeast Asia access to Grab's high-quality regional map data — covering 50 million addresses and POIs across Singapore, Malaysia, Thailand, Indonesia, Philippines, Vietnam, Myanmar, and Cambodia — directly through AWS.

Today, we're announcing that **Grab Maps is now available with the Amazon Location Service V2 APIs**. The V2 APIs bring a significantly simpler developer experience: resource pre-provisioning and CloudFormation stacks are no longer required, and client-side authentication no longer depends on Amazon Cognito Identity Pools. You can now get a fully working Grab-powered map, search, and routing integration up and running with just an API key.

This post walks through what changed between V1 and V2, and shows you how to build the same Maps, Places, and Routes functionality you know from V1 — using the new, streamlined V2 approach.

---

## What Changed in V2

The core Grab Maps capabilities available through Amazon Location Service remain the same in V2 — two map styles (`VectorGrabStandardLight` and `VectorGrabStandardDark`), geocoding and place search across 50M+ ASEAN POIs, and route calculation with ASEAN-specific transport modes. What changed is how you set up and call those capabilities.

| | V1 | V2 |
|---|---|---|
| **Setup** | Deploy CloudFormation stack | Create an API key in the console |
| **Authentication** | Amazon Cognito Identity Pool | API key |
| **Named resources** | Required (Map, Place Index, Route Calculator) | Not required |
| **Map tiles** | Cognito credentials + resource name | API key |
| **Place search** | `SearchPlaceIndexForText` + resource name | `SearchText` |
| **Reverse geocoding** | `SearchPlaceIndexForPosition` + resource name | `ReverseGeocode` |
| **Autocomplete** | `SearchPlaceIndexForSuggestions` + resource name | `SearchText` (with suggest parameters) |
| **Route calculation** | `CalculateRoute` + resource name | `CalculateRoutes` |

The most meaningful change for developers is in setup time. V1 required you to deploy an AWS CloudFormation template, wait for it to complete, copy the Cognito Identity Pool ID and resource names from the stack outputs, and wire all of that into your app config. With V2, you create an API key in the console — a process that takes under two minutes — and you're ready to go.

---

## Getting Started with V2

### Prerequisites

- An AWS account with access to Amazon Location Service in the **AWS Asia Pacific (Singapore)** Region (`ap-southeast-1`)
- Node.js and npm installed for the web application walkthrough
- [MapLibre GL JS](https://maplibre.org/maplibre-gl-js/docs/) (v3.x) for interactive map rendering

### Step 1: Create an API Key

In V1, you would start by clicking a CloudFormation deployment link, waiting for the stack to provision a Map resource, a Place Index, a Route Calculator, a Cognito Identity Pool, and the associated IAM roles and policies — then copying the outputs into your app.

In V2, navigate to the [Amazon Location Service Console](https://console.aws.amazon.com/location/), select **API keys** under **Manage resources**, and select **Create API key**. Give your key a name (e.g., `GrabMapsDemo`) and enable the following actions for a full Maps, Places, and Routes integration:

- `GetTile` — render interactive map tiles
- `SearchText` — search for places and addresses
- `ReverseGeocode` — convert coordinates to addresses
- `GetPlace` — retrieve place details
- `CalculateRoutes` — calculate routes between points

For production applications, set an **Expire time** and add your application domain to **Referers** to restrict where the key can be used. Once created, copy the key value — this replaces both the Cognito Identity Pool ID and all named resource references from V1.

> **TODO:** Add screenshot of the API key creation screen in the AWS Console, showing the action checkboxes and the optional Expire time / Referers fields.

> **TODO:** Add screenshot of the completed API key console page showing the key value (partially redacted).

### Step 2: Configure Your Application

In V1, `src/configuration.js` looked like this — requiring a Cognito Identity Pool ID and a named resource for every service:

```javascript
// V1 configuration.js
export const IDENTITY_POOL_ID = "ap-southeast-1:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
export const REGION = "ap-southeast-1";

export const MAP = {
  NAME: "GrabDemoMap",
  STYLE: "VectorGrabStandardLight",
};
export const PLACE = "GrabDemoPlaceIndex";
export const ROUTE = "GrabDemoRouteCalculator";
export const GEOFENCE = "GrabDemoGeofenceCollection";
export const TRACKER = "GrabDemoTrackers";
export const TRACKER_SIMULATED_DEVICE = "Vehicle-1";
```

In V2, none of the named resources are needed. Your configuration becomes:

```javascript
// V2 configuration.js
export const API_KEY = "v1.public.xxxxxxxxxxxxxxxxxxxxxxxx"; // your API key value
export const REGION = "ap-southeast-1";
export const MAP_STYLE = "VectorGrabStandardLight";
```

The `MAP.NAME`, `PLACE`, `ROUTE`, `GEOFENCE`, and `TRACKER` fields are no longer referenced in API calls — they can be removed entirely.

### Step 3: Render an Interactive Grab Map

Both V1 and V2 use MapLibre GL JS to render interactive maps from Grab's vector tile service. The difference is in how you authenticate the tile requests.

In V1, you used `@aws/amazon-location-utilities-auth-helper` with a Cognito Identity Pool to sign map tile requests with temporary AWS credentials:

```javascript
// V1 — Cognito-based map initialisation
import { withIdentityPoolId } from "@aws/amazon-location-utilities-auth-helper";

const authHelper = await withIdentityPoolId(IDENTITY_POOL_ID);

const map = new maplibregl.Map({
  container: "map",
  style: `https://maps.geo.${REGION}.amazonaws.com/maps/v0/maps/${MAP.NAME}/style-descriptor`,
  ...authHelper.getMapAuthenticationOptions(),
  center: [101.656125, 3.1506347], // Kuala Lumpur
  zoom: 11,
});
```

In V2, you pass the API key directly as a URL parameter — no auth helper or Cognito setup needed:

> **TODO:** Add code block showing the V2 MapLibre GL JS map initialisation using the API key and `VectorGrabStandardLight` style, centred on Kuala Lumpur (`longitude: 101.656125, latitude: 3.1506347, zoom: 11`). The style URL format for V2 is `https://maps.geo.${REGION}.amazonaws.com/v2/styles/${MAP_STYLE}/descriptor?key=${API_KEY}`.

> **TODO:** Add screenshot of the rendered interactive Grab map (`VectorGrabStandardLight`) in a browser, centred on Kuala Lumpur.

### Step 4: Search for Places

V1 used `SearchPlaceIndexForText` against a named Place Index resource. V2 replaces this with `SearchText`, which takes the same query parameters but no longer requires a resource name.

**V1 — `SearchPlaceIndexForText`:**

```javascript
// V1 place search
import { LocationClient, SearchPlaceIndexForTextCommand } from "@aws-sdk/client-location";

const client = new LocationClient({ region: REGION, credentials: cognitoCredentials });

const response = await client.send(new SearchPlaceIndexForTextCommand({
  IndexName: PLACE,          // named resource required in V1
  Text: "Marina Bay Sands",
  BiasPosition: [101.656125, 3.1506347],
  MaxResults: 5,
}));
```

**V2 — `SearchText`:**

> **TODO:** Add code block showing the V2 `SearchText` call using the `@aws-sdk/client-geo-places` package (or equivalent V2 client), passing the API key and the same query parameters but without a resource name. Show how to render the returned results as markers on the MapLibre map.

> **TODO:** Add screenshot of `SearchText` results rendered as map markers on the Grab map — for example, searching for "coffee shops" near a point in Kuala Lumpur or Singapore.

### Step 5: Reverse Geocode a Location

V1 used `SearchPlaceIndexForPosition` to convert coordinates to an address. In V2 this is replaced by `ReverseGeocode`.

**V1 — `SearchPlaceIndexForPosition`:**

```javascript
// V1 reverse geocode
const response = await client.send(new SearchPlaceIndexForPositionCommand({
  IndexName: PLACE,          // named resource required in V1
  Position: [103.8198, 1.3521], // Singapore
}));
```

**V2 — `ReverseGeocode`:**

> **TODO:** Add code block showing the V2 `ReverseGeocode` call with API key authentication, passing a coordinate in Southeast Asia and handling the structured address response.

### Step 6: Calculate a Route

V1 used `CalculateRoute` against a named Route Calculator resource, supporting ASEAN-specific travel modes including car, walking, motorcycle, and bicycle. V2 replaces this with `CalculateRoutes` — same capabilities, no resource name required.

**V1 — `CalculateRoute`:**

```javascript
// V1 route calculation
const response = await client.send(new CalculateRouteCommand({
  CalculatorName: ROUTE,      // named resource required in V1
  DeparturePosition: [101.656125, 3.1506347],
  DestinationPosition: [103.8198, 1.3521],
  TravelMode: "Motorcycle",
  DepartNow: true,
}));
```

**V2 — `CalculateRoutes`:**

> **TODO:** Add code block showing the V2 `CalculateRoutes` call with API key authentication, using `TravelMode: "Motorcycle"` and a departure time, between two ASEAN coordinates. Show how to extract the route geometry and render the polyline on the Grab map using MapLibre GL JS.

> **TODO:** Add screenshot of the calculated route rendered on the Grab map between two points in Southeast Asia (e.g., Kuala Lumpur city centre to Petaling Jaya).

---

## Full V1 → V2 Migration Reference

If you have an existing V1 application using Grab Maps, the table below covers every change you need to make.

### Configuration changes

| V1 | V2 |
|---|---|
| CloudFormation stack deployment | Create API key in the console |
| `IDENTITY_POOL_ID` in config | `API_KEY` in config |
| `MAP.NAME` (resource name) | Not needed — remove |
| `PLACE` (resource name) | Not needed — remove |
| `ROUTE` (resource name) | Not needed — remove |
| `@aws/amazon-location-utilities-auth-helper` with Cognito | API key passed as URL parameter or SDK config |
| Cognito-signed tile requests | API key-authenticated tile requests |

### API method name changes

| V1 Method | V2 Method | Notes |
|---|---|---|
| `SearchPlaceIndexForText` | `SearchText` | Remove `IndexName` parameter |
| `SearchPlaceIndexForSuggestions` | `SearchText` (with suggest mode) | Remove `IndexName` parameter |
| `SearchPlaceIndexForPosition` | `ReverseGeocode` | Remove `IndexName` parameter |
| `GetPlace` | `GetPlace` | Updated request shape — remove `IndexName` |
| `CalculateRoute` | `CalculateRoutes` | Remove `CalculatorName` parameter |

### Map style URL changes

| V1 Style URL | V2 Style URL |
|---|---|
| `https://maps.geo.{region}.amazonaws.com/maps/v0/maps/{MapName}/style-descriptor` | `https://maps.geo.{region}.amazonaws.com/v2/styles/{StyleName}/descriptor?key={APIKey}` |

The Grab map style names (`VectorGrabStandardLight`, `VectorGrabStandardDark`) remain unchanged.

> **TODO:** Add a complete before/after code block showing the full `configuration.js` and map initialisation diff between V1 and V2 — the most self-contained reference for developers migrating an existing integration.

---

## What Stays the Same

Migrating from V1 to V2 changes the setup and authentication model, but the Grab Maps capabilities your application relies on are unchanged:

- **Map styles** — `VectorGrabStandardLight` and `VectorGrabStandardDark` are both available in V2, with the same visual design and regional detail.
- **Place data** — The same 50M+ addresses and POIs across Singapore, Malaysia, Thailand, Indonesia, Philippines, Vietnam, Myanmar, and Cambodia.
- **Transport modes** — Car, walking, motorcycle, and bicycle routing — including the ASEAN-specific modes that make Grab Maps particularly valuable for last-mile delivery and ride-hailing in the region.
- **Route parameters** — Departure time and avoidances (ferries, tolls, highways) are all supported in V2.
- **AWS Region** — Grab Maps remains available in the AWS Asia Pacific (Singapore) Region (`ap-southeast-1`).

---

## Additional Resources

- [Amazon Location Service Documentation](https://docs.aws.amazon.com/location/)
- [Amazon Location Service V2 API Reference](https://docs.aws.amazon.com/location/latest/APIReference/Welcome.html)
- [Amazon Location Service Pricing](https://aws.amazon.com/location/pricing/)
- [Amazon Location Service Sample Projects on GitHub](https://github.com/aws-samples/amazon-location-samples)
- [MapLibre GL JS Documentation](https://maplibre.org/maplibre-gl-js/docs/)
- [Original Grab Maps V1 Announcement](https://aws.amazon.com/blogs/mobile/building-high-quality-cost-effective-apps-in-southeast-asia-with-amazon-location-service/)

---

## Conclusion

Grab Maps on Amazon Location Service V2 delivers the same high-quality ASEAN map data you rely on today — without the setup overhead. By replacing CloudFormation stacks, Cognito Identity Pools, and named resources with a single API key, V2 makes it faster to start a new project and simpler to maintain an existing one.

If you're starting fresh, you can have an interactive Grab map with place search and routing running in your application in under 15 minutes. If you're migrating from V1, the changes are straightforward: swap your Cognito auth for an API key, update your API method names, and remove the resource name parameters from your calls — the Grab map data and capabilities stay exactly the same.

We'd love to hear what you build. Share your projects and feedback through the [AWS Developer Forums](https://forums.aws.amazon.com/) or reach out to your AWS account team.

Happy building! 🗺️

---

*This post is part of the AWS Mobile & Location series. For the latest updates on Amazon Location Service, subscribe to the [AWS Blog](https://aws.amazon.com/blogs/).*
