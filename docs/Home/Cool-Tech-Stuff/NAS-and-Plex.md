## Helpful Links
[Synology NAS Specs](https://www.synology.com/en-us/products/DS716+II#spec)

[DSM Knowledgebase](https://www.synology.com/en-global/knowledgebase)
* [Getting Started](https://www.synology.com/en-global/knowledgebase/DSM/help/DSM/MainMenu/get_started)

[Plex](https://www.plex.tv/)

[FileBot](https://www.filebot.net/)

[Update Plex Server Version](https://jeremywsherman.com/blog/2015/11/08/updating-plex-on-synology-nas/)

---

# NAS Plus Plex Equals Awesomeness Everywhere

After setting up my NAS with a Plex server I'm now able to do basically everything on my Xbox One from games, to Netflix/HBO/Hulu, to home movies/music/photos!

I recently got my hands on a 2-bay [Synology NAS](https://www.synology.com/en-us/products/DS716+II#overview) with a couple of 4TB drives. So, I got to work setting it up with a Plex server and some media organizing automation for a result that requires little maintenance and a lot of payoff.

My setup process consisted basically of the following:
* NAS hardware installation
* NAS software installation/configuration
* FileBot installation onto NAS and configuration for preparing Plex-compliant named media files
* Plex installation onto NAS and configuration

## Setting Up the NAS

#### Bottom Line: Pretty much ready to go out-of-the-box
Synology makes it easy to get the NAS device up and running, and there is an abundance of [documentation](https://www.synology.com/en-global/knowledgebase) available, so there isn't really a point in duplicating documentation efforts in terms of how-to instructions.
* Super easy hardware installation
* Super easy software setup
* Intuitive Web Interface to the DSM OS

## FileBot

#### Bottom Line: Simple, but powerful defaults, and granular control is available
FileBot is an awesome automation tool that can be used to setup scheduled tasks for renaming files. You basically point it to a source directory containing all original media files, and provide a path to an output directory for the renamed files--it can be configured to create hard links instead of copying the files.

There are many settings for granular configuration, and it's basically just a script generator, so documentation can be found for writing the scripts instead of using the GUI for even more granular control.

## Plex

#### Bottom Line: Bottom Line: Pretty much ready to go out-of-the-box (especially with FileBot handling file renaming automation)
Just create a Plex account, install Plex on the NAS and start the Plex server, open up the web interface and add a library. That's it. Now anything with a browser can play media served by the Plex server.

## Xbox One

Aside from using a browser, the Xbox One has a Plex app available that has a pretty nice UI. This can be activated with a code similar to how you would add the Xbox One as a device connected to Netflix.

---

### Updating Plex Media Server on Synology NAS

This is a simple but slightly counterintuitive process. Here is a [maybe over-thorough set of instructions](https://jeremywsherman.com/blog/2015/11/08/updating-plex-on-synology-nas/).

**NOTE**
  * Package Center settings must be set to allow Synology Inc. and trusted publishers
  * Must add the [Plex package signing public key](https://support.plex.tv/hc/en-us/articles/205165858) if not already done
    * Alternatively, allow any publisher

#### How to Access the Update File
In the Plex server's Web UI under Settings/Server the current version will be specified and there will be an indicator that the version is up to date or there will be a link to download the new version.

#### Basic Process
* Download the update file from the Plex server's Web UI to the local machine (don't physically put it on your NAS)
* Click Manual Install from the Synology NAS Package Center application
* Point to the update file that was downloaded onto the local machine
* Click Next and the update will be applied (_existing database will be preserved_)
