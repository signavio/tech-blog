---
title: "Batch image optimization"
description: "How to highly optimize static PNG and GIFs to save some bandwidth"
author: Sebastian Friedrich
tags: static files, batch processing, CLI, Zopfli, gifsicle
layout: article
image:
  feature: /2018/image-compression-hero.jpg
  alt: "Seamless stacking of books forming solid wall"
---

While modern web frontends describe already almost every tiny detail of a user interface through rules of CSS in company with bandwidth saving SVG graphics and webfonts, it was not that long ago, that thousands of little raster images helped out to bring designs to life to a wide span of browsers.

Over the lifespan of a web application, this accumulates to a huge set of images easily. It immediately becomes legacy at the point of dropping support for certain old browsers and then cannot be transformed quickly. Even if there is in a long term perspective no way around tackling the cause, slight optimizations that work with the shotgun approach to at least reduce pain are always welcome.

Making use of state-of-the-art image compression while keeping full compatibility to traditional image decoders counts into that bucket and a big part of Signavio’s codebase should serve as an real world example to form my first blog post.

# The situation given

* almost only gif + PNG
* thousands of image files
* created from various editors, no information on quality
* some gif animations
* traditionally no color management strategy, no matter which profile might be embedded, sRGB would be expected

# What are the low hanging fruits?

* ZopfliPNG [Details about ZopfliPNG](https://github.com/google/zopfli/blob/master/README.zopflipng)
  * uses Zopfli compression while being compliant with Deflate
  * compares several strategies for choosing scanline filter codes
  * chooses a suitable color type to losslessly encode the image
  * removes unimportant chunks for the typical web use like metadata
  * optionally alters the hidden colors of fully transparent pixels
  * optionally converts 16-bit color channels to 8-bit
* gifsicle [Details of gifsicle](http://www.lcdf.org/gifsicle/)
  * stores only the changed portion of each frame in animations
  * removes redundant colors
  * only uses local color tables if it absolutely has to

# Working environment

Ubuntu Shell on Windows 10, git

```
$ sudo apt-get install gifsicle imagemagick imagemagick-doc
```

```
$ git clone https://github.com/google/zopfli.git
$ cd zopfli
$ make zopflipng
```

copy or softlink to your ~/bin directory or make available via PATH in the way you prefer
```
$ cp ./zopflipng ~/bin
```

# Measure the outcome

```
$ find . -iname '*.png' -type f -ls | awk '{sum += $7} END {print sum}'
```

```
$ find . -iname '*.gif' -type f -ls | \
  awk 'gsub(/^.+\//, "", $11)' | \
  awk '!seen[$11]++ {u_cnt++; u_sum+=$7}
       {i_cnt++; i_sum+=$7}
       END {printf "files: %s\nfiles unique: %s\nbytes total: %s\nbytes unique: %s\n",
            i_cnt, u_cnt, i_sum, u_sum}'
```

[show sample image of byte and file count]

measure time for ZopfliPNG via script

# Optimize all GIFs

execute

```
$ find . -iname '*.gif' -type f -print0 | xargs -0 -P4 -I{} gifsicle --batch --optimize=3 {}
```

# Optimize all PNGs

_batch-zopflipng.sh_ (expecting ZopfliPNG available via PATH)
```
#!/bin/bash

# batch zopfli compression, use like (from your working root folder):
# $ find . -iname '*.png' -type f | ~/your/path/to/batch-zopflipng.sh

# write a recompressed file to the temp dir, move when finished
fn_recompress() {
	# use local variable for thread safety
	local tmpfile=$(mktemp)

	# encode
	zopflipng -m -y --lossy_transparent $1 $tmpfile

	# move tempfile if there is one and size is greater than zero
	# it's the case zopfli was actually able to compress better
	if [ -s $tmpfile ]; then
		mv $tmpfile $1
	else
		rm $tmpfile
	fi
}

# make custom function available in subshell
export -f fn_recompress

# time tracking
start_time=$(date +%s)

# run font scans in parallel (8 cores)
xargs -n1 -P4 -I{} bash -c 'fn_recompress $1' _ {}

# time tracking result
end_time=$(date +%s)
echo "execution time was $(expr $end_time - $start_time)s."
```

# The moment of truth

[show screenshot of cpu load]

Discuss outcome, list numbers and savings, duration

# About the author

I am Sebastian, working already for a couple of years at Signavio as a Designer looking into various fields over time and currently settled in the Engineering department. I especially enjoy digging deeply into details of a problem and make an excursion into programming from time to time.

<a style="background-color:black;color:white;text-decoration:none;padding:4px 6px;font-family:-apple-system, BlinkMacSystemFont, &quot;San Francisco&quot;, &quot;Helvetica Neue&quot;, Helvetica, Ubuntu, Roboto, Noto, &quot;Segoe UI&quot;, Arial, sans-serif;font-size:12px;font-weight:bold;line-height:1.2;display:inline-block;border-radius:3px" href="https://unsplash.com/@diesektion?utm_medium=referral&amp;utm_campaign=photographer-credit&amp;utm_content=creditBadge" target="_blank" rel="noopener noreferrer" title="Download free do whatever you want high-resolution photos from Robert Anasch"><span style="display:inline-block;padding:2px 3px"><svg xmlns="http://www.w3.org/2000/svg" style="height:12px;width:auto;position:relative;vertical-align:middle;top:-1px;fill:white" viewBox="0 0 32 32"><title>unsplash-logo</title><path d="M20.8 18.1c0 2.7-2.2 4.8-4.8 4.8s-4.8-2.1-4.8-4.8c0-2.7 2.2-4.8 4.8-4.8 2.7.1 4.8 2.2 4.8 4.8zm11.2-7.4v14.9c0 2.3-1.9 4.3-4.3 4.3h-23.4c-2.4 0-4.3-1.9-4.3-4.3v-15c0-2.3 1.9-4.3 4.3-4.3h3.7l.8-2.3c.4-1.1 1.7-2 2.9-2h8.6c1.2 0 2.5.9 2.9 2l.8 2.4h3.7c2.4 0 4.3 1.9 4.3 4.3zm-8.6 7.5c0-4.1-3.3-7.5-7.5-7.5-4.1 0-7.5 3.4-7.5 7.5s3.3 7.5 7.5 7.5c4.2-.1 7.5-3.4 7.5-7.5z"></path></svg></span><span style="display:inline-block;padding:2px 3px">Robert Anasch</span></a>
