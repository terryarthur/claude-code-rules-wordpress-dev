# WordPress Development Instructions for Claude Code

## WordPress Coding Standards Implementation

### PHP Code Standards

#### Function and Class Naming
```php
// CORRECT: Use prefixes and snake_case for functions
function my_plugin_sanitize_input( $input ) {
    return sanitize_text_field( $input );
}

// CORRECT: Use PascalCase for classes with prefix
class My_Plugin_Admin {
    // Class content
}

// INCORRECT: No prefix, wrong case
function sanitizeInput( $input ) {
    return $input;
}
```

#### Variable and Constant Standards
```php
// CORRECT: Use snake_case for variables
$user_data = get_user_data();
$post_meta_key = '_my_plugin_setting';

// CORRECT: Use UPPER_SNAKE_CASE for constants with prefix
define( 'MY_PLUGIN_VERSION', '1.0.0' );
define( 'MY_PLUGIN_PLUGIN_DIR', plugin_dir_path( __FILE__ ) );

// INCORRECT: CamelCase variables, no prefix
$userData = getUserData();
define( 'VERSION', '1.0.0' );
```

#### File Header Requirements
```php
<?php
/**
 * Plugin Name: My Plugin Name
 * Description: Plugin description.
 * Version: 1.0.0
 * Author: Your Name
 * Text Domain: my-plugin
 * Domain Path: /languages
 */

// Prevent direct access
if ( ! defined( 'ABSPATH' ) ) {
    exit;
}
```

#### Indentation and Formatting
- Use tabs for indentation (not spaces)
- Opening braces on same line for functions and classes
- Space after control structures: `if (`, `foreach (`, `while (`
- No trailing whitespace on any lines

```php
// CORRECT formatting
if ( $condition ) {
    foreach ( $items as $item ) {
        do_something( $item );
    }
}

class My_Class {
    public function my_method() {
        // Method content
    }
}
```

### WordPress-Specific Coding Patterns

#### Hook Implementation
```php
// CORRECT: Use WordPress hooks properly
add_action( 'init', 'my_plugin_init' );
add_filter( 'the_content', 'my_plugin_modify_content' );

// CORRECT: Hook into WordPress admin
add_action( 'admin_menu', 'my_plugin_admin_menu' );
add_action( 'admin_enqueue_scripts', 'my_plugin_admin_scripts' );

// INCORRECT: Direct execution without hooks
my_plugin_init(); // Never call directly
```

#### Enqueue Scripts and Styles
```php
// CORRECT: Proper enqueuing with dependencies and versioning
function my_plugin_enqueue_scripts() {
    wp_enqueue_script(
        'my-plugin-admin',
        plugin_dir_url( __FILE__ ) . 'assets/js/admin.js',
        array( 'jquery' ),
        MY_PLUGIN_VERSION,
        true
    );
    
    // Localize script for AJAX
    wp_localize_script(
        'my-plugin-admin',
        'my_plugin_ajax',
        array(
            'ajax_url' => admin_url( 'admin-ajax.php' ),
            'nonce'    => wp_create_nonce( 'my_plugin_nonce' ),
        )
    );
}
add_action( 'admin_enqueue_scripts', 'my_plugin_enqueue_scripts' );
```

## Security Implementation Requirements

### Input Sanitization Patterns

#### Form Data Sanitization
```php
// CORRECT: Always sanitize before processing
function my_plugin_process_form() {
    if ( ! isset( $_POST['my_plugin_nonce'] ) || 
         ! wp_verify_nonce( $_POST['my_plugin_nonce'], 'my_plugin_action' ) ) {
        wp_die( 'Security check failed' );
    }
    
    $text_field = sanitize_text_field( $_POST['text_field'] );
    $textarea   = sanitize_textarea_field( $_POST['textarea'] );
    $email      = sanitize_email( $_POST['email'] );
    $url        = sanitize_url( $_POST['url'] );
    $number     = absint( $_POST['number'] );
    
    // Process sanitized data...
}

// INCORRECT: Never use raw $_POST data
function bad_process_form() {
    $data = $_POST['data']; // NEVER DO THIS
    update_option( 'my_option', $data ); // VULNERABLE
}
```

#### Array and Complex Data Sanitization
```php
function my_plugin_sanitize_array( $input ) {
    $clean_input = array();
    
    if ( is_array( $input ) ) {
        foreach ( $input as $key => $value ) {
            $clean_key = sanitize_key( $key );
            
            if ( is_array( $value ) ) {
                $clean_input[ $clean_key ] = my_plugin_sanitize_array( $value );
            } else {
                $clean_input[ $clean_key ] = sanitize_text_field( $value );
            }
        }
    }
    
    return $clean_input;
}
```

### Output Escaping Requirements

#### HTML Output Escaping
```php
// CORRECT: Always escape output
echo '<h2>' . esc_html( $title ) . '</h2>';
echo '<a href="' . esc_url( $link ) . '">' . esc_html( $text ) . '</a>';
echo '<input type="text" value="' . esc_attr( $value ) . '" />';

// For allowed HTML content
echo wp_kses_post( $content );

// For custom allowed HTML
$allowed_html = array(
    'strong' => array(),
    'em'     => array(),
    'a'      => array(
        'href' => array(),
        'title' => array(),
    ),
);
echo wp_kses( $content, $allowed_html );

// INCORRECT: Raw output
echo $title; // NEVER DO THIS
echo $user_input; // VULNERABLE TO XSS
```

#### JavaScript and CSS Escaping
```php
// CORRECT: Escape for JavaScript
?>
<script>
var myData = <?php echo wp_json_encode( $data ); ?>;
var myString = '<?php echo esc_js( $string ); ?>';
</script>

<style>
.my-class {
    color: <?php echo esc_attr( $color ); ?>;
}
</style>
<?php

// INCORRECT: Unescaped JavaScript
?>
<script>
var myData = '<?php echo $data; ?>'; // VULNERABLE
</script>
```

### Nonce Implementation Requirements

#### Form Nonce Implementation
```php
// CORRECT: Add nonce to forms
function my_plugin_settings_form() {
    ?>
    <form method="post" action="">
        <?php wp_nonce_field( 'my_plugin_save_settings', 'my_plugin_settings_nonce' ); ?>
        
        <input type="text" name="setting_value" value="<?php echo esc_attr( get_option( 'my_setting' ) ); ?>" />
        <input type="submit" value="Save" />
    </form>
    <?php
}

// CORRECT: Verify nonce on processing
function my_plugin_process_settings() {
    if ( ! isset( $_POST['my_plugin_settings_nonce'] ) || 
         ! wp_verify_nonce( $_POST['my_plugin_settings_nonce'], 'my_plugin_save_settings' ) ) {
        wp_die( 'Security verification failed' );
    }
    
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( 'Insufficient permissions' );
    }
    
    // Process form...
}
```

#### AJAX Nonce Implementation
```php
// CORRECT: JavaScript AJAX with nonce
jQuery(document).ready(function($) {
    $('#my-button').click(function() {
        $.ajax({
            url: my_plugin_ajax.ajax_url,
            type: 'POST',
            data: {
                action: 'my_plugin_ajax_action',
                nonce: my_plugin_ajax.nonce,
                data: 'some_data'
            },
            success: function(response) {
                // Handle response
            }
        });
    });
});

// CORRECT: PHP AJAX handler with nonce verification
function my_plugin_ajax_handler() {
    check_ajax_referer( 'my_plugin_nonce', 'nonce' );
    
    if ( ! current_user_can( 'edit_posts' ) ) {
        wp_die( 'Insufficient permissions' );
    }
    
    $data = sanitize_text_field( $_POST['data'] );
    
    // Process AJAX request...
    
    wp_send_json_success( $result );
}
add_action( 'wp_ajax_my_plugin_ajax_action', 'my_plugin_ajax_handler' );
add_action( 'wp_ajax_nopriv_my_plugin_ajax_action', 'my_plugin_ajax_handler' );
```

### Database Security Requirements

#### Prepared Statements
```php
// CORRECT: Always use prepared statements
global $wpdb;

// Single value
$user_id = 123;
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE user_id = %d",
        $user_id
    )
);

// Multiple values
$status = 'active';
$date = '2023-01-01';
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT * FROM {$wpdb->prefix}my_table WHERE status = %s AND created_date > %s",
        $status,
        $date
    )
);

// INCORRECT: String concatenation
$query = "SELECT * FROM {$wpdb->prefix}my_table WHERE user_id = " . $user_id; // VULNERABLE
$results = $wpdb->get_results( $query );
```

#### Custom Table Operations
```php
// CORRECT: Safe table creation
function my_plugin_create_table() {
    global $wpdb;
    
    $table_name = $wpdb->prefix . 'my_plugin_data';
    
    $charset_collate = $wpdb->get_charset_collate();
    
    $sql = "CREATE TABLE $table_name (
        id mediumint(9) NOT NULL AUTO_INCREMENT,
        user_id bigint(20) NOT NULL,
        data_value text NOT NULL,
        created_at datetime DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (id),
        KEY user_id (user_id)
    ) $charset_collate;";
    
    require_once( ABSPATH . 'wp-admin/includes/upgrade.php' );
    dbDelta( $sql );
}
```

### Capability and Permission Checks

#### User Capability Verification
```php
// CORRECT: Always check capabilities before sensitive operations
function my_plugin_admin_action() {
    // Check nonce first
    if ( ! wp_verify_nonce( $_POST['nonce'], 'my_plugin_action' ) ) {
        wp_die( 'Security check failed' );
    }
    
    // Check user capabilities
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( 'You do not have sufficient permissions to access this page.' );
    }
    
    // Perform action...
}

// CORRECT: Different capabilities for different actions
function my_plugin_edit_post_meta() {
    $post_id = absint( $_POST['post_id'] );
    
    if ( ! current_user_can( 'edit_post', $post_id ) ) {
        wp_die( 'You cannot edit this post.' );
    }
    
    // Edit post meta...
}

// CORRECT: File upload capabilities
function my_plugin_handle_upload() {
    if ( ! current_user_can( 'upload_files' ) ) {
        wp_die( 'You do not have permission to upload files.' );
    }
    
    // Handle upload...
}
```

### File Upload Security

#### Secure File Upload Implementation
```php
function my_plugin_handle_file_upload() {
    // Security checks
    if ( ! wp_verify_nonce( $_POST['upload_nonce'], 'my_plugin_upload' ) ) {
        wp_die( 'Security verification failed' );
    }
    
    if ( ! current_user_can( 'upload_files' ) ) {
        wp_die( 'Insufficient permissions' );
    }
    
    if ( ! isset( $_FILES['file'] ) || $_FILES['file']['error'] !== UPLOAD_ERR_OK ) {
        wp_die( 'File upload error' );
    }
    
    // Validate file type and extension
    $file = $_FILES['file'];
    $filetype = wp_check_filetype_and_ext( $file['tmp_name'], $file['name'] );
    
    if ( ! $filetype['type'] || ! $filetype['ext'] ) {
        wp_die( 'Invalid file type' );
    }
    
    // Allowed file types
    $allowed_types = array( 'jpg', 'jpeg', 'png', 'gif', 'pdf', 'doc', 'docx' );
    if ( ! in_array( $filetype['ext'], $allowed_types, true ) ) {
        wp_die( 'File type not allowed' );
    }
    
    // Check file size (2MB limit)
    if ( $file['size'] > 2 * 1024 * 1024 ) {
        wp_die( 'File too large' );
    }
    
    // Use WordPress upload handling
    $upload_overrides = array( 'test_form' => false );
    $movefile = wp_handle_upload( $file, $upload_overrides );
    
    if ( $movefile && ! isset( $movefile['error'] ) ) {
        // File successfully uploaded
        $file_url = $movefile['url'];
        $file_path = $movefile['file'];
        
        // Store file information securely
        update_option( 'my_plugin_uploaded_file', array(
            'url'  => sanitize_url( $file_url ),
            'path' => sanitize_text_field( $file_path ),
            'user' => get_current_user_id(),
            'date' => current_time( 'mysql' ),
        ) );
    } else {
        wp_die( 'Upload failed: ' . $movefile['error'] );
    }
}
```

## Vulnerability Prevention Measures

### SQL Injection Prevention
```php
// ALWAYS use these patterns:

// 1. WordPress Query Functions (preferred)
$posts = get_posts( array(
    'post_type'   => 'product',
    'meta_key'    => 'price',
    'meta_value'  => $price,
    'numberposts' => 10,
) );

// 2. WP_Query with meta_query
$query = new WP_Query( array(
    'post_type'  => 'product',
    'meta_query' => array(
        array(
            'key'     => 'category',
            'value'   => sanitize_text_field( $_GET['category'] ),
            'compare' => '=',
        ),
    ),
) );

// 3. Prepared statements for custom queries
global $wpdb;
$results = $wpdb->get_results(
    $wpdb->prepare(
        "SELECT p.ID, p.post_title, pm.meta_value as price 
         FROM {$wpdb->posts} p 
         JOIN {$wpdb->postmeta} pm ON p.ID = pm.post_id 
         WHERE p.post_type = %s 
         AND pm.meta_key = %s 
         AND pm.meta_value BETWEEN %d AND %d",
        'product',
        'price',
        $min_price,
        $max_price
    )
);
```

### XSS Prevention Patterns
```php
// ALWAYS escape output based on context:

// HTML content
echo '<div class="message">' . esc_html( $message ) . '</div>';

// HTML attributes
echo '<input type="text" value="' . esc_attr( $value ) . '" class="' . esc_attr( $css_class ) . '">';

// URLs
echo '<a href="' . esc_url( $link ) . '">Click here</a>';

// JavaScript strings
echo '<script>var message = "' . esc_js( $message ) . '";</script>';

// Rich content with allowed HTML
$allowed_html = array(
    'p'      => array(),
    'strong' => array(),
    'em'     => array(),
    'a'      => array(
        'href'   => array(),
        'title'  => array(),
        'target' => array(),
    ),
    'img'    => array(
        'src'   => array(),
        'alt'   => array(),
        'class' => array(),
    ),
);
echo wp_kses( $rich_content, $allowed_html );
```

### CSRF Protection Implementation
```php
// ALWAYS implement CSRF protection for state-changing operations:

// 1. Form-based actions
function my_plugin_form_handler() {
    // Verify nonce
    if ( ! isset( $_POST['_wpnonce'] ) || ! wp_verify_nonce( $_POST['_wpnonce'], 'my_plugin_action' ) ) {
        wp_die( __( 'Are you sure you want to do this?', 'my-plugin' ), __( 'Security check', 'my-plugin' ), array( 'response' => 403 ) );
    }
    
    // Verify user capabilities
    if ( ! current_user_can( 'manage_options' ) ) {
        wp_die( __( 'You do not have sufficient permissions.', 'my-plugin' ) );
    }
    
    // Process action
}

// 2. URL-based actions
$action_url = wp_nonce_url(
    admin_url( 'admin.php?page=my-plugin&action=delete&item=' . $item_id ),
    'delete_item_' . $item_id,
    '_wpnonce'
);

// 3. AJAX actions
function my_plugin_ajax_handler() {
    // Use check_ajax_referer for AJAX
    check_ajax_referer( 'my_plugin_ajax_nonce', 'security' );
    
    // Additional capability check
    if ( ! current_user_can( 'edit_posts' ) ) {
        wp_send_json_error( 'Insufficient permissions' );
    }
    
    // Process AJAX request
    wp_send_json_success( $result );
}
```

### Path Traversal Prevention
```php
// ALWAYS validate and sanitize file paths:

function my_plugin_read_template( $template_name ) {
    // Sanitize template name
    $template_name = sanitize_file_name( $template_name );
    
    // Remove any directory traversal attempts
    $template_name = basename( $template_name );
    
    // Define allowed template directory
    $template_dir = plugin_dir_path( __FILE__ ) . 'templates/';
    
    // Build full path
    $template_path = $template_dir . $template_name . '.php';
    
    // Verify the file exists and is within allowed directory
    if ( ! file_exists( $template_path ) || strpos( realpath( $template_path ), realpath( $template_dir ) ) !== 0 ) {
        return false;
    }
    
    return $template_path;
}
```

## User Experience Implementation

### WordPress Admin Integration
```php
// CORRECT: Follow WordPress admin patterns
function my_plugin_admin_menu() {
    add_options_page(
        __( 'My Plugin Settings', 'my-plugin' ),
        __( 'My Plugin', 'my-plugin' ),
        'manage_options',
        'my-plugin-settings',
        'my_plugin_settings_page'
    );
}

function my_plugin_settings_page() {
    ?>
    <div class="wrap">
        <h1><?php echo esc_html( get_admin_page_title() ); ?></h1>
        
        <?php settings_errors(); ?>
        
        <form method="post" action="options.php">
            <?php
            settings_fields( 'my_plugin_settings' );
            do_settings_sections( 'my_plugin_settings' );
            submit_button();
            ?>
        </form>
    </div>
    <?php
}
```

### Responsive Design Requirements
```css
/* ALWAYS implement mobile-first responsive design */
.my-plugin-container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

/* Mobile styles (default) */
.my-plugin-grid {
    display: grid;
    grid-template-columns: 1fr;
    gap: 20px;
}

/* Tablet styles */
@media (min-width: 768px) {
    .my-plugin-grid {
        grid-template-columns: repeat(2, 1fr);
    }
}

/* Desktop styles */
@media (min-width: 1024px) {
    .my-plugin-grid {
        grid-template-columns: repeat(3, 1fr);
    }
}
```

### Accessibility Implementation
```php
// ALWAYS implement proper accessibility
function my_plugin_accessible_form() {
    ?>
    <form method="post" action="">
        <?php wp_nonce_field( 'my_plugin_form', 'my_plugin_nonce' ); ?>
        
        <fieldset>
            <legend><?php esc_html_e( 'User Information', 'my-plugin' ); ?></legend>
            
            <p>
                <label for="user_name"><?php esc_html_e( 'Name', 'my-plugin' ); ?> <span aria-label="<?php esc_attr_e( 'required', 'my-plugin' ); ?>">*</span></label>
                <input type="text" id="user_name" name="user_name" required aria-describedby="name-description" />
                <span id="name-description" class="description"><?php esc_html_e( 'Enter your full name', 'my-plugin' ); ?></span>
            </p>
            
            <p>
                <label for="user_email"><?php esc_html_e( 'Email', 'my-plugin' ); ?> <span aria-label="<?php esc_attr_e( 'required', 'my-plugin' ); ?>">*</span></label>
                <input type="email" id="user_email" name="user_email" required aria-describedby="email-description" />
                <span id="email-description" class="description"><?php esc_html_e( 'We will never share your email', 'my-plugin' ); ?></span>
            </p>
        </fieldset>
        
        <p>
            <input type="submit" value="<?php esc_attr_e( 'Submit Form', 'my-plugin' ); ?>" class="button-primary" />
        </p>
    </form>
    <?php
}
```

### Performance Optimization Requirements

#### Conditional Loading
```php
// ONLY load assets when needed
function my_plugin_conditional_assets() {
    // Only load admin assets on plugin pages
    $screen = get_current_screen();
    if ( $screen->id === 'settings_page_my-plugin-settings' ) {
        wp_enqueue_style( 'my-plugin-admin', plugin_dir_url( __FILE__ ) . 'assets/css/admin.css', array(), MY_PLUGIN_VERSION );
        wp_enqueue_script( 'my-plugin-admin', plugin_dir_url( __FILE__ ) . 'assets/js/admin.js', array( 'jquery' ), MY_PLUGIN_VERSION, true );
    }
}
add_action( 'admin_enqueue_scripts', 'my_plugin_conditional_assets' );

// Only load frontend assets when shortcode is present
function my_plugin_frontend_assets() {
    global $post;
    
    if ( is_a( $post, 'WP_Post' ) && has_shortcode( $post->post_content, 'my_plugin_shortcode' ) ) {
        wp_enqueue_style( 'my-plugin-frontend', plugin_dir_url( __FILE__ ) . 'assets/css/frontend.css', array(), MY_PLUGIN_VERSION );
        wp_enqueue_script( 'my-plugin-frontend', plugin_dir_url( __FILE__ ) . 'assets/js/frontend.js', array( 'jquery' ), MY_PLUGIN_VERSION, true );
    }
}
add_action( 'wp_enqueue_scripts', 'my_plugin_frontend_assets' );
```

#### Caching Implementation
```php
// ALWAYS implement caching for expensive operations
function my_plugin_get_expensive_data( $user_id ) {
    $cache_key = 'my_plugin_user_data_' . $user_id;
    $cached_data = get_transient( $cache_key );
    
    if ( false === $cached_data ) {
        // Expensive operation
        $cached_data = my_plugin_calculate_user_stats( $user_id );
        
        // Cache for 1 hour
        set_transient( $cache_key, $cached_data, HOUR_IN_SECONDS );
    }
    
    return $cached_data;
}

// Clear cache when relevant data changes
function my_plugin_clear_user_cache( $user_id ) {
    $cache_key = 'my_plugin_user_data_' . $user_id;
    delete_transient( $cache_key );
}
add_action( 'profile_update', 'my_plugin_clear_user_cache' );
```

## Internationalization Requirements

### Text Domain Implementation
```php
// ALWAYS use proper text domains and translation functions
function my_plugin_translatable_content() {
    $message = __( 'Welcome to My Plugin', 'my-plugin' );
    
    printf(
        /* translators: %s: user name */
        __( 'Hello, %s! You have new messages.', 'my-plugin' ),
        esc_html( $user_name )
    );
    
    $items_text = sprintf(
        /* translators: %d: number of items */
        _n( 'You have %d item', 'You have %d items', $item_count, 'my-plugin' ),
        number_format_i18n( $item_count )
    );
    
    // For escaped output
    echo '<h2>' . esc_html__( 'Settings Page', 'my-plugin' ) . '</h2>';
    echo '<input type="submit" value="' . esc_attr__( 'Save Changes', 'my-plugin' ) . '" />';
}

// Load text domain
function my_plugin_load_textdomain() {
    load_plugin_textdomain( 'my-plugin', false, dirname( plugin_basename( __FILE__ ) ) . '/languages/' );
}
add_action( 'plugins_loaded', 'my_plugin_load_textdomain' );
```

## Validation and Testing Procedures

### Pre-Commit Validation Checklist
1. **Run PHPCS with WordPress standards:**
   ```bash
   phpcs --standard=WordPress file.php
   ```

2. **Check for security issues:**
   - Verify all inputs are sanitized
   - Confirm all outputs are escaped
   - Validate nonce implementation
   - Check capability requirements

3. **Test functionality:**
   - Test with different user roles
   - Verify error handling
   - Check mobile responsiveness
   - Validate accessibility

4. **Performance check:**
   - Verify conditional loading
   - Check for unnecessary queries
   - Validate caching implementation

### Automated Security Scanning
```php
// Implement security headers
function my_plugin_security_headers() {
    if ( ! is_admin() ) {
        header( 'X-Content-Type-Options: nosniff' );
        header( 'X-Frame-Options: SAMEORIGIN' );
        header( 'X-XSS-Protection: 1; mode=block' );
    }
}
add_action( 'send_headers', 'my_plugin_security_headers' );
```

## Error Handling and Logging

### Proper Error Handling
```php
function my_plugin_safe_operation() {
    try {
        $result = my_plugin_risky_operation();
        
        if ( is_wp_error( $result ) ) {
            error_log( 'My Plugin Error: ' . $result->get_error_message() );
            return false;
        }
        
        return $result;
    } catch ( Exception $e ) {
        error_log( 'My Plugin Exception: ' . $e->getMessage() );
        return false;
    }
}

// Use WP_Error for WordPress-style error handling
function my_plugin_validate_input( $input ) {
    if ( empty( $input ) ) {
        return new WP_Error( 'empty_input', __( 'Input cannot be empty', 'my-plugin' ) );
    }
    
    if ( strlen( $input ) > 255 ) {
        return new WP_Error( 'input_too_long', __( 'Input is too long', 'my-plugin' ) );
    }
    
    return sanitize_text_field( $input );
}
```

---

**CRITICAL REMINDER**: Every single instruction in this document is mandatory. No exceptions will be made for convenience or speed of development. Security and code quality are non-negotiable requirements for all WordPress development using Claude Code.