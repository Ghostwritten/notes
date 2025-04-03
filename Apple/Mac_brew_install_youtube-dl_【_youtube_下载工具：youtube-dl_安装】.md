![](https://i-blog.csdnimg.cn/blog_migrate/86c27fe921e6c14d162bb4534c8eccf9.png)

## 1. 简介
youtube-dl - 从youtube.com或其他视频平台下载视频

- [https://github.com/ytdl-org/youtube-dl](https://github.com/ytdl-org/youtube-dl)

## 2. 预备

- [安装并配置 git](https://ghostwritten.blog.csdn.net/article/details/105240194)
- [安装 brew](https://blog.csdn.net/xixihahalelehehe/article/details/129151854)
## 3. 安装
```bash
MacBook-Pro ~ % brew install youtube-dl
Warning: youtube-dl has been deprecated because it has a failing test since forever and no new release since 2021!
==> Downloading https://ghcr.io/v2/homebrew/core/youtube-dl/manifests/2021.12.17-4
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/3e5fe850d76ce50aa781115af8c54ae4d2e1ca78acfb239ba5ccfeef12dc9fa7--youtube-dl-2021.12.17-4.bottle_manifest.json
==> Fetching dependencies for youtube-dl: mpdecimal, ca-certificates, openssl@3, readline, sqlite, xz and python@3.12
==> Downloading https://ghcr.io/v2/homebrew/core/mpdecimal/manifests/2.5.1
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/f367c2ee08c56b88be0662703a8e4275f8657608a268c8c44e845154b0cea543--mpdecimal-2.5.1.bottle_manifest.json
==> Fetching mpdecimal
==> Downloading https://ghcr.io/v2/homebrew/core/mpdecimal/blobs/sha256:95079d5b0a5114d1f3ccbe5f036a4729f3731d062ea5fc9d8c61241ddc22f7e2
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/2ce90c561ee2bedfebafe76ec3982114b68a77808932e731fd513cef0d8afafd--mpdecimal--2.5.1.arm64_sonoma.bottle.tar.gz
==> Downloading https://ghcr.io/v2/homebrew/core/ca-certificates/manifests/2023-12-12
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/13aa86e429e05d02a76005d1881eaf625091a5ac4dc7d7674c706d12ba48796a--ca-certificates-2023-12-12.bottle_manifest.json
==> Fetching ca-certificates
==> Downloading https://ghcr.io/v2/homebrew/core/ca-certificates/blobs/sha256:5c99ffd0861f01adc19cab495027024f7d890e42a9e7b689706b85c8e2b9c9b3
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/5505d583ff92cdbc68f67ea2ab46070695997373cf79e708dd7fa7c352265322--ca-certificates--2023-12-12.all.bottle.tar.gz
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/manifests/3.2.0_1
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/8e5415de690efd057f74775ab4b808fed9a50bf29c34ee9cb52118d189ef73a9--openssl@3-3.2.0_1.bottle_manifest.json
==> Fetching openssl@3
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/blobs/sha256:6519a6ff8e3e10f921ba8ec7ac00a67afc80e346f262115956b3c826541899f5
############################################################################################################################################################################################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/readline/manifests/8.2.7
############################################################################################################################################################################################################################################## 100.0%
==> Fetching readline
==> Downloading https://ghcr.io/v2/homebrew/core/readline/blobs/sha256:4b08134e70e90a968bf82227fbec6861b07fdf630e7ab66e6effe95b6721cf36
############################################################################################################################################################################################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/sqlite/manifests/3.44.2
############################################################################################################################################################################################################################################## 100.0%
==> Fetching sqlite
==> Downloading https://ghcr.io/v2/homebrew/core/sqlite/blobs/sha256:0e4b89c838f4523296c2250d68a1b5f74196ea2ed15bd5021afeb143ede57ebb
############################################################################################################################################################################################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/xz/manifests/5.4.5
############################################################################################################################################################################################################################################## 100.0%
==> Fetching xz
==> Downloading https://ghcr.io/v2/homebrew/core/xz/blobs/sha256:ea718d075502d4457709da89e8e424aa22cfa19d2e16088c43feb87f7c2c477a
############################################################################################################################################################################################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/python/3.12/manifests/3.12.1
############################################################################################################################################################################################################################################## 100.0%
==> Fetching python@3.12
==> Downloading https://ghcr.io/v2/homebrew/core/python/3.12/blobs/sha256:cb1129c121cf124512dde5a7002d9aa10b671c5cb6c86402392f2c6c2bfe26e9
############################################################################################################################################################################################################################################## 100.0%
==> Fetching youtube-dl
==> Downloading https://ghcr.io/v2/homebrew/core/youtube-dl/blobs/sha256:86ad6daf8ba59b37855706201030abfdfa6a334d66ee80ee6351212baa1fccaa
############################################################################################################################################################################################################################################## 100.0%
==> Installing dependencies for youtube-dl: mpdecimal, ca-certificates, openssl@3, readline, sqlite, xz and python@3.12
==> Installing youtube-dl dependency: mpdecimal
==> Downloading https://ghcr.io/v2/homebrew/core/mpdecimal/manifests/2.5.1
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/f367c2ee08c56b88be0662703a8e4275f8657608a268c8c44e845154b0cea543--mpdecimal-2.5.1.bottle_manifest.json
==> Pouring mpdecimal--2.5.1.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/mpdecimal/2.5.1: 71 files, 2.1MB
==> Installing youtube-dl dependency: ca-certificates
==> Downloading https://ghcr.io/v2/homebrew/core/ca-certificates/manifests/2023-12-12
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/13aa86e429e05d02a76005d1881eaf625091a5ac4dc7d7674c706d12ba48796a--ca-certificates-2023-12-12.bottle_manifest.json
==> Pouring ca-certificates--2023-12-12.all.bottle.tar.gz
==> Regenerating CA certificate bundle from keychain, this may take a while...
🍺  /opt/homebrew/Cellar/ca-certificates/2023-12-12: 3 files, 226.7KB
==> Installing youtube-dl dependency: openssl@3
==> Downloading https://ghcr.io/v2/homebrew/core/openssl/3/manifests/3.2.0_1
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/8e5415de690efd057f74775ab4b808fed9a50bf29c34ee9cb52118d189ef73a9--openssl@3-3.2.0_1.bottle_manifest.json
==> Pouring openssl@3--3.2.0_1.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/openssl@3/3.2.0_1: 6,805 files, 31.9MB
==> Installing youtube-dl dependency: readline
==> Downloading https://ghcr.io/v2/homebrew/core/readline/manifests/8.2.7
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/14125f7fa4b49853f76160b864f58379d90e52833ffeb8bd0643609bcd7f02a7--readline-8.2.7.bottle_manifest.json
==> Pouring readline--8.2.7.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/readline/8.2.7: 50 files, 1.7MB
==> Installing youtube-dl dependency: sqlite
==> Downloading https://ghcr.io/v2/homebrew/core/sqlite/manifests/3.44.2
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/1e19b0e6cb419159b49df50f3595c3b44e67ec5f5d1110ece66a9785df57a844--sqlite-3.44.2.bottle_manifest.json
==> Pouring sqlite--3.44.2.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/sqlite/3.44.2: 11 files, 4.7MB
==> Installing youtube-dl dependency: xz
==> Downloading https://ghcr.io/v2/homebrew/core/xz/manifests/5.4.5
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/4e81fda476fb634a7e1ac650019bfe768a65d6c387015992df4cd75adf9b3fce--xz-5.4.5.bottle_manifest.json
==> Pouring xz--5.4.5.arm64_sonoma.bottle.tar.gz
🍺  /opt/homebrew/Cellar/xz/5.4.5: 163 files, 2.6MB
==> Installing youtube-dl dependency: python@3.12
==> Downloading https://ghcr.io/v2/homebrew/core/python/3.12/manifests/3.12.1
Already downloaded: /Users/zongxun/Library/Caches/Homebrew/downloads/b13664874124f24d86fbf576412860426d44202df0a3a858c00900d566c12ea4--python@3.12-3.12.1.bottle_manifest.json
==> Pouring python@3.12--3.12.1.arm64_sonoma.bottle.tar.gz
==> /opt/homebrew/Cellar/python@3.12/3.12.1/bin/python3.12 -Im ensurepip
==> /opt/homebrew/Cellar/python@3.12/3.12.1/bin/python3.12 -Im pip install -v --no-index --upgrade --isolated --target=/opt/homebrew/lib/python3.12/site-packages /opt/homebrew/Cellar/python@3.12/3.12.1/Frameworks/Python.framework/Versions/3.12/l
🍺  /opt/homebrew/Cellar/python@3.12/3.12.1: 3,225 files, 65.3MB
==> Installing youtube-dl
==> Pouring youtube-dl--2021.12.17.arm64_sonoma.bottle.4.tar.gz
==> Caveats
The current youtube-dl version has many unresolved issues.
Upstream have not tagged a new release since 2021.

Please use yt-dlp instead.
==> Summary
🍺  /opt/homebrew/Cellar/youtube-dl/2021.12.17: 846 files, 5.9MB
==> Running `brew cleanup youtube-dl`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
==> Caveats
==> youtube-dl
The current youtube-dl version has many unresolved issues.
Upstream have not tagged a new release since 2021.

Please use yt-dlp instead.
```

## 4. 命令

```bash
MacBook-Pro ~ % youtube-dl -h
Usage: youtube-dl [OPTIONS] URL [URL...]

Options:
  General Options:
    -h, --help                           Print this help text and exit
    --version                            Print program version and exit
    -U, --update                         Update this program to latest version. Make sure that you have sufficient permissions (run with sudo if needed)
    -i, --ignore-errors                  Continue on download errors, for example to skip unavailable videos in a playlist
    --abort-on-error                     Abort downloading of further videos (in the playlist or the command line) if an error occurs
    --dump-user-agent                    Display the current browser identification
    --list-extractors                    List all supported extractors
    --extractor-descriptions             Output descriptions of all supported extractors
    --force-generic-extractor            Force extraction to use the generic extractor
    --default-search PREFIX              Use this prefix for unqualified URLs. For example "gvsearch2:" downloads two videos from google videos for youtube-dl "large apple". Use the value "auto" to let youtube-dl guess ("auto_warning" to emit a
                                         warning when guessing). "error" just throws an error. The default value "fixup_error" repairs broken URLs, but emits an error if this is not possible instead of searching.
    --ignore-config                      Do not read configuration files. When given in the global configuration file /etc/youtube-dl.conf: Do not read the user configuration in ~/.config/youtube-dl/config (%APPDATA%/youtube-dl/config.txt on
                                         Windows)
    --config-location PATH               Location of the configuration file; either the path to the config or its containing directory.
    --flat-playlist                      Do not extract the videos of a playlist, only list them.
    --mark-watched                       Mark videos watched (YouTube only)
    --no-mark-watched                    Do not mark videos watched (YouTube only)
    --no-color                           Do not emit color codes in output

  Network Options:
    --proxy URL                          Use the specified HTTP/HTTPS/SOCKS proxy. To enable SOCKS proxy, specify a proper scheme. For example socks5://127.0.0.1:1080/. Pass in an empty string (--proxy "") for direct connection
    --socket-timeout SECONDS             Time to wait before giving up, in seconds
    --source-address IP                  Client-side IP address to bind to
    -4, --force-ipv4                     Make all connections via IPv4
    -6, --force-ipv6                     Make all connections via IPv6

  Geo Restriction:
    --geo-verification-proxy URL         Use this proxy to verify the IP address for some geo-restricted sites. The default proxy specified by --proxy (or none, if the option is not present) is used for the actual downloading.
    --geo-bypass                         Bypass geographic restriction via faking X-Forwarded-For HTTP header
    --no-geo-bypass                      Do not bypass geographic restriction via faking X-Forwarded-For HTTP header
    --geo-bypass-country CODE            Force bypass geographic restriction with explicitly provided two-letter ISO 3166-2 country code
    --geo-bypass-ip-block IP_BLOCK       Force bypass geographic restriction with explicitly provided IP block in CIDR notation

  Video Selection:
    --playlist-start NUMBER              Playlist video to start at (default is 1)
    --playlist-end NUMBER                Playlist video to end at (default is last)
    --playlist-items ITEM_SPEC           Playlist video items to download. Specify indices of the videos in the playlist separated by commas like: "--playlist-items 1,2,5,8" if you want to download videos indexed 1, 2, 5, 8 in the playlist. You
                                         can specify range: "--playlist-items 1-3,7,10-13", it will download the videos at index 1, 2, 3, 7, 10, 11, 12 and 13.
    --match-title REGEX                  Download only matching titles (regex or caseless sub-string)
    --reject-title REGEX                 Skip download for matching titles (regex or caseless sub-string)
    --max-downloads NUMBER               Abort after downloading NUMBER files
    --min-filesize SIZE                  Do not download any videos smaller than SIZE (e.g. 50k or 44.6m)
    --max-filesize SIZE                  Do not download any videos larger than SIZE (e.g. 50k or 44.6m)
    --date DATE                          Download only videos uploaded in this date
    --datebefore DATE                    Download only videos uploaded on or before this date (i.e. inclusive)
    --dateafter DATE                     Download only videos uploaded on or after this date (i.e. inclusive)
    --min-views COUNT                    Do not download any videos with less than COUNT views
    --max-views COUNT                    Do not download any videos with more than COUNT views
    --match-filter FILTER                Generic video filter. Specify any key (see the "OUTPUT TEMPLATE" for a list of available keys) to match if the key is present, !key to check if the key is not present, key > NUMBER (like "comment_count >
                                         12", also works with >=, <, <=, !=, =) to compare against a number, key = 'LITERAL' (like "uploader = 'Mike Smith'", also works with !=) to match against a string literal and & to require multiple
                                         matches. Values which are not known are excluded unless you put a question mark (?) after the operator. For example, to only match videos that have been liked more than 100 times and disliked less than 50
                                         times (or the dislike functionality is not available at the given service), but who also have a description, use --match-filter "like_count > 100 & dislike_count <? 50 & description" .
    --no-playlist                        Download only the video, if the URL refers to a video and a playlist.
    --yes-playlist                       Download the playlist, if the URL refers to a video and a playlist.
    --age-limit YEARS                    Download only videos suitable for the given age
    --download-archive FILE              Download only videos not listed in the archive file. Record the IDs of all downloaded videos in it.
    --include-ads                        Download advertisements as well (experimental)

  Download Options:
    -r, --limit-rate RATE                Maximum download rate in bytes per second (e.g. 50K or 4.2M)
    -R, --retries RETRIES                Number of retries (default is 10), or "infinite".
    --fragment-retries RETRIES           Number of retries for a fragment (default is 10), or "infinite" (DASH, hlsnative and ISM)
    --skip-unavailable-fragments         Skip unavailable fragments (DASH, hlsnative and ISM)
    --abort-on-unavailable-fragment      Abort downloading when some fragment is not available
    --keep-fragments                     Keep downloaded fragments on disk after downloading is finished; fragments are erased by default
    --buffer-size SIZE                   Size of download buffer (e.g. 1024 or 16K) (default is 1024)
    --no-resize-buffer                   Do not automatically adjust the buffer size. By default, the buffer size is automatically resized from an initial value of SIZE.
    --http-chunk-size SIZE               Size of a chunk for chunk-based HTTP downloading (e.g. 10485760 or 10M) (default is disabled). May be useful for bypassing bandwidth throttling imposed by a webserver (experimental)
    --playlist-reverse                   Download playlist videos in reverse order
    --playlist-random                    Download playlist videos in random order
    --xattr-set-filesize                 Set file xattribute ytdl.filesize with expected file size
    --hls-prefer-native                  Use the native HLS downloader instead of ffmpeg
    --hls-prefer-ffmpeg                  Use ffmpeg instead of the native HLS downloader
    --hls-use-mpegts                     Use the mpegts container for HLS videos, allowing to play the video while downloading (some players may not be able to play it)
    --external-downloader COMMAND        Use the specified external downloader. Currently supports aria2c,avconv,axel,curl,ffmpeg,httpie,wget
    --external-downloader-args ARGS      Give these arguments to the external downloader

  Filesystem Options:
    -a, --batch-file FILE                File containing URLs to download ('-' for stdin), one URL per line. Lines starting with '#', ';' or ']' are considered as comments and ignored.
    --id                                 Use only video ID in file name
    -o, --output TEMPLATE                Output filename template, see the "OUTPUT TEMPLATE" for all the info
    --output-na-placeholder PLACEHOLDER  Placeholder value for unavailable meta fields in output filename template (default is "NA")
    --autonumber-start NUMBER            Specify the start value for %(autonumber)s (default is 1)
    --restrict-filenames                 Restrict filenames to only ASCII characters, and avoid "&" and spaces in filenames
    -w, --no-overwrites                  Do not overwrite files
    -c, --continue                       Force resume of partially downloaded files. By default, youtube-dl will resume downloads if possible.
    --no-continue                        Do not resume partially downloaded files (restart from beginning)
    --no-part                            Do not use .part files - write directly into output file
    --no-mtime                           Do not use the Last-modified header to set the file modification time
    --write-description                  Write video description to a .description file
    --write-info-json                    Write video metadata to a .info.json file
    --write-annotations                  Write video annotations to a .annotations.xml file
    --load-info-json FILE                JSON file containing the video information (created with the "--write-info-json" option)
    --cookies FILE                       File to read cookies from and dump cookie jar in
    --cache-dir DIR                      Location in the filesystem where youtube-dl can store some downloaded information permanently. By default $XDG_CACHE_HOME/youtube-dl or ~/.cache/youtube-dl . At the moment, only YouTube player files (for
                                         videos with obfuscated signatures) are cached, but that may change.
    --no-cache-dir                       Disable filesystem caching
    --rm-cache-dir                       Delete all filesystem cache files

  Thumbnail Options:
    --write-thumbnail                    Write thumbnail image to disk
    --write-all-thumbnails               Write all thumbnail image formats to disk
    --list-thumbnails                    Simulate and list all available thumbnail formats

  Verbosity / Simulation Options:
    -q, --quiet                          Activate quiet mode
    --no-warnings                        Ignore warnings
    -s, --simulate                       Do not download the video and do not write anything to disk
    --skip-download                      Do not download the video
    -g, --get-url                        Simulate, quiet but print URL
    -e, --get-title                      Simulate, quiet but print title
    --get-id                             Simulate, quiet but print id
    --get-thumbnail                      Simulate, quiet but print thumbnail URL
    --get-description                    Simulate, quiet but print video description
    --get-duration                       Simulate, quiet but print video length
    --get-filename                       Simulate, quiet but print output filename
    --get-format                         Simulate, quiet but print output format
    -j, --dump-json                      Simulate, quiet but print JSON information. See the "OUTPUT TEMPLATE" for a description of available keys.
    -J, --dump-single-json               Simulate, quiet but print JSON information for each command-line argument. If the URL refers to a playlist, dump the whole playlist information in a single line.
    --print-json                         Be quiet and print the video information as JSON (video is still being downloaded).
    --newline                            Output progress bar as new lines
    --no-progress                        Do not print progress bar
    --console-title                      Display progress in console titlebar
    -v, --verbose                        Print various debugging information
    --dump-pages                         Print downloaded pages encoded using base64 to debug problems (very verbose)
    --write-pages                        Write downloaded intermediary pages to files in the current directory to debug problems
    --print-traffic                      Display sent and read HTTP traffic
    -C, --call-home                      Contact the youtube-dl server for debugging
    --no-call-home                       Do NOT contact the youtube-dl server for debugging

  Workarounds:
    --encoding ENCODING                  Force the specified encoding (experimental)
    --no-check-certificate               Suppress HTTPS certificate validation
    --prefer-insecure                    Use an unencrypted connection to retrieve information about the video. (Currently supported only for YouTube)
    --user-agent UA                      Specify a custom user agent
    --referer URL                        Specify a custom referer, use if the video access is restricted to one domain
    --add-header FIELD:VALUE             Specify a custom HTTP header and its value, separated by a colon ':'. You can use this option multiple times
    --bidi-workaround                    Work around terminals that lack bidirectional text support. Requires bidiv or fribidi executable in PATH
    --sleep-interval SECONDS             Number of seconds to sleep before each download when used alone or a lower bound of a range for randomized sleep before each download (minimum possible number of seconds to sleep) when used along with
                                         --max-sleep-interval.
    --max-sleep-interval SECONDS         Upper bound of a range for randomized sleep before each download (maximum possible number of seconds to sleep). Must only be used along with --min-sleep-interval.

  Video Format Options:
    -f, --format FORMAT                  Video format code, see the "FORMAT SELECTION" for all the info
    --all-formats                        Download all available video formats
    --prefer-free-formats                Prefer free video formats unless a specific one is requested
    -F, --list-formats                   List all available formats of requested videos
    --youtube-skip-dash-manifest         Do not download the DASH manifests and related data on YouTube videos
    --merge-output-format FORMAT         If a merge is required (e.g. bestvideo+bestaudio), output to given container format. One of mkv, mp4, ogg, webm, flv. Ignored if no merge is required

  Subtitle Options:
    --write-sub                          Write subtitle file
    --write-auto-sub                     Write automatically generated subtitle file (YouTube only)
    --all-subs                           Download all the available subtitles of the video
    --list-subs                          List all available subtitles for the video
    --sub-format FORMAT                  Subtitle format, accepts formats preference, for example: "srt" or "ass/srt/best"
    --sub-lang LANGS                     Languages of the subtitles to download (optional) separated by commas, use --list-subs for available language tags

  Authentication Options:
    -u, --username USERNAME              Login with this account ID
    -p, --password PASSWORD              Account password. If this option is left out, youtube-dl will ask interactively.
    -2, --twofactor TWOFACTOR            Two-factor authentication code
    -n, --netrc                          Use .netrc authentication data
    --video-password PASSWORD            Video password (vimeo, youku)

  Adobe Pass Options:
    --ap-mso MSO                         Adobe Pass multiple-system operator (TV provider) identifier, use --ap-list-mso for a list of available MSOs
    --ap-username USERNAME               Multiple-system operator account login
    --ap-password PASSWORD               Multiple-system operator account password. If this option is left out, youtube-dl will ask interactively.
    --ap-list-mso                        List all supported multiple-system operators

  Post-processing Options:
    -x, --extract-audio                  Convert video files to audio-only files (requires ffmpeg/avconv and ffprobe/avprobe)
    --audio-format FORMAT                Specify audio format: "best", "aac", "flac", "mp3", "m4a", "opus", "vorbis", or "wav"; "best" by default; No effect without -x
    --audio-quality QUALITY              Specify ffmpeg/avconv audio quality, insert a value between 0 (better) and 9 (worse) for VBR or a specific bitrate like 128K (default 5)
    --recode-video FORMAT                Encode the video to another format if necessary (currently supported: mp4|flv|ogg|webm|mkv|avi)
    --postprocessor-args ARGS            Give these arguments to the postprocessor
    -k, --keep-video                     Keep the video file on disk after the post-processing; the video is erased by default
    --no-post-overwrites                 Do not overwrite post-processed files; the post-processed files are overwritten by default
    --embed-subs                         Embed subtitles in the video (only for mp4, webm and mkv videos)
    --embed-thumbnail                    Embed thumbnail in the audio as cover art
    --add-metadata                       Write metadata to the video file
    --metadata-from-title FORMAT         Parse additional metadata like song title / artist from the video title. The format syntax is the same as --output. Regular expression with named capture groups may also be used. The parsed parameters
                                         replace existing values. Example: --metadata-from-title "%(artist)s - %(title)s" matches a title like "Coldplay - Paradise". Example (regex): --metadata-from-title "(?P<artist>.+?) - (?P<title>.+)"
    --xattrs                             Write metadata to the video file's xattrs (using dublin core and xdg standards)
    --fixup POLICY                       Automatically correct known faults of the file. One of never (do nothing), warn (only emit a warning), detect_or_warn (the default; fix file if we can, warn otherwise)
    --prefer-avconv                      Prefer avconv over ffmpeg for running the postprocessors
    --prefer-ffmpeg                      Prefer ffmpeg over avconv for running the postprocessors (default)
    --ffmpeg-location PATH               Location of the ffmpeg/avconv binary; either the path to the binary or its containing directory.
    --exec CMD                           Execute a command on the file after downloading and post-processing, similar to find's -exec syntax. Example: --exec 'adb push {} /sdcard/Music/ && rm {}'
    --convert-subs FORMAT                Convert the subtitles to other format (currently supported: srt|ass|vtt|lrc)
```

## 5. 测试

```bash
MacBook-Pro Movies % mkdir youtube && cd youtube
MacBook-Pro youtube % youtube-dl 'https://www.youtube.com/watch?v=zBAxiQi2nPc'
[youtube] zBAxiQi2nPc: Downloading webpage
WARNING: unable to extract uploader id; please report this issue on https://yt-dl.org/bug . Make sure you are using the latest version; see  https://yt-dl.org/update  on how to update. Be sure to call youtube-dl with the --verbose flag and include its complete output.
[download] Destination: NVIDIA REFUSED To Send Us This - NVIDIA A100-zBAxiQi2nPc.mp4
[download] 100% of 98.50MiB in 39:29
```
下载 98.50MB 视频竟然花费了 40分钟，效率相当低。需要参数改进。
