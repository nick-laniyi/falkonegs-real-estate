# Technical Implementation Details

## Authentication System

The authentication system uses PHP sessions with secure password hashing:

```php
// Password hashing
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Password verification
if (password_verify($input_password, $stored_hash)) {
    // Login successful
    session_regenerate_id(true);
    $_SESSION['user_id'] = $user['id'];
    $_SESSION['user_name'] = $user['full_name'];
    $_SESSION['user_type'] = $user['user_type'];
}
Security features:

Session regeneration after login

Password hashing with bcrypt

Remember me tokens with expiration

Login attempt limiting

CSRF tokens on forms

Property Search Implementation
The search query dynamically builds based on selected filters:

php
// Base query
$sql = "SELECT p.*, 
        (SELECT image_path FROM property_images 
         WHERE property_id = p.id AND is_primary = 1 LIMIT 1) as primary_image
        FROM properties p
        WHERE p.status = 'available'";

$params = [];

// Add filters
if (!empty($type)) {
    $sql .= " AND p.property_type = ?";
    $params[] = $type;
}

if (!empty($transaction)) {
    $sql .= " AND p.transaction_type = ?";
    $params[] = $transaction;
}

if (!empty($location)) {
    $sql .= " AND (p.location LIKE ? OR p.city LIKE ?)";
    $params[] = "%$location%";
    $params[] = "%$location%";
}

if (!empty($price_range)) {
    list($min, $max) = explode('-', $price_range);
    $sql .= " AND p.price BETWEEN ? AND ?";
    $params[] = $min;
    $params[] = $max;
}

$sql .= " ORDER BY p.created_at DESC";
Image Upload Handling
Property images are uploaded with validation and organized in folders:

php
// Image validation
$allowed = ['image/jpeg', 'image/png', 'image/webp'];
$max_size = 5 * 1024 * 1024; // 5MB

if (!in_array($_FILES['image']['type'], $allowed)) {
    $error = "Only JPG, PNG, and WebP images are allowed";
} elseif ($_FILES['image']['size'] > $max_size) {
    $error = "Image size must be less than 5MB";
} else {
    // Generate unique filename
    $extension = pathinfo($_FILES['image']['name'], PATHINFO_EXTENSION);
    $filename = uniqid() . '_' . time() . '.' . $extension;
    $upload_path = 'uploads/properties/' . $filename;
    
    if (move_uploaded_file($_FILES['image']['tmp_name'], $upload_path)) {
        // Save to database
        $sql = "INSERT INTO property_images (property_id, image_path, is_primary) 
                VALUES (?, ?, ?)";
        // Execute with parameters
    }
}
Favorites System
The favorites system uses a junction table with unique constraint to prevent duplicates:

sql
CREATE TABLE user_favorites (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    property_id INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY unique_favorite (user_id, property_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (property_id) REFERENCES properties(id) ON DELETE CASCADE
);
Toggle functionality via AJAX:

javascript
function toggleFavorite(propertyId, button) {
    fetch('toggle-favorite.php', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
        },
        body: 'property_id=' + propertyId
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            if (data.action === 'added') {
                button.classList.add('active');
                showNotification('Added to favorites');
            } else {
                button.classList.remove('active');
                showNotification('Removed from favorites');
            }
        }
    });
}
Responsive Design Approach
CSS Grid and Flexbox with mobile-first approach:

css
/* Mobile first (base styles) */
.properties-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 1.5rem;
}

/* Tablet */
@media (min-width: 768px) {
    .properties-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* Desktop */
@media (min-width: 1024px) {
    .properties-grid {
        grid-template-columns: repeat(3, 1fr);
    }
}

/* Large desktop */
@media (min-width: 1280px) {
    .properties-grid {
        grid-template-columns: repeat(4, 1fr);
    }
}
Security Measures
SQL Injection Prevention:

All database queries use prepared statements with PDO

No dynamic table/column names in queries

Input validation before database operations

XSS Prevention:

htmlspecialchars() on all user output

Content Security Policy headers

HTTP-only cookies for session

File Upload Security:

File type validation by MIME, not extension

Maximum file size limits

Store uploads outside web root when possible

Random filename generation

Authentication Security:

Password hashing with bcrypt

Session regeneration after login

Remember me tokens stored hashed

Login attempt limiting

HTTPS enforcement

Performance Optimizations
Database:

Indexes on frequently queried columns

Limited result sets with pagination

Avoid SELECT * in production queries

Frontend:

Optimized images (compression, proper sizes)

Minified CSS and JavaScript

Lazy loading for images below the fold

Deferred JavaScript loading

Caching:

Browser caching headers for static assets

Session data in files (not database)

Property counts cached for dashboard