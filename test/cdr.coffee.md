    describe 'A CDR', ->
      cdr = require './cdr.json'

      it 'should run through count', ->
        counted = 0
        emit = (key,value) ->
          console.log key, value
          counted++
        count = (require '../count') {emit}
        count cdr
        strictEqual 3, counted

      it 'should run through stats', ->
        counted = 0
        emit = (key,value) ->
          console.log key, value
          counted++
        stats = (require '../stats') {emit}
        stats cdr
        strictEqual 4, counted

    {strictEqual} = require 'assert'
