
.. impress::
   :func: linear
   :hide-title: false
   :data-scale: 5

=================================================
Managing a complex Odoo integration with buildout
=================================================

.. slide::
   :class: first

A configuration-based project assembly tool

G. Racinet, lead developer

Talk on https://github.com/gracinet/opendays-2014

Introduction
============

* Started three years ago (OpenERP 6)
* Homepage: http://pypi.python.org/pypi/anybox.recipe.openerp
* Documentation for all versions: http://docs.anybox.fr/anybox.recipe.openerp
* Active community
* Good quality: unit and integration tests, continuous integration,
  static analysis (pep8, pyflakes)
* Developer documentation

Plan of the talk
================

#. Motivation: integrator challenges
#. Quick tour and demos
#. Advanced features

Part 1/3: integrator challenges
===============================

Challenge: dependencies
=======================

In a big project, one may need

* several specific modules :

  * community addons (e.g, from OCA)
  * in-house mutualized modules (e.g, anybox_download_package)
  * final customer module(s)

* external Python libraries

  * interconnection (prestapyt, SOAPPy)
  * 


more on dependencies
====================

* specific versions of usual libraries (lxml, psycopg2…).
  Motivations:

  * bugfix
  * recent features (JSON for pyscopg2 >= 2.5)

* components from outside of the Odoo world, or even Python !

Challenge: configuration handling
=================================

What **is** configuration anyway ?

* configuration is a relative concept
* needs to be split, e.g, shared vs local

Transversal challenges
======================

* controlling versions for everything
* reliable reproduction
* uniformity

  * across our projects
  * across OpenERP versions

How much time wasted ?
======================

* Involving a new developer

  * regular one
  * punctual help, escalation

* Because of imprecise reproduction

  * test > staging > production
  * bug reproduction

Part 2/3: buildout tour
=======================

zc.buildout
===========

* designed from the ground up with all of these expectations for a
  world as rich and complex as OpenERP/Odoo

* very mature, lots of features (extensions and recipes)

* very generic design, can install anything, but

* natural bias towards python distributions

* There is a whole Linux distribution based on buildout: *SlapOS*

Simple configuration file
=========================

.. literalinclude:: demo/7_with_addons.cfg
   :language: ini

Running a buildout
==================

The first time::

  python bootstrap.py

Then::

  bin/buildout

Alternatively::

  bin/buildout -c my_config.cfg

(demo and description of the outcome)

Making variations
=================

Same project on latest 7.0 bzr branches

.. literalinclude:: demo/7_with_addons_bzr.cfg
   :language: ini

Oops, I forgot
==============

Sorry, now it's on github !

.. literalinclude:: demo/7_with_addons_github.cfg
   :language: ini

More variation
==============

On OpenERP Community Branch with my development version of the recipe

.. literalinclude:: demo/7_with_addons_ocb.cfg
   :language: ini

Smoothing transitions
=====================

Does the project work on Odoo ?

.. literalinclude:: demo/on_odoo.cfg
   :language: ini

Demo


A really big project
====================

.. literalinclude:: examples/mlf.cfg
   :language: ini

Separate version pinning
========================

See later for generation

.. literalinclude:: examples/mlf_revs.cfg
   :language: ini

Host-specific configuration
===========================

On the testing server :

.. literalinclude:: examples/mercure2.cfg
   :language: ini

anybox.recipe.odoo ?
====================

YES

* works already on Odoo, but Odoo still changes much
* will be a fresh start (drop legacy support)
* will be on github
* in about 2 weeks

Part 3/3: advanced features
===========================

Advanced features overview
==========================

* Scripts

  * written for OpenERP, e.g, batch jobs
  * test runners
  * instance upgrade & install

* Freeze and extraction

* Merges
  brand new feature by S. Rijnhart (Therp)

Dedicated scripts
=================

demo: interpreter and session

in ``my.script/my/script/main.py``::

  def run(session):
      admin = session.registry('res.users').browse(session.cr,
                                                   session.uid, 1)
      print(admin.password)

in ``my.script/setup.py``::

      entry_points="""

      [console_scripts]
      my_script = my.script.main:run
      """

in buildout config::

  [my_project]
  (...)
  eggs = my.script
  openerp_scripts = my_script


External script
===============

It works with any script provided by a Python distribution, but some
require database preloading.
Main use-case: test runners::

  [my_project]
  (...)
  eggs = nose
         coverage
         nose-cprof
  openerp_scripts = nosetests command-line-options=-d

Then run::

  bin/nosetests_myproject -d mydb -- [NOSE REGULAR OPTIONS & ARGUMENTS]


Now you can replay failed tests, stop with pdb on error, get profiling
and coverage info

Upgrade/install
===============

A special script with **one argument** for uniformity across projects

   bin/upgrade_my_project -d mydb

You have total control of what it does::

   def run_upgrade(session, logger):
       db_version = session.db_version  # state after latest upgrade
       if db_version < '1.0':
          session.update_modules(['account_account'])
       else:
          logger.warn("Not upgrading account_account, as we know it "
                      "to be currently a problem with our setup. ")
       session.update_modules(['crm', 'sales'])

Also can take care initial installations (part of the ``Session`` API)

Freeze
======
Release or pass what you **have right now** !

Get an extending buildout configuration that freezes everything::

  bin/buildout --offline my_project:freeze-to=frozen.cfg

Designed to be safe by rejecting local modifications

Extraction
==========
There are so many things that can go wrong or be too slow while
installing

A production server should not

* require access to your private VCS
* rely on launchpad or github availability

Command similar to freeze, produces a directory with

* the extracted VCS sources
* the ``release.cfg``

(Example on our buildbot)

Merges and autocommit
=====================
Make a derivative distribution and automerge it::

  [openerp]
  recipe = anybox.recipe.openerp[bzr]:server
  version = bzr lp:~anybox/ocb-server/7.0-anybox openerp last:1
  addons = bzr lp:~anybox/ocb-addons/7.0-anybox addons/ocb-addons last:1
           bzr lp:~anybox/ocb-web/7.0-anybox addons/ocb-web last:1 subdir=addons

  merges = bzr lp:ocb-server/7.0 parts/openerp last:1
           bzr lp:ocb-addons/7.0 addons/ocb-addons last:1
           bzr lp:ocb-web/7.0 addons/ocb-web last:1
  with_autocommit = true
  vcs-revert = on-merge

Commit/push is in a separate command::

  bin/autocommit_openerp

Our buildbot commits and pushes iff the tests pass.

Thank you !
===========

* Presentation made with Gawel's integration of impress.js in Sphinx:
  http://gawel.github.io/impress/

* Authors: C Combelles, G. Racinet

* External contributors:

  * L. Mignon, S. Bidoul (Acsone)
  * S. Rinjhart (Therp)
  * Y. Vaucher, G. Baconnier (Camp2camp)
  * L. Pistone, J.-E. Baudoux (Bcim)
