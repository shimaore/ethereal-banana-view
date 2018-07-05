    describe 'A CDR', ->
      cdr = require './cdr.json'

      it 'should run through count', ->
        emit = console.log
        count = (require '../count') {emit}
        count cdr

      it 'should run through stats', ->
        emit = console.log
        stats = (require '../stats') {emit}
        stats cdr
