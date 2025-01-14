# DB-Design
Instagram Database Design

<img width="1298" alt="image" src="https://github.com/user-attachments/assets/56e8bb5e-2de0-43fe-b083-117023515fd6" />


```sql

-- Users Table: Stores information about users of the platform
CREATE TABLE Users (
    user_id SERIAL PRIMARY KEY,                    -- Unique ID for each user (auto-incremented using SERIAL)
    username VARCHAR(255) UNIQUE NOT NULL,         -- Unique username (indexed for faster lookup)
    email VARCHAR(255) UNIQUE NOT NULL,            -- User's email address (unique)
    password_hash VARCHAR(255) NOT NULL,           -- Hashed password for user authentication
    full_name VARCHAR(255) NOT NULL,               -- User's full name
    bio TEXT,                                      -- Short description or bio of the user
    profile_picture_url VARCHAR(255),              -- URL of the user's profile picture
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the user account was created
    last_login TIMESTAMPTZ,                        -- Timestamp of the last login
    is_verified BOOLEAN DEFAULT FALSE,             -- Whether the user is verified (e.g., a public figure)
    is_active BOOLEAN DEFAULT TRUE                 -- Whether the account is active
);

-- Posts Table: Stores posts created by users
CREATE TABLE Posts (
    post_id SERIAL PRIMARY KEY,                    -- Unique ID for each post
    user_id INT NOT NULL,                          -- User who created the post (foreign key to Users table)
    content TEXT,                                  -- Text content of the post
    image_url VARCHAR(255),                        -- URL to the image or video associated with the post
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the post was created
    updated_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the post was last updated
    is_deleted BOOLEAN DEFAULT FALSE,              -- Marks whether the post is deleted (soft delete)
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE -- Ensures posts are deleted when the user is deleted
);

-- Comments Table: Stores comments on posts
CREATE TABLE Comments (
    comment_id SERIAL PRIMARY KEY,                 -- Unique ID for each comment
    post_id INT NOT NULL,                           -- The post on which the comment was made (foreign key to Posts table)
    user_id INT NOT NULL,                           -- The user who made the comment (foreign key to Users table)
    content TEXT NOT NULL,                          -- Content of the comment
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the comment was made
    FOREIGN KEY (post_id) REFERENCES Posts(post_id) ON DELETE CASCADE, -- Ensures comments are deleted when the post is deleted
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE  -- Ensures comments are deleted when the user is deleted
);

-- Likes Table: Stores likes on posts
CREATE TABLE Likes (
    like_id SERIAL PRIMARY KEY,                    -- Unique ID for each like
    post_id INT NOT NULL,                           -- The post that was liked (foreign key to Posts table)
    user_id INT NOT NULL,                           -- The user who liked the post (foreign key to Users table)
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the like was made
    FOREIGN KEY (post_id) REFERENCES Posts(post_id) ON DELETE CASCADE, -- Ensures likes are deleted when the post is deleted
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE, -- Ensures likes are deleted when the user is deleted
    CONSTRAINT unique_like UNIQUE (post_id, user_id) -- Ensures a user can only like a post once
);

-- Followers Table: Represents the following relationship between users (many-to-many relationship)
CREATE TABLE Followers (
    follower_id INT NOT NULL,                      -- The user who is following (foreign key to Users table)
    following_id INT NOT NULL,                     -- The user being followed (foreign key to Users table)
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the follow relationship was created
    PRIMARY KEY (follower_id, following_id),       -- Composite primary key for unique follow relationships
    FOREIGN KEY (follower_id) REFERENCES Users(user_id) ON DELETE CASCADE, -- Deletes follow records when the user is deleted
    FOREIGN KEY (following_id) REFERENCES Users(user_id) ON DELETE CASCADE, -- Deletes follow records when the followed user is deleted
    CONSTRAINT unique_follow UNIQUE (follower_id, following_id) -- Prevents duplicate follow relationships
);

-- Direct Messages Table: Stores private messages sent between users
CREATE TABLE DirectMessages (
    message_id SERIAL PRIMARY KEY,                  -- Unique ID for each message
    sender_id INT NOT NULL,                         -- The user who sent the message (foreign key to Users table)
    receiver_id INT NOT NULL,                       -- The user who received the message (foreign key to Users table)
    content TEXT NOT NULL,                          -- Content of the message
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the message was sent
    is_read BOOLEAN DEFAULT FALSE,                  -- Marks whether the message has been read
    FOREIGN KEY (sender_id) REFERENCES Users(user_id) ON DELETE CASCADE, -- Ensures messages are deleted when the sender is deleted
    FOREIGN KEY (receiver_id) REFERENCES Users(user_id) ON DELETE CASCADE -- Ensures messages are deleted when the receiver is deleted
);

-- Media Table: Stores media files (images, videos, etc.)
CREATE TABLE Media (
    media_id SERIAL PRIMARY KEY,                    -- Unique ID for each media file
    user_id INT NOT NULL,                           -- The user who uploaded the media (foreign key to Users table)
    media_type VARCHAR(10) CHECK (media_type IN ('image', 'video', 'gif')) NOT NULL, -- Type of media (image, video, etc.)
    media_url VARCHAR(255) NOT NULL,                 -- URL to the media file
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the media file was uploaded
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE -- Ensures media is deleted when the user is deleted
);

-- Notifications Table: Stores notifications for users (like likes, comments, etc.)
CREATE TABLE Notifications (
    notification_id SERIAL PRIMARY KEY,              -- Unique ID for each notification
    user_id INT NOT NULL,                            -- The user who will receive the notification (foreign key to Users table)
    notification_type VARCHAR(50) CHECK (notification_type IN ('like', 'comment', 'follow', 'mention', 'message', 'story')) NOT NULL, -- Type of notification
    message TEXT,                                    -- The content of the notification
    is_read BOOLEAN DEFAULT FALSE,                   -- Marks whether the notification has been read
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the notification was created
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE -- Ensures notifications are deleted when the user is deleted
);

-- Stories Table: Stores stories created by users
CREATE TABLE Stories (
    story_id SERIAL PRIMARY KEY,                     -- Unique ID for each story
    user_id INT NOT NULL,                            -- User who created the story (foreign key to Users table)
    media_url VARCHAR(255),                          -- URL to the media file associated with the story
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP, -- Timestamp when the story was created
    expires_at TIMESTAMPTZ,                          -- Timestamp when the story expires
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE -- Ensures stories are deleted when the user is deleted
);

-- Hashtags Table: Stores hashtags used in posts (e.g., #vacation)
CREATE TABLE Hashtags (
    hashtag_id SERIAL PRIMARY KEY,                   -- Unique ID for each hashtag
    hashtag_name VARCHAR(100) UNIQUE NOT NULL        -- Hashtag name (e.g., '#vacation')
);

-- Post Hashtags Table: Many-to-many relationship between posts and hashtags
CREATE TABLE PostHashtags (
    post_id INT NOT NULL,                            -- The post associated with the hashtag (foreign key to Posts table)
    hashtag_id INT NOT NULL,                         -- The hashtag associated with the post (foreign key to Hashtags table)
    PRIMARY KEY (post_id, hashtag_id),               -- Composite primary key
    FOREIGN KEY (post_id) REFERENCES Posts(post_id) ON DELETE CASCADE, -- Ensures hashtags are deleted when the post is deleted
    FOREIGN KEY (hashtag_id) REFERENCES Hashtags(hashtag_id) ON DELETE CASCADE -- Ensures hashtags are deleted when the hashtag is deleted
);

-- Activity Log Table: Stores user activity (e.g., logins, post interactions, etc.)
CREATE TABLE ActivityLog (
    activity_id SERIAL PRIMARY KEY,                  -- Unique ID for each activity record
    user_id INT NOT NULL,                             -- The user performing the activity (foreign key to Users table)
    activity_type VARCHAR(50) CHECK (activity_type IN ('login', 'post', 'comment', 'like', 'follow')) NOT NULL, -- Type of activity
    activity_description TEXT,                        -- Detailed description of the activity
    created_at TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,  -- Timestamp when the activity occurred
    FOREIGN KEY (user_id) REFERENCES Users(user_id) ON DELETE CASCADE -- Ensures activity records are deleted when the user is deleted
);


```

![image](https://user-images.githubusercontent.com/115500959/196991411-265404f4-add1-42de-a7fd-9ecc6080c4be.png)



## Below Code for https://dbdiagram.io/d

```SQL

Table users {
  user_id int [pk, increment]             // Unique ID for each user (auto-incremented)
  username varchar(255) [unique, not null] // Unique username
  email varchar(255) [unique, not null]    // User's email address
  password_hash varchar(255) [not null]    // Hashed password for authentication
  full_name varchar(255) [not null]        // User's full name
  bio text                                  // Bio of the user
  profile_picture_url varchar(255)          // URL of the profile picture
  created_at timestamp                      // Timestamp when account was created
  last_login timestamp                      // Timestamp of the last login
  is_verified boolean [default: false]      // Whether the user is verified
  is_active boolean [default: true]         // Whether the user account is active
}

Table posts {
  post_id int [pk, increment]               // Unique post ID
  user_id int [ref: > users.user_id]         // User who created the post
  content text                              // Content of the post
  image_url varchar(255)                     // URL of the media/image in the post
  created_at timestamp                       // Timestamp of post creation
  updated_at timestamp                      // Timestamp of post update
  is_deleted boolean [default: false]        // Marks whether the post is deleted
}

Table comments {
  comment_id int [pk, increment]             // Unique comment ID
  post_id int [ref: > posts.post_id]         // Post on which the comment was made
  user_id int [ref: > users.user_id]         // User who made the comment
  content text [not null]                    // Content of the comment
  created_at timestamp                      // Timestamp of comment creation
}

Table likes {
  like_id int [pk, increment]                // Unique like ID
  post_id int [ref: > posts.post_id]          // Post that was liked
  user_id int [ref: > users.user_id]          // User who liked the post
  created_at timestamp                        // Timestamp of like

  Indexes {
    (post_id, user_id) [unique]               // Prevents duplicate likes by the same user on the same post
  }
}

Table followers {
  follower_user_id int [ref: > users.user_id]     // User who is following
  following_user_id int [ref: > users.user_id]    // User being followed
  created_at timestamp       // Timestamp of follow action
  Indexes {
    (follower_user_id, following_user_id) [pk]
    (follower_user_id, following_user_id) [unique]
  }
}

Table direct_messages {
  message_id int [pk, increment]             // Unique message ID
  sender_id int [ref: > users.user_id]       // Sender of the message
  receiver_id int [ref: > users.user_id]     // Receiver of the message
  content text [not null]                    // Message content
  created_at timestamp      // Timestamp of the message
  is_read boolean [default: false]           // Whether the message is read
}

Table media {
  media_id int [pk, increment]               // Unique media ID
  user_id int [ref: > users.user_id]         // User who uploaded the media
  media_type varchar(10) [not null]           // Type of media (image, video, etc.)
  media_url varchar(255) [not null]           // URL of the media file
  created_at timestamp       // Timestamp of media upload
}

Table notifications {
  notification_id int [pk, increment]        // Unique notification ID
  user_id int [ref: > users.user_id]         // User receiving the notification
  notification_type varchar(50) [not null]   // Type of notification
  message text                               // Content of the notification
  is_read boolean [default: false]           // Whether the notification has been read
  created_at timestamp       // Timestamp of notification creation
}

Table stories {
  story_id int [pk, increment]               // Unique story ID
  user_id int [ref: > users.user_id]         // User who created the story
  media_url varchar(255)                     // URL to the media for the story
  created_at TIMESTAMP      // Timestamp when the story was created
  expires_at timestamp                      // Timestamp when the story expires
}

Table hashtags {
  hashtag_id int [pk, increment]             // Unique hashtag ID
  hashtag_name varchar(100) [unique, not null] // Hashtag name (e.g., #vacation)
}

Table post_hashtags {
  post_id int [ref: > posts.post_id]         // Post associated with the hashtag
  hashtag_id int [ref: > hashtags.hashtag_id] // Hashtag associated with the post
  primary key (post_id, hashtag_id)          // Composite primary key
}

Table activity_log {
  activity_id int [pk, increment]            // Unique activity ID
  user_id int [ref: > users.user_id]         // User performing the activity
  activity_type varchar(50) [not null]       // Type of activity
  activity_description text                  // Detailed description of the activity
  created_at TIMESTAMP      // Timestamp of activity
}


```

