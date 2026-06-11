# week-18-prisma

This folder contains a Prisma + PostgreSQL example using Prisma 6.x + the new `@prisma/adapter-pg` adapter.

## What this project does

- Defines a PostgreSQL database schema in `prisma/schema.prisma`
- Uses Prisma Client generated into `src/generated/prisma`
- Connects to PostgreSQL using the `DATABASE_URL` environment variable
- Demonstrates CRUD operations on `User` and `ToDos` models in `src/index.ts`

---

## Prerequisites

- Node.js and npm installed
- PostgreSQL server available locally or remotely
- A `DATABASE_URL` connection string for PostgreSQL

Example local PostgreSQL connection string:

```bash
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

Example Neon/Supabase connection string:

```bash
DATABASE_URL="postgresql://username:password@host:port/dbname?sslmode=require"
```

---

## Setup steps

### 1. Install dependencies

From `week-18-prisma`:

```bash
npm install
```

This installs:

- `prisma` - Prisma CLI / tooling
- `@prisma/client` - generated Prisma Client runtime
- `@prisma/adapter-pg` - PostgreSQL adapter for Prisma Client
- `pg` and `@types/pg` - PostgreSQL driver and types
- `dotenv` - environment variable loader
- `typescript` and `@types/node`

### 2. Create `.env`

Create or update `week-18-prisma/.env` with your PostgreSQL connection string:

```bash
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

If you are using a remote provider, include `sslmode=require` when needed.

### 3. Confirm Prisma config

The project has `prisma.config.ts` configured to read `DATABASE_URL` from `.env`:

- `schema: "prisma/schema.prisma"`
- `migrations.path: "prisma/migrations"`
- `engine: "classic"`
- `datasource.url: env("DATABASE_URL")`

This means Prisma will use the schema file and migrations folder in the `prisma/` directory.

### 4. Inspect the schema

Open `prisma/schema.prisma`. It defines two models:

- `User`
  - `id` autoincrement primary key
  - `username` unique string
  - `password` string
  - `age` integer
  - `city` optional string
  - relation to `ToDos[]`

- `ToDos`
  - `id` autoincrement primary key
  - `title` string
  - `description` string
  - `done` boolean
  - `userId` foreign key referencing `User.id`

### 5. Generate Prisma Client

From `week-18-prisma`:

```bash
npx prisma generate
```

This generates the client into `src/generated/prisma`, matching the `generator` output path in `prisma/schema.prisma`.

### 6. Apply the schema to the database

Use Prisma migrations to create the tables in your database:

```bash
npx prisma migrate dev --name init
```

What happens:

- Prisma reads `prisma/schema.prisma`
- Creates a new migration under `prisma/migrations`
- Applies the migration to the database
- Updates the `_prisma_migrations` table
- Keeps the database schema in sync with the Prisma schema

If you only want to push the schema without generating a migration, use:

```bash
npx prisma db push
```

### 7. Build and run the app

The project script compiles TypeScript and runs `dist/index.js`:

```bash
npm run dev
```

This executes the code in `src/index.ts` after compilation.

---

## Understanding `src/index.ts`

This file demonstrates connecting to Postgres and using Prisma CRUD operations.

### Environment loading

```ts
import dotenv from 'dotenv'
dotenv.config()
```

This loads `.env` values into `process.env`.

### Connection string validation

```ts
const connectionString = process.env.DATABASE_URL
if (!connectionString) {
  throw new Error("DATABASE_URL is not defined in environment variables")
}
```

This ensures the app does not run without a valid connection string.

### Prisma adapter and client

```ts
const adapter = new PrismaPg({ connectionString })
const prisma = new PrismaClient({ adapter, log: [...] })
```

- `PrismaPg` is the adapter for PostgreSQL
- `PrismaClient` is the generated Prisma client
- `log` is configured so SQL queries, info, warnings, and errors are printed to stdout

### CRUD functions

#### `createUser()`

Creates a new `User` record:

```ts
await prisma.user.create({
  data: {
    username: "Dad",
    age: 34,
    password: "12345",
    city: "Bangalore"
  }
})
```

This inserts one user into the `User` table.

#### `deleteUser()`

Deletes a user by primary key:

```ts
await prisma.user.delete({ where: { id: 1 } })
```

#### `updateUser()`

Updates a single user:

```ts
await prisma.user.update({
  where: { id: 2 },
  data: { username: "random" }
})
```

#### `getUser()`

Reads one user and returns only the `username` field:

```ts
const user = await prisma.user.findFirst({
  where: { id: 1 },
  select: { username: true }
})
```

This is a projection query that returns only selected fields.

#### `getUserwithTodos()`

Reads a user and includes related todos:

```ts
include: { todos: true }
```

This returns the user plus all related `ToDos` records.

#### `getUserWithTodosSpecificFields()`

Returns a user with a nested selection of `todos.description` only:

```ts
select: {
  username: true,
  todos: {
    select: { description: true }
  }
}
```

This demonstrates how to query specific fields from related models.

#### `deleteUsers()`

This function is defined but currently performs a find operation rather than a delete. To delete by condition, use `prisma.user.deleteMany(...)`.

### Running a query

The file currently calls:

```ts
getUserWithTodosSpecificFields()
```

So if you run the app, it will execute that function and print the resulting user with selected todo descriptions.

---

## Common commands summary

```bash
npm install
npx prisma generate
npx prisma migrate dev --name init
npm run dev
```

If you need to inspect the database schema:

```bash
npx prisma db pull
```

If you want to run raw SQL migrations or introspect a database:

```bash
npx prisma migrate status
npx prisma studio
```

---

## Notes

- Keep your `DATABASE_URL` private.
- If you use a hosted database, add the required `sslmode` query parameter if needed.
- `prisma.schema` and `prisma.config.ts` are the central files controlling model definitions and database connection.
- After changing the schema, regenerate the client and rerun migrations.
