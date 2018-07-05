    module.exports = ({normalize_account,emit}) ->

For this map the reduce function is `_stats`:

      ({variables,callStats}) ->
        return unless variables?
        {start_stamp,billsec} = variables
        parts = start_stamp
          .split(/[- :]/)[0..3]
          .map (x) -> parseInt x, 10
        billsec = parseInt billsec, 10
        return if isNaN billsec

        xref = variables.session_reference
        direction = variables.ccnq_direction
        account = variables.ccnq_account
        account = normalize_account account if account? and normalize_account?

        skip = ->

System-wide stats

        skip parts, billsec
        skip [ {d:direction}, parts... ], billsec

        waitmsec = parseInt variables.waitmsec, 10
        if billsec is 0
          skip [ 'z', parts... ], waitmsec

By-call

        if xref?
          skip [ {x:xref}, parts... ], billsec

By-number

        number = variables.ccnq_from_e164
        if number?
          emit [ number, start_stamp ], billsec # legacy
          skip [ {n:number}, parts... ], billsec
          skip [ {n:number,d:direction}, parts... ], billsec
          if billsec is 0
            skip [ {n:number}, 'z', parts... ], waitmsec
        number = variables.ccnq_to_e164
        if number?
          emit [ number, start_stamp ], billsec # legacy
          skip [ {n:number}, parts... ], billsec
          skip [ {n:number,d:direction}, parts... ], billsec
          if billsec is 0
            skip [ {n:number}, 'z', parts... ], waitmsec

By-endpoint

        endpoint = variables.ccnq_endpoint
        inbound = callStats?.audio?.inbound
        if endpoint?
          skip [ {e:endpoint}, parts... ], billsec
          emit [ {e:endpoint,d:direction}, parts... ], billsec
          if inbound?
            skip [ {e:endpoint,d:direction}, 'm', parts... ], inbound.mos
            skip [ {e:endpoint,d:direction}, 's', parts... ], inbound.skip_packet_count
            skip [ {e:endpoint,d:direction}, 'j', parts... ], inbound.jitter_packet_count

By-account

        if account?
          skip [ {a:account}, parts... ], billsec
          skip [ {a:account,d:direction}, parts... ], billsec

By-"carrier"

On the client-side this will often be `sbc`; on the carrier-side it should be some reference to the carrier. (But this is merely a convention.)

        profile = variables.ccnq_profile
        if profile?
          skip [ {p:profile}, parts... ], billsec
          skip [ {p:profile,d:direction}, parts... ], billsec
          skip [ {p:profile,d:direction}, 'w', parts... ], waitmsec
          if billsec is 0
            skip [ {p:profile}, 'z', parts... ], waitmsec

          skip [ {p:profile,d:direction}, 'b', parts... ], parseInt variables.billmsec, 10
          skip [ {p:profile,d:direction}, 'p', parts... ], progressmsec = parseInt variables.progressmsec, 10
          skip [ {p:profile,d:direction}, 'a', parts... ], parseInt variables.answermsec, 10

        return
