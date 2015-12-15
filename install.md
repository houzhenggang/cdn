# Overview

This is a step-by-step guide to adding HolaCDN to your website.

If you have any questions during implementation, please email cdn-help [at] hola [dot] or skype:holacdn.com 

# 1. Create an account

If you didn’t do so already, create an account on [https://holacdn.com](https://holacdn.com/), and note your customer ID which was sent to you following the registration.

On the HolaCDN customer portal, you will be able to view user experience statistics such as video start time, buffering, video quality and other settings. You will also be able to configure your HolaCDN system. To see a live portal account, login to the [demo account] (http://bit.ly/HolaCDNPortalDemo) with user: portal_demo@hola.org, password: holacdn.

Note that some of the instructions below are relevant to progressive download, and some to adaptive protocols.

# 2. Server-side settings 

To ensure optimal operation of Hola CDN, certain HTTP headers need to be enabled on the servers serving video files. The following section describes how to verify and if needed, enable these headers.

## 2.1 CORS settings for MP4/FLV/WEBM progressive video

Hola free bandwidth saver and CDN work by requesting your MP4/FLV/WEBM files from the video server in chunks. For this to work, certain HTTP headers need to be enabled.

Test to see if your HTTP server is configured correctly by using:

```curl -v -H "Origin: <site origin link>" -X OPTIONS -H  "Access-Control-Request-Headers: range" <link to MP4/FLV/WEBM file>```

Note: If you are using Amazon S3, use:

```curl -v -H "Origin: <site origin link>" -X OPTIONS -H "Access-Control-Request-Headers: range" -H "Access-Control-Request-Method: GET" <link to MP4/FLV/WEBM file>```

The desired response is:

```
HTTP/1.1 200 OK
Content-Length: 0
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: HEAD, GET, OPTIONS
Access-Control-Expose-Headers: Content-Range, Date, Etag
Access-Control-Allow-Headers: Content-Type, Origin, Accept, Range, Cache-Control
Access-Control-Max-Age: 600
Timing-Allow-Origin: *
```

If the response is different from the desired response, configure the missing headers by enabling CORS on the web server(s) that is serving the video files. Go line by line to ensure all headers are configured correctly. See the ‘Configuring CORS headers’ section for instructions.

## 2.2 Optional: CORS settings for HLS/HDS

Hola CDN works with modern, chunked video protocols by requesting video segments from multiple servers in parallel. Basic operation does not require any changes to CORS settings.

Hola recommends configuring certain HTTP headers. This allows HolaCDN to calculate bandwidth and maximize performance further. These changes are optional, and can be enabled at any time.

Test to see if your HTTP server is configured correctly by using:

```curl -v -H "Origin: <site origin link>" -X OPTIONS <link to m3u8/f4m manifest file>```

The desired response is:

```
HTTP/1.1 200 OK
Content-Length: 0
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: HEAD, GET, OPTIONS
Access-Control-Expose-Headers: Date, Etag
Access-Control-Max-Age: 600
Timing-Allow-Origin: *
```

In case the response is different from the desired response, configure the missing headers by enabling CORS on the web server(s) that is serving the video files. Go line by line to ensure all headers are configured correctly. Please see the ‘Configuring CORS headers’ section for instructions.

## 2.3 Configuring CORS headers

For step by step instructions regarding how to enable CORS on different web servers, see the [[original CORS documentation](http://enable-cors.org/server.html)] (http://enable-cors.org/server.html). If you are using Amazon S3, please click [[here](https://github.com/hola/cdn/blob/master/progressive_download.md#using-amazon-s3)] (https://github.com/hola/cdn/blob/master/progressive_download.md#using-amazon-s3). Make sure you add all the required headers, not just '*' referenced in the generic instructions.

After committing the configuration changes, verify response that headers to from this server(s) include required headers, as described above.

## 2.4 Handling redirects

In some implementations, the first video URL is redirected to another URL. In this case, make sure that:

- Any video data links that redirect also respond to OPTIONS with CORS headers as detailed above.

- The response headers must respond to OPTIONS with 200/204 status, and not with 302.

# 3. Allowing Hola CDN to download content

Hola’s CDN servers need to download a first copy of the video from your infrastructure to serve to future users. 

## 3.1 Configuring video origin servers

Hola’s CDN needs to know where to download a copy of the video from your infrastructure to serve to future users. Please email cdn-help [at] hola [dot] with your customer ID and a list of the video servers. 

For example, if the URL to your video file looks like

```[http://video.example.org/content/sample_video.mp4](http://video.example.org/content/sample_video.mp4)
```
, or
```[http://video.example.org/content/sample_video.](http://video.example.org/content/sample_video.m3u8)
```

simply send ‘video.example.org’ with your customerID to HolaCDN. 

## 3.2 Handling content protection

If your video servers do not use any content protection algorithms, skip to step 4.

In case your video URLs use content protection scheme, Hola servers will not be able to download videos, and Hola CDN will not be able to function. There are a few ways of dealing with content protection:

### 3.2.1 Whitelisting Hola CDN servers

Whitelisting the Hola CDN servers is the fastest way to enable Hola CDN to operate. Add the following servers to your list of whitelisted IPs:

```
50.7.1.2
46.105.109.214
185.18.206.193
85.17.24.129
37.187.161.44
76.73.18.98
5.196.82.58
204.45.27.2
```

### 3.2.2 Allow Hola servers to access your videos using other methods 

In case whitelisting IPs is not an option, you will need to work with Hola to define alternative ways to allow the Hola servers to download video files. 

There are a few ways: 

* Share the key generation algorithms with Hola, so that Hola servers will generate valid requests

* Set up a direct/hidden URL

* Set-up a special key which will identify Hola servers.

Please contact Hola in order to determine the best way to address this issue.

# 4. Add Hola JS to your website

## The implementation varies slightly based on the video player used - see below.

## Note that you can safely add the JS code to your web page, it is disabled by default on the server side in order to protect from accidental mass deployment. After the code is on your web pages, you can enable Hola CDN gradually in order to ensure a smooth deployment:

* First on your local machine (step 4)

* Then to a subset of users, then to all users (step 5)

## 4.1 HTML5 video players

When integrating with an HTML5 source, HolaCDN attaches itself to a <video> tag. A video tag can be embedded in the raw HTML itself, or it can be dynamically created by your video player (e.g. VideoJS)

Add the script to your page as follows:

```video src="http://example.com/uploads/myVideo.mp4" controls

...

  <script async src="//player.h-cdn.com/loader.js?customer=HC_XXXXXX"></script>

...

/body

/html```

### Live examples:

http://js.do/gilad/html5_video

## 4.2 Using Hola VideoJS player

http://js.do/gilad/html5_hola_player

## 4.3 Flash based video players

### 4.3.1 JW Player

If your site uses JW Player with flash technology, follow these steps:

1. Add Hola loader to the <head> element of the video HTML page:
```
<head>
….
  **<script type="text/javascript” async src=”//player.h-cdn.com/loader.js?customer=XXXXXX”></script>
**….
</head>
```

2. Replace your JW player SWF with the Hola-enabled version. This is done by configuring {flashplayer: <url>} option in jwplayer(‘video-container’).setup(opt) call:
```
jwplayer(‘video-container’).setup({
    file: ‘//cdn.example.com/popular_videos/example.mp4’,
    **flashplayer: ‘//example.com/static/<new-version-flashplayer>.swf’,
**    primary: ‘flash’,
    width: 640,
    height: 360
});
```
Download the Hola-enabled version and place it on your own server. Get it from:

* JWPlayer6: [https://player.h-cdn.com/jwplayer.flash.hls.swf](https://player.h-cdn.com/jwplayer.flash.hls.swf)

* JWPlayer7: [https://player.h-cdn.com/jwplayer.flash.7_1_0.swf](https://player.h-cdn.com/jwplayer.flash.7_1_0.swf)

.

3. Initialize Hola CDN loader right after the call to ```jwplayer(‘video-container’).setup(opt)```. call:
```
jwplayer(‘video-container’).setup({
    file: ‘//cdn.example.com/popular_videos/example.mp4’,
    flashplayer: ‘//example.com/static/<new-version-flashplayer>.swf’,
    primary: ‘flash’,
    width: 640,
    height: 360
});
**if (window.hola_cdn)
    window.hola_cdn.init(CustomerID=XXXXX);  
else
    window.hola_cdn_on_load = true;**
```

Note: In case you load the player and its init code in a separate script which you can not modify, enable Hola as follows. Make sure that Hola init code is executed after ```jwplayer(‘video-container’).setup(opt)``` call:
```
<script type="text/javascript” src=”https://content.jwplatform.com/players/<player_script>.js”></script>
**<script>**
**if (window.hola_cdn)
    window.hola_cdn.init(CustomerID=HC_XXXXXXXX);
else
    window.hola_cdn_on_load = true;
</script>**
```

### Live examples:

JWPlayer/Flash/MP4: [http://js.do/hola/jwplayer6_flash](http://js.do/hola/jwplayer6_flash)
JWPlayer/Flash/MP4 with Hola CDN: [http://js.do/hola/jwplayer6_flash_with_hola_cdn](http://js.do/hola/jwplayer6_flash_with_hola_cdn)

### 4.3.2 VideoJS

http://js.do/gilad/html5_vjs

### Live Example

<link to JS.DO example playing demo video without CDN>

<link to JS.DO example playing demo video with CDN>

# 5. Testing Hola CDN locally on your PC

Once the code is live on the webpage, remember it is still disabled by default on the server side. You can test the live code locally by either appending a command to your URL, or by entering commands in the browser developer console. 

## Configuring via address bar

To control HolaCDN via the address bar, append ```?hola_mode=xxx``` to the URL.

To enable CDN mode:	append ```?hola_mode=cdn```

To enable stats mode: 	append ```?hola_mode=stats```

To disable HolaCDN:	append ```?hola_mode=disabled```

Note that in order to check mode or see statistics, you will need to use the console, see below. 

## Configuring via browser console

In the browser developer console, enter one of the following commands:

To enable: 		```hola_cdn.set_mode_cdn()```

To disable:		```hola_cdn.set_mode_disable()```

To view the mode: 	```hola_cdn.mode```

To see statistics: 	```hola_cdn.get_stats()```

To see all settings:	```hola_cdn.help()```

Note: If your site includes frames, don’t forget to enter the console commands in the frame where the video player is located.

## Checking statistics on the portal

Login to your account on [www.holacdn.com](http://www.holacdn.com) and verify that statistics are written to your account. To view individual events, click on ‘debug mode’. Note that it may take a few minutes for statistics to appear on the portal.

# 6. Deployment to production

When you are satisfied with local testing, you can gradually enable the service in production. Login to your portal account and go to the configuration section. Use the granular controls to enable Hola on different platforms/browsers

For example:

* Start by enabling Hola CDN for 10% of Chrome/Win users, and 90% only for statistics collection.

* Increase Hola CDN to 90% of Chrome/Win users, and leave 10% for statistics.

* Add more/browsers/platforms.

Changes take effect immediately, and you will receive a confirmation email every time you change settings on the portal.