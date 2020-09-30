---
layout: post
title:  Using CDN to improve file uploads
date:   2020-09-29
categories: software
tags: ruby performance
excerpt_separator: <!--more-->
---

For one of my client projects, I explored different ways to increase upload speeds for large files uploaded from a desktop computer to AWS S3. The desktop computer was located in a different country, halfway around the world, with a throttled/restrictive outbound internet connection to the S3 storage located in the US region. One approach is doing parallel uploads but the focus was on how to get a faster connection.

I discovered 3 ways to architect file uploads to AWS S3:

1. Standard mode
In this mode, you setup a bucket in S3 in a specified region with all the default settings and the client uploads the files to the S3 region directly no matter where the client is located in the world. This is the most simplest setup. The file upload speed will greatly depend on quality of the entire connection and ping times will depend on the distance to the storage.

2. S3 Accelerated mode
You create a bucket like you normally do in #1 above but you check the box for file acceleration. This allows a client device to connect to the nearest S3 location and then uses the AWS hi-speed backbone connection to the specified S3 bucket region. The upload speed will depend on the quality of the connection between the client and the nearest S3 location as we can assume AWS inter-region connection is a high bandwidth pipe that won't be the limiting factor. You might experience slights delays for S3 to synchronize files between regions. This is very easy to implement however there are additional costs (see AWS S3 pricing) to use this.

3. Using Cloudfront or a CDN as an edge location
This was an interesting idea and a different use case of a CDN. A CDN is normally used to cache assets so clients all the world will have a closer location to download the assets resulting in faster download times. However instead of using a CDN to cache asset files for clients to download, we can use it as an edge location for client devices to connect to, and then transport the stream to the specified S3 location using larger high bandwidth connections. It is essentially the same as #2 above, but you're not limited to S3 locations and CDN's can have more locations for clients to connect to. The CDN does not buffer or store the file, it is simply allowing the data stream to pass through it to the end destination. There are additional costs to using CDN's and the setup is more involved requiring knowledge on how to configure the CDN and AWS to accept connections from it.

### Results

Unfortunately, none of these solutions resulted in any significant speed improvements over the standard mode (#1) most likely due to the country where the desktop computer was located. Also, AWS did not own any servers located in that country and as such the restrictive outbound connection to the nearest S3 location or CDN was probably the bottleneck.

However if we were doing file uploads in a different country that didn't have a restrictive outbound connection, I think the S3 Accelerated mode or using the CDN method would have resulted in a potential improvement in upload speeds and is worth exploring.
