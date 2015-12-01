==============
vmod_saintmode
==============

------------------------
Varnish Saintmode Module
------------------------

:Author: Dag Haavi Finstad
:Date: 2015-12-01
:Version: 0.8
:Manual section: 3

SYNOPSIS
========

import saintmode;

DESCRIPTION
===========

This VMOD provides saintmode functionality for Varnish Cache 4.1.0 and
newer. The code is in part based on Poul-Henning Kamp's saintmode
implementation in Varnish 3.0.

Saintmode lets you deal with a backend that is failing in random ways
for specific requests. It maintains a blacklist per backend, marking
the backend as sick for specific objects. When the number of objects
marked as sick for a backend reaches a set threshold, the backend is
considered sick for all requests. Each blacklisted object carries a
TTL, which denotes the time it will stay blacklisted.

Saintmode in Varnish 4.1.0 is implemented as a director VMOD. We
instantiate a saintmode object and give it a backend as an
argument. The resulting object can then be used in place of the
backend, with the effect that it also has added saintmode
capabilities.

Example::

  backend b0 {
	.host = "foo";
	.port = "8080";
  }

  backend b1 {
  	.host = "bar";
  	.port = "8080";
  }

  sub vcl_init {
  	# Instantiate sm0, sm1 for backends b0, b1
  	new sm0 = saintmode.saintmode(b0, 10);
  	new sm1 = saintmode.saintmode(b1, 10);

	# Add both to a director. Use sm0, sm1 in place of b0, b1
  	new mydir = directors.random();
  	mydir.add_backend(sm0.backend(), 1);
  	mydir.add_backend(sm1.backend(), 1);
  }

  sub vcl_backend_fetch {
  	set bereq.backend = mydir.backend();
  }

  sub vcl_backend_response {
  	if (beresp.status == 500) {
  		# This marks the backend as sick for this specific
  		# object for the next 20s.
  		saintmode.blacklist(20s);
  		return (retry);
  	}
  }


FUNCTIONS
=========

saintmode.saintmode
-------------------

Prototype
::

	saintmode.saintmode(BACKEND b, INT threshold)

Description
	Constructs a saintmode director object. The ``threshold``
	argument sets the saintmode threshold, which is the maximum
	number of items that can be blacklisted before the whole
	backend is regarded as sick. Corresponds with the
	``saintmode_threshold`` parameter of Varnish 3.0.

Example
::

        sub vcl_init {
		new sm = saintmode.saintmode(b, 10);
	}


saintmode.backend()
-------------------

Prototype
::

	BACKEND saintmode.backend()

Description
	Used for assigning the backend from the saintmode object.

Example
::

	sub vcl_backend_fetch {
		set bereq.backend = sm.backend();
	}

blacklist()
-----------

Prototype
::

	VOID blacklist(DURATION expires)

Description
	Marks the backend as sick for a specific object. Used in
	vcl_backend_response. Corresponds to the use of
	``beresp.saintmode`` in Varnish 3.0. Only available in
	vcl_backend_response.

Example
::

	sub vcl_backend_response {
		if (beresp.http.broken-app) {
			saintmode.blacklist(20s);
			return (retry);
		}

	}

INSTALLATION
============

The source tree is based on autotools to configure the building, and
does also have the necessary bits in place to do functional unit tests
using the varnishtest tool.

Usage::

 ./configure

Make targets:

* make - builds the vmod
* make install - installs your vmod
* make check - runs the unit tests in ``src/tests/*.vtc``

COPYRIGHT
=========

This document is licensed under the same license as the
libvmod-saintmode project. See LICENSE for details.

* Copyright (c) 2011-2015 Varnish Software
