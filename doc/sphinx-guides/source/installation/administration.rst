Administration
==============

This section focuses on system and database administration tasks. Please see the :doc:`/user/index` for tasks having to do with having the "Admin" role on a dataverse or dataset.

.. contents:: :local:

Solr Search Index
-----------------

Dataverse requires Solr to be operational at all times. If you stop Solr, you should see a error about this on the home page, which is powered by the search index Solr provides. You set up Solr by following the steps in the :doc:`prerequisites` section and the :doc:`config` section explained how to configure it. This section is about the care and feeding of the search index. PostgreSQL is the "source of truth" and the Dataverse application will copy data from PostgreSQL into Solr. For this reason, the search index can be rebuilt at any time but depending on the amount of data you have, this can be a slow process. You are encouraged to experiment with production data to get a sense of how long a full reindexing will take.

Full Reindex
++++++++++++

There are two ways to perform a full reindex of the Dataverse search index. Starting with a "clear" ensures a completely clean index but involves downtime. Reindexing in place doesn't involve downtime but does not ensure a completely clean index.

Clear and Reindex
~~~~~~~~~~~~~~~~~

Clearing Data from Solr
.......................

Please note that the moment you issue this command, it will appear to end users looking at the home page that all data is gone! This is because the home page is powered by the search index.

``curl http://localhost:8080/api/admin/index/clear``

Start Async Reindex
...................

Please note that this operation may take hours depending on the amount of data in your system. This known issue is being tracked at https://github.com/IQSS/dataverse/issues/50

``curl http://localhost:8080/api/admin/index``

Reindex in Place
~~~~~~~~~~~~~~~~

An alternative to completely clearing the search index is to reindex in place.

Clear Index Timestamps
......................

``curl -X DELETE http://localhost:8080/api/admin/index/timestamps``

Start or Continue Async Reindex
................................

If indexing stops, this command should pick up where it left off based on which index timestamps have been set, which is why we start by clearing these timestamps above. These timestamps are stored in the ``dvobject`` database table.

``curl http://localhost:8080/api/admin/index/continue``

Glassfish
---------

``server.log`` is the main place to look when you encounter problems. Hopefully an error message has been logged. If there's a stack trace, it may be of interest to developers, especially they can trace line numbers back to a tagged version.

For debugging purposes, you may find it helpful to increase logging levels as mentioned in the :doc:`/developers/debugging` section of the Developer Guide.

This guide has focused on using the command line to manage Glassfish but you might be interested in an admin GUI at http://localhost:4848

Monitoring
----------

In production you'll want to monitor the usual suspects such as CPU, memory, free disk space, etc.

https://github.com/IQSS/dataverse/issues/2595 contains some information on enabling monitoring of Glassfish, which is disabled by default.

There is a database table called ``actionlogrecord`` that captures events that may be of interest. See https://github.com/IQSS/dataverse/issues/2729 for more discussion around this table.

User Administration
-------------------

There isn't much in the way of user administration tools built in to Dataverse.

Deleting an API Token
+++++++++++++++++++++

If an API token is compromised it should be deleted. Users can generate a new one for themselves as explained in the :doc:`/user/account` section of the User Guide, but you may want to preemptively delete tokens from the database.

Using the API token 7ae33670-be21-491d-a244-008149856437 as an example:

``delete from apitoken where tokenstring = '7ae33670-be21-491d-a244-008149856437';``

You should expect the output ``DELETE 1`` after issuing the command above.

