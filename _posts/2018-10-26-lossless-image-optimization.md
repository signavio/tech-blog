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

Over the lifespan of a web application, this accumulates to a huge set of images easily. It immediately becomes legacy at the point of dropping support for certain old browsers and then cannot be transformed quickly. Even if there is in a long term perspective no way around tackling the cause, slight optimizations that work with the shotgun approach are always welcome.

Making use of state-of-the-art image compression while keeping full compatibility to traditional image decoders counts into that bucket and is the method that is going to shape my first blog post, taking some bigger portion of Signavio’s codebase as real world example.

# The situation given

After checking out the repo I performed a search to get a feeling which files I should have a look onto.

```
# find any PNG file, case-insensitive, count lines of the output
$ find . -iname '*.png' -type f | wc -l
```

Ok, there are around **2800 PNGs** and close to **700 GIFs**, but nearly no JPGs. Let's get an impression of how many space this consumes in bytes.

```
# find any PNG file, case-insensitive, print total byte size
$ find . -iname '*.png' -type f -ls | awk '{sum += $7} END {print sum}'
```

PNGs are around **14.8 MB**, GIFs need another **0.75 MB** on top. The average PNG file is 5,4 KB, the average GIF 1,2 KB. But don’t be tricked by the numbers, there are a lot GIFs, that are just some pixels and this way only need some dozens of bytes, so on the other hand, there are definitely some bigger loader animation GIFs or PNG sprites.

I said it already, GIF animation is used, it needs to be preserved if in place. The images come from various places, what means various encoders and people where involved over time, some of them probably cared about the size, some of them maybe also about generating spec compliant output, most likely none of them about color profiles, as these where long time ignored by browsers and a mistake would have not even been visible.

# What are the low hanging fruits?

[Google's Web Fundamentals](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/image-optimization) are good read to learn about image optimization. After googling around, I made a choice to use these tools:

## ZopfliPNG

In the [GitHub repository of Google's Zopfli](https://github.com/google/zopfli/blob/master/README.zopflipng) gzip/deflate compressor, there is also a tool hidden, that recompresses data within PNGs keeping compliance with Deflate. It also does some other optimizations, like removing metadata, leaving out color information for fully transparent pixels etc. but always with producing the same final result.

## Gifsicle

[Gifsicle](http://www.lcdf.org/gifsicle/) is almost cross-plattform available and is like a very advanced GIF editor, that runs in a shell. It optimizes via compression as far as possible, but also allows for several minification strategies, like storing only the changed portion of each frame in animations, remove redundant colors and only embed color tables that are really required.

Both tools are open source and free for (commercial) use.

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
