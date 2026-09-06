# PetZonic — Products API

> **Version**: 1.0.0  
> **Base URL**: `/api/v1/products`

---

## 1. Overview

The Products API manages the e-commerce store — product catalog, categories, variants, inventory, and search. Products are "own inventory" items sold directly by PetZonic (food, accessories, medicines) unlike the marketplace pets.

---

## 2. Endpoints

### 2.1 List Products

```
GET /api/v1/products
```

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | integer | 1 | Page number |
| limit | integer | 20 | Items per page (max 50) |
| category | string | — | Category slug filter |
| species | string | — | Target species (dog, cat, etc.) |
| minPrice | number | — | Minimum price (INR) |
| maxPrice | number | — | Maximum price (INR) |
| brand | string | — | Brand filter |
| sort | string | relevance | `relevance`, `price_asc`, `price_desc`, `newest`, `rating`, `popular` |
| inStock | boolean | true | Only show in-stock items |
| q | string | — | Search query (fuzzy matched via pg_trgm) |

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "prod_abc123",
      "name": "Royal Canin Maxi Adult",
      "slug": "royal-canin-maxi-adult-15kg",
      "description": "Premium nutrition for large breed adults",
      "shortDescription": "Large breed adult dry food",
      "brand": "Royal Canin",
      "category": {
        "id": "cat_food",
        "name": "Dog Food",
        "slug": "dog-food"
      },
      "targetSpecies": ["dog"],
      "images": [
        {
          "url": "https://cdn.petzonic.com/products/royal-canin-maxi.jpg",
          "alt": "Royal Canin Maxi Adult 15kg",
          "isPrimary": true
        }
      ],
      "price": {
        "mrp": 7500,
        "selling": 6200,
        "discount": 17,
        "currency": "INR"
      },
      "rating": {
        "average": 4.5,
        "count": 234
      },
      "variants": [
        { "id": "var_1", "label": "4kg", "price": 2800, "inStock": true },
        { "id": "var_2", "label": "15kg", "price": 6200, "inStock": true }
      ],
      "inStock": true,
      "isFeatured": true,
      "tags": ["large-breed", "adult", "dry-food"]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8
  },
  "filters": {
    "categories": [
      { "slug": "dog-food", "name": "Dog Food", "count": 45 }
    ],
    "brands": [
      { "name": "Royal Canin", "count": 12 }
    ],
    "priceRange": { "min": 99, "max": 15000 }
  }
}
```

---

### 2.2 Get Product Detail

```
GET /api/v1/products/:slug
```

**Response** `200 OK`:

```json
{
  "id": "prod_abc123",
  "name": "Royal Canin Maxi Adult",
  "slug": "royal-canin-maxi-adult-15kg",
  "description": "Complete and balanced nutrition...",
  "shortDescription": "Large breed adult dry food",
  "brand": "Royal Canin",
  "category": {
    "id": "cat_food",
    "name": "Dog Food",
    "slug": "dog-food",
    "breadcrumb": ["Pet Supplies", "Dog", "Dog Food", "Dry Food"]
  },
  "targetSpecies": ["dog"],
  "images": [
    {
      "url": "https://cdn.petzonic.com/products/royal-canin-maxi.jpg",
      "alt": "Royal Canin Maxi Adult 15kg",
      "isPrimary": true,
      "order": 1
    }
  ],
  "price": {
    "mrp": 7500,
    "selling": 6200,
    "discount": 17,
    "currency": "INR"
  },
  "variants": [
    {
      "id": "var_1",
      "sku": "RC-MAXI-4KG",
      "label": "4kg",
      "price": 2800,
      "mrp": 3200,
      "inStock": true,
      "stockCount": 45
    },
    {
      "id": "var_2",
      "sku": "RC-MAXI-15KG",
      "label": "15kg",
      "price": 6200,
      "mrp": 7500,
      "inStock": true,
      "stockCount": 22
    }
  ],
  "specifications": {
    "weight": "15kg",
    "lifeStage": "Adult (15 months - 5 years)",
    "breedSize": "Large (26-44 kg)",
    "ingredients": "Dehydrated poultry protein, maize..."
  },
  "rating": {
    "average": 4.5,
    "count": 234,
    "distribution": { "5": 120, "4": 70, "3": 25, "2": 12, "1": 7 }
  },
  "reviews": [
    {
      "id": "rev_1",
      "userName": "Rahul M.",
      "rating": 5,
      "title": "My dog loves it!",
      "comment": "Switched from another brand and the coat improved within weeks.",
      "createdAt": "2026-04-15T10:30:00Z",
      "helpful": 12
    }
  ],
  "relatedProducts": ["prod_def456", "prod_ghi789"],
  "inStock": true,
  "deliveryEstimate": "2-4 days",
  "returnPolicy": "7 day return if unopened",
  "createdAt": "2026-01-15T10:00:00Z",
  "updatedAt": "2026-05-01T14:30:00Z"
}
```

---

### 2.3 Search Products (via PostgreSQL `pg_trgm`)

```
GET /api/v1/products/search
```

**Query Parameters**:

| Param | Type | Description |
|-------|------|-------------|
| q | string | Search query (typo-tolerant) |
| category | string | Category filter |
| species | string | Target species |
| page | integer | Page number |
| limit | integer | Results per page |

**Response** `200 OK`:

```json
{
  "data": [...],
  "query": "royal canin",
  "processingTimeMs": 12,
  "pagination": { "page": 1, "limit": 20, "total": 8 }
}
```

---

### 2.4 Get Categories

```
GET /api/v1/products/categories
```

**Response** `200 OK`:

```json
{
  "data": [
    {
      "id": "cat_1",
      "name": "Dog Food",
      "slug": "dog-food",
      "icon": "https://cdn.petzonic.com/icons/dog-food.svg",
      "image": "https://cdn.petzonic.com/categories/dog-food.jpg",
      "parentId": "cat_dog",
      "productCount": 156,
      "children": [
        {
          "id": "cat_1a",
          "name": "Dry Food",
          "slug": "dry-food",
          "productCount": 89
        }
      ]
    }
  ]
}
```

---

### 2.5 Get Product Reviews

```
GET /api/v1/products/:productId/reviews
```

**Query Parameters**:

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| page | integer | 1 | Page number |
| limit | integer | 10 | Reviews per page |
| rating | integer | — | Filter by star rating |
| sort | string | newest | `newest`, `helpful`, `rating_high`, `rating_low` |

---

## 3. Admin Endpoints (Requires `admin` role)

### 3.1 Create Product

```
POST /api/v1/admin/products
```

**Headers**: `Authorization: Bearer <admin_token>`

**Body**:

```json
{
  "name": "Royal Canin Maxi Adult",
  "description": "Complete and balanced nutrition...",
  "shortDescription": "Large breed adult dry food",
  "brand": "Royal Canin",
  "categoryId": "cat_food",
  "targetSpecies": ["dog"],
  "images": [
    { "url": "https://cdn.petzonic.com/products/img1.jpg", "isPrimary": true }
  ],
  "variants": [
    {
      "sku": "RC-MAXI-4KG",
      "label": "4kg",
      "price": 2800,
      "mrp": 3200,
      "stock": 100
    }
  ],
  "specifications": {
    "weight": "4kg",
    "lifeStage": "Adult"
  },
  "tags": ["large-breed", "adult"],
  "isFeatured": false,
  "isActive": true
}
```

**Response** `201 Created`:

```json
{
  "id": "prod_new123",
  "slug": "royal-canin-maxi-adult-4kg",
  "message": "Product created successfully"
}
```

---

### 3.2 Update Product

```
PATCH /api/v1/admin/products/:id
```

**Body**: Partial product fields to update.

---

### 3.3 Delete Product (Soft Delete)

```
DELETE /api/v1/admin/products/:id
```

**Response** `200 OK`:

```json
{ "message": "Product deactivated" }
```

---

### 3.4 Manage Inventory

```
PATCH /api/v1/admin/products/:productId/variants/:variantId/stock
```

**Body**:

```json
{
  "adjustment": 50,
  "reason": "restock",
  "note": "New shipment received"
}
```

---

### 3.5 Bulk Import Products

```
POST /api/v1/admin/products/bulk-import
```

**Body**: `multipart/form-data` with CSV file.

**CSV Columns**: `name, sku, category, brand, price, mrp, stock, description`

**Response** `202 Accepted`:

```json
{
  "jobId": "job_import_123",
  "message": "Import queued. 150 products being processed.",
  "statusUrl": "/api/v1/admin/jobs/job_import_123"
}
```

---

## 4. Error Responses

| Status | Code | Description |
|--------|------|-------------|
| 404 | `PRODUCT_NOT_FOUND` | Product with given slug/id not found |
| 400 | `INVALID_VARIANT` | Variant ID doesn't exist for product |
| 400 | `OUT_OF_STOCK` | Requested quantity exceeds stock |
| 409 | `SKU_DUPLICATE` | SKU already exists |
| 422 | `VALIDATION_ERROR` | Missing/invalid required fields |

---

## 5. Webhooks (Internal Events)

| Event | Trigger | Consumer |
|-------|---------|----------|
| `product.created` | New product added | Outbox event log |
| `product.updated` | Product modified | Outbox event log |
| `product.stock_low` | Stock < threshold | Admin notification |
| `product.out_of_stock` | Stock = 0 | Remove from active listings |
