--------------------------------------------------------------------------------

# Should Registries offer DoH?

   * Author: Bjarni R. Einarsson, <bre@isnic.is>
   * Version: 0.2
   * Date: 2019-11-07

## 0.1. Abstract

This document gives an overview over DNS over HTTPS (DoH), both the technical
and political issues surrounding the technology, with an analysis designed to
inform decisions on whether DNS Registries in general should invest in offering
DoH-based recursive resolver service to the public.

## 0.2. Disclaimer

This document is the work of a single person who has opinions.
Oversimpifications and rampant speculation abound. Reader beware!

## 0.3. Versions

|    date    | version | comments       | Author
|:----------:|:-------:|:-------------- |:-------------------
| 2019-11-07 |   0.2   | Draft          | Bjarni R. Einarsson
| 2019-11-05 |   0.1   | Draft          | Bjarni R. Einarsson

## 0.4. META: Tooling

    # Prerequisites
    sudo apt install plantuml discount

    # Render HTML
    markdown -STf+fencedcode <isnic_doh_report.md >isnic_doh_report.html     #!~

    # Generate SVG
    plantuml -tsvg isnic_doh_report.md

    # Or...
    grep '#!~' isnic_doh_report.md |grep -v grep |bash -v

    # Look at the output
    ls -lrth isnic_doh_report*                                               #!~
    opener ./isnic_doh_report.html


--------------------------------------------------------------------------------

# 1. Technological background

## 1.1. What is DNS over HTTPS (DoH)?

DNS over HTTPS is a protocol which allows an application (such as a web
browser) to fetch information from the global Internet Domain Name system,
using the same technology as is used to secure e-commerce on the World Wide
Web: HTTP/2 and TLS.

DNS over HTTPS is specified in [RFC8484][1]. The documenet defines:

> [...] a specific protocol, DNS over HTTPS (DoH), for sending DNS
> ([RFC1035][2]) queries and getting DNS responses over HTTP
> ([RFC7540][3]) using https ([RFC2818][4]) URIs (and therefore TLS
> ([RFC8446][5]) security for integrity and confidentiality).  Each
> DNS query-response pair is mapped into an HTTP exchange.
>
> [...]
>
> Two primary use cases were considered during this protocol's
> development.  These use cases are preventing on-path devices from
> interfering with DNS operations, and also allowing web applications
> to access DNS information via existing browser APIs in a safe way
> consistent with Cross Origin Resource Sharing ([CORS][6]).  No
> special effort has been taken to enable or prevent application to
> other use cases.

Of particular note, is that the specification mandates use of both HTTP/2 and
TLS encryption.

TLS provides both confidentiality (protection against eavesdropping) and
integrity (protection against modification) of communication, both of which are
lacking from traditional DNS.

HTTP/2 provides a common vocabulary and toolset for developers and
administrators, as well as protocol optimizations which counterbalance somewhat
the performance penalty caused by switching from the stateless UDP protocol DNS
requests have traditionally used, to the stateful connections of TCP.

It should be noted that in the context of DNS, DNS over HTTPS only provides
security guarantees on a **hop-by-hop** basis (from Client to Resolver,
Resolver to Resolver, or from Resolver to Authoritative Server), not
end-to-end. A malicious DoH resolver can still monitor and modify DNS traffic.

## 1.2. Alternatives: DoT and DNSCrypt

Multiple alternatives to DoH have been proposed, but only some have been
deployed. [DNS over TLS (DoT)][10] and [DNSCrypt][11] are the most mature.

DNS over TLS differs from DNS over HTTPS, mostly in that it omits the
HTTP/2 protocol, sending DNS data directly over the TLS circuit itself.
This simplification means DoT misses the opportunity to build upon the
tooling and experience provided by the web community, while inheriting most
of the same technical challenges (certificate management, implementation
complexity, and performance penalties) from TCP and TLS.

Before the browser vendors threw their weight behind DoH, DNSCrypt was the
most widely deployed scheme for securing DNS. DNSCrypt does not build upon
TLS and TCP, instead it uses a custom cryptographic scheme to protect the
integrity and confidentiality of the UDP payload. This is an attractive
approach; simplicity is usually a good thing when it comes to cryptographic
security, and UDP is faster than TCP. However, DNSCrypt may as a result
continue to suffer from the same denial-of-service and packet fragmenting
concerns which standard DNS has inherited from the UDP protocol.

## 1.3. Alternatives: DNSSEC

[DNSSEC][12] is a previous attempt to standardize cryptographic protections for
DNS requests. DNSSEC is an end-to-end protocol which guarantees the integrity
and authenticity of DNS responses, but does not attempt to provide
confidentiality or protect the queries themselves in any way.

DNS over HTTPS addresses all of these concerns (integrity and confidentiality
for both queries and responses), but only on a hop-by-hop basis - not
end-to-end.

DNSSEC and DoH can be considered complementary, each compensates for some of
the weaknesses of the other if they are deployed simultaneously.

## 1.4. Anonymity and User Tracking

As stated above, DNSSEC provides protections which guarantee integrity and
authenticity of DNS data. DoH, DoT and DNSCrypt all augment this by protecting
query data as well, and providing confidentiality. There is a third security
concern, addressed directly by none of these, which is client anonymity.

Client anonymity is most frequently discussed in the context of preventing user
tracking for targeting of commercial advertisements, but it has importance in
other contexts as well.

Traditionally in the world of DNS, clients have had no meaningful anonymity in
their exchanges with their local resolver, which was usually operated by their
ISP. However, the opportunities for anyone else to track users and monitor
their DNS requests have been quite limited.

The DNS protocol itself is not designed to facilitate user tracking (although
[DNS cookies][13] and [EDNS Client Subnet signaling][18] have undermined this)
and DNS users often share a common resolver with a larger group. This means the
operator of the resolver, and the operators of any intervening networks, could
monitor and track users' DNS traffic. The content providers themselves (and
advertisers) usually could not.

The political side of this will be analyzed in more detail later in this
document, but it is safe to say that the overall structure of the network (who
provides services, who has access to user data) is changing with the shift to
DNS over HTTPS. The technical capabilities are also changing, since DoH is a
much more "trackable" and less anonymous protocol than normal DNS.

## 1.5. Is DoH winning?

DNS over HTTPS is "winning" compared to the other DNS security protocols, in the
sense that it is being deployed at scale to end users as built-in functionality
of two of the most popular main-stream web browsers, Chrome and Firefox.

At the time of writing, Firefox have enabled the feature by default for a
subset of their North American user-base and the Chrome team is running a live
experiment for their users as well. The same can not be said of any of the
other proposals to improve the security of DNS.

Whether other applications (or entire operating systems) follow the browsers'
down this path is yet to be seen, but seems likely.

Market dynamics aside, DNS over HTTP also has compelling technical advantages.

[According to one publication][7], DNS over HTTP was the only secured DNS
resolver protocol with performance that compared favourably with traditional
(unencrypted) DNS queries. Although other implementations (DoT) might be able
to catch up over time, this speaks to the immediate benefit of building on
existing web technology. With work already well underway to define an even
faster HTTP/3, DoH seems likely to maintain this head start.

The browsers (and the business which fund them) tend to focus a great deal on
performance, and the performance of DNS lookups is a significant factor in the
overall performance of the web. The fact that the browsers already have a "sunk
cost" in developing and maintaining good support for HTTP/2 and TLS is surely
also a factor; from the browser's point of view supporting DoH has a relatively
small footprint in terms of new code.

The latter argument also translates directly from the world of browsers to the
world of DNS providers: building upon DoH allows providers to leverage the
immense and ongoing work to improve the security and performance of the World
Wide Web. In particular, TCP-based protocols (DoH included) are much easier to
secure against Denial of Service Attacks than UDP based protocols such as
traditional (DNS or DNSCrypto).

Finally, HTTP/2 is also a very feature-rich protocol which has the potential to
enable some interesting new behaviours in the context of DNS. Time will tell
which of these lead to useful applications, but here are some potential
examples:

   * HTTP/2 allows servers to "push" related content to the client, which may
     have meaningful performance implications within the context of DNS.
   * HTTP requests build upon URLs, which affords a rich vocabulary for
     requesting custom behaviours on a client-by-client or app-by-app basis.
   * URLs (and Cookies) make fine-grained user tracking much easier to do.

--------------------------------------------------------------------------------

# 2. Politics

DNS over HTTPS is controversial.

[RFC7258][14] states:

> The IETF community has expressed strong agreement that PM [ed. Pervasive
> Monitoring] is an attack that needs to be mitigated where possible, via the
> design of protocols that make PM significantly more expensive or infeasible"

DNS over HTTPS is such a protocol. So why is it so controversial, even within
the tech community?

Much of the controversy revolves around the fact that DNS over HTTPS is
perceived to enable certain forms of monitoring, even as it thwarts others.

DoH is being deployed by Mozilla in such a way that it deprives ISPs (and local
governments) of the opportunity to track, monitor and modify DNS traffic.
Instead, that capability is shifted to a relatively smaller number of dedicated
DoH providers (and their local governments); providers who are already major
providers of other, unrelated services such as hosting, advertisements or
content distribution.

[The Chrome team's approach is currently more nuanced][16]. However, it seems
likely that Google will eventually follow Mozilla's lead, for reasons discussed
in sections [3.3.3](#L3.3.3.-Facilitating-Content-Delivery) and 
[3.3.4](#L3.3.4.-User-Tracking-and-Metrics).

The good behaviour of these new DoH providers will be guaranteed not through
technical means, but through contractual arrangements such as the [Mozilla DoH
Resolver Policy][15] and social pressure.

Some see this as unconvincing and insufficient, especially in light of the fact
that these new providers are already perceived to have too many opportunities
and too much vested interest in tracking the general public. Others accept that
it is not a perfect solution, but believe it is a necessary step in the right
direction.

## 2.1. Points of View

Those who have a vested interest in the current status quo, oppose DoH. This
includes traditional ISPs who may have developed business models around user
tracking or advertising, governments which have deployed surveillance or
censorship tools based on existing DNS systems, and systems administrators who
are simply accustomed to being "in control" over this aspect of their networks.

Those who fear increased centralization will lead to monopoly abuse, whether
through degradation of privacy or other manipulation of DNS, also oppose the
move to DoH. In particular, a growing, vocal segment of the Internet community
is deeply concerned with how powerful Google and CloudFlare have become and
oppose this change on the grounds that it will embed them even further into the
fabric of the Internet itself. 

On the other hand, those who are unhappy with the track record of local ISPs
when it comes to respecting their users' privacy and the integrity of DNS, tend
to welcome the change.

Those who are concerned about government surveillance and overreach, also tend
to welcome DoH, as the large content providers are perceived to be somewhat
less cooperative with the worlds' many governments than local ISPs have been.
The fact that DoH makes it easier for DNS traffic to cross legal jurisdictions
is seen by some as a benefit, in this context.

--------------------------------------------------------------------------------

# 3. Business Issues

This section aims to explore some of the factors relating to the business
questions around DoH.

## 3.1. Who can Provide DoH Service?

Theoretically, anyone with the ability to run a traditional DNS resolver could
easily upgrade and provide DoH as well. All the necessary software is freely
available as Open Source.

In practice, this is insufficent as there is no standardized, automated way for
anyone outside managed enterprise settings, to provision the service to their
users. The [Chrome team is attempting to automatically detect and enable DoH in
specific cases][16], but that mechanism is both experimental and only applies
to providers already running DNS resolvers for a group of users.

Any new organization interested in providing DoH services to the general public
currently needs to either get added to Mozilla's official lists of default
providers, or undertake sufficient marketing so as to convince end-users to
reconfigure their browsers manually to make use of the service.

(For ISPs in particular, it is relevant that there is no DoH field in DHCP, home
routers do not support the protocol and even if they did, browsers and
individual apps could bypass them anyway. This may be a temporary situation,
addressed by future protocol upgrades, but for now there is no denying that the
role of traditional ISPs within the global DNS system stands to be reduced as
the browsers roll out DoH.)

## 3.2. Scale and Cost

DoH is more expensive to run than a traditional DNS server.

This derives from the fact that DoH is a stateful, connection-oriented
protocol. Clients create long-lived connections, each of which consumes some
RAM on both the client and the server for the duration of the connection, even
if it is not in active use (it will expire after a while). This is much less
true for traditional DNS, due to its reliance on UDP, where each request stands
alone and once the server has responded it can forget all about it and put any
available resources to other use (traditional DNS traffic may also use TCP, but
it is considered a secondary fallback mechanism and is less used). DoH is also
more bandwidth intensive and requires more computation than plain DNS.

How great the cost difference is, is still suprisingly difficult to estimate.
Traditional DNS servers, in practice, are required to over-provision in order
to cope with denial of service attacks, which are less of a concern for DoH.
And as DNSSEC has become more common, increased response size has increased the
use of TCP for DNS, further reducing the performance advantage.

It still seems safe to assume that in order to handle a comparable amount of
request traffic, a DoH provider will need to invest in more infrastructure than
a traditional DNS resolver of similar size. The requirements for scalable web
services are relatively well understood by the industry as a whole, but do
involve a different "stack" of technologies from those commonly relied upon by
providers specializing in DNS. So additional training may also be neccessary.

FIXME: Numbers!

   * NGINX sizing guide: <https://www.nginx.com/wp-content/uploads/2018/10/Sizing-Guide-for-Deploying-NGINX-on-Bare-Metal-Servers-2018-07-09.pdf>
   * Find numbers about DNS query frequencies?

## 3.3. Why Provide DoH?

### 3.3.1. Public Interest

Those who share the position stated in [RFC7258][14], that pervasive
surveillance is an attack on the Internet-using public, will consider offering
secure and encrypted services to be a social good and in the public interest.

The concerns about DoH increasing centralization ([section 2](#L2.-Politics))
may also be mitigated somewhat by increasing the diversity and availability of
DoH providers.

Organizations whose mandate includes operating public interest services for
their communities, can therefore easily make a case for operating DNS over
HTTPS resolvers.

### 3.3.2. Subscription Custom Resolvers

There are numerous problems in enterprise and public network management, which
can be addressed by a custom resolver service. These include:

   * Monitoring and management of Internet use, activity
   * Content filtering (ad blocking, "family filters", malware protection)
   
Businesses offering such services have existed for a long time, and many of
them already offer DoH as an option.

DoH provides new capabilities to this market. Due to the features DoH inherits
from HTTP/2 it is now possible to offer much higher level of service than
before: with DoH, every single user of a Custom Resolver could be using a
different service URL, which in turn makes it possible for the provider to
customize settings and monitor usage (e.g. for billing purposes) on an
individual user or even per-app basis.

As an example, a DoH-enabled Custom Resolver could allow a savvy customer to
use one settings profile when they browse using Firefox, and a different set of
DNS filters for use with Chrome. Or alternately, a savvy customer could use the
same custom settings on multiple devices on disparate networks. This would be
infeasible using legacy DNS (or DoT, or DNSCrypt).

### 3.3.3. Facilitating Content Delivery

The largest public DoH providers, Google and CloudFlare, are both also major
providers of web-based content and services, and both have a very strong focus
on performance and speed.

DNS lookup times are one of the most significant factors when it comes to
performance on the web, and it is a factor that has traditionally been out of
the control of the hosting provider. With DoH, and by working with the browser
vendors, these providers are given the opportunity to optimize this part of the
web experience as well, and they seem quite eager to do so.

Since many hosting providers heavily rely on dynamic DNS ([often with very low
time-to-live values][17]) to route traffic, they are likely to see offering DoH
as an opportunity to eliminate a 3rd party (other resolvers), allowing them to
better manage and optimize their offerings.

Finally, these providers are also frequently the target of ad-blockers or other
network imposed content filters. Moving as many users as possible to DNS
resolvers which do not block their content is very valuable to them.

(Note: although Google does currently provide a public DoH service, the Chrome
team has not yet announced a plan to make use of it by default and have
publicly [refuted claims that this is their intention][16]. So even if the
above analysis applies, the company has *so far* given in to political pressure
and forgone these opportunities.)

### 3.3.4. User Tracking and Metrics

Public resolvers (DoH or otherwise) give their operators potential access to
information about which domain names are being looked up, and roughly by whom.
This information has value when it comes to optimizing network operations and
detecting malware, and to a lesser degree for user profiling and ad targeting.

DoH is potentially much more trackable than traditional DNS in this regard.
Businesses which already derive value from such data harvesting, whether for
operational reasons or for marketing, are very likely to find DoH could provide
them with new opportunities to track peoples' activities online.

For businesses already competing in this space, DoH also strikes a blow against
the ability of others to collect information about their users' activities,
making them less competetive as a result. So even if a DoH provider does not
directly monetize insights derived from monitoring DNS, they may still have a
valid business reason to thwart their competitors' efforts to do exactly that.

--------------------------------------------------------------------------------

# 4. Relevance to DNS Registries

## 4.1. Discussion

In general, DNS Domain registries often already have built up teams and skills
pertaining to DNS and they also have an interest in guaranteeing the DNS system
functions well and efficiently. So it is natural that many are considering
whether they should take an active part in providing DoH infrastructure.

Given the context of [section 3.3](#L3.3.-Why-Provide-DoH-3f-), the following
assesment can be made:

DoH in the [Public Interest](#L3.3.1.-Public-Interest) is a relevant motivation
for many Domain Registries.

Using DoH to provide [Subscription Custom
Resolvers](#L3.3.2.-Subscription-Custom-Resolvers) is a compelling option for
businesses already offering such services, but for most registries this would
be a new product, with no guarantee of market success.

Registries may well see a benefit to [Facilitating Content
Delivery](#L3.3.3.-Facilitating-Content-Delivery), especially if their domains
are associated with a well defined user base, as is often the case for ccTLD
(country-code TLD) registries.  Taking steps to ensure that a given ccTLD is
fast and reliable (compared with other top level domains) in its region is
perfectly reasonable in both competitive and technological contexts. Given the
public statements made by the Chrome and Firefox teams, it seems likely they
would welcome such offerings and encourage their use. It also seems plausible
that a ccTLD operator could negotiate partnerships with local ISPs, defusing
some of the political concerns and participating in Chrome's current deployment
strategy.

Using DoH for [User Tracking and Metrics](#L3.3.4.-User-Tracking-and-Metrics)
may have value to Domain Registries, but registries operating in accordance
with [RFC7258][14] would probably find it difficult to justify monetizing user
tracking directly. That leaves aggregate statistics, which although valuable,
are probably not a business opportunity in themselves unless the Registry
already offers related products which would benefit from this new data source.

## 4.2. Registry-specific Risks

Launching a DoH service is a non-trivial effort and not without risk. Meeting
both the technical requirements and the operational requirements of the major
browser vendors, is a prerequisite for the service getting adopted, and the
effort required will be expensive whether the service is a success or not.

Registries are themselves often under special scrutiny due to their
monopoly-like position as sole stewards of a branch of the DNS naming tree.  It
might well be considered undesirable to instigate further centralization of DNS
traffic, centralization which would inevitably result from the same entity
running both the authoritative top level name servers and the main DNS
resolvers for a specific region.

Many DNS Registries have traditionally resisted efforts by local governments to
censor the DNS system, instead directing the authorities' attention to content
hosting providers or operators of DNS resolvers. In various countries, public
DNS resolvers have received court orders mandating they block access to
specific domains. If the registries take on this role as well, they should
expect to also more often be targetted with court orders and takedown requests.
There may be a risk that the Registry's authoritative name servers could become
"collateral damage" of overly broad court orders.


## 4.3. Conclusions

DoH is here to stay.

Offering DNS over HTTP service to the general public may be a reasonable
activity for DNS Registries to participate in, both as a Public Interest effort
to bolster the health of the wider DNS ecosystem, and as a way to compete and
improve the performance of the domains they manage.

The latter applies especially in the case of ccTLDs who primarily serve a
specific geographic region and have a well defined user base.

The risks associated with increased DNS traffic centralization, and increased
legal scrutiny may outweigh these benefits.


--------------------------------------------------------------------------------

[1]: https://tools.ietf.org/html/rfc8484
[2]: https://tools.ietf.org/html/rfc1035
[3]: https://tools.ietf.org/html/rfc7540
[4]: https://tools.ietf.org/html/rfc2818
[5]: https://tools.ietf.org/html/rfc8446
[6]: https://tools.ietf.org/html/rfc8484#ref-FETCH
[7]: http://eecs.qmul.ac.uk/~boettget/assets/doh-imc19.pdf
[8]: https://tools.ietf.org/html/rfc7858
[9]: https://www.dnscrypt.org/
[10]: https://tools.ietf.org/html/rfc7858
[11]: https://dnscrypt.info/
[12]: https://tools.ietf.org/html/rfc4033
[13]: http://dnscookie.com/ 
[14]: https://tools.ietf.org/html/rfc7258
[15]: https://wiki.mozilla.org/Security/DOH-resolver-policy
[16]: https://blog.chromium.org/2019/10/addressing-some-misconceptions-about.html
[17]: https://00f.net/2019/11/03/stop-using-low-dns-ttls/
[18]: https://tools.ietf.org/html/rfc7871

<style type='text/css'>
  body { max-width: 45em; margin: 1em auto;
         font-family: sans-serif; font-size: 16px; }
  body p { line-height: 19px; }
  body h1:not(:first-child) { page-break-before: always; }
  table { border-collapse: collapse; font-size: 1em; width: 100%; }
  tr { padding: 0; margin: 0; }
  tr:nth-child(even) {background-color: #f2f2f2;}
  td, th { border: 1px solid #777; padding: 3px; margin: 0; }
  code.uml { display: none; }
  pre { margin: 0 0 2em 1em; }
  hr { margin-top: 3em; color: #777; border-color: #777; }
</style>
