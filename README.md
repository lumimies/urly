# What is Urly

Urly is a tiny Clojure library that unifies parsing of URIs, URLs and URL-like values like relative href values
in real-world HTML.

## Why Urly Was Necessary

java.net.URI and java.net.URL in general do a great job of parsing valid (per RFCs) URIs and URLs. However, when
working with real world HTML markup, it is common to come across href attribute values that are not valid URIs or URLs but
are recognized and accepted by Web browsers. Normalization and resolution of such values cannot use java.net.URI or
java.net.URL because both will throw illegal format exceptions.

Urly tries to make this less painful.

## Supported Clojure versions

Urly is built from the ground up for Clojure 1.3+ and JDK 6+.



## Usage

### Installation

With Leiningen

    [clojurewerkz/urly "1.0.0"]


### clojurewerkz.urly.UrlLike

The central concept in Urly is the UrlLike class. It unifies java.net.URI and java.net.URL as much as practical
and also supports relative href attributes values like "/search?q=Clojure". UrlLike instances are immutable and
perform normalizations that are safe (for example, uses default pathname of "/" and lowercases protocol and hostnames but not pathnames).

UrlLike instances are immutable. To mutate them, use `UrlLike#mutateProtocol`, `UrlLike#mutatePath` and similar methods (see examples
below).

Urly is built around Clojure protocols so most of functions are polymorphic and can take strings as well as instances of

 * clojurewerkz.urly.UrlLike
 * java.net.URI
 * java.net.URL

as their first argument.


### Key Functions

``` clojure
(ns my.app
  (:refer-clojure :exclude [resolve])
  (:use clojurewerkz.urly.core)
  (:import [java.net URI URL]))

;; Instantiate a UrlLike instance
(url-like (URL. "http://clojure.org"))
(url-like (URI. "http://clojure.org"))
(url-like "http://clojure.org")

;; unline java.net.URI, valid Internet domain names like "clojure.org" and "amazon.co.uk"
;; will be recognized as hostname, not paths
(url-like "clojure.org")
(url-like "amazon.co.uk")


;; accessing parts of the URL

(let [u (url-like "http://clojure.org")]
  (protocol-of u)  ;; => "http"
  (.getProtocol u) ;; => "http"
  (.getSchema u)   ;; => "http"
  (host-of u)      ;; => "clojure.org"
  (.getHost u)     ;; => "clojure.org"
  (.getHostname u) ;; => "clojure.org"
  (port-of u)     ;; => -1
  (path-of u)     ;; => "/", path is normalized to be "/" if not specified
  (query-of u)    ;; => nil
  (fragment-of u) ;; => nil
  (tld-of u)      ;; => "org"
  ;; returns all of the above as an immutable Clojure map
  (as-map u))

;; absolute & relative URLs

(absolute? "/faq") ;; => false
(relative? "/faq") ;; => true

(absolute? (java.net.URL. "http://clojure.org")) ;; => true
(relative? (java.net.URL. "http://clojure.org")) ;; => false

;; resolving URIs

(resolve (URI. "http://clojure.org") (URI. "/Protocols"))                   ;; => (URI. "http://clojure.org/Protocols")
(resolve (URI. "http://clojure.org") "/Protocols")                          ;; => (URI. "http://clojure.org/Protocols")
(resolve (URI. "http://clojure.org") (URL. "http://clojure.org/Protocols")) ;; => (URI. "http://clojure.org/Protocols")
(resolve "http://clojure.org"        (URI. "/Protocols"))                   ;; => (URI. "http://clojure.org/Protocols")
(resolve "http://clojure.org"        (URL. "http://clojure.org/Protocols")) ;; => (URI. "http://clojure.org/Protocols")

;; mutating URL parts

(let [u (url-like "http://clojure.org")]
  ;; returns a UrlLike instance that represents "http://clojure.org/Protocols"
  (.mutatePath u "/Protocols")
  ;; returns a UrlLike instance that represents "https://clojure.org/"
  (.mutateProtocol u "https")
  ;; returns a UrlLike instance with query string URL-encoded using UTF-8 as encoding
  (encode-query (url-like "http://clojuredocs.org/search?x=0&y=0&q=%22predicate function%22~10"))
  ;; returns a UrlLike instance that represents "http://clojure.org/"
  (-> u (.mutateQuery "search=protocols")
        (.withoutQueryStringAndFragment))
  ;; the same via Clojure API
  (-> u (mutate-query "search=protocols")
        (.withoutQueryStringAndFragment))
  ;; returns a UrlLike instance with the same parts as u but no query string
  (.withoutQuery u)
  ;; returns a UrlLike instance with the same parts as u but no fragment (#hash)
  (.withoutFragment u)
  ;; returns a UrlLike instance that represents "http://clojuredocs.org/search?x=0&y=0&q=%22predicate+function%22~10"
  (-> u (mutate-query "x=0&y=0&q=%22PREDICATE+FUNCTION%22~10")
        (mutate-query-with (fn [^String s] (.toLowerCase s)))))



;; stripping of extra protocol prefixes (commonly found in URLs on the Web)

(eliminate-extra-protocol-prefixes "http://https://broken-cms.com") ;; => https://broken-cms.com
(eliminate-extra-protocol-prefixes "https://http://broken-cms.com") ;; => http://broken-cms.com
```


## Documentation & Examples

Documentation site for Urly is coming in the future (sorry!). Please see our extensive test suite for more code examples.



## Continuous Integration

[![Continuous Integration status](https://secure.travis-ci.org/michaelklishin/urly.png)](http://travis-ci.org/michaelklishin/urly)

CI is hosted by [travis-ci.org](http://travis-ci.org)


## Urly Is a ClojureWerkz Project

Urly is part of the group of [Clojure libraries](http://clojurewerkz.org) known as ClojureWerkz, together with
[Monger](https://github.com/michaelklishin/monger), [Neocons](https://github.com/michaelklishin/neocons), [Langohr](https://github.com/michaelklishin/langohr), [Elastisch](https://github.com/clojurewerkz/elastisch), [Quartzite](https://github.com/michaelklishin/quartzite), [Welle](https://github.com/michaelklishin/welle) and several others.



## Development

Urly uses [Leiningen 2](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md). Make
sure you have it installed and then run tests against all supported Clojure versions using

    lein with-profile dev javac
    lein all test

Then create a branch and make your changes on it. Once you are done with your changes and all
tests pass, submit a pull request on Github.



## License

Copyright (C) 2011-2012 Michael S. Klishin

Distributed under the Eclipse Public License, the same as Clojure.


[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/michaelklishin/urly/trend.png)](https://bitdeli.com/free "Bitdeli Badge")

