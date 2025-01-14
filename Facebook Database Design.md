# Face Book DB-Design

Face Book Data Base Design

## Features
![image](https://user-images.githubusercontent.com/115500959/197022529-f7e7c010-8780-4cb9-8ad6-6606ec52b4aa.png)

## DB Design

<img width="1610" alt="image" src="https://github.com/user-attachments/assets/5c3f7249-9806-4530-9747-2f749740e673" />

```sql

-- Users Table: Stores information about each user
CREATE TABLE Users (
    user_id SERIAL PRIMARY KEY,                      -- Unique ID for each user (using SERIAL for auto-increment)
    username VARCHAR(255) UNIQUE NOT NULL,           -- Unique username
    email VARCHAR(255) UNIQUE NOT NULL,              -- User's email address
    password_hash VARCHAR(255) NOT NULL,             -- Hashed password
    first_name VARCHAR(100) NOT NULL,                -- First name
    last_name VARCHAR(100) NOT NULL,                 -- Last name
    birthdate DATE,                                  -- Birthdate of the user
    gender VARCHAR(10) CHECK (gender IN ('Male', 'Female', 'Other')),  -- Gender with CHECK constraint
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP -- Timestamp with timezone (PostgreSQL default)
);

-- Profiles Table: Stores additional profile details for the user
CREATE TABLE Profiles (
    profile_id SERIAL PRIMARY KEY,                   -- Unique profile ID
    user_id INT NOT NULL,                            -- Foreign Key to Users table
    bio TEXT,                                        -- Bio or description of the user
    profile_picture_url VARCHAR(255),                -- URL to the profile picture
    cover_picture_url VARCHAR(255),                  -- URL to the cover picture
    location VARCHAR(255),                           -- Location of the user
    website_url VARCHAR(255),                        -- User's website link
    updated_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the profile was updated
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Posts Table: Stores the posts made by users
CREATE TABLE Posts (
    post_id SERIAL PRIMARY KEY,                      -- Unique post ID
    user_id INT NOT NULL,                            -- Foreign Key to Users table
    content TEXT NOT NULL,                           -- Content of the post
    image_url VARCHAR(255),                          -- URL to any image or video attached to the post (optional)
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the post was created
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Friendships Table: Represents relationships between users
CREATE TABLE Friendships (
    friendship_id SERIAL PRIMARY KEY,                -- Unique friendship ID
    user_id_1 INT NOT NULL,                          -- Foreign Key to Users table (first user)
    user_id_2 INT NOT NULL,                          -- Foreign Key to Users table (second user)
    status VARCHAR(10) CHECK (status IN ('Pending', 'Accepted', 'Blocked')), -- Friendship status with CHECK constraint
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the friendship request was created
    FOREIGN KEY (user_id_1) REFERENCES Users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id_2) REFERENCES Users(user_id) ON DELETE CASCADE,
    CONSTRAINT unique_friendship UNIQUE (user_id_1, user_id_2)
);

-- Comments Table: Stores the comments on posts
CREATE TABLE Comments (
    comment_id SERIAL PRIMARY KEY,                   -- Unique comment ID
    post_id INT NOT NULL,                             -- Foreign Key to Posts table
    user_id INT NOT NULL,                             -- Foreign Key to Users table
    content TEXT NOT NULL,                            -- Content of the comment
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the comment was made
    FOREIGN KEY (post_id) REFERENCES Posts(post_id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Likes Table: Stores the likes given by users on posts or comments
CREATE TABLE Likes (
    like_id SERIAL PRIMARY KEY,                      -- Unique like ID
    user_id INT NOT NULL,                             -- Foreign Key to Users table
    post_id INT,                                     -- Foreign Key to Posts table (nullable)
    comment_id INT,                                  -- Foreign Key to Comments table (nullable)
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the like was made
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (post_id) REFERENCES Posts(post_id) ON DELETE CASCADE,
    FOREIGN KEY (comment_id) REFERENCES Comments(comment_id) ON DELETE CASCADE,
    CONSTRAINT unique_like UNIQUE (user_id, post_id, comment_id)
);

-- Groups Table: Stores groups that users can join
CREATE TABLE Groups (
    group_id SERIAL PRIMARY KEY,                      -- Unique group ID
    group_name VARCHAR(255) UNIQUE NOT NULL,           -- Name of the group
    description TEXT,                                 -- Description of the group
    created_by INT NOT NULL,                           -- Foreign Key to Users table (the user who created the group)
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the group was created
    FOREIGN KEY (created_by) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Group Memberships Table: Tracks users who belong to each group
CREATE TABLE GroupMemberships (
    group_membership_id SERIAL PRIMARY KEY,          -- Unique membership ID
    user_id INT NOT NULL,                             -- Foreign Key to Users table
    group_id INT NOT NULL,                            -- Foreign Key to Groups table
    joined_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the user joined the group
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (group_id) REFERENCES Groups(group_id) ON DELETE CASCADE,
    CONSTRAINT unique_membership UNIQUE (user_id, group_id)
);

-- Messages Table: Stores private messages sent between users
CREATE TABLE Messages (
    message_id SERIAL PRIMARY KEY,                    -- Unique message ID
    sender_id INT NOT NULL,                           -- Foreign Key to Users table (sender)
    receiver_id INT NOT NULL,                         -- Foreign Key to Users table (receiver)
    content TEXT NOT NULL,                            -- Content of the message
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the message was sent
    FOREIGN KEY (sender_id) REFERENCES Users(user_id) ON DELETE CASCADE,
    FOREIGN KEY (receiver_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Notifications Table: Stores notifications received by users
CREATE TABLE Notifications (
    notification_id SERIAL PRIMARY KEY,               -- Unique notification ID
    user_id INT NOT NULL,                             -- Foreign Key to Users table
    type VARCHAR(50) CHECK (type IN ('Friend Request', 'Like', 'Comment', 'Message', 'Event', 'Group Invite')),  -- Type of notification
    message TEXT,                                     -- Message associated with the notification
    read_status BOOLEAN DEFAULT FALSE,                -- Whether the notification has been read
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the notification was created
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Events Table: Stores events created by users
CREATE TABLE Events (
    event_id SERIAL PRIMARY KEY,                      -- Unique event ID
    user_id INT NOT NULL,                             -- Foreign Key to Users table (creator of the event)
    event_name VARCHAR(255) NOT NULL,                  -- Name of the event
    event_description TEXT,                           -- Description of the event
    event_date TIMESTAMPTZ NOT NULL,                   -- Date and time of the event
    location VARCHAR(255),                            -- Location of the event
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the event was created
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

-- Media Table: Stores photos and videos uploaded by users
CREATE TABLE Media (
    media_id SERIAL PRIMARY KEY,                      -- Unique media ID
    user_id INT NOT NULL,                             -- Foreign Key to Users table
    media_type VARCHAR(10) CHECK (media_type IN ('Photo', 'Video')) NOT NULL, -- Type of media (Photo or Video)
    media_url VARCHAR(255) NOT NULL,                   -- URL to the media content
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the media was uploaded
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE
);

```


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
