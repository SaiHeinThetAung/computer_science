Here are **all** the TypeScript examples you need to learn quickly!  

---

## **1. What is TypeScript?**  
TypeScript is a superset of JavaScript that **adds types** for better code safety and readability.  

### **JavaScript vs. TypeScript**
```javascript
// JavaScript (no type safety)
let name = "Alice";
name = 123; // No error until runtime

// TypeScript (strict type safety)
let name: string = "Alice";
name = 123; // Error: Type 'number' is not assignable to type 'string'
```

---

## **2. Installing TypeScript**
```bash
npm install -g typescript
tsc --version  # Check installation
```
To compile a TypeScript file (`file.ts`):  
```bash
tsc file.ts  # Generates file.js
```

---

## **3. Basic Types**  
```typescript
let isDone: boolean = false;
let age: number = 30;
let firstName: string = "John";
let list: number[] = [1, 2, 3];
let tuple: [string, number] = ["Alice", 25]; // Tuple
let unknownType: any = "Can be anything"; // Avoid using 'any'
```

---

## **4. Functions & Type Annotations**  
```typescript
function add(a: number, b: number): number {
  return a + b;
}
console.log(add(2, 3)); // Output: 5
```

**Optional Parameters & Default Values:**
```typescript
function greet(name: string, age?: number): string {
  return age ? `Hello ${name}, you are ${age} years old.` : `Hello ${name}`;
}
console.log(greet("Alice")); // Hello Alice
console.log(greet("Alice", 25)); // Hello Alice, you are 25 years old.
```

---

## **5. Interfaces (Defining Object Structure)**  
```typescript
interface Person {
  name: string;
  age: number;
}

const user: Person = { name: "Alice", age: 30 };
console.log(user.name); // Alice
```

---

## **6. Classes in TypeScript**  
```typescript
class Animal {
  name: string;

  constructor(name: string) {
    this.name = name;
  }

  speak(): void {
    console.log(`${this.name} makes a noise.`);
  }
}

const dog = new Animal("Dog");
dog.speak();  // Output: Dog makes a noise.
```

---

## **7. TypeScript with React (if needed for work)**  

Install TypeScript with React:  
```bash
npx create-react-app my-app --template typescript
```

**React Component with Props:**
```tsx
interface ButtonProps {
  label: string;
}

const Button: React.FC<ButtonProps> = ({ label }) => {
  return <button>{label}</button>;
};

export default Button;
```

---

## **8. Advanced: Generics (Reusable Code)**  
```typescript
function identity<T>(arg: T): T {
  return arg;
}
console.log(identity<string>("Hello")); // Output: Hello
console.log(identity<number>(123));     // Output: 123
```

**Generic Array Type:**
```typescript
function logArray<T>(arr: T[]): void {
  arr.forEach((item) => console.log(item));
}
logArray<string>(["Apple", "Banana"]);
```

---

## **9. TypeScript Config (`tsconfig.json`)**  
```bash
tsc --init
```
Common settings:  
```json
{
  "compilerOptions": {
    "target": "ES6",
    "strict": true,
    "module": "CommonJS"
  }
}
```

---

## **10. TypeScript with Node.js (If using Express)**  
### **Install dependencies:**
```bash
npm install express @types/express
```

### **Basic Express Server with TypeScript**
```typescript
import express from "express";

const app = express();

app.get("/", (req, res) => res.send("Hello TypeScript!"));

app.listen(3000, () => console.log("Server running on port 3000"));
```

---

## **11. TypeScript with APIs (Fetch Example)**  
```typescript
async function fetchData(): Promise<void> {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
  const data: { userId: number; id: number; title: string; body: string } = await response.json();
  console.log(data);
}
fetchData();
```

---

## **12. Enums (Useful for Constants)**  
```typescript
enum Color {
  Red,
  Green,
  Blue,
}

let selectedColor: Color = Color.Green;
console.log(selectedColor); // Output: 1
```

---

## **13. Union & Intersection Types**  
```typescript
let id: string | number; // Union Type
id = "123";
id = 123;

interface A { a: string; }
interface B { b: number; }
type AB = A & B; // Intersection Type
const obj: AB = { a: "hello", b: 42 };
```

---

## **14. Utility Types**  
```typescript
type Person = { name: string; age: number; city?: string };

const user1: Readonly<Person> = { name: "Alice", age: 30 };
// user1.age = 31; // Error: Cannot assign to 'age' because it is a read-only property.

type RequiredPerson = Required<Person>; // Makes all properties required
type PartialPerson = Partial<Person>;   // Makes all properties optional
```

---

## **15. Type Guards**  
```typescript
function printValue(value: string | number) {
  if (typeof value === "string") {
    console.log(`String: ${value.toUpperCase()}`);
  } else {
    console.log(`Number: ${value.toFixed(2)}`);
  }
}
printValue("hello");
printValue(10.567);
```

---

## **16. TypeScript in a Full MERN Stack Project**
If using **TypeScript with MongoDB and Express**, install:  
```bash
npm install mongoose @types/mongoose
```

### **Mongoose Model in TypeScript**
```typescript
import mongoose, { Schema, Document } from "mongoose";

interface IUser extends Document {
  name: string;
  email: string;
  password: string;
}

const UserSchema: Schema = new Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true }
});

export default mongoose.model<IUser>("User", UserSchema);
```

---

## **17. TypeScript with AWS (S3 Upload Example)**  
```typescript
import AWS from "aws-sdk";

const s3 = new AWS.S3({
  accessKeyId: process.env.AWS_ACCESS_KEY_ID,
  secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
});

const params = {
  Bucket: "my-bucket",
  Key: "file.txt",
  Body: "Hello World"
};

s3.upload(params, (err, data) => {
  if (err) console.error(err);
  else console.log(`File uploaded successfully: ${data.Location}`);
});
```

---

### **Final Summary**
✔ **Core Concepts:** Types, Interfaces, Classes, Functions  
✔ **Advanced Features:** Generics, Enums, Utility Types  
✔ **Framework Integrations:** React, Node.js, Express, MongoDB, AWS  

---

Now, you’re ready to use TypeScript at work! 🚀 Want help with a specific project setup?
