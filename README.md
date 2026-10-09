# Live MongoDB Data Visualization with R

A Spring Boot application that reads numeric values from MongoDB and renders a continuously updating SVG chart with R. Java handles data access and request processing, while GraalVM Polyglot executes the R plotting function.

## Features

- Reads one MongoDB document at a time in ascending `_id` order
- Cycles back to the first document after reaching the end of the collection
- Maintains a rolling window of the latest 100 numeric values
- Uses R's `lattice::xyplot()` to generate the chart
- Returns the chart as SVG from a Spring MVC endpoint
- Refreshes the visualization every second

## Architecture

```text
Browser
   | GET /plot
   v
Spring Boot / PlotController
   |
   +--> MongoDataService --> MongoDB
   |                         (next numeric value)
   |
   +--> GraalVM Polyglot --> plot.R --> lattice::xyplot()
                                |
                                v
                              SVG
```

## Technology Stack

- Java 17
- Spring Boot 4.1.1
- Spring Web MVC
- MongoDB
- GraalVM SDK 22.3.0
- FastR
- R `lattice` package
- Maven

## Prerequisites

Install the following before running the application:

- A Java 17 GraalVM distribution compatible with GraalVM SDK 22.3.0
- FastR installed in GraalVM
- Maven 3.8 or newer, or use the included Maven Wrapper
- MongoDB running locally or on a reachable server

Verify that Java and R are available:

```bash
java -version
R --version
R -e 'library(lattice); print("lattice is available")'
```

## Configuration

Application settings are stored in `src/main/resources/application.properties`:

```properties
spring.application.name=spring_r
app.mongo.uri=mongodb://localhost:27017
app.mongo.database=admin
app.mongo.collection=veri
app.column=Col-8
```

| Property | Description |
| --- | --- |
| `app.mongo.uri` | MongoDB connection URI |
| `app.mongo.database` | Database containing the source data |
| `app.mongo.collection` | Collection read by the application |
| `app.column` | Numeric field plotted on the chart |

Update these values to match your MongoDB environment.

## MongoDB Data

Each document must contain the numeric field configured by `app.column`. For example:

```json
{
  "Col-8": 12.75
}
```

The application sorts documents by `_id`, reads them sequentially, and loops back to the beginning when it reaches the end.

## Running the Application

Clone the repository:

```bash
git clone https://github.com/HarunIlkerKaya/SWE307_Project1_FINAL.git
cd SWE307_Project1_FINAL
```

Start MongoDB, then run the application:

```bash
./mvnw spring-boot:run
```

On Windows:

```powershell
.\mvnw.cmd spring-boot:run
```

Open the visualization:

```text
http://localhost:8080/plot
```

The endpoint returns an SVG image and instructs the browser to refresh it every second.

## Project Structure

```text
src/
├── main/
│   ├── java/org/graalvm/demos/spring_r/
│   │   ├── DataHolder.java
│   │   ├── MongoDataService.java
│   │   ├── PlotController.java
│   │   └── SpringRApplication.java
│   └── resources/
│       ├── application.properties
│       └── plot.R
└── test/
    └── java/org/graalvm/demos/spring_r/
        └── SpringRApplicationTests.java
```

## How the Plot Is Generated

1. The browser requests `GET /plot`.
2. `MongoDataService` reads the next configured numeric value from MongoDB.
3. `PlotController` passes the value to the R function through GraalVM Polyglot.
4. `plot.R` updates its 100-value rolling window.
5. `lattice::xyplot()` renders a dark-brown line chart with a grid.
6. The generated SVG is returned to the browser.

## Troubleshooting

### The MongoDB collection is empty

Import data into the configured database and collection, then confirm that each document contains the field named by `app.column`.

### The configured field cannot be read

Make sure the selected MongoDB field is numeric in every document.

### R or `lattice` is unavailable

Run the application with the intended GraalVM installation and confirm that FastR and `lattice` are installed.

### Port 8080 is already in use

Stop the process using the port or add a different port to `application.properties`:

```properties
server.port=8081
```
