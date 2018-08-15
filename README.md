# King Media Script Vulnerabilities
King Media is a premium CMS that allows you create a website where share all type multimedia content like images, videos, etc... I ve found and reported some critical vulnerabilities and here I explain it.

## King Media > 2.x/3.x/4.x Blind RCE
### Information
Date: 15 Aug 2018
Author: Efren Diaz (https://twitter.com/elefr3n)
Vendor: Redkings / Kingthemes (https://codecanyon.net/user/redkings)
Reported to vendor: 11 Jun 2018
Software Link: https://codecanyon.net/item/king-media-video-image-upload-and-share/7877877
Version: from 2.0 to 4.1 (latest)
CVE: None

### Vulnerability description
The RCE is in the file videoupload.php, when the script finish the upload, execs a command with the ffmepg library to get a video thumbnail and here his where we can inject our command. The script receives the file with the $_FILES['myfile'] parameter, first checks that the request come from AJAX (HTTP_X_REQUESTED_WIDTH), second checks that the file type is 'video/mp4', and then uploads the video (nothing that we can not bypass). When the upload is finished calls with shell_exec a command to get the video thumbnail with /usr/bin/ffmpeg, we can see in the next line:
```
$cmd = "$ffmpeg -i $video; ls #  -deinterlace -an -ss $second -t 00:00:01 -r 1 -y -vcodec mjpeg -f mjpeg $image 2>&1";
```
We can send a video with our command in the file extension (example: foo.mp4;command #), but php gets the file name from $_FILES['myfile']['name'], then if our command contents a slash or other invalid filename character, will be removed. To avoid this problem, we need encode our command in hexadecimal and execute with a "custom payload" like this:
```
evil.mp4;echo $hex_encoded_command | xxd -r -p | sh #
```
With these payload our command will be executed correctly, but we dont get the command result because the return of shell_exec function is not printed, if we want get the command result we can send to a file in a directory where we have permissions (uploads for example)

### Exploit
exploit_blind_rce.php

## KingMedia 1.x/2.x/3.x/4.1 - Arbitrary File Upload

### Information
Date: 15 Aug 2018
Author: Efren Diaz (https://twitter.com/elefr3n)
Greets: **Daniel F. Rodriguez** aka dj.thd
Vendor: Redkings / Kingthemes (https://codecanyon.net/user/redkings)
Reported to vendor: 11 Jun 2018
Software Link: https://codecanyon.net/item/king-media-video-image-upload-and-share/7877877
Version: from 1.0 to 4.1 (Lastest)
CVE: None

### Vulnerability description
The vulnerability resides in processupload.php, this file process all image uploads. In resume, the script checks that the file is a valid image, but dont check the extension. Other big issue of the script is that works with unauthenticated request.

-First checks that the execution comes from an AJAX request (HTTP_X_REQUESTED_WITH).
-Second check that the filetype is valid image (image/png | image/gif | image/jpeg | image/pjpeg).
-Third executes a custom function called resizeImage() that basically do a thumbnail with some GD library php functions, if the thumbnail creation fails the script stop working, you need upload a image file with a small php code inside the metadata (ex: <?php shell_exec($_POST['command']); ?>) and it will saved on http://site.com/uploads/{your_file_name}-{random_numbers}.{you_file_extension}

Note: For any reason, I haved found some versions of the CMS with a rule in the .htaccess file to disable the ".php" execution on the uploads directory, no problem we can try with other extensions (let your imagination run wild !)

### Exploit
exploit_arbitrary_file_upload.php


## King Media 1.9/2.x/3.x/4.x Arbitrary file delete
### Information
Date: 15 Aug 2018
Author: Efren Diaz (https://twitter.com/elefr3n)
Vendor: Redkings / Kingthemes (https://codecanyon.net/user/redkings)
Reported to vendor: 11 Jun 2018
Software Link: https://codecanyon.net/item/king-media-video-image-upload-and-share/7877877
Version: 4.0, 4.1
CVE: None
### Vulnerability Description:
The vulnerability resides in the file multipledelete.php file, this file receive a string in the parameter $_POST['name'], and try if exists the files and delete the next patterns:
```
uploads/CURRENT_YEAR/CURRENT_MONTH/{$_POST['name']}
uploads/CURRENT_YEAR/CURRENT_MONTH/thumb_/{$_POST['name']}
```
First, the script dont checks if we are athenticated, second, to avoid that you can upload directory, haves the next code in the line 10:
```
$fileName=str_replace("..",".",$fileName); //required. if somebody is trying parent folder files
```
To avoid this check is than easy like put four points, for example if we put ..../..../..../x.php in the $_POST['name'] parameter, we can delete a php file in the application path.

### Exploit
Do it yourself :D

