    module.exports = ({normalize_account,emit}) ->

      skip = ->

For this map the reduce function is `_count`:

      ({variables}) ->
        return unless variables?
        {start_stamp,billsec} = variables
        parts = start_stamp
          .split(/[- :]/)[0..3]
          .map (x) -> parseInt x, 10

        xref = variables.session_reference
        direction = variables.ccnq_direction
        account = variables.ccnq_account

        number = variables.ccnq_from_e164
        if number?
          emit [ number, start_stamp ] # legacy
          skip [ {n:number}, 'x', parts... ], {xref}
          skip [ {n:number,d:direction}, parts... ], {xref}
        number = variables.ccnq_to_e164
        if number?
          emit [ number, start_stamp ] # legacy
          skip [ {n:number}, 'x', parts... ], {xref}
          skip [ {n:number,d:direction}, parts... ], {xref}

        return
