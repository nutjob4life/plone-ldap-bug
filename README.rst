******************************
 LDAP Role Regression - 12094
******************************

This is an isolated test suite that demonstrates the LDAP Role Regression (bug
ID 12094_).  In Plone_ 4.0, it works as expected.  In 4.1, though, it does
not.


Issue Description
=================

To summarize: LDAP users with Manager role cannot see private items in Plone
4.1 (works in 4.0).

Content in the default workflow that is in the "private" state should be
visible in folder listings, catalog queries, etc., to authenticated users who
have the "Manager" Zope role, even when such users are sourced from LDAP and
the role is conferred by an LDAP-to-Zope role mapping.  Indeed, this is the
case with Plone 4.0, and it works great: managers can see at a glance private
items in red and published items in blue.

However, in Plone 4.1, this doesn't work anymore.  In Plone 4.1, users from
LDAP who have the "Manager" role do not see "private" content in folders.  In
folder contents, red items don't appear for such users.


Reproduction
============

To reproduce the bug using the files provided here, you'll need to be on a
relatively Unixy system.  You'll also need Python_ 2.6, OpenLDAP_ 2.3 or so,
and have an active internet connection.

Edit the file ``local.cfg`` in the same directory as this readme file to
reflect your systems' local paths.  Once that's done, do the following:

1. Set up a test LDAP server.
2. Start up Plone 4.0 and confirm the correct behavior.
3. Start up Plone 4.1 and observe the incorrect behavior.

The rest of this document details the above steps.


Setting Up LDAP
---------------

The ``ldap`` subdirectory provides a configuration and `test data`_ for a
testing OpenLDAP standalone LDAP server or ``slapd``.  It includes:

* A single test user ``vivian`` with password ``twit``.
* A single test group ``Managers``, with a single member, ``vivian``.

To set it up:

1. Make ``ldap`` the current working directory.
2. Run ``python2.6 bootstrap.py -d``
3. Run ``bin/buildout``

The buildout will configure an OpenLDAP server, load the test data, and
generate a launch script.

To launch the server, run: ``bin/slapd``.  Note that the server does *not*
daemonize itself; it will run in foreground.

You can test if the server is running by executing an authenticated (bound)
query with the ``ldapsearch`` command.  Try running::

    ldapsearch -H ldap://localhost:PORT -x -b dc=localhost -D uid=vivian,dc=localhost -W '(uid=vivian)'

Replace PORT with the port number you set in ``local.cfg`` (default 10389).
When prompted for a password, enter ``twit``.  You should see the LDAP entry
for ``vivian``.


Plone 4.0
---------

Plone 4.0 currently demonstrates the correct behavior.  If you don't need any
convincing, skip to the next section, "Plone 4.1".

To set up Plone 4.0, do the following:

1. Make ``plone-4.0`` the current working directory.
2. Run ``python2.6 bootstrap.py -d``
3. Run ``bin/buildout``
4. Start Plone with ``bin/plone fg``

Plone 4.0 is now up and running on http://localhost:1040/Plone (substitute
port numbers if you edited ``local.cfg``), and the plone.app.ldap add-on is
already activated.  You can then set up test content and LDAP as described
below.


Plone 4.1
---------

Plone 4.1 currently exhibits incorrect behavior.  To set up Plone 4.1, do the
following:

1. Make ``plone-4.1`` the current working directory.
2. Run ``python2.6 bootstrap.py -d``
3. Run ``bin/buildout``
4. Start Plone with ``bin/plone fg``

Plone 4.1 is now up and running on http://localhost:1041/Plone (substitute
port numbers if you edited ``local.cfg``), and the plone.app.ldap add-on is
already activated.  Continue into the next section to set up test content,
configure LDAP, and expose the bug.


Exposing the Bug
----------------

To demonstrate the correct behavior (in Plone 4.0) and expose the bug (in
Plone 4.1), do the following steps on each Plone site deployed above:

1.  Visit the Plone site with a browser (http://localhost:1040/Plone for Plone
    4.0, http://localhost:1041/Plone for Plone 4.1, excepting adjustments you
    may have made to ``local.cfg``).
2.  Click "Log in" and log in with the Zope username ``admin`` and password
    ``admin``.
3.  Open the "Add new" menu, and add a new Page.  Give it any title, summary,
    and body text.  Click Save.  Leave the page in the private state and note
    its appearance in the global navigation bar.
4.  Click the "admin" username in the upper right and click "Site Setup".
5.  Click "LDAP Connection".
6.  Enter/select the following parameters:

======================= =========================
Parameter               Value
----------------------- -------------------------
LDAP server type        LDAP
rDN attribute           uid
User id attribute       uid
Login name attribute    uid
LDAP object classes     person
Bind DN                 (leave blank)
Bind password           (leave blank)
Base DN for users       dc=localhost
Search scope for users  one level
Base DN for groups      dc=localhost
Search scope for groups one level
User password encrypt   SHA
Default user roles      Anonymous
======================= =========================

7.  Click Save.
8.  Click "Up to Site Setup", then "Zope Management Interface"
9.  Click "acl_users", then "ldap-plugin".
10. Click the "Contents" tab, then the inner "acl_users" object.
11. Click the "LDAP Servers" tab, then under "Add LDAP Server", enter the
    following parameters (adjust the port number if needed):

======================= =========================
Parameter               Value
----------------------- -------------------------
Server host, IP         localhost
Server port             10389
Protocol                LDAP
Connection timeout      5 seconds
Operation timeout       10 seconds
======================= =========================

12. Click "Add Server".
13. Click the "Groups" tab.  Under "Add LDAP group to Zope role mapping",
    select "Managers" as the LDAP group, and "Manager" as the Zope role, then
    click Add.
14. Return to the home page of the Plone site.
15. Click the "admin" username in the upper right and click "Log out".

You can now expose the bug: log in with user name ``vivian`` and password
``twit``.  On the Plone 4.0 site, the private page in step 3 is visible.  On
the Plone 4.1 site, it is not.


.. References:
.. _12094: https://dev.plone.org/plone/ticket/12094
.. _OpenLDAP: http://openldap.org/
.. _Plone: http://plone.org/
.. _Python: http://python.org/
.. _`test data`: http://en.wikipedia.org/wiki/Upper_Class_Twit_of_the_Year
