Eucabot is a multi-purpose fount of information that can be found on [Freenode](http://freenode.net/).  It is capable of managing bugs [<sup>[1]</sup>](#bug_management), IRC users [<sup>[2]</sup>](#misc_commands), and IRC meetings [<sup>[3]</sup>](#meeting_management).  It subsists primarily on botsnacks <sup>[ _citation needed_ ]</sup>.

# Guidelines

1. Eucabot resides in most Eucalyptus-related IRC channels, but not all of them.  If you need it to join another channel then contact a bot administrator in #eucalyptus.
1. Do not spam the channel with bot messages.  Go to #eucabot if you're going to trigger eucabot for anything longer than a couple lines.  If you wish to trigger even more, do it in private message to eucabot.

# Docs
## Addressing

Eucabot rarely responds when not addressed directly.  You can address eucabot directly by prefixing your message with "eucabot:" or "eucabot," (Your IRC client may automatically do this.)

You can also address eucabot by preceding your message with a period.

## Bug management<a id="bug_management"/>

Eucabot can provide short summaries and quick links to bugs in any of several bug trackers.  It can sometimes do simple bug management as well.

### Eucalyptus JIRA

Eucabot can show information about bugs in [Eucalyptus's bug tracker](https://eucalyptus.atlassian.net/).  If your bot account has the right privileges, it can also manage bugs as well.

* **issue _issue_** - Show a summary of an issue.

In addition, these commands are usable by people with `jirawrite` privileges:

* **assign _issue_ to _person_ [ _commentary_ ]** - Assign an issue to someone.  Be sure to use the assignee's JIRA account name.
* **confirm _issue_ [ _commentary_ ]** - Set an issue's state to Confirmed.  Note that this may fail, depending on the issue's current state.
* **resolve _issue_ [ --resolution _resolution_ ] [ _commentary_ ]** - Set an issue's state to Resolved.  To use a resolution different from JIRA's default, supply its name with the --resolution option.  If the resolution's name contains spaces, surround it with double quotes.  Note that this may fail, depending on the issue's current state.
* **target _issue_ _version_** - Replace an issue's target version(s), if any, with the one supplied.  This command currently supports only one version at a time.

### Launchpad

[Launchpad](https://launchpad.net/) is the main bug tracker for Ubuntu and many other projects.

* **lp _issue_** - Show a summary of a bug.

### Red Hat Bugzilla

[Red Hat's bugzilla instance](https://bugzilla.redhat.com/) is the main bug tracker for Red Hat Enterprise Linux and Fedora.

* **rh _issue_** - Show a summary of a bug.

## Meeting management<a id="meeting_management"/>

Eucabot can take notes during IRC meetings.  After a meeting finishes, it will publish logs and meeting minutes to [its web server](https://meetbot.eucalyptus.com/meeting-logs/).  To employ this functionality, use the commands below.  The general syntax of a command is the command name, which begins with a #, followed by command-specific text.  If a line does not begin with a # then eucabot will not interpret it as a meeting command, so _do not_ address it directly when using such a command.

Eucabot refers to people who run meetings as _chairs_ and allows them to run extra commands for the duration of the meeting.  The person who starts a meeting automatically becomes a chair for that meeting.

* **#startmeeting** - Start a meeting with a given overall topic.  The meeting topic becomes the title that appears at the top of the meeting minutes.  You are designated the meeting's first chair.
* **#meetingname** - Set the file name to give the meeting minutes.  Stick to the `[:word:]` character class or the name you supply will be mangled.  (Chairs only)
* **#topic** - Start a new topic of discussion.  This creates a new section in the meeting minutes.  (Chairs  only)
* **#info** - Add a note to the meeting minutes.  Use this command liberally, as general chat does not appear in the meeting minutes.
* **#link** - Add a link, along with any text that follows it, to the meeting minutes.  Note that eucabot automatically does this with certain links even when this command is not used.
* **#idea** - Add an idea that was proposed during the meeting to the meeting minutes.
* **#action** - Add an action item to the meeting minutes.  Eucabot searches action items for IRC nicks that have spoken during the meeting and breaks them down into a list by IRC nick at the end of the meeting.  See also the #nick command.
* **#help** - Add a call for help to the meeting minutes.  Use #commands if you are looking for help with the meeting bot.
* **#agreed, #accepted, #rejected** - Mark something as agreed upon, accepted, or rejected.  (Chairs only)
* **#endmeeting** - End the meeting, then show links to the meeting minutes and logs.  (Chairs only)
* **#chair** - Add a new chair to the meeting.  (Chairs only)
* **#nick** - Force eucabot to recognize something as an IRC nick even though that person has not spoken.  This is only useful for ensuring action items are assigned to the correct people.
* **#undo** - Remove the last item from the meeting minutes.  (Chairs only)

### Example

```xml
<alice> #startmeeting Bug Scrub (30 Jun 2011)
<eucabot> Meeting started Mon Jun 30 18:00:58 2011 UTC.  The chair is alice. Information about MeetBot at http://wiki.debian.org/MeetBot.
<eucabot> Useful Commands: #action #agreed #help #info #idea #link #topic.
<alice> #meetingname bugscrub
<eucabot> The meeting name has been set to 'bugscrub'
<alice> #topic Roll call
<bob> I'm here!
<carol> Howdy
* dave is here
<alice> Great! Let's get started. We only have one bug today, so this should be quick.
<alice> #topic EUCA-640: hats, we need euca hats
<alice> .issue euca-640
<eucabot> alice: EUCA-640 (Unconfirmed): hats, we need euca hats - https://eucalyptus.atlassian.net/browse/EUCA-640
<bob> I think hats are great ways to start conversations at events.
<carol> #info Hats are good conversation starters
<dave> I'd wear a decent baseball cap.
<alice> Yeah, we should definitely get hats.
<alice> .confirm euca-640
<eucabot> Okay, alice.
<alice> But what color?
<dave> Blue!
<bob> Green!
<carol> How about green?
* alice prefers blue
<dave> Why not just buy some of each?
<alice> #idea Buy both blue and green hats
<alice> How does that sound?
<carol> alice: +1
<dave> alice: +1
<bob> alice: +1 yesplz
<alice> #agreed Will buy both blue and green hats
<alice> #topic Open floor
<alice> Anything else for this meeting?  Anyone stuck on any bugs?
<alice> Thanks for coming, everyone!
<alice> #endmeeting
```

## Channel logging

Eucabot logs activity in every channel in which it resides.  It publishes these logs to [its web server](https://meetbot.eucalyptus.com/channel-logs/).

## Eucabot accounts

If you need special permission to do something (or if you just want eucabot to know who you are) you can have eucabot create an account for you.  You can log into your account by either sending eucabot "identify" along with your account name and password in a private message, or by logging in from a computer that matches something in your account's list of hostmasks.

* **register _name_ _password_** - Create a new account with the given name and password.  _The machine you are logged in from will be added to your hostmask list._  (Must be sent via PM)
* **identify _name_ _password_** - Log into your account.
* **hostmask list** - View your account's list of hostmasks.  By default, anyone who is logged in from a computer that matches your hostmask list is automatically logged into your account.
* **hostmask add [ _hostmask_ ]** - Add a new hostmask (or your current one, if you don't supply one yourself) to your account.  (Must be sent via PM)
* **hostmask remove _name_ ( _hostmask_ | all )** - Remove a hostmask (or all of them) from your account.  (Must be sent via PM)
* **set password _password_** - Change your password.  (Must be sent via PM)
* **set secure _password_ [ True | False ]** - Sets (or toggles) the secure flag on your account.  When the secure flag is set, you must identify yourself with your password _and_ be connected from a computer in your account's hostmask list in order to log in.  (Must be sent via PM)
* **whoami** - Say what account you are logged into.

## Miscellaneous commands<a id="misc_commands"/>

We occasionally discover new commands that are not documented.  Some of them exist intentionally.  Others do not.  This section attempts to document them.

* **docs** - Show a link to eucabot's documentation.
* **more** - If a message is too long to tell you in one line, eucabot will end it with something like "(1 more message)".  The "more" command asks eucabot to say the next line of the message.
* **seen _nick_** - Show the last message that eucabot remembers a person saying in the channel.
* **any _nick_** - Show the last thing that eucabot remembers a person doing in the channel.
* **weather _location_** - Show weather conditions for somewhere in the world.  If you are logged into a eucabot account then eucabot will remember the last location whose weather you checked so you do not have to type it next time.  Weather data are provided by [Weather Underground](https://www.wunderground.com/).  
![Weather Underground logo](https://www.wunderground.com/logos/images/wundergroundLogo_4c_horz.jpg)
* **botsnack** - `:D`

# Administration

Users with `admin` capabilities can manage other accounts' privileges and also do general bot maintenance.  Admin commands include:

* **admin capability add _user_ _capability_** - Add a capability, such as `jirawrite`, to someone's account.  You can only add capabilities that you have yourself.
* **admin capability remove _user_ _capability_** - Remove a capability from someone's account.
* **admin ignore list** - List all of the hostmasks eucabot is currently ignoring.
* **admin ignore add ( _hostmask_ | _nick_ ) [ _seconds_ ]** - Make eucabot ignore everything from a hostmask (or the hostmask that matches a certain nick) for a given number of seconds.  If you do not supply an expiration time then the ignore will be permanent.
* **admin ignore remove ( _hostmask_ | _nick_ )** - Make eucabot stop ignoring things from a hostmask (or the hostmask that matches a certain nick).
* **admin nick [ _newnick_ ]** - If a new nick is supplied, change eucabot's nickname.  Otherwise, say what eucabot's current nick is.

# Quotes
```
<viglesias> eucabot: pong
<eucabot> viglesias: You're doing it wrong.
```

[[category.infrastructure]]