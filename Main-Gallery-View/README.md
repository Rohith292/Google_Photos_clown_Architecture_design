# F1.3: Main Gallery View - Architecture Summary

This is a simple, secure read operation. The React Native app makes a `GET /photos` request to API Gateway.

The Gateway validates the user's JWT (returning `401` if invalid). If valid, it triggers a Lambda function (`getPhotosLambda`).

This Lambda extracts the `user_id` from the JWT and executes a `SELECT` query on the PostgreSQL `photos` table, returning a JSON list of all records where the `owner_id` matches the user's ID. The Lambda handles any database errors by returning a `500` status code.