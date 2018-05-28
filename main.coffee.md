    module.exports = ({normalize_account,emit}) ->

For this map the reduce function is `_stats`:

      stats: ({variables,callStats}) ->
        return unless variables?
        {start_stamp,billsec} = variables
        parts = start_stamp
          .split /[- :]/
          .map (x) -> parseInt x, 10
        billsec = parseInt billsec, 10, 10
        return if isNaN billsec

        xref = variables.session_reference
        direction = variables.ccnq_direction
        account = variables.ccnq_account
        account = normalize_account account if account? and normalize_account?

System-wide stats

        emit parts, billsec
        emit [ {direction}, parts... ], billsec

        waitmsec = parseInt variables.waitmsec, 10
        if billsec is 0
          emit [ 'zero', parts... ], waitmsec

By-call

        if xref?
          emit [ {xref}, parts... ], billsec

By-number

        number = variables.ccnq_from_e164
        if number?
          emit [ {number}, parts... ], billsec
          emit [ {number,direction}, parts... ], billsec
          if billsec is 0
            emit [ {number}, 'zero', parts... ], waitmsec
        number = variables.ccnq_to_e164
        if number?
          emit [ {number}, parts... ], billsec
          emit [ {number,direction}, parts... ], billsec
          if billsec is 0
            emit [ {number}, 'zero', parts... ], waitmsec

By-endpoint

        endpoint = variables.ccnq_endpoint
        inbound = callStats?.audio?.inbound
        if endpoint?
          emit [ {endpoint}, parts... ], billsec
          emit [ {endpoint,direction}, parts... ], billsec
          if inbound?
            emit [ {endpoint,direction}, 'mos', parts... ], inbound.mos
            emit [ {endpoint,direction}, 'skip_packet_count', parts... ], inbound.skip_packet_count
            emit [ {endpoint,direction}, 'jitter_packet_count', parts... ], inbound.jitter_packet_count

By-account

        if account?
          emit [ {account}, parts... ], billsec
          emit [ {account,direction}, parts... ], billsec

By-"carrier"

On the client-side this will often be `sbc`; on the carrier-side it should be some reference to the carrier. (But this is merely a convention.)

        profile = variables.ccnq_profile
        if profile?
          emit [ {profile}, parts... ], billsec
          emit [ {profile,direction}, parts... ], billsec
          emit [ {profile,direction}, 'waitmsec', parts... ], waitmsec
          if billsec is 0
            emit [ {profile}, 'zero', pars... ], waitmsec

          emit [ {profile,direction}, 'billmsec', parts... ], parseInt variables.billmsec, 10
          emit [ {profile,direction}, 'progressmsec', parts... ], progressmsec = parseInt variables.progressmsec, 10
          emit [ {profile,direction}, 'answermsec', parts... ], parseInt variables.answermsec, 10

        return

For this map the reduce function is `_count`:

      count: ({variables}) ->
        return unless variables?
        {start_stamp,billsec} = variables
        parts = start_stamp
          .split /[- :]/
          .map (x) -> parseInt x, 10

        xref = variables.session_reference
        direction = variables.ccnq_direction
        account = variables.ccnq_account

        if number = variables.ccnq_from_e164?
          emit [ {number}, 'xref', parts... ], {xref}
          emit [ {number,direction}, parts... ], {xref}
        if number = variables.ccnq_to_e164?
          emit [ {number}, parts... ], {xref}
          emit [ {number,direction}, parts... ], {xref}

        return
