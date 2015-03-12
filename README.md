# ToolStack WP Utilities

Contributors: Greg Ross
URI: http://toolstack.com  
License: GPLv2  

## Description ##

A utility class of functions for use in WordPress plugins.  

# License #
	
This code is released under the GPL v2, see license.txt for details.

## Documentation ##

Have you written a WordPress plugin before?  More than one?  Found yourself rewriting the same code each time you do?

Me to.  So here's a library of functions I found myself rewriting each time, extracted to it's own class and made available for everyone!

At the moment the library covers several broad catagroies:

- Plugin Options
- User Options
- Debugging
- Options Display

This list will grow over time.

### Installation ###

The first thing you need to do is download the library and put it in your plugin directory.  Once you have that done you'll need to include it in your source code.

To do so, add the following lines:

	include_once( 'ToolStack-WP-Utilities.class.php' );
	$TSU = new ToolStack_WP_Utilities_V2_3( 'my_plugin_slug' );
	
This should be done early in your plugin so you have access to the utilities as soon as possible.

There are three items to note in the above code:

1. The class name includes the version number.  This is done to avoid version conflicts as others will have included the class in their plugins as well.
2. 'my_plugin_slug' is used in several places inside the code and should be unique, contain no spaces and not be version dependent.  If it is left blank, the plugin install directory will be used.
3. $TSU should be replaced with a unique vairable name for your plugin.

### Class creation ###

During the class creation there are only two things done:

1. The plugin slug is determined.
2. The plugin options are loaded from the WordPress options table.

The plugin slug is used as the key name for the options for the plugin.

### Plugin Options ###

WordPress has a built in set of functions for handling options (get_option, update_option, etc.) and they work well.  However once you get beyond a certain point, the number of options in a plugin can become unwieldly with the format that WordPress gives you.

Instead, I (like many other plugin authors) use a single option for my plugins which stores an array.  Managing this array can be cumbersome and so the ToolStack WP Utilities implements several functions to simplify the process:

- load_options()
- get_option( $option, $default )
- update_option( $option, $value ) 
- store_option( $option, $value ) 
- save_options()
- isset_option()

The first three mimic the built in WordPress functions and are drop in replacements, so you can simply add $TSU-> to the front of the WordPress functions and everything will work as expected.

The difference is that instead of saving the option as a unqiue option in the WordPress options table, the option will be stored as the key name or an array stored as the plugin slug.

For example, if you use the WordPress function like this:

	update_option( 'my_plugin_setting_one', true );
	update_option( 'my_plugin_setting_two', false );
	
You would have a two rows in the wp_options table one with the option name being 'my_plugin_setting_one' and the second with the option name bieng 'my_plugin_setting_two'.

Using the library instead:

	$TSU->update_option( 'setting_one', true );
	$TSU->update_option( 'setting_two', false );

You will have a single row in the WordPress options table with the option name being 'my_plugin_slug' (from when you created the $TSU object), the option value will be an array with two items, setting_one and setting_two.

The last three functions provide additional functionality:

- store_option(): update_option() will immediately write out the new array to the database, however if your updating a lot of settings sequentially you may not want to do this so store_option() will update the value in memory but not write it to the database.
- save_options(): Used in conjunction with store_option(), once you are ready, save_options() will write out the plugin options to the database.
- isset_option(): Direct access to the options array is not a good idea so there needs to be a way to tell if an option has been set.  This function will return work just like isset() but for the assoiated option name.

### User Options ###

WordPress has some built in functions for handling user settings just like it does for global options and it has the same issues with them as well.

This set of functions takes the same approach as the plugin option for user options:

- set_user_id( $id )
- load_user_options( $user_id )
- get_user_option( $option, $default )
- update_user_option( $option, $value ) 
- get_the_author_meta( $option, $user_id )
- update_user_meta( $user_id, $option, $default )
- store_user_option( $option, $value ) 
- save_user_options() 
- isset_user_option( $option ) 

The user option code is a little more complex than the plugin option code.  This has a few sources:

- You need to load user options later in your plugin, after the user had been authenticated with WordPress.
- You may want to load more than one set of user options

The first two functions handle the loading of user options:

- set_user_id(): This will set the current user to get options for to the passed in WordPress user id.
- load_user_options(): This will load the user options in to memory.  If no user id is set or passed in, the current user will be the default.

The next two functions mimic the plugin options functions above:

- get_user_option(): This will retrieve the option setting for the user.
- update_user_option(): Ths will update the option setting for the user.

The next two functions are drop in replacements for the WordPress equivalents, but of course use the array instead of individual settings:

- get_the_author_meta( $option, $user_id )
- update_user_meta( $user_id, $option, $default )

And the final three accomplish the same task as their plugin options counterparts, see above for details:

- store_user_option() 
- save_user_options() 
- isset_user_option() 

### Debugging ###

One of the challenges with WordPress is getting good debugging information out of the system, the help out there are several functions available:

- print_r_html( $var, $string )
- var_dump_html()
- var_export_html( $var, $string = false )
- set_line_type( $type )
- set_debug_log( $file )
- open_debug_log()
- write_debug_log( $text )
- write_debug_log_var( $var )
- close_debug_log() 

The first three functions mimic the functionality of their PHP counterparts with the exception that they output html formatted text instead of plain text.

The rest of the functions all relate to writting a debug log for your plugin:

- set_line_type(): By default the library uses Unix style line terminiation (\n), but you can set it to anything you want with this function.
- set_debug_log( $file ): By default the debug log will be stored in sys_get_temp_dir() . '/debug.txt'.  This lets you set a different location and filename.
- open_debug_log(): You probably don't need to ever call this function as the first call to any of the writing functions will call this function for you.  However if for some reason you do, this will open the debug log for appending.
- write_debug_log( $text ): Write out a line of text to the debug log, it will be formated as: [YYYY-MM-DD HH:MM:SS] Text
- write_debug_log_var( $var ): Write out a variable to the debug log, this will indent it correctly to align past the timestamp.
- close_debug_log(): Another function you probably won't use as the debug log is closed when the class is destroyed.

### Options Display ###

A common task in plugins is to display a series of options in a table.  Hand coding the html is doable and on frequently used pages probably even preferable, but kind of a pain, so this funciton automates the task:

- generate_options_table( $options )

At the moment supported option types are:

- Title: An H3 formatted row that spans the entire table.
- Description: A span value above the setting with the "description" class.
- Boolean: A checkbox.
- Image: An image link (same as text at the moment).
- Hidden: This adds a hidden input tag.
- Select: A select with options.
- Static: A static text line.
- Text: A text box, either a simple one line input or a text area.

$options is an array or arrays as follows:

	type: They type (all lower case). 
	desc: The description to display to the left of the field.
	size: The size (width) of the field.
	setting: The setting to set as the default value.
	option_list: For select's this is the option list, this is a single text line that will be placed between the <select> and </select> tags.
	height: For text fields only, this will use a textarea if set to a value greater than 1.
	post: for text fields this will display some extra html/text after the input box (for showing defaults, etc.).

Some examples
	
	$options['id'] 				= array( 'type' => 'static', 'desc' => 'Player ID', 'size' => 10, 'setting' => 'Greg' );
	$options['email']			= array( 'type' => 'text', 'desc' => 'E-Mail', 'setting' => $user_email) );
	$options['hidden']			= array( 'type' => 'bool', 'desc' => 'Hidden', 'setting' => $hidden) );

	$TSU->generate_options_table( $options );

## Frequently Asked Questions ##

# None #

To be completed.

