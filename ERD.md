### **Step-by-Step Guide to Create an ER Diagram in draw.io**

Here is a detailed step-by-step guide for creating the ERD for Bloomfield Garden Centre using draw.io:

---

### **Step 1: Open draw.io**
1. Go to [draw.io](https://app.diagrams.net/).
2. Click **"Create New Diagram"**.
3. Choose a blank diagram or the "Entity Relationship" template.
4. Save the file to your local drive or Google Drive.

---

### **Step 2: Set Up the Entities**

1. **Drag and Drop Rectangle Shapes**:
   - Find the "Entity" shape in the toolbar (left side).
   - Drag it into the workspace.

2. **Add Text for Entity Names**:
   - Double-click the rectangle.
   - Type the entity name (e.g., "Inventory", "Employees", etc.).

3. **Add Attributes**:
   - Inside each rectangle, list the attributes of the entity.
   - Format as follows:
     - **Primary Key**: Underline it (e.g., `ProductID`).
     - **Foreign Key**: Add "(FK)" after the attribute name.

   **Entities and Attributes**:
   - **Inventory**:
     ```
     ProductID (PK)
     Name
     Type
     Price
     Description
     ```
   - **Employees**:
     ```
     EmployeeID (PK)
     Name
     ContactDetails
     Role
     WorkHours
     TrainingRecords
     SpecialQualifications
     ```
   - **SalesTransactions**:
     ```
     TransactionID (PK)
     Date
     EmployeeID (FK)
     TotalPrice
     ```
   - **Customers**:
     ```
     CustomerID (PK)
     Name
     ContactDetails
     ```
   - **WorkshopsAndEvents**:
     ```
     EventID (PK)
     Title
     Description
     Date
     Time
     ExpectedAttendance
     Equipment
     StaffAssignments
     ```
   - **TransactionDetails** (Associative Table):
     ```
     TransactionID (FK)
     ProductID (FK)
     Quantity
     ```

---

### **Step 3: Define Relationships**

1. **Use the Arrow Tool**:
   - Connect entities with lines or arrows to define relationships.
   - Label the relationship (e.g., "Processes", "Purchases", "Attends").

2. **Add Cardinalities**:
   - Use "1" for one side and "N" (or "M") for many sides.
   - **Examples**:
     - `SalesTransactions` ↔ `Employees`: **1:N** (one employee processes many transactions).
     - `SalesTransactions` ↔ `TransactionDetails`: **1:N** (one transaction can include many items).
     - `TransactionDetails` ↔ `Inventory`: **N:1** (many items can relate to one product).

3. **Many-to-Many Relationships**:
   - Represent as associative tables.
   - Example: Between `SalesTransactions` and `Inventory`, use `TransactionDetails` as an intermediate entity.

---

### **Step 4: Organize the Layout**

1. **Align Entities Neatly**:
   - Place related entities close together.
   - Suggested Layout:
     - Top row: `Inventory`, `Employees`, `Customers`.
     - Bottom row: `SalesTransactions`, `TransactionDetails`, `WorkshopsAndEvents`.

2. **Style the Diagram**:
   - Use colors for entities (e.g., blue) and relationships (e.g., green).
   - Add labels to relationships for clarity.

---

### **Step 5: Ensure Third Normal Form (3NF)**

1. **First Normal Form (1NF)**:
   - Each attribute contains atomic values (no repeating groups).
   - Example: Move multiple training records from `Employees` to a separate table `EmployeeTraining`.

2. **Second Normal Form (2NF)**:
   - Ensure all attributes depend on the whole primary key.
   - Example: In `TransactionDetails`, both `TransactionID` and `ProductID` are needed to determine `Quantity`.

3. **Third Normal Form (3NF)**:
   - Remove transitive dependencies.
   - Example: Separate customer details from `SalesTransactions` into a `Customers` table.

---

### **Step 6: Add Security Measures**
1. **Role-Based Access Control**:
   - Add annotations to specify roles (e.g., "Admin", "Employee").
   - Highlight entities requiring restricted access (e.g., `Employees`, `SalesTransactions`).

---

### **Step 7: Save and Export**
1. **Save Your Work**:
   - Click **File > Save As** and choose a location.
2. **Export Diagram**:
   - Go to **File > Export As** and choose PNG, PDF, or other formats.

---

### **Step 8: Write a Justification**

In your report, include:
1. **Entity Justification**:
   - Explain why each entity and attribute is necessary.
2. **Normalization Compliance**:
   - Detail how your design meets 3NF.
3. **Security Features**:
   - Describe role-based access control measures.

---

If you follow these steps and ensure clarity and accuracy, you'll create a professional-grade ER diagram that meets the assignment's requirements and maximizes your marks. Let me know if you need further assistance!
