# How to convert a phpBB 3.0 MOD into a phpBB 3.1 Extension


This guide should give a quick overview of the needed tasks to MOD-Authors for converting a phpBB 3.0 MOD to a phpBB 3.1 Extension, using NV Newspage as an example.

## Extension Structure

The most obvious change should be the location the MODs/Extensions are stored in 3.1. In phpBB 3.0 all files were put into the core's root folder. In version 3.1 a special directory for Extensions has been created. It's called **ext/**.

### Directory

Each extension has its own directory. However, you can (and should) also use an additional vendor directory (with your author name or author-group name). In case of my newspage the files will be in

> phpBB/ext/nickvergessen/newspage/

There should not be a need to have files located outside of that direcotiry. No matter which files, may it be styles, language or ACP module files. All of them will be moved into your extension's directory.

			new directory			| current directory
			------------------------+----------------
	.../newspage/					|
			acp/					| phpBB/includes/acp/
									| phpBB/includes/acp/info/
			adm/style/				| phpBB/adm/style/
			config/					|	---
			controller/				|	---
			event/					|	---
			language/				| phpBB/language/
			migration/				|	---
			styles/					| phpBB/styles/

Newly added, additional directories have already been listed. Their use will be explained in the following paragraphs.

### Front-facing files, routes and services

While in 3.0 you just created a new file in the root directory of phpBB, you might want to use the new controller system of 3.1 in future. Your links change from something like `phpBB/newspage.php` to `phpBB/app.php?controller=newspage` in first place, but with a little htaccess rule this can be rewritten to `phpBB/newspage`

In order to link a specific routing rule to your extension, you need to define the route in your extension's **config/routing.yml**

For the easy start of the newspage, 2 rules are enough. The first rule is for the basic page currently `newspage.php`, the second one is for the pagination, like `newspage.php?start=5`. The first rule sets a default page (1), while the second rule requires a second part of the url to be an integer.


	newspage_base_controller:
	    pattern: /newspage/
	    defaults: { _controller: nickvergessen.newspage.controller:base, page: 1 }

	newspage_page_controller:
	    pattern: /newspage/{page}/
	    defaults: { _controller: nickvergessen.newspage.controller:base }
	    requirements:
	        page:  \d+

The string we define for `_controller` defines a service (`nickvergessen.newspage.controller`) and a method (`base`) of the class which is then called. Services are defined in your extensions **config/services.yml**. Services are instances of classes. Services are used, so there is only one instance of the class which is used all the time. You can also define the arguments for the constructor of your class. The example definition of the newspage controller service would be something similar to:

	services:
	    nickvergessen.newspage.controller:
	        class: phpbb_ext_nickvergessen_newspage_controller_main
	        arguments:
	            - @auth
	            - @cache
	            - @config
	            - @dbal.conn
	            - @request
	            - @template
	            - @user
	            - @controller.helper
	            - %core.root_path%
	            - %core.php_ext%

Any service that is previously defined in your file, or in the file of the phpBB core `phpBB/config/services.yml`, can also be used as an argument, aswell as some predefined string (like `core.root_path` here).

**NOTE:** The classes from phpBB/ext/ are automatically loaded by their names, whereby underscored ( _ ) can represent directories. In this case the class `phpbb_ext_nickvergessen_newspage_controller_main` would be located in `phpBB/ext/nickvergessen/newspage/controller/main.php`

For more explanations about [Routing](http://symfony.com/doc/2.1/book/routing.html) and  [Services](http://symfony.com/doc/2.1/book/service_container.html) see the Symfony 2.1 Documentation.

In this example my **controller/main.php** would look like the following:

	<?php
	
	/**
	*
	* @package NV Newspage Extension
	* @copyright (c) 2013 nickvergessen
	* @license http://opensource.org/licenses/gpl-2.0.php GNU General Public License v2
	*
	*/
	
	class phpbb_ext_nickvergessen_newspage_controller_main
	{
		/**
		* Constructor
		* NOTE: The parameters of this method must match in order and type with
		* the dependencies defined in the services.yml file for this service.
		*
		* @param phpbb_auth		$auth		Auth object
		* @param phpbb_cache_service	$cache		Cache object
		* @param phpbb_config	$config		Config object
		* @param phpbb_db_driver	$db		Database object
		* @param phpbb_request	$request	Request object
		* @param phpbb_template	$template	Template object
		* @param phpbb_user		$user		User object
		* @param phpbb_controller_helper		$helper		Controller helper object
		* @param string			$root_path	phpBB root path
		* @param string			$php_ext	phpEx
		*/
		public function __construct(phpbb_auth $auth, phpbb_cache_service $cache, phpbb_config $config, phpbb_db_driver $db, phpbb_request $request, phpbb_template $template, phpbb_user $user, phpbb_controller_helper $helper, $root_path, $php_ext)
		{
			$this->auth = $auth;
			$this->cache = $cache;
			$this->config = $config;
			$this->db = $db;
			$this->request = $request;
			$this->template = $template;
			$this->user = $user;
			$this->helper = $helper;
			$this->root_path = $root_path;
			$this->php_ext = $php_ext;
	
			if (!class_exists('bbcode'))
			{
				include($this->root_path . 'includes/bbcode.' . $this->php_ext);
			}
			if (!function_exists('get_user_rank'))
			{
				include($this->root_path . 'includes/functions_display.' . $this->php_ext);
			}
		}
	
		/**
		* Base controller to be accessed with the URL /newspage/{page}
		* (where {page} is the placeholder for a value)
		*
		* @param int	$page	Page number taken from the URL
		* @return Symfony\Component\HttpFoundation\Response A Symfony Response object
		*/
		public function base($page = 1)
		{
			/*
			* Do some magic here,
			* load your data and send it to the template.
			*/			

			/*
			* The render method takes up to three other arguments
			* @param	string		Name of the template file to display
			*						Template files are searched for two places:
			*						- phpBB/styles/<style_name>/template/
			*						- phpBB/ext/<all_active_extensions>/styles/<style_name>/template/
			* @param	string		Page title
			* @param	int			Status code of the page (200 - OK [ default ], 403 - Unauthorized, 404 - Page not found, etc.)
			*/
			return $this->helper->render('newspage_body.html');
		}
	}

You can also have multiple different methods in one controller aswell as having multiple controllers, in order to organize your code a bit better.

If we now add the entry for our extension into the phpbb_ext table, and go to `example.tld/app.php?controller=newspage/` you can see your template file. **Congratulations!** You just finished the "Hello World" example for phpBB Extensions. ;)

### ACP Modules

This section also applies to MCP and UCP modules.

As mentioned before these files are also moved into your extensions directory. The info-file, currently located in `phpBB/includes/acp/info/acp_newspage.php`, is going to be `ext/nickvergessen/newspage/acp/main_info.php` and the module itself is moved from `phpBB/includes/acp/acp_newspage.php` to `ext/nickvergessen/newspage/acp/main_module.php`. In order to be able to automatically load the files by their class names we need to make some little adjustments to the classes themselves.

As for the `main_info.php` I need to adjust the class name from `acp_newspage_info` to `phpbb_ext_nickvergessen_newspage_acp_main_info` and also change the value of `'filename'` in the returned array.

	<?php
	
	/**
	*
	* @package NV Newspage Extension
	* @copyright (c) 2013 nickvergessen
	* @license http://opensource.org/licenses/gpl-2.0.php GNU General Public License v2
	*
	*/
	
	/**
	* @ignore
	*/
	if (!defined('IN_PHPBB'))
	{
		exit;
	}
	
	class phpbb_ext_nickvergessen_newspage_acp_main_info
	{
		function module()
		{
			return array(
				'filename'	=> 'main_module',
				'title'		=> 'ACP_NEWSPAGE_TITLE',
				'version'	=> '1.0.1',
				'modes'		=> array(
					'config_newspage'	=> array('title' => 'ACP_NEWSPAGE_CONFIG', 'auth' => 'acl_a_board', 'cat' => array('ACP_NEWSPAGE_TITLE')),
				),
			);
		}
	}

In case of the module, I just adjust the class name:

	<?php
	
	/**
	*
	* @package NV Newspage Extension
	* @copyright (c) 2013 nickvergessen
	* @license http://opensource.org/licenses/gpl-2.0.php GNU General Public License v2
	*
	*/
	
	/**
	* @ignore
	*/
	if (!defined('IN_PHPBB'))
	{
		exit;
	}
	
	class phpbb_ext_nickvergessen_newspage_acp_main_module
	{
		var $u_action;
	
		function main($id, $mode)
		{
			// Your magic stuff here
		}
	}

And there you go. Your Extensions ACP module can now be added through the ACP and you just finished another step of successfully converting a MOD into an Extension.

## Include extension's language files

As the language files in your extension are not detected by the `$user->add_lang()` any more, you need to use the `$user->add_lang_ext()` method. This method takes two arguments, the first one is the fullname of the extension (including the vendor) and the second one is the file name or array of file names. so in order to load my newspage language file I now call

	$user->add_lang_ext('nickvergessen/newspage', 'newspage');

to load my language from `phpBB/ext/nickvergessen/newspage/language/en/newspage.php`

## File edits - Better don't edit anything, just use Events and Listeners

As for the newspage Modification, the only thing that is now missing from completion is the link in the header section, so you can start browsing the newspage.

In order to do this, I used to define the template variable in the `page_header()`-function of phpBB and then edit the `overall_header.html`. But this is 3.1 so we don't like file edits anymore and added **events** instead. With events you can hook into several places and execute your code, without editing them.

### php Events

So instead of adding

	$template->assign_vars(array(
		'U_NEWSPAGE'	=> append_sid($phpbb_root_path . 'app.' . $phpEx, 'controller=newspage/'),
	));

to the `page_header()`, we put that into an event listener, which is then called, everytime `page_header()` itself is called.

So we add the **event/main_listener.php** file to our extension, which implements some Symfony class:

	<?php
	
	/**
	*
	* @package NV Newspage Extension
	* @copyright (c) 2013 nickvergessen
	* @license http://opensource.org/licenses/gpl-2.0.php GNU General Public License v2
	*
	*/
	
	/**
	* @ignore
	*/
	
	if (!defined('IN_PHPBB'))
	{
		exit;
	}
	
	/**
	* Event listener
	*/
	use Symfony\Component\EventDispatcher\EventSubscriberInterface;
	
	class phpbb_ext_nickvergessen_newspage_event_main_listener implements EventSubscriberInterface
	{
	}

In the `getSubscribedEvents()` method we tell the system for which events we want to get notified and which function should be executed in case it's called. In our case we want to subscribe to the `core.page_header`-Event (a full list of events can be found [here](https://wiki.phpbb.com/Event_List) ):

		static public function getSubscribedEvents()
		{
			return array(
				'core.page_header'				=> 'add_page_header_link',
			);
		}

Now we add the function which is then called:

		public function add_page_header_link($event)
		{
			global $user, $template, $phpbb_root_path, $phpEx;
	
			// I use a second language file here, so I only load the strings global which are required globally.
			// This includes the name of the link, aswell as the ACP module names.
			$user->add_lang_ext('nickvergessen/newspage', 'newspage_global');

			$template->assign_vars(array(
				'U_NEWSPAGE'	=> append_sid($phpbb_root_path . 'app.' . $phpEx, 'controller=newspage/'),
			));
		}

and we are done with the php-editing. Your users will not get conflicts on searching for files blocks and other things because another MOD already edited the code. Again like with the controllers, you can have multiple listeners in the event/ directory, aswell as subscribe to multiple events with one listener.

### Template Event

Now the only thing left is, adding the code to the html output. For templates you need one file per event.

The filename thereby includes the event name. In order to add the newspage link next to the FAQ link, we need to use the `'overall_header_navigation_prepend'`-event (a full list of events can be found [here](https://wiki.phpbb.com/Event_List) ).

So we add the `styles/prosilver/template/events/overall_header_navigation_prepend_listener.html` to our extensions directory and add the html code into it.

	<li class="icon-newspage"><a href="{U_NEWSPAGE}">{L_NEWSPAGE}</a></li>

And that's it. No file edits required for the template files aswell.

### Adding Events

You can also add events to your extensions php and template code. If you miss an event from the core, please post a topic into the [[3.x] Event Requests](https://area51.phpbb.com/phpBB/viewforum.php?f=111)-Forum and we will include it for the next release.
We try to include a huge bunch of events by default, but surely we can not cover every place your MODs need to be covered.

### Basics finished!

And that's it, the 3.0 Modification was successfully converted into a 3.1 Extension.

## Compatibility

In some cases the compatibility of functions and classes count not be kept, while increasing their power. You can see a list of things in the Wiki-Article about [PhpBB3.1](https://wiki.phpbb.com/PhpBB3.1)

### Pagination

When you use your old 3.0 code you will receive an error like the following:

> Fatal error: Call to undefined function generate_pagination() in ...\phpBB3\ext\nickvergessen\newspage\controller\main.php on line 534

The problem is, that the pagination is now not returned by the function anymore, but instead automatically put into the template. In the same step, the function name was updated with a phpbb-prefix.

The old pagination code was similar to:

		$pagination = generate_pagination(append_sid("{$phpbb_root_path}app.$phpEx", 'controller=newspage/'), $total_paginated, $config['news_number'], $start);
	
		$this->template->assign_vars(array(
			'PAGINATION'		=> $pagination,
			'PAGE_NUMBER'		=> on_page($total_paginated, $config['news_number'], $start),
			'TOTAL_NEWS'		=> $this->user->lang('VIEW_TOPIC_POSTS', $total_paginated),
		));

The new code should look like:


		$base_url = append_sid("{$phpbb_root_path}app.$phpEx", 'controller=newspage/');
		phpbb_generate_template_pagination($this->template, $base_url, 'pagination', 'start', $total_paginated, $this->config['news_number'], $start);

		$this->template->assign_vars(array(
			'PAGE_NUMBER'		=> phpbb_on_page($this->template, $this->user, $base_url, $total_paginated, $this->config['news_number'], $start),
			'TOTAL_NEWS'		=> $this->user->lang('VIEW_TOPIC_POSTS', $total_paginated),
		));


