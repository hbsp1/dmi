
# DMI

- DMI ETL
- myDMI API

<div style="text-align:right; font-size: 12px">
næste: datakilde
</div>


---

### DMI Open Data API
- Datakilde

---

- "Hvert minut indsamler DMI store mængder vejr-, havdata ... (som anvendes til fx) vejrudsigter..."

- Forskellige datasæt frit for alle: https://www.dmi.dk/frie-data

---

2 relevante datasæt
1. [Meteorological Observations Data](https://www.dmi.dk/friedata/dokumentation/meteorological-observations-data "Meteorological Observations Data") 

2. [Climate Data](https://www.dmi.dk/friedata/dokumentation/climate-data "Climate Data")

---

1. **Meteorological observation data (metObs)**
	- "..so-called raw data..
	- neither quality controlled nor processed..."


2. **Climate data**
	- "...is meteorological observation data
	- ... more than an hour old
	- ... undergoes quality control 
	- ... checked and erroneous data is removed"

---

... anvender observationsdata (ikke klimadata)

- det er rå, ukontrolleret og 
- aktuelle målerdata (live data "Hvert minut..")



<div style="text-align:right; font-size: 12px">
næste: data struktur
</div>

---

DMI schema: liste med observationer

```json
"type": "Feature", (Observation)
{
	"properties": {
		"parameterId": "temp_dry", (hvad måles)
		"stationId": "06070", (hvor, århus)
		"value": 13.9, (resultat af måling)
		"observed": "2026-10-16T05:00Z" (hvornår)
	}
},
{
"type": "Feature", (Observation)
"type": "Feature", (Observation)
```

<div style="text-align:right; font-size: 12px">
næste: etl process
</div>


---

# ETL
- Fra DMI til my_database


---

### Setup ETL projekt: 

1. Run database_setup_script

2. ETL process (opg.1)

3. Presentation-layer, fx: 
	1. myDmi API (opg.2)
	2. BI-rapport (visuel)


<div style="text-align:right; font-size: 12px">
næste: db_schema
</div>

---

### 1. database schema

1. stations tabel 
2. observations tabel 

	- one-to-many  relation: station_id
		- spørgsmål fra sidst - test
		- Se relation med data

<div style="text-align:right; font-size: 12px">
næste: test relation
</div>


---

Spørgsmål: Virker relationen ? 

![C:\Users\hoang\Documents\GitHub\dmi\notes\slides\pics\foreign_key_spm.png](file:///c%3A/Users/hoang/Documents/GitHub/dmi/slides/pics/foreign_key_spm.png)




---

DB_tabel properties (pgadmin)

![[metobs_foreign_key_constraint.png]]


---

DB Entity‑Relationship Diagram (pgadmin)

![[foreign_key_erd.png]]

<div style="text-align:right; font-size: 12px">
næste: test constraint
</div>

---

Test FK_constraint - sql_insert metObs (pg4admin)

![[foreign_key test.png]]

Constraint OK

<div style="text-align:right; font-size: 12px">
næste: se excel
</div>

---

Visuel: relation i Excel
(one-to-many)

![[foreign_key test_excel.png]]


---

Excel - expand table

![[foreign_key test_excel_2.png]]

stations.stations_id == obs.stations_id



<div style="text-align:right; font-size: 12px">
næste: database -> etl ->
</div>

---


### 2. ETL process

- Split store data udtræk 
	- (langsomt -> pagination)
- Dataflow

<div style="text-align:right; font-size: 12px">
næste: pagination
</div>

---

Pagination 
- Node med næste side: "links"
- "rel" -> next_page
- "href" = next_url

```json
"type": "FeatureCollection", (liste med observationer + links)
... (mange observationer her) 
  "links": [
    {
      "title": "Next set of results",
      "rel": "next",
      "href": "https://opendataapi.dmi.dk... (next_url)",
    }
  ]
```


---

Next URL findes i "href"

```python
def get_query(api_service, params=None) -> list[]: ... hide
		url = f"{base_url}{api_service}"
		while url:
            response = requests.get(f"{url}", params=params) 
            ... 
            url = None
            for link in r["links"]:
                if link.get("rel") == "next":
                    url = link.get("href")
                    params = None # med fra next_url
                    break
	return collection_items: list[]
```


<div style="text-align:right; font-size: 12px">
næste: projekt struktur
</div>


---

ETL Projektstruktur:

    dmi/
    ├── src/            
    │   ├── database/   : Database setup script
    │   ├── extract/    : Hent rå data fra DMI
    │   ├── transform/  : Databehandling
    │   ├── load/       : Indlæs clean_data til database
    │   └── main.py     : etl bliver kørt main
    │
    └── (unit) tests/
        ├── extract/
        ├── transform/
        └── load/


<div style="text-align:right; font-size: 12px">
næste: dataflow eksempel
</div>


---

Dataflow: Extract_stations

![[1_dataflow_extract_stations.png]]


<div style="text-align:right; font-size: 12px">
næste: extract fortsætter til metobs
</div>

---

Dataflow: Extract_metObs
-> Limit=100.000 obs. per side
![[2_dataflow_extract_metobs.png]]

---

Dataflow: Transform

![[3_dataflow_transform.png]]


---

Dataflow: Load

![[4_dataflow_load.png]]

<div style="text-align:right; font-size: 12px">
næste: test get_dmi
</div>

---
### Test GET_request
...hente vejrdata fra den 16. oktober 2026

![[dmi_response.png]]


- Resultat: DMI = ETL DB 

<div style="text-align:right; font-size: 12px">
næste: 3. Presentation-layer
</div>

---

# Udstil data
- næste: gør data tilgængeligt for end-user
	- API og
	- Rapport


<div style="text-align:right; font-size: 12px">
næste: myDMI API
</div>

---

# myDMI API
- Opgave 2: 

---

myDMI Projektstruktur:

    my_dmi/
    ├── app/            
    │   ├── database/   : Connection til my_PostgreSQL DMI_etl
	│   ├── models/     : SQLAlchemy ORM <rel> db mapping tbl/cols
    │   ├── schemas/    : FastAPI response.json schema
    │   ├── api/        : URL Station_routes og metObs_routes
    │   └── main.py     : Starter api og tilføjer api_routes(endpoints)
	│
    └── (unit tests/)
        └── TODO/



<div style="text-align:right; font-size: 12px">
næste: eksempel med metObs -> /models/
</div>

---

app/models/     : database mapping tbl/cols

```python
class Observation(Base):
    __tablename__ = "observations"
    __table_args__ = {"schema": "dmi"}
	station_id = Column(String, FK..)
    parameter_id = Column(String)
    observed = Column(DateTime)
    value = Column(Float)  
	...
```


<div style="text-align:right; font-size: 12px">
næste: /schema/
</div>


---

app/schemas/    : response.json format/schema

```Python
class ObservationBase(BaseModel): # En enkelt observation
    station_id: str
    parameter_id: str
	...

class ObservationResponse(BaseModel): # response.json, se swag
    numberReturned: int
    offset: int # Paginering
    limit: int
    observations: List[ObservationBase] # Liste med observationer
```

<div style="text-align:right; font-size: 12px">
næste: /api/get_observations_routes
</div>

---
app/api/        : URL_route (GET_OBSERVATIONS)
(main.py) base_URL + service

```python
@router.get("/items/", response_model=ObservationResponse) # Response List[Obs]
def get_observations(
    stationId: str | None = None,
    parameterId: str | None = None,
    offset: int = 0,
    limit: int = 1000,
    db: Session = Depends(get_db) # db_sesson pool
    ):
	... hent i database via ORM MODEL 
    return {
```


<div style="text-align:right; font-size: 12px">
næste: /api/get_routes_return
</div>

---

app/api/ return liste med observationer

```python
    return {
        "numberReturned": len(observations),
        "offset": offset,
        "limit": limit,
        "observations": observations # liste med observationer
    }
```


<div style="text-align:right; font-size: 12px">
næste: test i client -> swagger/browser
</div>



---

![[my_dmi_swagger.png]]


<div style="text-align:right; font-size: 12px">
næste: response body
</div>


---

liste med observationer
![[my_dmi_swagger_response.png]]
API: 200 OK

<div style="text-align:right; font-size: 12px">
næste: rapport
</div>

---

# rapport
- "... (og) ...sammenligne data på tværs af målestationer."

![[power_bi_logon.png]]

---
PBI Datamodel. relation (postgres_db)
![[pbi_datamodel.png]]


---
PBI: sammenlign data
dato den 16. okt. (-2t timezone)
![[pbi_aarhus_bornholm_16.10.2026.png]]

<div style="text-align:right; font-size: 12px">
næste: se DMI
</div>

---

![[dmi_aarhus_16.10.2026.png]]

[Lokationarkiv | DMI](https://www.dmi.dk/lokationarkiv/show/DK/2624652/Aarhus/#arkhttps://www.dmi.dk/lokationarkiv/show/DK/2624652/Aarhus/#arkiviv)

<div style="text-align:right; font-size: 12px">
næste: igen
</div>

---

Data til Excel: koldere i grønland

![[sammenlign_excel_16.10.2026.png]]


<div style="text-align:right; font-size: 12px">
næste: slut
</div>


---

# status

- ETL: OK

- myDMI API : OK

---
# TO-DO

- data-validering: 
	- "(metObs) data is neither quality controlled nor processed in any way...
	- ... Some of the parameters use codes instead of values."

- Unit tests

- Sæt en schedule på ETL - hvordan? 

- README.md


---
# Slut