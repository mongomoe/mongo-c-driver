:man_page: mongoc_database_aggregate

mongoc_database_aggregate()
===========================

Synopsis
--------

.. code-block:: c

  mongoc_cursor_t *
  mongoc_database_aggregate (mongoc_database_t *database,
                             const bson_t *pipeline,
                             const bson_t *opts,
                             const mongoc_read_prefs_t *read_prefs)
     BSON_GNUC_WARN_UNUSED_RESULT;

Parameters
----------

* ``database``: A :symbol:`mongoc_database_t`.
* ``pipeline``: A :symbol:`bson:bson_t`, either a BSON array or a BSON document containing an array field named "pipeline".
* ``opts``: A :symbol:`bson:bson_t` containing options for the command, or ``NULL``.
* ``read_prefs``: A :symbol:`mongoc_read_prefs_t` or ``NULL``.

``opts`` may be NULL or a BSON document with additional command options:

* ``readConcern``: Construct a :symbol:`mongoc_read_concern_t` and use :symbol:`mongoc_read_concern_append` to add the read concern to ``opts``. See the example code for :symbol:`mongoc_client_read_command_with_opts`.
* ``writeConcern``: For aggregations that include "$out", you can construct a :symbol:`mongoc_write_concern_t` and use :symbol:`mongoc_write_concern_append` to add the write concern to ``opts``. See the example code for :symbol:`mongoc_client_write_command_with_opts`.
* ``sessionId``: Construct a :symbol:`mongoc_client_session_t` with :symbol:`mongoc_client_start_session` and use :symbol:`mongoc_client_session_append` to add the session to ``opts``. See the example code for :symbol:`mongoc_client_session_t`.
* ``bypassDocumentValidation``: Set to ``true`` to skip server-side schema validation of the provided BSON documents.
* ``collation``: Configure textual comparisons. See :ref:`Setting Collation Order <setting_collation_order>`, and `the MongoDB Manual entry on Collation <https://docs.mongodb.com/manual/reference/collation/>`_.
* ``serverId``: To target a specific server, include an int32 "serverId" field. Obtain the id by calling :symbol:`mongoc_client_select_server`, then :symbol:`mongoc_server_description_id` on its return value.
* ``batchSize``: To specify the number of documents to return in each batch of a response from the server, include an int "batchSize" field.

For a list of all options, see `the MongoDB Manual entry on the aggregate command <http://docs.mongodb.org/manual/reference/command/aggregate/>`_.

Description
-----------

This function creates a cursor which sends the aggregate command on the underlying database upon the first call to :symbol:`mongoc_cursor_next()`. For more information on building aggregation pipelines, see `the MongoDB Manual entry on the aggregate command <http://docs.mongodb.org/manual/reference/command/aggregate/>`_. Note that the pipeline must start with a compatible stage that does not require an underlying collection (e.g. "$currentOp", "$listLocalSessions").

Read preferences, read and write concern, and collation can be overridden by various sources. The highest-priority sources for these options are listed first in the following table. In a transaction, read concern and write concern are prohibited in ``opts`` and the read preference must be primary or NULL. Write concern is applied from ``opts``, or if ``opts`` has no write concern and the aggregation pipeline includes "$out", the write concern is applied from ``database``.

================== ============== ============== =========
Read Preferences   Read Concern   Write Concern  Collation
================== ============== ============== =========
``read_prefs``     ``opts``       ``opts``       ``opts``
Transaction        Transaction    Transaction
``database``       ``database``   ``database``
================== ============== ============== =========

:ref:`See the example for transactions <mongoc_client_session_start_transaction_example>` and for :ref:`the "distinct" command with opts <mongoc_client_read_command_with_opts_example>`.

Returns
-------

This function returns a newly allocated :symbol:`mongoc_cursor_t` that should be freed with :symbol:`mongoc_cursor_destroy()` when no longer in use. The returned :symbol:`mongoc_cursor_t` is never ``NULL``; if the parameters are invalid, the :symbol:`bson:bson_error_t` in the :symbol:`mongoc_cursor_t` is filled out, and the :symbol:`mongoc_cursor_t` is returned before the server is selected. The user must call :symbol:`mongoc_cursor_next()` on the returned :symbol:`mongoc_cursor_t` to execute the aggregation pipeline.

.. warning::

  Failure to handle the result of this function is a programming error.

Example
-------

.. code-block:: c

  #include <bson/bson.h>
  #include <mongoc/mongoc.h>

  static mongoc_cursor_t *
  current_op_query (mongoc_client_t *client)
  {
     mongoc_cursor_t *cursor;
     mongoc_database_t *database;
     bson_t *pipeline;

     pipeline = BCON_NEW ("pipeline",
                          "[",
                          "{",
                          "$currentOp",
                          "{",
                          "}",
                          "}",
                          "]");

     /* $currentOp must be run on the admin database */
     database = mongoc_client_get_database (client, "admin");

     cursor = mongoc_database_aggregate (
        database, pipeline, NULL, NULL);

     bson_destroy (pipeline);
     mongoc_database_destroy (database);

     return cursor;
  }
