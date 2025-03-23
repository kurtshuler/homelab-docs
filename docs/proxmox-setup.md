---
title: Proxmox Setup
nav_order: 1
---

# Proxmox Setup
{: .no_toc }

Just the Docs has some specific configuration parameters that can be defined in your Jekyll site's \_config.yml file.
{: .fs-6 .fw-300 }

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Install Proxmox VE from ISO

Installing Proxmox VE from a Ventoy USB on bare metal is easy. Proxmox documentation with details is at [Proxmox Installation Documentation](https://pve.proxmox.com/pve-docs/chapter-pve-installation.html){:target="_blank"}.

### Make a bootable Ventoy USB drive

First, we need to create bootable USB drive with the Proxmox VE ISO. We will use Ventoy to do this.

The latest Ventoy installers are at [https://sourceforge.net/projects/ventoy/files/](https://sourceforge.net/projects/ventoy/files/){:target="_blank"}.

{: .important }
>The Ventoy USB can be created in Linux or Windows only. For **Mac**, use Parallels Windows VM or Linux VM.
>
> **After** you have created the Ventoy USB, you can **copy files to it** using your Mac or PC.

Important downloads and ISO images to install on Ventoy USB are below:

Downloads
{: .label .label-blue}

   | Download                                 | URL                                                  |  
   |:-------------------------------------|:-----------------------------------------------------|
   | Ventoy Installer                     | [https://sourceforge.net/projects/ventoy/files/](https://sourceforge.net/projects/ventoy/files/){:target="_blank"}|
   | System Rescue ISO                    | [https://www.system-rescue.org/Download/](https://www.system-rescue.org/Download/){:target="_blank"}|
   | Proxmox PVE and PBS ISOs             | [https://www.proxmox.com/en/downloads](https://www.proxmox.com/en/downloads){:target="_blank"}|
   | Ubuntu Server Live Install ISO       | [https://releases.ubuntu.com/](https://releases.ubuntu.com/)|
   | Ubuntu Server Cloud-Init Install ISO | [https://cloud-images.ubuntu.com/noble/current/](https://cloud-images.ubuntu.com/noble/current/){:target="_blank"}|
   | Ubuntu Desktop ISO                   | [https://ubuntu.com/download/desktop/](https://ubuntu.com/download/desktop/){:target="_blank"}|

### Boot from USB Drive and Install

If you need a walkthough on how to boot from USB and install, there's one at [https://medium.com/@r00tb33r/how-to-install-proxmox-a-step-by-step-guide-5d00439bc551](https://medium.com/@r00tb33r/how-to-install-proxmox-a-step-by-step-guide-5d00439bc551){:target="_blank"}.

## Proxmox post-install setup

### Check that SSH is running

Do
{: .label .label-green}

```shell
systemctl status ssh.service
```

### Run tteck's Proxmox VE Post Install Script

{: .warning }
Run tteck scripts from the **Proxmox GUI shell**, not SSH!

Do
{: .label .label-green}

```bash
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/post-pve-install.sh)"
```

This script cleans up a lot of cruft after Proxmox installation. Specifically, "this script provides options for managing Proxmox VE repositories, including disabling the Enterprise Repo, adding or correcting PVE sources, enabling the No-Subscription Repo, adding the test Repo, disabling the subscription nag, updating Proxmox VE, and rebooting the system."

{: .note-title}
> Who was tteck?
>
> Tteck was a volunteer who created a ton of scripts to install applications and environments within Proxmox. He did a lot to make Proxmox virtualization easy to use for mere mortals. He passed away November 2024.
>
> *A message from tteck's wife, Angie:*
>
> {: .opaque .fs-2}
> <div markdown="block">
> {: .important-title }
> This is Angie, and to say I am overwhelmed by all the comments, thoughts, love, prayers, and condolences is an understatement. As well as the donations. My husband loved this site, and he was passionate about computers, coding, and all the stuff that goes with it. He showed me a lot of the comments that were made when he posted about his cancer and it made him smile. Thank you all for the love and support that you have shown to him, and now me.
>He would be amazed that I even found his sight and was able to make a post. Thank you all again, it warms my heart that he touched so many lives.
> </div>
> The Ko-Fi link is still available if you want to support Angie and tteck's family (unfortunately I never knew his name):
> [https://ko-fi.com/proxmoxhelperscripts](https://ko-fi.com/proxmoxhelperscripts){:target="_blank"}

tteck's Helper-Scripts are now managed by a group of commuity members.

- Download and install the Proxmox scripts from [https://community-scripts.github.io/ProxmoxVE/](https://community-scripts.github.io/ProxmoxVE/){:target="_blank"}
<span class="fs-3">
[Proxmox Scripts Webpage](https://community-scripts.github.io/ProxmoxVE/){: .btn }
</span>

- View the Proxmox scripts source code at [https://github.com/community-scripts/ProxmoxVE/](https://github.com/community-scripts/ProxmoxVE/){:target="_blank"}
<span class="fs-3">
[Proxmox Scripts Github](https://github.com/community-scripts/ProxmoxVE/){: .btn }
</span>

The script we want is the "Proxmox VE Post Install" script at [https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install](https://community-scripts.github.io/ProxmoxVE/scripts?id=post-pve-install){:target="_blank"}

### Run tteck's Proxmox VE Processor Microcode Script

{: .warning }
Run tteck scripts from the **Proxmox GUI shell**, not SSH!

Do
{: .label .label-green}

```shell
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/misc/microcode.sh)"
```

Then reboot:

```sh
reboot
```

Check whether any microcode updates are currently in effect:

```sh
journalctl -k | grep -E "microcode" | head -n 1
```

This procedure updates the microcode on your Intel or AMD processor, if required. Intel and AMD update processor microcode usually to fix bugs and security issues.

### Set up IKoolcore-specific Proxmox summary (OPTIONAL)

{: .note}
> These instructions are specific to my iKoolcore R2 server hardware. Don't do it if you aren't installing on one.

Follow the steps in the iKoolcore R2 wiki at [https://github.com/KoolCore/Proxmox_VE_Status](https://github.com/KoolCore/Proxmox_VE_Status){:target="_blank"}

Download
{: .label .label-blue}

Add iKoolcore R2 hardware stats to the Proxmox summary page by downloading and running this shell script that I modified here [https://github.com/kurtshuler/proxmox-ubuntu-server/blob/main/Proxmox%20files/Proxmox_VE_Status_zh.sh](https://github.com/kurtshuler/proxmox-ubuntu-server/blob/main/Proxmox%20files/Proxmox_VE_Status_zh.sh){:target="_blank"}

Do
{: .label .label-green}

```sh
cd Proxmox_VE_Status
```

```sh
bash ./Proxmox_VE_Status_zh.sh
```

## Set up the Proxmox terminal

### Install neofetch

```shell
sudo apt install neofetch
```

### Install Oh My Bash

```shell
bash -c "$(curl -fsSL https://raw.githubusercontent.com/ohmybash/oh-my-bash/master/tools/install.sh)"
```

Reload `.bashrc`

```shell
source ~/.bashrc
```

If there is an error loading OMB, set the proper OMB file location in `.bashrc`
```shell
export OSH='/root/.oh-my-bash'
```

### Add plugins and completions to `.bashrc`
You can edit your `.bashrc` by copying and comparing to my GitHub Proxmox [`.bashrc`](https://github.com/kurtshuler/proxmox-ubuntu-server/blob/main/Proxmox%20files/.bashrc){:target="_blank"}

```shell
nano .bashrc
```

Reload `.bashrc`

```shell
source ~/.bashrc
```

### Install iTerm shell integration:
In the iTerm 2 GUI, click on `iTerm2 → Iterm Shell Integration`





## Site favicon

```yaml
# Set a path/url to a favicon that will be displayed by the browser
favicon_ico: "/assets/images/favicon.ico"
```

If the path to your favicon is `/favicon.ico`, you can leave `favicon_ico` unset.

## Search

```yaml
# Enable or disable the site search
# Supports true (default) or false
search_enabled: true

search:
  # Split pages into sections that can be searched individually
  # Supports 1 - 6, default: 2
  heading_level: 2
  # Maximum amount of previews per search result
  # Default: 3
  previews: 3
  # Maximum amount of words to display before a matched word in the preview
  # Default: 5
  preview_words_before: 5
  # Maximum amount of words to display after a matched word in the preview
  # Default: 10
  preview_words_after: 10
  # Set the search token separator
  # Default: /[\s\-/]+/
  # Example: enable support for hyphenated search words
  tokenizer_separator: /[\s/]+/
  # Display the relative url in search results
  # Supports true (default) or false
  rel_url: true
  # Enable or disable the search button that appears in the bottom right corner of every page
  # Supports true or false (default)
  button: false
  # Focus the search input by pressing `ctrl + focus_shortcut_key` (or `cmd + focus_shortcut_key` on macOS)
  focus_shortcut_key: 'k'
```

## Mermaid Diagrams

{: .d-inline-block }

New (v0.4.0)
{: .label .label-green }

The minimum configuration requires the key for `version` ([from jsDelivr](https://cdn.jsdelivr.net/npm/mermaid/)) in `_config.yml`:

```yaml
mermaid:
  # Version of mermaid library
  # Pick an available version from https://cdn.jsdelivr.net/npm/mermaid/
  version: "9.1.3"
```

Provide a `path` instead of a `version` key to load the mermaid library from a local file.

See [the Code documentation]({% link docs/ui-components/code/index.md %}#mermaid-diagram-code-blocks) for more configuration options and information.

## Aux links

```yaml
# Aux links for the upper right navigation
aux_links:
  "Just the Docs on GitHub":
    - "//github.com/just-the-docs/just-the-docs"

# Makes Aux links open in a new tab. Default is false
aux_links_new_tab: false
```

## Navigation sidebar

```yaml
# Enable or disable the side/mobile menu globally
# Nav menu can also be selectively enabled or disabled using page variables or the minimal layout
nav_enabled: true
```

## Heading anchor links

```yaml
# Heading anchor links appear on hover over h1-h6 tags in page content
# allowing users to deep link to a particular heading on a page.
#
# Supports true (default) or false
heading_anchors: true
```

## External navigation links

{: .d-inline-block }

New (v0.4.0)
{: .label .label-green }

External links can be added to the navigation through the `nav_external_links` option.
See [Navigation Structure]({% link docs/navigation/main/external.md %}) for more details.

## Footer content

```yaml
# Footer content
# appears at the bottom of every page's main content
# Note: The footer_content option is deprecated and will be removed in a future major release. Please use `_includes/footer_custom.html` for more robust
markup / liquid-based content.
footer_content: "Copyright &copy; 2017-2020 Patrick Marsceill. Distributed by an <a href=\"https://github.com/just-the-docs/just-the-docs/tree/main/LICENSE.txt\">MIT license.</a>"

# Footer last edited timestamp
last_edit_timestamp: true # show or hide edit time - page must have `last_modified_date` defined in the frontmatter
last_edit_time_format: "%b %e %Y at %I:%M %p" # uses ruby's time format: https://ruby-doc.org/stdlib-2.7.0/libdoc/time/rdoc/Time.html

# Footer "Edit this page on GitHub" link text
gh_edit_link: true # show or hide edit this page link
gh_edit_link_text: "Edit this page on GitHub."
gh_edit_repository: "https://github.com/just-the-docs/just-the-docs" # the github URL for your repo
gh_edit_branch: "main" # the branch that your docs is served from
# gh_edit_source: docs # the source that your files originate from
gh_edit_view_mode: "tree" # "tree" or "edit" if you want the user to jump into the editor immediately
```

*note: `footer_content` is deprecated, but still supported. For a better experience we have moved this into an include called `_includes/footer_custom.html` which will allow for robust markup / liquid-based content.*

- the "page last modified" data will only display if a page has a key called `last_modified_date`, formatted in some readable date format
- `last_edit_time_format` uses Ruby's DateTime formatter; for examples and information, please refer to the [official Ruby docs on `strftime` formatting](https://docs.ruby-lang.org/en/master/strftime_formatting_rdoc.html)
- `gh_edit_repository` is the URL of the project's GitHub repository
- `gh_edit_branch` is the branch that the docs site is served from; defaults to `main`
- `gh_edit_source` is the source directory that your project files are stored in (should be the same as [site.source](https://jekyllrb.com/docs/configuration/options/))
- `gh_edit_view_mode` is `"tree"` by default, which brings the user to the github page; switch to `"edit"` to bring the user directly into editing mode

## Color scheme

```yaml
# Color scheme supports "light" (default) and "dark"
color_scheme: dark
```

<button class="btn js-toggle-dark-mode">Preview dark color scheme</button>

<script>
const toggleDarkMode = document.querySelector('.js-toggle-dark-mode');

jtd.addEvent(toggleDarkMode, 'click', function(){
  if (jtd.getTheme() === 'dark') {
    jtd.setTheme('light');
    toggleDarkMode.textContent = 'Preview dark color scheme';
  } else {
    jtd.setTheme('dark');
    toggleDarkMode.textContent = 'Return to the light side';
  }
});
</script>

See [Customization]({% link docs/customization.md %}) for more information.

## Callouts

{: .d-inline-block }

New (v0.4.0)
{: .label .label-green }

To use this feature, you need to configure a `color` and (optionally) `title` for each kind of callout you want to use, e.g.:

```yaml
callouts:
  warning:
    title: Warning
    color: red
```

This uses the color `$red-000` for the background of the callout, and `$red-300` for the title and box decoration.[^dark] You can then style a paragraph as a `warning` callout like this:

```markdown
{: .warning }
A paragraph...
```

[^dark]:
    If you use the `dark` color scheme, this callout uses `$red-300` for the background, and `$red-000` for the title.

The colors `grey-lt`, `grey-dk`, `purple`, `blue`, `green`, `yellow`, and `red` are predefined; to use a custom color, you need to define its `000` and `300` levels in your SCSS files. For example, to use `pink`, add the following to your `_sass/custom/setup.scss` file:

```scss
$pink-000: #f77ef1;
$pink-100: #f967f1;
$pink-200: #e94ee1;
$pink-300: #dd2cd4;
```

You can override the default `opacity` of the background for a particular callout, e.g.:

```yaml
callouts:
  custom:
    color: pink
    opacity: 0.3
```

You can change the default opacity (`0.2`) for all callouts, e.g.:

```yaml
callouts_opacity: 0.3
```

You can also adjust the overall level of callouts.
The value of `callouts_level` is either `quiet` or `loud`;
`loud` increases the saturation and lightness of the backgrounds.
The default level is `quiet` when using the `light` or custom color schemes,
and `loud` when using the `dark color scheme.`

See [Callouts]({% link docs/ui-components/callouts.md %}) for more information.

## Google Analytics

{: .warning }
> [Google Analytics 4 will replace Universal Analytics](https://support.google.com/analytics/answer/11583528). On **July 1, 2023**, standard Universal Analytics properties will stop processing new hits. The earlier you migrate, the more historical data and insights you will have in Google Analytics 4.

Universal Analytics (UA) and Google Analytics 4 (GA4) properties are supported.

```yaml
# Google Analytics Tracking (optional)
# Supports a CSV of tracking ID strings (eg. "UA-1234567-89,G-1AB234CDE5")
ga_tracking: UA-2709176-10
ga_tracking_anonymize_ip: true # Use GDPR compliant Google Analytics settings (true/nil by default)
```

### Multiple IDs

{: .d-inline-block .no_toc }

New (v0.4.0)
{: .label .label-green }

This theme supports multiple comma-separated tracking IDs. This helps seamlessly transition UA properties to GA4 properties by tracking both for a while.

```yaml
ga_tracking: "UA-1234567-89,G-1AB234CDE5"
```

## Document collections

By default, the navigation and search include normal [pages](https://jekyllrb.com/docs/pages/).
You can also use [Jekyll collections](https://jekyllrb.com/docs/collections/) which group documents semantically together.

{: .warning }
> Collection folders always start with an underscore (`_`), e.g. `_tests`. You won't see your collections if you omit the prefix.

For example, put all your test files in the `_tests` folder and create the `tests` collection:

```yaml
# Define Jekyll collections
collections:
  # Define a collection named "tests", its documents reside in the "_tests" directory
  tests:
    permalink: "/:collection/:path/"
    output: true

just_the_docs:
  # Define which collections are used in just-the-docs
  collections:
    # Reference the "tests" collection
    tests:
      # Give the collection a name
      name: Tests
      # Exclude the collection from the navigation
      # Supports true or false (default)
      # nav_exclude: true
      # Fold the collection in the navigation
      # Supports true or false (default)
      # nav_fold: true  # note: this option is new in v0.4
      # Exclude the collection from the search
      # Supports true or false (default)
      # search_exclude: true
```

The navigation for all your normal pages (if any) is displayed before those in collections.

<span>New (v0.4.0)</span>{: .label .label-green }
Including `nav_fold: true` in a collection configuration *folds* that collection:
an expander symbol appears next to the collection name,
and clicking it displays/hides the links to the top-level pages of the collection.[^js-disabled]

[^js-disabled]: <span>New (v0.6.0)</span>{: .label .label-green }
    When JavaScript is disabled in the browser, all folded collections are automatically expanded,
    since clicking expander symbols has no effect.
    (In previous releases, navigation into folded collections required JavaScript to be enabled.)

You can reference multiple collections.
This creates categories in the navigation with the configured names.

```yaml
collections:
  tests:
    permalink: "/:collection/:path/"
    output: true
  tutorials:
    permalink: "/:collection/:path/"
    output: true

just_the_docs:
  collections:
    tests:
      name: Tests
    tutorials:
      name: Tutorials
```

When *all* your pages are in a single collection, its name is not displayed.

The navigation for each collection is a separate name space for page titles: a page in one collection cannot be a child of a page in a different collection, or of a normal page.
