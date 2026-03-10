# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository contains RDF/Turtle documents describing OpenLink Virtuoso 8.x software license offers, pricing, and related metadata. The data is designed to be loaded into a Virtuoso SPARQL endpoint and consumed by the OpenLink shop and product catalog systems.

## Key Ontologies and Namespaces

All Turtle files use a consistent set of prefixes:

- `schema:` — schema.org vocabulary (Offer, Product, UnitPriceSpecification, etc.)
- `gr:` — GoodRelations (`http://purl.org/goodrelations/v1#`) for e-commerce semantics
- `oplic:` / `license:` — OpenLink license ontology (`http://www.openlinksw.com/ontology/licenses#`)
- `offers:` — OpenLink offers ontology (`http://www.openlinksw.com/ontology/offers#`)
- `oplsof:` — OpenLink software ontology (`http://www.openlinksw.com/ontology/software#`)
- `oplpro:` — OpenLink products ontology (`http://www.openlinksw.com/ontology/products#`)
- `wdrs:` — W3C POWDER-S (`wdrs:describedby` links licenses/offers back to their source document)

## Data Model

Each offer is composed of three tightly coupled entities:

1. **`schema:Offer`** — The offer itself, with `schema:validFrom`/`schema:validThrough`, `schema:price`, `schema:itemOffered` (pointing to the license), and `schema:priceSpecification` (pointing to UnitPriceSpecification).
2. **`schema:UnitPriceSpecification`** — Comes in two subtypes: `SpecialUnitPriceSpecification` (discounted) and `RetailUnitPriceSpecification`. Special price specs link to retail via `offers:hasRetailPriceSpecification`.
3. **`oplic:ProductLicense`** (also `schema:Product`) — The actual license object with `oplic:hasMaximumProcessorCores`, `oplic:hasSessions`, `oplic:hasDuration`, `oplic:hasLicenseModule`, `oplsof:hasOperatingSystemFamily`, and `oplsof:hasOperatingSystemType`.

The inverse relationship `oplic:partOf` / `schema:itemOffered` connects licenses back to offers.

## Named Graphs and Upload Targets

Two named graphs are used in the Virtuoso backend:

- `<urn:opl:shop:offering:sponging:cache:official>` — shop/cart system (loaded via `offers-upload-script-shop`)
- `<urn:data:openlink:products>` — product catalog / SEO (loaded via `offers-upload-script`)

## Loading Data into Virtuoso

Run these SQL scripts directly in Virtuoso's `isql` or Conductor interface:

```sql
-- Load into product catalog graph
-- (paste contents of offers-upload-script into isql)

-- Load into shop graph
-- (paste contents of offers-upload-script-shop into isql)
```

Both scripts use `SPARQL CLEAR GRAPH` first, then `SPARQL LOAD ... INTO` to reload from the live DAV URLs. The `define get:soft "no-sponge"` pragma bypasses the Sponger middleware.

To delete expired offers:
```sql
-- Run delete-expired-offers-from-shop-graph.sql in isql
```

## Offer Validity

Each offer has `schema:validFrom` and `schema:validThrough` timestamps in ISO 8601 format. When updating expiry dates, change both the `schema:Offer` and its associated `schema:UnitPriceSpecification` entities (both retail and special). Use `xsd:dateTime` typed literals throughout.

## File Organization

- Root-level `.ttl` files — primary offer definitions by product line/year (Virtuoso 8.x, OPAL, LOD, Enterprise)
- `Offers-authorizations-and-restrictions.ttl` — ACL groups, authorization rules, and OPAL-specific restrictions (separate from pricing)
- `aws-cloud-offers/` — AWS/Azure EC2/EBS cloud-specific offer descriptions
- `special-offers-listing-query.rq` — SPARQL SELECT to retrieve active offers (filters by `validThrough >= 2018-12-31`)
- `offers-upload-script` / `offers-upload-script-shop` — Virtuoso `isql` scripts to reload named graphs
- `delete-expired-offers-from-shop-graph.sql` — SPASQL cleanup for expired offers
- `manual-offers-loading-*.sql` — Alternative manual load scripts

## Editing Conventions

- Offer URIs follow the pattern: `http://data.openlinksw.com/oplweb/offer/Offer-{YYYY-MM}-{product-slug}#this`
- License URIs: `http://data.openlinksw.com/oplweb/license/License-{YYYY-MM}-{product-slug}#this`
- UnitPriceSpecification URIs: `http://data.openlinksw.com/oplweb/offer-unitprice/UnitPriceSpecification-{YYYY-MM}-{slug}-{retail|special}-price#this`
- The `rdfs:label` on offers is the short display name (e.g., `"Personal"`, `"Project"`, `"Workgroup"`)
- The `offers:offerNumber` string mirrors the slug portion of the offer URI
- `wdrs:describedby source:` must appear on both the offer and license to link them to their containing document
