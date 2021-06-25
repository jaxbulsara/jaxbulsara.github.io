---
layout: post
title: Hello World!
---

Today's the day I got my site up and running. Here's how I did it.

## Environment

I am working on Windows 10 with the Windows Subsystem for Linux (WSL) and Ubuntu installed. I've set up Windows Terminal and VS code to have a complete Linux development environment.

I also have Docker Desktop installed. I enabled `Use the WSL 2 based engine` so that I could use Docker within the WSL terminal. I use Docker to build my site with Jekyll so I don't have to install all of the dependencies myself.

## Buying the domain

I bought the domains `jaybulsara.dev` and `jaybulsara.com` from [Google Domains](https://domains.google.com). During the setup, I added an email server to `jaybulsara.dev`.

I wanted `jaybulsara.dev` to be my main website, so I set up `jaybulsara.com` to forward to the `.dev` address. I found the setting to forward the address under the `Website` tab of the domain management page.

## Setting up GitHub Pages

I used [this guide](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site) to set up the GitHub Page for my account.

Once I created the `jaxbulsara.github.io` repository and enabled GitHub Pages, I cloned the repository to my computer:

```bash
$ git clone https://github.com/jaxbulsara/jaxbulsara.github.io
```

To push changes, you have to create a personal access token on GitHub under [Settings > Developer settings > Personal access tokens](https://github.com/settings/tokens). This will be your password when you log in from the command line.

## Configuring my GitHub page as 'jaybulsara.dev'

To make my own domain, `jaybulsara.dev`, be my GitHub page, I followed [this guide](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site). Since I wanted to use the top-level domain, so I followed the steps under [Configuring an apex domain and the www subdomain variant](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain-and-the-www-subdomain-variant).

The DNS settings in Google Domains are in the `DNS` tab of the domain management page. Under `Resource records > Custom records > Manage custom records`, I created the following records:

![Google domains custom records setup.](/assets/images/2021-06-25-google-domains-dns-records-setup-for-github-pages.png)

Leave the `hostname` for the `A` record blank to indicate the top-level domain. The `hostname` for the `CNAME` record should be `www`.

I think I had to remove and re-add the custom domain on GitHub. I definitely did it wrong a few times. But once the DNS records were in, the custom domain was added successfully. I checked the `Enforce HTTPS` option. This is how the `Pages` setting page looked when I was done:

![GitHub Pages setting page after configuration.](/assets/images/2021-06-25-github-pages-custom-domain-setup-final.png)

Now whenever I go to [jaybulsara.dev](https://jaybulsara.dev), my browser will automatically load my GitHub page.

## Creating the Jekyll site

### Initializing Jekyll

I cloned the `jaxbulsara.github.io` repository to `~/code/personal/jaxbulsara.github.io`. So all of the commands below will be run from that directory.

I wanted to try and use Docker to do everything Jekyll related, so I found this post, [Setting up a GitHub page with Jekyll and a Docker container](https://medium.com/@sebagomez/setting-up-a-github-page-with-jekyll-and-a-docker-container-c712e448649b), by [Sebastian Gomez](https://medium.com/@sebagomez). Since I'm super new to Docker, this was a great help.

I used the following command to initialize the Jekyll page:

```bash
$ docker run --rm -v `pwd`:/srv/jekyll/ jekyll/jekyll jekyll init .
```

Breaking down the command:

- `docker run` - Runs a Docker container.
- `--rm` - Deletes the container when it exits.
- `` -v `pwd`:/srv/jekyll/  ``
  - The `-v` flag mounts a volume on my local machine to a directory in the container
  - `` `pwd`:/srv/jekyll/ `` mounts the current directory (from the subcommand `pwd`) to `/srv/jekyll/` inside the container
- `jekyll/jekyll` - After all the flags, this is the name of the container to run. [jekyll/jekyll](https://hub.docker.com/r/jekyll/jekyll/) is a container that already contains all the Jekyll dependencies you need to work with Jekyll sites.
- `jekyll init .` - This is the command that is run inside the container. `.` resolves to `/srv/jekyll/` since the container [sets its working directory there](https://github.com/envygeeks/jekyll-docker/blob/575057ab5f14055e161b871e75f8295afd571aa9/repos/jekyll/Dockerfile#L191). Then, since I mounted my own directory to `/srv/jekyll/`, the path further resolves to `~/code/personal/jaxbulsara.github.io`. `jekyll init` will set up all of the files and directory it needs for a basic Jekyll site.

### Changing defaults

I tinkered with the site a lot until it looked good and I started writing this blog post. Here, I'll summarize what I did as if I know what I'm doing (I don't) to get the site to this point. This is just the starting point though, and all of it will change in the future.

I deleted the following files from the default Jekyll stuff:

- 404.html
- about.markdown
- The 'welcome-to-jekyll' post
- Gemfile.lock

I then changed the default content of the rest to the following:

`Gemfile`:

```ruby
# file: Gemfile

source "https://rubygems.org"

gem "jekyll", "~> 4.2.0"
gem "minima"
```

`_config.yml`:

```yml
# file: _config.yml

# info
title: Jay Bulsara
author: Jay Bulsara
email: jay@jaybulsara.dev
description: >
  This is my website. If it looks bad, email me about it!

# social links
url: "https://jaybulsara.dev"
github_username: jaxbulsara
rss: rss

# theme
minima:
  date_format: "%b %-d, %Y"

# build
theme: minima
destination: docs
exclude:
  - .liquidrc
  - README.md
  - build.sh
  - serve.sh
  - workspace.code-workspace
plugins:
  - jekyll-feed
  - jekyll-seo-tag
```

`index.md`:

````markdown
<!-- file: index.md -->

---
layout: home
---

# Jay Bulsara

```python
>>> failure == success
True
```
````

### Building the site

I used Docker to build and preview the site on my local computer too. For this, I made three shell scripts:

`shell.sh`
```bash
docker run --rm -v `pwd`:/srv/jekyll/ -p 4000:4000 -it jekyll/jekyll bash
```

`build.sh`
```bash
docker run --rm -v `pwd`:/srv/jekyll/ jekyll/jekyll jekyll build
```

`serve.sh`
```bash
docker run --rm -v `pwd`:/srv/jekyll/ -p 4000:4000 jekyll/jekyll jekyll serve
```

I made them executable with:

```bash
$ chmod u+x {filename}
```

I used `shell.sh` when I was actively developing so that I could work in a bash instance within the container. If I don't need to enter the container, then I run `build.sh` or `serve.sh` whenever I need to rebuild the site or preview it at `http://0.0.0.0:4000`.

Making a new container every time makes all of the dependencies download again. It does take longer, but I like that I will always build the site based on the absolute latest versions of every dependency. In the long run, waiting a little more to build the site is worth it to never have the site become outdated.

When the site is built and it looks alright in the preview, I commit the changes and push the repository back to GitHub to update my GitHub page.