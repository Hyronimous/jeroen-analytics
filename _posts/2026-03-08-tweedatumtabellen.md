---
title: "Meerdere datumrollen modelleren in Power BI met Calculation Groups en UDF Function"
date: 2026-03-08
tags: [fabric, engineering, architecture, notebooks]
categories: [powerbi, fabric, dax]
---

Een scenario dat vaak voorkomt is dat een fact table meerdere datums heeft. Een verkoop fact heeft orderdatum, shipment datum, return datum, factuurdatum en uitendelijk ook een betaaldatum
PowerBI ontwikkelaars die al wat verder gevorderd zijn zien op een gegeven moment de mogelijkheid om met USERELATIONSHIP() te werken. Vervolgens met Calculation groups ga je nog verder

```
CALCULATE(
    SELECTEDMEASURE(),
    USERELATIONSHIP(FactSales[DeliveryDate], DimDate[Date])
)
```
Nu met een paar calculation items kun je al heel flexibel switchen tussen de verschillende rollen van de datum dimensie. 

# Het probleem ontstaat bij twee tijdlijnen tegelijk

Het model loopt vast zodra je **twee tijdperspectieven tegelijk wilt analyseren**.

Bijvoorbeeld een matrix zoals:

| Order Month | Delivery Jan | Delivery Feb | Delivery Mar |
|--------------|-------------|-------------|-------------|
| Jan | 120 | 45 | 12 |
| Feb | 0 | 210 | 60 |
| Mar | 0 | 0 | 175 |

Dit is een **flow-analyse tussen twee tijdlijnen**:

- wanneer werd iets besteld  
- wanneer werd het geleverd  

Met slechts één datumtabel kan Power BI maar **één filtercontext tegelijk toepassen**.

Als dezelfde datumtabel zowel rijen als kolommen filtert, krijg je logisch gezien:

Wat je dan kunt doen is gebruik maken van een UDF in combinatie met Calculation Groups. Daarnaast maak je ook een tweede datumdimensie aan. Net zoals met de eerste datumdimensie maak je alle relaties aan. Kiest als actieve relatie een andere standaard rol.
de UDF ziet er dan zo uit:
```
DEFINE
    FUNCTION _measurebyDate =
    (
        _measure: decimal expr,
        _factTableColumn: anyref,
        _DimensionColumn: anyref
    ) =>
    CALCULATE(
        _measure,
        USERELATIONSHIP(
            _factTableColumn,
            _DimensionColumn
        )
    )
```
Door het zo op te stellen kun je nu twee Calculation groups aanmaken. De eerste zal steeds Dim_Datum[datum] als parameter doorgeven aan _DimensionColumn. De items in calculation group 2 juist [Dim_Datum2][datum].
Daarnaast laat je de optie ook nog open voor rapportbouwers om lokaal zelf een Measure aan te maken die gebruik maakt van deze functie, misschien zelfs input op basis van een slicer.

# Ontwerp

Het belangrijkste inzicht achter dit patroon is:

In analytische modellen is tijd geen dimensie maar een rol.

Door:
datumtabellen generiek te houden en rollen via Calculation Groups te bepalen ontstaat een model dat eenvoudig blijft maar geen mappen vol met measures krijgt

maximale vrijheid geeft aan thin reports. Semantic Models moeten een basis bieden. Dat kan zijn voor raportagebouwers en voor excel draaitabellen. Daarvoor is het nodig dat eenvoud blijft. Daartegenover staat
dat je veel opties beschikbaar wilt maken. Door gebruik te maken van Calculation Groups, UDFS en flexibele modellering kun je veel van deze doelen bereiken.

# Waarom een UDF gebruiken

De UDF voorkomt dat USERELATIONSHIP logica in iedere calculation item
moet worden herhaald. Daardoor blijft de calculation group leesbaar en uitbreidbaar. 
UDF is vrij nieuw, we ontdekken steeds meer mogelijkheden die onze modellen beter maken. Dit soort structurering is 1 daarvan.
Daarmee hebben we een nieuwe mogelijkheid om de complexiteit van onze modellen onder controle houden
