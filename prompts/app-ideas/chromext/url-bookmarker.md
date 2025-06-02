It's always been my dream to create a magic bookmark Chrome Extension which works this way.
Every time I'm on a website, say a youtube page or a medium article, this extension looks up a DB (say Google Cloud Firestore, for instance, to keep schema flexible and size low) and if there is some sort of bookmark (url, title, description) it renders that note. So the app is basically a post it note which gets trigered by me navigating a certain bookmark, stored by [ url, user_id, title, description, .. and usual timestamp modifiers as in rails]. Inititally we make it work just for myself (user_id = 1, email=palladiusbonton@gmail.com) but keep it multitennant for the future.
I've never written a chromext, can you please help me?

It's important there is some sort of deduplication functionality to make sure that if I'm in 2 different URLs which represent the "same" website, they should be mapped to the same "CanonicalURL". This function needs to be written WELL and be VERY extensible. For instance, we might have some:
* generic regex (eg, remove everything AFTER the querystring)
* specific rules for certain domains (eg, youtube.com/watch?v=abc should be the same as youtube.com/watch?v=abc&t=10s).
  * manual overrides. For instance, youtube should be able to remove the regex querystring, or at least preserve the "v" field.

