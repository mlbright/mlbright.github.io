+++
title = "iCloud Photos Reset"
date = 2022-12-29
[taxonomies] 
tags=["TIL"]
+++

My Apple [iCloud.com][icloud] account's 200GB storage limit was fast approaching when I realized that
most of the 2TB storage from my Google One subscription was unused.

I decided it was time to do a reset: I do not have time to sort through all the photos and videos to decide which ones to delete.
I tried [a few different ways][photos] to find and remove duplicate content and wasn't very successful in reducing the total storage used.
It was time to bite the bullet and copy all the photos and videos to [Google Photos][gphotos].
Then I would delete the files from iCloud.
In this way I would preserve all the files and not worry about disk space in the Apple ecosystem for a little while. 
(It has taken me 15 years to reach the 200GB iCloud limit.)

Here is what I did:

1. Copied the files to a local volume.
1. Made a backup copy to AWS S3.
1. Uploaded the files to Google Photos.
1. Deleted the files from iCloud.

If and when I run into the 200GB iCloud limit again, I will lean on the above process to once again preserve my photos and free up space.
By the time this occurs, there will no doubt be a better way.
If you know of a better method __today__, please let me know via email.

## Copying the files to a local EBS volume

I wanted to do all this in the cloud to avoid excessive bandwidth use in my home network.
This may not have been an actual problem but I had an existing [AWS][aws] t2.micro machine running for another purpose, so I decided to use that system to do everything.
Downloading the files into AWS does not cost anything, but there are egress charges to send the data to Google.

Via the AWS web console, I created and mounted a local EBS volume onto the Ubuntu 22.04 t2.micro machine mentioned previously.
I attached the volume and it showed up as `/dev/xvdf`.
Then I mounted it as `/data`.

```bash
sudo mount /dev/xvdf /data
sudo chown ubuntu:ubuntu /data
```

Copying the files from iCloud was very easy thanks to the wonderful [icloudpd] script and its very sensible default settings.
It was just a matter of finding the right invocation to download all the files.

```bash
data_folder="/data"
mkdir -p "$data_folder"

icloudpd --directory "$data_folder" \
  --username "$(cat mac-user)" \
  --password "$(cat mac-password)" \
  --no-progress-bar
```

This was a single threaded process.
I let it run the entire night and found it had completed sucessfully the following morning.

## Backing up to S3

While not necessary or cheaper to make another copy of the files, I thought it was a good idea to avoid putting all my eggs in one cloud basket.

I copied all the files to S3.

```bash
path="/data"
s3Dir="s3://mlbright-photos-0/0"

for entry in "$path"/*; do
  name="$(echo "$entry" | sed 's/.*\///')"
  
  if [[ -d "$entry" ]]; then
    aws s3 cp  --recursive "$entry" "$s3Dir/$name/"
  else
    aws s3 cp "$entry" "$s3Dir/"
  fi
done
```

## Uploading to Google Photos

Uploading the files to Google was easy thanks to the [Google Photos Uploader CLI][gphotos-uploader-cli].
I had over 40000 files and the daily Google Photos quota is 10000 file upload operations.
The CLI continued attempting to upload even after the quota was exceeded and the API was returning HTTP 429 responses.
So I fixed this and submitted a couple of pull requests to the project ([CLI][pr-cli], [lib][pr-lib]) and they were accepted!

I ran a daily cron job for 5 days to run out my daily 10000 file upload limit.
The cron job was:

```bash
cd /home/ubuntu || exit
export GPHOTOS_CLI_TOKENSTORE_KEY="<redacted>"
/home/ubuntu/.go/bin/gphotos-uploader-cli --debug push 2>&1 | tee -a gphotos-uploader-cli-modified.log
```

(Above, the `gphotos-uploader-cli` binary was compiled on the t2.micro machine where the GOPATH aws `/home/ubuntu/.go`.)

The first time you run `gphotos-uploader-cli`, it walks you through the OAuth2 dance with Google and saves a token.
I believe it is valid for 7 days.
Subsequent runs via cron job succeeded without prompting the user.

## Deleting the files off of iCloud.com

This was a scary moment.
Before deleting 200GB of family memories, I did quite a few spot checks on Google Photos by looking at old pictures and videos: fun times!

Then it was time to use the [pyicloud API][pyicloud] directly to delete all the files off iCloud.
Via the browser, we are limited to deleting 1000 files at a time.
Switching to Python instead of Bash, we can do:

```python
from pyicloud import PyiCloudService

api = PyiCloudService("youriclouduser@mac.com")

for p in api.photos.all:
    p.delete()
```

## Conclusion

Using a Linux system in the cloud and a few open source tools, I was able to make better use of the cloud storage I was already paying for and make a backup copy of all my pictures.

The costs for all this in AWS were not obvious to itemize because I was using the same machine for other purposes.
However, it's fair to say that the S3 standard storage tier was more expensive than I thought it would be (about $10/month), so I will delete it or use a different storage tier since I have the Google Photos copy.
Network egress was cheaper than I thought it would be, at a few extra dollars on the monthly AWS bill.

By far, the most expensive item was the EBS volume, which I deleted once I was confident the upload to Google Photos was successful.

I wonder if there's a better way to do all this.
[Upspin][upspin]?.

[photos]: /photos
[icloud]: https://www.icloud.com/
[gphotos]: https://photos.google.com
[aws]: https://aws.amazon.com/
[mount]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html
[icloudpd]: https://github.com/icloud-photos-downloader/icloud_photos_downloader
[gphotos-uploader-cli]: https://github.com/gphotosuploader/gphotos-uploader-cli
[pr-cli]: https://github.com/gphotosuploader/gphotos-uploader-cli/pull/341
[pr-lib]: https://github.com/gphotosuploader/google-photos-api-client-go/pull/75
[pyicloud]: https://github.com/picklepete/pyicloud
[upspin]: https://upspin.io/