    ({expect} = require 'chai').should()

    describe 'The module', ->
      it 'should load', -> require '..'

      it 'should contain a couchapp', ->
        {couchapp} = require '..'
        couchapp().should.have.property 'views'
        couchapp().views.should.have.property 'count'
        couchapp().views.count.should.have.property 'map'
        couchapp().views.count.should.have.property 'reduce', '_count'
        expect(couchapp().views.count.map).to.be.a 'string'
        couchapp().views.should.have.property 'stats'
        couchapp().views.stats.should.have.property 'map'
        couchapp().views.stats.should.have.property 'reduce', '_stats'
        expect(couchapp().views.stats.map).to.be.a 'string'

      it 'should contain a couchapp', ->
        {couchapp} = require '..'
        o = normalize_account: 'function (account) { return account }'
        couchapp(o).should.have.property 'views'
        couchapp(o).views.should.have.property 'count'
        couchapp(o).views.count.should.have.property 'map'
        couchapp(o).views.count.should.have.property 'reduce', '_count'
        expect(couchapp(o).views.count.map).to.be.a 'string'
        couchapp(o).views.should.have.property 'stats'
        couchapp(o).views.stats.should.have.property 'map'
        couchapp(o).views.stats.should.have.property 'reduce', '_stats'
        expect(couchapp(o).views.stats.map).to.be.a 'string'
