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
