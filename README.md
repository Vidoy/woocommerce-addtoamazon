# woocommerce-addtoamazon


/*

 *** "Buy on Amazon" Wordpress WooCommerce plugin ***
 
 This "plugin" changes the "Add to Cart" button to "Buy on Amazon" and opens the link
 to an amazon product url, also works with affiliate links. It will open the URL in a 
 new page or tab.

 I'm not learning the entire structure of WooCommerce and making an entire module and
 updating it each version just for these few lines, you can simply implement this by 
 choosing WooCommerce in your Wordpress Pludin Editor. Simply replace the code below 
 as instructed.
 
 I hereby grant you right to edit, sell, sublience... do whatever you want with this
 code, make nuclear bombs with it for all I care.
 
 This was made for WooCommerce 4.7.1, but should work for a long time unless they
 drastically alter the code. You may need to reapply the code if you update 
 WooCommerce.
 
 
*/
 
 
 ** How to use the plugin **
 
 - Add a new or edit a product in the "Products" section of your WordPress with
 WooCommerce.
 
 - Scroll down to "Product Data" and "Attributes". Choose "custom product attribute" and
 click "Add".
 
 - In "Name:" type "aurl" (without quotes) and paste your amazon affiliate link under
 "Value(s):", then uncheck "Visible on product page".
 
 - Click "Save attributes", save the product, and your product should now have a "Buy on
 Amazon" button instead of "Add to cart". 
 
 - If you don't add this attribute, the normal button will appear instead.
 
 
 ** How to "install" the plugin **
 
 - In your WordPress admin panel, choose "Plugin Editor" under "Plugins" in the menu.
 
 - Choose WooCommerce under "Select plugin to edit" on the right.
 
 - Change the files as stated below.
 
 
 --- Go to the file templates/single-product/add-to-cart/simple.php
 
 --- Find the code (~ on line 32)
 
	<form class="cart" action="<?php echo esc_url( apply_filters( 'woocommerce_add_to_cart_form_action', $product->get_permalink() ) ); ?>" method="post" enctype='multipart/form-data'>
		<?php do_action( 'woocommerce_before_add_to_cart_button' ); ?>

		<?php
		do_action( 'woocommerce_before_add_to_cart_quantity' );

		woocommerce_quantity_input(
			array(
				'min_value'   => apply_filters( 'woocommerce_quantity_input_min', $product->get_min_purchase_quantity(), $product ),
				'max_value'   => apply_filters( 'woocommerce_quantity_input_max', $product->get_max_purchase_quantity(), $product ),
				'input_value' => isset( $_POST['quantity'] ) ? wc_stock_amount( wp_unslash( $_POST['quantity'] ) ) : $product->get_min_purchase_quantity(), // WPCS: CSRF ok, input var ok.
			)
		);

		do_action( 'woocommerce_after_add_to_cart_quantity' );
		?>

		<button type="submit" name="add-to-cart" value="<?php echo esc_attr( $product->get_id() ); ?>" class="single_add_to_cart_button button alt"><?php echo esc_html( $product->single_add_to_cart_text() ); ?></button>

		<?php do_action( 'woocommerce_after_add_to_cart_button' ); ?>
	</form>

		
 --- Replace it with
 
	<form class="cart" action="<?php echo esc_url( apply_filters( 'woocommerce_add_to_cart_form_action', $product->get_permalink() ) ); ?>" method="post" enctype='multipart/form-data'>

		<?php
		$aurl = $product->get_attribute("aurl");
		if ($aurl) 
		{ ?>
		<a class="more-link" href="<?php echo $aurl; ?>" target="_blank">Buy on Amazon</a>
		<?php 
		} 
		else 
		{ 
		do_action( 'woocommerce_before_add_to_cart_button' );


		do_action( 'woocommerce_before_add_to_cart_quantity' );

		woocommerce_quantity_input(
			array(
				'min_value'   => apply_filters( 'woocommerce_quantity_input_min', $product->get_min_purchase_quantity(), $product ),
				'max_value'   => apply_filters( 'woocommerce_quantity_input_max', $product->get_max_purchase_quantity(), $product ),
				'input_value' => isset( $_POST['quantity'] ) ? wc_stock_amount( wp_unslash( $_POST['quantity'] ) ) : $product->get_min_purchase_quantity(), // WPCS: CSRF ok, input var ok.
			)
		);

		do_action( 'woocommerce_after_add_to_cart_quantity' );

		?>
		<button type="submit" name="add-to-cart" value="<?php echo esc_attr( $product->get_id() ); ?>" class="single_add_to_cart_button button alt"><?php echo esc_html( $product->single_add_to_cart_text() ); ?></button>
		<?php } ?>

		<?php do_action( 'woocommerce_after_add_to_cart_button' ); ?>
	</form>
	
	
--- Go to the file templates/loop/add-to-cart.php

--- Find the code (~ on line 24)

echo apply_filters(
	'woocommerce_loop_add_to_cart_link', // WPCS: XSS ok.
	sprintf(
		'<a href="%s" data-quantity="%s" class="%s" %s>%s</a>',
		esc_url( $product->add_to_cart_url() ),
		esc_attr( isset( $args['quantity'] ) ? $args['quantity'] : 1 ),
		esc_attr( isset( $args['class'] ) ? $args['class'] : 'button' ),
		isset( $args['attributes'] ) ? wc_implode_html_attributes( $args['attributes'] ) : '',
		esc_html( $product->add_to_cart_text() )
	),
	$product,
	$args
);


--- Replace it with 

$aurl = $product->get_attribute("aurl");

if($aurl) {
	echo apply_filters('woocommerce_loop_add_to_cart_link','<a class="more-link" href="'.$aurl.'" target="_blank">Buy on Amazon</a>');
}
else {
echo apply_filters(
	'woocommerce_loop_add_to_cart_link', // WPCS: XSS ok.
	sprintf(
		'<a href="%s" data-quantity="%s" class="%s" %s>%s</a>',
		esc_url( $product->add_to_cart_url() ),
		esc_attr( isset( $args['quantity'] ) ? $args['quantity'] : 1 ),
		esc_attr( isset( $args['class'] ) ? $args['class'] : 'button' ),
		isset( $args['attributes'] ) ? wc_implode_html_attributes( $args['attributes'] ) : '',
		esc_html( $product->add_to_cart_text() )
	),
	$product,
	$args
);
}

--- You're done, unless you want the optional code for pricing.
--- Optional code for pricing; Adds "Retails from" in front of the price. As you need to 
 update the price mannually, it may differ from Amazon.
 

--- Go to the file templates/single-product/price.php

--- Find the code

<p class="<?php echo esc_attr( apply_filters( 'woocommerce_product_price_class', 'price' ) ); ?>"><?php echo $product->get_price_html(); ?></p>

--- Replace it with

<?php

$aurl = $product->get_attribute("aurl");

if($aurl) {
?><p class="<?php echo esc_attr( apply_filters( 'woocommerce_product_price_class', 'price' ) ); ?>">Retails from <?php echo $product->get_price_html(); ?></p><?php
}
else {
?><p class="<?php echo esc_attr( apply_filters( 'woocommerce_product_price_class', 'price' ) ); ?>"><?php echo $product->get_price_html(); ?></p><?php
}


--- Go to the file templates/loop/price.php

--- Find the code

<?php if ( $price_html = $product->get_price_html() ) : ?>
	<span class="price"><?php echo $price_html; ?></span>
<?php endif; ?>


--- Replace it with


<?php
$aurl = $product->get_attribute("aurl");

if($aurl) {

if ( $price_html = $product->get_price_html() ) : ?>
	<span class="price">Retails from <?php echo $price_html; ?></span>
<?php endif; 

}
else {
if ( $price_html = $product->get_price_html() ) : ?>
	<span class="price"><?php echo $price_html; ?></span>
<?php endif;
}


/*

 You're done. Good luck out there!
 
*/
