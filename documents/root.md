# Prisma

## INSTALLATION

### TypeScript

```javascript
// Initialize a TypeScript project using npm

npm install typescript ts-node @types/node --save-dev

// Initialize TypeScript

px tsc --init
```

### Prisma Setup

```javascript
// Install Prisma
npm install prisma --save-dev

// Install prisma client
npm install @prisma/client
```

### Data Sources And Providers

You can only have one datasource block in a schema.

```javascript
npx prisma init --datasource-provider <name>

npx prisma init --datasource-provider postgresql
```

To connect your database, you need to set the `url` field of the `datasource` block in your Prisma schema to your database connection URL:

```prisma
// 📁 prisma/schema.prisma

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

```javascript
// 📁.env

DATABASE_URL =
  "postgresql://johndoe:randompassword@localhost:5432/database_name?schema=public";

/**
USER:       The name of your database user
PASSWORD:   The password for your database user
HOST:       The name of your host name (for the local environment, it is localhost)
PORT:       The port where your database server is running (typically 5432 for PostgreSQL)
DATABASE:   The name of the database
SCHEMA:     The name of the schema inside the database
*/
```

### Migrate

```javascript
npx prisma migrate dev --name init
```

### Generator

```javascript
npx prisma generate
```

## Prisma Schema

If the file is named differently, you can provide the `--schema` argument to the Prisma CLI with the path to the schema file

```bash
prisma generate --schema ./database/myschema.prisma
```

## Data Models

Every record of a model must be uniquely identifiable. You must define at least one of the following attributes per model:

```prisma
@unique
@@unique
@id
@@id
```

With this model definition, Prisma automatically maps the Comment model to the comments table in the underlying database.

_Note: You can also @map a column name or enum value, and @@map an enum._

### **Datatype**

- `String`

  ```prisma
    model String {
      name  String  @db.Text
      motto String  @db.Char(55)
      bio   String  @db.Varchar(505)
      is    String  @db.bit(1)
    }
  ```

- `Int` | `BigInt` | `Float` | `Decimal`

  ```prisma
  model Integer {
    students  Int   @db.Integer
    age       Int   @db.SmallInt
    gpa       Float
  }
  ```

- `Boolean`
- `DateTime`

### **Model field Type Modifiers**

[ref](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference#model-field-type-modifiers)

- `[]` modifier

  Makes a field a list.

  ```prisma
  model User {
    id             Int      @id @default(autoincrement())
    favoriteColors String[] @default(["red", "blue", "green"])
  ```

- `?` modifier

  Makes a field optional.

  ```prisma
  model User {
    id   Int     @id @default(autoincrement())
    name String?
  }
  ```

### **Attributes**

Attributes modify the behavior of a field or block (e.g. models).

- Field attributes are prefixed with `@`
- Block attributes are prefixed with `@@`

Some attributes take arguments. Arguments in attributes are always named, but in most cases the argument name can be omitted.

_Note: The leading underscore in a signature means the argument name can be omitted._

- `@id`

  Defines a single-field ID on the model.

  ❌ Cannot be defined on a relation field.

  ❌ Cannot be optional

  Can be annotated with a `@default()` value that uses functions to auto-generate an ID:

  - `autoincrement()`
  - `cuid()`
  - `uuid()`

  Can be defined on any scalar field (`String`, `Int`, `enum`)

- `@@id` | composite ID

  Defines a multi-field ID (composite ID) on the model.

  Corresponding database type: `PRIMARY KEY`
  Can be annotated with a `@default()` value that uses functions to auto-generate an ID

  Can be defined on any scalar field (`String`, `Int`, `enum`)

  ```prisma
  model User {
    firstName String
    lastName  String
    email     String  @unique
    isAdmin   Boolean @default(false)

    @@id([firstName, lastName])
  }
  ```

  When you create a user, you must provide a unique combination of firstName and lastName:

- `@default`

  Defines a default value for a field.

  Corresponding database type: `DEFAULT`

  - `autoincrement()`
  - `cuid()`
  - `uuid()`
  - `dbgenerated()`
  - `now()`

  ```prisma
  model User {
    email           String    @unique
    profileViews    Int       @default(0)
    number          Float     @default(1.1)
    name            String    @default("")
    isAdmin         Boolean   @default(false)
    data            DateTime  @default("2020-03-19T14:21:00+02:00")
    role            Role      @default(USER)
    favoriteColors  String[]  @default(["red", "yellow", "purple"])
  }
  ```

- `@@unique`

  Block attributes.

  Defines a compound unique constraint for the specified fields.

  All fields that make up the unique constraint must be mandatory fields.

  Corresponding database type: `UNIQUE`

  ```prisma
  model User {
    id        Int     @default(autoincrement())
    firstName String
    lastName  String
    isAdmin   Boolean @default(false)

    @@unique([firstName, lastName])
    @@unique(fields: [firstName, lastName, isAdmin], name: "admin_identifier")
  }
  ```

- `@@index`

  Block attributes.

  Defines an index in the database.

  Corresponding database type: `INDEX`

  ```prisma
  model Post {
    id Int @id @default(autoincrement())
    title String
    content String?

    <!-- Define a single-column index -->
    @@index([title])

    <!-- Define a multi-column index -->
    @@index([title, content])

    <!-- Define an index with a name -->
    @@index(fields: [title, content], name: "main_index")
  }
  ```

- `@relation`

  Defines meta information about the relation. Learn more.

  Corresponding database types: `FOREIGN KEY` / `REFERENCES`

- `@@map`

  Maps the Prisma schema model name to a table (relational databases) or collection (MongoDB) with a different name, or an enum name to a different underlying enum in the database.

  ```prisma
  model User {
    id   Int    @id @default(autoincrement())
    name String

    @@map("users")
  }

  enum Role {
    ADMIN     @map("admin")
    CUSTOMER  @map("customer")

    @@map("_Role")
  }
  ```

### **Attribute functions**

- `autoincrement()`

  ```prisma
  model User {
    id   Int    @id @default(autoincrement())
    name String
  }
  ```

- `now()`

  Set a timestamp of the time when a record is created.

  Compatible with DateTime

## Relation

### One-to-one Relations

One-to-one (1-1) relations refer to relations where at most **one** record can be connected on both sides of the relation. In the example below, there is a one-to-one relation between `User` and `Profile`:

```prisma
model User {
  id      Int      @id @default(autoincrement())
  profile Profile?
}

model Profile {
  id     Int  @id @default(autoincrement())
  user   User @relation(fields: [userId], references: [id])
  userId Int  @unique // relation scalar field (used in the `@relation` attribute above)
}
```

The `userId` relation scalar is a direct representation of the foreign key in the underlying database. This one-to-one relation expresses the following:

- "a user can have zero profiles or one profile" (because the profile field is optional on User).
- "a profile must always be connected to one user"

In the previous example, the `user` relation field of the `Profile` model references the `id` field of the `User` model. You can also reference a different field. In this case, you need to mark the field with the `@unique` attribute, to guarantee that there is only a single User connected to each `Profile`. In the following example, the `user` field references an `email` field in the `User` model, which is marked with the `@unique` attribute:

```prisma
model User {
  id      Int      @id @default(autoincrement())
  email   String   @unique // <-- add unique attribute
  profile Profile?
}

model Profile {
  id        Int    @id @default(autoincrement())
  user      User   @relation(fields: [userEmail], references: [email])
  userEmail String @unique // relation scalar field (used in the `@relation` attribute above)
}
```

**Required and optional 1-1 relation fields**

In a one-to-one relation, the side of the relation without a relation scalar (the field representing the foreign key in the database) must be optional:

```prisma
model User {
  id      Int      @id @default(autoincrement())
  profile Profile? // No relation scalar - must be optional
}
```

**you can choose if the side of the relation with a relation scalar should be optional or mandatory.** In the following example, `profile` and `profileId` are **mandatory**. This means that you **cannot create** a `User` without connecting or creating a `Profile`:

```prisma
model User {
  id        Int     @id @default(autoincrement())
  profile   Profile @relation(fields: [profileId], references: [id]) // references `id` of `Profile`
  profileId Int     @unique // relation scalar field (used in the `@relation` attribute above)

  // you can create a user without connecting or creating a Profile:
  profile   Profile? @relation(fields: [profileId], references: [id]) // references `id` of `Profile`
  profileId Int?
}

model Profile {
  id   Int   @id @default(autoincrement())
  user User?

  // you can create a user without connecting or creating a Profile:
  user User?
}
```
