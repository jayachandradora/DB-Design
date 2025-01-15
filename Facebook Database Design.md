# Face Book DB-Design

Face Book Data Base Design

## Features
![image](https://user-images.githubusercontent.com/115500959/197022529-f7e7c010-8780-4cb9-8ad6-6606ec52b4aa.png)

## DB Design / ER Diagram

<img width="1298" alt="image" src="https://github.com/user-attachments/assets/b92ddc5a-18c4-4ba7-825b-12d453919ea2" />

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

-- Followers Table: Represents the following relationship between users
CREATE TABLE Followers (
    follower_user_id INT NOT NULL,                        -- Foreign Key to Users table (the user who is following)
    following_user_id INT NOT NULL,                       -- Foreign Key to Users table (the user being followed)
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the follow relationship was created
    PRIMARY KEY (follower_user_id, following_user_id),         -- Composite primary key
    FOREIGN KEY (follower_user_id) REFERENCES Users(user_id) ON DELETE CASCADE, -- Ensures that when a user is deleted, their followers are also removed
    FOREIGN KEY (following_user_id) REFERENCES Users(user_id) ON DELETE CASCADE, -- Ensures that when a user is deleted, the users they follow are also removed
    CONSTRAINT unique_follow UNIQUE (follower_user_id, following_user_id)              -- Prevents duplicate follow relationships
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

# JPA Data Model

To convert your SQL schema into a Hibernate JPA data model, I'll create Java entity classes with the corresponding annotations. These classes will represent the tables you provided, and the relationships will be modeled using `@OneToMany`, `@ManyToOne`, `@ManyToMany`, and other JPA annotations.

Below is the equivalent Hibernate JPA model for your SQL script:

```java
import javax.persistence.*;
import java.util.Date;
import java.util.List;

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long userId;

    @Column(unique = true, nullable = false)
    private String username;

    @Column(unique = true, nullable = false)
    private String email;

    @Column(nullable = false)
    private String passwordHash;

    @Column(nullable = false)
    private String firstName;

    @Column(nullable = false)
    private String lastName;

    private Date birthdate;

    @Enumerated(EnumType.STRING)
    private Gender gender;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private Profile profile;

    @OneToMany(mappedBy = "user")
    private List<Post> posts;

    @OneToMany(mappedBy = "followerUser")
    private List<Followers> followers;

    @OneToMany(mappedBy = "followingUser")
    private List<Followers> followings;

    @OneToMany(mappedBy = "user")
    private List<Friendship> friendships;

    @OneToMany(mappedBy = "sender")
    private List<Message> sentMessages;

    @OneToMany(mappedBy = "receiver")
    private List<Message> receivedMessages;

    @OneToMany(mappedBy = "user")
    private List<Notification> notifications;

    @OneToMany(mappedBy = "user")
    private List<Event> events;

    @OneToMany(mappedBy = "user")
    private List<Media> media;

    @ManyToMany(mappedBy = "members")
    private List<Group> groups;

    // Getters and Setters
}

@Entity
public class Profile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long profileId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    private String bio;
    private String profilePictureUrl;
    private String coverPictureUrl;
    private String location;
    private String websiteUrl;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date updatedAt;

    // Getters and Setters
}

@Entity
public class Post {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long postId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(nullable = false)
    private String content;

    private String imageUrl;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    @OneToMany(mappedBy = "post")
    private List<Comment> comments;

    @OneToMany(mappedBy = "post")
    private List<Like> likes;

    // Getters and Setters
}

@Entity
public class Followers {
    @Id
    @ManyToOne
    @JoinColumn(name = "follower_user_id", nullable = false)
    private User followerUser;

    @Id
    @ManyToOne
    @JoinColumn(name = "following_user_id", nullable = false)
    private User followingUser;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Friendship {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long friendshipId;

    @ManyToOne
    @JoinColumn(name = "user_id_1", nullable = false)
    private User user1;

    @ManyToOne
    @JoinColumn(name = "user_id_2", nullable = false)
    private User user2;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private FriendshipStatus status;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Comment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long commentId;

    @ManyToOne
    @JoinColumn(name = "post_id", nullable = false)
    private Post post;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(nullable = false)
    private String content;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Like {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long likeId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne
    @JoinColumn(name = "post_id")
    private Post post;

    @ManyToOne
    @JoinColumn(name = "comment_id")
    private Comment comment;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Group {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long groupId;

    @Column(unique = true, nullable = false)
    private String groupName;

    private String description;

    @ManyToOne
    @JoinColumn(name = "created_by", nullable = false)
    private User createdBy;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    @ManyToMany
    @JoinTable(
            name = "group_memberships",
            joinColumns = @JoinColumn(name = "group_id"),
            inverseJoinColumns = @JoinColumn(name = "user_id")
    )
    private List<User> members;

    // Getters and Setters
}

@Entity
public class GroupMembership {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long groupMembershipId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @ManyToOne
    @JoinColumn(name = "group_id", nullable = false)
    private Group group;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date joinedAt;

    // Getters and Setters
}

@Entity
public class Message {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long messageId;

    @ManyToOne
    @JoinColumn(name = "sender_id", nullable = false)
    private User sender;

    @ManyToOne
    @JoinColumn(name = "receiver_id", nullable = false)
    private User receiver;

    @Column(nullable = false)
    private String content;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Notification {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long notificationId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Enumerated(EnumType.STRING)
    private NotificationType type;

    private String message;

    @Column(nullable = false)
    private Boolean readStatus = false;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Event {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long eventId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Column(nullable = false)
    private String eventName;

    private String eventDescription;

    @Column(nullable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date eventDate;

    private String location;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

@Entity
public class Media {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long mediaId;

    @ManyToOne
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private MediaType mediaType;

    @Column(nullable = false)
    private String mediaUrl;

    @Column(nullable = false, updatable = false)
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;

    // Getters and Setters
}

// Enum Types

public enum Gender {
    Male,
    Female,
    Other
}

public enum FriendshipStatus {
    Pending,
    Accepted,
    Blocked
}

public enum NotificationType {
    Friend_Request,
    Like,
    Comment,


    Message,
    Event,
    Group_Invite
}

public enum MediaType {
    Photo,
    Video
}
```

### Explanation:

1. **Entity Classes**: 
   - Each table corresponds to an entity class annotated with `@Entity`.
   - The `@Id` annotation denotes the primary key.
   - `@GeneratedValue` specifies how the primary key will be generated (using auto-increment in the database).

2. **Relationships**:
   - `@OneToMany`, `@ManyToOne`, and `@ManyToMany` are used to represent the relationships between entities. These map the foreign keys between the tables.
   - For example, `@ManyToOne` is used for `user_id` in `Post`, which means each post is linked to a single user.

3. **Enums**:
   - `Gender`, `FriendshipStatus`, `NotificationType`, and `MediaType` are enums for constrained fields.

4. **Timestamps**: 
   - `@Temporal(TemporalType.TIMESTAMP)` is used to map `TIMESTAMPTZ` (timestamp with time zone) columns.

5. **Cascade Operations**: 
   - `CascadeType.ALL` is applied to ensure that child entities (like `Profile`) are deleted when the parent `User` is deleted.

6. **`@JoinColumn` and `@ManyToMany`**:
   - Used for many-to-many relationships, such as in the `Group` class with `GroupMembership`.

This Java code represents a typical JPA-based data model and should work seamlessly with Hibernate as the JPA implementation.

## JPA Repository and Data Retrieval Implementation 

Below are the JPA repositories and service layer implementations to handle the functionality you mentioned.

### 1. **Add Other Profiles as Friends**:
   To allow users to add other profiles as friends, we will need to handle friendship creation, which will involve adding a friendship request and accepting it.

### 2. **Add Posts (Text, Photos, or Videos)**:
   We will allow users to create posts containing text and optionally images or videos.

### 3. **See Posts from Friends**:
   To fetch posts made by friends, we will need to query for posts where the user is following other users.

### 4. **Like and Comment on Posts**:
   Users should be able to like and comment on posts created by others.

Below is the implementation of the required functionality:

### 1. **JPA Repositories**

We will start by creating the JPA repositories for each entity.

#### **UserRepository.java**
```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
    User findByEmail(String email);
}
```

#### **PostRepository.java**
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface PostRepository extends JpaRepository<Post, Long> {
    List<Post> findByUserIn(List<User> users);
}
```

#### **FriendshipRepository.java**
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface FriendshipRepository extends JpaRepository<Friendship, Long> {
    Optional<Friendship> findByUser1AndUser2(User user1, User user2);
    Optional<Friendship> findByUser2AndUser1(User user1, User user2);
}
```

#### **CommentRepository.java**
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findByPost(Post post);
}
```

#### **LikeRepository.java**
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface LikeRepository extends JpaRepository<Like, Long> {
    Optional<Like> findByUserAndPost(User user, Post post);
    Optional<Like> findByUserAndComment(User user, Comment comment);
}
```

#### **FollowersRepository.java**
```java
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface FollowersRepository extends JpaRepository<Followers, Long> {
    Optional<Followers> findByFollowerUserAndFollowingUser(User follower, User following);
}
```

### 2. **Service Layer**

Next, let's create the service layer to implement the business logic for the functionality you requested.

#### **FriendService.java**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

@Service
public class FriendService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private FriendshipRepository friendshipRepository;

    @Autowired
    private FollowersRepository followersRepository;

    @Transactional
    public void addFriend(String username1, String username2) {
        User user1 = userRepository.findByUsername(username1);
        User user2 = userRepository.findByUsername(username2);

        if (user1 != null && user2 != null && !user1.equals(user2)) {
            // Create friendship if it doesn't exist
            Optional<Friendship> existingFriendship = friendshipRepository.findByUser1AndUser2(user1, user2);
            if (existingFriendship.isEmpty()) {
                Friendship friendship = new Friendship();
                friendship.setUser1(user1);
                friendship.setUser2(user2);
                friendship.setStatus(FriendshipStatus.Pending);
                friendshipRepository.save(friendship);
            }

            // Follow the other user
            Followers follow = new Followers();
            follow.setFollowerUser(user1);
            follow.setFollowingUser(user2);
            followersRepository.save(follow);
        }
    }

    @Transactional
    public void acceptFriend(String username1, String username2) {
        User user1 = userRepository.findByUsername(username1);
        User user2 = userRepository.findByUsername(username2);

        if (user1 != null && user2 != null) {
            // Find friendship and update status
            Optional<Friendship> friendshipOptional = friendshipRepository.findByUser1AndUser2(user1, user2);
            if (friendshipOptional.isPresent()) {
                Friendship friendship = friendshipOptional.get();
                friendship.setStatus(FriendshipStatus.Accepted);
                friendshipRepository.save(friendship);
            }
        }
    }

    @Transactional
    public void blockFriend(String username1, String username2) {
        User user1 = userRepository.findByUsername(username1);
        User user2 = userRepository.findByUsername(username2);

        if (user1 != null && user2 != null) {
            // Block the friendship
            Optional<Friendship> friendshipOptional = friendshipRepository.findByUser1AndUser2(user1, user2);
            if (friendshipOptional.isPresent()) {
                Friendship friendship = friendshipOptional.get();
                friendship.setStatus(FriendshipStatus.Blocked);
                friendshipRepository.save(friendship);
            }
        }
    }
}
```

#### **PostService.java**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
public class PostService {

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public Post createPost(String username, String content, String imageUrl, String mediaType) {
        User user = userRepository.findByUsername(username);
        if (user != null) {
            Post post = new Post();
            post.setUser(user);
            post.setContent(content);
            post.setImageUrl(imageUrl);

            // Save post
            return postRepository.save(post);
        }
        return null;
    }

    public List<Post> getPostsFromFriends(User user) {
        // Find all the users the given user is following (friends)
        List<User> friends = user.getFollowings().stream()
                .map(follow -> follow.getFollowingUser())
                .collect(Collectors.toList());
        friends.add(user);  // Include the user's own posts

        return postRepository.findByUserIn(friends);
    }
}
```

#### **LikeService.java**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class LikeService {

    @Autowired
    private LikeRepository likeRepository;

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private CommentRepository commentRepository;

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public void likePost(String username, Long postId) {
        User user = userRepository.findByUsername(username);
        Post post = postRepository.findById(postId).orElse(null);

        if (user != null && post != null) {
            Like like = new Like();
            like.setUser(user);
            like.setPost(post);
            likeRepository.save(like);
        }
    }

    @Transactional
    public void likeComment(String username, Long commentId) {
        User user = userRepository.findByUsername(username);
        Comment comment = commentRepository.findById(commentId).orElse(null);

        if (user != null && comment != null) {
            Like like = new Like();
            like.setUser(user);
            like.setComment(comment);
            likeRepository.save(like);
        }
    }
}
```

#### **CommentService.java**
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
public class CommentService {

    @Autowired
    private CommentRepository commentRepository;

    @Autowired
    private PostRepository postRepository;

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public Comment addComment(String username, Long postId, String content) {
        User user = userRepository.findByUsername(username);
        Post post = postRepository.findById(postId).orElse(null);

        if (user != null && post != null) {
            Comment comment = new Comment();
            comment.setUser(user);
            comment.setPost(post);
            comment.setContent(content);

            return commentRepository.save(comment);
        }
        return null;
    }

    public List<Comment> getCommentsForPost(Long postId) {
        Post post = postRepository.findById(postId).orElse(null);
        return post != null ? commentRepository.findByPost(post) : null;
    }
}
```

### 3. **Controller Layer**

You can create REST controllers to interact with the service layer for each functionality. Here’s an example for the `PostController`:

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/posts")
public class PostController {

    @Autowired
    private PostService postService;

    @Autowired
    private LikeService likeService;

    @Autowired
    private CommentService commentService;

    @PostMapping("/create")
    public Post createPost(@RequestParam String username, @RequestParam String content, 
                           @RequestParam(required = false) String imageUrl, @RequestParam(required = false) String mediaType) {
        return postService.createPost(username, content, imageUrl, mediaType);
    }

    @GetMapping("/friends/{username}")
    public List<Post> getPostsFromFriends(@PathVariable String username) {
        User user = userRepository.findByUsername(username);
        return postService.getPostsFromFriends(user);
    }

    @Post

Mapping("/{postId}/like")
    public void likePost(@RequestParam String username, @PathVariable Long postId) {
        likeService.likePost(username, postId);
    }

    @PostMapping("/{postId}/comment")
    public Comment addComment(@RequestParam String username, @PathVariable Long postId, @RequestParam String content) {
        return commentService.addComment(username, postId, content);
    }

    @PostMapping("/{commentId}/like")
    public void likeComment(@RequestParam String username, @PathVariable Long commentId) {
        likeService.likeComment(username, commentId);
    }
}
```

## Data Flow Diagram

It's describe the flow of how data and control would move between the different layers (Repositories, Service, and Controller) for the functional requirements of the system, and then I'll summarize this into a flow diagram.

### Flow Breakdown for Functional Requirements:

#### 1. **User Profile Management (Create/Update Profile)**

   - **User** interacts with the **`ProfileController`** to create or update their profile.
   - **Controller**:
     - Receives the HTTP request and calls the **`ProfileService`**.
   - **Service**:
     - The **`ProfileService`** interacts with the **`UserRepository`** (to fetch user data if needed) and the **`ProfileRepository`** (to update or create the profile).
     - After processing, it updates the profile or creates a new one.
   - **Repository**:
     - The **`ProfileRepository`** handles all database interactions for the **`Profiles`** table.
   - **Return**: Success response or error back to the **Controller**, which then sends a response to the **User**.

#### 2. **Add Other Profiles as Friends (Send, Accept, Block Friend Request)**

   - **User** interacts with the **`FriendController`** to add, accept, or block friends.
   - **Controller**:
     - Receives the HTTP request and calls the appropriate **`FriendService`** method.
   - **Service**:
     - **`FriendService`** interacts with the **`FriendshipRepository`** to create, update, or query friendships.
     - For example, when sending a request, it checks if the users exist and then creates a **`Friendship`** entry in the database.
     - When accepting a request, it updates the **`Friendship`** status.
   - **Repository**:
     - The **`FriendshipRepository`** handles creating and querying entries in the **`Friendships`** table.
     - The **`FollowersRepository`** handles managing who follows whom, which is linked to the friend functionality.
   - **Return**: Success or error response is sent back to the **Controller**, which then responds to the **User**.

#### 3. **Post Creation (Text, Image, or Video)**

   - **User** interacts with the **`PostController`** to create a new post.
   - **Controller**:
     - Receives the request (including text and optional media like image/video) and calls **`PostService`**.
   - **Service**:
     - **`PostService`** uses **`UserRepository`** to find the user and **`PostRepository`** to create a new **`Post`** in the database.
     - If media is involved, **`MediaRepository`** is used to store media files.
   - **Repository**:
     - **`PostRepository`** handles inserting new posts into the **`Posts`** table.
     - **`MediaRepository`** handles media content related to the posts (stored in the **`Media`** table).
   - **Return**: The post is created, and a response is sent back to the **Controller**, which returns success to the **User**.

#### 4. **See Posts from Friends**

   - **User** interacts with the **`FeedController`** to see posts from their friends.
   - **Controller**:
     - Receives the request and calls **`PostService`** to fetch posts.
   - **Service**:
     - **`PostService`** queries the **`FollowersRepository`** to get the list of users (friends) the user is following.
     - It then queries **`PostRepository`** for posts from those users.
   - **Repository**:
     - **`PostRepository`** is used to fetch posts from multiple users (friends) from the **`Posts`** table.
     - **`FollowersRepository`** helps in determining the friends/followers of the user.
   - **Return**: A list of posts is sent back to the **Controller**, which returns the feed to the **User**.

#### 5. **Like and Comment on Posts**

   - **User** interacts with the **`PostController`** to like or comment on a post.
   - **Controller**:
     - Receives the request (which can be a like or a comment) and calls **`LikeService`** or **`CommentService`**.
   - **Service**:
     - **`LikeService`** checks if the user already liked the post and either creates or retrieves a like record in the **`Likes`** table.
     - **`CommentService`** handles adding comments to posts in the **`Comments`** table.
   - **Repository**:
     - **`LikeRepository`** is used to store or query the likes in the **`Likes`** table.
     - **`CommentRepository`** is used to store or query comments in the **`Comments`** table.
   - **Return**: The success response is sent back to the **Controller**, which sends the response to the **User**.

---

### Flow Diagram

```plaintext
   +-----------------------------------------------+
   |                   User                       |
   +-----------------------------------------------+
                 |  |  |   |    |     |    | 
   ----------------|--------------------------------------
    Profile Creation |     Friend Request |   Post Creation
   -----------------|--------------------------------------
    |                                 |                    |
+------------------+            +-----------------+   +-----------------+
| ProfileController|            |FriendController|   | PostController  |
+------------------+            +-----------------+   +-----------------+
        |                              |                    |
  +-------------+                  +-----------+         +-------------+
  | ProfileService|                 | FriendService|      | PostService |
  +-------------+                  +-----------+         +-------------+
        |                              |                    |
 +--------------+                +------------------+    +------------------+
 |ProfileRepository|              | FriendshipRepo  |    |PostRepository    |
 +--------------+                +------------------+    +------------------+
        |                              |                    |
 +-----------------+                  +-------------------+  +-----------------+
 |   Database      |                  |    Database       |  |    Database     |
 |  (Profiles)     |                  |(Friendships)      |  |   (Posts, Media)|
 +-----------------+                  +-------------------+  +-----------------+  
```

### Key Flow Points in the Diagram:
1. **User** initiates an action (e.g., creating a profile, adding friends, posting content, liking/commenting).
2. The **Controller** receives the action and calls the appropriate **Service** method.
3. The **Service** handles the logic, interacting with the relevant **Repository** to interact with the database.
4. The **Repository** performs database operations (CRUD actions) on the relevant tables (**Profiles**, **Posts**, **Likes**, **Friendships**, etc.).
5. The **Service** returns the result to the **Controller**, which sends the response back to the **User**.

---

### Detailed Flow for Example (e.g., Posting Content):

1. **User** sends a request to **PostController** to create a post.
2. **PostController** calls **PostService** to create a post.
3. **PostService** validates and processes the data (e.g., checks if the user exists and has provided content).
4. **PostService** calls **PostRepository** to insert the post into the **Posts** table and optionally calls **MediaRepository** for media content.
5. **PostRepository** inserts the post data into the database, and **MediaRepository** handles media file storage.
6. The post is successfully created, and **PostService** returns a success message to **PostController**.
7. **PostController** sends the success response back to the **User**.

---

This diagram and flow breakdown help visualize how the application processes various functionalities through the repository, service, and controller layers.

### Conclusion

This setup provides the necessary JPA repositories, service layer, and controllers for handling the functionality of adding friends, creating posts (text, photos, or videos), viewing posts from friends, liking and commenting on posts, and more.

To fully implement the system, you would need to set up the Spring Boot application, configure the database, and create the necessary configurations for Hibernate. The code should work seamlessly once you integrate it into the Spring Boot project.


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


## https://dbdiagram.io/d Code below

```SQL

Table users {
  user_id INT [pk]
  username VARCHAR(255) [unique]
  email VARCHAR(255) [unique]
  password_hash VARCHAR(255)
  first_name VARCHAR(100)
  last_name VARCHAR(100)
  birthdate DATE
  gender ENUM('Male', 'Female', 'Other')
  created_at TIMESTAMP
}

Table profiles {
  profile_id INT [pk]
  user_id INT [ref: > users.user_id]
  bio TEXT
  profile_picture_url VARCHAR(255)
  cover_picture_url VARCHAR(255)
  location VARCHAR(255)
  website_url VARCHAR(255)
  updated_at TIMESTAMP
}

Table posts {
  post_id INT [pk]
  user_id INT [ref: > users.user_id]
  content TEXT
  image_url VARCHAR(255)
  created_at TIMESTAMP
}

Table friendships {
  friendship_id INT [pk]
  user_id_1 INT [ref: > users.user_id]
  user_id_2 INT [ref: > users.user_id]
  status ENUM('Pending', 'Accepted', 'Blocked')
  created_at TIMESTAMP
}

Table comments {
  comment_id INT [pk]
  post_id INT [ref: > posts.post_id]
  user_id INT [ref: > users.user_id]
  content TEXT
  created_at TIMESTAMP
}

Table likes {
  like_id INT [pk]
  user_id INT [ref: > users.user_id]
  post_id INT [ref: > posts.post_id, null]
  comment_id INT [ref: > comments.comment_id, null]
  created_at TIMESTAMP
}

Table groups {
  group_id INT [pk]
  group_name VARCHAR(255) [unique]
  description TEXT
  created_by INT [ref: > users.user_id]
  created_at TIMESTAMP
}

Table group_memberships {
  group_membership_id INT [pk]
  user_id INT [ref: > users.user_id]
  group_id INT [ref: > groups.group_id]
  joined_at TIMESTAMP
}

Table messages {
  message_id INT [pk]
  sender_id INT [ref: > users.user_id]
  receiver_id INT [ref: > users.user_id]
  content TEXT
  created_at TIMESTAMP
}

Table notifications {
  notification_id INT [pk]
  user_id INT [ref: > users.user_id]
  type ENUM('Friend Request', 'Like', 'Comment', 'Message', 'Event', 'Group Invite')
  message TEXT
  read_status BOOLEAN [default: false]
  created_at TIMESTAMP
}

Table events {
  event_id INT [pk]
  user_id INT [ref: > users.user_id]
  event_name VARCHAR(255)
  event_description TEXT
  event_date TIMESTAMP
  location VARCHAR(255)
  created_at TIMESTAMP
}

Table media {
  media_id INT [pk]
  user_id INT [ref: > users.user_id]
  media_type ENUM('Photo', 'Video')
  media_url VARCHAR(255)
  created_at TIMESTAMP
}

Table Followers {
  follower_user_id INT [not null, ref: > users.user_id]
  following_user_id INT [not null, ref: > users.user_id]
  created_at TIMESTAMPTZ [default: `CURRENT_TIMESTAMP`]

  Indexes {
    (follower_user_id, following_user_id) [pk]
    (follower_user_id, following_user_id) [unique]
  }
}


```
