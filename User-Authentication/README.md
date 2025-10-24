# F1.1: User Authentication - Architecture Summary

This feature provides secure user login and registration by federating our identity to Google using **Amazon Cognito**.

When a *new* user signs in for the first time, a **Cognito Post-Confirmation Lambda Trigger** is fired. This Lambda's job is to `INSERT` the new user's profile (their `user_id` and `email`) into our PostgreSQL `users` table, which syncs our database and sets their default storage quota.

Once authenticated, Cognito provides the app with a secure **JWT** to use for all future API calls.