# traphic

Forked from: https://github.com/zerodiff/traphic

This repo is an extension to [arcanist](https://github.com/phacility/arcanist) to enable GitHub-integrated continuous
integration services (e.g. Codeship, CircleCI) to run against [Phabricator](http://phabricator.org/) diffs.

## What It Does

Traphic adds a post-diff and post-land hook to arcanist.  The post-diff hook pushes the diff as a remote branch to
GitHub, letting continuous integration services handle the diff. The post-land hook prunes the remote branch after
landing.

## How To Use It

Step 0: Make sure you're using [Blueprint's forked version](https://github.com/calblueprint/libphutil)  of libphutil.
You should have cloned this version when setting up Phabricator for your project.

First, clone the Traphic repo to your preferred location.  Next, export an environment variable from your `.bashrc`
that points to the location of the `/traphic` subdirectory within the `Traphic` repo:

    export TRAPHIC_PATH=/path/to/Traphic/traphic

Make sure the path is absolute.  Then, add the following to your projects's `.arcconfig':

```
"load" : [
  "$TRAPHIC_PATH"
],
"arcanist_configuration" : "TraphicConfiguration"

```

## Phabricator Settings

Change Diffusion settings to track only branches that are *not* your diff branches.  All Traphic brances are prefixed
with `TR_D`, so set *Track Only* to `regexp(/^(?!TR\_D)/)`.  Also, set *Autoclose Only* to `master`, or whatever
branches you want to autoclose.

**If you don't configure this correctly, Phabricator will auto-close your diffs.**


## Development

If you're adding new classes, let arcanist know what to look for:
```
cd conphig
arc liberate
```
See also:
https://secure.phabricator.com/book/phabricator/article/libraries/
https://secure.phabricator.com/book/phabricator/article/arcanist_lint_unit/
https://secure.phabricator.com/book/arcanist/class/ArcanistConfiguration


## extra bonus scripts
TODO(aleks): fix this part

For our integration, we also added a script that can be run on Travis to add a message to the diff
when the build succeeds or fails. We created a bot user in our Phabricator instance to have
the credentials to do this.

This, unfortunately, means you need to install PHP and other things
every time you do a CI. Here is some `travis.yml` config for your approbation:
```
env:
  global:
    - JAVA_OPTS="-Xms2000m -Xmx4000m -XX:MaxPermSize=512M"
    - PATH="$PATH:./path/to/phaceWrapScript"

install:
  - sudo apt-get install php5 php5-curl jq
  - git clone git://github.com/facebook/libphutil.git
  - git clone git://github.com/facebook/arcanist.git
  # And somehow either have phaceWrap in your repo or install it separately

before_script:
  - phaceWrap "starting"

after_failure:
  - phaceWrap "failure"

after_success:
  - phaceWrap "success"
```
In hindsight, I should have written these scripts in PHP instead of Bash, but such is life.

The `.traphic` script in the base dir of your git repo should have some settings like
how to find the conduit credentials and where you're Phabricator install is. See
`scripts/traphic.sample`.

The `phobot.arcrc` file looks something like this:
```
{
  "hosts": {
    "https:\/\/your.phabricator.com\/api\/": {
      "token": "cli-somelongtoken"
    }
  }
}
```
It can be created by hand, but you need to get the conduit api token for your bot (or other) user
to create it. Usually this is done by `arc install-certificate` and following the instructions.
