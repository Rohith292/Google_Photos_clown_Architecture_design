# F1.2: Secure Photo Upload - Architecture Summary

This is a 3-part asynchronous flow to ensure security, prevent storage quota violations, and handle failed uploads.

1.  **Permission:** The app requests a pre-signed S3 URL, sending the `file_size`. A Lambda function validates the user's total available space (including other pending uploads) and `INSERT`s a temporary record into a `pending_uploads` table with a timestamp.
2.  **Finalization:** The app uploads the file directly to S3. This triggers a second Lambda (`processPhotoLambda`), which performs a database transaction to `INSERT` the `photos` record, `UPDATE` the user's `storage_used`, and `DELETE` the record from `pending_uploads`.
3.  **Cleanup:** A third Lambda (`cleanupOrphanedUploadsLambda`) is triggered by **Amazon EventBridge** every hour. Its only job is to `DELETE` any "orphaned" records from `pending_uploads` that are more than 1 hour old, returning the reserved space to the user.