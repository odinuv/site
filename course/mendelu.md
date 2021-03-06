---
title: MENDELU environment
permalink: /course/mendelu/
---

* TOC
{:toc}

This section is dedicated to description of MENDELU environment, especially to the usage of Akela server.

## Connecting to Akela server via SSH
You can use [SSH - Secure Shell](https://cs.wikipedia.org/wiki/Secure_Shell) to transfer files and to actually execute
commands on remote computer. Read [technical support section](/course/technical-support/) where you can find more
about required software, here is just a list:

- [WinSCP](https://winscp.net/eng/download.php)
- [Putty](https://www.putty.org/)
- [FileZilla](https://filezilla-project.org/)
- Linux has `ssh user@server` command usually available in basic installation

The important address is `akela.mendelu.cz`. Login and password is same as your UIS credentials.

{: .alert}
Akela server is not suitable as serious application hosting because anybody can read your database credentials
from source files. Static or database-less presentations are OK.

## Accessing your web presentation on Akela server
Each student has a profile folder on Akela server. Let's say, that your surname is "Smith", the university assigned you
with a login `xsmith12`. Therefore your home folder on Akela server is `/export/home/xsmith12`.

To create a public WWW presentation, you have to create a folder `public_html` in your home folder. The final path is
`/export/home/xsmith12/public_html`. Anything placed into this folder is **public** and is eventually crawled by Google.

To access your public presentation use this address: `https://akela.mendelu.cz/~xsmith12/`

The disadvantage is, that all uploaded files and folders are publicly visible. You have to configure Apache HTTP server
to [hide](/course/technical-support/#set-a-directory-as-inaccessible-from-the-internet) folders and
[disable access](/articles/web-applications/environments/#the-env-file) to `.env` file.

{: .note}
Do not reject read access to this folder for other users of Akela server. It would block Apache HTTP server and your
presentation would stop working. 

### Using devel folder
You can create a subfolder called `devel` in your `public_html` folder. Devel folder is used for your development
purposes and is not accessible publicly from internet (you have to know your UIS password to access it).

The public HTTP path is `https://akela.mendelu.cz/~xsmith12/devel/`

You do not have to worry about `.env` file inside this folder, nobody can open it using web browser.

### Accessing other students' or teachers' folders
It is possible to browse other users' folders and files. You can block others by setting the read/write/execute
permissions using WinSCP/FileZilla or `chmod` command.
 
## Using database administration tools on Akela server
There are actually two installed DB administration tools:

- [https://akela.mendelu.cz/db/adminer/](https://akela.mendelu.cz/db/adminer/)
- [https://akela.mendelu.cz/db/pgadmin/](https://akela.mendelu.cz/db/pgadmin/) - older, but has working export for PostgreSQL

Current version of Adminer is a bit old and is updated irregularly. You can download your own copy of
[Adminer](https://www.adminer.org/) and place it into your `public_html` folder if you wish. Remember to select
*PostgreSQL* database type when logging in with Adminer.

There is network tunnel between Akela server and real database server -- therefore the hostname of database server
is `localhost`.

The database account is created separately when you start to study APV course and the password is **NOT** same as UIS
password. The default password is disclosed to you by the teacher in the beginning of the course. Database login is
same as your UIS login and also the name of your dedicated database.

{: .alert}
Do **not** change the default password to UIS password because source files which contain the password are readable by
your classmates and other Akela server users. Use **unique** password which you do not use anywhere else.

You can change database password with this SQL command: `ALTER USER your_login WITH PASSWORD 'new_password'`

## Accessing database from PHP scripts
The credentials and configurations are (for user `xsmith12` with database password `garfield`):

```
DB_TYPE=pgsql
DB_HOST=localhost
DB_USER=xsmith12
DB_PASS=garfield
DB_NAME=xsmith12
```

The database name is same as your login.

## Running Composer on Akela server

{: .alert}
This is currently impossible as Akela server has some problem with accessing HTTPs resources. 

If you do not have [Composer](https://getcomposer.org/) [installed on your own PC](https://getcomposer.org/doc/00-intro.md)
(you also need to install [PHP](http://php.net/downloads.php) as command line script interpreter and it can be a bit
difficult for beginners), you can execute Composer on Akela server:

- Download `composer.phar` file from their [site](https://getcomposer.org/download/) in the "Manual Download" section.
- Upload the file using WinSCP/FileZilla to your project root (where `composer.json` and `composer.lock` files are
  located).
- Connect to Akela using SSH (use putty on Windows or `ssh` command on Linux).
- Use `cd` to access your project folder.
- Execute `php composer.phar install` or `php composer.phar require lib` or whatever. You are actually telling PHP to
  execute that *phar* file. Other options and switches are interpreted by Composer application.
- Your `vendor` folder should be updated.

{: .note}
The *phar* file extension stands for ["PHP archive"](http://php.net/manual/en/book.phar.php) and is actually a bundle
of multiple PHP scripts executable with PHP interpreter. It means, that Composer itself is written in PHP.
