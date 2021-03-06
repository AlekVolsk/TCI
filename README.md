# TCI – Telegram Content Import

### A system plugin for CMS Joomla! providing import of Telegram posts in Joomla. <br>It is a CLI-solution based on the work of https://tg.i-c-a.su/ service.<br><br>

[![Buy now!](https://img.shields.io/badge/buy_now-$75-28A5F5.svg?style=for-the-badge)](https://alekvolsk.pw/ru/#callback)
[![Demo](https://img.shields.io/badge/-demo_(in_russian)-555555.svg?style=for-the-badge)](https://forester.alekvolsk.info/)<br><br>

![Joomla](https://img.shields.io/badge/joomla-3.7+-1A3867.svg?style=for-the-badge)
![Php](https://img.shields.io/badge/php-5.6+-8892BF.svg?style=for-the-badge)
![Last Update](https://img.shields.io/badge/last_update-2021.05.05-28A5F5.svg?style=for-the-badge)
![Version](https://img.shields.io/badge/version-1.5.2-1e87f0.svg?style=for-the-badge)

The plugin is compatible with Joomla! 4.

---

_description in Russian [here](README.ru.md)_

---

## A short tutorial on plugin configuration and launching

After installing the plugin, make sure it is enabled.
Specify a system name of the public Telegram channel in the plugin parameters (without @ symbol).

The plugin executes from the command line solely. Add a Cron job task with an execution period in minutes and use a value that is identical to the plugin param 'Period' and use the following command/URL:

```
wget -q -O -- "{your domain}/index.php?option=com_ajax&group=system&plugin=tci&method=post&format=raw&key={security key}[additional params]"
```

The format depends on your hosting/server requirements, but the methodology is identical: a Cron job should execute the command line in a specified period.

The security key will be automatically generated the first time the plugin settings are saved.

Additionally, you can use the following params in cron job task:

```
&channel={name of the channel}&limit={limit of posts}&period={execution period in minutes}&catid={Category ID of com_content}&lang={content language}&publiched={0|1}&featured={0|1}&userid={Joomla user ID}
```

This additional prefix can be used in the case when you need to import posts from custom channels that are not the same as the channel from default plugin settings.
It can be useful when you want to import data from various Telegram channels to the website.

---

## Plugin options

#### Basic

**Telegram server** \*. URL of the service which returns posts from any public Telegram channel in JSON format. The The default value is  `https://tg.i-c-a.su/`.

**Security Key**. It is necessary to protect the site from possible DDOS attacks. Automatically generated after you save plugin parameters first time.

**Debug mode**. A mode in which post data is requested only once and written to the `tci_{channel_name}.json` file in the Joomla cache folder. If a file exists again from the specified channel, json is not requested, the file is cleared by the standard Joomla cache clearing method. The parameter affects all channels, incl. specified through a request in cli-requests. Do not leave cache files after turning off debug mode!<br>
In debug mode, posts are always created unpublished.<br>
In debug mode, a message is written to the log about enabling debug mode and information about the reasons for skipping the import of each post according to the plugin parameters.<br>
This mode can be useful when developing a [content preprocessing plugin](#contentbeforedata).

#### Telegram channel

**Channel system name** \*. A system name of the Telegram channel (without @ symbol) you want to import posts from.

**Limit of received posts per one execution** \*. Set the total number of posts to get from the service. The The default value: `10`. *IMPORTANT NOTE: [What is a Telegram post?](#tgpost).*

**The period in minutes during which execution of posts is being processed** \*. The period is in minutes. Make sure that the execution period is identical to the plugin settings. E.g. if you set up Cron job execution in 60 minutes, the same param should be set in the plugin option. The The default value: `60`.

#### Joomla article

**Articles category** \*. The category of Joomla articles to which the posts should be imported. The The default value: `Uncategorized` (without category, ID=2).

**Articles language**. The The default value: for all articles.

**Articles author**. A Joomla user to be assigned as the author of articles imported from Telegram. The The default value: the default Super Administrator who created the site.

**Status of articles during the import**. Available values: `Unpublished|Published`. The The default value: `Unpublished`.

**Add post ID to the end of alias**. It allows importing posts that have the same header. Available values: `Yes|No`. The The default value: `No`.

**Set articles as Featured during import**. Available values: `Yes|No`. The The default value: `No`.

**Transform Telegram post hashtags to Joomla tags**. Available values `Yes|No`. The The default value: `No`.

**Transform hashtags inside the article to links that leads to Joomla tags**. Available values: `Yes|No`. The The default value: `No`.
This param appears when "Transform Telegram post hashtags to Joomla tags" option is enabled.

**Create Meta-Description**. Available values: `Yes|No`. The The default value: `No`. <br>
*The generation of Meta-Description has a feature*. The description is being created from the first 150 text symbols of Telegram post. All iconographic symbols (icons as symbols, emoji) are truncated (ignored), the crop is being processed until the nearest space from the end of the line. If the line is not a complete sentence, then the ellipsis (3 dots) is being added.

**Folder for images to be taken from Telegram posts**. A Joomla folder to be used to save images from imported posts. The The default value: `images` root folder in the site root. *IMPORTANT NOTE: [The nuances of Joomla article creation](#jarticle).*

**Action if the image download process is invalid**. Available values: `Miss image|Save original URL`. The The default value: `Miss image`. *IMPORTANT NOTE: [The nuances of Joomla article creation](#jarticle).*

**Actions with images of Telegram posts-galleries**. Available values:

- `Keep inside the article`,
- `First image – inside the Intro Image and Full Article Image`,
- `First image – inside the Intro Image, the second image – inside Full Article Image`.

The default value: `Keep inside the article`. *IMPORTANT NOTE: [The nuances of Joomla article creation](#jarticle).*

**Add caption text for image**. It works for Intro Image and Full Article Image only. Available values: `Yes|No`. The The default value: `No`.

**Prefix picture folder**. Available values:

- `Don't usage (not recommended)`,
- `Channel ID`,
- `Channel name`.

The default value: `Channel ID`.

**Include comments from the post**. Allows you to connect live comments attached to the post. If you have authorization from the telegram, it allows you to fully use the telegram comment system on the site. The connection is made by downloading the corresponding Telegram API script.
Connection is made only in cases when at the time of import the post had active comments, and the structure with information about the post is saved in the field of comments to the article.
In the markup of the article, the comment block is wrapped in a container with the `telegram-comments` class. Connection of a dark theme is supported. The plugin does not contain its own styles for the design of the comment block.
Available values: `Yes|No`. The default value: `No`.

**Dark theme**. Connecting a dark theme for the comments block. Available values: `Yes|No`. The default value: `No`.
This param appears when "Include comments from the post" option is enabled.

#### Parameters of conditions for skipping imported posts

*Note: the order of processing skip conditions is performed in the sequence of listing the following parameters.*

**Process posts that were forwarded from other channels**. Available values: `Yes|No`. The The default value: `No`.

**Skip media posts without text content**. Specifically: image posts, file posts. Available values: `Yes|No`. The default value: `No`.

**Skip posts of grouped content (gallery posts)**. Available values: `Yes|No`. The default value: `No`.

**Skip posts w/o images (including galleries)**. In the case of a post-gallery, the condition will work only if an intro/full image is saved in the article. Available values: `Yes|No`. The The default value: `No`.

**Skip posts without text content**. Allows you to skip image posts and galleries if they lack a text content. Available values: `Yes|No`. The The default value: `No`.

**Skip posts if text content is no more than characters**. Including spaces and punctuation marks. Enter 0 to remove restrictions. The default value: `0`.

**Skip posts if text content is no more than paragraphs**. The counting of the number of paragraphs is carried out before the formation of the heading of the material. Enter 0 to remove restrictions. The default value: `0`.

**Skip voting posts**. Available values: `Yes|No`. The default value: `No`.

**Skip posts with buttons**. Buttons can be either voting buttons or link buttons. Available values: `Yes|No`. The default value: `No`.

**Skip video post, incl. video from gallery's post**. Available values: `Yes|No`. The default value: `Yes`. *Video download is possible only if the file size does not exceed 50mb. If this threshold is exceeded, the video file will not be loaded, there will be no references to it.*

**Skip file posts**. Available values: `Yes|No`. The default value: `Yes`. *File download is possible only if the file size does not exceed 50mb. Upon exceeding this threshold, a link to the post in Telegram will be saved.*

**File action**. Available values:

- `Save link to Telegram post`,
- `Download file from Telegram`.

The default value: `Save link to Telegram post`.
This param appears when "Skip file posts" option is disabled.

---

## CLI parameters

Additional CLI parameters are due to the possible import of posts from more than 1 Telegram channel.

The correct specification of any of the CLI parameters below overrides the value of the corresponding plugin parameter.

**channel** – system name of the channel.

**limit** – Limit of received posts per one execution.

**period** – The period for what posts are being imported from the channel at the moment of execution. The minutes should be specified as a number.

**catid** – Category ID, can be specified as a number. You can find ID in the Joomla Administrator Panel – the list of com_content categories.

**lang** – Article language. It can be specified as the system constant (example: `en-GB`, `ru-RU`) or `*` symbol to specify **For all languages** value.

**publiched** – Published article. It should be specified as a number, the `0` value – No, `1` – Yes. Any other values are being ignored and the value to be taken from the plugin parameter. The parameter is ignored in debug mode.

**featured** – Set articles as Featured. It should be specified as a number, the `0` value – No, `1` – Yes. Any other values are being ignored and the value to be taken from the plugin parameter.

**userid** – Set up Juumla user ID as the article author.

---

## https://tg.i-c-a.su/ service features

This service has the following limits:

- It processes public channels only.
- No more than 15 queries per minute.
- No more than 100 posts for one query.
- Temporary ban for any error. Example: invalid channel name or the channel is private.

The service has a free source code (the URL is available on the official page) and allows running own service at the own server which is highly recommended to do if you need importing of a huge amount of posts for a small period.

The ban has a cumulative time: an attempt to access the service while the ban is in effect automatically doubles the ban duration!

---

## <a name="tgpost"></a>What is a Telegram post?

A Telegram post is a more complex structure than the classic visual representation of what is usually considered a post. Technically, a visual post is multiple posts at once. For example, a post-gallery of 10 images is 10 posts at once, i.e. each image is a separate post, they are just grouped according to a certain characteristic. Upon import, these images will also be grouped into one Joomla material.

Why is it important? Because the default service has a request limit. Images are received in separate requests and all this happens in one plugin run. Take this into account when calculating the period: it may well turn out that by importing 10 gallery posts, you will receive 10 galleries of 10 images at the output, which ultimately will result in 101 requests (1 request - to get a list of posts), which is clearly more than the specified service limit in 15 requests.

### What types of posts are ignored?

The system posts of Telegram, hidden posts (hidden by the channel administrator), post-stickers are ignored. Audio recordings are treated as ordinary unknown files.

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

If the plugin option *Convert hashtags from post to Joomla tags* is enabled – then all found hashtags to be transformed to Joomla tags and will be assigned to the article. Previously existing tags will not be re-created.

If the plugin option *Transform hashtags inside the article content into URLs that leads to tags* is enabled – then all found hashtags will be transformed to URLs that lead to the corresponding tags. In such a way, it is recommended to disable displaying article tags in the article options (or in the com_content component global options).

JSON structure that consist of Telegram post is saved in the «Note» field of Joomla article. This data can be taken by the 3rd party extension which operates with com_content. The structure consist of an object with tci name that has a set of fields. The structure fields consist of:

- `channelid` – channel ID,
- `channelname` – channel system name,
- `channeltitle` – public title of the channel,
- `postid` – post ID in channel,
- `replies` – comments attached to the post.

This structure is necessary for the correct operation of the functionality of connecting the comment block. Also, it can, without changing, use other custom third-party solutions.

---

### <a name="contentbeforedata"></a>Pre-processing the content of the material before saving it

You can process the content of the material before it is saved by a third party plugin on the `onContentBeforeData` event.

IMPORTANT: The plugin must be systemic! The content plugin works out once for the session and in the end only 1 material will be processed, the rest will be skipped: the resinuate plugin will simply do not start, so the Joomla is arranged.

The plugin gets 2 params as inputs:

- `(string) $context` – the value `com_plugins.plugin.system.tci` to be sent, you should check the context on this value solely and make sure that the article which is importing by this plugin is being processed;
- `(array) $data = []` – standard array of article object, not to be send by URL.

The return value is expected as an array of article object for these changes to be applied either false otherwise.

An example of implementing an event function:

```php
public function onContentBeforeData($context, $data = [])
{
    if ($context !== 'com_plugins.plugin.system.tci') {
        return false;
    }

    // change your's data

    return $data;
}
```

---

### Additional information
The plugin generates a monthly log via the core Joomla logging system. The filename is being created using the following mask: `tci_{year-month}.php.log`.
To view these logs from the Administrator Panel you can use the following component: [View logs](https://github.com/AlekVolsk/View-logs/releases/latest).

URLs that lead to Youtube video inside already imported articles can be transformed via the following content plugin: [YtVideo](https://github.com/AlekVolsk/ytvideo/releases/latest).

---

**The price of this solution: $75 USD**

You can pay in any currency of the world, except bitcoins, at the exchange rate at the time of payment.

You can purchase this solution by contacting the author via the submission form on the author's personal website.

Before submitting the form, please make sure to fill out your Telegram username, and mention "TCI Order" in the message.

[![Buy now](https://img.shields.io/badge/buy_now-$75-28A5F5.svg?style=for-the-badge)](https://alekvolsk.pw/ru/#callback)

Send PM to the author in Telegram to get product support.
