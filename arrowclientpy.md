## Python Arrow Client Code

- [A Version of this client is available on PyPl](https://pypi.org/project/dremio-simple-query/)
- [The Github Repo For this Library](https://github.com/developer-advocacy-dremio/dremio_simple_query)

The Code Below can be used to create an arrow client in any python project.

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

    #Returns a Polars Dataframe
    def toPolars(self, querystring):
        streamReader = self.query(querystring, self.client, self.headers)
        df = streamReader.read_pandas()
        return df
```
