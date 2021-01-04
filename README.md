# TCI – Telegram Content Import

### A system plugin for CMS Joomla! providing import of Telegram posts in Joomla. <br>It is a CLI-solution based on the work of https://tg.i-c-a.su/ service.<br><br>

[![Buy it!](https://img.shields.io/badge/заказать-$75-28A5F5.svg?style=for-the-badge)](https://alekvolsk.pw/ru/#callback)<br><br>

![Joomla](https://img.shields.io/badge/joomla-3.7+-1A3867.svg?style=for-the-badge)
![Php](https://img.shields.io/badge/php-5.6+-8892BF.svg?style=for-the-badge)
![Last Update](https://img.shields.io/badge/last_update-2020.01.04-28A5F5.svg?style=for-the-badge)
![Version](https://img.shields.io/badge/version-1.1.4-1e87f0.svg?style=for-the-badge)<br><br>

The plugin is compatible with Joomla! 4.

---

_description in Russian [here](README.ru.md)_

---

## A short tutorial on plugin configuration and launching

After installing the plugin, make sure it is enabled.
Specify a system name of the public Telegram channel in the plugin parameters (without @ symbol).

The plugin executes from the command line solely. Add a Cron job task with an execution period in minutes and use a value that is identical to the plugin param 'Period' and use the following command/URL:

```
wget -O '{your domain}/index.php?option=com_ajax&group=system&plugin=tci&method=post&format=raw[additional params]'
```

The format depends on your hosting/server requirements, but the methodology is identical: a Cron job should execute the command line in a specified period of time.

Additionally, you can use the following params in cron job task:

```
&channel={name of the channel}&limit={limit of posts}&period={execution period in minutes}&catid={Category ID of com_content}&lang={content language}&featured={0|1}&userid={Joomla user ID}
```

This additional prefix can be used in the case when you need to import posts from custom channels that are not the same as the channel from default plugin settings.
It can be useful when you want to import data from various Telegram channels to the website.

---

## Plugin options

**Telegram server** \*. URL of the service which returns posts from any public Telegram channel in JSON format. The default value is  `https://tg.i-c-a.su/`.

**Channel system name** \*. A system name of the Telegram channel (without @ symbol) you want to import posts from.

**Limit of received posts per one execution** \*. Set the total number of posts to get from the service. The default value: `10`. *IMPORTANT NOTE: [What is a Telegram post?](#tgpost).*

**The period in minutes during which execution of posts is being processed** \*. The period of time is in minutes. Make sure that the execution period is identical to the plugin settings. E.g. if you set up Cron job execution in 60 minutes, the same param should be set in the plugin option. The default value: `60`.

**Process posts that were forwarded from other channels**. Available values: `Yes|No`. The default value: `No`.

**Articles category** \*. The category of Joomla articles to which the posts should be imported. The default value: `Uncategorized` (without category, ID=2).

**Articles language**. The default value: for all articles.

**Articles author**. A Joomla user to be assigned as the author of articles imported from Telegram.
The default value: the default Super Administrator who created the site

**Status of articles during the import**. Available values: `Unpublished|Published`. the default value: `Unpublished`.

**Add post ID to the end of alias**. It allow importing posts that have the same header. Available values: `Yes|No`. The default value: `No`.

**Set articles as Featured during import**. Available values: `Yes|No`. The default value: `No`.

**Transform Telegram post hashtags to Joomla tags**. Available values `Yes|No`. The default value: `No`.

**Transform hashtags inside the article to links that leads to Joomla tags**. Available values: `Yes|No`. The default value: `No`.
This param appears when "Transform Telegram post hashtags to Joomla tags" option is enabled.

**Create Meta-Description**. Available values: `Yes|No`. The default value: `No`. <br>*
The generation of Meta-Description has a feature*: the description is being created from the first 150 text symbols of Telegram post.
All iconographic symbols (icons as symbols, emoji) are truncated (ignored), the crop is being processed until the nearest space from the end of the line. If the line is not a complete sentence, then the ellipsis (3 dots) is being added.

**Folder for images to be taken from Telegram posts**. A Joomla folder to be used to save images from imported posts. The default value: `images` root folder in the site root. *IMPORTANT NOTE: [The nuances of Joomla article creation](#jarticle).*

**Action if the image download process is invalid**. Available values: `Miss image|Save original URL`. The default value: `Miss image`. *IMPORTANT NOTE: [The nuances of Joomla article creation](#jarticle).*

**Actions with images of Telegram posts-galleries**. Available values:

- `Keep inside the article`,
- `First image – inside the Intro Image and Full Article Image`,
- `First image – inside the Intro Image, the second image - inside Full Article Image`.

The default image: `Keep inside the article`. *IMPORTANT NOTE: [The nuances of Joomla article creation](#jarticle).*

**Add caption text for image**. It works for Intro Image and Full Article Image only. Available values: `Yes|No`. The default value: `No`.

---

## CLI parameters

Additional CLI parameters are due to the possible import of posts from more than 1 Telegram channel.

The correct specification of any of the CLI parameters below overrides the value of the corresponding plugin parameter.

**channel** – system name of the channel.

**limit** – Limit of received posts per one execution.

**period** – The period for what posts are being imported from the channel at the moment of execution. The minutes should be specified as a number.

**catid** – Category ID, can be specified as a number. You can find ID in the Joomla Administrator Panel - the list of com_content categories.

**lang** – Article language. It can be specified as the system constant (example: `en-GB`, `ru-RU`) or `*` symbol to specify **For all languages** value.

**featured** – Set articles as Featured. It should be specified as a number, the `0` value – No, `1` – Yes. Any other values are being ignored and the value to be taken from the plugin parameter.

**userid** – Set up Juumla user ID as the article author.

---

## https://tg.i-c-a.su/ service features

This service has the following limits:

- It processes public channels only.
- No more than 15 queries per minute.
- No more than 100 posts for one query.
- Temporary ban for any error. Example: invalid channel name or the channel is private.

The service has a free source code (the URL is available on the official page) and allows running own service at the own server which is highly recommended to do if you need importing of a huge amount of posts for a small period of time.

The ban has a cumulative time: an attempt to access the service while the ban is in effect automatically doubles the ban duration!

---

## <a name="tgpost"></a>What is a Telegram post?

A Telegram post is a more complex structure than the classic visual representation of what is usually considered a post. Technically, a visual post is multiple posts at once. For example, a post-gallery of 10 images is 10 posts at once, i.e. each image is a separate post, they are just grouped according to a certain characteristic. Upon import, these images will also be grouped into one Joomla material.

Why is it important? Because the default service has a request limit. Images are received in separate requests and all this happens in one plugin run. Take this into account when calculating the period: it may well turn out that by importing 10 gallery posts, you will receive 10 galleries of 10 images at the output, which ultimately will result in 101 requests (1 request - to get a list of posts), which is clearly more than the specified service limit in 15 requests.

### What types of posts are ignored?

The system posts of Telegram, hidden posts (hidden by the channel administrator), voting posts are ignored. Videos from gallery posts are also ignored.

---

## <a name="jarticle"></a>The nuances of Joomla article creation

The plugin only imports text and images.

The text is separated into paragraphs.

The header is being formed from the first paragraph. All iconographic symbols are cut from the title, the header is limited to 100 characters (Joomla restriction on the head of the material), then cut off to the first space from the end of the line. An article alias is being created from the resulting line to keep the alias unique: if there is another article with the same alias, then such the article is skipped.

If the first paragraph was trimmed during the formation of the head, then in its raw view the paragraph completely form the article intro, otherwise, the intro of the article is taken from the second paragraph.

All subsequent paragraphs form the main part of the article.

If there are less than three paragraphs, the main part of the article will not be formed.

If there is only one paragraph it completely creates Article Intro.

Images from gallery posts, which do not form the Intro Image and Full Article Image of the article, are being added only to the Full Article and each image is wrapped in a separate paragraph. No editing actions are required to do with the images.

If the plugin option *Convert hashtags from post to Joomla tags* is enabled -then all found hashtags to be transformed to Joomla tags and will be assigned to the article. Previously existing tags will not be re-created.

If the plugin option *Transform hashtags inside the article content into URLs that leads to tags* is enabled - then all found hashtags will be transformed to URLs that lead to the corresponding tags. In such a way, it is recommended to disable displaying article tags in the article options (or in the com_content component global options).
---

### Additional information
The plugin generates a monthly log via the core Joomla logging system. The filename is being created using the following mask: `tci_{year-month}.php.log`.
To view these logs from the Administrator Panel you can use the following component: View logs](https://github.com/AlekVolsk/View-logs/releases/latest).

URLs that lead to Youtube video inside already imported articles can be transformed via the following content plugin: [YtVideo](https://github.com/AlekVolsk/ytvideo/releases/latest).

---

**The price of this solution: $75 USD**

You can purchase this solution by contacting the author via the submission form on the author's personal website.

Before submitting the form, please make sure to fill out your Telegram username, and mention "TCI Order" in the message.

[![Buy now](https://img.shields.io/badge/заказать-$75-28A5F5.svg?style=for-the-badge)](https://alekvolsk.pw/ru/#callback)

Send PM to the author in Telegram to get product support.
