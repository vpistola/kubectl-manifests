// Disable wp login
function custom_login_page() {
    $new_login_page_url = home_url( '/login/' ); // new login page
    global $pagenow;
    if( $pagenow == "wp-login.php" && $_SERVER['REQUEST_METHOD'] == 'GET') {
        wp_redirect($new_login_page_url);
        exit;
    }
}
if(!is_user_logged_in()){
    add_action('init','custom_login_page');
}


function wpb_login_logo() { ?>
    <style type="text/css">
        #login h1 a, .login h1 a {
            background-image: url(bloginfo('template_url') . '/images/agro4-invs-8-1-150x85.png');
        height:100px;
        width:300px;
        background-size: 300px 100px;
        background-repeat: no-repeat;
        padding-bottom: 10px;
        }
    </style>
<?php }
add_action( 'login_enqueue_scripts', 'wpb_login_logo' );