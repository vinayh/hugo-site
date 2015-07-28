+++
categories = []
date = "2015-07-28T14:49:41-04:00"
description = ""
keywords = []
title = "Git hook to deploy static sites to S3"

+++
It's a surprisingly simple process to setup S3 syncing with a static site generator with source that you sync using Git. This will allow you to track revision history on your actual writing of posts, etc. while having the generated site automatically deployed on S3 on each push.

All you really need is:

1. A way to upload a folder to S3 (I use the excellent `s3cmd`)
2. A hook that triggers the upload on every `git push`

For example, if your static site generator is `hugo`, with destination dir `public` (like Hugo by default) and your S3 bucket is called `<example_bucket>`, create a new shell script called `pre-push` in the `.git/hooks` directory of your site source as follows:
```bash
#!/bin/bash
set -e
echo -e "\t Building site..."
hugo
echo -e "\t Successfully generated site"
s3cmd sync --recursive public/ s3://<example_bucket>
echo -e "\t Successfully deployed to S3"
```
Mark the script as executable using `chmod +x .git/hooks/pre-push` and try pushing to your Git server. If everything goes well, `s3cmd` will automatically be triggered and will perform a `sync` operation to upload changed files to S3. Assuming you have your S3 settings configured for a public web server (static website hosting is enabled, index and error documents are set correctly, public read access is permitted) then you should be able to see the contents of your `public` dir (ie. your website) at the URL assigned to your bucket.

Note: This S3 bucket policy will enable public read access:
```json
{
	"Statement": [
		{
			"Sid": "AllowPublicRead",
			"Effect": "Allow",
			"Principal": {
				"AWS": "*"
			},
			"Action": "s3:GetObject",
			"Resource": "arn:aws:s3:::<example_bucket>/*"
		}
	]
}
```

Courtesy of [Nathan Moos' blog post](https://moosingin3space.github.io/post/git-hooks-hugo/) outlining this process for GitHub Pages, which requires a nested repository with your public files.