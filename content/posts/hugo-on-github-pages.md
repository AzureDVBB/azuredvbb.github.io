---
title: "Hugo on Github Pages"
date: 2023-01-24T11:54:38+01:00
draft: false
---

With guides it is easy, but it took me a while to google out the answer, so I shall leave the steps to take generating a [HUGO](https://gohugo.io/) website and posting it onto github for future reference.

You will need `git` and `hugo` installed for this.

This guide expects you to be familiar with the command line and Github, as well as having read the quick-start guide on HUGO.

## Step 0. - (LINUX) Make sure you have github `ssh` key setup

#### Step 0.1. - Follow github's guide on it

[Link to the guide, it's written pretty well.](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)

#### Step 0.2. - Make sure `ssh-agent` is running and has the key registered.

Every time you log in to your computer you need to ensure the `ssh-agent` is both running and has your own **ssh key** added with the command:

```bash
ssh-add /path/to/your/keyfile
```

**Failing to do this will cause errors when trying to use** `git push` **or** `git clone` **commands (the latter only in private repositories).**

## Step 1. - Setting up the Github Repository

There are two options for making your website with github pages, you can either make a 'catch-all' website with your user name, or you can make a website for every single project.

#### Option 1.1. - 'catch-all' website

Start by creating a special repository named `<username>.github.io` substituting `<username>` with your github username, all lower-case (as it is case insensitive).

This special repository will appear for people visiting `https://<username>.github.io/` directly.

> Notes:
> 
> 1. The website was originally ending with `.com` but later on began redirecting to `.io`, however recently it no longer auto-redirects and displays an error message.
>
> 1. This special repository HAS TO be public for github pages to be active

#### Option 1.2. - Project specific website

Create a repository on github as normal, **make sure it's public**, or you can use an existing repository.

#### Step 1.3. - Settin github pages

Now that you have a repository visit it on github, then go to **settings** and choose **pages**.

Here you will be able to select the folder in which the website will be searched for, select `docs`.

> Note:
> 
> Github pages need to be either in the project root folder `/` or in a docs folder `/docs/` for it to be rendered, this is a limitation as of this writing.


## Step 2. - Setting up the repository locally

Now that we have the repository on github, we need to get a local copy of it so we can set up the rest.

#### Step 2.1. - `clone` the repository

You can easily clone the repository in a new folder.
Firstly go to the Github website and select your repository, there is a green `CODE` button on the top right, we are going to need the **SSH** link from there.

With that we can use the following command to clone the repository, making a local copy of it:

```bash
git clone <THE-LINK-YOU-JUST-COPIED>
```

Ofcourse replacing **\<THE-LINK-YOU-JUST-COPIED\>** with the actual link.

#### Step 2.2. - Creating a new HUGO project

Navigate into the newly downloaded folder and use the following command:

```bash
hugo new site ./ --force
```

This creates a new empty website in the current folder, using the `--force` will make it create a site in a folder that is not empty.

#### Step 2.3. - Making sure HUGO outputs into the `docs` folder

By default HUGO will build the website in the `/public` folder, which is not what we want for github pages.

To remedy this, edit the `config.toml` file to include the following line:

```toml
publishDir = "docs"
```

#### Step 2.4. - Ensuring hugo's url scheme is correct

Now we need to make sure the base URL from which HUGO will generate the website-links is correct.

To do this we need to change the `config.toml` file once more depending on what option we took.

For **Option 1.1.** ensure the following line:

```toml
baseURL = "https://<USERNAME>.github.io/"
```

For **Option 1.2.** ensure the following:

```toml
baseURL = "https://<USERNAME>.github.io/<REPOSITORY-NAME>/"
```

> Note:
>
> The trailing `/` is vital in making sure it is working correctly.

## Step 3. - Setting up a theme for HUGO

There are literally hundreds of themes for HUGO and it doesn't actually ship with one, so it is up to us to find one.

#### Step 3.1. - Selecting and downloading themes

Luckily it is fairly simple, just go to the [HUGO theme list](https://themes.gohugo.io/) and pick one.

Once done you have to add it to your own website with the following command:

```bash
git submodule add --depth=1 <LINK-TO-THEME> themes/<THEME-NAME>
```

This will make sure that you not only clone it, but will be kept up to date and github itself can make use of it later on.
**Do note that each theme has it's own set of requirements and configurations that you yourself have to figure out.**
**Also you may omit** `--depth=1` **as it only reduces the amount of 'history' that will be downloaded from everything to the latest commit.**

#### Step 3.2. - Updating themes

To update themes added via submodule, you can use the following command:

```bash
git submodule update --recursive --remote
```

#### Step 3.3. - Selecting the theme for HUGO

To apply a specific theme, you need to edit your `config.toml` file to have this line:

```toml
theme = "<THEME-NAME>"
```

The theme name there is the folder in which the theme lives, normally under `themes/THEMENAME`.

## Step 4. - Add a new post and test locally

Now that we have HUGO more or less set up, we can test if we configured it correctly.

To do this first run `hugo new test.md` which will create a new file under the `content` folder called `test.md`.
Edit this file to have some text in it and save it.

Then we can run `hugo server --buildDrafts` and visit the shown local address to see our website.

>Note:
>
> The markdown files need to have `draft: false` at the top so that it is built normally without the `--buildDrafts` flag.

#### Step 5. - Build and publish the website

Finally we can build the website statically by using the following command:

```bash
hugo
```

Then we can simply use the following commands to upload the changes to our repository:

```bash
git add .
git commit -m "Write your message here."
git push
```
