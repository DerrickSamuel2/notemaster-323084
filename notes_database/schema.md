# NoteMaster database schema (PostgreSQL)

Connection info is available in `db_connection.txt` (example):
`psql postgresql://appuser:dbuser123@localhost:5000/myapp`

## Tables

- `users`
  - `id` (PK)
  - `email` (unique)
  - `password_hash`
  - `created_at`
- `notes`
  - `id` (PK)
  - `user_id` (FK -> users.id)
  - `title`, `content`
  - `pinned`, `favorited`
  - `created_at`, `updated_at`
- `tags`
  - `id` (PK)
  - `user_id` (FK -> users.id)
  - `name` (unique per user)
  - `created_at`
- `note_tags`
  - `note_id` (FK -> notes.id)
  - `tag_id` (FK -> tags.id)
  - composite PK (`note_id`, `tag_id`)

## Initialization commands (run ONE statement at a time)

Use the command from `db_connection.txt` and pass a `-c "..."` argument.

1) users
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS users (id SERIAL PRIMARY KEY, email VARCHAR(255) NOT NULL UNIQUE, password_hash VARCHAR(255) NOT NULL, created_at TIMESTAMPTZ NOT NULL DEFAULT NOW());"
```

2) notes
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS notes (id SERIAL PRIMARY KEY, user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE, title VARCHAR(120) NOT NULL DEFAULT '', content TEXT NOT NULL DEFAULT '', pinned BOOLEAN NOT NULL DEFAULT FALSE, favorited BOOLEAN NOT NULL DEFAULT FALSE, created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(), updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW());"
```

3) tags
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS tags (id SERIAL PRIMARY KEY, user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE, name VARCHAR(32) NOT NULL, created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(), CONSTRAINT uq_tags_user_name UNIQUE (user_id, name));"
```

4) note_tags
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE TABLE IF NOT EXISTS note_tags (note_id INTEGER NOT NULL REFERENCES notes(id) ON DELETE CASCADE, tag_id INTEGER NOT NULL REFERENCES tags(id) ON DELETE CASCADE, PRIMARY KEY (note_id, tag_id));"
```

5) indexes
```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE INDEX IF NOT EXISTS idx_notes_user_updated ON notes(user_id, updated_at DESC);"
```

```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE INDEX IF NOT EXISTS idx_note_tags_tag ON note_tags(tag_id);"
```

```bash
psql postgresql://appuser:dbuser123@localhost:5000/myapp -c "CREATE INDEX IF NOT EXISTS idx_note_tags_note ON note_tags(note_id);"
```
