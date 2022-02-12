+++
title = "Photos"
date = 2022-02-11
[taxonomies] 
tags=["TIL"]
+++

If you have gigabytes of digital photos and movies, they can fill up your MacBook Air SSD very quickly.
Sorting through them can be a challenge, and frankly I don't have a good way of doing this.

### Find duplicates

```bash
find "$some_directory" -type f -exec md5sum {} \; \
    | sort | awk 'visited[$1]++' \
    | awk '{$1=""; print}' | tee duplicates-to-delete.txt
```

### Find similar images

I wrote a [program to find similar images][thing], and it found a few gigabytes of data to eliminate, but it was horribly slow while only comparing files of the same size.
It was fun to write, but maybe not the most time efficient endeavour.

### Use AI

There's a neat thing called [Photoprism][photoprism].
It's quite slow, but it will categorize your pictures in all kinds of dimensions.
This may help you to decide which pictures to delete, while also letting your browse them in a meaningful way.

[photoprism]: https://github.com/photoprism/photoprism
[thing]: https://github.com/mlbright/find-similar-images