---
title: Conversion from UA into GA4
date: 2023-04-10 23:15:57
tags: [GoogleAnalytics, Hexo]
---

# _`Overview`_

Google noticed that the support for Universal Analytics will be ended in 2023/06/30. Check [this document](https://support.google.com/analytics/answer/12938611) for more details.

Therefore, I had to migrate my UA settings into GA4. Here is a solution for Hexo blog, which is the framework I am using for this blog.

# _`Google Tag`_

For activating Google Analytics, Google provides you a tag named as "Google Tag". First of all, you should find out what tag should be installed. You can find out this at Google Analytics 4's page.

{% asset_img 02.png %}

Click the button `Admin`.

{% asset_img 03.png %}

Click the button `Account Access Management`.

{% asset_img 04.png %}

Click the button `Data Streams`.

{% asset_img 05.png %}

Click the right arrow at your data stream.

{% asset_img 06.png %}

Click the right arrow at the option `Configure tag settings`.

{% asset_img 07.png %}

Click the button `Installation instructions`.

{% asset_img 08.png %}

Check out the code that you should include manually. This is the Google Tag for analytics.

# _`Theme Config`_

{% asset_img 01.png %}

Most of themes for Hexo have the config file for Google Analytics. You can find it by just searching "google_analytics" with text.

{% asset_img 09.png %}

Especially, `google-analytics.ejs` file would be containing the Google Tag.

{% asset_img 10.png %}
<br/>
{% asset_img 11.png %}

The tag is inserted at the front of page of every post in your blog, so you can check that with `View page source`. Thus, you should replace the Google Tag with new one. Copy the new one we have prepared and paste it to the `google-analytics.ejs` file. Here are the commits I used for that.

- https://github.com/BaeMinCheon/let-us-go-abroad/commit/076b8a509f1c317cd2b92d875d00945c1ac2a718
- https://github.com/BaeMinCheon/let-us-go-abroad/commit/b3ca2983b70daa46acbb8fb1387922ed0320a25f

# _`Result`_

After the setup, the data stream will be constructed. But, it would take some time...about 1 day or 2 days ? So just keep calm and wait for that.

{% asset_img 12.png %}

When it constructed successfully, you can see the result `DATA FLOWING` just like above at `GA4 / Admin / Account Access Management / Setup Assistant`.