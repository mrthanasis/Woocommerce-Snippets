# Woocommerce-Snippets

1 – Add Payment Type to WooCommerce Admin Email
add_action( 'woocommerce_email_after_order_table', 'add_payment_method_to_admin_new_order', 15, 2 );
 
function add_payment_method_to_admin_new_order( $order, $is_admin_email ) {
  if ( $is_admin_email ) {
    echo '<p><strong>Payment Method:</strong> ' . $order->payment_method_title . '</p>';
  }
}
2 – Up-sells products per page / per line
remove_action( 'woocommerce_after_single_product_summary', 'woocommerce_upsell_display', 15 );
add_action( 'woocommerce_after_single_product_summary', 'woocommerce_output_upsells', 15 );
 
if ( ! function_exists( 'woocommerce_output_upsells' ) ) {
	function woocommerce_output_upsells() {
	    woocommerce_upsell_display( 3,3 ); // Display 3 products in rows of 3
	}
}
3 – Remove product categories from shop page
add_action( 'pre_get_posts', 'custom_pre_get_posts_query' );

function custom_pre_get_posts_query( $q ) {
 
	if ( ! $q->is_main_query() ) return;
	if ( ! $q->is_post_type_archive() ) return;
	
	if ( ! is_admin() && is_shop() && ! is_user_logged_in() ) {
 
		$q->set( 'tax_query', array(array(
			'taxonomy' => 'product_cat',
			'field' => 'slug',
			'terms' => array( 'color', 'flavor', 'spices', 'vanilla' ), // Don't display products in these categories on the shop page
			'operator' => 'NOT IN'
		)));
	
	}
 
	remove_action( 'pre_get_posts', 'custom_pre_get_posts_query' );
 
}
4 – Quickly translate any string
add_filter('gettext',  'translate_text');
add_filter('ngettext',  'translate_text');
 
function translate_text($translated) {
     $translated = str_ireplace('Choose and option',  'Select',  $translated);
     return $translated;
}
5 – Exclude a category from the WooCommerce category widget
add_filter( 'woocommerce_product_categories_widget_args', 'woo_product_cat_widget_args' );

function woo_product_cat_widget_args( $cat_args ) {
	
	$cat_args['exclude'] = array('16');
	
	return $cat_args;
}
6 – Add a custom field to a product variation
//Display Fields
add_action( 'woocommerce_product_after_variable_attributes', 'variable_fields', 10, 2 );
//JS to add fields for new variations
add_action( 'woocommerce_product_after_variable_attributes_js', 'variable_fields_js' );
//Save variation fields
add_action( 'woocommerce_process_product_meta_variable', 'variable_fields_process', 10, 1 );

function variable_fields( $loop, $variation_data ) { ?>	
	<tr>
		<td>
			<div>
					<label></label>
					<input type="text" size="5" name="my_custom_field[]" value=""/>
			</div>
		</td>
	</tr>

<tr>
		<td>
			<div>
					<label></label>
					
			</div>
		</td>
	</tr>
<?php }
 
function variable_fields_process( $post_id ) {
	if (isset( $_POST['variable_sku'] ) ) :
		$variable_sku = $_POST['variable_sku'];
		$variable_post_id = $_POST['variable_post_id'];
		$variable_custom_field = $_POST['my_custom_field'];
		for ( $i = 0; $i < sizeof( $variable_sku ); $i++ ) :
			$variation_id = (int) $variable_post_id[$i];
			if ( isset( $variable_custom_field[$i] ) ) {
				update_post_meta( $variation_id, '_my_custom_field', stripslashes( $variable_custom_field[$i] ) );
			}
		endfor;
	endif;
}
7 – Replace “Out of stock” by “sold”
add_filter('woocommerce_get_availability', 'availability_filter_func');

function availability_filter_func($availability)
{
	$availability['availability'] = str_ireplace('Out of stock', 'Sold', $availability['availability']);
	return $availability;
}
8 – Display “product already in cart” instead of “add to cart” button
/**
 * Change the add to cart text on single product pages
 */
add_filter( 'woocommerce_product_single_add_to_cart_text', 'woo_custom_cart_button_text' );

function woo_custom_cart_button_text() {

	global $woocommerce;
	
	foreach($woocommerce->cart->get_cart() as $cart_item_key => $values ) {
		$_product = $values['data'];
	
		if( get_the_ID() == $_product->id ) {
			return __('Already in cart - Add Again?', 'woocommerce');
		}
	}
	
	return __('Add to cart', 'woocommerce');
}

/**
 * Change the add to cart text on product archives
 */
add_filter( 'add_to_cart_text', 'woo_archive_custom_cart_button_text' );

function woo_archive_custom_cart_button_text() {

	global $woocommerce;
	
	foreach($woocommerce->cart->get_cart() as $cart_item_key => $values ) {
		$_product = $values['data'];
	
		if( get_the_ID() == $_product->id ) {
			return __('Already in cart', 'woocommerce');
		}
	}
	
	return __('Add to cart', 'woocommerce');
}
9 – Hide products count in category view
add_filter( 'woocommerce_subcategory_count_html', 'woo_remove_category_products_count' );

function woo_remove_category_products_count() {
	return;
}
10 – Make account checkout fields required
add_filter( 'woocommerce_checkout_fields', 'woo_filter_account_checkout_fields' );
 
function woo_filter_account_checkout_fields( $fields ) {
	$fields['account']['account_username']['required'] = true;
	$fields['account']['account_password']['required'] = true;
	$fields['account']['account_password-2']['required'] = true;

	return $fields;
}
11 – Rename a product tab
add_filter( 'woocommerce_product_tabs', 'woo_rename_tab', 98);
function woo_rename_tab($tabs) {

 $tabs['description']['title'] = 'More info';

 return $tabs;
}
12 – List WooCommerce product Categories
$args = array(
    'number'     => $number,
    'orderby'    => $orderby,
    'order'      => $order,
    'hide_empty' => $hide_empty,
    'include'    => $ids
);

$product_categories = get_terms( 'product_cat', $args );

$count = count($product_categories);
 if ( $count > 0 ){
     echo "<ul>";
     foreach ( $product_categories as $product_category ) {
       echo '<li><a href="' . get_term_link( $product_category ) . '">' . $product_category->name . '</li>';
        
     }
     echo "</ul>";
 }
13 – Replace shop page title
add_filter( 'woocommerce_page_title', 'woo_shop_page_title');

function woo_shop_page_title( $page_title ) {
	
	if( 'Shop' == $page_title) {
		return "My new title";
	}
}
14 – Change a widget title
/*
 * Change widget title
 */
add_filter( 'widget_title', 'woo_widget_title', 10, 3);

function woo_widget_title( $title, $instance, $id_base ) {
	
	if( 'onsale' == $id_base) {
		return "My new title";
	}
}
15 – Remove WooCommerce default settings
add_filter( 'woocommerce_catalog_settings', 'woo_remove_catalog_options' );

function woo_remove_catalog_options( $catalog ) {

	unset($catalog[23]); //Trim zeros (no) 
	unset($catalog[22]); //2 decimals 
	unset($catalog[21]); //decimal sep (.) 
	unset($catalog[20]); //thousand sep (,) 
	unset($catalog[19]); //currency position (left)	
	unset($catalog[18]); //currency position (left)	
	unset($catalog[5]); // ajax add to cart (no)	
	
	return $catalog; 
}
16 – Change “from” email address
function woo_custom_wp_mail_from() {
        global $woocommerce;
        return html_entity_decode( 'your@email.com' );
}
add_filter( 'wp_mail_from', 'woo_custom_wp_mail_from', 99 );
17 – Decode from name in WooCommerce email
function woo_custom_wp_mail_from_name() {
        global $woocommerce;
        return html_entity_decode( get_option( 'woocommerce_email_from_name' ) );
}
add_filter( 'wp_mail_from_name', 'woo_custom_wp_mail_from_name', 99 );

function woo_custom_wp_mail_from() {
        global $woocommerce;
        return html_entity_decode( get_option( 'woocommerce_email_from' ) );
}
add_filter( 'wp_mail_from_name', 'woo_custom_wp_mail_from_name', 99 );
18 – Return featured products ids
function woo_get_featured_product_ids() {
	// Load from cache
	$featured_product_ids = get_transient( 'wc_featured_products' );

	// Valid cache found
	if ( false !== $featured_product_ids )
		return $featured_product_ids;

	$featured = get_posts( array(
		'post_type'      => array( 'product', 'product_variation' ),
		'posts_per_page' => -1,
		'post_status'    => 'publish',
		'meta_query'     => array(
			array(
				'key' 		=> '_visibility',
				'value' 	=> array('catalog', 'visible'),
				'compare' 	=> 'IN'
			),
			array(
				'key' 	=> '_featured',
				'value' => 'yes'
			)
		),
		'fields' => 'id=>parent'
	) );

	$product_ids = array_keys( $featured );
	$parent_ids  = array_values( $featured );
	$featured_product_ids = array_unique( array_merge( $product_ids, $parent_ids ) );

	set_transient( 'wc_featured_products', $featured_product_ids );

	return $featured_product_ids;
}
19 – Add custom field to edit address page
// add fields to edit address page
function woo_add_edit_address_fields( $fields ) {

	$new_fields = array(
				'date_of_birth'     => array(
				'label'             => __( 'Date of birth', 'woocommerce' ),
				'required'          => false,
				'class'             => array( 'form-row' ),
			),
		);
		
	$fields = array_merge( $fields, $new_fields );
	
    return $fields;
	
}

add_filter( 'woocommerce_default_address_fields', 'woo_add_edit_address_fields' );
20 – Display onsale products catalog shortcode
function woocommerce_sale_products( $atts ) {

    global $woocommerce_loop;

    extract(shortcode_atts(array(
        'per_page'  => '12',
        'columns'   => '4',
        'orderby' => 'date',
        'order' => 'desc'
    ), $atts));

    $woocommerce_loop['columns'] = $columns;

    $args = array(
        'post_type' => 'product',
        'post_status' => 'publish',
        'ignore_sticky_posts'   => 1,
        'posts_per_page' => $per_page,
        'orderby' => $orderby,
        'order' => $order,
        'meta_query' => array(
            array(
                'key' => '_visibility',
                'value' => array('catalog', 'visible'),
                'compare' => 'IN'
            ),
            array(
                'key' => '_sale_price',
                'value' =>  0,
                'compare'   => '>',
                'type'      => 'NUMERIC'
            )
        )
    );
    query_posts($args);
    ob_start();
    woocommerce_get_template_part( 'loop', 'shop' );
    wp_reset_query();

    return ob_get_clean();
}

add_shortcode('sale_products', 'woocommerce_sale_products');
21 – Have onsale products
function woo_have_onsale_products() {
	
	global $woocommerce;

	// Get products on sale
	$product_ids_on_sale = array_filter( woocommerce_get_product_ids_on_sale() );

	if( !empty( $product_ids_on_sale ) ) {
		return true;
	} else {
		return false;
	}
	
}

// Example:
if( woo_have_onsale_products() ) {
	echo 'have onsale products';
} else {
	echo 'no onsale product';
}
22 – Set minimum order amount
add_action( 'woocommerce_checkout_process', 'wc_minimum_order_amount' );
function wc_minimum_order_amount() {
	global $woocommerce;
	$minimum = 50;
	if ( $woocommerce->cart->get_cart_total(); < $minimum ) {
           $woocommerce->add_error( sprintf( 'You must have an order with a minimum of %s to place your order.' , $minimum ) );
	}
}
23 – Order by price, date or title on shop page
add_filter('woocommerce_default_catalog_orderby', 'custom_default_catalog_orderby');
 
function custom_default_catalog_orderby() {
     return 'date'; // Can also use title and price
}
24 – Redirect add to cart button to checkout page
add_filter ('add_to_cart_redirect', 'redirect_to_checkout');

function redirect_to_checkout() {
    global $woocommerce;
    $checkout_url = $woocommerce->cart->get_checkout_url();
    return $checkout_url;
}
25 – Add email recipient when order completed
function woo_extra_email_recipient($recipient, $object) {
    $recipient = $recipient . ', your@email.com';
    return $recipient;
}
add_filter( 'woocommerce_email_recipient_customer_completed_order', 'woo_extra_email_recipient', 10, 2);
