---
title: "Helper functions: hero or villain"
date: 2026-02-01
tags: [fabric, engineering, architecture, notebooks]
categories: [architecture, programming, fabric]
---

# Helper functions: hero or villain

Wanneer je met PySpark taken uitvoert, zie je al snel dat veel handelingen steeds opnieuw terugkomen. Al snel kom je op het idee om helper-functies te maken. Deze zijn handig, erg handig zelfs. Maar om een quote uit een film te lenen: *“You either die a hero or you live long enough to become the villain.”*

Op een gegeven moment wordt een helper-functie steeds ingewikkelder: meer parameters, meer uitzonderingen en steeds minder hulp beschikbaar. En wie de functie zelf niet heeft geschreven, kent haar niet of vindt het veel te lastig om te gebruiken. Die persoon krijgt een geweldig idee: waarom maak ik zelf niet een helper-functie?

Wat er vaak gebeurt, is dat een handige functie gaat groeien: meer functionaliteit, meer uitzonderingssituaties en meer parameters. Net zoals je in statistische modellen het concept *overfitting* kent, kun je bij functies tegen dezelfde problematiek aanlopen. De functie is zo specifiek voor één situatie gemaakt, dat zij voor andere situaties minder bruikbaar wordt.

Dus volgt eigenlijk een logische eerste regel voor een helper-functie: een goede helper-functie lost een alledaags probleem op.

Een voorbeeld van zo’n eenvoudige helper-functie:

```python
from pyspark.sql.types import StructType
from pyspark.sql import DataFrame
import os
import pandas as pd

def read_csv(folder: str, schema: StructType = None) -> DataFrame:
    """
    Reads all CSV files from a folder and returns a Spark DataFrame.

    Args:
        folder:  Path to the folder containing the CSV files.
        schema:  Optional Spark schema to enforce on the DataFrame.
                 If None, Spark will infer the schema from the data.

    Returns:
        A Spark DataFrame with all positions combined.
    """
    csv_files = [f for f in os.listdir(folder) if f.endswith('.csv')]
    print(f"Found {len(csv_files)} files: {csv_files}")

    pandas_df = pd.concat(
        [pd.read_csv(f"{folder}/{f}") for f in csv_files],
        ignore_index=True
    )
    print(f"Total rows: {len(pandas_df)}")

    return spark.createDataFrame(pandas_df, schema=schema)
```

Je vertelt alleen waar de CSV-bestanden staan en je krijgt een DataFrame terug met alle CSV-bestanden samengevoegd in één DataFrame. Je kunt hier eventueel nog een zoekstring aan toevoegen, maar veel ingewikkelder moet je het niet maken.

Juist die eenvoud zorgt ervoor dat de functie hergebruikt wordt. Ze is handig, wordt veel gebruikt en bespaart in iedere notebook een aantal regels code. Daardoor wordt de taak in die notebook duidelijker en beter inzichtelijk.

Het tweede punt is bekendheid. Je moet weten dat een helper-functie bestaat, anders kun je haar niet gebruiken. Je moet ook weten hoe je haar kunt gebruiken. Als je dat niet weet, ga je zelf op zoek. En op het internet zijn vele, beter vindbare alternatieven voorhanden die ook goed werken.

Het gevolg is dat er tientallen versies van dezelfde functie door notebooks heen gaan zwerven. En iedereen die ermee werkt, moet bij iedere functie opnieuw leren hoe deze afwijkt van de vorige.

Dus blijft er eigenlijk maar één optie over: als een functie waardevol is, moet je documenteren. Helptekst toevoegen. Uitleggen wat de functie doet en hoe je haar moet gebruiken. En ze moet op een plek staan waar ze makkelijk te vinden is.

De eerste plek voor documentatie is in de functie zelf. In het voorbeeld zie je de volgende onderdelen terug:

- Wat verwacht de functie  
- Wat doet de functie  
- Wat krijg je terug  

Eenvoudig. Als ik dit doe, dan krijg ik dat. Als ik het zelf zou doen, moet ik dat programmeren.

Misschien moet je nog specifieke dingen doen. Die doe je dan buiten de functie om, in je eigen notebook. Dat is specifiek voor het probleem dat jij oplost — daar hoeft niemand anders last van te hebben.

Wat betreft bereikbaarheid zijn er meerdere opties beschikbaar: een centrale notebook die mensen kunnen kopiëren, een wheel, of gebruikmaken van sempy. Zonder hier te diep op in te duiken: kijk hoe belangrijk deze functies zijn binnen je organisatie en stem je keuze daarop af.

Hier komt het concept start architecture om de hoek kijken: zorg ervoor dat de eenvoudigste manier van werken ook de manier is die binnen de architectuur past. Als je het heel eenvoudig maakt om de beste versies van helper-functies te gebruiken, dan doen mensen dat ook.

In de openingsalinea had ik het over hero of villain. Beide beginnen op dezelfde voet. Maar daarna zie je wat er nodig is om helper-functies heroes te laten blijven. Mensen moeten ze kennen, ze moeten te vinden zijn en het moet duidelijk zijn wat ze doen.

Vaak worden briljante ideeën gegoten in helper-functies, waarna ze een gespecialiseerd villain-bestaan leiden als ondersteuning van een paar notebooks van de programmeur zelf. De rest blijft uit de buurt, want voor hen is deze functie een villain.
