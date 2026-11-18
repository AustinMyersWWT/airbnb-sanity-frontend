# GROQ Query Structure - Sanity Data Fetching Guide

## Overview

This Airbnb clone uses **GROQ** (Graph-Relational Object Queries) to fetch data from Sanity.io. GROQ is a powerful, declarative query language that allows you to fetch structured data from your Sanity dataset with precise control over what fields you retrieve.

---

## Setup & Configuration

### Sanity Client Initialization
Located in `sanity.js`:

```javascript
import { createClient, createImageUrlBuilder } from "next-sanity"

const config = {
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET || "production",
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID,
  useCdn: process.env.NODE_ENV === "production",
  // useCdn: false for always fresh data (bypasses CDN for real-time fetches)
}

export const sanityClient = createClient(config)
```

**Key Configuration Points:**
- `useCdn`: Enables content delivery network caching in production for faster reads
- `projectId` & `dataset`: Environment variables that connect to your Sanity project
- Authenticated requests (like preview mode) bypass the CDN automatically

### Image URL Builder
```javascript
export const urlFor = (source) => createImageUrlBuilder(config).image(source)
```
Helper function to generate optimized image URLs from Sanity asset references.

---

## Query 1: Home Page - All Properties List

### Location
`pages/index.js` → `getServerSideProps`

### Query String
```javascript
const query = '*[ _type == "property"]'
const properties = await sanityClient.fetch(query)
```

### Structure Breakdown

| Component | Purpose |
|-----------|---------|
| `*` | Wildcard selector - matches all documents in the dataset |
| `[ _type == "property" ]` | Filter - selects only documents with type "property" |
| Results returned | **All fields** from all matching property documents |

### What Data is Fetched
This basic query returns **complete property objects** including:
- `_id` - Unique document ID
- `title` - Property name
- `slug` - URL-friendly identifier
- `mainImage` - Featured image asset reference
- `pricePerNight` - Rental price
- `reviews` - Array of review objects
- All other fields defined in the property schema
- (Nested references NOT automatically resolved)

### Usage in Component
```javascript
{properties.map((property) => (
  <div className="card">
    <img src={urlFor(property.mainImage)} />
    <p>{property.reviews.length} review{isMultiple(property.reviews.length)}</p>
    <h3>{property.title}</h3>
    <h3><b>£{property.pricePerNight}/per Night</b></h3>
  </div>
))}
```

### Performance Considerations
- ✅ **Lightweight**: Returns basic fields only
- ✅ **Fast**: Cached on CDN in production
- ⚠️ **Limited relationships**: Host and traveller data not resolved

---

## Query 2: Property Detail Page - Single Property with Related Data

### Location
`pages/property/[slug].js` → `getServerSideProps`

### Query String
```javascript
const query = `*[ _type == "property" && slug.current == $pageSlug][0]{
  title,
  location,
  propertyType,
  mainImage,
  images,
  pricePerNight,
  beds,
  bedrooms,
  description,
  host->{
    _id,
    name,
    slug,
    image
  },
  reviews[]{
    ...,
    traveller->{
      _id,
      name,
      slug,
      image
    }
  }
}`

const property = await sanityClient.fetch(query, { pageSlug })
```

### Structure Breakdown - Part by Part

#### 1. **Selection & Filtering**
```groql
*[ _type == "property" && slug.current == $pageSlug][0]
```
- `*` - Search all documents
- `[ _type == "property" && slug.current == $pageSlug ]` - Filter by type AND match slug parameter
- `[0]` - Get only the first result (single document)
- `$pageSlug` - Parameter placeholder, populated via `{ pageSlug }` in fetch call

#### 2. **Field Projection** 
```groql
{
  title,
  location,
  propertyType,
  mainImage,
  images,
  pricePerNight,
  beds,
  bedrooms,
  description,
  ...
}
```
Explicitly specify which fields to retrieve (rather than returning all fields).

#### 3. **Reference Resolution with `->` Operator**

##### Host Reference
```groql
host->{
  _id,
  name,
  slug,
  image
}
```
- `host` - A reference field pointing to a "host" document
- `->` - "Dereference operator" - follows the reference and retrieves the related document
- `{ _id, name, slug, image }` - Specify which fields from the host document to fetch
- **Result**: Instead of just the reference ID, you get the full host object with its data

**Before dereference**: `host: "abc123xyz"`
**After dereference**: 
```javascript
host: {
  _id: "abc123xyz",
  name: "John Doe",
  slug: { current: "john-doe" },
  image: { asset: {...} }
}
```

##### Reviews with Traveller References
```groql
reviews[]{
  ...,
  traveller->{
    _id,
    name,
    slug,
    image
  }
}
```
- `reviews[]` - Array field containing review objects
- `{}` - Projection applied to each review item
- `...` - Spread operator - includes all fields from the review document
- `traveller->{ ... }` - Dereference the traveller reference within each review

**Result**: Array of review objects with full traveller details:
```javascript
reviews: [
  {
    _key: "review1",
    _type: "object",
    rating: 5,
    text: "Great place!",
    traveller: {
      _id: "traveller1",
      name: "Jane Smith",
      slug: { current: "jane-smith" },
      image: { asset: {...} }
    }
  },
  ...
]
```

### Complete Data Structure Returned
```javascript
{
  title: "Cozy London Flat",
  location: { lat: 51.5074, lng: -0.1278 },
  propertyType: "Apartment",
  mainImage: { asset: {...}, hotspot: {...} },
  images: [{ asset: {...} }, ...],
  pricePerNight: 150,
  beds: 2,
  bedrooms: 1,
  description: "Beautiful flat in central London...",
  host: {
    _id: "host123",
    name: "John Doe",
    slug: { current: "john-doe" },
    image: { asset: {...} }
  },
  reviews: [
    {
      // ... review fields
      traveller: {
        _id: "traveller1",
        name: "Jane",
        slug: { current: "jane" },
        image: { asset: {...} }
      }
    }
  ]
}
```

### Usage in Component
```javascript
// Host info
<h2>{propertyType} hosted by {host?.name}</h2>

// Review details
{reviews.map((review) => (
  <Review key={review._key} review={review} />
))}

// Images gallery
{images.map(({ _key, asset }) => (
  <Image key={_key} image={asset} />
))}
```

---

## GROQ Query Language Syntax Reference

### Common Operators & Syntax

| Syntax | Purpose | Example |
|--------|---------|---------|
| `*` | Select all documents | `*` |
| `[ filter ]` | Filter documents | `[ _type == "property" ]` |
| `&&` | AND condition | `[ _type == "property" && active == true ]` |
| `\|\|` | OR condition | `[ _type == "property" \|\| _type == "hotel" ]` |
| `[0]` | Get first result | `*[...][0]` |
| `{}` | Field projection | `{ title, description }` |
| `->` | Dereference reference | `author->{ name, email }` |
| `[]` | Array selector | `reviews[]` |
| `...` | Spread/include all fields | `{ ..., custom_field }` |
| `$variable` | Parameter placeholder | `$pageSlug` |
| `@` | Accessor functions | Various functions like @[now](https://www.sanity.io/docs/query-cheat-sheet) |

### Parameter Passing

**Query with parameter:**
```javascript
const query = `*[ _type == "property" && slug.current == $pageSlug][0]`
const result = await sanityClient.fetch(query, { pageSlug })
```

Parameters prevent query injection and allow dynamic values.

---

## Data Flow: From Sanity to Component

```
┌─────────────────────────────────────────────────────────┐
│ 1. getServerSideProps triggered (during build or on-demand)
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│ 2. GROQ Query constructed with filters & projections
│    Example: *[ _type == "property" && slug.current == $slug ][0]{...}
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│ 3. sanityClient.fetch(query, params) executed
│    - Sends request to Sanity API
│    - Uses CDN in production (cached reads)
│    - Uses live API in development (fresh data)
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│ 4. Sanity returns structured JSON data
│    - References are dereferenced (->)
│    - Only requested fields included ({})
│    - Array items filtered/projected ([]{})
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│ 5. Data passed to Component via props
│    return { props: { property, host, reviews, ... } }
└─────────────────────┬───────────────────────────────────┘
                      │
┌─────────────────────▼───────────────────────────────────┐
│ 6. React Component renders with data
│    - Properties destructured from props
│    - Images URL-generated with urlFor()
│    - Data mapped & displayed in JSX
└─────────────────────────────────────────────────────────┘
```

---

## Performance & Best Practices

### 1. **Field Projection** ✅
Do this - only fetch what you need:
```javascript
{ title, pricePerNight, reviews }  // ✅ Efficient
```

Don't do this - fetch everything:
```javascript
*[ _type == "property" ]  // ⚠️ Returns all fields
```

### 2. **Reference Dereferencing** ✅
Use `->` to resolve relationships in one query:
```javascript
host->{ name, image }  // ✅ One query with all data
```

Instead of multiple queries:
```javascript
// ⚠️ Two queries needed
const property = await sanityClient.fetch(query1)
const host = await sanityClient.fetch(query2, { hostId: property.host })
```

### 3. **Array Operations** ✅
```javascript
reviews[]{
  ...,
  traveller->{ name }
}
```
Apply projections to each array item with `[]{}`.

### 4. **Caching Strategy**
- **Production**: `useCdn: true` - Cached results, faster but slightly stale
- **Development**: `useCdn: false` - Always fresh data
- **Preview mode**: Always bypasses CDN (authenticated requests)

### 5. **Parameterized Queries** ✅
Always use parameters for dynamic values:
```javascript
const query = `*[ slug.current == $slug ][0]`
await sanityClient.fetch(query, { slug: pageSlug })  // ✅ Safe
```

Avoid string interpolation:
```javascript
const query = `*[ slug.current == "${pageSlug}" ][0]`  // ⚠️ Risky
```

---

## Summary

| Feature | Query 1 (Home) | Query 2 (Detail) |
|---------|----------------|-----------------|
| **Complexity** | Simple | Advanced |
| **Documents** | All properties | Single property |
| **Filtering** | By type only | By type + slug |
| **References** | None resolved | Host + Travellers dereferenced |
| **Use Case** | List view, fast loading | Detail page, rich content |
| **Data Size** | Smaller (basic fields) | Larger (all related data) |

Both queries demonstrate GROQ's powerful ability to:
- Filter data precisely
- Dereference relationships efficiently
- Project only needed fields
- Support dynamic parameters
- Fetch normalized data in a single call
