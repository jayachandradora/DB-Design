# Many-To-Many Relationships

Sure! Here’s an example of a many-to-one relationship in a database using Hibernate in Java.

### Database Schema

In this example, let’s consider two entities: `Order` and `Customer`. Many orders can be associated with one customer.

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_number VARCHAR(50) NOT NULL,
    customer_id INT REFERENCES customers(id)
);
```

### Hibernate Entities

#### Customer Entity

```java
import javax.persistence.*;
import java.util.List;

@Entity
@Table(name = "customers")
public class Customer {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Order> orders;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }

    public List<Order> getOrders() { return orders; }
    public void setOrders(List<Order> orders) { this.orders = orders; }
}
```

#### Order Entity

```java
import javax.persistence.*;

@Entity
@Table(name = "orders")
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String orderNumber;

    @ManyToOne
    @JoinColumn(name = "customer_id", nullable = false)
    private Customer customer;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getOrderNumber() { return orderNumber; }
    public void setOrderNumber(String orderNumber) { this.orderNumber = orderNumber; }

    public Customer getCustomer() { return customer; }
    public void setCustomer(Customer customer) { this.customer = customer; }
}
```

### Example Usage

Here’s how you might use these entities in a Hibernate session:

```java
import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {
    public static void main(String[] args) {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = session.beginTransaction();

        // Create a Customer
        Customer customer = new Customer();
        customer.setName("Alice Smith");

        // Create Orders
        Order order1 = new Order();
        order1.setOrderNumber("ORD123");
        order1.setCustomer(customer);

        Order order2 = new Order();
        order2.setOrderNumber("ORD124");
        order2.setCustomer(customer);

        // Associate orders with the customer
        customer.setOrders(List.of(order1, order2));

        // Save the customer (this will also save the orders due to CascadeType.ALL)
        session.save(customer);

        transaction.commit();
        session.close();
    }
}
```

### Summary

- **Entities**: The `Customer` entity has a many-to-one relationship with the `Order` entity.
- **Annotations**: The `@OneToMany` and `@ManyToOne` annotations define the relationship in Hibernate.
- **Cascade**: Using `CascadeType.ALL` on the `Customer` entity allows you to save associated orders automatically.

This setup allows you to manage the customer-order relationship effectively with Hibernate, where multiple orders can be linked to a single customer.

The technical differences between one-to-many and many-to-one relationships in database design and Hibernate can be understood through several key aspects:

### 1. **Data Structure in the Database**

- **One-to-Many**:
  - In a one-to-many relationship, a single record in the "one" table can relate to multiple records in the "many" table. 
  - This is typically represented by a foreign key in the "many" table pointing back to the "one" table.

  **Example**: 
  - Table `User` (one):
    ```sql
    CREATE TABLE users (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL
    );
    ```

  - Table `Post` (many):
    ```sql
    CREATE TABLE posts (
        id SERIAL PRIMARY KEY,
        title VARCHAR(100) NOT NULL,
        user_id INT REFERENCES users(id) -- Foreign key to User
    );
    ```

- **Many-to-One**:
  - In a many-to-one relationship, multiple records in the "many" table can relate to a single record in the "one" table. 
  - The structure is the same, but the emphasis is on the "many" side having a foreign key that references the "one" side.

  **Example**: 
  - Table `Order` (many):
    ```sql
    CREATE TABLE orders (
        id SERIAL PRIMARY KEY,
        order_number VARCHAR(50) NOT NULL,
        customer_id INT REFERENCES customers(id) -- Foreign key to Customer
    );
    ```

  - Table `Customer` (one):
    ```sql
    CREATE TABLE customers (
        id SERIAL PRIMARY KEY,
        name VARCHAR(100) NOT NULL
    );
    ```

### 2. **Entity Mapping in Hibernate**

- **One-to-Many**:
  - The entity representing the "one" side (e.g., `User`) has a `@OneToMany` annotation, and the entity representing the "many" side (e.g., `Post`) has a `@ManyToOne` annotation.

  ```java
  @Entity
  public class User {
      @Id
      private Long id;

      @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
      private List<Post> posts; // Relationship defined here
  }
  
  @Entity
  public class Post {
      @Id
      private Long id;

      @ManyToOne
      @JoinColumn(name = "user_id", nullable = false)
      private User user; // Relationship defined here
  }
  ```

- **Many-to-One**:
  - The mapping is conceptually the same, but the focus is on the many side having a reference to the one side.

  ```java
  @Entity
  public class Order {
      @Id
      private Long id;

      @ManyToOne
      @JoinColumn(name = "customer_id", nullable = false)
      private Customer customer; // Relationship defined here
  }

  @Entity
  public class Customer {
      @Id
      private Long id;

      @OneToMany(mappedBy = "customer", cascade = CascadeType.ALL)
      private List<Order> orders; // Relationship defined here
  }
  ```

### 3. **Data Retrieval**

- **One-to-Many**:
  - When retrieving a user, you can fetch all associated posts using a single query if you use an appropriate fetch strategy (e.g., `JOIN FETCH`).

  ```java
  User user = session.createQuery("FROM User u JOIN FETCH u.posts WHERE u.id = :id", User.class)
                     .setParameter("id", userId)
                     .getSingleResult();
  ```

- **Many-to-One**:
  - When retrieving an order, the associated customer can also be fetched. However, this typically results in querying the `Customer` table for the corresponding record.

  ```java
  Order order = session.get(Order.class, orderId);
  Customer customer = order.getCustomer(); // Access the associated customer
  ```

### 4. **Cardinality and Relationships**

- **One-to-Many**:
  - Represents a single parent to multiple children. This is useful for aggregating related data (e.g., all posts by a user).

- **Many-to-One**:
  - Represents multiple children pointing back to a single parent. This is often used when the child entity requires the context of its parent (e.g., all orders made by a customer).

### Conclusion

In technical terms, the primary differences between one-to-many and many-to-one relationships are about the perspective and context of the relationship being modeled, the way foreign keys are structured, the annotations used in Hibernate, and how data retrieval is managed. Both relationships are complementary and often used together to model complex data interactions effectively.

Many-To-Many Relationships

![image](https://user-images.githubusercontent.com/115500959/196847687-e4483f9f-d007-4317-bf57-36b54d82fde9.png)

![image](https://user-images.githubusercontent.com/115500959/196847780-fafa5176-f798-4a36-a24c-ee400962600d.png)

![image](https://user-images.githubusercontent.com/115500959/196847882-217947c4-2ce5-417d-b93a-38dd3e8c5407.png)

![image](https://user-images.githubusercontent.com/115500959/196848096-8936ee33-1b61-428c-8b27-099b9b8fd738.png)

![image](https://user-images.githubusercontent.com/115500959/196848225-70d27fb3-966b-4689-bb56-c4bb76a01876.png)

![image](https://user-images.githubusercontent.com/115500959/196848332-1a0c97b8-4571-496a-978d-60160c214687.png)

![image](https://user-images.githubusercontent.com/115500959/196848439-971dcfc9-d219-41e6-8ec7-78083941202e.png)

![image](https://user-images.githubusercontent.com/115500959/196848501-6a4d1003-2083-401a-afec-1ed60cf63659.png)

### Another Many to Many Example.

![image](https://user-images.githubusercontent.com/115500959/196874577-6155bd40-c22d-4172-8694-5d5c60f99874.png)
