webkit2png-driver
=================
Wrapper around `webkit2png` for taking and archiving screengrabs of websites,
with some support for interaction such logging in/out and filling out forms.

This can be used as a manual pre-flight checklist before a website deploy, a
permanent archive of the evolution of a site's appearance, a regression tester
(when combined with some sort of image analysis tool), or just for teh shiny.

Installing
----------
1. If you are not using OS X ... er ... good luck with that.
1. Install my fork of [webkit2png][webkit2png] in your PATH:
   * `sudo cp webkit2png /usr/bin`
   * `sudo chmod 755 /usr/bin/webkit2png`
1. Install `cpanm` (makes installing perl modules easy):
   * *homebrew*: `brew install cpanminus`
   * *not homebrew*: `curl -L http://cpanmin.us | sudo perl - --self-upgrade`
1. Install required perl modules:
   * `sudo cpanm Config::Std IO::All http://bumph.cackhanded.net/Template-Jigsaw-v0.1.tar.gz`
1. Put this script in your PATH:
   * `sudo cp webkit2png-driver /usr/bin`
   * `sudo chmod 755 /usr/bin/webkit2png-driver`

Using
-----
Make a directory to keep your configuration and resulting screengrabs in.
The rest of the instructions assume that this is your current working 
directory (ie. `cd` into it before going any further).

You will need at least one INI file to describe the screengrabs you want
taken. A couple of examples are included in this repository that you might
like to copy to your directory and then modify. Although they can be named
anything, the default is `grabs.ini`.

    # copy a sample INI file
    % cp .../webkit2png-driver/sample-twitter.ini grabs.ini


### Take the screengrabs

The default mode of `webkit2png-driver` is to take all of the screengrabs
specified in your `grabs.ini` file.

    # take all screengrabs
    % webkit2png-driver
    
    # use a different config file
    % webkit2png-driver -c other.ini
    
    # take only those screengrabs that match the argument(s);
    # arguments can be full or partial names, and support
    # wildcards as it is really a regexp match
    % webkit2png-driver homepage/logged-out
    % webkit2png-driver nojs noflash
    % webkit2png-driver home.*out

### Generate HTML indexes

If you want to more easily explore the screengrabs you have taken (especially
useful if you are archiving them), you can generate HTML indexes.

First, you will need to copy the sample HTML templates from this repository 
to your screengrabs directory. You can customise them later (if you can
stand to look at the embedded perl code).

    % cp -R .../webkit2png-driver/sample_templates ./_templates
    
To regenerate the indexes:

    % webkit2png-driver -i

To temporarily explore the screengrabs using the HTML indexes without
transferring them to a web server first:

    # indexes browseable at <http://localhost:8000/>
    % python -m SimpleHTTServer
    
### Permanently archive screengrabs

If you want to keep more permanent copies of the screengrabs (eg. to be able
to demonstrate later how the site has evolved over time), issue an archive
command after you have taken the screengrabs.

    % webkit2png-driver -a [name of this archive]

Each archived copy needs a name (some examples include `alpha-release`,
`pre-awesome`, `deploy-20111122`, `tag-revolver-ocelot`). Archiving copies
the 'latest' screengrabs to files matching the name, and then regenerates
the HTML indexes.

Using the same name again will overwrite previous archived copies without
any issued warnings.


INI options
-----------
The configuration file is in the "INI" format. Options are on a line with 
the key and value separated by an equals sign (=). Sections (which are the
individual URLs to be screengrabbed) are named in square brackets ([]).
Comments are lines that start with a hash (#).

The file should start with any default options.

Also see `sample-twitter.ini` for an example.

### Default options

* **base**: base URL, eg. `http://twitter.com`
* **title**: title to put into generated HTML indexes
  (default: same as **base**)
* **width**: width of full size screengrabs in pixels
  (default: 1024)
* **delay**: delay after page load before taking screengrabs in seconds
* **loggedout**: enforce starting screengrabs in a logged out state
  (see "Using logins/forms")
* **make_clipped**: generate the clipped screengrabs 
  (default: 1; 0 to suppress these files)
* **make_fullsize**: generate the full size screengrabs
  (default: 1; 0 to suppress these files)
* **make_thumb**: generate the thumbnail screengrabs
  (default: 1; 0 to suppress these files)
* **clipheight**: height of clipped screengrabs in pixels
  (default: 300)
* **clipwidth**: width of clipped screengrabs in pixels
  (default: 250)

### Screengrab options

Individual grabs can override any of the default options as well as using 
these options:

* **url**: the URL to grab; **base** is prepended unless it is an absolute URL
* **login**: should be logged in as this account before taking the screengrab
  (see "Using logins/forms")
* **nojs**: take screengrab with JavaScript support turned off
* **js**: JavaScript to execute before taking the screengrab
* **timeout**: execute the JavaScript specified in **js** after 
  this many milliseconds (wraps it in a setTimeout call)
* **chain_after**: section name of another screengrab that this one should
  be performed immediately after (most useful for chaining the after-effects
  of form submissions)
* **click**: jQuery selector specifying element(s) to be "clicked" before
  taking the screengrab
* **fill_name**: jQuery selector specifying the input element(s) to fill out
  with a random name (John [12 random letters] Doe)
* **fill_email**: jQuery selector specifying the input element(s) to fill out 
  with a random email ([12 random letters]@mailinator.com)
* **fill_pass**: jQuery selector specifying the input element(s) to fill out 
  with a random password ([12 random letters])
* **submit**: jQuery selector specifying the element to "click" after
  filling out the form to submit it

Using logins/forms
------------------
`webkit2png-driver` has some limited support for taking screen grabs with
different logins, and filling out forms. **Note: these all expect jQuery to be
loaded in the webpage**.

### Logging in

Create a user with a section called `user#[username]`, and provide the 
following options:

* **username**: the ID/username/email of the login
* **password**: the password of the login

Create a section called `form#login`, and provide the following options:

* **url**
* **username**: jQuery selector specifying the input element to fill out
  with the user ID/username/email
* **password**: jQuery selector specifying the input element to fill out
  with the user password
* **checkbox**: jQuery selector specifying the input element(s) to click
  before submitting the login form (eg. "Agree to T&Cs" or "Remember me")
* **submit**: jQuery selector specifying the element to "click" to after
  filling out the username and password.

Create a section called `form#logout`, and provide the following options:

* **url**
* **submit**: jQuery selector specifying the element to "click" to log the
  user out

Then any screengrab can have the option **login** set to `[username]`
that matches a `user#[username]` section to be logged in as that user before
taking the screengrab.

No error detection is performed on the submission of logins/logouts/forms,
so if they fail for some reason the screengrabs will not be in the correct 
state.


[webkit2png]: https://github.com/norm/webkit2png
