PhotoSync (AWS Cloud Clone) - Project Architecture

This repository contains the full architectural design and planning documents for the PhotoSync project, an AWS-based clone of Google Photos.

The core project plan can be found in [PhotoSyncc_phase1.docx](<link-to-your-doc-or-just-the-filename>).

All detailed architectural diagrams (User Flows, Server-Side Architectures, and Block Diagrams) are located in the /architecture directory, organized by feature.

Phase 1: MVP Feature Architecture

This is a brief overview of the architectural solution for each core feature of our Minimum Viable Product.

F1.1: User Authentication

Solution: We are federating our identity to Google using Amazon Cognito. This avoids storing passwords. When a new user signs in, a Cognito Lambda Trigger automatically runs to INSERT their profile into our PostgreSQL users table, syncing our database and setting their default storage quota.

F1.2: Secure Photo Upload

Solution: This is a 3-part asynchronous flow to ensure security and prevent data loss.

Permission: The app asks a Lambda for a pre-signed S3 URL. The Lambda checks the user's storage quota and creates a temporary record in a pending_uploads table before returning the URL.

Finalization: The app uploads the file directly to S3. This triggers a second Lambda, which performs a database transaction to move the record from pending_uploads to the final photos table and update the user's storage_used.

Cleanup: A third Lambda runs on a schedule (via Amazon EventBridge) to find and delete any "orphaned" records from pending_uploads that are more than 1 hour old (i.e., failed uploads).

F1.3: Main Gallery View

Solution: A simple, secure read operation. The app makes a GET request to API Gateway, which validates the user's JWT. A Lambda function then queries the PostgreSQL photos table, returning a JSON list of all records where the owner_id matches the user's ID.

F1.4: Download Media to Device

Solution: This is the secure "read" equivalent of the upload flow. The app asks a Lambda for a specific photo_id. The Lambda first queries the photos table to verify the owner_id matches the user's ID. If the check passes, it generates a temporary, read-only pre-signed S3 URL and returns it to the app.

F1.5: Delete Media

Solution: A secure, transactional delete to ensure data integrity. The app asks a Lambda to delete a photo_id. The Lambda first checks for ownership. If the check passes, it performs a database transaction to (1) DELETE the record from the photos table and (2) UPDATE the users table to subtract the file_size from their storage_used. After this transaction commits, the Lambda sends a final DeleteObject command to S3.

F1.6: Basic Album Creation

Solution: A simple database write. The app sends a new album_name. A Lambda function INSERTs a new row into the albums table, linking it to the user's owner_id. This creates a new, empty album.

F1.7: Storage Quota Management

Solution: This is a business rule, not a standalone feature. It is implemented within:

F1.2 (Upload): Logic checks the storage_used + pending_uploads before granting an upload URL.

F1.5 (Delete): Logic subtracts the file_size from storage_used to return space to the user.

F1.8: "Free Up Space" Utility

Solution: This is a client-side (React Native) feature. The server is not involved. The app will:

Call our F1.3 API to get a list of all synced photos.

Compare this "synced" list against the phone's local camera roll.

Create a list of all photos that exist in both places.

Ask the user for permission to delete the local copies of these backed-up photos.
