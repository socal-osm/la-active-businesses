# la-active-businesses
A repo to import L.A.'s active businesses into OpenStreetMap

## Data sources
- [L.A. County's address point database](http://egis3.lacounty.gov/dataportal/2012/06/19/la-county-address-points/) offers more specific geocoding by lat/lon than the 
- [L.A. City listing of active businesses](https://data.lacity.org/A-Prosperous-City/Listing-of-Active-Businesses/6rrh-rzua) which has all the pertinent information for the business
- [Census geocoder](http://geocoding.geo.census.gov) which could be helpful (but possibly not help get node close to building)
- [L.A. city addresses](https://data.lacity.org/A-Well-Run-City/Addresses-in-the-City-of-Los-Angeles/4ca8-mxuh) not sure how they differ from other sets

## Next steps
1. Test data
2. Define import strategy
3. Define update strategy

## Testing steps
Download both data source files and import into a Postgres database.

Join both using code like this (replace la_businesses and la_addresses with whatever you named those tables):
```
SELECT 
	replace(initcap(lower(la_businesses."DBA NAME")),'''S','''s') as name, 
	la_businesses."STREET ADDRESS",
	la_businesses."CITY",
	la_businesses."PRIMARY NAICS DESCRIPTION",
	la_businesses."LOCATION START DATE",
	SUBSTR("LOCATION START DATE", 7, 4)||'-'||SUBSTR("LOCATION START DATE", 1, 2)||'-'||SUBSTR("LOCATION START DATE", 4, 2) as "start_date",
	la_addresses."X" as lon,
	la_addresses."Y" as lat,
	la_addresses."Number" as "addr:housenumber",
	la_addresses."PreDir"||' '||la_addresses."StreetName"||' '||la_addresses."PostType" as "addr:street",
	la_addresses."ZipCode" as "addr:postcode",
	la_addresses."LegalComm" as "addr:city"
FROM la_businesses
INNER JOIN la_addresses ON la_addresses."Number"||' '||la_addresses."PreDirAbbr"||' '||upper(la_addresses."StreetName")||' '||upper(la_addresses."PostType") = la_businesses."STREET ADDRESS"
WHERE la_businesses."DBA NAME" LIKE '%DONUT%'
```

Which outputs

| name                        | STREET ADDRESS         | CITY        | PRIMARY NAICS DESCRIPTION                                                           | LOCATION START DATE | start_date | lon                  | lat                | addr:housenumber | addr:street           | addr:postcode | addr:city   |
|-----------------------------|------------------------|-------------|-------------------------------------------------------------------------------------|---------------------|------------|----------------------|--------------------|------------------|-----------------------|---------------|-------------|
| 7 Stars Donuts And Sandwich | 1754 W SLAUSON AVENUE  | LOS ANGELES | Bakeries & tortilla mfg.                                                            | 09/28/2011          | 2011-09-28 | -118.308467372461621 | 33.98899176382843  | 1754             | West Slauson Avenue   | 90047         | Los Angeles |
| Big Jim's Donuts            | 702 N VERMONT AVENUE   | LOS ANGELES | All other miscellaneous store retailers (including tobacco, candle, & trophy shops) | 03/09/2005          | 2005-03-09 | -118.291519663591998 | 34.083983484452311 | 702              | North Vermont Avenue  | 90029         | Los Angeles |
| Big Mama's Donuts           | 5409 N FIGUEROA STREET | LOS ANGELES | Other miscellaneous mfg.                                                            | 07/26/2010          | 2010-07-26 | -118.196345588844025 | 34.10802959949406  | 5409             | North Figueroa Street | 90042         | Los Angeles |
| Boston Cream Donuts         | 1009 W ANAHEIM STREET  | WILMINGTON  |                                                                                     | 10/01/2004          | 2004-10-01 | -118.274513639040862 | 33.779496806845508 | 1009             | West Anaheim Street   | 90744         | Los Angeles |
| California Donuts           | 801 S ALVARADO STREET  | LOS ANGELES | Bakeries & tortilla mfg.                                                            | 08/10/1993          | 1993-08-10 | -118.27814801768271  | 34.054792499963845 | 801              | South Alvarado Street | 90057         | Los Angeles |
| California Donuts           | 3540 W 3RD STREET      | LOS ANGELES | Full-service restaurants                                                            | 01/06/1982          | 1982-01-06 | -118.293239355574485 | 34.068898644827705 | 3540             | West 3rd Street       | 90020         | Los Angeles |

This has problems. For instance, there are 341 records in the business file `WHERE "DBA NAME" LIKE '%DONUT%'` instead of the 63 shown above. Probably because there's no matching address. Join script needs more refining and logic to catch items that don't hava perfect match. We're also seeing duplicates. And the NAICS DESCRIPTION field may not be easily translatable to OSM's tags.
