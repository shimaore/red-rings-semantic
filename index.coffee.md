Changes Semantic(s)
===================

Analyzes what operation was realized on the record/document by injecting a message in at most one of `.create`, `.update` or `.delete`.

    changes_semantic = (stream) ->

      source = stream
        .map cleanup
        .filter valid_id
        .multicast()

      create: source.filter(_create).multicast()
      update: source.filter(_update).multicast()
      delete: source.filter(_delete).multicast()

Message examples
----------------

The fields we're concerned with are:
- `id` / `doc._id`: should never start with `_`
- `rev` / `doc._rev`
- `doc`
- `operations`

`id` / `doc._id` is always required if this message is going to be interpreted as a changeset.

    valid_id = (msg) ->
      id = msg.get 'id'
      id? and typeof id is 'string' and id[0] isnt '_'

`rev` / `doc._rev` is required for `update` and `delete`.

    valid_rev = (msg) ->
      rev = msg.get 'rev'
      rev? and typeof rev is 'string'

The following descriptions indicate first how the message is normalized by the function before being sent.

### Create

Create requires a document field.

    _create = (msg) ->
      not msg.get('deleted') and not msg.has('rev') and
        msg.get('had_doc') and not msg.get('operations')

Normalized:

```
{
 id: "something"
 doc: { field: value, … }
}
```

Acceptable inputs:

```
{
 doc: { _id: "something", field: value, … }
}
```

### Update

Updates can be expressed with a document (denoting changes) and/or operations.

    _update = (msg) ->
      not msg.get('deleted') and valid_rev(msg) and
        (msg.get('had_doc') or msg.get('operations'))

Normalized:

```
{
 id: "something"
 rev: "my rev"
 doc: { field: value, … }
 operations: {
  "json-path": ["set",value]
  "json-path": ["set"]
  …
 }
}
```

Note: the `doc` field is a shortcut for the `set` operation; the difference with `operations` is that the key is a field name, not a JSON path.

Acceptable inputs:

```
{
 doc: { _id: "something", _rev: "my rev", field: value, … }
 operations: {
  "json-path": ["set",value]
  "json-path": ["set"]
  …
 }
}
```

### Delete

Deletion.

    _delete = (msg) ->
      msg.get('deleted') and valid_rev(msg) and
        not msg.get('operations')

Normalized:

```
{
 id: "something"
 rev: "my rev"
 deleted: true
}
```

Acceptable inputs:

```
{
 doc: {_id,_rev,_deleted:true}
}
```

Cleanup
-------

The message is cleaned-up before being routed.

    cleanup = (msg) -> msg.withMutations (msg) ->

Re-import the meta-data from the document (this allows clients to only specify `.doc` in most cases).

      if msg.has 'doc'
        msg = msg.set 'had_doc', true
        doc = msg.get 'doc'
      else
        msg = msg.set 'had_doc', false
        msg.set 'doc', Immutable.Map {}

`.id` and `.doc._id` are equivalent

      if doc?.has '_id' and not msg.has 'id'
        msg = msg.set 'id', doc.get '_id'

`.rev` and `.doc._rev` are equivalent

      if doc?.has '_rev' and not msg.has 'rev'
        msg = msg.set 'rev', doc.get '_rev'

Deletion might be indicated in the message metadata, in the document's `_deleted` field, or by a operation on the `_deleted` json-path.

      unless msg.has 'deleted'
        msg = msg.set 'deleted', false
      if doc?.get '_deleted'
        msg = msg.set 'deleted', true

      msg
      .removeIn ['doc','_id']
      .removeIn ['doc','_rev']
      .removeIn ['doc','_deleted']

    module.exports = changes_semantic
    Immutable = require 'immutable'
