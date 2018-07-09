    module.exports = ({normalize_account,is_emergency,emit}) ->

For this map the reduce function is `_count`:

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

The main goal here is to avoid having to deal with `include_docs` etc.

        value =
          direction: direction
          from: variables.ccnq_from_e164
          to: variables.ccnq_to_e164
          duration: billsec
          start_stamp: start_stamp

By-number

        number = variables.ccnq_from_e164
        if number?
          emit [ ALL+dir, {n:number}, parts... ], value
        number = variables.ccnq_to_e164
        if number?
          emit [ ALL+dir, {n:number}, parts... ], value

By-endpoint ("agent")

        endpoint = variables.ccnq_endpoint
        if endpoint?
          emit [ ALL+dir, {e:endpoint}, parts... ], value

By-account

        ###
        account = variables.ccnq_account
        account = normalize_account account if account? and normalize_account?
        if account?
          emit [ ALL+dir, {a:account}, parts... ], value
        ###

By-"carrier"

On the client-side this will often be `sbc`; on the carrier-side it should be some reference to the carrier. (But this is merely a convention.)

        ###
        profile = variables.ccnq_profile
        if profile?
          emit [ ALL+dir, {p:profile}, parts... ], value
        ###

        return
