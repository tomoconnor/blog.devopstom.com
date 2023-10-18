---
title: Raid != Backup
date: "2009-01-03T00:00:00.000Z"
description: "RAID is not a backup"
---
Another lesson learnt by a company who really should know better.

Raid != Backup.

This might be widely regarded as old news, but it’s not too late IMO for me to add my $0.02.

I picked this up on Slashdot about 20 minutes ago,  and there’s a few things that strike me as odd about the whole malarkey.

Before I go any further though, I’ve never heard of Journalspace until this article arose,  then again, they’re not really in my general field of view, I’ve always had my own blog, on my own space.. so it’s not really my ‘thing’.. Anyway, one thing that is my ‘thing’ is data security and assurance.


> Journalspace is no more.
>
> DriveSavers called today to inform me that the data was unrecoverable.
> Here is what happened: the server which held the journalspace data had two large drives in a RAID configuration. As data is written (such as saving an item to the  database), it’s automatically copied to both drives, as a backup mechanism.

Anyway… A few things strike me as odd.

Was that RAID really their ONLY backup?.. for a site that had been going for 6 years, and probably had >1000 users, I’m surprised that they didn’t write backups into their disaster recovery plan, and/or their plan to scale their site to meet their user’s needs.

Blaming OSX? That’s a bit of a low blow.   I’ve never used OSX Server for webhosting, but it seems reasonably unlikely that this is the cause of their problems.  And even if it was, that’s still NO excuse not to have some form of backup.

**Disgruntled Employee Syndrome.**  
While it’s not always possible to keep 100% of employees happy for 100% of the time, it is reasonably easy to revoke root keys on servers, delete user accounts, remove privileges, etc when the employee leaves the company.  It’s like not taking their office keys from them when you escort them from the building.  “Come back and steal our data, we’re practically leaving the whole office open”.

On a slightly different note, RAID is not Backup, it’s part of the solution.
Well, actually, it’s closer related to high availability and redundancy, but that is a different story.
Off-machine backups are the key here.  While they’re costly, and time consuming to set up, they’re also an essential part of the plan to maintain and scale.

Mirrored disks on a RAID array will restore data fine if one of the disks fails, but if you run rm -rf /path/to/raid/folder, then the RAID will just mirror that command on each of the disks.  Bye bye data 

I think it’s somewhat unfair to expect all users to keep full backups of their own data.  Not outrageous a demand, but you kinda expect at least some  form of data storage, so that your 6 years of precious thoughts and feelings aren’t lost into the ether one day.

Given that each user might have 6 years of blogposts, maybe 2 a week?

For 10,000 users, I make that about 600GB, averaging 100k per blogpost.

`(6*2*52*100*10000)`

Still, that’s not a vast amount.. Could fit that on a fairly small DLT tape.. Could easily replicate that across a few servers, seperated geographically, you could write it to a shitload of  DVDs, few dozen blurays, Shove it on a portable USB HDD and stick it in a fire safe in the CTO’s basement.

Hell, it’s still less than a terabyte of data.
