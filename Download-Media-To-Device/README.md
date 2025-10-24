# F1.4: Download Media - Architecture Summary

This is the secure "read" equivalent of the upload flow, designed to protect our private S3 bucket.

The app requests to download a specific `photo_id`. A Lambda function (`getDownloadUrlLambda`) receives this request.

Its first and most important job is to query the `photos` table to verify that the `owner_id` of that photo matches the `user_id` from the JWT. If the check fails, it returns a `404 Not Found` error. If it passes, the Lambda generates a temporary, *read-only* pre-signed S3 URL and returns it to the app for the download.