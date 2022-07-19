.. _ref_datamodel_globals:

=======
Globals
=======

.. note::

  ⚠️ Only available in EdgeDB 2.0 or later.

Schemas can contain scalar-typed *global variables*.

.. code-block:: sdl

  global current_user_id -> uuid;

These provide a useful mechanism for specifying session-level data that can be
referenced in queries with the ``global`` keyword.

.. code-block:: edgeql

  select User {
    id,
    posts: { title, content }
  }
  filter .id = global current_user_id;


As in the example above, this is particularly useful for representing the
notion of a session or "current user".

Setting global variables
^^^^^^^^^^^^^^^^^^^^^^^^

Global variables are set when initializing a client. The exact API depends on
which client library you're using.

.. tabs::

  .. code-tab:: typescript

    import createClient from 'edgedb';

    const baseClient = createClient()
    const clientWithGlobals = baseClient.withGlobals({
      current_user_id: '2141a5b4-5634-4ccc-b835-437863534c51',
    });

    await client.query(`select global current_user_id;`);

  .. code-tab:: python

    from edgedb import create_client

    client = create_client().with_globals({
        'current_user_id': '580cc652-8ab8-4a20-8db9-4c79a4b1fd81'
    })

    result = client.query("""
        select global current_user_id;
    """)
    print(result)


The ``.withGlobals/.with_globals`` method returns a new ``Client`` instance
that stores the provided globals and sends them along with all future queries.

Cardinality
-----------

Global variables can be marked ``required``; in this case, you must specify a
default value.

.. code-block:: sdl

  required global one_string -> str {
    default := "Hi Mom!"
  };

Computed globals
----------------

Global variables can also be computed. The value of computed globals are
dynamically computed when they are referenced in queries.

.. code-block:: sdl

  required global random_global := datetime_of_transaction();

The provided expression will be computed at the start of each query in which
the global is referenced. There's no need to provide an explicit type; the
type is inferred from the computed expression.

Computed globals are not subject to the same constraints as non-computed ones;
specifically, they can be object-typed and have a ``multi`` cardinality.

.. code-block:: sdl

  global current_user_id -> uuid;

  # object-typed global
  global current_user := (
    select User filter .id = global current_user_id
  );

  # multi global
  global current_user_friends := (global current_user).friends;


Usage in schema
---------------

.. You may be wondering what purpose globals serve that can't.
.. For instance, the simple ``current_user_id`` example above could easily
.. be rewritten like so:

.. .. code-block:: edgeql-diff

..     select User {
..       id,
..       posts: { title, content }
..     }
..   - filter .id = global current_user_id
..   + filter .id = <uuid>$current_user_id

.. There is a subtle difference between these two in terms of
.. developer experience. When using parameters, you must provide a
.. value for ``$current_user_id`` on each *query execution*. By constrast,
.. the value of ``global current_user_id`` is defined when you initialize
.. the client; you can use this "sessionified" client to execute
.. user-specific queries without needing to keep pass around the
.. value of the user's UUID.

.. But that's a comparatively marginal difference.

Unlike query parameters, globals can be referenced
*inside your schema declarations*.

.. code-block:: sdl

  type User {
    property name -> str;
    property is_self := (.id = global current_user_id)
  };

This is particularly useful when declaring :ref:`access policies
<ref_datamodel_ols>`.

.. code-block::

  type Person {
    required property name -> str;
    access policy my_policy allow all using (.id = global current_user_id);
  }

Refer to :ref:`Access Policies <ref_datamodel_ols>` for complete
documentation.


.. list-table::
  :class: seealso

  * - **See also**
  * - :ref:`SDL > Object types <ref_eql_sdl_object_types>`
  * - :ref:`DDL > Object types <ref_eql_ddl_object_types>`
  * - :ref:`Introspection > Object types <ref_eql_introspection_object_types>`
  * - :ref:`Cheatsheets > Object types <ref_cheatsheet_object_types>`