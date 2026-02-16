# Database Overview

## Design Philosophy

The database is designed around properties and users as core entities, with supporting tables for features like favorites, inquiries, and search tracking.

## Table Groups

**User Management (5 tables)**
- `users` - Authentication and basic user data
- `user_profiles` - Extended profile information
- `password_resets` - Password recovery tokens
- `user_favorites` - Saved properties per user
- `user_notifications` - In-app notifications

**Property Management (3 tables)**
- `properties` - Core property listings
- `property_images` - Multiple images per property
- `property_comparisons` - User comparison lists

**Inquiries & Contact (2 tables)**
- `inquiries` - Property-specific inquiries
- `contact_messages` - General contact form

**Analytics & Search (3 tables)**
- `search_suggestions` - Popular search keywords
- `popular_searches` - Search analytics
- `viewing_history` - User property views

**Reference (1 table)**
- `states` - Nigerian states for dropdowns

## Core Table Structures

### users
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    full_name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    phone VARCHAR(20),
    password VARCHAR(255) NOT NULL,
    user_type ENUM('client','agent','admin') DEFAULT 'client',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
properties
sql
CREATE TABLE properties (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(12,2) NOT NULL,
    location VARCHAR(255) NOT NULL,
    city VARCHAR(100),
    state VARCHAR(100),
    property_type ENUM('house','apartment','land','commercial') DEFAULT 'house',
    transaction_type ENUM('sale','rent','shortlet') DEFAULT 'sale',
    bedrooms INT,
    bathrooms INT,
    square_feet INT,
    amenities TEXT,
    featured TINYINT(1) DEFAULT 0,
    status ENUM('available','sold','rented') DEFAULT 'available',
    view_count INT DEFAULT 0,
    agent_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    FOREIGN KEY (agent_id) REFERENCES users(id) ON DELETE SET NULL,
    INDEX (title),
    INDEX (location),
    INDEX (city),
    INDEX (state),
    INDEX (price),
    INDEX (status),
    INDEX (featured)
);
property_images
sql
CREATE TABLE property_images (
    id INT PRIMARY KEY AUTO_INCREMENT,
    property_id INT NOT NULL,
    image_path VARCHAR(500) NOT NULL,
    is_primary TINYINT(1) DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (property_id) REFERENCES properties(id) ON DELETE CASCADE
);
inquiries
sql
CREATE TABLE inquiries (
    id INT PRIMARY KEY AUTO_INCREMENT,
    property_id INT NOT NULL,
    user_id INT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL,
    phone VARCHAR(20),
    message TEXT,
    status ENUM('new','contacted','closed') DEFAULT 'new',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (property_id) REFERENCES properties(id) ON DELETE CASCADE,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE SET NULL
);
Key Relationships
properties → property_images (one-to-many)

users → user_favorites → properties (many-to-many via favorites)

users → inquiries (one-to-many, optional)

properties → inquiries (one-to-many)

Indexing Strategy
All foreign keys are indexed

Searchable columns have indexes (title, location, city, state, price)

Status and featured flags have indexes for filtering

Email field has unique index for login