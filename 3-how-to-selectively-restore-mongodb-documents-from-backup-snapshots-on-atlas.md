# How to selectively restore MongoDB documents from backup snapshots on Atlas.

I pay extra (around $100/mo) for my MongoDB cluster on Atlas. It's expensive for what I do (running a small block with a few extra features). But it comes with hourly snapshots which I need as data loss is common in one particular case that I just can't get around to fix: it happens rarely enough that I can't justify spending the time needed to improve my system and benign enough for the rest of the app that I forget the pain quickly after the incident.

Unfortunately, it's impossible to restore a single document from the backups Atlas provides. While it's easy to roll back the entire databse to that point, reading the **snapshots** that the service provides is a huge pain in the ass and there's no documentation online that shows a definitive and quick way of doing that. I and I really need a definitive and quick way to restore my data, as I'm sure the reader would too, should something go wrong. The solutions I've come across so far vary greately in approaches and the last time I tried (all) of them, none worked (due to driver compatability — which this article should help you circumvent/resolve).

I'm a little ticked off that Atlas doesn't make this task easy. They could've written something like this and placed a link on the dashboard for anyone to benefit from. Worse, their chat support may not even help you (once you reach them) unless you subscribe to a tier with money allocated for "advanced" assistance.

Hopefully this helps someone else stuck in this situation or the future me. Here's how it works:

## Restore strategy: use a local Docker container to spin up and preview snapshot.

Restoring the entire database to a preivous point in time is a bad idea if the issue has occured in a single or select few documents on a system that updates often. My blog accepts user comments and creates new accounts daily. If I roll the entire thing back, some users will get disappeared, same with comments and any other non-owner interactions. The best way to roll back an isolated document is to read the backup, then copy the contents of that document from the backup into target environment DB.

I run local version of MongoDB in a Docker container; but even if I didn't replacing my local database with a production snapshot is also a bad idea: I want my test DB to remain as-is to keep development smooth and avoid potential issues down the road.

Thankfully, Docker makes spinning isolated MongoDB containers easy (if you know what to type into terminal). And MongoDB Compass can make locating that data even simpler thanks to its UI (although Compass is an optional crutch you may not need).

## Download and view MongoDB backup snapshot contents.

Paid Atlas teirs provide backups with various frequencies. You'll need to **request yours backup** via their website (I'm assuming you know your way around there) and wait a few minutes for it to generate. Then a few more minutes as it downlaods.

Next, you'll need to **extract the archive** and move contents into a folder you have access to from your Terminal. (I use macOS but I think this is applicable on Windows and Linux as well.)

You may want to run `pwd` to learn the absolute path to the directory your CLI is pointing to (which is where you should keep your extracted backup files — in a folder the name of which is easy to type).

Then type `docker run -it -p 27017:27017 -v /Users/d/Documents/Code/tmp:/data/db  mongo:4`. Here, your CLI should be pointing to `/Users/d/Documents/Code` and your extracted backup files shoud be in `/Users/d/Documents/Code/tmp` — you can obviously name your backup folder as you wish and your code will probably live somewhere elase. But I'm sure you know this already. Note `mongo:4` bit at the end: this was very important in my case as my production DB does not run the latest version of MongoDB which lead to errors which are difficult to research a solution for online. You should use the exact major version of your MongoDB in this docker command as your production DB.

Once the Docker is running, you can connect to your new local databse using `mongodb://localhost:27017` via Compass or CLI. Note that if those ports are occupied, you can switch the Docker command to use something else.

## Copy previous version of the document and place it into prodcution/env DB.

Now all that you have left to do is to find the document(s) that need to be rolled back to the backup version and copy their contents into your production or whatever environment DB. Don't forget to clean up your directory from the backup files once done.
