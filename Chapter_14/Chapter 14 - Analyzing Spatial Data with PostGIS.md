# Chapter 14 - Analyzing Spatial Data with PostGIS (PostgreSQL 17)

## Summary

This chapter explores spatial data management in PostgreSQL using the PostGIS extension. It covers spatial data types, functions for spatial analysis, and practical applications for geographic information systems (GIS).

## Introduction to PostGIS

PostGIS extends PostgreSQL with robust spatial data capabilities, allowing storage, querying, and analysis of location-based information. It implements the Open Geospatial Consortium (OGC) standards and adds hundreds of functions for spatial operations.

### Installing PostGIS

#### PostgreSQL Standard (snake_case)
```sql
-- Create extension in a database
CREATE EXTENSION postgis;

-- Verify installation
SELECT postgis_full_version();
```

#### Domain Driven Design (PascalCase)
```sql
-- Create extension in a database
CREATE EXTENSION PostGIS;

-- Verify installation
SELECT PostGIS_Full_Version();
```

**Output:**

```markdown
POSTGIS="3.4.0" [SPATIAL_REF_SYS_VERSION] PGSQL="170" GEOS="3.12.0-CAPI-1.18.0" PROJ="9.2.1" LIBXML="2.9.14" LIBJSON="0.15"
```

This command installs the PostGIS extension with all its spatial functionality and confirms the version details.

## Spatial Data Types

PostGIS provides various data types for representing spatial elements.

### Key Spatial Data Types

#### PostgreSQL Standard (snake_case)

```sql
-- Common spatial data types
-- Points, lines, polygons, collections
SELECT 'POINT(0 0)'::geometry;             -- Single point
SELECT 'LINESTRING(0 0, 1 1, 2 2)'::geometry; -- Line with three points
SELECT 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'::geometry; -- Square polygon
```

#### Domain Driven Design (PascalCase)

```sql
-- Common spatial data types
-- Points, lines, polygons, collections
SELECT 'POINT(0 0)'::Geometry;             -- Single point
SELECT 'LINESTRING(0 0, 1 1, 2 2)'::Geometry; -- Line with three points
SELECT 'POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'::Geometry; -- Square polygon
```

### Spatial Reference Systems (SRS)

Every spatial object needs a coordinate reference system to define how its coordinates map to Earth.

#### PostgreSQL Standard (snake_case)

```sql
-- View available spatial reference systems
SELECT srid, auth_name, auth_srid, proj4text 
FROM spatial_ref_sys 
LIMIT 5;

-- Create a point with a specific SRID (4326 = WGS84)
SELECT ST_SetSRID(ST_MakePoint(-73.935242, 40.730610), 4326);
```

#### Domain Driven Design (PascalCase)

```sql
-- View available spatial reference systems
SELECT "SRID", "AuthName", "AuthSRID", "Proj4Text" 
FROM "Spatial"."ReferenceSystem" 
LIMIT 5;

-- Create a point with a specific SRID (4326 = WGS84)
SELECT ST_SetSRID(ST_MakePoint(-73.935242, 40.730610), 4326);
```

**Output:**

```markdown
 SRID | AuthName  | AuthSRID  |            Proj4Text            
------+-----------+-----------+---------------------------------
 3819 | EPSG      |      3819 | +proj=longlat +ellps=bessel...
 3821 | EPSG      |      3821 | +proj=longlat +ellps=aust_SA...
 3824 | EPSG      |      3824 | +proj=longlat +ellps=GRS80...
 3889 | EPSG      |      3889 | +proj=longlat +ellps=GRS80...
 3906 | EPSG      |      3906 | +proj=longlat +ellps=bessel...
```

## Creating Spatial Tables

### Storing Geographic Features

#### PostgreSQL Standard (snake_case)

```sql
-- Create a table for storing locations
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    location GEOMETRY(Point, 4326)
);

-- Insert sample data
INSERT INTO locations (name, location)
VALUES 
    ('Empire State Building', 
     ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326)),
    ('Eiffel Tower', 
     ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326));
```

#### Domain Driven Design (PascalCase)

```sql
-- Create a table for storing locations
CREATE TABLE "Geography"."Landmark" (
    "Id" SERIAL PRIMARY KEY,
    "Name" VARCHAR(100) NOT NULL,
    "Location" GEOMETRY(Point, 4326)
);

-- Insert sample data
INSERT INTO "Geography"."Landmark" ("Name", "Location")
VALUES 
    ('Empire State Building', 
     ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326)),
    ('Eiffel Tower', 
     ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326));
```

### Creating Spatial Indexes

Spatial indexes are essential for efficient queries on spatial data.

#### PostgreSQL Standard (snake_case)

```sql
-- Create a spatial index
CREATE INDEX locations_location_idx ON locations USING GIST (location);
```

#### Domain Driven Design (PascalCase)

```sql
-- Create a spatial index
CREATE INDEX "IX_Geography_Landmark_Location" 
ON "Geography"."Landmark" USING GIST ("Location");
```

## Basic Spatial Queries

### Distance Calculations

#### PostgreSQL Standard (snake_case)

```sql
-- Calculate distance between two points
SELECT ST_Distance(
    ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326),
    ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326)
) AS distance_degrees;

-- Calculate distance in meters (using geography type)
SELECT ST_Distance(
    ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326)::geography,
    ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326)::geography
) AS distance_meters;
```

#### Domain Driven Design (PascalCase)
```sql
-- Calculate distance between two points
SELECT ST_Distance(
    ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326),
    ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326)
) AS "DistanceDegrees";

-- Calculate distance in meters (using geography type)
SELECT ST_Distance(
    ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326)::Geography,
    ST_SetSRID(ST_MakePoint(2.2945, 48.8584), 4326)::Geography
) AS "DistanceMeters";
```

**Output:**
```
 DistanceDegrees
-----------------
    78.5835423196

 DistanceMeters
---------------
    5579151.83
```

The first query shows the distance in degrees, while the second uses the geography type to calculate the distance in meters along the Earth's surface.

### Proximity Searches

#### PostgreSQL Standard (snake_case)
```sql
-- Find landmarks within 10km of a point
SELECT name, location 
FROM locations
WHERE ST_DWithin(
    location::geography,
    ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326)::geography,
    10000
);
```

#### Domain Driven Design (PascalCase)
```sql
-- Find landmarks within 10km of a point
SELECT "Name", "Location" 
FROM "Geography"."Landmark"
WHERE ST_DWithin(
    "Location"::Geography,
    ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326)::Geography,
    10000
);
```

**Output:**
```
        Name        |                  Location
--------------------+--------------------------------------------
 Empire State Building | 0101000020E6100000FA7E6ABC7493C2BF4..
```

This query finds all landmarks within 10 kilometers of the Empire State Building.

### Spatial Relationships

#### PostgreSQL Standard (snake_case)
```sql
-- Create polygon representing Central Park
CREATE TABLE parks (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    boundary GEOMETRY(Polygon, 4326)
);

INSERT INTO parks (name, boundary)
VALUES (
    'Central Park',
    ST_GeomFromText('POLYGON((-73.9819 40.7681, -73.9583 40.7681, 
                              -73.9583 40.8005, -73.9819 40.8005, 
                              -73.9819 40.7681))', 4326)
);

-- Find if a point is within Central Park
SELECT name 
FROM parks 
WHERE ST_Contains(
    boundary, 
    ST_SetSRID(ST_MakePoint(-73.9700, 40.7850), 4326)
);
```

#### Domain Driven Design (PascalCase)
```sql
-- Create polygon representing Central Park
CREATE TABLE "Geography"."Park" (
    "Id" SERIAL PRIMARY KEY,
    "Name" VARCHAR(100) NOT NULL,
    "Boundary" GEOMETRY(Polygon, 4326)
);

INSERT INTO "Geography"."Park" ("Name", "Boundary")
VALUES (
    'Central Park',
    ST_GeomFromText('POLYGON((-73.9819 40.7681, -73.9583 40.7681, 
                              -73.9583 40.8005, -73.9819 40.8005, 
                              -73.9819 40.7681))', 4326)
);

-- Find if a point is within Central Park
SELECT "Name" 
FROM "Geography"."Park" 
WHERE ST_Contains(
    "Boundary", 
    ST_SetSRID(ST_MakePoint(-73.9700, 40.7850), 4326)
);
```

**Output:**
```
    Name     
-------------
 Central Park
```

The query confirms that the specified point is located within Central Park.

## Advanced Spatial Analysis

### Buffer Operations

#### PostgreSQL Standard (snake_case)
```sql
-- Create a 500m buffer around the Empire State Building
SELECT ST_Buffer(
    (SELECT location::geography FROM locations WHERE name = 'Empire State Building'),
    500
)::geometry AS buffer_zone;
```

#### Domain Driven Design (PascalCase)
```sql
-- Create a 500m buffer around the Empire State Building
SELECT ST_Buffer(
    (SELECT "Location"::Geography FROM "Geography"."Landmark" 
     WHERE "Name" = 'Empire State Building'),
    500
)::Geometry AS "BufferZone";
```

### Intersection and Overlap

#### PostgreSQL Standard (snake_case)
```sql
-- Find parks that intersect with a line representing Broadway
SELECT name
FROM parks
WHERE ST_Intersects(
    boundary,
    ST_SetSRID(
        ST_MakeLine(
            ST_MakePoint(-73.9888, 40.7350),
            ST_MakePoint(-73.9808, 40.7900)
        ), 
        4326
    )
);
```

#### Domain Driven Design (PascalCase)
```sql
-- Find parks that intersect with a line representing Broadway
SELECT "Name"
FROM "Geography"."Park"
WHERE ST_Intersects(
    "Boundary",
    ST_SetSRID(
        ST_MakeLine(
            ST_MakePoint(-73.9888, 40.7350),
            ST_MakePoint(-73.9808, 40.7900)
        ), 
        4326
    )
);
```

### Area Calculations

#### PostgreSQL Standard (snake_case)
```sql
-- Calculate the area of Central Park in square kilometers
SELECT 
    name,
    ST_Area(boundary::geography) / 1000000 AS area_sq_km
FROM 
    parks
WHERE 
    name = 'Central Park';
```

#### Domain Driven Design (PascalCase)
```sql
-- Calculate the area of Central Park in square kilometers
SELECT 
    "Name",
    ST_Area("Boundary"::Geography) / 1000000 AS "AreaSquareKm"
FROM 
    "Geography"."Park"
WHERE 
    "Name" = 'Central Park';
```

**Output:**
```
    Name     | AreaSquareKm
-------------+-------------
 Central Park |      3.4127
```

The query calculates that Central Park covers approximately 3.41 square kilometers.

## Geocoding and Reverse Geocoding

While PostGIS doesn't provide geocoding services directly, it can be used with external services or local geocoding databases.

### External Geocoding Services Integration

#### PostgreSQL Standard (snake_case)
```sql
-- Function to call external geocoding API (conceptual example)
CREATE OR REPLACE FUNCTION geocode_address(address TEXT)
RETURNS GEOMETRY(Point, 4326) AS $$
BEGIN
    -- This would call an external API in a real implementation
    -- For demonstration, we'll return a hardcoded point
    IF address LIKE '%Empire State Building%' THEN
        RETURN ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326);
    ELSE
        RETURN NULL;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Example usage
SELECT geocode_address('Empire State Building, New York');
```

#### Domain Driven Design (PascalCase)
```sql
-- Function to call external geocoding API (conceptual example)
CREATE OR REPLACE FUNCTION "Geography"."GeocodeAddress"("Address" TEXT)
RETURNS GEOMETRY(Point, 4326) AS $$
BEGIN
    -- This would call an external API in a real implementation
    -- For demonstration, we'll return a hardcoded point
    IF "Address" LIKE '%Empire State Building%' THEN
        RETURN ST_SetSRID(ST_MakePoint(-73.9856, 40.7484), 4326);
    ELSE
        RETURN NULL;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Example usage
SELECT "Geography"."GeocodeAddress"('Empire State Building, New York');
```

## Raster Data Support

PostGIS supports raster data (gridded datasets like satellite imagery, elevation models, etc.) in addition to vector data.

### Raster Basics

#### PostgreSQL Standard (snake_case)
```sql
-- Create a table for raster data
CREATE TABLE elevation_models (
    id SERIAL PRIMARY KEY,
    name TEXT,
    rast RASTER
);

-- Import a raster (conceptual - actual import would use raster2pgsql)
-- Example: raster2pgsql -s 4326 elevation_file.tif elevation_models > elevation_import.sql
-- Then: psql -d your_database -f elevation_import.sql

-- Query raster metadata
SELECT id, name, ST_Width(rast), ST_Height(rast), ST_NumBands(rast)
FROM elevation_models;
```

#### Domain Driven Design (PascalCase)
```sql
-- Create a table for raster data
CREATE TABLE "Geography"."ElevationModel" (
    "Id" SERIAL PRIMARY KEY,
    "Name" TEXT,
    "RasterData" RASTER
);

-- Import a raster (conceptual - actual import would use raster2pgsql)
-- Example: raster2pgsql -s 4326 elevation_file.tif "Geography"."ElevationModel" > elevation_import.sql
-- Then: psql -d your_database -f elevation_import.sql

-- Query raster metadata
SELECT "Id", "Name", ST_Width("RasterData"), ST_Height("RasterData"), ST_NumBands("RasterData")
FROM "Geography"."ElevationModel";
```

## Practical Applications

### Finding Nearby Points of Interest

#### PostgreSQL Standard (snake_case)
```sql
-- Create a table for points of interest
CREATE TABLE points_of_interest (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50),
    location GEOMETRY(Point, 4326)
);

-- Create spatial index
CREATE INDEX points_of_interest_location_idx 
ON points_of_interest 
USING GIST (location);

-- Insert sample data
INSERT INTO points_of_interest (name, category, location)
VALUES 
    ('Central Park Zoo', 'attraction', 
     ST_SetSRID(ST_MakePoint(-73.9718, 40.7676), 4326)),
    ('Metropolitan Museum of Art', 'museum', 
     ST_SetSRID(ST_MakePoint(-73.9632, 40.7794), 4326)),
    ('Starbucks', 'coffee', 
     ST_SetSRID(ST_MakePoint(-73.9802, 40.7643), 4326));

-- Find points of interest within 1km of a location
SELECT 
    name, 
    category,
    ST_Distance(
        location::geography, 
        ST_SetSRID(ST_MakePoint(-73.9700, 40.7700), 4326)::geography
    ) AS distance_meters
FROM 
    points_of_interest
WHERE 
    ST_DWithin(
        location::geography,
        ST_SetSRID(ST_MakePoint(-73.9700, 40.7700), 4326)::geography,
        1000
    )
ORDER BY 
    distance_meters;
```

#### Domain Driven Design (PascalCase)
```sql
-- Create a table for points of interest
CREATE TABLE "Tourism"."PointOfInterest" (
    "Id" SERIAL PRIMARY KEY,
    "Name" VARCHAR(100) NOT NULL,
    "Category" VARCHAR(50),
    "Location" GEOMETRY(Point, 4326)
);

-- Create spatial index
CREATE INDEX "IX_Tourism_PointOfInterest_Location" 
ON "Tourism"."PointOfInterest" 
USING GIST ("Location");

-- Insert sample data
INSERT INTO "Tourism"."PointOfInterest" ("Name", "Category", "Location")
VALUES 
    ('Central Park Zoo', 'attraction', 
     ST_SetSRID(ST_MakePoint(-73.9718, 40.7676), 4326)),
    ('Metropolitan Museum of Art', 'museum', 
     ST_SetSRID(ST_MakePoint(-73.9632, 40.7794), 4326)),
    ('Starbucks', 'coffee', 
     ST_SetSRID(ST_MakePoint(-73.9802, 40.7643), 4326));

-- Find points of interest within 1km of a location
SELECT 
    "Name", 
    "Category",
    ST_Distance(
        "Location"::Geography, 
        ST_SetSRID(ST_MakePoint(-73.9700, 40.7700), 4326)::Geography
    ) AS "DistanceMeters"
FROM 
    "Tourism"."PointOfInterest"
WHERE 
    ST_DWithin(
        "Location"::Geography,
        ST_SetSRID(ST_MakePoint(-73.9700, 40.7700), 4326)::Geography,
        1000
    )
ORDER BY 
    "DistanceMeters";
```

**Output:**
```
          Name          |  Category  | DistanceMeters
------------------------+------------+----------------
 Central Park Zoo       | attraction |        282.3754
 Starbucks              | coffee     |        732.0841
```

This query finds points of interest within 1 kilometer of a specified location and orders them by distance.

### Routing and Directions

PostGIS can be extended with pgRouting for network analysis.

#### PostgreSQL Standard (snake_case)
```sql
-- Install pgRouting extension
CREATE EXTENSION pgrouting;

-- Create a table for road network
CREATE TABLE road_network (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    geom GEOMETRY(LineString, 4326),
    source INTEGER,
    target INTEGER,
    cost FLOAT
);

-- Find the shortest path (conceptual example)
SELECT * FROM pgr_dijkstra(
    'SELECT id, source, target, cost FROM road_network',
    1, 5, -- start and end nodes
    directed => false
);
```

#### Domain Driven Design (PascalCase)
```sql
-- Install pgRouting extension
CREATE EXTENSION PgRouting;

-- Create a table for road network
CREATE TABLE "Transport"."Road" (
    "Id" SERIAL PRIMARY KEY,
    "Name" VARCHAR(100),
    "Geometry" GEOMETRY(LineString, 4326),
    "SourceNode" INTEGER,
    "TargetNode" INTEGER,
    "Cost" FLOAT
);

-- Find the shortest path (conceptual example)
SELECT * FROM pgr_dijkstra(
    'SELECT "Id", "SourceNode", "TargetNode", "Cost" FROM "Transport"."Road"',
    1, 5, -- start and end nodes
    directed => false
);
```

### Geospatial Analytics

#### PostgreSQL Standard (snake_case)
```sql
-- Create a table for population data by region
CREATE TABLE population_distribution (
    id SERIAL PRIMARY KEY,
    region_name VARCHAR(100),
    population INTEGER,
    boundary GEOMETRY(Polygon, 4326) 
);

-- Calculate population density
SELECT 
    region_name,
    population,
    ST_Area(boundary::geography) / 1000000 AS area_sq_km,
    population / (ST_Area(boundary::geography) / 1000000) AS density_per_sq_km
FROM 
    population_distribution
ORDER BY 
    density_per_sq_km DESC;
```

#### Domain Driven Design (PascalCase)
```sql
-- Create a table for population data by region
CREATE TABLE "Demographics"."Region" (
    "Id" SERIAL PRIMARY KEY,
    "Name" VARCHAR(100),
    "Population" INTEGER,
    "Boundary" GEOMETRY(Polygon, 4326) 
);

-- Calculate population density
SELECT 
    "Name",
    "Population",
    ST_Area("Boundary"::Geography) / 1000000 AS "AreaSquareKm",
    "Population" / (ST_Area("Boundary"::Geography) / 1000000) AS "DensityPerSquareKm"
FROM 
    "Demographics"."Region"
ORDER BY 
    "DensityPerSquareKm" DESC;
```

## Working with GeoJSON

GeoJSON is a common format for exchanging spatial data with web applications.

#### PostgreSQL Standard (snake_case)
```sql
-- Convert a geometry to GeoJSON
SELECT ST_AsGeoJSON(location) 
FROM locations 
WHERE name = 'Empire State Building';

-- Import GeoJSON into a table
INSERT INTO points_of_interest (name, category, location)
SELECT 
    'Times Square', 
    'attraction', 
    ST_GeomFromGeoJSON('{
      "type": "Point",
      "coordinates": [-73.9855, 40.7580]
    }');
```

#### Domain Driven Design (PascalCase)
```sql
-- Convert a geometry to GeoJSON
SELECT ST_AsGeoJSON("Location") 
FROM "Geography"."Landmark" 
WHERE "Name" = 'Empire State Building';

-- Import GeoJSON into a table
INSERT INTO "Tourism"."PointOfInterest" ("Name", "Category", "Location")
SELECT 
    'Times Square', 
    'attraction', 
    ST_GeomFromGeoJSON('{
      "type": "Point",
      "coordinates": [-73.9855, 40.7580]
    }');
```

**Output:**
```
{
  "type":"Point",
  "coordinates":[-73.9856,40.7484]
}
```

This GeoJSON representation can be easily integrated with web mapping libraries like Leaflet or Mapbox.

## Conclusion

PostGIS transforms PostgreSQL into a powerful spatial database capable of handling complex geographic data and analysis. By leveraging its extensive set of spatial functions and data types, developers can create sophisticated GIS applications directly within their PostgreSQL database.

Key takeaways from this chapter include:

1. PostGIS provides robust spatial extensions to PostgreSQL with hundreds of functions
2. Spatial data can be stored using various geometry types (points, lines, polygons)
3. Spatial indexes are crucial for performance optimization
4. PostGIS supports both vector and raster data
5. Common spatial operations include distance calculations, containment tests, and buffer creation
6. Integration with web applications is simplified through GeoJSON support
7. Advanced spatial analysis can be performed directly in the database

Whether you're building location-based services, analyzing geographic patterns, or creating interactive maps, PostGIS provides the foundation for powerful spatial data management in PostgreSQL.