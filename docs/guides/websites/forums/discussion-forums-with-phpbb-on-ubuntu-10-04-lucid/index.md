---
slug: discussion-forums-with-phpbb-on-ubuntu-10-04-lucid
deprecated: true
author:
  name: Stan Schwertly
  email: docs@linode.com
description: 'Configuring the phpBB system for hosting web-based discussion forums on Ubuntu 10.04 (Lucid).'
keywords: ["phpBB", "forum software", "web applications", "PHP"]
tags: ["ubuntu"]
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
aliases: ['/websites/forums/discussion-forums-with-phpbb-on-ubuntu-10-04-lucid/','/web-applications/bulletin-boards/phpbb/ubuntu-10-04-lucid/']
modified: 2013-10-01
modified_by:
  name: Linode
published: 2010-09-22
title: 'Discussion Forums with phpBB on Ubuntu 10.04 (Lucid)'
relations:
    platform:
        key: how-to-install-phpbb
        keywords:
           - distribution: Ubuntu 10.04
---



phpBB is one of the most widely used open source forum solutions. It is easy to install and free to use, along with being fully customizable. If you don't want to spend money on other forum software like vBulletin, consider using phpBB.

For this guide, we'll assume you've already completed the [Setting Up and Securing a Compute Instance](/docs/guides/set-up-and-secure/) guide and have a working [LAMP stack](/docs/lamp-guides/) on your Linode. You should be connected to your server via SSH and logged in as root.

## Prerequisites

Before installing phpBB, make sure your system is up to date and install the `php5-gd` and `imagemagick` packages:

    apt-get update
    apt-get upgrade
    apt-get install php5-gd imagemagick

After you have installed the packages above, you will need to restart Apache. Issue the following command:

    /etc/init.d/apache2 restart

You are now ready to install phpBB!

## Downloading and Unpacking

We'll begin by downloading the latest release of phpBB. As of this writing, the latest stable release of phpBB is version 3.0.7. You can find the download for the latest link by visiting the [phpBB downloads](http://www.phpbb.com/downloads/) section of their website.

Use `wget` to download the latest package. You may need to install the wget command first by issuing `apt-get install wget`.

    wget http://d10xg45o6p6dbl.cloudfront.net/projects/p/phpbb/phpBB-3.0.7-PL1.tar.bz2

This will allow you to decompress the file you just downloaded. Using the following command, decompress the phpBB package:

    tar -jxvf phpBB-3.0.7-PL1.tar.bz2

Move the newly formed `phpBB3` folder into your `public_html` folder. This will make the location on your server `example.com/phpBB3` in the example provided by our LAMP guides. Feel free to rename the folder after you move it:

    mv phpBB3/ /srv/www/example.com/public_html/forum/
    cd /srv/www/example.com/public_html/forum/

This will move the PHP instance into a directory called `forum/`. You can move the location of the phpBB software into the top level of your `DocumentRoot`, in this case `public_html/` but phpBB can be installed in any folder you wish. Now, visit that directory in your browser. In the above example, that would be `http://example.com/forum/`. At this point, you will be guided through the rest of the configuration by phpBB's browser-driven install process.

## Configure phpBB

Before you can install phpBB, you need to make sure that it has access to write to the config file. Issue the following command to make sure that phpBB can write to `config.php`:

    chmod 0777 /srv/www/example.com/public_html/forum/config.php

Click the "Install" tab in the top left region of the page. You should be looking at a requirements page. If you followed the [LAMP guide](/docs/websites/apache/apache-2-web-server-on-ubuntu-10-04-lts-lucid/), your server will meet the requirements specified by the phpBB installation process. Click "Proceed to next step." The next page is simply a confirmation that your server meets the minimum installation requirements. Click "Start Install."

You will need to fill in your database information. Assuming you installed MySQL while following the LAMP guide, here you would fill in your `database name`, `database username`, and `database password`. The other fields can be left blank for the defaults. If you are unsure of what to put here, head over to the [MySQL database guides](/docs/databases/mysql/) to become familiar with MySQL and to create the database and username. Click "Proceed to next step."

This brings us to the "Administrator Configuration" page where you can set your Administrator username, password, and admin e-mail address. Fill this out, then click "Proceed to next step". Click "Proceed to next step" again on the next page confirming the settings worked. Click "Proceed to next step" again after you reach the next page.

You should now be on the "Advanced Settings" page. Modify these settings as you wish. Continue following the instructions, clicking the "Proceed to next step" button. You should reach the "Congratulations" page.

Issue the following commands to delete the `install/` directory and change the permissions on `config.php`:

    rm -rf /srv/www/example.com/public_html/forum/install/
    chmod 0644 /srv/www/example.com/public_html/forum/config.php

You should now be done configuring your phpBB setup!

## Monitor for Software Updates and Security Notices

When running software compiled or installed directly from sources provided by upstream developers, you are responsible for monitoring updates, bug fixes, and security issues. After becoming aware of releases and potential issues, update your software to resolve flaws and prevent possible system compromise. Monitoring releases and maintaining up to date versions of all software is crucial for the security and integrity of a system.

Please monitor the phpBB security announcement posts and community forums to ensure that you are aware of all updates to the software and can upgrade appropriately or apply patches and recompile as needed:

-   [phpBB Security Information](http://www.phpbb.com/security/)
-   [phpBB Community Forums](http://www.phpbb.com/community/index.php)

When upstream sources offer new releases, repeat the instructions for installing the phpBB software as needed. These practices are crucial for the ongoing security and functioning of your system.

## More Information

You may wish to consult the following resources for additional information on this topic. While these are provided in the hope that they will be useful, please note that we cannot vouch for the accuracy or timeliness of externally hosted materials.

- [phpBB Styles Database](http://www.phpbb.com/styles/)
- [Styles Forum](http://www.phpbb.com/community/viewforum.php?f=80)
- [phpBB Modifications](http://www.phpbb.com/mods/)
- [Modifications Forum](http://www.phpbb.com/community/viewforum.php?f=81)



