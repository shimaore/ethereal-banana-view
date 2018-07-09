    module.exports = ({normalize_account,is_emergency,emit}) ->

For this map the reduce function is `_stats`:

      ({variables,callStats}) ->
        return unless variables?

        ALL = 'A' # returns billsec
        ZEROES = 'Z' # return waitmsec

        INGRESS = 'I'
        EGRESS = 'E'

        direction = variables.ccnq_direction

        switch direction
          when 'ingress'
            dir = INGRESS
          when 'egress'
            dir = EGRESS
          else
            return

        {start_stamp,billsec} = variables
        billsec = parseInt billsec, 10
        return if isNaN billsec

Emergency calls should _never_ make it through.

        return if is_emergency? variables
        if variables.ccnq_attrs?
          attrs = {}
          try
            attrs = JSON.parse variables.ccnq_attrs
          return if attrs.emergency

        parts = start_stamp
          .split(/[- T:]/)[0...4]
          .map (x) -> parseInt x, 10

        ###
        month = start_stamp[0...7]
        ###

        waitmsec = parseInt variables.waitmsec, 10

        MOS = 'm'
        SKIP = 's'
        JITTER = 'j'
        WAIT = 'w' # returns waitmsec
        BILLABLE = 'b' # return billmsec
        PROGRESS = 'p' # return progressmsec
        ANSWER = 'a' # return answermsec

System-wide stats

        ###
        emit [ ALL+dir, parts... ], billsec

        if billsec is 0
          emit [ ZEROES+dir, parts... ], waitmsec
        ###

By-call

        xref = variables.session_reference
        if xref?
          emit {x:xref}, billsec

By-number

        number = variables.ccnq_from_e164
        if number?
          emit [ ALL+dir, {n:number}, parts... ], billsec
          ###
          if billsec is 0
            emit [ ZEROES+dir, {n:number}, parts... ], waitmsec
          ###
        number = variables.ccnq_to_e164
        if number?
          emit [ ALL+dir, {n:number}, parts... ], billsec
          ###
          if billsec is 0
            emit [ ZEROES+dir, {n:number}, parts... ], waitmsec
          ###

By-endpoint ("agent")

        endpoint = variables.ccnq_endpoint
        if endpoint?
          emit [ ALL+dir, {e:endpoint}, parts... ], billsec
          ###
          inbound = callStats?.audio?.inbound
          if inbound?
            emit [ MOS+dir, {e:endpoint}, parts... ], inbound.mos
            emit [ SKIP+dir, {e:endpoint}, parts... ], inbound.skip_packet_count
            emit [ JITTER+dir, {e:endpoint}, parts... ], inbound.jitter_packet_count
          ###

By-account

        ###
        account = variables.ccnq_account
        account = normalize_account account if account? and normalize_account?
        if account?
          emit [ ALL+dir, {a:account}, parts... ], billsec
        ###

By-"carrier"

On the client-side this will often be `sbc`; on the carrier-side it should be some reference to the carrier. (But this is merely a convention.)

        ###
        profile = variables.ccnq_profile
        if profile?
          emit [ ALL+dir, {p:profile}, parts... ], billsec
          if billsec is 0
            emit [ ZEROES+dir, {p:profile}, parts... ], waitmsec

          progressmsec = parseInt variables.progressmsec, 10
          answermsec = parseInt variables.answermsec, 10
          emit [ WAIT+dir,     {p:profile}, parts... ], waitmsec
          emit [ BILLABLE+dir, {p:profile}, parts... ], parseInt variables.billmsec, 10
          emit [ PROGRESS+dir, {p:profile}, parts... ], progressmsec
          emit [ ANSWER+dir,   {p:profile}, parts... ], answermsec

Find all calls with a progress time within a range, for a given day.
Kind of useless without multi-dimensional indexing, though.

          day = start_stamp.split(/[ T]/)[0]
          emit [ PROGRESS+dir, {p:profile}, progressmsec, day ], billsec
          emit [ ANSWER+dir,   {p:profile}, answermsec, day ], billsec

        ###

        return
