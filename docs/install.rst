************
Installation
************

ProMA in 1, 2, 3 (and 4)


Step 1: ProFTPd and MySQL
=========================

Install ProFTPd and MySQL as described in their respective documentation.

- `MySQL <http://www.mysql.com/>`_
- `ProFTPd <http://www.proftpd.org/>`_
- `mod_sql for ProFTPd
  <http://www.lastditcheffort.org/~aah/proftpd/mod_sql/>`_

The SQL-part from my proftpd.conf::

    SQLConnectInfo    database@127.0.0.1 user password
    SQLAuthenticate   users
    SQLAuthTypes      backend
    SQLDefaultHomedir /some/fancy/homedir
    SQLUserInfo       users userid passwd uid gid homedir shell
    SQLGroupInfo      groups groupname gid members
    SQLLog            PASS updatecount
    SQLNamedQuery     updatecount UPDATE "count=count+1 WHERE userid='%u'" users

The two last lines are needed for the login counter.

Check the `mod_sql documentation
<http://www.proftpd.org/docs/directives/linked/config_ref_mod_sql.html>`_ for
further information.


Step 2: Database setup
======================

Modify the ``users`` table and create an initial admin user. Run these queries
in the ``mysql`` client or in e.g. phpMyAdmin.


Step 2.1: Create/modify ``users`` table
---------------------------------------

mod_sql's README recommends this setup:

==============  ======  ===  =====  ========  ===============
COLUMN          TYPE    R    DUP    NULL?     USE
==============  ======  ===  =====  ========  ===============
userid          text    Y    N      N         user's login id
passwd          text    Y    Y      N         user's password
uid             num     N    N      Y         user's uid
gid             num     N    Y      Y         user's gid
homedir         text    N    Y      Y         user's homedir
shell           text    N    Y      Y         user's shell
==============  ======  ===  =====  ========  ===============

ProMA uses these column names as default, so if you use this setup you don't
have to change that in the config. Additionally ProMA needs these columns in
the user table:

==============  ======  ===  =====  ========  =====================
COLUMN          TYPE    R    DUP    NULL?     USE
==============  ======  ===  =====  ========  =====================
name            text    Y    Y      Y         user's full real name
mail            text    Y    Y      Y         user's mail address
note            text    N    Y      Y         note about user
count           num     N    Y      Y         user's login counter
admin           num     N    Y      Y         user's admin status
closed          num     N    Y      Y         account closed/open
==============  ======  ===  =====  ========  =====================

You can create this table with this query::

    CREATE TABLE users (
      userid varchar(255) NOT NULL UNIQUE,
      name varchar(255) NULL,
      mail varchar(255) NULL,
      uid int(11) NULL default '65534',
      gid int(11) NULL default '65534',
      passwd varchar(255) NOT NULL,
      shell varchar(255) NOT NULL default '/bin/nonexistent',
      homedir varchar(255) NOT NULL default '/some/fancy/homedir',
      note text NULL default '',
      count int(11) NOT NULL default 0,
      admin int(1) NOT NULL default 0,
      closed int(1) NOT NULL default 0
    );


Step 2.2: Make an admin user
----------------------------

Admin status is required for using the admin section of ProMA.
Thus, you need at least one initial admin user::

    INSERT INTO users
    SET
      userid = 'my_account_name',
      passwd = PASSWORD('my_password'),
      name = 'The Boss',
      mail = 'boss@my.ftp',
      admin = 1;

Or, if you already have an existing user you want to upgrade to
an admin::

    UPDATE users
    SET admin = 1
    WHERE userid = 'my_account_name';

You can add more admins once you're logged into the admin section.


Step 3: ProMA configuration
===========================

Copy ``config.inc.php-example`` to ``config.inc.php`` and edit it to fit your
setup and needs. There are more instructions in the configuration file.

``config.inc.php`` should not be world readable because it carries the password
to the MySQL database. For example, it is recommended (on a Debian system) to
run ``chgrp www-data config.inc.php`` and ``chmod 640 config.inc.php``.


Step 4: Enjoy
=============

Point your browser at the URL you installed ProMA at and enjoy.
