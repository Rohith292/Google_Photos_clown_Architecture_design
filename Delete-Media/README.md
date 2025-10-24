# F1.5: Delete Media - Architecture Summary

This is a secure, transactional delete to ensure data integrity and give the user their storage space back.

The app sends a `DELETE /photos/{id}` request. A Lambda (`deletePhotoLambda`) first performs an ownership check (just like the download flow).

If the check passes, the Lambda executes a database **transaction** to:
1.  `UPDATE` the `users` table to *subtract* the photo's `file_size` from their `storage_used`.
2.  `DELETE` the record from the `photos` table.

After this transaction successfully commits, the Lambda sends a final `DeleteObject` command to S3 to remove the file.