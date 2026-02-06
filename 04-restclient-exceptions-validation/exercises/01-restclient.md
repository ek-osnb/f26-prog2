# RestClient exercise

In this exercise you will practice using Spring Boot's RestClient to make HTTP requests to a public API. You will implement several functions that utilize RestClient to fetch and process data.

## Introduction
Your task is to call three different APIs using RestClient and combine the results, into a single response.
- https://agify.io
- https://genderize.io
- https://nationalize.io

**Note that these APIs are free and public, but may have a limit on the number of requests you can make in a certain time period.**

You should create a REST API that should accept a name as a query parameter and return a JSON response combining the results from the three APIs. Note that it doesn't make sense to call a `GET` endpoint with a body, so you should pass the name as a query parameter, like this:
- `GET /persondata?firstName=Harry&lastName=Potter`


### Response format
The combined response should be in the following format:
```json
{
  "fullName": "Harry James Potter",
  "firstName": "Harry",
  "middleName": "James",
  "lastName": "Potter",
  "gender": "male",
  "genderProbability": 0.89,
  "age": 17,
  "ageProbability": 0.45,
  "country": "uk",
  "countryProbability": 0.93
}
```

### Tasks
- Set up a new Spring Boot project with Web dependency.
- Create a RestClient bean to be used for making HTTP requests.
- Implement a service that uses RestClient to call the three APIs and fetch data based on the provided name.
- Combine the results from the three APIs into a single response object.
- Create a REST controller that exposes an endpoint to accept the name as a query parameter and returns the combined response.
- **The application should cache previous results for the same name to avoid redundant API calls.**
    - How you implement the caching is up to you (in-memory, database).
- If there is an attribute that is missing, it should be set to `null` in the response.