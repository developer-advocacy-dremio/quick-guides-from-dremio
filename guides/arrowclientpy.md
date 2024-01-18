## Python Arrow Client Code

- [A Version of this client is available on PyPl](https://pypi.org/project/dremio-simple-query/)
- [The Github Repo For this Library](https://github.com/developer-advocacy-dremio/dremio_simple_query)

The Code Below can be used to create an arrow client in any python project.

- Example of what a Dremio Cloud Arrow Flight URL would look like: `grpc+tls://data.dremio.cloud:443`
- Example of what a Dremio Software Arrow Flight URL would look like: `grpc+tls://<ip-address>:32010` (SSL) or `grpc://<ip-address>:32010` (no SSL)

This require the `pyarrow` library

- for the `toDuckDB` function you must also have the `duckdb` library

- for the `toPolars` function you must have the `polars` library

- for the `toPandas` function you must have the `pandas` library


```py
#----------------------------------
# IMPORTS
#----------------------------------
## Import Pyarrow
from pyarrow import flight
from pyarrow.flight import FlightClient
import duckdb
import pyarrow.dataset as ds
import polars as pl
import pandas as pd

class DremioConnection:
    
    def __init__(self, token, location):
        self.token = token
        self.location = location
        self.headers = [
    (b"authorization", f"bearer {token}".encode("utf-8"))
    ]
        self.client = FlightClient(location=(location))
        
    def query(self, query, client, headers):
        ## Options for Query
        options = flight.FlightCallOptions(headers=headers)
        
        ## Get ticket to for query execution, used to get results
        flight_info = client.get_flight_info(flight.FlightDescriptor.for_command(query), options)
    
        ## Get Results (Return Value a FlightStreamReader)
        results = client.do_get(flight_info.endpoints[0].ticket, options)
        return results
        
    # Returns a FlightStreamReader
    def toArrow(self, query):
        return self.query(query, self.client, self.headers)
    
    #Returns a DuckDB Relation
    def toDuckDB(self, querystring):
        streamReader = self.query(querystring, self.client, self.headers)
        table = streamReader.read_all()
        my_ds = ds.dataset(source=[table])
        return duckdb.arrow(my_ds)

    #Returns a Polars Dataframe
    def toPolars(self, querystring):
        streamReader = self.query(querystring, self.client, self.headers)
        table = streamReader.read_all()
        df = pl.from_arrow(table)
        return df

    #Returns a Pandas Dataframe
    def toPandas(self, querystring):
        streamReader = self.query(querystring, self.client, self.headers)
        df = streamReader.read_pandas()
        return df
```

### Function for Getting Dremio PAT Token via API

In Dremio software you'll have to make an API call to get your PAT token, here is an example of a function you can use to do this using the requests library:

```py
import requests
import os

## Function to Retrieve PAT TOken from Dremio
def get_token(uri, payload):
    # Make the POST request
    response = requests.post(uri, json=payload)

    # Check if the request was successful
    if response.status_code == 200:
        # Parse the JSON response
        data = response.json()
        # Extract the token
        return data.get("token", "")
        print("Token:", token)
    else:
        print("Failed to get a valid response. Status code:", response.status_code)

## Username and Password for Dremio Account
username = "username"
password = "password"

## Dremio REST API URL
uri = "http://localhost:9047/apiv2/login"

## Payload for Get Token Requests
payload = {
    "userName": username,
    "password": password
}

token = get_token(uri, payload)
```
