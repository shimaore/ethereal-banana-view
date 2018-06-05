    module.exports = ({normalize_account,emit}) ->

For this map the reduce function is `_count`:

      ({variables}) ->
        return unless variables?
        {start_stamp,billsec} = variables
        parts = start_stamp
          .split /[- :]/
          .map (x) -> parseInt x, 10

        xref = variables.session_reference
        direction = variables.ccnq_direction
        account = variables.ccnq_account

        number = variables.ccnq_from_e164
        if number?
          emit [ {number}, 'xref', parts... ], {xref}
          emit [ {number,direction}, parts... ], {xref}
        number = variables.ccnq_to_e164
        if number?
          emit [ {number}, parts... ], {xref}
          emit [ {number,direction}, parts... ], {xref}

        return
