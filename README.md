Settings Framework for ClassicPress
============================

The Settings Framework aims to take the pain out of creating settings pages for your plugins
by effectively creating a wrapper around the settings API and making it super simple to create and maintain
settings pages.

Setting Up Your Plugin
----------------------

1. Install the Settings Framework with `composer require erichk4/settings-framework`
2. Create a "settings" folder in your plugin root.
3. Create a settings file in your new "settings" folder (e.g. `settings-general.php`)

Now you can set up your plugin like:

```php
class SFTest {
	/**
	 * @var string
	 */
	private $plugin_path;

	/**
	 * @var WordPressSettingsFramework
	 */
	private $settings;

	/**
	 * WPSFTest constructor.
	 */
	function __construct() {
		$this->plugin_path = plugin_dir_path( __FILE__ );

		// Create a new SettingsFramework
		$this->settings = new SettingsFramework(
			$this->plugin_path . 'settings/settings-general.php', // the settings file
			'prefix_settings_general',                            // the option group
			$this );                                              // the caller object 
		
		// Add admin menu
		add_action( 'admin_menu', array( $this, 'add_settings_page' ), 20 );
		
		// Add an optional settings validation filter (recommended)
		add_filter( $this->settings->get_option_group() . '_settings_validate', array( &$this, 'validate_settings' ) );
	}

	/**
	 * Add settings page.
	 */
	function add_settings_page() {
		$this->settings->add_settings_page( array(
			'parent_slug' => 'woocommerce',
			'page_title'  => __( 'Page Title', 'text-domain' ),
			'menu_title'  => __( 'menu Title', 'text-domain' ),
			'capability'  => 'manage_woocommerce',
		) );
	}

	/**
	 * Validate settings.
	 * 
	 * @param $input
	 *
	 * @return mixed
	 */
	function validate_settings( $input ) {
		// Do your settings validation here
		// Same as $sanitize_callback from http://codex.wordpress.org/Function_Reference/register_setting
		return $input;
	}

	// ...
}
```

Your settings values can be accessed like so:

```php
// Get settings
$this->settings->get_settings();
```

This will get either the saved setting values, or the default values that you set in your settings file.

Or by getting individual settings:

```php
// Get individual setting
$setting = wpsf_get_setting( 'prefix_settings_general', 'general', 'text' );
```


The Settings Files
------------------

The settings files work by filling the global `$wpsf_settings` array with data in the following format:

```php
add_filter( 'wpsf_register_settings_prefix_settings_general', 'prefix_settings_general_settings' );

function prefix_settings_general_settings( $settings )
{

        // Tabs.
        $settings[ 'tabs' ] = array(
            array(
                'id'    => 'tab1',
                'title' => esc_html__( 'Tab1', 'domain' ),
            ),
            array(
                'id'    => 'tab2',
                'title' => esc_html__( 'Tab2', 'domain' ),
            ),
        );
	
	$settings[] = array(
	    'tab_id' => 'tab1', 
	    'section_id' => 'general', // The section ID (required)
	    'section_title' => 'General Settings', // The section title (required)
	    'section_description' => 'Some intro description about this section.', // The section description (optional)
	    'section_order' => 5, // The order of the section (required)
	    'fields' => array(
		array(
		    'id' => 'text',
		    'title' => 'Text',
		    'desc' => 'This is a description.',
		    'placeholder' => 'This is a placeholder.',
		    'type' => 'text',
		    'default' => 'This is the default value'
		),
		array(
		    'id' => 'select',
		    'title' => 'Select',
		    'desc' => 'This is a description.',
		    'type' => 'select',
		    'default' => 'green',
		    'choices' => array(
			'red' => 'Red',
			'green' => 'Green',
			'blue' => 'Blue'
		    )
		),

		// add as many fields as you need...

	    )
	);
	
	return $settings;
    
}


```

Valid `fields` values are:

* `id` - Field ID
* `title` - Field title
* `desc` - Field description
* `placeholder` - Field placeholder
* `type` - Field type (text/password/textarea/select/radio/checkbox/checkboxes/color/file/editor/code_editor)
* `default` - Default value (or selected option)
* `choices` - Array of options (for select/radio/checkboxes)
* `mimetype` - Any valid mime type accepted by Code Mirror for syntax highlighting (for code_editor)

See `settings/example-settings.php` for an example of possible values.


API Details
-----------

    new WordPressSettingsFramework( string $settings_file [, string $option_group = ''] )

Creates a new settings [option_group](http://codex.wordpress.org/Function_Reference/register_setting) based on a setttings file.

* `$settings_file` - path to the settings file
* `$option_group` - optional "option_group" override (by default this will be set to the basename of the settings file)

<pre>wpsf_get_setting( $option_group, $section_id, $field_id )</pre>

Get a setting from an option group

* `$option_group` - option group id.
* `$section_id` - section id (change to `[{$tab_id}_{$section_id}]` when using tabs.
* `$field_id` - field id.

<pre>wpsf_delete_settings( $option_group )</pre>

Delete all the saved settings from a option group

* `$option_group` - option group id

Actions & Filters
---------------

**Filters**

* `wpsf_register_settings_[option_group]` - The filter used to register your settings. See `settings/example-settings.php` for an example.
* `[option_group]_settings_validate` - Basically the `$sanitize_callback` from [register_setting](http://codex.wordpress.org/Function_Reference/register_setting). Use `$wpsf->get_option_group()` to get the option group id.
* `wpsf_defaults_[option_group]` - Default args for a settings field

**Actions**

* `wpsf_before_field_[option_group]` - Before a field HTML is output
* `wpsf_before_field_[option_group]_[field_id]` - Before a field HTML is output
* `wpsf_after_field_[option_group]` - After a field HTML is output
* `wpsf_after_field_[option_group]_[field_id]` - After a field HTML is output
* `wpsf_before_settings_[option_group]` - Before settings form HTML is output
* `wpsf_after_settings_[option_group]` - After settings form HTML is output
* `wpsf_before_settings_fields_[option_group]` - Before settings form fields HTML is output (inside the `<form>`)
* `wpsf_do_settings_sections_[option_group]` - Settings form fields HTMLoutput (inside the `<form>`)
* `wpsf_do_settings_sections_[option_group]` - Settings form fields HTMLoutput (inside the `<form>`)
* `wpsf_before_tab_links_[option_group]` - Before tabs HTML is output
* `wpsf_after_tab_links_[option_group]` - After tabs HTML is output

Credits
-------

The WordPress Settings Framework was created by [Gilbert Pellegrom](http://gilbert.pellegrom.me) from [Dev7studios](http://dev7studios.com) and maintained by [James Kemp](https://jckemp.com) from [Iconic](https://iconicwp.com)

Please contribute by [reporting bugs](https://github.com/jamesckemp/WordPress-Settings-Framework/issues) and submitting [pull requests](https://github.com/jamesckemp/WordPress-Settings-Framework/pulls).

Want to say thanks? [Consider tipping me](https://www.paypal.me/jamesckemp).
