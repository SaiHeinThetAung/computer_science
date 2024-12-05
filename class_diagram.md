Below is a step-by-step guide for creating the full class diagram for the **Property Viewing System (PVS)** using **draw.io**, including all attributes, operations, relationships, and multiplicities:

---

### **Step-by-Step Visual Guide**

#### **1. Create the Classes**

1. **Employee (Abstract Class)**  
   - **Attributes:**
     - `name: String`
     - `employeeNumber: String`
   - **Operations:**
     - `addEmployee(): void`
     - `updateEmployee(): void`
     - `deleteEmployee(): void`
     - `getEmployeeDetails(): void`
   - **Note:** Mark the class as *Abstract* by setting the title in italics or writing "(Abstract)" next to the class name.

2. **Manager (Subclass of Employee)**  
   - Inherit all attributes and operations from `Employee`.

3. **Representative (Subclass of Employee)**  
   - **Attributes:**
     - `isPartTime: Boolean`
     - `grade: String`

4. **Branch**  
   - **Attributes:**
     - `branchName: String`
     - `location: String`
     - `manager: Manager`
     - `representatives: List<Representative>`
   - **Operations:**
     - `addBranch(): void`
     - `updateBranch(): void`
     - `deleteBranch(): void`
     - `getBranchDetails(): List<Branch>`

5. **Client**  
   - **Attributes:**
     - `clientID: String`
     - `name: String`
     - `address: String`
     - `isBuyer: Boolean`
     - `isSeller: Boolean`
     - `interestedProperties: List<Property>`
   - **Operations:**
     - `registerClient(): void`
     - `updateClient(): void`
     - `deleteClient(): void`
     - `browseProperties(): List<Property>`
     - `searchProperties(criteria: Map): List<Property>`

6. **Property**  
   - **Attributes:**
     - `propertyID: String`
     - `address: String`
     - `description: String`
     - `price: Float`
     - `type: String`
     - `pictures: List<Image>`
     - `seller: Client`
     - `branch: Branch`
     - `status: String`
     - `numBedrooms: Integer`
     - `dateFirstOffered: Date`
   - **Operations:**
     - `addProperty(): void`
     - `updateProperty(): void`
     - `deleteProperty(): void`
     - `getPropertyDetails(): Property`

7. **Offer**  
   - **Attributes:**
     - `offerID: String`
     - `offerPrice: Float`
     - `date: Date`
     - `status: String`
     - `client: Client`
   - **Operations:**
     - `makeOffer(): void`
     - `updateOffer(): void`
     - `deleteOffer(): void`
     - `getOfferDetails(): Offer`

8. **Appointment**  
   - **Attributes:**
     - `appointmentID: String`
     - `date: Date`
     - `time: Time`
     - `property: Property`
     - `representative: Representative`
     - `buyer: Client`
   - **Operations:**
     - `scheduleAppointment(): void`
     - `updateAppointment(): void`
     - `deleteAppointment(): void`

---

#### **2. Add Relationships and Multiplicities**

1. **Employee ↔ Branch**  
   - Use an association line from `Branch` to `Employee`.  
   - **Multiplicity:**  
     - `1` (Manager oversees one branch).  
     - `1..*` (Representatives work in one or more branches).

2. **Branch ↔ Property**  
   - Use an association line from `Branch` to `Property`.  
   - **Multiplicity:**  
     - `1..*` (A branch can manage multiple properties).  
     - `1` (Each property belongs to one branch).  

3. **Property ↔ Seller (Client)**  
   - Use an association line from `Property` to `Client`.  
   - **Multiplicity:**  
     - `1..*` (A seller can sell multiple properties).  
     - `1` (Each property has one seller).  

4. **Property ↔ Appointment**  
   - Use an association line from `Property` to `Appointment`.  
   - **Multiplicity:**  
     - `1..*` (Each property can have multiple appointments).  
     - `1` (Each appointment is for one property).  

5. **Appointment ↔ Representative**  
   - Use an association line from `Appointment` to `Representative`.  
   - **Multiplicity:**  
     - `1` (Each appointment is facilitated by one representative).  
     - `0..*` (A representative can have multiple appointments).  

6. **Offer ↔ Property**  
   - Use an association line from `Offer` to `Property`.  
   - **Multiplicity:**  
     - `1..*` (A property can have multiple offers).  
     - `1` (Each offer applies to one property).  

7. **Offer ↔ Client**  
   - Use an association line from `Offer` to `Client`.  
   - **Multiplicity:**  
     - `1..*` (A client can make multiple offers).  
     - `1` (Each offer is made by one client).

---

#### **3. Adding Visual Elements in Draw.io**

1. **Create Classes:**
   - Use the **rectangle shape** for each class.  
   - Divide it into three sections for **class name**, **attributes**, and **operations**.  

2. **Relationships and Multiplicities:**
   - Use **straight lines** for associations.  
   - Add **multiplicity indicators** at each end (e.g., `1`, `1..*`).  

3. **Inheritance (Abstract Classes):**
   - Use a **solid line with a triangle head** to represent inheritance.  
   - E.g., `Manager` and `Representative` inherit from `Employee`.

4. **Labels and Additional Notes:**
   - Label each association meaningfully (e.g., "Manages", "Schedules").  
   - Use **text boxes** to add notes or clarify relationships.

---

#### **Diagram Layout Recommendation**

- **Top Section:** Abstract class `Employee` with subclasses `Manager` and `Representative`.  
- **Middle Section:** `Branch`, with connections to `Employee` and `Property`.  
- **Bottom Section:**  
  - `Client` on the left, linked to `Property`, `Offer`, and `Appointment`.  
  - `Property` at the center, with links to `Branch`, `Client`, and `Appointment`.

---

Let me know if you'd like me to regenerate this layout or an image with a sample!
