    most = require 'most'
    Immutable = require 'immutable'
    {UPDATE} = require 'red-rings/operations'

    describe 'The module', ->
      changes_semantic = require '..'

      it 'should filter create messages', (done) ->
        s = most.just Immutable.fromJS op: UPDATE, id:'none', doc: v: 'hello world'
        changes_semantic(s).create.forEach (msg) -> done() if 'none' is msg.get 'id'
        return

      it 'should filter update messages', (done) ->
        s = most.just Immutable.fromJS op: UPDATE, id:'none', rev: '4', doc: v: 'hello world'
        changes_semantic(s).update.forEach (msg) -> done() if 'none' is msg.get 'id'
        return

      it 'should filter delete messages', (done) ->
        s = most.just Immutable.fromJS op: UPDATE, id:'none', rev: '4', deleted: true
        changes_semantic(s).delete.forEach (msg) -> done() if 'none' is msg.get 'id'
        return
