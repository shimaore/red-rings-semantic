    most = require 'most'
    Immutable = require 'immutable'
    {UPDATE,NOTIFY,SUBSCRIBE,UNSUBSCRIBE} = require 'red-rings/operations'

    describe 'The main module', ->
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

    describe 'The routed module', ->
      Routed = require '../routed'

      it 'should route create messages', (done) ->
        s = most.just Immutable.fromJS op: UPDATE, id:'none', doc: v: 'hello world', operations: []
        Routed(s).create.forEach (msg) -> done() if 'none' is msg.get('id') and not msg.has 'operations'
        return

      it 'should route updates', (done) ->
        s = most.just Immutable.fromJS op: UPDATE, id:'none', rev:'4', doc: v: 'hello world'
        Routed(s).update.forEach (msg) -> done() if 'none' is msg.get('id') and 'hello world' is msg.getIn ['operations',0,2]
        return

      it 'should route delete messages', (done) ->
        s = most.just Immutable.fromJS op: UPDATE, id:'none', rev: '4', doc: { v: 'hello world', _deleted: true }
        Routed(s).delete.forEach (msg) -> done() if 'none' is msg.get('id') and not msg.has 'doc'
        return

      it 'should route notify messages', (done) ->
        s = most.just Immutable.fromJS op: NOTIFY, key: 'some', id:'none', rev: '4', doc: { v: 'hello world' }, value: 'hello'
        Routed(s).notify.forEach (msg) -> done() if 'none' is msg.get 'id'
        return

      it 'should route subscribe messages', (done) ->
        s = most.just Immutable.fromJS op: SUBSCRIBE, key: 'some', id:'none', rev: '4', doc: { v: 'hello world' }, value: 'hello'
        Routed(s).subscribe.forEach (msg) -> done() if 'none' is msg.get 'id'
        return

      it 'should route unsubscribe messages', (done) ->
        s = most.just Immutable.fromJS op: UNSUBSCRIBE, key: 'some', id:'none', rev: '4', doc: { v: 'hello world' }, value: 'hello'
        Routed(s).unsubscribe.forEach (msg) -> done() if 'none' is msg.get 'id'
        return
