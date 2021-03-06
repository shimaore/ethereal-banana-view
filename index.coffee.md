    app_name = 'ethereal-banana'

This is the version number for the Couch app / design document itself.
Notice that it doesn't contain a patch-level indication (because we
can't do resolution), but the major and minor should follow semver
(minor change = new functionality with no changes to existing
functionalities, major change = changes to existing functionality).

    {version} = require './package.json'
    {major,minor} = require 'semver'

    app_version = "#{major version}.#{minor version}"

Ensure backward-compatibility between patches and minors!

- Do not change the content or the length of existing keys.
- Do not change the set returned for a given query.
- Only extend values (this is only possible if values are objects).

    app = "#{app_name}-#{app_version}"

Current views
-------------

    {p_fun} = require 'coffeescript-helpers'

    couchapp = (options) ->
      options ?= {}
      extra = """
        var normalize_account = #{options.normalize_account ? null};
        var is_emergency = #{options.is_emergency ? null};
      """

      stats = require('./stats') options
      count = require('./count') options

      _id: "_design/#{app}"
      version: version
      language: 'javascript'
      views:
        stats:
          map: p_fun extra, stats
          reduce: '_stats'
        count:
          map: p_fun extra, count
          reduce: '_count'

    module.exports = {app,app_version,couchapp}
