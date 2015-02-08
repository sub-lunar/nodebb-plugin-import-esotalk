nodebb-plugin-import-ubb
========================

An esoTalk forum exporter to be required by [nodebb-plugin-import](https://github.com/akhoury/nodebb-plugin-import), based on [nodebb-plugin-import-ubb](https://github.com/akhoury/nodebb-plugin-import-ubb).

### BEWARE

Might crash for large numbers of records, especially posts! Use at your own risk! (It’s MIT license anyway, so what do I care?).

Due to esoTalks database structure (and my poor MySQL skills) I wasn’t able to properly select the posts without the topic-starting posts in my query, so I have to detect and throw them out via javascript. Since I do not know how many of them will be there at query time, I can’t apply the limit to the query properly. Thus, **all** posts are selected everytime and the array is sliced in JavaScript after the starting posts are thrown out! Which means that, if you have a lot of posts in your forum, you might need a lot of ram to do this ...

### What is this?

It's __just__ an exporter of [esoTalk Threads data](http://esotalk.org/), that provides an API that [nodebb-plugin-import](https://github.com/akhoury/nodebb-plugin-import) can use to export source forum data and import it to NodeBB's database. So, it's not really a conventional nodebb-plugin.

### Why is it even a NodeBB plugin?

it doesn't really need to be, nor that you can use it within NodeBB it self, but, having this as a plugin have few benefits:
* a nodebb- namespace, since you can't really use it for anything else
* it can easily `require` NodeBB useful tools, currently

### Usage within NodeJS only

```
// you don't have to do this, nodebb-plugin-import will require this plugin and use its api
// but if you want a run a test

var exporter = require('nodebb-plugin-import-ubb');

exporter.testrun({
    dbhost: '127.0.0.1',
    dbport: 3306,
    dbname: 'ubb',
    dbuser: 'user',
    dbpass: 'password',

    tablePrefix: 'ubbt_'
}, function(err, results) {

    /*
        results[0] > config
        results[1] > [usersMap, usersArray]
        results[2] > [categoriesMap, categoriesArray]
        results[3] > [topicsMap, topicsArray]
        results[4] > [postsMap, postsArray]
    */
});

```

### What does it export?
read carefully:

- ####Users:
    * `_uid` (esoTalk memberId) YES
    * `_username` YES
    * `_email` YES
    * `_password` NO. Just to tell your users to reset their passwords after the migration. Or, if you use [nodebb-plugin-import](https://github.com/akhoury/nodebb-plugin-import) it can generate random passwords and hand them to you so can email them.
    * `_level` (administrator) YES. Admins will stay Admins. Moderators can’t be preserved, because esoTalk doesn’t handle Moderators as account type, but through user groups.
    * `_joindate` YES, esoTalk uses Seconds, the exported will convert to Milliseconds
    * `_picture` NO. I didn’t find out where (if at all) the avatar urls are saved in esoTalk, and it’s not that important for me (I only have ~20 Users and only four of them have actually uploaded a picture).
    * `_banned` NO.


- ####Categories (AKA Channels per esoTalk speak):
    * `_name` YES
    * `_slug` YES
    * `_description` YES

- ####Topics (AKA Conversations per esoTalk speak):
**BEWARE**: in esoTalk, there can be private Conversations that are only visible to certain users. After migration, they will be visible for everyone, like normal Topics!
    * `_tid` (esoTalk conversationId) YES
    * `_cid` (esoTalk channelId) YES
    * `_pid` (esoTalk postId) YES
    * `_uid` (esoTalk memberId) YES
    * `_title` YES
    * `_content` __(or the 'parent-post` content of this topic)__ YES (BB Code - read the [Markdown Note](#markdown-note) below)
    * `_timestamp` YES, esoTalk uses Seconds, the exporter will convert to Milliseconds
    * `_pinned` YES (0 or 1) (I don't know how many you can pin in NodeBB)
    * `_viewcount` YES

- ####Posts:
    * `_pid` (esoTalk postId) YES
    * `_tid` (esoTalk conversationId) YES
    * `_uid` (esoTalk memberId) YES
    * `_timestamp` YES, esoTalk uses Seconds, the exporter will convert to Milliseconds
    * `_subject` YES (esoTalk unnecessarily saves the (same) post.title (= title/subject of the conversation/topic) to every post)
    * `_content` YES (BB Code - read the [Markdown Note](#markdown-note) below)

### esoTalk Versions tested on:
  - esoTalk 1.0.0g3

### Markdown note

read [nodebb-plugin-import#markdown-note](https://github.com/akhoury/nodebb-plugin-import#markdown-note)

### It's an exporter, why does it have 'import' in its title

To keep the namespacing accurate, this __exporter__ is designed to export data for [nodebb-plugin-import](https://github.com/akhoury/nodebb-plugin-import) only, also for a 1 time use, so why do you care.
