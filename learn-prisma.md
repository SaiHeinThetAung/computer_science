Prisma is a modern database toolkit that makes working with databases in Node.js and TypeScript applications easier. Here's a step-by-step guide to learning Prisma from basic to advanced:

---

## **Step 1: Install Prisma**
First, make sure you have **Node.js** and **npm/yarn** installed.

1. Create a new project:
   ```sh
   mkdir my-prisma-app && cd my-prisma-app
   npm init -y
   ```

2. Install Prisma and the Prisma CLI:
   ```sh
   npm install prisma --save-dev
   ```

3. Initialize Prisma:
   ```sh
   npx prisma init
   ```
   This creates a `prisma` folder with a `schema.prisma` file and a `.env` file for database configuration.

---

## **Step 2: Set Up the Database**
Edit `.env` to configure your database. For example, if using **PostgreSQL**, update:
```env
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
```

For **MySQL**, MongoDB, or SQLite, change the connection string accordingly.

---

## **Step 3: Define the Data Model**
Edit `prisma/schema.prisma` to define models. Example:
```prisma
model User {
  id    String @id @default(uuid())
  name  String
  email String @unique
  posts Post[]
}

model Post {
  id        String @id @default(uuid())
  title     String
  content   String?
  published Boolean @default(false)
  author    User   @relation(fields: [authorId], references: [id])
  authorId  String
}
```

---

## **Step 4: Migrate the Database**
Run the migration to sync the database:
```sh
npx prisma migrate dev --name init
```
This will:
- Create the database tables
- Track changes to the schema

---

## **Step 5: Use Prisma Client**
Generate the Prisma client:
```sh
npx prisma generate
```

Then, use Prisma in a **Node.js** script (`index.js`):
```js
const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

async function main() {
  const user = await prisma.user.create({
    data: {
      name: 'John Doe',
      email: 'john@example.com',
      posts: {
        create: {
          title: 'My First Post',
          content: 'Hello, world!',
        },
      },
    },
    include: { posts: true },
  });

  console.log(user);
}

main()
  .catch((e) => console.error(e))
  .finally(() => prisma.$disconnect());
```
Run:
```sh
node index.js
```

---

## **Step 6: Query Data**
Fetch users with their posts:
```js
const users = await prisma.user.findMany({
  include: { posts: true },
});
console.log(users);
```

---

## **Step 7: Update & Delete Data**
Update a user:
```js
await prisma.user.update({
  where: { email: 'john@example.com' },
  data: { name: 'John Updated' },
});
```

Delete a user:
```js
await prisma.user.delete({
  where: { email: 'john@example.com' },
});
```

---

## **Step 8: Advanced Prisma Features**
- **Middleware**: Run logic before/after queries.
- **Soft Deletes**: Add `deletedAt` field instead of hard deletion.
- **Raw Queries**: Use SQL if needed:
  ```js
  const result = await prisma.$queryRaw`SELECT * FROM "User";`;
  console.log(result);
  ```
- **Prisma Studio** (GUI to manage the database):
  ```sh
  npx prisma studio
  ```

---

## **Step 9: Deploy with Prisma**
For production, use:
```sh
npx prisma migrate deploy
```
Then, deploy the application on **AWS, Vercel, or DigitalOcean**.

---

## **Step 10: Learn More**
- Prisma Docs: [https://www.prisma.io/docs](https://www.prisma.io/docs)
- GitHub: [https://github.com/prisma/prisma](https://github.com/prisma/prisma)
