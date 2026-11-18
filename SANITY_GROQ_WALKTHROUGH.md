# Sanity GROQ Query Structure Walkthrough

## Overview

This app queries data from Sanity CMS using GROQ (Graph Retrieval Object Query Language). Sanity is a headless CMS, meaning it stores content in a structured database that's accessed via APIs. GROQ is a powerful query language specifically designed for querying Sanity content.

---

## Setup & Configuration

### 1. **Sanity Client Setup** (`sanity.js`)

The app uses the `next-sanity` package to create a configured Sanity client:

```javascript
import { createClient, createImageUrlBuilder } from "next-sanity"

const config = {
  dataset: process.env.NEXT_PUBLIC_SANITY_DATASET || "production",
  projectId: process.env.NEXT_PUBLIC_SANITY_PROJECT_ID,
  useCdn: process.env.NODE_ENV === "production",
}

export const sanityClient = createClient(config)
```

**Key Configuration Points:**
- **`dataset`**: Which Sanity dataset to query (e.g., "production", "staging")
- **`projectId`**: Your Sanity project ID (stored in environment variables for security)
- **`useCdn`**: Uses Sanity's CDN in production for faster queries; disabled in dev for fresh data

---

## Query #1: Get All Properties (Homepage)

**File:** `pages/index.js`

```javascript
export const getServerSideProps = async () => {
  const query = '*[ _type == "property"]'
  const properties = await sanityClient.fetch(query)
  // ...
}
```

### Breaking Down This Query:

```
*[ _type == "property"]
```

| Part | Meaning |
|------|---------|
| `*` | **Wildcard selector** - Start with all documents in the dataset |
| `[` `]` | **Filter operator** - Apply conditions inside the brackets |
| `_type == "property"` | **Filter condition** - Only return documents where `_type` equals "property" |

**Plain English:** "Get all documents from Sanity where the document type is 'property'"

### What Gets Returned:

When no specific fields are requested, Sanity returns **all fields** by default from each property document:
- `_id`, `_type`, `_createdAt`, `_updatedAt`
- `title`, `slug`, `mainImage`, `pricePerNight`, `reviews`
- Any other fields defined in the property schema

---

## Query #2: Get Single Property by Slug (Detail Page)

**File:** `pages/property/[slug].js`

```javascript
export const getServerSideProps = async (pageContext) => {
  const pageSlug = pageContext.query.slug

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
  // ...
}
```

### Breaking Down This Complex Query:

#### **1. Initial Filter:**
```
*[ _type == "property" && slug.current == $pageSlug][0]
```

| Part | Meaning |
|------|---------|
| `*` | Start with all documents |
| `_type == "property"` | Filter: Document type is "property" |
| `&&` | **AND operator** - Both conditions must be true |
| `slug.current == $pageSlug` | Filter: Slug matches the dynamic route parameter |
| `[0]` | **Array indexing** - Get only the first result (there should be only one) |
| `{ ... }` | **Projection** - Specify which fields to return |

**Plain English:** "Get the first property document where type is 'property' AND its slug matches the page parameter, and return only these specific fields"

#### **2. Parameter Substitution:**
```javascript
sanityClient.fetch(query, { pageSlug })
```

The `$pageSlug` is a **placeholder** that gets replaced with the actual value from `pageContext.query.slug`. This prevents injection attacks and makes queries safer.

---

### Field Projection Breakdown:

#### **Simple Fields:**
```groq
title,
location,
propertyType,
pricePerNight,
beds,
bedrooms,
description,
mainImage,
images
```
These return the field value directly.

#### **Referenced Object - Dereferencing with `->` :**
```groq
host->{
  _id,
  name,
  slug,
  image
}
```

**What's happening:**
- `host` is a **reference field** - it stores a pointer to another document (the host user)
- The `->` operator **dereferences** the reference, meaning: "follow the reference and fetch the actual document"
- Inside the braces, we specify which fields from the referenced host document to return
- **Result:** Instead of just getting `host: { _ref: "user123" }`, we get the full host object:
  ```json
  {
    "host": {
      "_id": "user123",
      "name": "John Doe",
      "slug": "john-doe",
      "image": { /* image data */ }
    }
  }
  ```

#### **Array of Referenced Objects - `[]` with dereferencing:**
```groq
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

**What's happening:**
- `reviews[]` means: "get the array and iterate over all items"
- `...` (**spread operator**) means: "include all fields from each review"
- `traveller->{ ... }` means: "dereference the traveller reference and include only these fields"
- **Result:** An array of review objects with all review data + the dereferenced traveller info:
  ```json
  {
    "reviews": [
      {
        "_key": "review1",
        "text": "Great place!",
        "rating": 5,
        "traveller": {
          "_id": "user456",
          "name": "Jane Smith",
          "slug": "jane-smith",
          "image": { /* image data */ }
        }
      },
      { /* more reviews */ }
    ]
  }
  ```

---

## Key GROQ Concepts Used in This App

| Concept | Symbol | Example | Explanation |
|---------|--------|---------|-------------|
| **Wildcard Selector** | `*` | `*[_type == "property"]` | Get all documents matching criteria |
| **Filter** | `[ ]` | `[_type == "property"]` | Filter documents by condition |
| **Logical AND** | `&&` | `[type == "A" && status == "active"]` | Both conditions must be true |
| **Dereference** | `->` | `host->{name}` | Follow a reference to get actual document |
| **Array Iteration** | `[]` | `reviews[]{}` | Process each item in an array |
| **Spread Operator** | `...` | `{..., extraField}` | Include all existing fields plus new ones |
| **Array Indexing** | `[0]` | `results[0]` | Get first item (index 0) |
| **Projection** | `{ }` | `{title, author}` | Specify which fields to return |
| **Parameter** | `$name` | `slug.current == $pageSlug` | Placeholder for injected values |

---

## Data Flow in the App

### Homepage (`/`):
```
1. User visits homepage
   ↓
2. Next.js calls getServerSideProps
   ↓
3. GROQ Query: *[_type == "property"]
   ↓
4. Sanity returns all properties (all fields)
   ↓
5. Component receives properties as props
   ↓
6. Maps through and displays each property card
   ↓
7. User can click on a property to view details
```

### Property Detail (`/property/[slug]`):
```
1. User clicks on property (navigates to /property/my-apartment)
   ↓
2. Next.js calls getServerSideProps with slug = "my-apartment"
   ↓
3. GROQ Query: *[_type == "property" && slug.current == $pageSlug][0]{ ... }
   ↓
4. Sanity returns single property with:
   - Basic fields (title, location, etc.)
   - Dereferenced host object
   - Reviews array with dereferenced traveller objects
   ↓
5. Component receives props with fully populated data
   ↓
6. Renders property details with host info and reviews
```

---

## Why This Structure Works

### 1. **Efficiency**
- Only fetch fields you need (projection) - saves bandwidth
- Dereference data at query time instead of doing multiple requests

### 2. **Type Safety**
- Filters ensure you get the right document type
- Parameters prevent injection attacks

### 3. **Nested Data Access**
- The `->` operator lets you access related documents in one query
- No need for separate API calls to fetch host info or traveller data

### 4. **Dynamic Routes**
- The `[slug].js` file creates dynamic routes
- Parameters like `$pageSlug` make queries flexible and reusable

---

## Example Data Structure (What Sanity Returns)

### Single Property Document:
```json
{
  "_id": "property-123",
  "_type": "property",
  "title": "Cozy Apartment in London",
  "slug": {
    "current": "cozy-apartment-london"
  },
  "location": "London, UK",
  "propertyType": "Apartment",
  "pricePerNight": 120,
  "beds": 2,
  "bedrooms": 1,
  "description": "Beautiful apartment...",
  "mainImage": { "asset": { "_ref": "image-xyz" } },
  "images": [{ "asset": { "_ref": "image-abc" } }],
  "host": {
    "_id": "host-456",
    "name": "Alice Johnson",
    "slug": "alice-johnson",
    "image": { "asset": { "_ref": "image-host" } }
  },
  "reviews": [
    {
      "_key": "review1",
      "text": "Amazing stay!",
      "rating": 5,
      "traveller": {
        "_id": "user-789",
        "name": "Bob Smith",
        "slug": "bob-smith",
        "image": { "asset": { "_ref": "image-user" } }
      }
    }
  ]
}
```

---

## Common GROQ Patterns for Next Steps

If you wanted to extend this app, here are common GROQ patterns:

### Get properties filtered by price:
```groq
*[_type == "property" && pricePerNight < 200]
```

### Get properties sorted by price:
```groq
*[_type == "property"] | order(pricePerNight asc)
```

### Count total reviews for a property:
```groq
*[_type == "property" && slug.current == $slug][0] {
  title,
  "reviewCount": count(reviews[])
}
```

### Get all properties with excellent reviews:
```groq
*[_type == "property" && reviews[].rating >= 4.5]
```

---

## Summary

The app uses **two GROQ queries**:

1. **Simple query** (homepage): Fetch all properties quickly
2. **Complex query** (detail page): Fetch one property with dereferenced relationships (host + travellers)

GROQ's power comes from its ability to:
- ✅ Filter documents
- ✅ Project specific fields
- ✅ Dereference relationships
- ✅ Use parameters safely
- ✅ Process arrays with iteration

This structure allows the app to build fast, dynamic real estate marketplace pages without making multiple API calls!
