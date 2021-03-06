# Munro Library Challenge

## The Challenge

Provided in this archive is a CSV file containing information about munros and munro tops
within Scotland. The goal of your solution is to create a simple API which other software can use
to sort and filter the munro data. Your solution should be developed using either Java or Kotlin.
You are welcome to use any relevant libraries or frameworks, including application frameworks
such as Spring or Micronaut. However, you should not use a database (in-memory or otherwise)
to implement the search functionality.

The API should provide the following functionality:
- Filtering of search by hill category (i.e. Munro, Munro Top or either). If this information is not provided by the 
  user it should default to either. This should use the “post 1997” column and if it is blank the hill should be always 
  excluded from results.
- The ability to sort the results by height in meters and alphabetically by name. For both options it should be possibly
  to specify if this should be done in ascending or descending order.
- The ability to limit the total number of results returned, e.g. only show the top 10
- The ability to specify a minimum height in meters
- The ability to specify a maximum height in meters
- Queries may include any combination of the above features and none are mandatory.
- Suitable error responses for invalid queries (e.g. when the max height is less than the minimum height) 
  
Optionally you may choose to include the following feature if you can think of a good approach but it is not required to
complete the solution. Correctness and structure of the rest of your solution is more important than adding this extra 
objective:
- The ability to combine sort criteria in order of preference. For example: sort by height descending and then 
  alphabetically by name as a secondary criteria (for when the height is the same) The query results should be returned 
  as a list of items using JSON. Each item should contain the name, height in meters, hill category and grid reference 
  (e.g. NN773308). Other fields should not be included.

Notes:
- Parsed Munro data should be held in memory, without the use of a database.
- The munro data should be loaded from the local CSV file on API startup.
- There is no need to add authentication or rate limiting to endpoints developed.
- Please include any testing code you write in your submitted solution.
- Whilst developing your solution please commit your work into a git repository as you go. This is not to see how much 
  time you take or at what times you worked on the solution but is so that we can evaluate how you broke down and 
  approached the problem.

## CI/CD Setup

This project uses CircleCI to run the Continuous Integration and Deployment to Heroku. There are three parts to the CI
process:
1. build_test - Runs `maven verify` to build and execute tests.
2. docker/publish - Runs `Docker build` to create a Docker image containing the fat jar, then `Docker push` to upload it
   to the registry.
3. heroku_deploy - Runs `heroku container:release` to update the Heroku Dyno to run the latest version

### Step 1 build_test

This can be run locally as:

```
mvn verify 
```

Internally this executes `validate`, `compile`, `package`, then `verify`. Together this gives confidence in the quality
of the change.

### Step 2 docker/publish

This executes a default CircleCI Orb to build and push a docker image to the heroku image registry. Likewise this can
be achieved locally by:

```
docker login -i registry.heroku.com
docker build -t registry.heroku.com/$HEROKU_USERNAME/$HEROKU_APP_NAME/web:latest ./Dockerfile
docker push push registry.heroku.com/$HEROKU_USERNAME/$HEROKU_APP_NAME/web:latest
```

### Step 3 heroku_deploy

This is the final step that deploys the newly published docker image to the Heroku Dyno. This step can be achieved 
locally with:

```
heroku container:login
heroku container:release -a $HEROKU_APP_NAME web
```

## The Deployed Application

The deployed application can be accessed at anytime at: https://munro-library-challenge.herokuapp.com

## Swagger 2.0

The application provides Swagger 2.0 documentation. This can be accessed directly at: 
https://munro-library-challenge.herokuapp.com/v2/api-docs or using Swagger UI at:
https://munro-library-challenge.herokuapp.com/swagger-ui/#/ This can be useful for understanding the API


## The API

_It is advisable to use the Swagger UI to determine the correct structure for the requests._

### GET /munros

This path provides access to the full list of Munros in the CSV file. It offers the option to then page, filter, and order the results.

#### Paging

Two parameters are required to page the results:

- `page` - this is the zero indexed page number, this should initially be `0`
- `limit` - this is the maximum number of munros to return in a single request. This must be greater than `0`

The result will have the HTTP Header `Link` that follows RFC8288 (Section 3) and is in the format:

```
<{url}> rel="{relation}"
```

where the `{url}` component is the URL to be followed, and the `{relation}` component is the relation of that page to this, one of `prev`, `next`, `first`, `last`

#### Ordering

It is possible to order the response by either name, height, or both (where the order of the components dictate the preference in ordering).
The format for this parameter is:

```
{property};{direction}
```

Property must be one of `name` or `height`, and direction one of `asc` or `desc`, or can be omitted completely. Multiple orders should be comma separated.

This parameter should be passed as `order`

#### Querying

There are 3 query parameters:

- `category` - filter by hill category, this should be one of `MUNRO`, `TOP`
- `minHeight` - filter by a minimum inclusive height.
- `maxHeight` - filter by a maximum inclusive height.

### Some example queries:

- https://munro-library-challenge.herokuapp.com/munros - the full dataset
- https://munro-library-challenge.herokuapp.com/munros?page=0&limit=10 - the full dataset paged
- https://munro-library-challenge.herokuapp.com/munros?page=0&limit=10&order=height;desc,name - the full dataset paged and ordered by height descending and then by name ascending
- https://munro-library-challenge.herokuapp.com/munros?order=name&category=MUNRO&minHeight=918&maxHeight=918 - all 918m munros ordered by name ascending.