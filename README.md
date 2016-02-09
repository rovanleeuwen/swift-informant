# Middleware to time OpenStack requests

OpenStack Middleware to send events to a [statsd](http://github.com/etsy/statsd/ "statsd") instance.
Borrowed most code from: https://github.com/pandemicsyn/swift-informant

**After** the request has been serviced (using a event posthook) it will fire a statsd counter incrementing the request method's status code.  It breaks these up by whether the request was an operation on an Account, Container, or an Object. In addition to the counter, two timer event's are fired for the request duration as well as time until start_response was seen. If present a counter for bytes transferred is included as well.

Counter Sample:

    obj.GET.200:1|c

Timer Sample (duration, and start_response_time:

    acct.GET.200:140|ms
    srt.acct.GET.200:11|ms

Bytes Transferred Sample:

    tfer.obj.PUT.201 2423.9

To enable, load informant as the first pipeline entry (even before catcherrors):

    pipeline = informant catch_errors healthcheck cache ratelimit proxy-server

And add the following filter config:

    [filter:informant]
    use = egg:informant#informant
    # statsd_host = 127.0.0.1
    # statsd_port = 8125
    # standard statsd sample rate 0.0 <= 1
    # statsd_sample_rate = 0.5
    # list of allowed methods, all others will generate a "BAD_METHOD" event
    # valid_http_methods = GET,HEAD,POST,PUT,DELETE,COPY
    # They key used to combine events for reporting to statsd, alternate
    # versions used a # to seperate events. The offical way is by newline
    # combine_key = \n
    # prepends name to metric collection output for easier recognition, e.g. company.swift.
    # metric_name_prepend =
    # A list of accounts for who we'llaccount prefix the metric name with their account name
    # This lets you track metrics for specific accounts independently.
    # prefix_accounts = AUTH_something,

The commented out values are the defaults. This module does not require any additional statsd client modules.

# Building packages

Clone the version you want and build the package with [stdeb](https://github.com/astraw/stdeb "stdeb"):

    git clone git@github.com:pandemicsyn/swift-informant.git informant-0.0.8
    cd informant-0.0.8
    git checkout 0.0.8
    python setup.py --command-packages=stdeb.command bdist_deb
    dpkg -i deb_dist/python-informant_0.0.8-1_all.deb
