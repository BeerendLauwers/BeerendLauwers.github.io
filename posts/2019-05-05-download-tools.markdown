---
title: Useful tools for downloading and verifying bulk (video) datasets
---

## Downloading

I like to archive YouTube channels and playlists I find interesting or entertaining. Usually, [`youtube-dl`]([https://ytdl-org.github.io/youtube-dl/](https://ytdl-org.github.io/youtube-dl/)) does the job admirably in downloading YouTube videos en-masse. 
However, there have been times where channels have been deleted, but someone else archived them and has thrown them in an open directory or on Google Drive, Mega or some other downloading service that I can't be bothered to download one by one. 

This is where [JDownloader 2](http://jdownloader.org/jdownloader2) comes in.
Unfortunately, it is a GUI-centric application, but it's great for mass downloads via these file sharing services: just copy and paste the URL of the Google Drive directory, Mega directory, or just a bunch of URLs into JDownloader's LinkGrabber, and it'll automatically enumerate them and check them for validity. 
Then, you can add them to your download queue.
Unfortunately, download speed limiting during certain times of the day is not built-in, but it is available with the [Scheduler]([http://jdownloader.org/knowledge/wiki/addons/list/scheduler](http://jdownloader.org/knowledge/wiki/addons/list/scheduler)) plugin in a roundabout way: just add a schedule rule that pauses and unpauses your download queue at certain times.

For classic open directories, I still prefer `wget` if it's small and I need it now, or the ripper script + `aria2` combo for larger open directories that I'd like to only download during certain times of the day. 
[See my previous post to find out more](/posts/2018-09-01-emulating-wget-directory-structure.html).

I usually run all of these in a VM on my unRAID box that has access to a share on the drives in the DS4243, so I don't have to move things around later.

## Verifying

Sometimes (especially with `wget`), a download doesn't come through correctly and the video is (partially) corrupted.
I wanted some kind of tool that allowed me to verify the integrity of a video download (yes, I know hashes exist, but they are usually not provided).

Looking around online, I found suggestions to use [ffmpeg](https://superuser.com/a/100290/497182):

```bash
ffmpeg -v error -i file.avi -f null - 2>error.log
```

Because this can take quite a while to run, some suggest to either only run it on the last 60 seconds of the video by adding the `-sseof -60` parameter. It also can still produce false-positives (that is, no errors are reported while they are, in fact, present).

Òthers suggested using `mediainfo` to quickly read out the metadata and get the video's duration:

```bash
mediainfo --Inform="Video;%Duration%" /Media/Movies/moviename.avi
```

This is a lot faster, of course, because it just tries to load the metadata and get some data from it. You'll have to massage the output (which is in milliseconds) and do some eyeballing or filter the data in some other way.

An in-the-middle approach is [using `ffmpeg` to generate thumbnails](https://stackoverflow.com/a/49774856/4737779):

```bash
find . -iname "*.mp4" | while read -r line; do 
  line=`echo "$line" | sed -r 's/^\W+//g'`; 
  echo 'HERE IT IS ==>' "$line"; 
  if ffmpeg -i "$line" -t 2 -r 0.5 %d.jpg; 
    then echo "DONE for" "$line"; 
  else echo "FAILED for" "$line" >>error.log; 
  fi; 
done;
```

## Mass unzipping / unrarring

Sometimes, you have a bunch of zipped-up files, but want to collect their contents into a single directory. 
Using `unrar` (*not* `unrar-free`, which doesn't have the recursive option), it's pretty easy to do this by running `unrar x -r *.rar`.

## Cleaning up unwanted files

Sometimes, there's a bunch of file of a particular file type you don't really care about strewn about in a bunch of directories.
For example, the `aria2`approach I use does have the annoying side effect that an `.aria2` file is placed next to the actual file. Luckily, `find` is useful here:

```bash
find . -name "*.aria2" -type f -delete
```

Similarly, to remove empty directories:

```bash
find . -empty -type d -delete
```
