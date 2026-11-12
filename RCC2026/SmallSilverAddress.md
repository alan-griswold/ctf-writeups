# A Small Silver Address

RowdyCyberCon CTF 2026 · OSINT · Solved

## Challenge

Given a single photo of two oval metal pole tags, find the physical location of the lower-right tag. The tag read: **PGE Co / C73-14D / 329**.

## Identifying the utility

The first move was reading the tag precisely. "PGE Co" with*no ampersand is Portland General Electric (Oregon), not PG&E, the California utility it's easy to confuse it with. That one-character distinction set the entire search region.

Portland General Electric's streetlight documentation describes exactly this kind of small metal pole tag, showing a **map number** and a **pole number**. That matches the format on the tag: `C73-14D` (map) + `329` (pole).

## Finding the public data source

PGE publishes a public ArcGIS "Pole Search Map" (data owner `PGE_PublicData`). Public ArcGIS feature services are queryable directly over HTTP, which turns a map into a searchable database.

- Feature service: `https://services6.arcgis.com/eqsXlDArbs19VWhn/arcgis/rest/services/Public_Data/FeatureServer`
- Layer 13 ("Distribution Pole") has a `FACILITYID` field formatted `<map>-<pole>`.

## The query

Reformatting the tag to match the field's style (`C7314D-329`) and querying the layer directly:

```
.../FeatureServer/13/query?where=FACILITYID='C7314D-329'&outFields=*&outSR=4326&f=json
```

Result: a 35 ft distribution pole at latitude `44.9581`, longitude `-123.0180`, in Salem, OR.

## Verification

Rather than trusting a single hit, I queried neighboring `C7314D-*` poles and confirmed they cluster within a few hundred meters of each other, consistent with a real pole grid, which confirms the match is the right one and not a coincidental field collision.

Flag: precise coordinates

## Takeaway

The whole solve turned on two things: reading the source material exactly (PGE vs PG&E), and knowing that a public ArcGIS "map" is really a queryable REST API. Infrastructure that utilities publish for transparency doubles as a precise geolocation database if you know how to query it. Verification against neighboring records is what separates a confident answer from a lucky guess.
