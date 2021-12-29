---
title: "Not Storing Files in Databases"
date: "2012-01-29T20:00:00.000Z"
description: "Don't store files *in* databases."
warning: ancient
---

Originally a comment [here](https://web.archive.org/web/20120303041110/http://creativedev.in/2012/01/storing-a-file-in-database/)

In the above article, Bhumi gives a method for storing files *in* the database, using MySQL and PHP.

My personal distaste for PHP aside, I don't think I could ever find a reason to store files *in* the database, rather than *on* the filesystem.

I'm also primarily talking about RDBMS type databases, not NoSQL, which tend to have a mechanism for storing files a little bit more sanely than "old-fashioned" databases.

Let's take a look at this idea in a bit more depth.

If you read the original article, there's the very basic bare bones of a web application.  There's a form.  There's a MySQL table definition.  There's a bit of PHP for handling uploaded files.

Personally, I write pretty much exclusively in Python.  This isn't going to be a technical article with codesamples however, more just a look at comparing the methodologies of storing in a database to storing in a filesystem.

Why would you choose a BLOB database type over a Filesystem?  

I think there's gotta be about one use-case for storing files in a database.  Hang on.  I take that back.  There aren't any.

Filesystems are optimised for File Access.  Databases are best optimised for row/column based tabular data access.  The two should not be confused.

Here's how I store uploaded files:  

1. Upload the file to .../uploaded_files/...
1. Rename it to something sensible.  
1. In the database, store the original filename, and the path to the uploaded file.  
1. Possibly also store some neat metadata, call magic() on it, store the size, the MD5Sum, and so on.  

It greatly depends on the application, but it's conceivable that some of the file metadata might need to be retrieved in the file's lifespan within the application.  It's a lot quicker to stat the file once, and store the result, than it would be to stat it many times whenever the information is requested.

This makes a few drastic differences to database-based file storage.

1. The database contains data of a predictable size.  I mean, it's easier to calculate and predict the size of the Table based on the known bit-widths for each column.  Once you start introducing BLOBs into your database, all bets for size are off.
1. Database Backups are smaller. - Not having arbitrary binary data in your database means that gzipping a SQL dump is likely to be more deterministic than it would if you already had gzipped (for example) binary data stored as a BLOB.  When you attempt to compress already compressed data, the output is frequently larger.
1. You can scale easier by having a shared/clustered filesystem, such as Gluster.
1. If it's a website, loading files can happen externally of your webapp, simply have a media subdomain and handle files with a lightweight webserver such as LigHTTP or Nginx.

This means that your webapplication isn't a bottleneck for loading every damn file that's requested, into memory, stripping the slashes or base64_decoding it, before streaming it back to the user. 

This is actually one of the most important points.  If the data is stored in a proprietary format in the database, you can't use regular filesystem utilities to access it.  You need to do all that in your application. 

Why reinvent so many wheels when the OS-provided Filesystem tools are all so outstanding?

How would you stat the file inside a database? Write it out to `/tmp/tmpXXXXX` and then stat that, before deleting the temporary file?

Sounds a bit slow, to me.

What if the file is Huge? How long could that take? What if the uploaded file is bigger than your system RAM? Surely it'd make sense to be able to handle multiple large files...

What if your application breaks? Could it silently corrupt files on their way in / out? What if Something Bad Happens, and your database is partially corrupted.  Would all the files potentially be corrupted? What about recovery on a non-application-serving system.  Could be tricky potentially..  Certainly more tricky than just rsyncing files around.

See what I mean?  

You *can* store files in a database.  

Doesn't mean you should.