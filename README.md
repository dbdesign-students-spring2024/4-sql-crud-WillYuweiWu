# Part 1

In this part, I first generated a CSV file from [Mockaroo](https://mockaroo.com/) and save it as [`restaurant.csv`](https://github.com/dbdesign-students-spring2024/4-sql-crud-WillYuweiWu/blob/main/data/restaurants.csv), and performed the following regarding this database. 

## Database Design and Data Importation

To achieve the goals of this part, I constructed two databases, `restaurants` and `reviews`, using the following commands: 

```
CREATE TABLE restaurants (
    restaurant_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    price_tier TEXT NOT NULL,
    neighborhood TEXT NOT NULL,
    opening_hours TEXT NOT NULL,
    average_rating REAL,
    good_for_kids BOOLEAN NOT NULL
);

CREATE TABLE reviews (
    review_id INTEGER PRIMARY KEY,
    restaurant_id INTEGER,
    user_id INTEGER,
    rating REAL,
    comment TEXT,
    date_posted TEXT,
    FOREIGN KEY (restaurant_id) REFERENCES restaurants(restaurant_id)
);
```

Note that for the `reviews` table, `restaurant_id` is the foreign key. 

After checking that the columns in the CSV file are in the same order as the columns in my SQLite table, I used the following commands to import the CSV file into the `restaurants` table: 

```
sqlite> sqlite3 restaurants.db
   ...> .mode csv
   ...> .import /Users/willwu/Desktop/NYU/CSCI-UA 60/4-sql-crud-WillYuweiWu/data/restaurants.csv restaurants
```

## Queries

Once the data is imported, I use the following SQL queries to achieve each of the specified tasks. 

### 1. Find all cheap restaurants in a particular neighborhood (pick any neighborhood as an example). 

Assume that I want to find all cheap restaurants in Chinatown. 

```
SELECT * FROM restaurants WHERE price_tier = 'cheap' AND neighborhood = 'Chinatown';
```

### 2. Find all restaurants in a particular genre (pick any genre as an example) with 3 stars or more, ordered by the number of stars in descending order. 

Assume that I want to find all Italian restaurants with 3 stars or more. 

```
SELECT * FROM restaurants WHERE category = 'Italian' AND average_rating >= 3 ORDER BY average_rating DESC;
```

### 3. Find all restaurants that are open now (see hint below).

For the restaurants that are open now, I compared the closing time in `opening_hours` with the current time in the format `HH:MM`:

```
SELECT * FROM restaurants WHERE substr(opening_hours, 1, 5) > CURRENT_TIME;
```

### 4. Leave a review for a restaurant (pick any restaurant as an example; note that leaving a review has no automatic effect on the average rating of the restaurant).

Assume that I am leaving a review for the restaurant with a `restaurant_id` of 74 and a combination of       `user_id`, `rating`, `comment`, and `date_posted` as follows:

```
INSERT INTO reviews (restaurant_id, user_id, rating, comment, date_posted) VALUES (74, 100, 4.5, 'Great food, even greater service!', '2024-02-28');
```

### 5. Delete all restaurants that are not good for kids.

```
DELETE FROM restaurants WHERE good_for_kids = FALSE;
```

### 6. Find the number of restaurants in each NYC neighborhood.

```
SELECT neighborhood, COUNT(*) AS num_restaurants FROM restaurants GROUP BY neighborhood;
```

# Part 2

In this part, I generated two CSV files from [Mockaroo](https://mockaroo.com/): [`users.csv`](https://github.com/dbdesign-students-spring2024/4-sql-crud-WillYuweiWu/blob/main/data/users.csv) and [`post.csv`](https://github.com/dbdesign-students-spring2024/4-sql-crud-WillYuweiWu/blob/main/data/posts.csv) and then performed the following on the databases. 

## Database Design and Data Importation

To achieve the goals of this part, I constructed two SQL tables, `users` and `posts`, using the following commands:

```
CREATE TABLE users (
    user_id INTEGER PRIMARY KEY AUTOINCREMENT,
    email TEXT UNIQUE NOT NULL,
    password TEXT NOT NULL,
    handle TEXT UNIQUE NOT NULL
);

CREATE TABLE posts (
    post_id INTEGER PRIMARY KEY AUTOINCREMENT,
    user_id INTEGER NOT NULL,
    content TEXT NOT NULL,
    post_type TEXT NOT NULL,
    recipient_id INTEGER,
    visible BOOLEAN NOT NULL DEFAULT 1,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    FOREIGN KEY (recipient_id) REFERENCES users(user_id)
);
```

Note that `user_id` and `recipient_id` are foreign keys in the `posts` table that refers to the `user_id` column in the `users` table. 

After checking that the columns in the CSV files are in the same order as the columns in my SQLite tables, I used the following commands to import the CSV files into the `users` and the `posts` table: 

```
sqlite> sqlite3 users.db
   ...> sqlite3 posts.db
   ...> .mode csv
   ...> .import /Users/willwu/Desktop/NYU/CSCI-UA 60/4-sql-crud-WillYuweiWu/data/users.csv users
   ...> .import /Users/willwu/Desktop/NYU/CSCI-UA 60/4-sql-crud-WillYuweiWu/data/posts.csv posts
```

## Queries
Once the data is imported, I use the following SQL queries to achieve each of the specified tasks. 

### 1. Register a new User.

Assume some random email, password, and username. 

```
INSERT INTO users (email, password, handle) VALUES ('ufhueiwj@gmail.com', 'Wjuh$3nfuen', 'jfiewfwf23');
```

### 2. Create a new Message sent by a particular User to a particular User (pick any two Users for example).

Assume that a new message is sent from the user with ID 1 to the user with ID 2. 

```
INSERT INTO posts (user_id, content, post_type, recipient_id, visible, created_at) VALUES (1, 'Hello there!', 'Message', 2, 1, '2024-02-28 12:00:00');
```

### 3. Create a new Story by a particular User (pick any User for example).

Assume that a new story is posted by the user with ID 1. 

```
INSERT INTO posts (user_id, content, post_type, visible, created_at) VALUES (1, 'I love the world!', 'Story', 1, '2024-02-28 12:00:00');
```

### 4. Show the 10 most recent visible Messages and Stories, in order of recency.

```
SELECT * FROM posts WHERE visible = 1 ORDER BY created_at DESC LIMIT 10;
```

### 5. Show the 10 most recent visible Messages sent by a particular User to a particular User (pick any two Users for example), in order of recency.

Assume that I want the 10 mist recent visible messages sent by the user with ID 1 to the user with ID 2. 

```
SELECT * FROM posts WHERE post_type = 'Message' AND user_id = 1 AND recipient_id = 2 AND visible = 1 ORDER BY created_at DESC LIMIT 10;
```

### 6. Make all Stories that are more than 24 hours old invisible.

```
UPDATE posts SET visible = 0 WHERE post_type = 'Story' AND (JULIANDAY('now') - JULIANDAY(created_at)) * 24 > 24;
```

### 7. Show all invisible Messages and Stories, in order of recency.

```
SELECT * FROM posts WHERE visible = 0 ORDER BY created_at DESC;
```

### 8. Show the number of posts by each User.

```
SELECT user_id, COUNT(*) AS total_posts FROM posts GROUP BY user_id;
```

### 9. Show the post text and email address of all posts and the User who made them within the last 24 hours.

```
SELECT p.content, u.email FROM posts p JOIN users u ON p.user_id = u.user_id WHERE (JULIANDAY('now') - JULIANDAY(p.created_at)) * 24 <= 24;
```

### 10. Show the email addresses of all Users who have not posted anything yet.

```
SELECT email FROM users WHERE user_id NOT IN (SELECT DISTINCT user_id FROM posts);
```