Map
---

Transform all updates by removing the `.doc` and putting those operations into `.operations`. (This way we only get to validate `.operations`.)

    prepend_set = (operations,v,k) ->
      path = Immutable.List [k]
      operations.unshift Immutable.List ['set', path, v]

    all_operations = (msg) ->
      doc = msg.get 'doc'
      return msg unless doc?
      unless msg.get 'operations'
        msg = msg.set 'operations', Immutable.List []
      msg
      .update 'operations', (operations) ->
        doc.reduce prepend_set, operations
      .remove 'doc'

Similarly for create and delete.

    remove_operations = (msg) -> msg.remove 'operations'
    remove_changes = (msg) -> remove_operations(msg).remove 'doc'

Transform
---------

All updates are cleaned through changes-semantic (which means at least that the document does not contain `_id`, `_rev`, nor `_deleted`).

    Routed = (source) ->

      cs = source
        .filter operation UPDATE
        .thru changes_semantic

      create: cs.create.map remove_operations
      update: cs.update.map all_operations
      delete: cs.delete.map remove_changes
      notify:       source.filter operation NOTIFY
      subscribe:    source.filter operation SUBSCRIBE
      unsubscribe:  source.filter operation UNSUBSCRIBE

    module.exports = Routed
    changes_semantic = require './index'
    Immutable = require 'immutable'
    {UPDATE,NOTIFY,SUBSCRIBE,UNSUBSCRIBE} = require 'red-rings/operations'
    {operation} = require 'abrasive-ducks-transducers'
