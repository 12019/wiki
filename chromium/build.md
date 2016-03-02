This is a cheat sheet for building Chromium so you should read the full instructions at the link in each section.

## Install deptools
The depot_tools package includes gclient, gcl, git-cl, repo, and others.

1. Fetch depot_tools: git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
2. Add depot_tools to your PATH:

<pre>
 $ export PATH="$PATH":`pwd`/depot_tools
</pre>
* http://dev.chromium.org/developers/how-tos/install-depot-tools

## Get Chromium code using git

<pre>
 $ git config --global user.name "My Name"
 $ git config --global user.email "my.name@intel.com"
 $ git config --global core.autocrlf false
 $ git config --global core.filemode false
 $ git config --global core.deltaBaseCacheLimit 1G 

 $ mkdir ~/git/chromium
 $ cd ~/git/chromium

 $ gclient config --spec 'solutions = [{u'"'"'managed'"'"': True, u'"'"'name'"'"': u'"'"'src'"'"', u'"'"'url'"'"': u'"'"'https://chromium.googlesource.com/chromium/src.git'"'"', u'"'"'custom_deps'"'"': {}, u'"'"'deps_file'"'"': u'"'"'.DEPS.git'"'"', u'"'"'safesync_url'"'"': u'"'"''"'"', u'"'"'custom_vars'"'"': {u'"'"'webkit_rev'"'"': u'"'"''"'"'}}]'
</pre>

### Initial checkout
<pre>
 $ fetch blink --nosvn=True
</pre>
* https://code.google.com/p/chromium/wiki/UsingGit

### Updating the code
First you need to make sure you are in the master branch.
<pre>
 $ cd src
 $ git checkout master
 $ git pull  (only chromium code)
</pre>
If you want to update third party tools, use gclient sync
 
Updating can be extremely slow with a limited number of jobs, --jobs=16 helps make it less painful.
<pre>
  gclient sync --jobs=16
</pre>

## Bootstrap notes for Ubuntu
* Make sure your dependencies are up to date by running the install-build-deps.sh script:
<pre>
 $ cd ~/git/chromium/src
 $ sudo build/install-build-deps.sh
</pre>

## Build chromium
<pre>
 $ rm -rf out/Debug
 $ export GYP_GENERATORS='ninja'
 $ build/gyp_chromium -D component=shared_library
 $ ninja -C out/Debug chrome
</pre>

### Dealing with gcc warnings
By default we fail to build if there are any compiler warnings. If you're getting warnings, can't build because of that, but just want to get things done, you can specify -Dwerror= to turn that off:
* one-off
: build/gyp_chromium -Dwerror=
* via variable
: export GYP_DEFINES="werror="


* http://code.google.com/p/chromium/wiki/LinuxBuildInstructions#gyp_%28configuring%29


## Running Chromium
<pre>
 $ cd ~/git/chromium/src
 $ out/Debug/chrome
</pre>

## Contributing your patches
<pre>
 $ cd ~/git/chromium/src/third_party/WebKit
 $ git branch myChange
 $ git checkout myChange
 $ vi Source/WebCore/editing/FrameSelection.cpp  (editing)
 $ git commit
 $ git cl upload 
</pre>
This will open your text editor.  Write your patch description as follows:
<pre>
We need to change the caret color according to the lightness of the background color. 
BUG=232188
TEST=Follow the bug description.
</pre>

Then, you will find suggested owners in the output of running "git cl upload".
Add the suggested owners to the Reviewers field after clicking the Edit issue button.

Note: You should add your name & email into the AUTHORS file for your first patch.
https://codereview.chromium.org/13818036/patch/1/2

## Updating your patch
You can update your patch on an already uploaded CL by running "git cl upload" again, but sometimes it create a new issue. In this case, you need to run git cl issue as follows:
<pre>
 $ git cl issue 14098003 (issue number)
 $ git cl upload
</pre>
* http://dev.chromium.org/developers/contributing-code

## Running Layout Test
<pre>
$ ninja -C ./ all_webkit DumpRenderTree
$ Xvfb :4 -screen 0 1024x768x24
src/third_party/WebKit$ DISPLAY=:4 Tools/Scripts/run-webkit-tests --debug
</pre>

How to debug layout test
<pre>
$ gdb --args /home/joone/git/chromium/src/out/Debug/content_shell --dump-render-tree --no-sandbox --single-
process fast/events/before-unload-returnValue.html
</pre>

https://code.google.com/p/chromium/wiki/LayoutTestsLinux

## References

 * http://www.chromium.org/blink/conversion-cheatsheet
 * https://code.google.com/p/chromium/wiki/LinuxBuildInstructions
 * https://code.google.com/p/chromium/wiki/LinuxFasterBuilds
