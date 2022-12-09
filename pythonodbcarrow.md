## Python pyODBC and pyArrow connect to Dremio Sample Code

### pyODBC

```py
#----------------------------------
# IMPORTS
#----------------------------------

## Import pyodbc
import pyodbc

## import pandas
import pandas as pd

## import environment variables
from os import environ

#----------------------------------
# SETUP
#----------------------------------

token=environ.get("token", "personal token not defined")
connector="Driver={Dremio ODBC Driver 64-bit};ConnectionType=Direct;HOST=sql.dremio.cloud;PORT=443;AuthenticationType=Plain;" + f"UID=$token;PWD={token};ssl=true;"

#----------------------------------
# CREATE CONNECTION AND CURSOR
#----------------------------------

# establish connection
cnxn = pyodbc.connect(connector, autocommit=True)

# set encoding
cnxn.setdecoding(pyodbc.SQL_CHAR, encoding='utf-8')

# creating a cursor to send messages through the connection
cursor = cnxn.cursor()

#----------------------------------
# RUN QUERY
#----------------------------------

## run a query
rows = cursor.execute("SELECT * FROM \"@dremio.demo@gmail.com\".\"nyc-taxi-data\" limit 1000000").fetchall()

##convert into pandas dataframe
df = pd.DataFrame([tuple(t) for t in rows])

print(df)


```


### pyArrow

```py
#----------------------------------
# IMPORTS
#----------------------------------
## Import Pyarrow
from pyarrow import flight
from pyarrow.flight import FlightClient

## import pandas
import pandas as pd

## Get environment variables
from os import environ

const token = environ.get('token', 'no personal token defined')

#----------------------------------
# Setup
#----------------------------------

## Headers for Authentication
headers = [
    (b"authorization", f"bearer {token}".encode("utf-8"))
    ]

## Create Client
client = FlightClient(location=("grpc+tls://data.dremio.cloud:443"))

#----------------------------------
# Function Definitions
#----------------------------------

## makeQuery function
def make_query(query, client, headers):
    

    ## Get Schema Description and build headers
    flight_desc = flight.FlightDescriptor.for_command(query)
    options = flight.FlightCallOptions(headers=headers)
    schema = client.get_schema(flight_desc, options)

    ## Get ticket to for query execution, used to get results
    flight_info = client.get_flight_info(flight.FlightDescriptor.for_command(query), options)
    
    ## Get Results 
    results = client.do_get(flight_info.endpoints[0].ticket, options)
    return results

#----------------------------------
# Run Query
#----------------------------------


results = make_query("SELECT * FROM \"@dremio.demo@gmail.com\".\"nyc-taxi-data\" limit 1000000", client, headers)

# convert to pandas dataframe
df = results.read_pandas()

print(df)

```
