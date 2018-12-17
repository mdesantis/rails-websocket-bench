# ActionCable vs AnyCable: fight!

## Introduction

I'm a long time Ruby on Rails developer. When [ActionCable](https://guides.rubyonrails.org/action_cable_overview.html) was released, the first things that came to mind was: "[will it perf?](https://www.youtube.com/watch?v=lAl28d6tbko)", given that Ruby has never been recognized for its ability with concurrency management, and that Rails has never been recognized for its lightness.

Browsing around, I stumbled upon [AnyCable](https://github.com/anycable/anycable), an alternative to ActionCable created for some reason. What if that reason is performance? Let's find out!

### How ActionCable works

ActionCable runs in an application server instance separate from the one that handles the classic HTTP requests. It uses the `hijack` API, a Rack API added specifically for sessions requiring evented IO, like Server-Sent Events and WebSockets, giving to the application the whole control of the socket. Within ActionCable, the WebSocket management is fully implemented in Ruby.

### How AnyCable works

Like for ActionCable, AnyCable receives and sends messages using Ruby, but the socket management is forwarded via gRPC to a server compatible to AnyCable specifications, which can be written in any language that communicates via gRPC. Currently, the official implementations are [AnyCable-Go](https://github.com/anycable/anycable-go), written in Go, and [ErlyCable](https://github.com/anycable/erlycable), written in Erlang. For details about AnyCable architecture I recommend reading the related posts listed in the AnyCable README.

## Setup

*If you're trying to run the test by yourself you should read the [Caveats](#caveats) section first*

We are going to load test ActionCable and AnyCable with [Tsung](https://www.process-one.net/en/tsung/) (credits to [this awesome post](https://www.thegreatcodeadventure.com/load-testing-rails-5-action-cable-with-tsung/) which pointed me on the right way). Below the testing environment details:

 - CPU: Intel i7-2600K

 - OS: Ubuntu 16.04.5

 - Ruby: 2.5.3

 - Rails: 5.2.1

 - AnyCable: 0.5.2

 - AnyCable-Go: 0.5.4

 - Load testing: Tsung 1.7.0

I wrote two Procfile configurations respectively for [ActionCable](Procfile.actioncable) and [AnyCable](Procfile,anycable) in order to run them using [Foreman](https://github.com/ddollar/foreman):

 - `foreman start -e .env.production,.env.actioncable -f Procfile.actioncable`

 - `foreman start -e .env.production,.env.anycable -f Procfile.anycable`

 Once the server is up and running, we can start Tsung:

 - `tsung -f tsung.xml -k start`

 Tsung will write its results in the `~/.tsung`, and serve them live at http://localhost:8091/.

### Caveats

Performing the test properly is a bit tricky; there are some workarounds and tweaks we have to take:

Tsung shows an error like this one: `Fail to generated reports: tsung_stats.pl was not found in the $PATH`
: Add Tsung binaries to `$PATH`: `export PATH=/usr/lib/x86_64-linux-gnu/tsung/bin:$PATH`

Overcome OS caps
: As suggested [here](https://github.com/hashrocket/websocket-shootout#open-file-limits), we need to increase OS limits related to open files and port ranges:
: - Add the following to `/etc/sysctl.conf`:

        net.ipv4.tcp_tw_reuse = 1
        net.ipv4.tcp_tw_recycle = 1
        net.ipv4.ip_local_port_range = 1024 65000
        fs.file-max = 65000
: - Add the following to `/etc/security/limits.conf`:

        *    soft nofile 1048576
        *    hard nofile 1048576

Reboot or reload the configuration and everything should be fine.

## Benchmarks

## Conclusions

ActionCable is limited regarding the connections threshold that it can handle, which isn't bad and it can be enough for use cases with a restricted amount of users. If instead your application requires to support a huge amount of WebSocket connections AnyCable is a perfectly viable option.
