# NAME

Plack::Middleware::Negotiate - Apply HTTP content negotiation as Plack middleware

[![Build Status](https://travis-ci.org/nichtich/Plack-Middleware-Negotiate.png)](https://travis-ci.org/nichtich/Plack-Middleware-Negotiate)
[![Coverage Status](https://coveralls.io/repos/nichtich/Plack-Middleware-Negotiate/badge.png)](https://coveralls.io/r/nichtich/Plack-Middleware-Negotiate)
[![Kwalitee Score](http://cpants.cpanauthors.org/dist/Plack-Middleware-Negotiate.png)](http://cpants.cpanauthors.org/dist/Plack-Middleware-Negotiate)

# SYNOPSIS

    builder {
        enable 'Negotiate',
            formats => {
                xml  => { 
                    type    => 'application/xml',
                    charset => 'utf-8',
                },
                html => { type => 'text/html', language => 'en' },
                _    => { size => 0 }  # default values for all formats           
            },
            parameter => 'format', # e.g. http://example.org/foo?format=xml
            extension => 'strip';  # e.g. http://example.org/foo.xml
        $app;
    };

# DESCRIPTION

[Plack::Middleware::Negotiate](https://metacpan.org/pod/Plack::Middleware::Negotiate) applies HTTP content negotiation to a [PSGI](https://metacpan.org/pod/PSGI)
request. In addition to normal content negotiation from a list of defined
`formats` one may enable explicit format selection with a path `extension` or
query `parameter`. In summary, the following methods are tried in this order 
to negotiate a known format:

- HTTP GET/POST format parameter (if enabled with option `parameter`)

    e.g. format is set to xml for http://example.org/foo?format=xml if format xml has been defined.

- URL path extension (if enabled with option `extension`)

    e.g. format is set to xml for http://example.org/foo.xml if format xml has been defined.

- HTTP Accept Header (unless disabled with option `explicit`)

    e.g. format is set to xml if format xml has been defined with content type application/xml for Accept header value "application/xml".

The PSGI environment key `negotiate.format` is set to the chosen format name
after negotiation.  The PSGI response is enriched with corresponding HTTP
headers Content-Type and Content-Language unless these headers already exist.

If used as pure application, this middleware returns a HTTP status code 406 if
no format could be negotiated.

# METHODS

## new( formats => { ... } \[, %options \] )

Creates a new negotiation middleware with a given set of formats.

## negotiate( $env )

Chooses a format based on a PSGI request. The request is first checked for
explicit format selection via `parameter` and `extension` (if configured) and
then passed to [HTTP::Negotiate](https://metacpan.org/pod/HTTP::Negotiate). On success the method returns a format name.
The method may modify the PSGI request environment keys PATH\_INFO and
SCRIPT\_NAME if format was selected by extension set to `strip`, and strips the
`format` HTTP GET query parameter from QUERY\_STRING if `parameter` is set to
a format. If format was selected by HTTP POST body parameter, the parameter it
is not stripped from the request.

## known( $format )

Tells whether a format name is known. By default this is the case if the format
name exists in the list of formats.

## about( $format )

If the format was specified, this method returns a hash with `quality`,
`type`, `encoding`, `charset`, and `language`. Missing values are set to
the default.

## variants

Returns a list of content variants to be used in [HTTP::Negotiate](https://metacpan.org/pod/HTTP::Negotiate). The return
value is an array reference of array references, each with seven elements:
format name, source quality, type, encoding, charset, language, and size. The
size is always zero.

## add\_headers( \\@headers, $format )

Add apropriate HTTP response headers for a format unless the headers are
already given.

# CONFIGURATION

- formats

    A list of formats to choose among.  Each format can be defined with `type`,
    `quality` (defaults to 1), `encoding`, `charset`, and `language`. The
    special format name `_` (underscore) is reserved to define default values for
    all formats.

    Formats can also be used to directly route the request to a PSGI application:

        my $app = Plack::Middleware::Negotiate->new(
            formats => {
                json => { 
                    type => 'application/json',
                    app  => $json_app,
                },
                html => {
                    type => 'text/html',
                    app  => $html_app,
                }
            }
        );

- parameter

    Enables explicit format selection with a query paramater, for instance
    `format`. Both HTTP GET and HTTP POST body parameters are supported.

- extension

    Enables explicit format selection with a virtual file extension. The value
    `strip` strips a known format name from the request path. The value `keep`
    keeps the format name extension after format selection.

    The middleware takes care for rewriting and restoring PATH\_INFO if it is
    configured to detect and strip a format extension. 

- explicit

    Disables content negotiation based on HTTP headers.

# LOGGING AND DEBUGGING

Plack::Middleware::Negotiate uses `Log::Contextual` to emit a logging message
during content negotiation on logging level `trace`. Just set:

    $ENV{PLACK_MIDDLEWARE_NEGOTIATE_TRACE} = 1;

# LIMITATIONS

The Content-Encoding HTTP response header is not automatically set on a
response and content negotiation based on size is not supported. Feel free to
comment on whether and how this middleware should support both.

# CONTRIBUTORS

Jakob Voß, Christopher A. Kirke

# COPYRIGHT AND LICENSE

Copyright 2014- Jakob Voß

This library is free software; you can redistribute it and/or modify it under
the same terms as Perl itself.

# SEE ALSO

Content negotiation in this module is based on [HTTP::Negotiate](https://metacpan.org/pod/HTTP::Negotiate). See 
[HTTP::Headers::ActionPack::ContentNegotiation](https://metacpan.org/pod/HTTP::Headers::ActionPack::ContentNegotiation) for an alternative approach.
This module has some overlap with [Plack::Middleware::SetAccept](https://metacpan.org/pod/Plack::Middleware::SetAccept).
