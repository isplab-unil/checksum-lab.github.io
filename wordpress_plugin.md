# WordPress Plugin

## Installing Wordpress

WordPress provides detailed [installation instructions](https://codex.wordpress.org/Installing_WordPress).

## Installing the plugin

First you need to download the plugin.

```wget https://checksum-lab.github.io/wordpress_plugin.zip```

You then have to unzip the archive in the plugin directory of your WordPress installation.

```unzip wordpress_plugin.zip -d [WORDPRESS_DIR]/wp-content/plugins/```

As illustrated in the figure below, the Link Integrity plugin should now appear in the administration panel of WordPress. Simply click on the activate button to finalize the installation of the plugin.

![Wordpress Plugin](https://checksum-lab.github.io/wordpress_plugin.png)

## Creating a dowload link

The plugin automatically verifies every HTML links when pages and posts are saved. 
If the content mime type of the targetted resource corresponds to a downloadable resource, the resource is fetched, its hash is computed and added to the integrity attribute of the link.

For example, you can try to create a post with a link for downloading [Transmission](https://raw.githubusercontent.com/transmission/transmission-releases/master/Transmission-2.93.dmg).
When the wordpress plugin is enabled, saving the post takes more time because the server has to download Transmission and compute the checksum.

Once the post is saved, you can preview it in the browser. 
You can check in the source of the page that the HTML link indeed contains an integrity attribute.
Also, if the [Chrome extension](http://checksum-lab.github.io/chrome_extension.zip) is enabled, a popup will appear and the checksum will be verified automatically when clicking on the link.
