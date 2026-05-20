# Insights Hub Asset Management API Integration Guide

## Key Endpoints

### Base Path
```
/api/assetmanagement/v3
```

### Endpoints for idlmatcher Flow

#### 1. Get a Specific Asset
```
GET /api/assetmanagement/v3/assets/{id}
```
Returns the asset with all static properties and HATEOAS links.

**Response includes:**
- `_links.children.href` - URL to query child assets
- `_links.aspects.href` - URL to get aspects of this asset

#### 2. List All Assets with Filtering
```
GET /api/assetmanagement/v3/assets?filter=<filter>
```

**Filter Examples:**
- Search by parent: `{ "parentId": "ce68c8847eee4c8f89ba33f257349040" }`
- Search by type: `{ "typeId": "mandal.ship" }`
- Inherited type: `{ "hasType": "mandal.ship" }`

#### 3. Get Child Assets (via HATEOAS link)
The response from `GET /assets/{id}` includes a `_links.children.href` property containing the URL to fetch child assets.

**Or directly:** 
```
GET /api/assetmanagement/v3/assets?filter={"parentId":"ce68c8847eee4c8f89ba33f257349040"}
```

#### 4. Get All Aspects of an Asset
```
GET /api/assetmanagement/v3/assets/{id}/aspects
```

**Query Parameters:**
- `page` - Page number (default: 0)
- `size` - Items per page (default: 20)
- `sort` - Sort field
- `filter` - Filter criteria
- `includeShared` - Include shared assets (true/false)

#### 5. Get Variables of an Asset
```
GET /api/assetmanagement/v3/assets/{id}/variables
```

## Asset Structure

An asset contains:
- `id` - Asset ID (unique identifier)
- `name` - Asset name
- `description` - Asset description
- `typeId` - Type of the asset
- `aspects` - Array of aspects with values
- `_links` - HATEOAS links (self, parent, children, aspects, variables)

## Aspect Structure

An aspect contains:
- `name` - Aspect name (e.g., "Ferrero_Alba_Lines")
- `variables` - Key-value pairs of aspect data
- `aspectTypeId` - Type definition of the aspect

## Task: Asset Mapping Logic

**Filename to Asset Name Matching:**

Pattern: `counter_<asset_name>.json`

Examples:
- File: `counter_linea_t.16_tart.rocher__m.6.json`
- Asset name: `LINEA_T.16_TART.ROCHER_(M.6)`
- Transformation: Remove "counter_" prefix, remove ".json" suffix, replace underscores with dots/spaces, restore brackets

## Flow Integration Points

1. **Get Parent Asset** → Extract `_links.children.href`
2. **Query Child Assets** → List all children
3. **Build Mapping** → Create name-to-ID object
4. **For each child**: Get aspects with name "Ferrero_Alba_Lines"
5. **Send Data** → POST aspect data using matched asset ID
