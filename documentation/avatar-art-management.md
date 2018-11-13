# Avatar Art Management

<!-- MarkdownTOC -->

- Managing piece display name, price, and active status
- About deactivating a piece
- About removing a piece

<!-- /MarkdownTOC -->

## Managing piece display name, price, and active status

- Log into [lms](http://lms.thinkthroughmath.com) as a superuser.
- Visit [avatar_pieces](http://lms.thinkthroughmath.com/avatar_pieces), where you can browse pieces by category and edit their price, active status, and display name.

## About deactivating a piece

### Why would you deactivate a piece?
A piece should be __deactivated__ when a piece is only intended to be available in the store for a certain period of time (eg. holidays, contests, special events). __Deactivating__ is also a gentle way to retire pieces that are in use but that we eventually want to remove from the store.

### Consequences of deactivating a piece
- Deactivating a piece is a global operation -- the piece will be deactivated for *all* students in *all* classrooms / schools / customers.
- The piece will no longer be available for purchase in the Avatar Builder store.
- No refunds will be given. 
- Students who have already purchased the deactivated piece 
  - __will still have that piece__; it will show up in *their* version of the Avatar Builder and on their Avatar if they are wearing it.
  - can continue to take it on and off.
- The piece *can be reactivated* (same process as deactivating, but updating active to 'true', at which point it will again be available for purchase in the Avatar Builder store by any student.

## About removing a piece

### Why would you remove a piece?
A piece should be __removed__ if it is deemed offensive or inappropriate and we need to permanently and completely eradicate it from the system. Also, unused pieces that *no one owns* can be __removed__ for housekeeping purposes without any negative consequences.

### How to remove a piece
- Make a pull request to remove the piece from [the avatar piece source directory](https://github.com/thinkthroughmath/apangea/tree/rc/app/assets/images/avatar_pieces/source). 
- Run `rake avatar_builder:load_pieces` to update [avatar_piece_information.csv](https://github.com/thinkthroughmath/apangea/blob/rc/app/assets/images/avatar_pieces/avatar_piece_information.csv), and include any changes to that csv file in the pull request.
- When the PR is merged and deployed the piece will be removed on production.
- Make sure that the source .ai file is also removed from [the Avatar Art Google Drive folder](https://drive.google.com/drive/u/1/folders/0Bz6JiTiQXxg4flhXYy1vU2NnMHdLOVhuUG04cEttQ3pwV3M0TEdLb3NfWTZpRVVlQTBIMTQ), so it isn't accidentally added back in when we reprocess the art. We've been using [this](https://drive.google.com/drive/u/1/folders/0B2vOdWgW-uG5fm81WDJlVkloTUI0MlJtT09oTVdLcmkxcV9TSGRqSVBudmdJQVhLWURvRVk) as the dumping-ground for misfit pieces.

### Consequences of removing a piece
- Removing a piece is a global operation -- the piece will be removed for *all* students in *all* classrooms / schools / customers.
- The piece will no longer be available for purchase in the Avatar Builder store.
- No refunds will be given, although it is possible to use the `PointCorrector` to do this manually. *We'd like to avoid refunds for as long as we can!* Please see [Student Point Management](student-point-management.md).
- The piece will no longer exist in the database or spritesheets at all, so __students who have purchased the piece will no longer have access to it, and their avatars will no longer be wearing it__.
