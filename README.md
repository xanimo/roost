# roost

reads an X/Twitter data export into sqlite and lets you search it. an export is
a pile of `.js` files that are json wearing a hat, one assignment line and then
an array, split across parts once you have enough of them. roost loads the lot
into one database with full text search and serves a page for reading it.

python 3 standard library only. no pip, no network, no account. your export
never leaves the machine.

## use

```
./roost load ~/Downloads/twitter-<date>-<hash>.zip --db somewhere/roost.db
./roost --db somewhere/roost.db stats
./roost --db somewhere/roost.db search "some phrase"
./roost --db somewhere/roost.db user @someone
./roost --db somewhere/roost.db thread <tweet-id>
./roost --db somewhere/roost.db serve
```

`load` takes the zip directly, no need to unpack 18gb to read the text. it also
takes an already-extracted directory.

`serve` binds `127.0.0.1` and nothing else. this is your dm archive, it does not
belong on a lan.

it opens on your own posts, newest first, with retweets excluded. on a busy
account retweets are the large majority of rows and none of them are your
words, so they are a filter you turn on rather than the thing you wade through.
there is a year row for jumping, and the dm tab lists conversations biggest
first and remembers where you were when you come back out of one.

## what it loads

```
tweets, including tweets-partN     text, timestamps, reply targets, counts
deleted-tweets                     kept and flagged, they are in the export
likes, including like-partN
direct-messages and group          sender, recipient, timestamp, text
mentions and urls                  extracted per tweet
account                            handle, id, created
handles                            id -> @name, built from mentions and replies
```

dm files store numeric account ids and no names, which is why other tools show
you a wall of digits. every `user_mentions` entry and every reply target in your
own tweets carries an id and a handle together, so roost builds a lookup from
them and names the conversations it can. expect roughly the people you also
talked to in public: on a real archive that resolved the top conversations by
volume and about one in six overall. the rest are people who never appear in a
tweet, and their names are not in the export in any form.

a large export, low six figures of tweets and likes, loads in well under a
minute. the database is a fraction of the zip, which is mostly media.

## the thing it is actually useful for

`user` does something the export does not obviously offer. when you reply to
someone, the export stores `in_reply_to_status_id_str`, which is **their** tweet
id, not yours. so even for an account that has deleted everything, your own
archive still holds the ids of the specific tweets you answered:

```
./roost --db roost.db user @someone
...
N of their tweet ids, recovered from your reply metadata:
  https://twitter.com/someone/status/<their-tweet-id>
```

those ids are what you feed to the wayback machine. it is the only route from
your own export back to somebody else's deleted posts.

## what it does not do

it does not contain other people's words. an export holds your tweets and the
ids they point at, never the text of what you replied to or quoted. no tool can
change that, it is not in the file. quote tweets are a link, not a copy.

it does not load media, followers, ad data, or the several dozen other files an
export ships. it does not fetch anything from the network, so it cannot tell you
whether a tweet still exists.

full text search is fts5, so it is matching tokens rather than substrings.
`search cat` will not hit `cats` unless you ask for `cat*`.

## data

`.gitignore` excludes `*.db`, `*.zip` and `data/`. keep the database outside the
repository and there is no path by which an export reaches a remote.
