# F1.6: Basic Album Creation - Architecture Summary

This is a simple database "write" operation that creates a new, empty album.

The app sends a `POST /albums` request with a new `album_name`. A Lambda function (`createAlbumLambda`) validates the JWT and `INSERT`s a new row into the `albums` table, linking it to the user's `owner_id`.

This feature also introduces the `album_photos` "join table" to our database schema, which will be used by future features to link photos to albums in a many-to-many relationship.