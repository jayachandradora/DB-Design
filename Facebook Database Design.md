# Face Book DB-Design

Face Book Data Base Design

## Features
![image](https://user-images.githubusercontent.com/115500959/197022529-f7e7c010-8780-4cb9-8ad6-6606ec52b4aa.png)

## DB Design

![image](https://user-images.githubusercontent.com/115500959/197022262-fdac9402-efc3-46ed-837e-a192ce27223a.png)

## SQL Query: Find Details Of your Friend
![image](https://user-images.githubusercontent.com/115500959/197023439-e9ef848b-5c66-4f0d-bc2a-d80702f094aa.png)

## SQL Query: All Comments on Posts
![image](https://user-images.githubusercontent.com/115500959/197025112-80f3616a-9a54-4c24-9b7e-7326d93c9da4.png)

## SQL Query: All your Text Posts
![image](https://user-images.githubusercontent.com/115500959/197025373-28bab2c4-e6b1-41f1-92db-c11f24fab4c7.png)


## Follower relationship on a platform like Facebook

To model a follower relationship on a platform like Facebook, we can design a database schema and corresponding Hibernate entities to represent the relationship between users. In this case, a user can follow many other users, and each user can be followed by many users. This creates a many-to-many relationship.

### Database Schema

We will need three tables:

1. **Users Table**: To store user information.
2. **Followers Table**: To represent the follower relationships.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);

CREATE TABLE followers (
    follower_id INT REFERENCES users(id),  -- User who follows
    followed_id INT REFERENCES users(id),   -- User being followed
    PRIMARY KEY (follower_id, followed_id)
);
```

### Hibernate Entities

#### User Entity

```java
import javax.persistence.*;
import java.util.HashSet;
import java.util.Set;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String email;

    // Users that this user follows
    @ManyToMany
    @JoinTable(
        name = "followers",
        joinColumns = @JoinColumn(name = "follower_id"),
        inverseJoinColumns = @JoinColumn(name = "followed_id")
    )
    private Set<User> following = new HashSet<>();

    // Users that follow this user
    @ManyToMany(mappedBy = "following")
    private Set<User> followers = new HashSet<>();

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    public Set<User> getFollowing() { return following; }
    public void setFollowing(Set<User> following) { this.following = following; }

    public Set<User> getFollowers() { return followers; }
    public void setFollowers(Set<User> followers) { this.followers = followers; }
}
```

### Example Usage

Here’s how you might use these entities to create a follower relationship in a Hibernate session:

```java
import org.hibernate.Session;
import org.hibernate.Transaction;

public class Main {
    public static void main(String[] args) {
        Session session = HibernateUtil.getSessionFactory().openSession();
        Transaction transaction = session.beginTransaction();

        // Create users
        User user1 = new User();
        user1.setUsername("john_doe");
        user1.setEmail("john@example.com");

        User user2 = new User();
        user2.setUsername("jane_smith");
        user2.setEmail("jane@example.com");

        // Establish follower relationship
        user1.getFollowing().add(user2); // John follows Jane
        user2.getFollowers().add(user1);  // Jane is followed by John

        // Save users (cascade will take care of the relationship)
        session.save(user1);
        session.save(user2);

        transaction.commit();
        session.close();
    }
}
```

### Summary

1. **Database Design**:
   - The `users` table holds user information.
   - The `followers` table establishes a many-to-many relationship between users, with a composite primary key.

2. **Hibernate Entities**:
   - The `User` entity has a `Set<User>` for both `following` and `followers`, using `@ManyToMany` annotations to represent the relationships.
   - The `@JoinTable` annotation defines the linking table for the many-to-many relationship.

This setup allows you to effectively manage follower relationships between users in a social media-like application.
