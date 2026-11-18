# GROQ Query Comparison - Visual Examples

## Quick Reference Guide

### Query 1: Home Page - Get All Properties

```javascript
// Location: pages/index.js

const query = '*[ _type == "property"]'
const properties = await sanityClient.fetch(query)
```

**Visual Breakdown:**
```
┌──────────────────────────────────────────┐
│ GROQ Query: *[ _type == "property"]      │
├──────────────────────────────────────────┤
│ *                = "Select everything"   │
│ [ ... ]          = "Apply filter"        │
│ _type == ...     = "Where type equals"   │
│ "property"       = "the string property" │
└──────────────────────────────────────────┘

Returns: [ {...}, {...}, {...} ]  // All property documents
```

**What You Get:**
```javascript
[
  {
    _id: "prop1",
    _type: "property",
    title: "Cozy London Flat",
    slug: { current: "cozy-london-flat" },
    mainImage: { asset: { _ref: "image-123" }, ... },
    reviews: [ { _key: "review1", text: "..." }, ... ],
    pricePerNight: 150,
    // ... all other fields from property schema
  },
  {
    _id: "prop2",
    // ... another property
  }
]
```

**How It's Used:**
```javascript
{properties.map((property) => (
  <div className="card">
    <img src={urlFor(property.mainImage)} />        // Asset URL generator
    <p>{property.reviews.length} reviews</p>         // Array length
    <h3>{property.title}</h3>                        // String
    <h3>£{property.pricePerNight}/per Night</h3>    // Number
  </div>
))}
```

---

### Query 2: Property Detail - Get Single Property with References

```javascript
// Location: pages/property/[slug].js

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

---

## Query 2 - Detailed Breakdown

### Step 1: Filter & Select Document

```javascript
*[ _type == "property" && slug.current == $pageSlug ][0]
```

```
┌────────────────────────────────────────────────────┐
│ *                                                  │
│ "Search in ALL documents across the dataset"       │
│                                                    │
│ ↓                                                  │
│ [ _type == "property" && slug.current == $pageSlug]
│ "Filter documents where:                           │
│   1. Type equals 'property'                        │
│   2. AND slug.current equals the parameter"        │
│                                                    │
│ ↓                                                  │
│ [0]                                                │
│ "Take only the FIRST matching result"              │
└────────────────────────────────────────────────────┘
```

**In Practice:**
```javascript
// User visits: /property/cozy-london-flat
const pageSlug = "cozy-london-flat"

// Query becomes: *[ _type == "property" && slug.current == "cozy-london-flat"][0]
// Returns: ONE property document (or null if not found)
```

---

### Step 2: Field Projection - Only Get What You Need

```javascript
{
  title,           // String field
  location,        // Object field { lat, lng }
  propertyType,    // String field
  mainImage,       // Image field (asset reference)
  images,          // Array of images
  pricePerNight,   // Number field
  beds,            // Number field
  bedrooms,        // Number field
  description,     // Text field
  host->{...},     // Reference field → DEREFERENCED
  reviews[]{...}   // Array field with projections
}
```

**Without Projection (returns everything):**
```javascript
*[ _type == "property" && slug.current == $pageSlug ][0]
// Result includes: all schema fields + internal Sanity fields
```

**With Projection (returns only specified fields):**
```javascript
*[ _type == "property" && slug.current == $pageSlug ][0]{
  title, location, pricePerNight, host->{ name }
}
// Result includes: ONLY these fields
```

---

### Step 3: Reference Dereferencing with `->` Operator

#### Simple Reference Example: Host

**Without Dereferencing:**
```javascript
host  // Just the reference ID
```
**Result:**
```javascript
host: "user-123-xyz"  // Just a string ID, not useful
```

**With Dereferencing:**
```javascript
host->{
  _id,
  name,
  slug,
  image
}
```
**Result:**
```javascript
host: {
  _id: "user-123-xyz",
  name: "John Doe",
  slug: { current: "john-doe" },
  image: { asset: {...} }
}

// Now you have all the host's data!
// Used in component: {host.name}
```

---

#### Complex Reference Example: Reviews with Traveller References

**Query Structure:**
```javascript
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

**Breaking Down the Syntax:**

```
reviews[]           = "The reviews array, apply following to each item"
  {                 = "Start projection"
    ...,            = "Include ALL review fields (spread operator)"
    traveller->{    = "Also dereference the traveller reference"
      _id,
      name,
      slug,
      image
    }
  }
```

**Result:**
```javascript
reviews: [
  {
    _key: "review-1",
    _type: "review",
    rating: 5,
    text: "Amazing place!",
    createdAt: "2023-10-15",
    
    // The dereferenced traveller object:
    traveller: {
      _id: "traveller-456",
      name: "Jane Smith",
      slug: { current: "jane-smith" },
      image: { asset: {...} }
    }
  },
  {
    _key: "review-2",
    rating: 4,
    text: "Great experience",
    traveller: {
      _id: "traveller-789",
      name: "Bob Johnson",
      slug: { current: "bob-johnson" },
      image: { asset: {...} }
    }
  }
]
```

---

## Schema Relationships

### Data Model Structure

```
┌──────────────────────────────────────────────────────────┐
│ Property Document                                        │
├──────────────────────────────────────────────────────────┤
│ Fields:                                                  │
│ • title: string                                          │
│ • slug: { current: string }                              │
│ • mainImage: image asset                                 │
│ • images: [ image assets ]                               │
│ • description: text                                      │
│ • pricePerNight: number                                  │
│ • beds: number                                           │
│ • bedrooms: number                                       │
│ • location: { lat: number, lng: number }                │
│ • propertyType: string                                   │
│                                                          │
│ • host: reference → ┐ (just ID without ->)              │
│ • reviews: array  ─────────────────────────┐            │
│                                            │            │
└────────────────────────────────────────────┼────────────┘
                                             │
                    ┌────────────────────────┴───┐
                    │                            │
          ┌─────────▼────────────┐  ┌───────────▼─────────┐
          │ Host/User Document   │  │ Review Object       │
          ├──────────────────────┤  ├─────────────────────┤
          │ • _id: string        │  │ • _key: string      │
          │ • name: string       │  │ • text: string      │
          │ • slug: {...}        │  │ • rating: number    │
          │ • image: asset       │  │ • traveller: ref ──┐│
          │                      │  └─────────────────────┘│
          └──────────────────────┘                         │
                                                           │
                                    ┌──────────────────────┘
                                    │
                          ┌─────────▼──────────────┐
                          │ Traveller/User Doc     │
                          ├────────────────────────┤
                          │ • _id: string          │
                          │ • name: string         │
                          │ • slug: {...}          │
                          │ • image: asset         │
                          └────────────────────────┘
```

---

## Query Execution Timeline

```
User Navigation Flow:
│
└─ Visits: /property/cozy-london-flat
   │
   └─ Next.js triggers: getServerSideProps({ query: { slug: "cozy-london-flat" } })
      │
      └─ GROQ Query Executes:
         │
         ├─ *[ _type == "property" ...]
         │  ↓
         │  Finds property in Sanity dataset
         │
         ├─ && slug.current == $pageSlug
         │  ↓
         │  Filters to matching slug
         │
         ├─ [0]
         │  ↓
         │  Gets first (only) result
         │
         ├─ { title, location, ... host->{...} }
         │  ↓
         │  Projects specific fields
         │
         └─ Dereference References:
            ├─ host->{ name, image }
            │  ↓
            │  Fetches full host document
            │
            └─ reviews[]{ ... traveller->{...} }
               ↓
               For each review, fetch traveller document
      │
      └─ Returns Complete Data Object:
         {
           title: "...",
           host: { name: "...", image: {...} },
           reviews: [{ traveller: {...} }, ...]
         }
      │
      └─ Passed to Property Component
         │
         └─ Component Renders:
            ├─ {host.name} ← Uses dereferenced host
            ├─ {reviews.map(r => r.traveller.name)} ← Uses dereferenced travellers
            └─ {title}, {pricePerNight}, etc. ← Uses direct fields
```

---

## Performance Metrics

### Query 1: All Properties

```javascript
const query = '*[ _type == "property"]'
```

**Characteristics:**
- ✅ **Speed**: Very fast (simple filter)
- ✅ **Caching**: Excellent (CDN friendly)
- ⚠️ **Data Size**: Medium (all fields in all properties)
- 📊 **Network**: ~50KB per 10 properties (depends on schema)

**Best For:** List views, dashboards

---

### Query 2: Single Property with References

```javascript
const query = `*[ _type == "property" && slug.current == $pageSlug][0]{
  ...,
  host->{ ... },
  reviews[]{ ..., traveller->{ ... } }
}`
```

**Characteristics:**
- ✅ **Speed**: Fast (single document + dereferences)
- ✅ **Caching**: Good (single item, predictable)
- ✅ **Data Size**: Optimized (field projection)
- 📊 **Network**: ~20-50KB (one property + references)
- ⭐ **Benefit**: All related data in ONE query (no N+1 problem)

**Best For:** Detail pages, rich content

---

## Common GROQ Patterns

### Pattern 1: Count Documents
```javascript
'count(*[ _type == "property" ])'
// Returns: 42
```

### Pattern 2: Filter by Multiple Conditions
```javascript
'*[ _type == "property" && pricePerNight < 200 && beds >= 2 ]'
// Returns: properties under £200 with 2+ beds
```

### Pattern 3: Sort Results
```javascript
'*[ _type == "property" ] | order(pricePerNight asc)'
// Returns: properties sorted by price ascending
```

### Pattern 4: Limit Results
```javascript
'*[ _type == "property" ][0..9]'
// Returns: first 10 properties
```

### Pattern 5: Nested Dereferences
```javascript
'*[ _type == "review" ]{ ..., traveller->{ name, host->{ name } } }'
// Returns: reviews with traveller and traveller's host data
```

### Pattern 6: Array Filtering
```javascript
'*[ _type == "property" ]{ ..., reviews[][ rating > 4 ] }'
// Returns: properties with only 5-star reviews
```

---

## Troubleshooting

### Issue: Getting all fields instead of projected ones

**Problem:**
```javascript
const query = '*[ _type == "property" && slug.current == $pageSlug][0]'
// Returns: ALL fields (heavy payload)
```

**Solution:**
```javascript
const query = `*[ _type == "property" && slug.current == $pageSlug][0]{
  title,
  pricePerNight,
  host->{ name }
}`
// Returns: ONLY specified fields (optimized payload)
```

### Issue: Host data is null or just an ID

**Problem:**
```javascript
const query = '*[ _type == "property" ][0]{ host }'
// Returns: host: "user-123" (just ID, not useful)
```

**Solution:**
```javascript
const query = '*[ _type == "property" ][0]{ host->{ _id, name } }'
// Returns: host: { _id: "user-123", name: "John" } (full data)
```

### Issue: N+1 query problem (fetching review travellers separately)

**Problem:**
```javascript
// Get property
const property = await fetch(query1)
// Then for each review, fetch traveller separately ❌
for (const review of property.reviews) {
  review.traveller = await fetch(query2, { travellerId: review.traveller })
}
```

**Solution:**
```javascript
// Fetch everything in one query with dereferences ✅
const query = `*[ _type == "property" ][0]{
  ...,
  reviews[]{ ..., traveller->{ _id, name } }
}`
```

---

## Summary Table

| Aspect | Query 1 (All Properties) | Query 2 (Single with References) |
|--------|--------------------------|----------------------------------|
| **Scope** | `*[ filter ]` | `*[ filter ][0]` |
| **Fields** | All returned | Projected with `{}` |
| **References** | Not dereferenced | Dereferenced with `->` |
| **Arrays** | Basic array fields | Advanced `[]{ }` syntax |
| **Parameters** | Static query | Dynamic with `$param` |
| **Use Case** | Listing/Dashboard | Detail page |
| **Response Size** | Larger | Optimized |
| **Queries** | 1 query | 1 query (no N+1) |
