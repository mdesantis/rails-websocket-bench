# ActionCable vs AnyCable: fight!

## Introduction

I'm a long time Ruby developer. When [ActionCable](https://guides.rubyonrails.org/action_cable_overview.html) was released, the first things that came to mind was: ["will it perf?"](https://www.youtube.com/watch?v=lAl28d6tbko), given that Ruby has never been recognized for its ability with concurrency management, and that Rails has never been recognized for its lightness.

Browsing around, I stumbled upon [AnyCable](https://github.com/anycable/anycable), an alternative to ActionCable created for some reason. What if that reason is performance? Let's find out!

### How ActionCable works

ActionCable runs in a application server instance unconnected separate from the one that handles the standard HTTP requests. It uses the `hijack` API, a Rack API added specifically for sessions requiring evented IO, like Server-Sent Events and WebSockets, giving to the application the whole control of the socket. Within ActionCable, the WebSocket management is fully implemented in Ruby.

### How AnyCable works

Like for ActionCable, AnyCable receives and sends messages with a Ruby application, but the socket management is forwarded via gRPC to a server compatible to AnyCable specifications, which can be written in any language that communicates via gRPC. Currently, the official implementations are [AnyCable-Go](https://github.com/anycable/anycable-go), written in Go, and [ErlyCable](https://github.com/anycable/erlycable), written in Erlang. For details about AnyCable architecture I recommend reading the AnyCable ReadMe and the related posts linked there.

## Setup

*If you're trying to run the test by yourself read [Caveats](#caveats) first*

We are going to load test ActionCable and AnyCable with [Tsung](https://www.process-one.net/en/tsung/) (credits to this [great post](https://www.thegreatcodeadventure.com/load-testing-rails-5-action-cable-with-tsung/) which pointed me on the right way). Here are the hardware specs:

 - CPU: Intel i7-2600K

 - OS: Ubuntu 16.04.5

 - Ruby: 2.5.3

 - Rails: 5.2.1

 - AnyCable: 0.5.2

 - AnyCable-Go: 0.5.4

 - Load testing: Tsung 1.7.0

I made two Procfile configurations respectively for [ActionCable]() and [AnyCable]() in order to run them using Foreman:

 - `foreman start -e .env.production,.env.actioncable -f Procfile.actioncable`

 - `foreman start -e .env.production,.env.anycable -f Procfile.anycable`

 Tsung invece lo facciamo partire così:

 - `tsung -f tsung.xml -k start`

 Tsung will report its test at http://localhost:8091/.

### Caveats

Performing il test properly is a bit tricky; there are some workarounds and tweaks we have to take:

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

Reboot (or reload the configuration files changed, but I don't know how to do :) ) and everything should be fine

## Benchmarks

## Conclusion

ActionCable è quindi limitato riguardo il numero di utenti che può sopportare, che comunque non è basso, e può andare bene per i siti che aprono connessioni websocket solo per un numero limitato di utenti, come per esempio solo per la sezione admin. Se però la vostra applicazione tira su molte connessioni websocket AnyCable is a perfectly viable option.









# ActionCable vs AnyCable: fight!

TL;DR: ActionCable supports up to ~30000 simultaneous connections, while AnyCable goes up to 60000 and more

Il rilascio della versione 5 di Ruby on Rails ha portato con sé l'integrazione con i WebSockets sottoforma di ActionCable. Ciò ha suscitato in me (ed immagino in molti rubisti) curiosità su quanto Ruby possa scalare in un ambito che notoriamente non è il suo forte, quello della concorrenza.

We are going to test ActionCable and AnyCable load with [Tsung](https://www.process-one.net/en/tsung/) (credits to this [great post](https://www.thegreatcodeadventure.com/load-testing-rails-5-action-cable-with-tsung/) which pointed me on the right way)

## Setup

Ubuntu 16.04 comes with an outdated version of Tsung, 1.5.1, while the latest is 1.7.0. Looking for a more recent version on [Ubuntu Packages search](https://packages.ubuntu.com/search?keywords=tsung&searchon=names&exact=1&suite=all&section=all) showed an up-to-date package, which I downloaded and installed successfully. We want Tsung to be as up-to-date as possible, since websockets support has been added recently, so latest versions should be more stable.

After that, we need a Rails app configured with ActionCable and AnyCable and configured to work against Tsung; I published it [here](app_url). It's a default Rails 5.2.1 app with some customizations:

 - `config.action_cable.allowed_request_origins = [/.*/]`: altrimenti le richieste di Tsung vengono bloccate
 - due Procfiles diversi per avviare l'applicazione usando ActionCable o AnyCable tramite Foreman

## Caveats

## Benchmarks

## Testing environment

CPU: Intel i7-2600K (4 cores, 8 threads)
OS: Ubuntu 16.04.5
Ruby: 2.5.3
Rails: 5.2.1
AnyCable: 0.5.2
AnyCable-Go: 0.5.4
Load testing: Tsung 1.7.0
