## Query Dremio in Other Languages

Here is sample reference code that can help in Querying Dremio in other languages. You may have to edit certain variables to fit your particular use case, but this should be helpful in developing the functions and code you need.


### Using Go with Arrow Flight into an Arrow Flight Table

```go
package main

import (
	"context"
	"fmt"
	"log"

	"github.com/apache/arrow/go/arrow/flight"
	"github.com/apache/arrow/go/arrow/memory"
	"github.com/apache/arrow/go/arrow/array"
	"google.golang.org/grpc"
)

type basicAuth struct {
	username string
	password string
}

func (b *basicAuth) Authenticate(ctx context.Context, c flight.AuthConn) error {
	c.Send([]byte(b.username + ":" + b.password))
	return nil
}

func (b *basicAuth) IsValid(token string) (interface{}, error) {
	return nil, nil
}

func main() {
	ctx := context.Background()

	authHandler := &basicAuth{
		username: "username",
		password: "password",
	}

	client, err := flight.NewClientWithMiddleware("yourDremioEndpoint", authHandler, nil, grpc.WithInsecure())
	if err != nil {
		log.Fatal(err)
	}
	defer client.Close()

	// Authenticate with the server.
	if err := client.Authenticate(ctx); err != nil {
		log.Fatal(err)
	}

	// Send SQL query to the server.
	desc := &flight.FlightDescriptor{
		Type: flight.FlightDescriptor_CMD,
		Cmd:  []byte("yourSQLQuery"),
	}

	// Retrieve the schema of the result.
	info, err := client.GetFlightInfo(ctx, desc)
	if err != nil {
		log.Fatal(err)
	}

	// Retrieve the result.
	stream, err := client.DoGet(ctx, info.Endpoint[0].Ticket)
	if err != nil {
		log.Fatal(err)
	}

	mem := memory.NewGoAllocator()

	// Load the result into an Arrow table.
	table := array.NewTableFromRecord(mem, stream.Schema(), stream)
	fmt.Println(table)
}
```

### Using Javascript and Dremio's Rest API to Populate a Chart.js Chart
(Note: Assumes propers CORS permissions, you may have to do the fetch serverside then retrieve the data from your server in your frontend code)

```js
// Fetch data from Dremio's REST API
async function fetchData() {
  const endpoint = 'yourDremioEndpoint';  // Replace with your Dremio endpoint
  const username = 'yourUsername';  // Replace with your Dremio username
  const password = 'yourPassword';  // Replace with your Dremio password
  const sqlQuery = 'yourSQLQuery';  // Replace with your SQL query

  const auth = 'Basic ' + btoa(username + ':' + password);

  const response = await fetch(endpoint + '/api/sql', {
    method: 'POST',
    headers: {
      'Authorization': auth,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({ sql: sqlQuery })
  });

  if (!response.ok) {
    throw new Error('HTTP error ' + response.status);
  }

  return await response.json();
}

// Create chart with Chart.js
async function createChart() {
  const data = await fetchData();

  const labels = data.columns;
  const datasets = data.values.map((value, index) => ({
    label: labels[index],
    data: value
  }));

  const ctx = document.getElementById('myChart').getContext('2d');
  new Chart(ctx, {
    type: 'line',
    data: {
      labels: labels,
      datasets: datasets
    },
    options: {
      responsive: true,
      title: {
        display: true,
        text: 'Dremio Data'
      }
    }
  });
}

// Call createChart when the page is fully loaded
window.addEventListener('load', createChart);
```


