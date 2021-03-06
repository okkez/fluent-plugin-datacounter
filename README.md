# fluent-plugin-datacounter

This is a plugin for [Fluentd](http://fluentd.org)

## Component

### DataCounterOutput

Count messages with data matches any of specified regexp patterns in specified attribute.

- Counts per min/hour/day
- Counts per second (average every min/hour/day)
- Percentage of each pattern in total counts of messages

DataCounterOutput emits messages contains results data, so you can output these message (with 'datacount' tag by default) to any outputs you want.

    output ex1 (aggregates all inputs): {"pattern1_count":20, "pattern1_rate":0.333, "pattern1_percentage":25.0, "pattern2_count":40, "pattern2_rate":0.666, "pattern2_percentage":50.0, "unmatched_count":20, "unmatched_rate":0.333, "unmatched_percentage":25.0}
    output ex2 (aggregates per tag): {"test_pattern1_count":10, "test_pattern1_rate":0.333, "test_pattern1_percentage":25.0, "test_pattern2_count":40, "test_pattern2_rate":0.666, "test_pattern2_percentage":50.0, "test_unmatched_count":20, "test_unmatched_rate":0.333, "test_unmatched_percentage":25.0}

'input_tag_remove_prefix' option available if you want to remove tag prefix from output field names.

## Configuration

### DataCounterOutput

Count messages that have attribute 'referer' as 'google.com', 'yahoo.com' and 'facebook.com' from all messages matched, per minutes.

    <match accesslog.**>
      type datacounter
      unit minute
      aggregate all
      count_key referer
      # patternX: X(1-20)
      pattern1 google google.com
      pattern2 yahoo  yahoo.com
      pattern3 facebook facebook.com
      # but patterns above matches 'this-is-facebookXcom.xxxsite.com' ...
    </match>

Or, more exact match pattern, output per tags (default 'aggregate tag'), per hours.

    <match accesslog.**>
      type datacounter
      unit hour
      count_key referer
      # patternX: X(1-20)
      pattern1 google ^http://www\.google\.com/.*
      pattern2 yahoo  ^http://www\.yahoo\.com/.*
      pattern3 twitter ^https://twitter.com/.*
    </match>

HTTP status code patterns.

    <match accesslog.**>
      type datacounter
      count_interval 1m    # just same as 'unit minute' and 'count_interval 60s'
                           # you can also specify '30s', '5m', '2h' ....
      count_key status
      # patternX: X(1-20)
      pattern1 2xx ^2\d\d$
      pattern2 3xx ^3\d\d$
      pattern3 404 ^404$    # we want only 404 counts...
      pattern4 4xx ^4\d\d$  # pattern4 doesn't matches messages matches pattern[123]
      pattern5 5xx ^5\d\d$
    </match>

If you want not to include 'unmatched' counts into percentage, use 'outcast_unmatched' configuration:

    <match accesslog.**>
      type datacounter
      count_key status
      # patternX: X(1-20)
      pattern1 2xx ^2\d\d$
      pattern2 3xx ^3\d\d$
      pattern3 4xx ^4\d\d$
      pattern4 5xx ^5\d\d$
      outcast_unmatched yes # '*_percentage' fields culculated without 'unmatched' counts, and
                            # 'unmatched_percentage' field will not be produced
    </match>

With 'output_per_tag' option and 'tag_prefix', we get one result message for one tag, like below:

    <match accesslog.{foo,bar}>
      type datacounter
      count_key status
      pattern1 OK ^2\d\d$
      pattern2 NG ^\d\d\d$
      input_tag_remove_prefix accesslog
      output_per_tag yes
      tag_prefix status
    </match>
    # => tag: 'status.foo' or 'status.bar'
    #    message: {'OK_count' => 60, 'OK_rate' => 1.0, 'OK_percentage' => 70, 'NG_count' => , ....}

And you can get tested messages count with 'output_messages' option:

    <match accesslog.{foo,bar}>
      type datacounter
      count_key status
      pattern1 OK ^2\d\d$
      pattern2 NG ^\d\d\d$
      input_tag_remove_prefix accesslog
      output_messages yes
    </match>
    # => tag: 'datacount'
    #    message: {'foo_messages' => xxx, 'foo_OK_count' => ... }
    
    <match accesslog.baz>
      type datacounter
      count_key status
      pattern1 OK ^2\d\d$
      pattern2 NG ^\d\d\d$
      input_tag_remove_prefix accesslog
      output_per_tag yes
      tag_prefix datacount
      output_messages yes
    </match>
    # => tag: 'datacount.baz'
    #    message: {'messages' => xxx, 'OK_count' => ...}

## Parameters

* count\_key (required)

    The key to count in the event record.

* tag

    The output tag. Default is `datacount`.

* tag\_prefix

    The prefix string which will be added to the input tag. `output_per_tag yes` must be specified together. 

* input\_tag\_remove\_prefix

    The prefix string which will be removed from the input tag.

* count\_interval

    The interval time to count in seconds. Default is `60`.

* unit

    The interval time to monitor specified an unit (either of `minute`, `hour`, or `day`).
    Use either of `count_interval` or `unit`.

* aggregate

    Calculate in each input `tag` separetely, or `all` records in a mass. Default is `tag`.

* ouput\_per\_tag

    Emit for each input tag. `tag_prefix` must be specified together. Default is `no`.

* outcast\_unmatched

    Specify `yes` if you do not want to include 'unmatched' counts into percentage. Default is `no`.

* output\_messages

    Specify `yes` if you want to get tested messages. Default is `no`.

* store\_file

    Store internal data into a file of the given path on shutdown, and load on starting.

## TODO

- consider what to do next
- patches welcome!

## Copyright

Copyright:: Copyright (c) 2012- TAGOMORI Satoshi (tagomoris)
License::   Apache License, Version 2.0
